# UC-0001 — Reduce manual KYC reviews for low-risk personal accounts

> **Worked example — fintech overlay.** Sample Use Case scored with the fintech WSJF rubric and graded with generic + fintech Q4 framing.

| Field | Value |
|---|---|
| **ID** | UC-0001 |
| **Submitted** | 2026-05-04 |
| **Status** | Scored — awaiting disposition |
| **Source** | Head of Onboarding memo + risk-ops dashboard (`use-cases/onboarding-kyc-q1.md`) |

## Problem

> "Our personal-account KYC manual-review queue averaged 4,200 cases/month in Q1, up from 2,800 baseline. Of these, 87% are ultimately approved with no findings — they ended up in manual review because our automated KYC vendor flagged a low-confidence match. Each manual review takes ~7 min and the queue routinely exceeds our 24-hour onboarding SLA, costing us ~6% drop-off on flagged customers vs. ~1% on auto-approved." — Head of Onboarding, 2026-04-29

## Stakeholders

| Stakeholder | Volume | Evidence | Source |
|---|---|---|---|
| Personal-account applicants flagged for manual KYC | ~4,200/month | Evidence | Q1 dashboard |
| KYC review analysts | 12 FTEs | Evidence | Memo |
| BSA officer | 1 | Implicit (oversight) | Org structure |
| External KYC vendor | 1 | Evidence | Memo |

## Desired outcomes

- Reduce manual-review volume by ≥50% within 90 days of launch (Evidence: memo target)
- Maintain or improve false-negative rate (no degradation in caught bad actors) (Inferred constraint)
- Improve onboarding SLA conformance from current 73% to ≥95% (Evidence: memo target)

## WSJF scoring — fintech rubric

| Dimension | Score | Rationale |
|---|---|---|
| User-Business Value | **13** | Customer experience + onboarding conversion (~6% drop-off reduction) plus material operational cost reduction. Anchor: "shifts P&L". |
| Time-Criticality | **5** | Real decay — drop-off compounding monthly; competitor onboarding speed cited as differentiator. No hard regulator deadline. |
| Risk Reduction / Opportunity Enablement | **8** | Reduces BSA exam risk (queue backlog itself can be cited as control weakness); enables future model-driven decisioning for higher-risk segments. |
| Job Size | **8** | Estimated 6-week build: integrate enriched signal sources, build calibrated decisioning layer above vendor, fairness review, BSA officer sign-off, gradual rollout. |
| **WSJF** | **(13 + 5 + 8) / 8 = 3.25** | Moderate; competes with other fintech priorities. |

## Fintech-specific Q4 considerations pre-Discovery

- **F1 — Consumer protection**: ECOA / fair-lending — KYC decisioning is *not* lending decisioning, but adverse-action communication for denied accounts must be reviewed.
- **F2 — BSA/AML**: this is touching the AML/KYC control surface. BSA officer is a required reviewer. F2 ❌ until sign-off recorded.
- **F3 — KYC integrity**: cannot degrade the verification chain. Any auto-approval of a previously-flagged case requires a documented basis (which signal source provided the confidence boost).
- **F5 — Fair-lending exposure**: even though this isn't credit decisioning, the auto-approval logic must not produce disparate outcomes across protected classes. A bias review is required, per MR5 in the model-risk standard.
- **F6 — Payment rails**: N/A — KYC sits upstream of money movement.

## Recommendation

- **Disposition: route** to the Onboarding product.
- Discovery priorities: F2 (BSA officer review) and F3 (KYC integrity preservation) are gating sub-questions. F5 (fair-lending review) is a parallel workstream that cannot block but must complete before promote.

## Open questions before scoring confidence

- "87% are ultimately approved with no findings" — over what sample window? Need confirmation that the proposed signal sources actually predict this 87% reliably.
- BSA officer's appetite for raising or lowering auto-approval threshold over time — needed for the rollout plan.
