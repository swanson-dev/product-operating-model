# Model risk — fintech starter standard

> Inspired by SR 11-7 (Federal Reserve model risk guidance). Adapt to your scale and regulator expectations.

## Scope

Every production model that influences a financial outcome: credit decisioning, pricing, fraud detection, AML monitoring, capital/liquidity, customer-segmentation-driven offers. Includes ML and statistical models.

## Baseline rules

### MR1 — Model inventory
A single inventory of production models exists. Each entry: name, owner, use, vendor (if applicable), risk tier, last validation date.

### MR2 — Risk tiering
Each model is tiered (high / medium / low) based on financial impact + customer impact + regulatory exposure. Tier drives validation cadence and governance scrutiny.

### MR3 — Independent validation before deployment
High-tier models receive independent validation (separate from development team) before production: conceptual soundness, implementation correctness, ongoing performance plan. Validation report committed to `enabling/model-risk/validations/`.

### MR4 — Performance monitoring
Each production model has performance monitoring (accuracy, drift, distribution shift). Thresholds defined per model. Drift breaches trigger investigation, not silent.

### MR5 — Fair-lending review for credit / pricing models
Credit underwriting and pricing models additionally receive fair-lending review (disparate-impact analysis across protected classes), separate from validation.

### MR6 — Vendor model governance
Vendor-supplied models receive the same scrutiny as in-house — vendor's claims do not substitute for independent validation. Vendor SOC reports and SR 11-7 attestations are reviewed if available.

### MR7 — Decommissioning
Retired models stay in inventory (status=decommissioned) with the date and reason. Models cannot be silently replaced; replacement requires new validation.

## Open questions for the standard owner

- Tier-1 validation cadence — annually, or on material change?
- Who owns the validation function (independent function, or dual-hatted analytics team)?
- Vendor models specifically: is there a procurement gate for SR 11-7 attestation?

## How this interacts with Q4

Fintech F5 (fair lending) and F2 (AML model governance) read this. Cross-references generic G6 (algorithmic accountability — this is the fintech specialisation).
