# Test: `pom-form-pod`

## Scenario A: form a pod for a product with PB items

**Setup:** A POM repo with `products/claims/` and at least one PB item ready to pull. No pods exist yet.

**Invocation:** `/pom-form-pod claims triage-pod --sm="Alex Brown" --po="Sam Lee" --members="Jordan,Casey,Riley"`

**Expected outcome:**
- Skill validates the pod slug (kebab-case).
- Skill copies `products/claims/pods/_template/` → `products/claims/pods/triage-pod/` (or uses the canonical template if no per-product _template exists).
- Skill fills `pod.md`:
  - SM = Alex Brown
  - PO = Sam Lee
  - Members = Jordan, Casey, Riley (3 members + SM + PO = 5 total → within 2-7 range)
  - "What this pod owns" section — prompts via `AskUserQuestion` for surface area
  - Cadence defaults
- Creates `products/claims/pods/triage-pod/pod-backlog/` and `slices/` as empty subdirectories with .gitkeep equivalents (or just leave them as empty dirs).
- Updates `products/README.md` portfolio index (Pods count for claims).
- Reports: pod file path; suggests the PO pulls a PB item via natural workflow.

**Pressure points:**
- If member count is < 2 or > 7 → WARN per R3.10 (skill should refuse if < 2 or > 7 and offer to proceed via --force)
- If SM or PO not specified → must prompt via `AskUserQuestion`
- If `products/<slug>/` doesn't exist → abort and suggest pom-spinup-product

---

## Scenario B: composition out of range

**Setup:** Product exists.

**Invocation:** `/pom-form-pod claims giant-pod --members="A,B,C,D,E,F,G,H,I,J"` (10 members)

**Expected outcome:** Skill detects the size violation. Reports it. Suggests forming 2+ smaller pods. Does not write a non-compliant pod by default. Optionally allow `--force` with explicit confirmation, but mark a WARN in the pod.md.

---

## Scenario C: pod slug collision

**Setup:** `products/claims/pods/triage-pod/` already exists.

**Invocation:** `/pom-form-pod claims triage-pod`

**Expected outcome:** Skill refuses. Reports the conflict. Suggests a different slug or editing the existing pod.

---

## Scenario D: product doesn't exist

**Invocation:** `/pom-form-pod nonexistent-product triage-pod`

**Expected outcome:** Skill suggests `pom-spinup-product nonexistent-product` first. Does not modify anything.

---

## Reference comparison

The output `pod.md` should match the structure of `the pom-core plugin's methodology/templates/products/_template/pods/_template/pod.md` with placeholders filled.
