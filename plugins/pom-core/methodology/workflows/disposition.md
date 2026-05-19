<purpose>
Apply a formal disposition (kill / park / route / split) to a scored Use Case:
- Write the DISP file with all required fields per `disposition-rules.md`
- Generate a paste-ready stakeholder-facing summary
- Update the UC's Status and Disposition link
- Handle decision reversals via supersession (NOT editing)

Append-only: never edits past disposition files.
</purpose>

<process>

<step name="parse_args">
Parse `$ARGUMENTS`:
- First positional: `$UC_ID` (required) — e.g., `UC-0001`
- `--decision=<value>` (required) — one of: `kill`, `park`, `route`, `split`
- `--product=<slug>` (required if decision = route) — receiving product slug
- `--rationale=<text>` (optional) — short rationale; prompted if missing
- `--trigger=<text>` (optional, for park) — re-score trigger
- `--new-ucs=<UC-NNNN,UC-NNNN,…>` (optional, for split)

If `$UC_ID` missing, ask.
If `--decision` missing, use `AskUserQuestion` with the four options.
</step>

<step name="validate_decision">
`--decision` must be one of: `kill`, `park`, `route`, `split`. If not, abort:
```
Invalid decision: '{{value}}'

Valid: kill, park, route, split
```
</step>

<step name="precondition_checks">
Verify cwd is a POM repo (`intake/use-case-backlog/` and `intake/dispositions/` exist).

Locate the UC file: glob `intake/use-case-backlog/$UC_ID-*.md`. If no match, abort:
```
UC file for $UC_ID not found.

Run `/pom-ingest-use-case` first to create it.
```

Read the UC file. Check the `Status` field.

If `Status` is already `dispositioned` or shows a disposition link:
- Use `AskUserQuestion`: "UC-NNNN already has a disposition (link to existing). Are you reversing the prior decision (NEW DISP that supersedes), or did you not realize? (Reversal / Not aware — stop)"
- If "Not aware": abort cleanly.
- If "Reversal": continue, but the new DISP must include a `Supersedes: DISP-YYYY-MM-DD-UC-NNNN.md` field.

If decision is `route`, verify `--product` was provided. If not, prompt for it.

If decision is `park`, verify `--trigger` was provided. If not, prompt for the re-score trigger via `AskUserQuestion`.

If decision is `split`, verify `--new-ucs` was provided. If not, prompt for the list via `AskUserQuestion`. Validate format `UC-NNNN,UC-NNNN,…`.
</step>

<step name="gather_required_fields">
Use `AskUserQuestion` to collect any missing inputs:

- **Deciders** (always required): who was in the room? Format: name + role (e.g., "Beth Carter (Exec Sponsor), Carrie Lines (proposed PM)")
- **Rationale**: short paragraph explaining why this decision
- If route:
  - Trio assignments if known: PM, PD, TL
- If park:
  - Re-score trigger (date, event, dependency)
- If split:
  - For each new UC ID, a brief rationale for the split shape

Generate a draft **stakeholder-facing summary** based on inputs (one paragraph the stakeholder can read directly). Show to user via chat and use `AskUserQuestion` to confirm or refine.
</step>

<step name="compute_disposition_filename">
$DISP_DATE = today (ISO 8601: YYYY-MM-DD).
$DISP_FILENAME = `DISP-$DISP_DATE-$UC_ID.md`.

If a file already exists with this exact name (rare — same UC dispositioned twice on the same day):
- Add a suffix `-r2`, `-r3`, etc. to disambiguate. This handles the "reversal on the same day" edge case.
</step>

<step name="write_disp_file">
Use `the pom-core plugin's methodology/templates/intake/dispositions/DISP-template.md` as the base. Fill in:
- Disposition ID matching filename
- UC reference (relative path to UC file)
- Decision date
- Deciders
- Decision = `$DECISION`
- Rationale
- Next action filled per decision type:
  - **kill**: empty (UC status set to killed)
  - **park**: trigger condition
  - **route**: receiving product slug + Trio members + path where the Discovery file will be created
  - **split**: list of new UC IDs + per-UC split rationale
- Stakeholder-facing summary (paste-ready)
- If supersession: `Supersedes: DISP-YYYY-MM-DD-UC-NNNN.md` reference field

Write to `intake/dispositions/$DISP_FILENAME`.
</step>

<step name="update_uc_file">
Edit the UC file:
- `Status` field updated to reflect the disposition:
  - kill → `killed`
  - park → `parked (re-score trigger: {{trigger}})`
  - route → `routed → {{product-slug}}`
  - split → `killed-by-split (replaced by {{new-uc-ids}})`
- `Disposition` field updated to link to `../dispositions/$DISP_FILENAME`
- If route, leave `Discovery entry` field as a placeholder ("(to be filled by pom-route-to-discovery)")

For supersession, preserve the old disposition link in a "Disposition history" appended row, alongside the new one.
</step>

<step name="report">
```
✅ Disposition recorded: $DECISION

Files written:
- intake/dispositions/$DISP_FILENAME
- intake/use-case-backlog/$UC_ID-*.md (Status + Disposition fields updated)

Stakeholder summary (paste-ready):
─────────────────────────────────
{{stakeholder-facing summary}}
─────────────────────────────────

Next steps:
{{If kill:}}
- No further action. UC is closed.

{{If park:}}
- Re-score when: {{trigger}}
- Re-invoke /pom-ingest-use-case or /pom-disposition when the trigger fires.

{{If route:}}
- If products/{{product}}/ doesn't yet exist: run `/pom-spinup-product {{product}} --originating-uc=$UC_ID`
- Otherwise: `/pom-route-to-discovery $UC_ID {{product}}`

{{If split:}}
- For each new UC ID: `/pom-ingest-use-case <source-fragment>` (with --next-uc=<id>)
- Original $UC_ID is now closed (killed-by-split).
```
</step>

</process>

<guardrails>
- **NEVER edit or delete an existing DISP file.** Append-only is the load-bearing rule.
- **NEVER write a disposition without a stakeholder-facing summary.** It's a hard requirement.
- For decision = route, the receiving product MUST be named. No "route to TBD".
- For decision = park, the re-score trigger MUST be specified. No silent parking.
- For decision = split, the new UC IDs MUST be named. The original UC is closed (killed-by-split), and the new UCs are real items to be ingested.
- If the user attempts a reversal, force a NEW DISP that explicitly supersedes — never edit the old one.
</guardrails>

<success_criteria>
- [ ] DISP file written with all required fields per `disposition-rules.md`
- [ ] UC file Status + Disposition fields updated
- [ ] Stakeholder-facing summary is paste-ready
- [ ] Cross-references (UC ↔ DISP) resolve in `pom-validate` (R7.1)
- [ ] Append-only rule respected for any reversals
</success_criteria>
