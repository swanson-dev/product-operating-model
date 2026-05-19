---
name: pom-sync-ado
description: "Use when a POM portfolio needs bi-directional sync with Azure DevOps Boards. Pushes promoted Product Backlog items into ADO as Features/User Stories (PB-ID in title, slice contract in description, acceptance signals as checkboxes); pulls status and dates back into the PB markdown. Requires AZURE_DEVOPS_PAT env var. POM remains authoritative for slice/acceptance; ADO is authoritative for status/dates."
argument-hint: "[--push] [--pull] [--product=<slug>] [--dry-run]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Synchronise POM Product Backlog items with Azure DevOps Boards work items. POM is the strategic ledger; ADO is the execution surface. The PB-ID is the bridge.

**Modes:**
- `--push` (default if neither flag set): create ADO work items for PB items not yet synced (no `ado_work_item_id` field). PB items already synced are skipped — subsequent POM-side edits to slice / acceptance / out-of-scope are NOT propagated by this skill (the `diff_warn` phase will flag drift; resolving it is operator-driven). The skill writes to ADO and stores the returned work item ID in the PB markdown.
- `--pull`: read status, started date, and completed date from synced ADO work items; update the corresponding PB markdown fields.
- (Both): push then pull, in that order. Default behaviour.
- `--product=<slug>`: scope to one product.
- `--dry-run`: report what would happen; make no API calls and no file edits.

**Authority boundaries (NEVER cross these):**
- **POM owns** (push direction): work item `title`, `description` (slice contract + acceptance + out-of-scope), `acceptance criteria` (checkbox list), tags (`pom`, PB-ID, product slug). Originating UC/DISC IDs in description for traceability.
- **ADO owns** (pull direction): work item `state` (New / Active / Resolved / Closed), `Microsoft.VSTS.Common.ActivatedDate` (mapped to PB `started`), `Microsoft.VSTS.Common.ClosedDate` (mapped to PB `completed`). Never sync ADO's `title` back into POM — that direction is one-way only.

**Configuration:**
- `.pom/sync-ado.config.json` (committed) — stores: `organization_url`, `project`, `area_path`, `iteration_path` (optional), `work_item_type` (default `User Story`), `last_pushed_at` and `last_pulled_at` (one or both updated per run, depending on which phases ran).
- `AZURE_DEVOPS_PAT` environment variable (NEVER written to disk) — the Personal Access Token with `Work Items: Read & Write` scope. If missing, the skill aborts with setup instructions.

**Diff-warn (advisory, not auto-resolved):**
- If an ADO work item's `title` no longer starts with the PB-ID → scope drift warning.
- If ADO's description has been edited away from the slice contract → log a warning but DO NOT overwrite (operator decides).
- If a PB item is `Closed` in ADO but its PB markdown `status` is still `shaped` → flag for `pull` to reconcile.

**Refuses:**
- To push a PB item that does not yet exist as a file (no phantom work items).
- To push a PB item whose originating DISC is not 4/4 ✅ (gate enforcement carries through).
- To delete ADO work items (use ADO's UI for any deletes — POM never destroys execution-side state).
- To run without `AZURE_DEVOPS_PAT` set (the only credential needed; never written).

**After this command:**
- Engineers see the PB items as Features/User Stories in ADO Boards with PB-IDs as titles.
- POM remains the strategic ledger; PB markdown carries the ADO work item ID for traceability.
- Run `/pom-status --since=last` to see what changed since the last sync.
- Run `/pom-render` to refresh the cockpit with new status data.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/sync-ado.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
</execution_context>

<process>
Execute end-to-end.
Enforce the authority boundaries — never write into ADO fields POM doesn't own, never overwrite POM fields ADO owns.
Always honour `--dry-run` by short-circuiting BEFORE any API call or file edit.
</process>
