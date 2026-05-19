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
```
</step>

</process>

<guardrails>
- **READ-ONLY.** Never modifies the target repo.
- Counts must be derived from filesystem, not invented.
- Open blockers must point to real files / states, not fabricated.
- Output is plain text — no ANSI color, no HTML, < 80 columns wide.
- If a section has zero items, render "(none)" rather than omitting the section — readers expect consistent shape.
</guardrails>

<success_criteria>
- [ ] Output produces accurate counts matching filesystem
- [ ] Recent items list reflects actual most-recent files
- [ ] Open blockers correspond to real states
- [ ] Format is copy-pasteable into a status update
- [ ] No modifications to target repo
</success_criteria>
