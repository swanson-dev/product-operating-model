# Test: `pom-promote-to-backlog`

## Scenario A: 4/4 ✅ Discovery item, fresh promotion

**Setup:** A POM repo with a Discovery item whose Promotion Readiness table shows 4/4 ✅ and Trio-Note gaps ✅. No PB item yet exists for it.

**Invocation:** `/pom-promote-to-backlog DISC-2026-05-UC0001`

**Expected outcome:**
- Skill verifies all 4 questions are ✅ and Trio-Note gaps are ✅ by reading the DISC file. **Refuses** to promote otherwise.
- Skill uses `AskUserQuestion` to collect:
  - **Outcome** (one sentence — must reference one of the product's ratified outcomes from product.md)
  - **Vertical slice definition** (thin end-to-end customer journey description)
  - **Acceptance signals** (≥ 2 measurable signals)
  - **Runway prerequisites** (links to ADRs / patterns)
  - **Out of scope** for this slice
- Writes `products/<product>/product-backlog/PB-YYYY-MM-<slug>.md` with all required fields.
- Updates the DISC file's status to "Promoted to Product Backlog YYYY-MM-DD" with a link to the PB file.
- Appends a shaping log entry: "Promoted to product-backlog/PB-..."
- Updates `products/README.md` portfolio index (Product Backlog count).
- Reports: PB file path; suggests `pom-form-pod` if no pod exists yet, or that an existing pod can now pull.
- **Paste-ready vertical slice summary** in the CLI report, framed by dashed lines, stating outcome + slice description.
- **Engineering affordances** in the CLI report:
  - ADO-compatible branch name `pb/PB-YYYY-MM-<slug>` (lowercase, kebab-case, no spaces/underscores)
  - PR title in the form `PB-YYYY-MM-<slug>: <outcome>`
  - Markdown PR description body referencing the originating DISC, vertical slice, acceptance signals (as checkboxes), out-of-scope, and linked runway/ADRs

**Pressure points:**
- If skill promotes a 3/4 ✅ item → CATASTROPHIC FAIL (gate is the load-bearing rule)
- If skill writes PB without a vertical slice definition → FAIL
- If skill doesn't update the DISC file's status → FAIL (cross-reference R7.4)
- If skill omits the paste-ready vertical slice summary → FAIL (v0.2 requirement)
- If skill writes the branch name or PR description to disk → FAIL (must live in CLI report only)
- If the branch name contains uppercase, spaces, underscores, or non-PB-ID prefixes → FAIL (ADO compatibility)

---

## Scenario B: not all questions ✅

**Setup:** DISC with 3/4 ✅, Q4 still 🟡.

**Invocation:** `/pom-promote-to-backlog DISC-NNNN`

**Expected outcome:** Skill refuses. Reports the specific question(s) not ✅. Suggests `/pom-discovery-gate DISC-NNNN` to close them. Does not modify anything.

---

## Scenario C: Trio-Note gaps still ❌

**Setup:** DISC with all 4 questions ✅ BUT Trio-Note gaps (business value quantified, strategic priority link) still ❌.

**Invocation:** `/pom-promote-to-backlog DISC-NNNN`

**Expected outcome:** Skill refuses. Names the specific Trio-Note gaps. Suggests closing them before re-running. Does not promote.

---

## Scenario D: PB item already exists for this DISC

**Setup:** A PB file already references this DISC.

**Invocation:** `/pom-promote-to-backlog DISC-NNNN`

**Expected outcome:** Skill detects the existing PB file. Reports it. Asks via `AskUserQuestion` whether this is intentional (e.g., re-shaping after a parked state) — if yes, suggest creating a new PB item with a fresh slug; if no, stop.

---

## Reference comparison

A promoted PB file should match the structure of the template at `the pom-core plugin's methodology/templates/products/_template/product-backlog/PB-template.md` — with all placeholders filled, vertical slice clearly defined, acceptance signals measurable.
