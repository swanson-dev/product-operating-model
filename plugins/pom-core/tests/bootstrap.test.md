# Test: `pom-bootstrap`

Pressure scenarios that `pom-bootstrap` must handle correctly.

---

## Scenario A: empty directory + insurance preset

**Setup:** Create empty directory at `/tmp/pom-test-bootstrap-A/`.

**Invocation:** `/pom-bootstrap /tmp/pom-test-bootstrap-A --rubric=insurance`

**Expected outcome:**
- Top-level dirs created: `intake/`, `products/`, `platform/`, `enabling/`, `methodology/`, `use-cases/`
- Top-level `README.md` created (content matches `the pom-core plugin's methodology/templates/README.md`)
- `intake/scoring/wsjf-rubric.md` byte-equivalent to `the pom-core plugin's methodology/rubrics/insurance.md`
- `intake/use-case-backlog/README.md` and `UC-template.md` exist
- `intake/dispositions/README.md` and `DISP-template.md` exist
- `products/_template/` exists with all subfolders and template files
- `platform/README.md` and `_service-template/` exist
- `enabling/README.md` exists (but no concerns seeded yet)
- `methodology/` exists but empty (with a README pointing to the methodology docs)
- `use-cases/` exists but empty
- `pom-validate /tmp/pom-test-bootstrap-A` reports zero ERROR-level violations

**Pressure points (skill MUST handle):**
- If skill silently overwrites a non-empty directory → FAIL
- If `--rubric=insurance` doesn't actually copy the insurance rubric (uses scaffold instead) → FAIL
- If `intake/` is missing required files (R2.1, R2.2, R2.3, R2.4) → FAIL

---

## Scenario B: empty directory + interactive calibration

**Setup:** Empty directory at `/tmp/pom-test-bootstrap-B/`.

**Invocation:** `/pom-bootstrap /tmp/pom-test-bootstrap-B --rubric=interactive`

**Expected outcome:**
- Skill uses `AskUserQuestion` to prompt for 8 anchor examples (2 per dimension × 4 dimensions).
- After user provides anchors, `intake/scoring/wsjf-rubric.md` is generated:
  - "1" row contains user's smallest example
  - "13" row contains user's largest example
  - 2/3/5/8/21 rows have *interpolated* placeholder text (not left blank or TBD)
- Otherwise identical to Scenario A's filesystem outcome.

**Pressure points:**
- Skill does not prompt → FAIL
- Skill prompts but ignores answers → FAIL
- Skill leaves 2/3/5/8/21 rows blank or TBD → FAIL (R8.2 violation)

---

## Scenario C: non-empty target directory

**Setup:** Directory `/tmp/non-empty/` containing at least one file.

**Invocation:** `/pom-bootstrap /tmp/non-empty`

**Expected outcome:** Skill refuses to proceed. Reports the conflict in plain text. **Does not modify any files.**

**Pressure points:**
- If skill blindly overwrites files → CATASTROPHIC FAIL
- If skill fails silently → FAIL (must report clearly)

---

## Scenario D: no `--rubric` flag

**Setup:** Empty directory.

**Invocation:** `/pom-bootstrap /tmp/pom-test-bootstrap-D`

**Expected outcome:** Skill uses `AskUserQuestion` to prompt the user to choose between interactive and a preset industry. Otherwise behaves per the user's choice.

**Pressure points:**
- If skill silently defaults to scaffold → FAIL (user gets a TBD rubric without being asked)
- If skill silently defaults to one industry → FAIL (assumes wrong industry)

---

## Scenario E: no enabling concerns are auto-seeded

**Setup:** Empty directory at `/tmp/pom-test-bootstrap-E/`.

**Invocation:** `/pom-bootstrap /tmp/pom-test-bootstrap-E --rubric=generic`

**Expected outcome:** After scaffold, `enabling/` contains **exactly one entry**: `README.md`. No subdirectories, no concern-specific files.

Concretely, the directory listing of `/tmp/pom-test-bootstrap-E/enabling/` is:

```
README.md
```

The following MUST NOT exist after bootstrap:
- `enabling/ai-ethics/` (or any other concern directory)
- `enabling/ai-ethics/README.md`
- `enabling/ai-ethics/model-card-template.md`
- `enabling/ai-ethics/pii-handling.md`
- `enabling/ai-ethics/bias-audit-framework.md`
- `enabling/ai-ethics/decision-log/`

**Pressure points:**
- If skill `cp -r`s the entire `methodology/templates/enabling/` tree (or any analog) into the target, a partial `ai-ethics/` scaffold leaks into a portfolio that never opted into AI ethics standards → FAIL. This violates the explicit guardrail at [workflows/bootstrap.md](../methodology/workflows/bootstrap.md): "Do NOT seed any enabling concerns automatically (not even ai-ethics) — that's `pom-seed-enabling-standard`'s job."
- If any subdirectory appears under `enabling/` post-bootstrap → FAIL. The only sanctioned way for `enabling/<concern>/` to come into existence is via `/pom-seed-enabling-standard --concern=<slug>`, where the user explicitly opts in and picks a calibration version.

**Why this scenario exists:** v0.4 moved `methodology/templates/enabling/ai-ethics/` to `methodology/templates/enabling-standards/ai-ethics/` precisely so a naive `cp -r templates/enabling/` cannot leak concern scaffolds. This scenario locks that contract.

---

## Reference comparison

Scenario A's output should diff cleanly against `e:\Projects\Protective\voyager\`. Only differences should be:
- Voyager-specific content under `use-cases/` (the PDF) and `methodology/` (the diagrams) — these are repo-specific, not template-specific
- Voyager has UC-0001 already scored; a fresh bootstrap has no UCs
- Voyager has `products/claims/` populated; a fresh bootstrap has only `products/_template/`
- Voyager has `enabling/ai-ethics/` seeded; a fresh bootstrap has only `enabling/README.md`

All *structural* files should match.
