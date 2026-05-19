# PCI handling — fintech starter standard

> Customise to your actual PCI scope, charter, and processor relationships.

## Scope

Every system that stores, processes, or transmits cardholder data (PAN, full track data, CVV, PIN). Plus systems "connected to" the cardholder-data environment (CDE). Scope analysis follows PCI-DSS v4.

## Baseline rules

### P1 — Scope minimisation by design
New changes default to *out of PCI scope*. Any change that would expand scope (introduce PAN handling, expand CDE network reach, etc.) requires explicit security architecture review.

### P2 — Tokenisation as default
Where card data must be referenced, tokens (network tokens or processor tokens) are used, not PANs. PANs at rest exist only in approved vaults; PANs in transit only in approved flows.

### P3 — Approved processor / acquirer for direct PAN
If the portfolio handles raw PANs (not just tokens), only approved processors/acquirers are in the path, and the PCI-DSS responsibility matrix is documented per processor.

### P4 — Encryption-in-transit and at-rest baseline
Beyond the generic security baseline: PCI-specific cipher and key-management constraints apply (PCI-DSS Requirement 4 and 3.5). Key rotation is tracked.

### P5 — Cardholder-data discovery scan
Recurring (quarterly minimum) scan of non-CDE systems for inadvertent PAN storage. Findings tracked and remediated.

### P6 — Annual SAQ or audit
Depending on transaction volume, the portfolio completes an SAQ-A/D or full QSA audit annually. Evidence retained per PCI-DSS retention rules.

## Open questions for the standard owner

- Which processor's responsibility matrix governs (P3)?
- What's the PCI scope baseline today — purely tokens (SAQ-A path), or does the portfolio touch raw PANs?
- Who runs P5 scans — internal security, vendor, or both?

## How this interacts with Q4

Fintech F4 (PCI scope) reads this. Cross-references generic G2 (data minimisation), G7 (third-party — processor relationships).
