# WSJF Rubric — Retail / e-commerce

Calibrated for retail and e-commerce — physical, digital, omnichannel. **Re-calibrate after 10 use cases.**

For formula, scale, and rules, see [`../references/wsjf-formula.md`](../references/wsjf-formula.md).

---

## User-Business Value (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | Trivial. Internal QoL; no measurable customer impact. | Add a sort to an internal merch-ops dashboard. |
| 2 | Small. One internal team benefits; saves minutes/week. | Filter a returns queue by category. |
| 3 | Modest. Productivity gain for one role *or* small CSAT bump. | Pre-fill shipping fields from saved addresses. |
| 5 | Moderate. Material change to a workflow's KPI (PDP-to-cart, cart-to-checkout, returns handling time). | Lift add-to-cart rate by 2pp on a category page. |
| 8 | Substantial. Material lift on a product-line KPI (conversion, AOV, retention, fulfillment time). | Lift checkout completion rate from 65% to 75%. |
| 13 | Large. Shifts GMV, NPS, or retention numbers leadership reports on. | Reduce average fulfillment time by 24 hours across all SKUs. |
| 21 | Transformational. Repositions a product line or opens a new segment. | Launch a new marketplace or fulfillment-as-a-service product. |

## Time Criticality (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | Stable. Same value in 12+ months. | Refactor an internal merchandising report. |
| 2 | Mostly stable. Minor decay over 6–12 months. | Modernize a returns-management workflow. |
| 3 | Soft decay. Internal goal or soft deadline in 6–9 months. | Ship a feature ahead of next planning cycle. |
| 5 | Real decay. Quarterly cycle, **holiday window (Black Friday, back-to-school, Singles Day)**, customer commitment. | Match a competitor's checkout feature before peak season. |
| 8 | Significant. Partner-facing commitment, downstream-pod blocker, or **inventory-cycle dependency**. | Deliver an API a marketplace partner is launching against. |
| 13 | Hard deadline within 6 months. **PCI-DSS, ADA accessibility, GDPR/CCPA, state sales-tax automation**, contractual SLA, **executive-named holiday commitment**. | Implement a PCI-DSS v4 requirement by the carrier's effective date. |
| 21 | Imminent cliff. Compliance violation, payment-processor delisting, or peak-season business stoppage. | Remediate a PCI finding risking ability to process payments before Q4. |

## Risk Reduction & Opportunity Enablement (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | None. | Tooltip rewording. |
| 2 | Marginal. Paper-cut prevention. | Centralize a duplicated SKU-attribute mapping. |
| 3 | Modest. Fixes a known nuisance or enables one trivial future item. | Normalize a payment-method enum. |
| 5 | Real. Audit/security finding or unblocks one future item. | Add ATO-detection logging to satisfy a security gap. |
| 8 | Substantial. Eliminates an incident class or unblocks 2+ shaped items. | Build a feature-flag service letting marketing pods run gated rollouts. |
| 13 | Large. Systemic risk reduction or enables a product line of work. | Stand up shared real-time inventory runway used by 3+ surfaces. |
| 21 | Portfolio-wide. Existential risk reduction or platform unlock. | Replace the core OMS or checkout platform. |

## Job Size (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | < 1 week, 1 person, no schema/integration changes. | Tweak a discount-rule constant. |
| 2 | 1–2 weeks, 1 person, minor schema or 1 integration. | Add a new SKU attribute and surface in PDP. |
| 3 | 2–3 weeks, 1–2 people, contained to a single service. | Build a new ops screen on existing catalog APIs. |
| 5 | One sprint, full pod, 1–2 internal integrations. | Add a new promo type end-to-end. |
| 8 | Two sprints, full pod, multiple integrations or modest runway. | Launch a new marketplace-partner API. |
| 13 | 3–4 sprints, full pod, significant runway, **3PL/carrier/marketplace partner coordination**. | Build a new fulfillment API with new schema + new 3PL integration. |
| 21 | Multi-pod or multi-quarter. **Split first.** | "Replace the OMS" — a program. |

---

## Recalibration triggers

- After 10 use cases scored.
- **Before peak season** — re-score parked items because Time Criticality dynamics shift.
- On regulatory change (PCI-DSS major version, ADA case law, state sales-tax change).
- On material drift between scoring expectations and actual delivery.
