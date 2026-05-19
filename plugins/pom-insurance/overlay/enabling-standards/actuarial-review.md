# Actuarial review — insurance starter standard

> Customise to your portfolio's lines of business and actuarial organisation.

## Scope

Any change affecting pricing, reserving, loss-cost projections, or material assumptions feeding any of those. Includes rating algorithm changes, new variables, fraud-detection model deployments that affect claim outcomes, reserving model updates, premium-deficiency-reserve calculations.

Out of scope: changes that affect operational efficiency without changing financial outcomes.

## Baseline rules

### A1 — Actuarial memorandum required for pricing changes
Any rate change (existing or new product) ships with a signed actuarial memorandum stating: assumed loss costs, expected loss ratios, sensitivity ranges, and credibility considerations. The memorandum is committed to the product's `runway/actuarial/AM-<slug>.md`.

### A2 — Reserving model review on material data shifts
If a product change introduces a material shift in the data feeding the reserving model (new exposure base, new coverage form, new geography), the reserving actuary signs off before launch.

### A3 — Model validation for ML-influenced pricing or claims decisions
Any ML model influencing pricing or claim outcomes goes through model validation: independent replication, out-of-sample testing, stability assessment over time. Validation report is in `enabling/actuarial-review/validations/`.

### A4 — Independent peer review at materiality threshold
For changes above a defined materiality threshold (premium impact $X, loss ratio change Y%), an independent peer actuary reviews the work product before launch.

### A5 — IBNR / unreported claim impact assessment
For any claim-handling change (intake automation, triage, fraud detection), the change's effect on IBNR is assessed before launch, even if the assessment concludes "no material impact."

## Open questions for the standard owner

- Where is the materiality threshold for A4 set, and by whom?
- Who is the named independent peer actuary (internal or external)?
- For ML pricing/claims models, is validation owned by actuarial alone or jointly with data science?
- Is there a "go/no-go" actuarial sign-off in CI/CD, or is it manual?

## How this interacts with Q4

Insurance-specific Q4 sub-question I2 reads this standard. Any DISC touching pricing, reserving, or claim economics requires A1–A5 alignment (or recorded exception) before I2 ✅.
