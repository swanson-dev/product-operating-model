# Test: `pom-pulse`

## Scenario A: portfolio with multiple stuck items across categories

**Setup:** A POM repo with:
- 2 UCs at status `scored` for 10d (past 7d SLA)
- 1 DISC at `Q2 🟡` with last shaping log entry 20d ago (past 14d SLA)
- 1 PB at status `shaped` from 16d ago (past 14d SLA)
- 1 ADR with status `Proposed` from 9d ago (past 7d SLA)
- 1 product with TL unassigned, 30d old, with 2 Discovery items active (past 30d Trio SLA)

**Invocation:** `/pom-pulse`

**Expected outcome:**
- Output is a single markdown block, paste-ready into Teams.
- Header line includes today's date and total stuck count (5 items: 2 UCs + 1 DISC + 1 PB + 1 ADR + 1 Trio gap).
- All five emoji-bolded sections appear (🟠🟡🔵🟣🔴), each with the correct artifacts.
- Each line names the ID, age, and the responsible role/owner/Trio.
- A "Next steps" block at the end recommends `/pom-explain` and `/pom-status --since=last`.

**Pressure points:**
- If skill modifies ANY portfolio file → CATASTROPHIC FAIL (read-only on artifacts)
- If skill includes a Markdown table → FAIL (Teams renders tables poorly; bullets only)
- If skill includes an artifact NOT past its SLA → FAIL (must trace to real over-SLA state)
- If skill includes an empty section header with "(none)" → FAIL (empty sections must be omitted)

---

## Scenario B: nothing is stuck

**Setup:** A POM repo where every artifact is well within its SLA.

**Invocation:** `/pom-pulse`

**Expected outcome:** Skill renders the "all clear" message:
```
**POM Pulse — YYYY-MM-DD**
Portfolio: <target>

✅ No stuck artifacts past current SLAs. The portfolio is moving.
```

---

## Scenario C: custom SLAs

**Setup:** Same as Scenario A, but the team runs aggressive cadence and wants tighter SLAs.

**Invocation:** `/pom-pulse --uc-sla=3 --disc-sla=7 --pb-sla=7 --adr-sla=3 --trio-sla=14`

**Expected outcome:**
- More artifacts now classified as stuck (because thresholds are tighter).
- All flag-provided values appear in the section headers (e.g., "🟠 Use Cases past disposition SLA (3d)").

---

## Scenario D: trend line with carry-over

**Setup:** Same as Scenario A, but `.pom/snapshots/2026-05-18T10-00-00Z.json` exists from yesterday and shows 4 of today's 5 stuck items were also stuck then.

**Invocation:** `/pom-pulse`

**Expected outcome:**
- Header line includes "(4 carried over from 2026-05-18)" after the stuck total.
- Otherwise identical to Scenario A.

**Pressure points:**
- If skill reports carry-over count when no snapshots exist → FAIL (silently omit when no baseline)
- If skill miscounts carry-over → FAIL

---

## Scenario E: not a POM repo

**Setup:** Random directory.

**Invocation:** `/pom-pulse`

**Expected outcome:** Skill detects missing structure, suggests `pom-bootstrap`, exits cleanly. No fabricated digest.

---

## Output format reference

The digest is markdown for Teams/email paste. Bold headers + emoji indicators + bullet lists only. No tables. No HTML. No ANSI color codes.
