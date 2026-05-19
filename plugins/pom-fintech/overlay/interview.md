# Fintech overlay — bootstrap interview

Walk the user through each question via `AskUserQuestion`. Append answers to `<pom-repo-path>/intake/scoring/fintech-calibration.md`.

## F-Q1 — Product mix
"Which products does this portfolio cover? Multi-select: (a) Deposit accounts (checking/savings). (b) Consumer lending. (c) Commercial lending. (d) Payments (issuing, acquiring, processing). (e) Money movement (ACH, wire, RTP, cross-border). (f) Investment/brokerage. (g) Crypto / digital assets. (h) Insurance (cross-sell)."

## F-Q2 — Charter and licensing posture
"Are you: (a) Chartered bank (national, state). (b) Partner-chartered (sponsor bank model). (c) State-licensed money transmitter / lender. (d) Broker-dealer / RIA. (e) Hybrid (explain)."
**→** Drives F1/F2/F7 sub-question scope.

## F-Q3 — Customer base
"Customers: (a) US consumer. (b) US small business. (c) US mid-market+. (d) International (which jurisdictions). (e) Internal (B2B fintech provider)."

## F-Q4 — Payment rails in use
"Which rails do you operate over? Multi-select: ACH, wire (Fedwire/CHIPS), card networks (Visa/MC/Amex/Discover), RTP/FedNow, push-to-card (OCT), international (SWIFT/local), crypto/stablecoin."
**→** Drives F6 sub-question.

## F-Q5 — Regulator footprint
"Primary regulators you answer to (multi-select): (a) OCC. (b) CFPB. (c) FDIC. (d) Federal Reserve. (e) State banking regulators (list states). (f) FinCEN. (g) SEC. (h) FINRA. (i) International (EU FCA-equivalents, etc.)."

## F-Q6 — Model usage
"Where do you use ML models in production? Multi-select: (a) Credit underwriting. (b) Pricing. (c) Fraud detection. (d) AML / sanctions. (e) Customer service / chat. (f) Marketing / personalisation. (g) None currently."
**→** Drives F5 (fair-lending) and F2 (AML model) scope.

## F-Q7 — Custody arrangements
"Customer funds: (a) Held in own chartered bank. (b) Held in sponsor-bank FBO accounts. (c) Brokerage cash sweep. (d) Crypto custody (self or qualified custodian). (e) Mix (explain)."
**→** Drives F7 scope.

## F-Q8 — Upcoming compliance windows
"Any known regulator-driven deadlines in the next 12 months? (CFPB final rule, OCC enforcement, state-level effective date.) Type 'none' or list."
**→** Pre-seeds Time-Criticality anchors and watchlist in `enabling/consumer-protection/`.

## After the interview

`pom-fintech-init` writes answers to `intake/scoring/fintech-calibration.md` and uses them to tune rubric anchors and seed enabling-standards starter content.
