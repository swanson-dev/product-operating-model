# KYC / AML / BSA — fintech starter standard

> Customise to your charter, customer mix, and FinCEN/regulator examiner expectations.

## Scope

Onboarding identity verification (KYC/CIP), ongoing customer due diligence (CDD), enhanced due diligence (EDD) for high-risk customers, transaction monitoring, sanctions/OFAC screening, SAR/CTR pipeline, beneficial-ownership identification for legal-entity customers.

## Baseline rules

### A1 — CIP at onboarding
Every customer onboarded meets minimum Customer Identification Program requirements (legal name, DOB, address, ID number — SSN/EIN for US persons / acceptable equivalent for non-US). Failure modes (manual review queue) are tracked.

### A2 — Sanctions screening at onboarding and on change
OFAC SDN screening at onboarding, on material changes (name change, address change to sanctioned jurisdiction), and periodically (at minimum monthly batch re-screen). Hits trigger documented investigation workflow.

### A3 — Risk-rating each customer
Each customer is risk-rated at onboarding (low/medium/high). Risk rating drives CDD/EDD requirements and monitoring sensitivity.

### A4 — Transaction monitoring rules current
Transaction monitoring (rules-based and/or model-based) covers the portfolio's actual transaction patterns. Rules are reviewed at least annually and after material product changes.

### A5 — SAR / CTR pipeline functional
SAR filing pipeline (decision → drafting → BSA officer approval → filing → record retention) is documented and timely. CTRs are filed for cash transactions over $10k per FinCEN rules.

### A6 — Beneficial-ownership identification
For legal-entity customers, beneficial-ownership identification per FinCEN CDD final rule (25%+ owners and one controller).

### A7 — Model-risk governance for AML models
Any ML model used for AML/transaction monitoring goes through the `model-risk.md` governance (overlap with that standard).

## Open questions for the standard owner

- Risk-rating methodology in A3 — internal, vendor-provided, or hybrid?
- Transaction-monitoring system — vendor or homegrown? Tuning cadence?
- Who is the named BSA officer (SAR approval, exam point-of-contact)?
- For chartered banks: how does this interact with the parent's program (consolidated vs. distinct)?

## How this interacts with Q4

Fintech F2 (BSA/AML) and F3 (KYC integrity) read this. Cross-references generic G2 (data minimisation — minimum data needed for verification), G6 (algorithmic accountability — AML models), G7 (third-party — vendor screening solutions).
