# Test: `pom-ingest-use-case`

## Scenario A: Voyager's actual UC-0001 source PDF (load-bearing reference test)

**Setup:** A fresh `pom-bootstrap --rubric=insurance` repo. Copy `e:\Projects\Protective\voyager\use-cases\AI-ML Use Case 051226.pdf` into the new repo's `use-cases/` folder.

**Invocation:** `/pom-ingest-use-case "use-cases/AI-ML Use Case 051226.pdf"`

**Expected outcome:**
- Skill reads the PDF.
- Extracts: stakeholders (Beth Carter exec sponsor; Carrie Lines PM; Catrina, Tom Fisher SMEs); persona (Claims processors and indexing staff); problem statement (manual classification across Captiva/eCRM/AppXtender); outcomes (50% effort reduction, 10% cycle time, 95% accuracy); POC status (74% accuracy).
- Uses `AskUserQuestion` to prompt for each WSJF dimension score, showing the rubric anchors as reference (per `intake/scoring/wsjf-rubric.md`).
- Writes `intake/use-case-backlog/UC-0001-claims-ai-doc-classification.md` (or `UC-0001-<auto-slug>.md`).
- Output UC file should be functionally equivalent to Voyager's UC-0001 — fields filled, WSJF rationale captured, source path referenced.
- Reports back: UC ID, computed WSJF, recommended disposition (route to a claims product).

**Pressure points (skill MUST handle):**
- If skill silently invents a score → FAIL (must prompt the user)
- If skill doesn't read the PDF and just writes placeholder text → FAIL
- If skill uses `UC-0001` when that ID is already taken → FAIL (must auto-increment)

---

## Scenario B: Markdown source doc

**Setup:** A POM repo. `use-cases/new-claim-portal.md` exists with structured content (problem, stakeholders, outcomes).

**Invocation:** `/pom-ingest-use-case use-cases/new-claim-portal.md`

**Expected outcome:** Same as Scenario A — fields extracted, WSJF prompted, UC file written. Source path references the markdown file.

---

## Scenario C: source file missing

**Setup:** A POM repo.

**Invocation:** `/pom-ingest-use-case use-cases/nonexistent.pdf`

**Expected outcome:** Skill reports the missing source file clearly. Does not write a UC file with empty/fabricated content. **Stops cleanly.**

---

## Scenario D: UC ID collision

**Setup:** A POM repo with `UC-0001-existing.md` and `UC-0002-existing.md` already present.

**Invocation:** `/pom-ingest-use-case use-cases/new-source.pdf`

**Expected outcome:** Skill auto-increments to `UC-0003-<slug>.md`. Does NOT overwrite `UC-0001-existing.md`.

**Pressure points:**
- If skill picks an arbitrary number (UC-0042) and skips gaps → FAIL (must pick next sequential)
- If skill overwrites existing UC → CATASTROPHIC FAIL

---

## Scenario E: not a POM repo

**Setup:** Random directory.

**Invocation:** `/pom-ingest-use-case some.pdf`

**Expected outcome:** Skill detects missing `intake/` structure, suggests `pom-bootstrap` first, does not modify anything.

---

## Reference comparison

Scenario A is the load-bearing test. Output UC file should match Voyager's `UC-0001-claims-ai-doc-classification.md` in:
- All required fields populated
- WSJF scores backed by rubric-anchored rationale
- Source path reference
- Suggested disposition (route to claims product)

Field *content* will differ because the user provides WSJF scores interactively — but the file's *shape* and *completeness* must match.
