# Consumer protection — fintech starter standard

> Customise to your actual product mix and regulator footprint.

## Scope

Every consumer-facing surface and decision: credit decisioning, deposit account servicing, debt collection, dispute handling, disclosure, marketing, account closure. Subject to CFPB, FTC, applicable state attorneys-general.

## Baseline rules

### C1 — UDAAP review at change
Any user-facing copy or experience change passes a UDAAP (Unfair, Deceptive, or Abusive Acts or Practices) review by compliance. Especially: marketing copy, fees, disclosure timing, default settings.

### C2 — Reg E (electronic funds transfers) handling
For electronic transfers and disputes: timelines (10 business days for provisional credit, 45 days for resolution typical, longer for international/new accounts). Procedures documented and consistently applied.

### C3 — Reg Z (credit disclosures)
For credit products: Truth-in-Lending disclosures provided per Reg Z. APR calculations correct. Change-in-terms notices timely.

### C4 — Fair-lending compliance
For lending decisioning: adverse-action notices with specific reasons (FCRA / ECOA). Fair-lending monitoring per `model-risk.md` for any model-influenced decision. No prohibited basis variables in decisioning.

### C5 — Debt collection
If the portfolio collects debt (including post-charge-off): FDCPA compliance, communication-frequency limits, validation notices.

### C6 — Customer complaint handling
Customer complaint intake (any channel) is tracked. Material complaint patterns route to compliance for systemic review.

### C7 — Account closure protections
Closing a customer account follows documented procedure with applicable notice. Closures for AML reasons follow separate (typically SAR-aware) procedure.

## Open questions for the standard owner

- Where does CFPB complaint-portal monitoring live?
- For lending: how are adverse-action reason codes maintained — vendor mapping, internal mapping?
- Marketing copy review: pre-flight or post-flight by compliance?

## How this interacts with Q4

Fintech F1 (consumer-protection regulatory scope) and F5 (fair lending) read this. Cross-references generic G1 (vulnerable users — financially distressed users are a recurring theme here) and G3 (consent & transparency).
