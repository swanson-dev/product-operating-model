# Test: `pom-status`

## Scenario A: full portfolio status (Voyager)

**Setup:** Voyager repo at `e:\Projects\Protective\voyager\` — 1 UC, 1 product (claims, Trio incomplete), 1 Discovery item.

**Invocation:** `/pom-status e:\Projects\Protective\voyager`

**Expected outcome:**
- Skill scans the repo and produces a human-readable summary.
- Output sections:
  1. **Header**: target path, date, last activity (most recent commit or file change)
  2. **Intake**: counts of UCs by status (pending / scored / dispositioned / killed / parked / routed)
  3. **Products**: one row per product showing Trio status, Discovery count, PB count, Pod count
  4. **Recent dispositions**: last N (default 5)
  5. **Recent decisions**: last N decision-log entries across all enabling concerns
  6. **Recent ADRs**: last N across all products
  7. **Open blockers**: prominent surfaces (e.g., "Trio incomplete in 1 product", "Q4 not closeable for 1 DISC")
- Format: text-first, copy-pasteable into a stakeholder update. Tables OK, but no HTML or color codes.

**Pressure points:**
- If skill reports counts that don't match reality → FAIL (must be derived from filesystem)
- If skill fabricates an "open blocker" not supported by file contents → FAIL
- If output uses ANSI color codes or HTML → FAIL (must be plaintext-friendly)

---

## Scenario B: product-scoped summary

**Setup:** Voyager repo.

**Invocation:** `/pom-status e:\Projects\Protective\voyager --product=claims`

**Expected outcome:**
- Same shape as Scenario A but scoped to one product.
- Skip the "Intake" section (intake is portfolio-wide; not relevant to a single product).
- Add product-specific detail: roadmap state (Now / Next / Later counts), runway artifacts count, pods (with PO names).

---

## Scenario C: empty bootstrap

**Setup:** A fresh `pom-bootstrap` output with no UCs and no products.

**Invocation:** `/pom-status`

**Expected outcome:** Skill reports "Portfolio empty. Next step: ingest a UC via `pom-ingest-use-case`". Does not error.

---

## Scenario D: not a POM repo

**Setup:** Random directory.

**Invocation:** `/pom-status`

**Expected outcome:** Skill detects missing structure, suggests `pom-bootstrap`, exits cleanly.

---

## Output format reference

Status output must be plaintext, < 80 columns wide, copy-pasteable. Structure mirrors the disposition file's "Stakeholder-facing summary" pattern — proven readable.
