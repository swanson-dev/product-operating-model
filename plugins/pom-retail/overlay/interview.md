# Retail overlay — bootstrap interview

Walk via `AskUserQuestion`. Append answers to `<pom-repo-path>/intake/scoring/retail-calibration.md`.

## R-Q1 — Channel mix
"Channels operated (multi-select): (a) E-commerce (own site). (b) Mobile app. (c) Brick-and-mortar stores. (d) Marketplaces (Amazon/eBay/etc., as seller). (e) Marketplace (own — you operate a 3P seller marketplace). (f) Wholesale / B2B. (g) Direct mail / phone."

## R-Q2 — Geographic footprint
"Primary geographies: (a) US only. (b) US + Canada. (c) US + EU/UK. (d) Global. State-specific obligations to flag (e.g., NY accessibility law, CA Prop 65)?"

## R-Q3 — Fulfillment model
"Fulfillment (multi-select): (a) Own warehouses. (b) 3PL. (c) Drop-ship. (d) Store fulfillment / ship-from-store. (e) BOPIS / click-and-collect. (f) Same-day delivery (own / partner)."
**→** Drives R5 (returns/chargebacks) and fulfillment-integrity standard.

## R-Q4 — Payment processors and stack
"Processors used (top): (a) Stripe. (b) Adyen. (c) Braintree / PayPal. (d) Direct acquirer. (e) Multiple by region. Card-on-file: tokenised via processor / network tokens / both?"
**→** Drives R1 (PCI scope) and pci-retail standard.

## R-Q5 — Peak windows
"Material peak windows you plan around (multi-select): Black Friday/Cyber Monday, Holiday (Nov-Dec), Back-to-School, Mother's/Father's Day, Singles Day, Lunar New Year, Prime Day (if marketplace seller), regional events."
**→** Drives R6 framing.

## R-Q6 — Marketplace operator
"Do you operate a 3P seller marketplace? (Yes / No / Hybrid — you sell on others' marketplaces but don't run one)."
**→** R7 applicable only if yes.

## R-Q7 — Customer data and ML
"Customer-data ML usage (multi-select): (a) Personalisation/recommendations. (b) Search ranking. (c) Pricing/promotions. (d) Fraud detection. (e) Inventory/demand forecasting. (f) None."

## R-Q8 — Loyalty / membership
"Loyalty or paid-membership program? (Yes — paid / Yes — free / No)."
**→** Affects customer-data scope and engagement KPIs in the rubric.

## After the interview

Answers written to `intake/scoring/retail-calibration.md` for downstream rubric/standards customisation.
