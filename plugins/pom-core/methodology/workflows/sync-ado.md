<purpose>
Bi-directional sync between POM Product Backlog items and Azure DevOps Boards work items.

POM is the strategic ledger (owns slice / acceptance / outcome). ADO is the execution surface (owns state / dates).

The PB-ID is the bridge.
</purpose>

<process>

<step name="parse_args">
Parse:
- `--push` (optional)
- `--pull` (optional)
- `--product=<slug>` (optional) — scope to one product
- `--dry-run` (optional) — report intentions; no writes

If neither `--push` nor `--pull` is set, default to push-then-pull.
</step>

<step name="precondition_checks">
Verify cwd is a POM repo (`intake/` and `products/` exist).

Check for the PAT environment variable. If `AZURE_DEVOPS_PAT` is unset:
```
AZURE_DEVOPS_PAT is not set. The Personal Access Token is required.

In PowerShell:  $env:AZURE_DEVOPS_PAT = "<your-pat>"
In bash:        export AZURE_DEVOPS_PAT="<your-pat>"

The PAT must have scope "Work Items: Read & Write".

NEVER commit the PAT or write it to .pom/sync-ado.config.json — it lives only in the environment.
```
Then abort.

Verify `.pom/sync-ado.config.json` exists. If not, prompt the user via `AskUserQuestion` for:
- Organization URL (e.g., `https://dev.azure.com/contoso`)
- Project name
- Area path (e.g., `Contoso\\Claims`)
- Work item type (default `User Story`; alternatives `Feature`, `Product Backlog Item`)
- Iteration path (optional)

Write the answers to `.pom/sync-ado.config.json` (commit-safe — contains NO credentials).
</step>

<step name="load_config">
Read `.pom/sync-ado.config.json`. Validate fields. If `organization_url` doesn't match `https://dev.azure.com/<org>` pattern or `https://<org>.visualstudio.com/`, abort with a clear error.
</step>

<step name="scan_pb_items">
Glob `products/<slug>/product-backlog/PB-*.md` (scoped to `--product` if provided; otherwise all products).

