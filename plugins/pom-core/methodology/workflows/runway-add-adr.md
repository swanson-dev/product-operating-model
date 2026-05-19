<purpose>
Record an Architecture Decision Record (ADR) in a product's runway/. Sequential numbering per product. Immutable content once written (supersession via new ADR, not edit).

For a single binding architectural decision: context, options, decision, consequences.
</purpose>

<process>

<step name="parse_args">
Parse:
- First positional: `$PRODUCT_SLUG` (required)
- Second positional: `$ADR_SLUG` (required) — kebab-case short descriptor
- `--supersedes=ADR-NNNN` (optional) — if this ADR reverses a prior one
</step>

<step name="validate_slugs">
Both must be kebab-case. Abort with corrected suggestion if invalid.
</step>

<step name="precondition_checks">
Verify `products/$PRODUCT_SLUG/runway/` exists. If not, abort and suggest `pom-spinup-product` first.

Check for slug collision: glob `products/$PRODUCT_SLUG/runway/ADR-*-$ADR_SLUG.md`. If any match, refuse and prompt for a different slug.
</step>

<step name="compute_adr_number">
List existing files in `products/$PRODUCT_SLUG/runway/` matching `ADR-NNNN-*.md`. Extract numbers; take max + 1. Zero-pad to 4 digits → `$ADR_NUMBER`.

If no ADRs yet, $ADR_NUMBER = `0001`.
</step>

<step name="gather_fields">
Use `AskUserQuestion` to collect:

1. **Context and problem statement** — the situation forcing a decision (constraints: technical, organizational, regulatory)
2. **Decision drivers** — 2-5 quality attributes or constraints that shape the choice
3. **Considered options** — at least 2; for each: name, short description, pros, cons
4. **Decision outcome** — which option wins, with rationale
5. **Consequences** — positive, negative, neutral/follow-up
6. **Linked artifacts** — originating PB item or runway investment; related patterns

If `--supersedes` was provided, verify the referenced ADR exists.
</step>

<step name="write_adr_file">
Generate from `the pom-core plugin's methodology/templates/products/_template/runway/ADR-template.md`. Replace placeholders:

- `{{title}}` → derived from context (one-line summary)
- ADR ID matches filename
- Date = today
- Status = `Accepted` (default; user can change via flag in future iteration)
- Deciders = Trio TL (read from trio.md) + any others mentioned in fields
- Product = `$PRODUCT_SLUG`
- All fields filled per user input
- If `--supersedes`: add a `Supersedes: ADR-NNNN-<prior-slug>.md` line in the header table

Write to `products/$PRODUCT_SLUG/runway/ADR-$ADR_NUMBER-$ADR_SLUG.md`.
</step>

<step name="update_superseded_adr">
If `--supersedes` was provided:
- Read the superseded ADR.
- Update ONLY its `Status` field: `Superseded by ADR-$ADR_NUMBER-$ADR_SLUG`.
- Do NOT modify any other content.
</step>

<step name="update_runway_plan">
If `products/$PRODUCT_SLUG/runway/runway-plan.md` exists:
- Add a new row to the "Recent decisions" table with the new ADR.
- If superseding, also update the prior ADR's row to show its new "Superseded" status.

If runway-plan.md does NOT exist:
- Note in the report that the user should run `pom-runway-plan $PRODUCT_SLUG` to create it.
</step>

<step name="draft_stakeholder_summary">
Generate a draft **stakeholder-facing summary** (one paragraph any reader can paste):
- State the decision in plain English (chosen option) and what it replaces, if anything.
- Give the one-line rationale (the dominant decision driver).
- Name the most important positive consequence and the most important trade-off / negative consequence.
- If superseding: explicitly note the prior ADR being superseded and why.

Show the draft via chat and use `AskUserQuestion` to confirm or refine.
</step>

<step name="report">
```
✅ ADR-$ADR_NUMBER recorded.

Files:
- products/$PRODUCT_SLUG/runway/ADR-$ADR_NUMBER-$ADR_SLUG.md (new)
{{If supersedes:}}
- products/$PRODUCT_SLUG/runway/ADR-NNNN-<prior>.md (Status updated to "Superseded by ADR-$ADR_NUMBER")
{{If runway-plan exists:}}
- products/$PRODUCT_SLUG/runway/runway-plan.md (Recent decisions table updated)

Decision: {{title}}
Chosen: {{chosen option}}

Decision summary (paste-ready):
─────────────────────────────────
{{stakeholder-facing summary}}
─────────────────────────────────

Next steps:
{{If runway-plan doesn't exist:}}
- Run `/pom-runway-plan $PRODUCT_SLUG` to create a living runway plan.
{{Else:}}
- Mention this ADR at the next Trio sync.
- If implementing requires runway investment, add it to the runway-plan's "Upcoming runway investments" table.
```
</step>

</process>

<guardrails>
- **NEVER edit a prior ADR's content.** Supersession only updates the Status field of the prior ADR.
- ALWAYS require at least 2 considered options.
- ALWAYS document consequences (positive AND negative).
- Numbering is contiguous per product — never skip numbers.
- ALWAYS reflect the new ADR in runway-plan.md if it exists.
</guardrails>

<success_criteria>
- [ ] ADR file written with sequential number and valid slug
- [ ] All template sections filled
- [ ] If supersession, prior ADR Status updated (content unchanged)
- [ ] runway-plan.md reflects the new ADR (if it exists)
- [ ] Stakeholder-facing decision summary is paste-ready in the CLI report
- [ ] `pom-validate` continues to pass (R4.1, R4.2)
</success_criteria>
