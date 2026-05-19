<purpose>
Promote a 4/4 ✅ Discovery item to its product's Product Backlog. Writes a `PB-YYYY-MM-<slug>.md` file with the vertical slice definition, acceptance signals, runway prerequisites, and out-of-scope. Updates the DISC status to "Promoted to Product Backlog YYYY-MM-DD" with a link to the PB file.

**Refuses** to promote items that haven't passed the four-question gate.
</purpose>

<process>

<step name="parse_args">
Parse:
- First positional: `$DISC_ID` (required) — Discovery ID or full path to DISC file
</step>

<step name="locate_disc_file">
If `$DISC_ID` is a path, use it. Otherwise glob `products/*/discovery-backlog/$DISC_ID*.md`. Expect exactly one match.

Set `$DISC_PATH` and infer `$PRODUCT_SLUG` from the path.
</step>

<step name="check_promotion_readiness">
Read `$DISC_PATH`. Parse the Promotion Readiness table:
- Q1 Desirable
- Q2 Viable
- Q3 Feasible
- Q4 Ethical / compliant
- Trio-Note gap 1
- Trio-Note gap 2 (if present)

ALL must be ✅. If not, abort:
```
NOT READY for promotion: {{specific gap}}.

Run `/pom-discovery-gate $DISC_ID` to address the open questions.
```

**Stop.** Do not promote.
</step>

<step name="check_duplicate">
Glob `products/$PRODUCT_SLUG/product-backlog/PB-*.md`. For each, check if it references `$DISC_ID` in its "Originating Discovery item" field.

If a PB already exists for this DISC, prompt via `AskUserQuestion`:
- "PB-XYZ already references this Discovery. Is this a re-shape (create a new PB with different slug)? Or stop?"
</step>

<step name="gather_slice_definition">
Use `AskUserQuestion` to collect:

1. **Outcome** (one sentence) — must reference a specific outcome from `products/$PRODUCT_SLUG/product.md`. Show the product's outcomes list when prompting.
2. **Vertical slice definition** (≥ 3 sentences) — thin end-to-end customer journey. What the user does start-to-finish, what data flows, what systems touch. Be specific about what is and is NOT in the slice.
3. **Acceptance signals** (≥ 2 measurable signals) — how the Trio will know the slice delivered the outcome. Mix of measurable + behavioral + operational signals.
4. **Runway prerequisites** — list ADRs, patterns, or platform services this slice depends on. For each, link to the existing artifact OR flag it as "must be created first".
5. **Out of scope for this slice** — explicit exclusions to prevent scope creep mid-flight.
6. **Dependencies on other PB items** — if any.
</step>

<step name="compute_pb_filename">
Derive a PB slug from the originating UC's title (or ask the user to provide one).

$PB_MONTH = current YYYY-MM.

Filename: `PB-$PB_MONTH-$PB_SLUG.md`.
</step>

<step name="write_pb_file">
Read `~/.knowledge/pom-methodology/templates/products/_template/product-backlog/PB-template.md` (correcting to `the pom-core plugin's methodology/templates/products/_template/product-backlog/PB-template.md`). Replace placeholders:

- `{{slug}}` → PB slug
- `{{title}}` → derived from outcome
- PB ID matches filename
- Originating Discovery item → relative path back to `$DISC_PATH`
- Promoted on → today
- Promoted by → Trio (read from `products/$PRODUCT_SLUG/trio.md`)
- Current state → `shaped`
- Pulled by pod → blank
- Outcome → user-provided
- Vertical slice definition → user-provided
- Acceptance signals → user-provided
- Runway prerequisites → user-provided
- Out of scope → user-provided
- Dependencies → user-provided

Write to `products/$PRODUCT_SLUG/product-backlog/PB-$PB_MONTH-$PB_SLUG.md`.
</step>

<step name="update_disc_file">
Edit `$DISC_PATH`:
- `Shaping status` field updated to: "Promoted to Product Backlog YYYY-MM-DD"
- Add a "Promotion target" field (or update existing) with relative path to the new PB file
- Append shaping log entry dated today: "Promoted to product-backlog/PB-..."
</step>

<step name="update_portfolio_index">
Edit `products/README.md`. Update the row for `$PRODUCT_SLUG`:
- Increment Product Backlog count (or set to 1)
</step>

<step name="report">
```
✅ Promoted to Product Backlog.

Files:
- products/$PRODUCT_SLUG/product-backlog/PB-$PB_MONTH-$PB_SLUG.md (new)
- products/$PRODUCT_SLUG/discovery-backlog/<DISC-FILE> (status + log updated)
- products/README.md (portfolio index updated)

Vertical slice summary:
─────────────────────────────────
{{outcome + slice description}}
─────────────────────────────────

Next steps:
{{If no pods exist for this product:}}
- Run `/pom-form-pod $PRODUCT_SLUG <pod-slug>` to stand up a pod.
{{Else:}}
- The PO Sync can now order this PB item for a pod to pull.
- Run `/pom-validate` to confirm structural integrity.
```
</step>

</process>

<guardrails>
- **NEVER promote a DISC with any question still ❌ or 🟡.** The four-question gate is the load-bearing rule.
- **NEVER promote a DISC with open Trio-Note gaps.** Those must close first.
- **NEVER edit the DISC file's prior shaping log entries.** Append only.
- ALWAYS require a measurable outcome, a vertical slice (not a task list), and ≥ 2 acceptance signals.
</guardrails>

<success_criteria>
- [ ] PB file written with all required fields per `PB-template.md`
- [ ] DISC file status updated; promotion log entry appended
- [ ] Portfolio index updated
- [ ] `pom-validate` continues to pass (R7.4 cross-reference holds)
- [ ] User given a concrete next step (form pod or PO Sync)
</success_criteria>
