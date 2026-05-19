<purpose>
Generate a human-readable portfolio summary for a POM repo. Read-only.

Text-first, copy-pasteable into stakeholder updates. Counts derived from filesystem state. Surfaces open blockers prominently so the reader knows where to look.
</purpose>

<process>

<step name="parse_args">
Parse:
- First positional: `$TARGET_DIR` (optional, defaults to cwd)
- `--product=<slug>` (optional) — scope to a single product
- `--recent=<N>` (optional, default 5) — number of recent items in lists
- `--since=<date|last>` (optional) — diff mode. `<date>` is ISO 8601 (`YYYY-MM-DD`); `last` resolves to the newest snapshot under `.pom/snapshots/`. When set, the report includes a CHANGES SINCE section and a fresh snapshot is written at the end of the run.
</step>

<step name="precondition_checks">
Verify `$TARGET_DIR` is a POM repo. If not, abort with:
```
$TARGET_DIR is not a POM repo (missing intake/, products/, or other required dirs).

Run `pom-bootstrap` first.
```

If `--product=<slug>` provided, verify `products/<slug>/` exists.
</step>

<step name="scan_intake">
Scan `$TARGET_DIR/intake/use-case-backlog/` (excluding `README.md` and `UC-template.md`):
- Count total UC files
- For each UC, parse the Status field — bucket into: pending, scored, dispositioned (kill/park/route/split), routed

Scan `$TARGET_DIR/intake/dispositions/`:
- Most recent N disposition files by filename date (DISP-YYYY-MM-DD-...)
- For each: extract decision, UC reference, paste-ready stakeholder summary (one-line excerpt)
</step>

<step name="scan_products">
For each subdirectory in `$TARGET_DIR/products/` (excluding `_template`):
- Read `trio.md`: count assigned vs. unassigned roles (PM, PD, TL)
- Count `discovery-backlog/` files (DISC-*.md, excluding template)
  - For each DISC, parse Promotion Readiness table — bucket into: 0/4, 1/4, 2/4, 3/4, 4/4 (READY)
- Count `product-backlog/` files (PB-*.md)
- Count `pods/<slug>/` (excluding `_template`)
- Count `runway/ADR-*.md` and `runway/PATTERN-*.md`
</step>

<step name="scan_enabling">
For each subdirectory in `$TARGET_DIR/enabling/` (excluding `README.md`):
- Note presence of the concern
- Count `decision-log/DEC-*.md` files
- Most recent N decisions by filename date across all concerns
</step>

