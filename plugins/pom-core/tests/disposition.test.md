# Test: `pom-disposition`

## Scenario A: route disposition (the most common path)

**Setup:** A POM repo with `UC-0001-claims-ai-doc-classification.md` scored but not yet dispositioned.

**Invocation:** `/pom-disposition UC-0001 --decision=route --product=claims --pm="Carrie Lines"`

**Expected outcome:**
- Skill writes `intake/dispositions/DISP-YYYY-MM-DD-UC-0001.md` with:
  - Disposition ID matching filename
  - UC reference
  - Decision = route
  - Receiving product = claims
  - Deciders, rationale (prompted if not provided), next action chain
  - **Paste-ready stakeholder summary**
- Skill updates the UC file's `Status` field: `routed → claims`
- Skill updates the UC file's `Disposition` link
- Reports: DISP file path; suggests next step `pom-route-to-discovery UC-0001 claims` (or `pom-spinup-product claims` first if product doesn't exist)

**Pressure points:**
- If skill writes DISP file but doesn't update UC status → FAIL
- If skill omits the paste-ready stakeholder summary → FAIL (it's a hard requirement of disposition-rules.md)
- If skill silently invents rationale → FAIL (must prompt)

---

## Scenario B: park disposition with trigger

**Setup:** A POM repo with UC-0002 scored but the timing isn't right.

**Invocation:** `/pom-disposition UC-0002 --decision=park`

**Expected outcome:**
- Skill prompts via `AskUserQuestion` for the park trigger (date, event, or dependency).
- Skill writes DISP file with the trigger in the "Next action" field.
- UC status updated to `parked`.

**Pressure points:**
- If skill writes a park disposition WITHOUT a trigger → FAIL (disposition-rules.md requires it)

---

## Scenario C: kill disposition

**Setup:** UC-0003 was a duplicate of UC-0001.

**Invocation:** `/pom-disposition UC-0003 --decision=kill --rationale="Duplicate of UC-0001"`

**Expected outcome:** DISP file written with kill decision and rationale. UC status set to `killed`. Next action field empty (kill has no follow-up).

---

## Scenario D: split disposition

**Setup:** UC-0004 is too large (Job Size scored 21).

**Invocation:** `/pom-disposition UC-0004 --decision=split`

**Expected outcome:**
- Skill prompts (via `AskUserQuestion`) for the new UC IDs to be created (e.g., `UC-0005,UC-0006`) and a brief rationale per split shape.
- Skill writes DISP file listing the new UC IDs.
- UC-0004 status set to `killed-by-split`.
- Skill does NOT automatically create the new UC files — that's `pom-ingest-use-case`'s job for each new piece. It does, however, suggest the next steps.

---

## Scenario E: already dispositioned

**Setup:** UC-0001 has a prior `DISP-2026-05-18-UC-0001.md`.

**Invocation:** `/pom-disposition UC-0001 --decision=kill`

**Expected outcome:** Skill detects the existing disposition. Asks via `AskUserQuestion` whether this is a **decision reversal** (which requires a NEW DISP file that supersedes the old) or a typo (in which case stop).

If reversal:
- Writes a NEW DISP file dated today, with a "Supersedes DISP-2026-05-18-UC-0001" field.
- Updates UC status to reflect the new decision.
- Does NOT edit or delete the original DISP file (append-only rule).

**Pressure points:**
- If skill silently overwrites the existing DISP file → CATASTROPHIC FAIL (violates append-only rule)
- If skill silently writes a second DISP without flagging the supersession → FAIL

---

## Scenario F: invalid decision value

**Invocation:** `/pom-disposition UC-0001 --decision=maybe`

**Expected outcome:** Skill rejects the invalid value. Lists valid options (kill, park, route, split). Does not modify anything.