For each PB file, parse the YAML frontmatter / metadata table to extract:
- PB-ID
- Outcome
- Slice description
- Acceptance signals (list)
- Out-of-scope list
- Linked runway / ADRs
- Originating DISC and UC
- `ado_work_item_id` field (presence indicates it's already synced)
- `status` field (POM-side: `shaped`, `pulled`, `in-flight`, `shipped`)
- `started`, `completed` fields (POM-side dates, only populated by `--pull`)
</step>

<step name="push_phase">
**Only if `--push` is active.**

For each PB item WITHOUT an `ado_work_item_id`:

1. **Verify gate provenance.** Read the linked DISC file; confirm it was 4/4 ✅ at promotion. If not, refuse to push this item (this should never happen — `pom-promote-to-backlog` enforces the gate — but verify defensively).

2. **Build the ADO payload** as JSON Patch document (ADO REST API convention):
   ```json
   [
     {"op": "add", "path": "/fields/System.Title", "value": "PB-YYYY-MM-<slug>: <outcome>"},
     {"op": "add", "path": "/fields/System.AreaPath", "value": "<area_path>"},
     {"op": "add", "path": "/fields/System.IterationPath", "value": "<iteration_path if set>"},
     {"op": "add", "path": "/fields/System.Tags", "value": "pom; PB-YYYY-MM-<slug>; <product-slug>"},
     {"op": "add", "path": "/fields/System.Description", "value": "<HTML-escaped slice contract + originating DISC/UC + out-of-scope>"},
     {"op": "add", "path": "/fields/Microsoft.VSTS.Common.AcceptanceCriteria", "value": "<HTML <ul><li> list of acceptance signals>"}
   ]
   ```

3. **POST to ADO REST API** (via `Bash` + `curl`, OR via the `azure-devops` skill if available):
   ```
   POST {organization_url}/{project}/_apis/wit/workitems/${type}?api-version=7.1
   Headers: Content-Type: application/json-patch+json
            Authorization: Basic base64(":${AZURE_DEVOPS_PAT}")
   Body: <JSON Patch document>
   ```

4. **On success:** parse the returned work item ID (`response.id`). Edit the PB markdown to insert/update an `ado_work_item_id: <id>` field plus an `ado_url: <portal URL>` field. Append a one-line sync log entry to the PB file's "Sync history" section.

5. **On failure:** report the API error verbatim. Do NOT retry automatically. Do NOT modify the PB file.

For each PB item WITH an `ado_work_item_id`:
- Skip in `--push` mode unless a drift is detected (Phase: diff_warn). POM never auto-overwrites ADO content for already-synced items.

If `--dry-run`: skip steps 3-5; instead print the JSON Patch document that WOULD be sent, per item.
</step>

<step name="pull_phase">
**Only if `--pull` is active.**

For each PB item WITH an `ado_work_item_id`:

1. **GET the work item** from ADO:
   ```
   GET {organization_url}/{project}/_apis/wit/workitems/{id}?api-version=7.1
   Headers: Authorization: Basic base64(":${AZURE_DEVOPS_PAT}")
   ```

2. **Map ADO state → POM status** using this canonical table:
   | ADO State | POM `status` |
   |---|---|
   | New | shaped |
   | Active | in-flight |
   | Resolved | in-flight |
   | Closed | shipped |
   | Removed | (refuse; flag for human resolution — ADO doesn't delete, POM doesn't auto-archive) |

3. **Extract dates:**
   - `Microsoft.VSTS.Common.ActivatedDate` → POM `started`
   - `Microsoft.VSTS.Common.ClosedDate` → POM `completed`

4. **Update PB markdown ONLY** for the three owned fields (`status`, `started`, `completed`). Never touch outcome, slice, acceptance signals, out-of-scope, or runway links — those are POM-owned and immutable from this direction.

5. **Append a sync log entry** to the PB file's "Sync history" section: date, pulled fields, old → new.

If `--dry-run`: print the would-be field updates per item; do not edit.
</step>

<step name="diff_warn">
**Always runs (regardless of --push/--pull).**

For each PB item with `ado_work_item_id`, fetch (or reuse from pull phase) the ADO work item title and description and check:

1. **Title drift:** if ADO's title does not start with the PB-ID → emit a warning:
   ```
   ⚠️  Title drift on PB-YYYY-MM-<slug>:
         POM expects: "PB-YYYY-MM-<slug>: <outcome>"
         ADO has:     "<actual ADO title>"
       Action: scope may have changed. Update the PB file if intentional, or update ADO title to restore the prefix.
   ```

2. **Closed-but-shaped:** if ADO state is `Closed` but POM `status` is still `shaped` → emit a warning suggesting `--pull` to reconcile.

3. **Description-edited:** compute a content hash of the slice-contract section of ADO's description against the originating PB file's slice. If different → emit a warning. Never auto-rewrite.

Warnings DO NOT halt the run. They are accumulated and printed in the report.
</step>

<step name="update_config">
On successful run (non-dry-run), update `.pom/sync-ado.config.json`:
- `last_pushed_at`: ISO timestamp if push phase ran
- `last_pulled_at`: ISO timestamp if pull phase ran

Never write credentials. Never write per-item state (that lives in the PB markdown).
</step>

<step name="report">
```
✅ pom-sync-ado complete.

Mode: {{push|pull|push+pull|dry-run}}
Scope: {{all products | --product=<slug>}}

Push phase:
  Created: {{N}} new ADO work item(s)
    - PB-YYYY-MM-<slug> → ADO #{{id}}: {{title}}
    ...
  Skipped (already synced): {{N}}

Pull phase:
  Updated: {{N}} PB file(s)
    - PB-YYYY-MM-<slug>: status {{old}} → {{new}}, started {{...}}, completed {{...}}
    ...
  No-op (no state change): {{N}}

Diff warnings:
{{Numbered list of drift findings, or "None — POM and ADO are in agreement on titles and slice contracts."}}

Next steps:
- /pom-status --since=last to see what changed since this sync's snapshot.
- /pom-render to refresh portfolio.html with the new status data.
- /pom-sync-ado --pull on schedule (daily/weekly cron) to keep POM up to date with execution state.
```
</step>

</process>

<guardrails>
- **NEVER write the PAT to disk.** It lives only in `AZURE_DEVOPS_PAT` env var. If found in config or PB markdown, abort with a security warning.
- **NEVER cross the authority boundary.** Push: POM-owned fields only (title, description, acceptance, tags). Pull: ADO-owned fields only (state, ActivatedDate, ClosedDate). Crossing the boundary in either direction = FAIL.
- **NEVER delete an ADO work item.** Removal is human-only, in ADO's UI. POM only creates and reads.
- **NEVER auto-rewrite ADO content** that has been hand-edited away from the slice contract. Warn instead; the operator decides.
- **NEVER push a PB item whose originating DISC is not 4/4 ✅.** Gate enforcement carries through.
- **`--dry-run` short-circuits BEFORE any API call or file edit.** It is the safe planning mode.
- Network errors are reported verbatim — never silently swallowed.
- `.pom/sync-ado.config.json` is commit-safe (contains NO credentials). The PAT lives only in the env var.
</guardrails>

<success_criteria>
- [ ] `AZURE_DEVOPS_PAT` is read from environment, never written to disk
- [ ] Push creates ADO work items with PB-ID prefix in title and slice contract in description
- [ ] Push records the returned work item ID into the PB markdown (and only that field — no other PB edits)
- [ ] Pull updates only `status`, `started`, `completed` in PB markdown
- [ ] Diff warnings name specific drift cases and do not halt the run
- [ ] `--dry-run` makes zero API calls and zero file edits
- [ ] PB items without an originating 4/4 ✅ DISC are refused for push
- [ ] On network failure, no partial state is written
</success_criteria>