<step name="scan_platform">
For each canonical area (and any custom area):
- Count services (subdirs of the area, excluding `_service-template`)
- Note status of each (read each service's README for the Status field)
</step>

<step name="snapshot_current_state">
**Only runs when `--since` is set.**

Build an in-memory snapshot of current portfolio state:

```json
{
  "taken_at": "YYYY-MM-DDTHH:MM:SSZ",
  "target": "$TARGET_DIR",
  "artifacts": [
    {
      "id": "UC-0042",
      "type": "UC",
      "path": "intake/use-case-backlog/UC-0042-...md",
      "status": "scored",
      "created_at": "YYYY-MM-DD",
      "owners": ["J. Park"],
      "extra": { "wsjf": 21 }
    },
    {
      "id": "DISC-2026-05-claims-doc-ai",
      "type": "DISC",
      "path": "products/claims/discovery-backlog/...md",
      "status": "Q1✅/Q2🟡/Q3✅/Q4✅",
      "verdict": "NOT READY",
      "blocker": "actuarial sign-off",
      "owners": ["M. Park", "J. Lee", "D. Chen"],
      "created_at": "YYYY-MM-DD"
    }
    // ... one entry per UC / DISP / DISC / PB / ADR / DEC / pod / product
  ]
}
```

Use the same parsing logic as `pom-explain` so the two skills agree on field names and shapes.

Hold the snapshot in memory through this run; do not write it yet (write happens after the diff so a failed diff doesn't leave a partial snapshot).
</step>

<step name="resolve_baseline">
**Only runs when `--since` is set.**

Resolve `--since` to a baseline snapshot:

- If `--since=last`:
  - Glob `.pom/snapshots/*.json`. Pick the newest by filename (filenames are ISO timestamps, so lexicographic = chronological).
  - If none exist: report "No prior snapshot found. This run will write the first snapshot — no diff to show this time." Skip `compute_diff`. Still write the snapshot at the end.
- If `--since=YYYY-MM-DD`:
  - Glob `.pom/snapshots/*.json`. Pick the newest snapshot whose `taken_at` is `<= YYYY-MM-DDT23:59:59Z`.
  - If none qualifies (every snapshot is newer than the requested date): treat the same as "no baseline" — report "No snapshot exists on or before YYYY-MM-DD. Skipping diff this run." and skip `compute_diff`. Never silently diff against a future-of-requested-date baseline (that would hide real changes between the requested date and the chosen later baseline).
  - If no snapshots exist at all: report and skip diff (same as `--since=last` empty case).

Load the chosen baseline snapshot from disk.
</step>

<step name="compute_diff">
**Only runs when `--since` is set AND a baseline was resolved.**

Compute the artifact-level diff between baseline and current:

- **Added**: artifact ID in current but not in baseline. Bucket by type.
- **Removed**: artifact ID in baseline but not in current. (Rare — usually means a UC was renamed or a snapshot was taken mid-edit. Report as "removed/renamed" with the baseline path.)
- **Changed**: artifact ID in both, but `status` / `verdict` / `blocker` / `owners` differ. Report old → new for the changed fields.
- **Unchanged**: skip silently.

Group changes into stakeholder-meaningful events:
- `Promoted to Product Backlog` (DISC `verdict` flipped to READY → new PB exists with cross-ref)
- `Disposition recorded` (UC `status` flipped + new DISP added)
- `Discovery gate advanced` (DISC Q-state changed)
- `New ADR` (ADR type added)
- `New decision logged` (DEC type added)
- `Pod composition changed` (pod artifact `owners` differs)
- `Trio updated` (product `owners` differs)
</step>

<step name="detect_blockers">
Compute open blockers:
- For each product, if Trio has any unassigned role → "Trio incomplete in <product>"
- For each DISC at 0/4, 1/4, 2/4, or 3/4 → "Discovery shaping in progress: <DISC>" (count, don't list all unless < 5)
- For each PB with state `shaped` (not yet pulled) → "PB ready for pull: <PB>"
- For each ADR with status `Proposed` → "Open architectural decision: <ADR>"
- For each enabling concern marked v0.1 scaffold (in README) → "Concern awaiting authoritative policy: <concern>"
</step>

<step name="render_full_report">
If `--product` was NOT provided, render full portfolio summary:

```
POM Portfolio Status
─────────────────────
Target: $TARGET_DIR
Date:   YYYY-MM-DD HH:MM

INTAKE
  Total UCs:           {{total}}
  Pending disposition: {{n}}
  Scored:              {{n}}
  Routed:              {{n}}
  Parked:              {{n}}
  Killed:              {{n}}

PRODUCTS
  Product   | Trio   | Discovery | PB | Pods | Notes
  ----------|--------|-----------|----|------|------------------------------
  {{slug}}  | {{🟡/✅}} | {{count}}  | {{c}} | {{c}}  | {{any Trio gap or note}}
  ...

RECENT DISPOSITIONS (last {{N}})
  YYYY-MM-DD | UC-NNNN | route → claims  | "{{stakeholder summary first line}}"
  ...

RECENT DECISIONS (last {{N}})
  YYYY-MM-DD | enabling/ai-ethics | "{{decision summary}}"
  ...

RECENT ADRs (last {{N}})
  YYYY-MM-DD | products/claims | ADR-0001 | "{{adr title}}"
  ...

PLATFORM SERVICES
  {{area}}/{{service}}: {{status}} ({{N products consuming}})
  ...

OPEN BLOCKERS
  • Trio incomplete in {{N}} product(s): {{list}}
  • {{N}} Discovery item(s) below 4/4: see /pom-discovery-gate for details
  • {{N}} Proposed ADR(s) awaiting decision
  • {{N}} enabling concern(s) at v0.1 scaffold (awaiting authoritative policy)

{{If --since was set AND a baseline existed:}}
CHANGES SINCE {{baseline taken_at}}
  Added:
    • {{count}} new UC(s){{ — list IDs if ≤ 5}}
    • {{count}} new disposition(s){{ — list if ≤ 5}}
    • {{count}} new ADR(s){{ — list if ≤ 5}}
    • {{count}} new decision(s){{ — list if ≤ 5}}
    • {{count}} new PB item(s){{ — list if ≤ 5}}
  Promotions:
    • {{DISC-ID}} → PB-... ({{product}})
    ...
  Discovery gate advanced:
    • {{DISC-ID}}: {{old states}} → {{new states}}
    ...
  Other changes:
    • {{Pod composition / Trio / ADR status changes}}
    ...
  Removed / renamed:
    • {{baseline path}} no longer present (likely renamed)
    ...

─────────────────────────────────────────────
```
</step>

<step name="render_product_report">
If `--product=<slug>` was provided, render product-scoped summary:

```
POM Product Status: {{slug}}
───────────────────────────────
Target: $TARGET_DIR/products/{{slug}}
Date:   YYYY-MM-DD HH:MM

TRIO
  PM:  {{name or _Unassigned_}}
  PD:  {{name or _Unassigned_}}
  TL:  {{name or _Unassigned_}}
  Exec sponsor: {{name or _Unassigned_}}

ROADMAP
  Now:    {{n outcomes}}
  Next:   {{n}}
  Later:  {{n}}
  Won't:  {{n}}

DISCOVERY BACKLOG
  Total: {{n}}
  By gate state:
    0/4: {{n}}    1/4: {{n}}    2/4: {{n}}
    3/4: {{n}}    4/4 (READY): {{n}}

PRODUCT BACKLOG
  Total: {{n}}
  Shaped (not pulled): {{n}}
  Pulled / In flight:  {{n}}
  Shipped:             {{n}}

RUNWAY
  ADRs:     {{n}}  (most recent: ADR-NNNN — "{{title}}")
  Patterns: {{n}}
  Runway plan last updated: YYYY-MM-DD

PODS
  Total: {{n}}
  {{For each pod: name | SM | PO | members count | surface}}

OPEN BLOCKERS (product-scoped)
  • {{specific blockers for this product}}

─────────────────────────────────────────────
```
</step>

<step name="write_snapshot">
**Only runs when `--since` was set.**

Ensure `$TARGET_DIR/.pom/snapshots/` exists (create both `.pom/` and `snapshots/` if missing).

Write the in-memory snapshot from `snapshot_current_state` to:
```
$TARGET_DIR/.pom/snapshots/{{taken_at-as-ISO-with-colons-replaced-by-dashes}}.json
```

Filename example: `2026-05-19T14-32-07Z.json`. (Colons replaced with dashes for cross-platform filename safety.)

Pretty-print JSON (2-space indent) so diffs are git-friendly. Snapshots ARE meant to be committed — they're the portfolio's heartbeat.

After write, print one line to the report:
```
Snapshot written: .pom/snapshots/<filename>
```
</step>

<step name="report">
Print the rendered summary directly (no further commentary). The output IS the report.

If `--product` was scoped, end with:
```
Run `/pom-status $TARGET_DIR` (no --product) for the portfolio view.
```

Otherwise end with:
```
Run `/pom-status $TARGET_DIR --product=<slug>` for product-scoped detail.
Run `/pom-validate $TARGET_DIR` for structural conformance check.
Run `/pom-status $TARGET_DIR --since=last` to see what changed since the last snapshot.
```
</step>

</process>

<guardrails>
- **READ-ONLY on portfolio artifacts.** Never modifies any UC, DISP, DISC, PB, ADR, DEC, pod, product, platform, or enabling-concern file.
- **`.pom/` metadata is the ONLY allowed write target**, and only when `--since` is set. Snapshots are portfolio heartbeats — committed to git intentionally.
- Counts must be derived from filesystem, not invented.
- Open blockers must point to real files / states, not fabricated.
- Output is plain text — no ANSI color, no HTML, < 80 columns wide.
- If a section has zero items, render "(none)" rather than omitting the section — readers expect consistent shape.
- The `CHANGES SINCE` section only appears when `--since` was set AND a baseline snapshot existed. The first `--since=last` run writes a snapshot but cannot diff.
</guardrails>

<success_criteria>
- [ ] Output produces accurate counts matching filesystem
- [ ] Recent items list reflects actual most-recent files
- [ ] Open blockers correspond to real states
- [ ] Format is copy-pasteable into a status update
- [ ] No modifications to portfolio artifacts (only `.pom/snapshots/` when `--since` is used)
- [ ] When `--since` is set: a snapshot is written; the CHANGES SINCE section is rendered if a baseline existed
- [ ] Snapshot JSON is pretty-printed (2-space indent) for git-friendly diffs
</success_criteria>
