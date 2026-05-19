<purpose>
Initialize a new Product Operating Model (POM) repository: scaffold the filesystem from canonical templates, set up the WSJF rubric (interactive or industry preset), create the methodology pointer.

For a fresh portfolio adopting the methodology. After completion, run `pom-validate` to confirm structure and `pom-spinup-product <first-product>` to begin populating.
</purpose>

<process>

<step name="parse_args">
Parse `$ARGUMENTS`:
- First positional arg: `$TARGET_DIR` (required) — path to the directory to bootstrap. Resolve to absolute path.
- `--rubric=<value>` flag → `$RUBRIC_MODE` (optional) — one of: `interactive`, `insurance`, `fintech`, `healthcare`, `retail`, `generic`.

If `$TARGET_DIR` is missing, ask:
```
What's the target directory? (absolute path)
```

If `$RUBRIC_MODE` is missing, use `AskUserQuestion` to prompt with options:
- **Interactive** — calibrate WSJF anchors now via 8 questions
- **Generic** — industry-agnostic preset (good default)
- **Insurance** / **Fintech** / **Healthcare** / **Retail** — pre-calibrated preset
</step>

<step name="precondition_checks">
Verify `$TARGET_DIR`:
1. Parent directory exists.
2. `$TARGET_DIR` itself is either non-existent (will be created) OR exists and is empty (only `.git/` or a stub `README.md` allowed).

If `$TARGET_DIR` is non-empty:
```
Target directory $TARGET_DIR is not empty. Aborting.

Existing files would be at risk of conflict. Either:
- Use an empty directory
- Move existing files away first
- Run on a fresh git-init'd directory
```

**Stop. Do not modify anything.**
</step>

<step name="scaffold_filesystem">
Copy `the pom-core plugin's methodology/templates/` → `$TARGET_DIR/`:

- `README.md`
- `intake/README.md`
- `intake/scoring/wsjf-rubric.md` (scaffold — will be overwritten in next step if preset chosen)
- `intake/use-case-backlog/README.md`
- `intake/use-case-backlog/UC-template.md`
- `intake/dispositions/README.md`
- `intake/dispositions/DISP-template.md`
- `products/README.md`
- `products/_template/` (entire subtree)
- `platform/README.md`
- `platform/_service-template/` (entire subtree)
- `enabling/README.md`

Create empty directories with minimal stub READMEs:
- `methodology/README.md`: one paragraph pointing to `the pom-core plugin's methodology/references/` for the methodology rules. Note that diagrams or PDFs explaining the org-specific operating model belong in this folder.
- `use-cases/README.md`: one paragraph explaining this is where stakeholders' raw submissions (PDFs, decks) go; intake entries in `intake/use-case-backlog/` reference these by path.

**Do NOT** copy `templates/products/_template/discovery-backlog/DISC-template.md` to a target Discovery file — it's a template, copied into the target's `_template/` only.
</step>

<step name="set_up_rubric">
Based on `$RUBRIC_MODE`:

**If `interactive`:**

For each of the 4 dimensions, use `AskUserQuestion` to gather two anchors:
- "1" anchor — smallest plausible example from the user's domain
- "13" anchor — large but not catastrophic example

Dimensions (in order):
1. User-Business Value
2. Time Criticality
3. Risk Reduction & Opportunity Enablement
4. Job Size

After all 8 anchors are collected:
1. Start with `the pom-core plugin's methodology/templates/intake/scoring/wsjf-rubric.md` as the base
2. Fill in the user's "1" and "13" examples in those rows for each dimension
3. For 2/3/5/8/21 rows, write *interpolated* placeholder text — e.g., "Between [1-example] and [13-example] — refine after first scoring exercise"
4. Add a calibration header: `v0.1 — anchors provided by user on YYYY-MM-DD; recalibrate after 10 use cases`
5. Write the result to `$TARGET_DIR/intake/scoring/wsjf-rubric.md`

**If preset (`insurance`, `fintech`, `healthcare`, `retail`, `generic`):**

Copy `the pom-core plugin's methodology/rubrics/$RUBRIC_MODE.md` → `$TARGET_DIR/intake/scoring/wsjf-rubric.md`.
</step>

<step name="git_init_note">
Check if `$TARGET_DIR` is already a git repo (look for `.git/`).

**Do NOT auto-init git.** Leave that to the user. Note the state in the final report so the user can decide.
</step>

<step name="verify">
Run a lightweight version of `pom-validate` against `$TARGET_DIR` (or invoke `pom-validate` if available) and confirm R1 + R2 rules pass:
- All required top-level dirs exist
- `intake/` required files all present
- `products/_template/` exists

If any check fails, report which files are missing and abort. Do not claim success on partial scaffolding.
</step>

<step name="report">
Output:

```
✅ POM repo bootstrapped at $TARGET_DIR

Structure created:
- intake/ (use-case-backlog, dispositions, scoring/wsjf-rubric.md)
- products/ (template only — use pom-spinup-product to add)
- platform/ (services scaffolded as needed via pom-platform-add-service)
- enabling/ (concerns seeded via pom-seed-enabling-standard)
- methodology/ (README points to the pom-core plugin's methodology/references/)
- use-cases/ (raw stakeholder submissions go here)

Rubric: $RUBRIC_MODE
WSJF rubric: intake/scoring/wsjf-rubric.md

Next steps:
1. Run `pom-validate $TARGET_DIR` to confirm clean structure.
2. Stakeholders submit raw materials to use-cases/; ingest via `pom-ingest-use-case`.
3. As the first product emerges, run `pom-spinup-product <slug>`.
4. Seed enabling concerns (ai-ethics, security, etc.) via `pom-seed-enabling-standard --concern=<slug>`.

{{If $TARGET_DIR is NOT a git repo:}}
Optional: `git init` and commit the initial scaffold.
```
</step>

</process>

<guardrails>
- **NEVER overwrite a non-empty target directory.** If files exist, abort cleanly.
- **NEVER auto-init git.** Let the user decide.
- **NEVER skip the rubric step.** Every POM repo needs a rubric, even if it's just the generic preset.
- If the user picks `--rubric=interactive` but provides no anchors, do NOT silently fall back to scaffold — re-prompt or abort with explanation.
- Do NOT seed any enabling concerns automatically (not even ai-ethics) — that's `pom-seed-enabling-standard`'s job, and bootstraps should be minimal.
</guardrails>

<success_criteria>
- [ ] Target directory contains the full canonical structure
- [ ] `intake/scoring/wsjf-rubric.md` exists and has anchors filled (preset OR interactive)
- [ ] All required top-level dirs and intake/ files exist (R1.1–R1.2, R2.1–R2.4)
- [ ] `pom-validate` would pass with zero ERRORs (R8.2 may WARN if interactive interpolation incomplete)
- [ ] User has a clear next-step list
</success_criteria>
