<purpose>
Create or refresh a product's `runway-plan.md` — the living document that lists current commitments, recent decisions (ADRs), active patterns, upcoming runway investments, capacity assumptions, and open questions.

Preserves user-authored content in qualitative sections (commitments, assumptions, open questions). Re-scans and refreshes the auto-generated tables (Recent decisions, Active patterns) from the runway/ folder.
</purpose>

<process>

<step name="parse_args">
Parse:
- First positional: `$PRODUCT_SLUG` (required)
- `--force-regenerate` (optional, default false) — wipe and rebuild the entire file (loses user-authored content)
</step>

<step name="precondition_checks">
Verify `products/$PRODUCT_SLUG/runway/` exists. If not, abort and suggest `pom-spinup-product`.
</step>

<step name="detect_existing_plan">
Check if `products/$PRODUCT_SLUG/runway/runway-plan.md` exists.
- If NO → set `$MODE = "create"`.
- If YES, AND `--force-regenerate` set → set `$MODE = "regenerate"` (warn user that user-authored content will be lost; confirm via `AskUserQuestion`).
- If YES, no `--force-regenerate` → set `$MODE = "refresh"`.
</step>

<step name="scan_runway_artifacts">
Glob `products/$PRODUCT_SLUG/runway/`:
- ADR files matching `ADR-NNNN-*.md` — extract: number, title (from filename), status (read from each file's frontmatter table)
- PATTERN files matching `PATTERN-*.md` — extract: name, status, pods using it
- (Optional) Model Cards under `runway/model-cards/MC-*.md` — for reference
</step>

<step name="if_create_or_regenerate">
Generate the file from `the pom-core plugin's methodology/templates/products/_template/runway/runway-plan-template.md`.

Fill:
- `{{product-name}}` → product slug
- Owner = TL (read from `products/$PRODUCT_SLUG/trio.md`)
- Last updated = today
- Last Trio review = blank (user fills at next Trio sync)
- **Recent decisions table**: one row per ADR found
- **Active patterns table**: one row per PATTERN found

Use `AskUserQuestion` to gather:
- Current commitments (at least 1, or "none yet")
- Upcoming runway investments (at least 1, or "none currently")
- Capacity assumptions (at least 1)
- Open questions (at least 1, or "none currently")
- Retired (likely empty for new products)

Write to `products/$PRODUCT_SLUG/runway/runway-plan.md`.
</step>

<step name="if_refresh">
Read the existing `runway-plan.md`. Identify three sections to update vs. preserve:

**Update (auto-regenerate):**
- Last updated → today
- Recent decisions table → re-scan ADRs
- Active patterns table → re-scan PATTERNs

**Preserve (do NOT regenerate):**
- Current commitments
- Upcoming runway investments
- Capacity assumptions
- Open questions
- Retired

Edit the file in place, surgical updates only.

Check for **orphans**: ADRs/patterns referenced in the file but no longer present in `runway/`. List them at the bottom of the report as warnings (don't auto-remove from the file — let the user decide).
</step>

<step name="prompt_for_section_updates">
After the file is regenerated/refreshed, use `AskUserQuestion` to ask whether the user wants to update any of the preserved sections (commitments, investments, assumptions, open questions). For each "yes", prompt for the new content and update only that section.
</step>

<step name="report">
```
✅ Runway plan {{created / refreshed}} for $PRODUCT_SLUG.

File: products/$PRODUCT_SLUG/runway/runway-plan.md

Current state:
  ADRs: {{count}} ({{list of titles}})
  Patterns: {{count}} ({{list of names}})
  Last Trio review: {{date or "never — set at next Trio sync"}}

{{If refresh and orphans detected:}}
⚠️ Orphan references detected (artifacts referenced in plan but missing from runway/):
- {{ADR-NNNN or PATTERN-name}}
Investigate and remove the references manually if no longer applicable.

Next steps:
1. Review the runway plan at the next Trio sync.
2. Add new ADRs via `/pom-runway-add-adr $PRODUCT_SLUG <slug>`.
3. Re-run this skill periodically (recommend monthly or after any ADR addition).
```
</step>

</process>

<guardrails>
- **NEVER wipe user-authored sections** (commitments, investments, assumptions, open questions) on refresh unless `--force-regenerate` is set AND confirmed.
- ALWAYS re-scan the runway/ folder to keep the auto-generated tables current.
- DO NOT auto-remove orphan references (artifacts removed from disk but still in the plan) — warn the user instead.
- ALWAYS update the "Last updated" date.
</guardrails>

<success_criteria>
- [ ] `runway-plan.md` exists after this run
- [ ] Recent decisions and Active patterns tables reflect current `runway/` contents
- [ ] User-authored content preserved on refresh
- [ ] Orphans surfaced as warnings (not silently removed)
- [ ] `pom-validate` passes (R4.3 — runway-plan references existing ADRs)
</success_criteria>
