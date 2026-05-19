# Test: `pom-decision-log`

## Scenario A: append a decision to enabling/ai-ethics/decision-log/

**Setup:** A POM repo with `enabling/ai-ethics/` seeded (and its `decision-log/` folder exists).

**Invocation:** `/pom-decision-log --target=enabling/ai-ethics --slug=accept-cohort-gap-uc0002`

**Expected outcome:**
- Skill uses `AskUserQuestion` to collect:
  - Deciders (names + roles)
  - Context (what was being decided)
  - Decision (what was decided, exactly)
  - Rationale (why; alternatives considered)
  - Precedent scope (UC-only / product-only / portfolio-wide)
  - Expiration or re-review condition
- Writes `enabling/ai-ethics/decision-log/DEC-YYYY-MM-DD-accept-cohort-gap-uc0002.md`.
- Does NOT modify any prior decision files (append-only).
- Reports: file path; reminds the user that this decision is now precedent for future UCs scoring the same concern.
- **Paste-ready precedent summary** in the CLI report, framed by dashed lines, that states decision + concern + scope + re-review trigger.

**Pressure points:**
- If skill silently overwrites an existing decision file → CATASTROPHIC FAIL (append-only rule R6.4)
- If skill skips any of the required fields → FAIL (R6.3)
- If skill omits the paste-ready precedent summary → FAIL (v0.2 requirement)

---

## Scenario B: filename collision (same date, same slug)

**Setup:** A decision file with today's date and the slug `accept-cohort-gap-uc0002` already exists.

**Invocation:** `/pom-decision-log --target=enabling/ai-ethics --slug=accept-cohort-gap-uc0002`

**Expected outcome:** Skill detects the collision. Adds a disambiguator (`-2`, `-3`, etc.) to the filename. Does NOT overwrite the original.

---

## Scenario C: target folder doesn't exist

**Invocation:** `/pom-decision-log --target=enabling/accessibility`

**Expected outcome:** Skill detects missing `enabling/accessibility/` folder. Suggests `pom-seed-enabling-standard --concern=accessibility` first. Does not modify anything.

---

## Scenario D: target is a non-decision-log path

**Setup:** A POM repo.

**Invocation:** `/pom-decision-log --target=products/claims/runway`

**Expected outcome:** Skill refuses (runway is for ADRs, not generic decisions). Suggests `pom-runway-add-adr` instead. The valid targets are: `enabling/<concern>` paths.

---

## Reference comparison

The decision file format must follow `enabling/<concern>/decision-log/README.md` (which itself comes from `the pom-core plugin's methodology/templates/enabling/ai-ethics/decision-log/README.md`).
