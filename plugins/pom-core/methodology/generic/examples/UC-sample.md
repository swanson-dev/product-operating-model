# UC-0001 — Reduce checkout abandonment from saved-payment expiry

| Field | Value |
|---|---|
| **ID** | UC-0001 |
| **Submitted** | 2026-05-04 |
| **Status** | Scored — awaiting disposition |
| **Source** | Stakeholder memo from VP Customer (`use-cases/cx-memo-2026-04-29.pdf`, p.3) |
| **Disposition** | _Pending — see /pom-disposition_ |

> **Worked example.** This UC exists in `methodology/generic/examples/` as reference content. A real UC lives under `intake/use-case-backlog/UC-NNNN-<slug>.md` in the user's POM repo, with the same shape.

## Problem (verbatim from source)

> "Customer checkout abandonment in March hit 19% — up from a stable 11% over the prior 6 months. Customer support tickets correlate strongly: 'card declined' complaints rose 3.4× over the same window. Forensics on a sample of 200 abandoned sessions indicates ~70% involve a saved card whose expiry was reached during the session. We have not been notifying customers in advance of expiry, and the in-checkout error message says only 'payment declined — please try another method.'" — VP Customer memo, 2026-04-29

## Stakeholders

| Stakeholder | Role | Volume / context |
|---|---|---|
| Returning customers with saved cards | Primary user | ~340k active (Evidence: customer DB count, memo) |
| Customer Support | Internal | Receiving 3.4× ticket volume (Evidence: support dashboard) |
| Payments team | Owning team | Maintains card-on-file vault (Inferred from org chart) |

## Desired outcomes

- Checkout abandonment returns to baseline 11% within 90 days of fix shipping (Evidence: VP memo's target)
- "Card declined" support tickets drop to pre-March baseline (Evidence: memo, support dashboard)
- No regression in customers updating cards in self-service (Inferred — needs measurement)

## What's already true (constraints)

- Card-on-file vault is managed by an internal Payments service; expiry dates are stored.
- A nightly job exists that flags expiring cards but currently feeds an analytics dashboard only.
- Email and SMS channels are available; customer preferences are honoured.
- No legal requirement to notify of card expiry — this is a customer-experience improvement.

## Decisions already made upstream

- We will NOT auto-update card expiry from the network's account-updater service (separate strategic decision, recorded 2025-Q4).

## WSJF scoring

Following `methodology/references/wsjf-formula.md`:

| Dimension | Score | Rationale |
|---|---|---|
| User-Business Value | **13** | Direct revenue impact — abandonment delta of 8pp on a base of ~$2M/mo checkout flow. Anchor: revenue-direct preset (per `intake/scoring/wsjf-rubric.md`). |
| Time-Criticality | **8** | Bleeding now (compounding monthly). Window: a competitor recently launched proactive card-expiry notifications. |
| Risk Reduction / Opportunity Enablement | **3** | Marginal — doesn't unlock new capability beyond fixing this. |
| Job Size | **5** | Person-weeks estimate: 3-4 backend (use existing nightly job, add notification call), 1-2 frontend (richer in-checkout copy), 1 QA, ~2 weeks total for one pod. Anchor: 1pt ≈ 1 person-week (per calibration). |
| **WSJF** | **(13 + 8 + 3) / 5 = 4.8** | High enough to disposition; not the largest backlog item but clear ROI. |

## Recommendation

- Disposition: **route** to the Payments product (owner of the card-on-file vault).
- Discovery questions to focus on: Q1 (do customers actually want pro-active notification, or would they find it intrusive — quick prompt to a customer sample); Q4-G3 (consent / transparency — are existing comms preferences honoured automatically by the notification path).

## Open questions before scoring confidence

- Anchor "1pt ≈ 1 person-week" is the bootstrap calibration. If the team's actual median sprint slice is bigger, Job Size should rise to 8 (still scores cleanly above the kill threshold).
- "Returns to baseline within 90 days" — does VP want a target month, or 90 calendar days from launch? Memo isn't precise.
