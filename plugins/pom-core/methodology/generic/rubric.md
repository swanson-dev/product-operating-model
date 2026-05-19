# WSJF Rubric — Generic (industry-agnostic)

Default preset. Use when the portfolio's domain is mixed or doesn't fit insurance/fintech/healthcare/retail. **Re-calibrate after 10 use cases.**

For formula, scale, and rules, see [`../references/wsjf-formula.md`](../references/wsjf-formula.md).

---

## User-Business Value (1–21)

How much do customers or the business gain when this is delivered?

| Score | Definition | Example |
|---|---|---|
| 1 | Trivial. Internal QoL; no measurable customer impact. | Add a "last refreshed" timestamp to an internal admin screen. |
| 2 | Small. Benefits one internal team; saves minutes/week. | Filter a queue by category for one role. |
| 3 | Modest. Measurable productivity gain or small CSAT bump in one journey. | Auto-fill a frequently-typed field. |
| 5 | Moderate. Material change to a single workflow's KPI. | Reduce steps in a common task from 10 to 4. |
| 8 | Substantial. Material lift on a product-line KPI. | Automate 70% of a high-volume manual workflow. |
| 13 | Large. Shifts P&L, NPS, or retention numbers leadership reports on. | Cut a customer-facing cycle time by an order of magnitude. |
| 21 | Transformational. Repositions a product line or opens a new segment. **Ask if this is one use case.** | Launch an entirely new product surface. |

## Time Criticality (1–21)

How fast does the value decay if delayed?

| Score | Definition | Example |
|---|---|---|
| 1 | Stable. Same value in 12+ months. | Refactor a back-office report for readability. |
| 2 | Mostly stable. Minor decay over 6–12 months. | Modernize an internal tool getting harder to maintain. |
| 3 | Soft decay. Internal goal or soft deadline in 6–9 months. | Ship ahead of an annual planning cycle. |
| 5 | Real decay. Quarterly business cycle, customer commitment, competitor window. | Match a competitor's feature before an industry event. |
| 8 | Significant. Partner-facing commitment or downstream-pod blocker. | Deliver an API a partner has scheduled a launch around. |
| 13 | Hard deadline within 6 months. Regulatory, contractual, executive-named. | Implement a regulator-mandated change. |
| 21 | Imminent cliff. Compliance violation or business stoppage if late. **Triple-check this isn't urgency theater.** | Remediate a finding risking a license to operate. |

## Risk Reduction & Opportunity Enablement (1–21)

Does this reduce a *future* risk or unblock *future* work?

| Score | Definition | Example |
|---|---|---|
| 1 | None. | Add a sort option to a list view. |
| 2 | Marginal. Prevents a paper-cut. | Centralize a copy-pasted snippet. |
| 3 | Modest. Fixes a known nuisance or enables one trivial future tweak. | Standardize error codes in one service. |
| 5 | Real. Addresses an audit/security finding or unblocks one future item. | Add audit logging to satisfy a known compliance gap. |
| 8 | Substantial. Eliminates a class of incidents or unblocks 2+ shaped items. | Build a feature-flag service unlocking gated rollouts. |
| 13 | Large. Addresses systemic risk or enables a whole product line of work. | Stand up shared MLOps runway. |
| 21 | Portfolio-wide. Existential risk reduction or platform-level unlock. **Usually multi-quarter — split it.** | Replace a core platform. |

## Job Size (1–21)

Effort end-to-end through a pod, including discovery + runway. **Not story points.**

| Score | Definition | Example |
|---|---|---|
| 1 | < 1 week, 1 person, no schema/integration changes. | Tweak validation on an existing field. |
| 2 | 1–2 weeks, 1 person, minor schema or 1 integration. | Add a field, propagate to one report. |
| 3 | 2–3 weeks, 1–2 people, contained to a single service. | Build a new screen on existing APIs. |
| 5 | One sprint, full pod, 1–2 integrations. | Add a sub-type end-to-end on existing API. |
| 8 | Two sprints, full pod, multiple integrations or modest runway. | Launch a new partner-facing surface. |
| 13 | 3–4 sprints, full pod, significant runway, partner coordination. | New API + new schema + new integration. |
| 21 | Multi-pod or multi-quarter. **Do not score — split.** | "Replace the X platform" (a program, not a UC). |

---

## Recalibration triggers

- After 10 use cases have been scored — review for clustering.
- On regulatory change or new market entry.
- On first material drift between scoring expectations and actual delivery.
