# Test: `pom-runway-plan`

## Scenario A: create the first runway plan

**Setup:** A product `products/claims/` with some ADRs and patterns in `runway/`, but no `runway-plan.md` yet.

**Invocation:** `/pom-runway-plan claims`

**Expected outcome:**
- Skill scans `products/claims/runway/` for ADRs and patterns.
- Generates `runway-plan.md` from template, populated with:
  - Owner = TL (read from trio.md)
  - Last updated = today
  - **Recent decisions** table: all ADRs with title, date, status
  - **Active patterns** table: all PATTERN files
  - **Current commitments / Upcoming investments / Capacity assumptions / Open questions**: prompts user via `AskUserQuestion` for initial content
- Reports: file path; reminds the user this is a *living* document — review at every Trio sync.

---

## Scenario B: refresh an existing runway plan

**Setup:** `runway-plan.md` exists with stale content. 2 new ADRs were added since last refresh.

**Invocation:** `/pom-runway-plan claims`

**Expected outcome:**
- Skill detects existing runway-plan.md.
- Re-scans the runway/ folder for ADRs and patterns.
- Updates the "Recent decisions" and "Active patterns" tables to reflect current state (adds new entries; removes references to deleted artifacts).
- **Preserves** existing user-authored content in "Current commitments", "Upcoming investments", "Capacity assumptions", "Open questions" unless the user explicitly asks to update those sections via `AskUserQuestion`.
- Updates "Last updated" to today.

**Pressure points:**
- If skill wipes user-authored content and regenerates from scratch → FAIL (must preserve hand-written sections)
- If skill skips orphan-detection (ADRs that no longer exist but are still referenced) → WARN-level finding

---

## Scenario C: product has no runway artifacts

**Setup:** `products/claims/runway/` has only the README; no ADRs or patterns.

**Invocation:** `/pom-runway-plan claims`

**Expected outcome:** Skill creates a minimal runway-plan.md. "Recent decisions" and "Active patterns" tables are empty. Prompts user for at least one open question / commitment so the file isn't entirely placeholder.

---

## Scenario D: product doesn't exist

**Invocation:** `/pom-runway-plan nonexistent`

**Expected outcome:** Skill suggests `pom-spinup-product` first. Does not modify anything.
