# UC-0001 — Reduce e-commerce cart abandonment from out-of-stock surprises at checkout

> **Worked example — retail overlay.** Sample Use Case scored with the retail WSJF rubric and graded with generic + retail Q4 framing.

| Field | Value |
|---|---|
| **ID** | UC-0001 |
| **Submitted** | 2026-05-04 |
| **Status** | Scored — awaiting disposition |
| **Source** | VP Digital memo + funnel analytics dashboard (`use-cases/cart-abandon-Q1.md`) |

## Problem

> "Q1 cart abandonment hit 23%, up from 17% steady. Of the abandonments, 31% occur at checkout AFTER the customer has entered shipping/payment info — most of these (about 4,200/month) result from items going out-of-stock between cart add and checkout submission. Customer service tickets show this is a top NPS detractor. We have inventory visibility on the PDP (product detail page) but cart and checkout don't re-check stock before the customer commits." — VP Digital, 2026-04-29

## Stakeholders

| Stakeholder | Volume | Evidence | Source |
|---|---|---|---|
| E-commerce customers reaching checkout | ~340k/month | Evidence | Dashboard |
| E-commerce customers losing item at checkout | ~4,200/month | Evidence | Dashboard segmentation |
| Customer Service (handling NPS detractor follow-up) | 8 FTEs absorbing this | Evidence | Memo |
| Inventory / WMS team | Owning team | Inferred | Org structure |

## Desired outcomes

- Reduce checkout-stage out-of-stock surprises by ≥80% within 60 days of launch (Evidence: memo target)
- No false stockouts (i.e., turning away customers who would have been served) (Implicit constraint)
- Improve NPS detractor count on this driver by ≥30% in 90 days (Evidence: memo)

## WSJF scoring — retail rubric

| Dimension | Score | Rationale |
|---|---|---|
| User-Business Value | **13** | Direct GMV impact — recovered revenue at ~average order value $85 on 4,200 saved carts/month is material; NPS lever leadership reports on. Anchor: "shifts GMV / NPS." |
| Time-Criticality | **8** | Approaching peak window (Black Friday in ~6 months); cart abandonment also pre-school season; competitor cited in memo as having "real-time inventory at checkout." Anchor: holiday window + competitor window. |
| Risk Reduction / Opportunity Enablement | **5** | Enables future improvements (cart-to-checkout flow stability for higher-load events). |
| Job Size | **5** | Estimated 4-week build: real-time inventory check at cart commit + clear in-cart "stock changed" UX + reservation logic for the last-N-units edge case + load test for peak readiness. |
| **WSJF** | **(13 + 8 + 5) / 5 = 5.2** | High; one of the top items. |

## Retail-specific Q4 considerations pre-Discovery

- **R1 — PCI scope**: this is a checkout flow change; PCI scope analysis required even though we don't directly touch PAN handling (the change adds a step before payment submit, which could affect the iframe/hosted-fields flow).
- **R2 — Omnichannel data consistency**: changing how cart reads inventory must remain consistent with store inventory reads. Cannot show "in stock for shipping" on web while store shows "out of stock" with no reconciliation.
- **R3 — Accessibility regulatory exposure**: the new in-cart UX for "item out of stock — alternatives suggested" must meet WCAG. Screen-reader testing required before launch.
- **R5 — Returns / chargebacks fairness**: if we add reservation logic (hold items for cart customers for N minutes), the reservation must release reliably — long-running reservations create their own customer-facing failures.
- **R6 — Peak-window resilience**: changes to cart/checkout path require load testing for peak (Black Friday-scale concurrent inventory reads). Deployment-freeze policy applies if shipping near peak.

## Recommendation

- **Disposition: route** to the E-commerce product.
- Discovery priorities: R6 (peak-window resilience — must complete BEFORE peak freeze) and R2 (omnichannel consistency — coordination with store-fulfillment team) are gating sub-questions.

## Open questions before scoring confidence

- "31% of abandonments occur post-shipping/payment entry" — confirm against actual events; could include other failure modes (validation errors, payment failures) that look the same in the funnel.
- Reservation duration — how long can we hold items for an active cart? Affects inventory utilisation and the omnichannel consistency story.
- Real-time inventory check latency vs. checkout step latency — what's the budget?
