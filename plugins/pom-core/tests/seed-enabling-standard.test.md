# Test: `pom-seed-enabling-standard`

## Scenario A: seed ai-ethics in an existing POM repo

**Setup:** A POM repo. `enabling/` exists with `README.md` but no concerns.

**Invocation:** `/pom-seed-enabling-standard --concern=ai-ethics`

**Expected outcome:**
- `enabling/ai-ethics/` created with:
  - `README.md` (matches `the pom-core plugin's methodology/templates/enabling/ai-ethics/README.md`)
  - `model-card-template.md`
  - `pii-handling.md`
  - `bias-audit-framework.md`
  - `decision-log/README.md`
- `pom-validate` still passes — R6.2 (each concern has README + decision-log) is satisfied

**Pressure points:**
- If skill silently overwrites an existing concern → FAIL
- If `decision-log/` is missing → FAIL (R6.2)

---

## Scenario B: concern already exists

**Setup:** `enabling/ai-ethics/` already populated.

**Invocation:** `/pom-seed-enabling-standard --concern=ai-ethics`

**Expected outcome:** Skill refuses to proceed. Reports the conflict. Optionally offers a `--force-update` flag with explicit confirmation.

---

## Scenario C: concern with no template (e.g., `--concern=accessibility`)

**Setup:** A POM repo. Phase 1 has only `ai-ethics/` templates in `the pom-core plugin's methodology/templates/enabling/`.

**Invocation:** `/pom-seed-enabling-standard --concern=accessibility`

**Expected outcome:** Skill scaffolds a *generic* concern structure:
- `enabling/accessibility/README.md` with a "v0.1 — scaffold; populate as standards mature" header and pointers to relevant authoritative references
- `enabling/accessibility/decision-log/README.md` (the canonical decision-log format)

Skill prompts the user via `AskUserQuestion` whether to generate stub standards files (e.g., empty `checklist.md`) or leave the concern empty for the team to populate.

**Pressure points:**
- If skill refuses to seed unknown concerns → FAIL (this skill is meant to enable new concerns)
- If skill invents detailed concern-specific content it has no basis for → FAIL (over-claiming)

---

## Scenario D: not a POM repo

**Setup:** Random directory.

**Invocation:** `/pom-seed-enabling-standard --concern=ai-ethics`

**Expected outcome:** Skill detects missing structure, suggests `pom-bootstrap` first, does not modify.

---

## Reference comparison

Scenario A's output should match `e:\Projects\Protective\voyager\enabling\ai-ethics\` exactly (the structure we hand-built for Voyager).
