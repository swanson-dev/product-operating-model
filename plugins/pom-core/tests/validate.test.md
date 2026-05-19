# Test: `pom-validate`

## Scenario A: clean Voyager repo

**Setup:** `e:\Projects\Protective\voyager\` — the hand-built reference repo.

**Invocation:** `/pom-validate e:\Projects\Protective\voyager\`

**Expected outcome:**
- **Zero ERROR-level violations.**
- May report some WARN-level findings (e.g., R3.5 — PD and TL unassigned in `products/claims/trio.md` — but this is *expected* and is exactly what the WARN level is for).
- Exit status: 0 (success).
- Output is structured: grouped by rule ID, with file paths for each finding.

**Pressure points (this scenario is the load-bearing one):**
- If the canonical templates are over-prescriptive and Voyager doesn't match → FAIL (must fix templates, not Voyager)
- If `pom-validate` reports false-positive ERRORs on a legitimate Voyager state → FAIL (rule definitions in `validation-rules.md` need refinement)

---

## Scenario B: fresh `pom-bootstrap` output

**Setup:** Run `pom-bootstrap /tmp/pom-fresh --rubric=insurance` first.

**Invocation:** `/pom-validate /tmp/pom-fresh`

**Expected outcome:** Zero ERRORs. May have INFO-level findings about the empty backlog and empty product list. Exit 0.

---

## Scenario C: deliberately-broken repo

**Setup:** Take a `pom-bootstrap` output and break it:
- Delete `intake/README.md`
- Add a UC file with malformed name: `intake/use-case-backlog/uc1.md`
- Add a Discovery file that references a non-existent UC: `products/claims/discovery-backlog/DISC-2026-05-bad.md` with `Originating UC: UC-9999`
- Create `products/claims/pods/giant-pod/pod.md` listing 12 members

**Invocation:** `/pom-validate /tmp/pom-broken`

**Expected outcome (each violation explicitly reported with file path):**
- ERROR R2.1: `intake/README.md` missing
- ERROR R2.5: `intake/use-case-backlog/uc1.md` does not match `UC-NNNN-<slug>.md`
- ERROR R7.3: `DISC-2026-05-bad.md` references UC-9999 but no such UC exists
- WARN R3.10: `products/claims/pods/giant-pod/pod.md` shows 12 members; out of 2-7 range
- Exit status: non-zero (because ERRORs present)

**Pressure points:**
- If `pom-validate` misses any violation → FAIL
- If `pom-validate` reports the wrong rule ID → FAIL
- If `pom-validate` aggregates rather than naming the specific file → FAIL (must be actionable)

---

## Scenario D: non-POM repo

**Setup:** Run validate on a random directory (e.g., a Node.js project).

**Invocation:** `/pom-validate ~/some-other-project/`

**Expected outcome:** Skill detects missing top-level POM structure (R1.1 catastrophic), reports clearly, does NOT spam every other rule, exits non-zero.

---

## Output format (required)

`pom-validate` output must include:

1. **Header:** target path, ruleset version (v0.1.0), date.
2. **Summary line:** `N ERROR, M WARN, K INFO` and overall PASS/FAIL.
3. **Grouped findings** by rule ID with severity tags `[ERROR R2.5]`, `[WARN R3.10]`, etc., each naming the offending file path.
4. **Final exit status:** non-zero if any ERROR.

---

## Reference comparison

Scenario A (Voyager passes cleanly) is the load-bearing test. If it fails, the templates and/or `validation-rules.md` are wrong — fix those, not Voyager. The whole purpose of Voyager-derived templates is that Voyager IS the conforming reference.
