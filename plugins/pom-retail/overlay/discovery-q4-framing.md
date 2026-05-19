# Q4 — Retail / e-commerce industry framing

Extends the generic Q4 sub-questions in `pom-core`. Retail adds R1–R7.

## Retail-specific sub-questions

### R1 — PCI scope (retail context)
Does the change touch payment-card data — including in-store POS, e-commerce checkout, BOPIS pickup, returns refund-to-card, manual phone orders? If yes, is PCI scope assessed, with scope minimisation by design?
- **Evidence:** PCI scope analysis; tokenisation usage; processor's responsibility matrix.
- **Cross-ref:** generic G2; fintech F4 if you also ship pom-fintech.

### R2 — Omnichannel data consistency
For changes affecting inventory, pricing, promotion, or customer data: are all channels (web, mobile, store, marketplace, call centre) seeing consistent data within acceptable freshness windows? Inconsistent data across channels is the most common cause of customer-facing failure in retail.
- **Evidence:** channel synchronisation diagram; freshness SLAs per channel; reconciliation pipeline status.

### R3 — Accessibility regulatory exposure
Retail e-commerce in the US is subject to ADA Title III enforcement; EU markets are subject to EAA effective 2025; specific state laws (California, New York) impose additional obligations. Does the change preserve accessibility conformance, and does it cover regulatory-required surfaces (web, mobile, retail kiosks, gift cards, receipts)?
- **Evidence:** ADA/EAA conformance check on affected surfaces; VPAT update if applicable.
- **Cross-ref:** generic G1 + the generic accessibility-baseline (X1–X7).

### R4 — Sales-tax automation correctness
For changes affecting checkout, pricing display, or order capture: is sales-tax calculation correct across the portfolio's nexus footprint? Are marketplace-facilitator rules respected?
- **Evidence:** sales-tax vendor calibration; nexus footprint matrix; marketplace-facilitator handling per state.

### R5 — Return, refund, and chargeback fairness
For changes affecting returns, refunds, or post-purchase dispute handling: is the policy applied consistently? Are vulnerable customers (e.g., those targeted by fraudulent sellers) protected? Does the change introduce friction that disproportionately harms one customer cohort?
- **Evidence:** policy-application audit; cohort-stratified disposition analysis.

### R6 — Peak-window resilience
For changes shipping near or during peak windows (Black Friday / Cyber Monday, holiday, back-to-school, regional events): is the resilience plan documented (load test results, freeze plan, rollback procedure)?
- **Evidence:** load test report; deployment-freeze policy compliance; rollback drill.

### R7 — Marketplace / 3P seller integrity (if applicable)
For marketplace operators: does the change preserve integrity of 3P seller listings, pricing, fulfillment promises? Are buyer-protection commitments unaffected?
- **Evidence:** 3P seller impact analysis; buyer-protection coverage review.

## Grading

Same ✅/🟡/❌/N/A scale. Q4 ✅ requires applicable generic G1–G8 AND retail R1–R7 ✅.

## Recorded exceptions

Deviations from any retail enabling standard require entries in the relevant `enabling/<concern>/decision-log/` BEFORE Q4 ✅.

## See also

- `methodology/generic/discovery-q4-framing.md`
- `enabling/pci-retail/README.md`, `enabling/accessibility-regulatory/README.md`, `enabling/omnichannel-data/README.md`, `enabling/fulfillment-integrity/README.md`
- `pom-ethics-reviewer` agent
