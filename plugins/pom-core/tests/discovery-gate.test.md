# Test: `pom-discovery-gate`

## Scenario A: fresh Discovery file, run all 4 questions

**Setup:** A POM repo with `products/claims/discovery-backlog/DISC-2026-05-UC0001-claims-ai-doc-classification.md` just created by `pom-route-to-discovery` (all 4 questions at ❌).

**Invocation:** `/pom-discovery-gate DISC-2026-05-UC0001-claims-ai-doc-classification`

**Expected outcome:**
- Skill loads the Discovery file and the Trio file from the receiving product.
- For each of the four questions (Desirable / Viable / Feasible / Ethical), the skill:
  1. States the question and its owner per `four-question-gate.md`
  2. Names the blocker if the Trio role for that question is unassigned (e.g., "Q1 (Desirable) cannot be closed: PD seat in trio.md is unassigned")
  3. Uses `AskUserQuestion` to ask: what's the current state? what evidence exists? what specific work is still needed?
  4. Updates the question's status marker (❌ / 🟡 / ✅) and writes detailed answer prose into the Discovery file
- Updates the Promotion Readiness table.
- Updates the critical-path unblock sequence with current state.
- Appends a new entry to the Shaping log dated today, summarizing what was decided.
- Reports: question-by-question summary + overall verdict (READY / NOT READY for promotion).
- **Paste-ready stakeholder summary** in the CLI report, framed by dashed lines, that states gate result + open questions + next blocker.

**Pressure points:**
- If skill silently marks a question ✅ without evidence → FAIL (must require evidence per `four-question-gate.md`)
- If skill rewrites the entire Discovery file from scratch (losing prior shaping log entries) → FAIL (append-only on the log)
- If skill marks Q1 ✅ even though PD seat is unassigned → FAIL (Q1 requires the PD as owner)
- If skill omits the paste-ready stakeholder summary → FAIL (v0.2 requirement)

---

## Scenario B: update a specific question

**Setup:** A Discovery file with Q1 ✅, Q2 🟡, Q3 ❌, Q4 ❌. The TL was just assigned.

**Invocation:** `/pom-discovery-gate DISC-2026-05-UC0001 --question=Q3`

**Expected outcome:**
- Skill walks only Q3 (Feasible).
- Other questions remain unchanged.
- Promotion Readiness table and critical-path sequence updated to reflect Q3's new state.
- Shaping log entry appended noting the Q3 update.

---

## Scenario C: promotion-ready (4/4 ✅)

**Setup:** A Discovery file with all 4 questions ✅, Trio-Note gaps closed.

**Invocation:** `/pom-discovery-gate DISC-2026-05-UC0001`

**Expected outcome:**
- Skill confirms all 4 questions and any Trio-Note gaps are ✅.
- Updates the verdict to "READY for promotion to Product Backlog."
- Strongly suggests `pom-promote-to-backlog DISC-2026-05-UC0001`.
- Does NOT itself promote (that's `pom-promote-to-backlog`'s job).

---

## Scenario D: blocking risks in Trio

**Setup:** A Discovery file in a product whose Trio is fully unassigned.

**Invocation:** `/pom-discovery-gate DISC-2026-05-UC0001`

**Expected outcome:**
- Skill detects that **none** of the 4 questions can be answered because no Trio members are assigned.
- Reports the upstream blocker: "Trio in products/claims/trio.md has no assignments. Cannot proceed with Discovery gate."
- Updates the Discovery file's critical-path sequence to show Trio formation as step 1.
- Suggests Trio formation as the prerequisite.

---

## Scenario E: enabling/ai-ethics empty but Q4 relevant

**Setup:** A Discovery file for an AI/ML UC. `enabling/ai-ethics/` does not exist.

**Invocation:** `/pom-discovery-gate DISC-2026-05-UC0001`

**Expected outcome:** During Q4, the skill detects no `enabling/ai-ethics/` folder. Reports the gap. Suggests `pom-seed-enabling-standard --concern=ai-ethics` as a prerequisite to closing Q4. Marks Q4 as ❌ with an explicit blocker.

---

## Reference comparison

Scenario A's output state should match the way Voyager's `DISC-2026-05-UC0001-claims-ai-doc-classification.md` evolved through the conversation — same shape, same critical-path sequence pattern, same shaping-log append-only discipline.
