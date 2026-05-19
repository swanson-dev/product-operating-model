# Test: `pom-spinup-product`

## Scenario A: new product in an existing POM repo

**Setup:** A POM repo created by `pom-bootstrap`. No products exist yet.

**Invocation:** `/pom-spinup-product claims --pm="Carrie Lines" --executive-sponsor="Beth Carter"`

**Expected outcome:**
- `products/claims/` directory created with:
  - `README.md`
  - `product.md` (with `{{product-name}}` → `Claims`, dates filled, PM filled in trio.md ref, but vision/outcomes still `_seed_`)
  - `trio.md` (with PM=Carrie Lines (proposed), PD/TL unassigned and flagged as blocking risks; Executive Sponsor=Beth Carter)
  - `roadmap.md` (Now/Next/Later all empty seed state)
  - `discovery-backlog/` (empty)
  - `product-backlog/` (empty)
  - `runway/` (empty)
  - `pods/` (empty)
- `products/README.md` portfolio index updated to list the new product
- `pom-validate` still passes (Trio incomplete is WARN-level, not ERROR)

**Pressure points:**
- If skill overwrites an existing `products/claims/` → FAIL
- If trio.md doesn't reflect the named PM → FAIL
- If portfolio index in `products/README.md` is not updated → FAIL

---

## Scenario B: product already exists

**Setup:** `products/claims/` already exists from a prior run.

**Invocation:** `/pom-spinup-product claims`

**Expected outcome:** Skill refuses to proceed. Reports the conflict. Does not modify the existing product.

---

## Scenario C: not a POM repo

**Setup:** Random directory that doesn't have the POM structure.

**Invocation:** `/pom-spinup-product claims`

**Expected outcome:** Skill detects the missing structure (no `intake/`, `products/`, or other required dirs), reports clearly, suggests running `pom-bootstrap` first. Does not modify anything.

---

## Scenario D: invalid product slug

**Setup:** A POM repo.

**Invocation:** `/pom-spinup-product "Claims Operations"` (spaces, capitalization)

**Expected outcome:** Skill validates the slug (must be kebab-case, lowercase, no spaces). Reports the violation. Optionally suggests `claims-operations`.

**Pressure points:**
- If skill creates a folder with spaces in the name → FAIL (breaks cross-references later)
- If skill silently sanitizes without informing the user → FAIL (user thinks they got what they asked for)

---

## Reference comparison

Scenario A's output for `products/claims/` should match the structure (not specific content) of `e:\Projects\Protective\voyager\products\claims\`. The seed-state placeholders will obviously differ from Voyager's filled-in content, but the file set and shape must match.
