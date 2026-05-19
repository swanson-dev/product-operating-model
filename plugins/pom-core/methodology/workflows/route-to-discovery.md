<purpose>
Land a routed Use Case in its receiving product's Discovery Backlog as a **reference** (not a copy):
- Verify the UC was routed and the receiving product exists
- Create a `DISC-YYYY-MM-<slug>.md` file referencing the UC
- Update the UC's `Discovery entry` field for cross-reference integrity

This is the hand-off step from intake (portfolio-shared) to the receiving product (single-product).
</purpose>

<process>

<step name="parse_args">
Parse:
- First positional: `$UC_ID` (required)
- Second positional: `$PRODUCT_SLUG` (required) — receiving product

If either missing, ask via `AskUserQuestion`.
</step>

<step name="precondition_checks">
Verify cwd is a POM repo.

Locate the UC file: glob `intake/use-case-backlog/$UC_ID-*.md`. If no match, abort.

Read the UC file. Verify:
- `Status` field shows `routed → $PRODUCT_SLUG`. If not, abort with:
  ```
  UC-NNNN's disposition is not `route → $PRODUCT_SLUG`.
  Current status: {{actual status}}

  Run `/pom-disposition $UC_ID --decision=route --product=$PRODUCT_SLUG` first.
  ```
- `Disposition` field links to an existing DISP file.

Verify `products/$PRODUCT_SLUG/` exists. If not, abort:
```
Receiving product products/$PRODUCT_SLUG/ does not exist.

Run `/pom-spinup-product $PRODUCT_SLUG --originating-uc=$UC_ID` first.
```

Verify `products/$PRODUCT_SLUG/discovery-backlog/` exists. If not, this is a structural issue with the product — abort with the path it's missing.
</step>

<step name="check_duplicate">
Glob `products/$PRODUCT_SLUG/discovery-backlog/DISC-*.md`. For each file, read it and check if its `Originating UC` field references `$UC_ID`.

If a DISC already exists for this UC:
```
A Discovery item already exists for $UC_ID:

products/$PRODUCT_SLUG/discovery-backlog/DISC-{{date}}-{{slug}}.md

To update its shaping state, run: /pom-discovery-gate {{DISC-ID}}
To create a *new* Discovery item for the same UC (rare; usually a re-shape after parking), confirm explicitly.
```

If user confirms, proceed with a new DISC file. Otherwise stop.
</step>

<step name="compute_disc_filename">
Extract the UC slug from the UC filename (everything after `UC-NNNN-`).

DISC slug = `UC<NNNN>-<uc-slug>` (e.g., `UC0001-claims-ai-doc-classification`).

$DISC_MONTH = current YYYY-MM.

Filename: `DISC-$DISC_MONTH-$DISC_SLUG.md`.
</step>

<step name="generate_disc_content">
Read the originating UC file to extract:
- Title (for the DISC file header)
- Disposition file path (for the routing reference)

Read `the pom-core plugin's methodology/templates/products/_template/discovery-backlog/DISC-template.md` as the base.

Replace placeholders:
- `{{title}}` → UC title
- `{{slug}}` → DISC slug
- `YYYY-MM` → current month
- Originating UC link → relative path from DISC location back to the UC file (typically `../../../intake/use-case-backlog/UC-NNNN-<uc-slug>.md`)
- Routing disposition link → relative path to DISP file
- `Routed to Discovery` date → today
- `Shaping status` → "In progress — 0 / 4 questions fully answered"
- All four question status markers → ❌ Not yet answered
- Promotion readiness table → all ❌
- Critical-path unblock sequence → initial sketch (depends on Trio state — see next step)

**Do NOT copy UC content into the DISC file.** This file is a reference, not a duplicate. Source-of-truth for problem/outcomes/scoring is the UC file.
</step>

<step name="reflect_trio_state">
Read `products/$PRODUCT_SLUG/trio.md`. For each role (PM, PD, TL), check if assigned or `_TBD_` / `Unassigned`.

If any roles are unassigned, the critical-path sequence in the DISC file should start with "Trio formation" before the four-question gate work. Update the critical-path text accordingly.

If Trio is fully assigned, critical-path starts with question-by-question shaping.
</step>

<step name="write_disc_file">
Write to `products/$PRODUCT_SLUG/discovery-backlog/$DISC_FILENAME`.

Initial shaping log entry (today's date):
```
- **YYYY-MM-DD** — Discovery file created from routing disposition {{DISP-file-name}}. No shaping work performed yet. Trio state: {{full or "incomplete: missing X, Y"}}.
```
</step>

<step name="update_uc_file">
Edit the UC file. Update the `Discovery entry` field:
```
| **Discovery entry** | [`../../products/{{product-slug}}/discovery-backlog/$DISC_FILENAME`](../../products/{{product-slug}}/discovery-backlog/$DISC_FILENAME) |
```
</step>

<step name="update_product_index">
Read `products/README.md`. In the portfolio index table, find the row for `$PRODUCT_SLUG` and update its "Discovery items" column (increment count, or set to 1 if this is the first). If the count is hard to track in the table cell, just write the running count.
</step>

<step name="report">
```
✅ Discovery item created for $UC_ID in product $PRODUCT_SLUG.

Files:
- products/$PRODUCT_SLUG/discovery-backlog/$DISC_FILENAME (new)
- intake/use-case-backlog/$UC_ID-*.md (Discovery entry field updated)
- products/README.md (portfolio index updated)

Next steps:
1. Confirm Trio assignments if any roles are unassigned in products/$PRODUCT_SLUG/trio.md.
2. Run `/pom-discovery-gate {{DISC-ID}}` to walk through the four-question gate.
3. Address any Trio-Note gaps flagged in the originating UC.
```
</step>

</process>

<guardrails>
- **DISC must be a REFERENCE, not a copy of the UC.** Do not duplicate problem statements, outcomes, or scoring into the DISC file. Reference the UC instead.
- NEVER create a DISC in a non-existent product folder.
- NEVER route a UC that wasn't dispositioned as `route` to this product.
- ALWAYS update the UC's `Discovery entry` field for R7.2 / R7.3 cross-reference integrity.
- Detect duplicate DISC files for the same UC and prompt for explicit confirmation before creating a second one.
</guardrails>

<success_criteria>
- [ ] DISC file created in the correct product folder
- [ ] DISC references the originating UC and disposition (R7.3)
- [ ] UC file's `Discovery entry` field updated (R7.2)
- [ ] Initial shaping log entry written
- [ ] Portfolio index in `products/README.md` updated
- [ ] `pom-validate` continues to pass
</success_criteria>
