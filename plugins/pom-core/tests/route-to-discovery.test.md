# Test: `pom-route-to-discovery`

## Scenario A: UC routed + product exists (happy path)

**Setup:**
- A POM repo with UC-0001 dispositioned as `route → claims`.
- `products/claims/` exists from `pom-spinup-product`.

**Invocation:** `/pom-route-to-discovery UC-0001 claims`

**Expected outcome:**
- Skill writes `products/claims/discovery-backlog/DISC-YYYY-MM-<slug>.md`:
  - **Reference** to the originating UC (relative path), NOT a copy of the UC content
  - Routing disposition reference
  - 4-question gate scaffold (each question starts as ❌ Not yet answered)
  - Promotion-readiness table at the bottom showing 0/4
  - Shaping log with one entry: "Created from routing disposition"
- Skill updates UC-0001's `Discovery entry` field to point to the new DISC file.
- Reports: DISC file path; suggests next step `pom-discovery-gate <DISC-ID>` for shaping.

**Pressure points:**
- If skill **copies** the UC content into the DISC file → FAIL (DISC must be a reference, not a duplicate — the methodology explicitly forbids this)
- If skill doesn't update UC's Discovery entry field → FAIL (cross-reference R7.2 broken)

---

## Scenario B: UC routed + product does NOT exist

**Setup:** UC-0001 dispositioned as `route → claims`, but `products/claims/` does not exist.

**Invocation:** `/pom-route-to-discovery UC-0001 claims`

**Expected outcome:** Skill detects the missing product. Reports the issue and suggests `pom-spinup-product claims --originating-uc=UC-0001` first. Does NOT create the DISC file in a non-existent product folder.

---

## Scenario C: UC not yet dispositioned

**Setup:** UC-0001 exists but has no disposition.

**Invocation:** `/pom-route-to-discovery UC-0001 claims`

**Expected outcome:** Skill detects UC is undispositioned. Reports the issue and suggests `pom-disposition UC-0001 --decision=route --product=claims` first. Does not modify anything.

---

## Scenario D: UC was killed/parked/split

**Setup:** UC-0002 was killed.

**Invocation:** `/pom-route-to-discovery UC-0002 claims`

**Expected outcome:** Skill detects the non-route disposition. Reports the conflict ("UC-0002 was killed, cannot route") and stops.

---

## Scenario E: DISC already exists for this UC

**Setup:** A DISC file referencing UC-0001 already exists in `products/claims/discovery-backlog/`.

**Invocation:** `/pom-route-to-discovery UC-0001 claims`

**Expected outcome:** Skill detects the duplicate. Reports the existing DISC file. Does not create another one. Optionally suggests `pom-discovery-gate` to update the existing one.

---

## Reference comparison

Scenario A's DISC file output should match the *structure* of Voyager's `DISC-2026-05-UC0001-claims-ai-doc-classification.md` — same sections (4-question gate, Trio-Note gaps, promotion readiness, shaping log), but content starts in "❌ not yet answered" state (Voyager's was partially populated through later iterations).
