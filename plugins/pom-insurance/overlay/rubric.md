# WSJF Rubric — Insurance

Calibrated for insurance carriers (life, P&C, health) operating in regulated markets. Derived from Voyager (Protective Life) v1.0 calibration. **Re-calibrate after 10 use cases.**

## The formula

```
WSJF = Cost of Delay ÷ Job Size

Cost of Delay = User-Business Value
              + Time Criticality
              + Risk Reduction & Opportunity Enablement
```

Each component is scored on a **modified Fibonacci scale**: `1, 2, 3, 5, 8, 13, 21`.

**Rules:**
- Job Size = 21 is hard-prohibited — split first.
- Re-calibrate the rubric after 10 use cases.
- Score at least 3 use cases together; relative ordering is what matters.

For deeper rationale and recalibration triggers, see [`../references/wsjf-formula.md`](../references/wsjf-formula.md) (if your bootstrap created an `intake/references/` directory).

---

## User-Business Value (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | Trivial. Internal QoL; no measurable customer impact. | Add a "last refreshed" timestamp to an internal admin screen. |
| 2 | Small. Benefits one internal team; saves minutes/week. | Filter an underwriter queue by line of business. |
| 3 | Modest. Productivity gain for one role *or* small CSAT bump. | Auto-fill agent's saved customer details on a renewal form. |
| 5 | Moderate. Material change to a single workflow's KPI. | Reduce manual claim-data-entry steps from 12 to 4 for a peril type. |
| 8 | Substantial. Material lift on a product-line KPI (underwriting throughput, claims cycle time, distribution conversion). | Auto-triage 70% of property claims to the right queue at intake. |
| 13 | Large. Shifts P&L, NPS, or retention numbers leadership reports on. | Cut average claims resolution time from 48 hours to 4 hours. |
| 21 | Transformational. Repositions a product line or opens a new segment. | Launch a direct-to-consumer underwriting product on a new pricing model. |

## Time Criticality (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | Stable. Same value in 12+ months. | Refactor the agent commission report layout. |
| 2 | Mostly stable. Minor decay over 6–12 months. | Modernize a back-office report getting harder to maintain. |
| 3 | Soft decay. Internal goal or soft deadline in 6–9 months. | Ship a self-service feature ahead of open enrollment. |
| 5 | Real decay. Quarterly cycle, customer commitment, competitor window. | Match a competitor's quoting feature before an industry conference. |
| 8 | Significant. Partner-facing commitment or downstream-pod blocker. | Deliver an API a distribution partner scheduled their launch around. |
| 13 | Hard deadline within 6 months. **NAIC, NY DFS, Colorado SB21-169**, contractual SLA, executive commitment. | Implement state-mandated disclosure language by regulator's effective date. |
| 21 | Imminent cliff. Compliance violation or business stoppage. | Remediate a finding risking a product line's license to operate within the quarter. |

## Risk Reduction & Opportunity Enablement (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | None. | Add a sort option to a list view. |
| 2 | Marginal. Prevents a paper-cut. | Centralize a copy-pasted email template. |
| 3 | Modest. Fixes a known nuisance or enables one trivial future tweak. | Standardize an error code scheme in one service. |
| 5 | Real. Addresses an audit/security finding or unblocks one future item. | Add audit logging to satisfy a known compliance gap. |
| 8 | Substantial. Eliminates a class of incidents or unblocks 2+ items. | Build a feature-flag service letting 2 pods deliver gated rollouts. |
| 13 | Large. Addresses systemic risk or enables a product line of work. | Stand up shared MLOps runway for AI/ML use cases. |
| 21 | Portfolio-wide. Existential risk reduction or platform-level unlock. | Replace the policy admin core platform that every product depends on. |

## Job Size (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | < 1 week, 1 person, no schema/integration changes. | Tweak validation rules on an existing form field. |
| 2 | 1–2 weeks, 1 person, minor schema or one integration. | Add a field, propagate to one downstream report. |
| 3 | 2–3 weeks, 1–2 people, contained to a single service. | Build a new screen reading from existing APIs. |
| 5 | One sprint, full pod, 1–2 internal integrations. | Add a new claim sub-type end-to-end on existing intake API. |
| 8 | Two sprints, full pod, multiple integrations or modest runway investment. | Launch a new partner-facing API surface reusing existing services. |
| 13 | 3–4 sprints, full pod, significant runway, partner coordination. | Build a new claims intake API with new schema and new partner integration. |
| 21 | Multi-pod or multi-quarter. **Split first.** | "Replace the claims platform" — a program, not a UC. |

---

## Recalibration triggers

- After 10 use cases scored — review for clustering.
- On regulatory change (NAIC Model Bulletin, state DOI rule, federal action).
- On first regulatory examination touching a Voyager AI model.
- On material drift between scoring expectations and actual delivery.

## Worked example (calibration anchor)

See the originating Voyager UC: [`e:\Projects\Protective\voyager\intake\use-case-backlog\UC-0001-claims-ai-doc-classification.md`](file:///e:/Projects/Protective/voyager/intake/use-case-backlog/UC-0001-claims-ai-doc-classification.md). UBV=8, TC=3, RR=8, JS=13 → CoD=19, WSJF≈1.46.
