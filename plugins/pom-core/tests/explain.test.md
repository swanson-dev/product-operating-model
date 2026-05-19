# Test: `pom-explain`

## Scenario A: explain a scored UC

**Setup:** A POM repo with `intake/use-case-backlog/UC-0042-card-expiry-notifications.md` scored (WSJF=21), submitted by "J. Park".

**Invocation:** `/pom-explain UC-0042`

**Expected outcome:** Single line in the report. Shape:
```
UC-0042: Use Case, scored (WSJF=21), submitted by J. Park, Nd old
```

**Pressure points:**
- If skill writes ANY file → FAIL (read-only contract)
- If skill outputs multi-line report with headers → FAIL (single-line contract)
- If skill invents missing fields instead of writing `unknown` → FAIL

---

## Scenario B: explain a Discovery item with mixed gate states

**Setup:** A POM repo with `products/claims/discovery-backlog/DISC-2026-05-claims-doc-ai.md` at Q1 ✅, Q2 🟡, Q3 ✅, Q4 ✅. NOT READY because Q2 is yellow on "actuarial sign-off". Trio: M. Park / J. Lee / D. Chen.

**Invocation:** `/pom-explain DISC-2026-05-claims-doc-ai`

**Expected outcome:**
```
DISC-2026-05-claims-doc-ai: Discovery, ✅/🟡/✅/✅ blocked on actuarial sign-off, Trio M. Park / J. Lee / D. Chen, Nd old
```

---

## Scenario C: explain an ADR

**Setup:** `products/claims/runway/ADR-0003-drift-detection-cadence.md` exists, Status = Accepted, TL = D. Chen.

**Invocation:** `/pom-explain ADR-0003`

**Expected outcome:**
```
ADR-0003 (claims): drift-detection-cadence, Accepted, TL D. Chen, Nd old
```

---

## Scenario D: ambiguous ADR across products

**Setup:** Both `products/claims/runway/ADR-0001-foo.md` and `products/policy/runway/ADR-0001-bar.md` exist.

**Invocation:** `/pom-explain ADR-0001`

**Expected outcome:** Two lines, one per match — one for `ADR-0001 (claims)` and one for `ADR-0001 (policy)`. Skill explicitly notes ambiguity is real for `ADR-NNNN` across products.

---

## Scenario E: artifact not found

**Invocation:** `/pom-explain UC-9999` (no such UC exists)

**Expected outcome:** Skill aborts with "Artifact UC-9999 not found." Does not modify anything. Does not invent fields.

---

## Scenario F: path input instead of ID

**Invocation:** `/pom-explain products/claims/runway/ADR-0003-drift-detection-cadence.md`

**Expected outcome:** Skill uses the path directly, skips the glob step, and produces the same one-line summary as Scenario C.

---

## Scenario G: explain a Disposition

**Setup:** `intake/dispositions/DISP-2026-05-12-UC-0001.md` exists, Decision = route, receiving product = claims, Deciders = "Beth Carter (Exec Sponsor), Carrie Lines (PM)".

**Invocation:** `/pom-explain DISP-2026-05-12-UC-0001`

**Expected outcome:**
```
DISP-2026-05-12-UC-0001: route → claims, by Beth Carter + Carrie Lines, Nd old
```

---

## Pressure points (all scenarios)

- Read-only contract: any file write is a catastrophic FAIL.
- Single-line output contract: any multi-line, headered, or commentary-wrapped output is a FAIL.
- `unknown` for unparseable fields: any invented value is a FAIL.
