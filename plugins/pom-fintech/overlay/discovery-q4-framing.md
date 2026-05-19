# Q4 — Fintech industry framing

Extends (not replaces) the generic Q4 sub-questions in `pom-core`'s `methodology/generic/discovery-q4-framing.md`. When `pom-fintech` is installed and active, `pom-discovery-gate` and `pom-ethics-reviewer` read both the generic G1–G8 and the fintech-specific F1–F7 below.

## Fintech-specific sub-questions

### F1 — Consumer-protection regulatory scope (CFPB, Reg E/Z, UDAAP)
Does the change touch credit decisioning, deposit accounts, payment disputes, debt collection, or any user-facing financial outcome subject to consumer-protection regulation? If yes, has compliance reviewed the proposed UX, copy, and decision logic?
- **Evidence:** compliance review record; Reg-by-Reg applicability matrix; copy review for UDAAP risk.
- **Default:** ❌ until compliance sign-off is recorded.

### F2 — BSA/AML and sanctions screening applicability
Does the change touch onboarding, money movement, or beneficial ownership identification? If yes, are the BSA/AML controls preserved or enhanced; is OFAC screening still effective; is the SAR-triggering signal pipeline unbroken?
- **Evidence:** BSA officer sign-off; sanctions-screening regression test result.

### F3 — KYC integrity
For onboarding or re-verification changes, does the change preserve the KYC chain of evidence (document review, biometric checks if applicable, audit trail)? Are downstream consumers of the verification state (e.g., money-movement, lending) still reading from the authoritative source?
- **Evidence:** KYC flow walk-through; downstream-consumer enumeration.

### F4 — PCI scope
Does the change introduce or alter the handling of payment-card data (PAN, CVV, expiry, magnetic stripe)? Even tangentially? If yes, is PCI scope assessed and the change designed to minimise scope?
- **Evidence:** PCI scope analysis; tokenisation/network-token usage; QSA conversation if scope expanded.
- **Cross-ref:** generic G2 (data minimisation).

### F5 — Fair lending and disparate impact (lending-specific)
For lending decisioning changes (underwriting model, pricing model, line management): has a fair-lending analysis been completed for the proposed change, covering both disparate treatment and disparate impact across protected classes?
- **Evidence:** fair-lending analysis from compliance/quant; mitigation plan if disparate impact detected.
- **Cross-ref:** generic G6 (algorithmic accountability).

### F6 — Payment-rails compliance (Reg E, NACHA, card-network rules, RTP)
Does the change touch a specific payment rail? Each has its own rules (Reg E dispute timelines, NACHA same-day ACH rules, card-network chargeback rights, RTP irrevocability). Is the proposed handling aligned per rail?
- **Evidence:** per-rail review note; dispute SLA compliance.

### F7 — Custody and customer-asset segregation
If the change touches customer funds (custody, hold/release, settlement timing), are the segregation/custody rules respected (e.g., FBO accounts, sweep arrangements, bankruptcy-remote structures)?
- **Evidence:** treasury sign-off; legal review for custody structure; reconciliation pipeline still passing.

## Grading

Same ✅/🟡/❌/N/A scale as the generic gate. Q4 ✅ requires all applicable generic G1–G8 AND fintech F1–F7 to be ✅.

## Recorded exceptions

Deviation from any fintech enabling standard requires an entry in the relevant `enabling/<concern>/decision-log/` via `/pom-decision-log` BEFORE Q4 ✅.

## See also

- `methodology/generic/discovery-q4-framing.md`
- `enabling/pci-handling/README.md`, `enabling/kyc-aml-bsa/README.md`, `enabling/consumer-protection/README.md`, `enabling/model-risk/README.md`
- `pom-ethics-reviewer` agent
