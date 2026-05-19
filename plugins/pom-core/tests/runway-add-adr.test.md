# Test: `pom-runway-add-adr`

## Scenario A: first ADR for a product

**Setup:** A POM repo with `products/claims/` standing. `products/claims/runway/` exists but is empty (no ADRs yet).

**Invocation:** `/pom-runway-add-adr claims ml-serving-platform-choice`

**Expected outcome:**
- Skill computes next ADR number: ADR-0001 (since no prior ADRs).
- Skill prompts via `AskUserQuestion` for: context, decision drivers, considered options (≥ 2), chosen option, rationale, consequences (positive + negative + neutral).
- Writes `products/claims/runway/ADR-0001-ml-serving-platform-choice.md` with all sections filled per `ADR-template.md`.
- If `products/claims/runway/runway-plan.md` exists, updates the "Recent decisions" table to reference the new ADR.
- Reports: ADR file path; suggests `/pom-runway-plan claims` to refresh the runway plan if not already up to date.

**Pressure points:**
- If skill skips the "considered options" section → FAIL (ADR template requires ≥ 2 options)
- If skill writes ADR-0002 without ADR-0001 existing → FAIL (R4.2 contiguous numbering)

---

## Scenario B: subsequent ADR

**Setup:** `products/claims/runway/` has ADR-0001 through ADR-0003.

**Invocation:** `/pom-runway-add-adr claims drift-detection-cadence`

**Expected outcome:** Skill picks ADR-0004 (next sequential). Writes the file. Updates runway-plan.md if present.

---

## Scenario C: collision (same slug already exists)

**Setup:** `ADR-0004-drift-detection-cadence.md` already exists.

**Invocation:** `/pom-runway-add-adr claims drift-detection-cadence`

**Expected outcome:** Skill detects the duplicate slug (regardless of number). Refuses; asks user via `AskUserQuestion` to provide a different slug.

---

## Scenario D: product doesn't exist

**Invocation:** `/pom-runway-add-adr nonexistent ml-serving-choice`

**Expected outcome:** Skill suggests `pom-spinup-product nonexistent` first. Does not modify anything.

---

## Scenario E: superseding a prior ADR

**Setup:** ADR-0001 exists. User wants to record a new ADR that supersedes it.

**Invocation:** `/pom-runway-add-adr claims new-ml-serving-platform --supersedes=ADR-0001`

**Expected outcome:**
- New ADR (ADR-0004 or next available) written with `Supersedes: ADR-0001` field.
- The OLD ADR (ADR-0001) is edited only to update its Status field to `Superseded by ADR-0004` — its content is NOT changed.
- Runway-plan.md updated to show ADR-0001 status changed.

**Pressure points:**
- If skill deletes or rewrites the content of the superseded ADR → FAIL (ADRs are immutable in content)
