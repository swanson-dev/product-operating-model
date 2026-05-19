# WSJF Rubric — Fintech (banking, lending, payments)

Calibrated for fintech operating in regulated markets (US: OCC, CFPB, state banking regulators; EU: PSD2, MiCA; UK: FCA). **Re-calibrate after 10 use cases.**

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
| 1 | Trivial. Internal QoL; no measurable customer impact. | Add a sort to an internal ops dashboard. |
| 2 | Small. One internal team benefits; saves minutes/week. | Surface a derived field in a compliance tool. |
| 3 | Modest. Productivity gain for one role *or* small CSAT bump. | Auto-populate KYC fields from prior verification. |
| 5 | Moderate. Material change to a workflow's KPI. | Cut KYC review handle time by 30% for a tier. |
| 8 | Substantial. Material lift on a product-line KPI (conversion, AUM growth, default rate, fraud loss rate). | Lift onboarding completion rate from 70% to 85%. |
| 13 | Large. Shifts P&L, NPS, retention, or default-rate numbers leadership reports on. | Reduce charge-off rate by 50bps on a lending product. |
| 21 | Transformational. Repositions a product line or opens a new segment. | Launch a new chartered banking product. |

## Time Criticality (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | Stable. Same value in 12+ months. | Refactor an internal accounting report. |
| 2 | Mostly stable. Minor decay over 6–12 months. | Modernize a settlement reconciliation tool. |
| 3 | Soft decay. Internal goal or soft deadline within 6–9 months. | Ship a feature ahead of next annual sales cycle. |
| 5 | Real decay. Quarterly business cycle, customer commitment, competitor window. | Match a fintech peer's feature before their next funding announcement. |
| 8 | Significant. Partner-facing commitment or downstream-pod blocker. | Deliver an API a card network or ledger partner is scheduling around. |
| 13 | Hard deadline within 6 months. **Reg E, Reg Z, BSA/AML, PCI-DSS, GDPR, PSD2 SCA, CFPB rule**, contractual SLA. | Implement a CFPB-required disclosure change by effective date. |
| 21 | Imminent cliff. Compliance violation, charter risk, or business-stopping outage if late. | Remediate a BSA finding risking the bank charter. |

## Risk Reduction & Opportunity Enablement (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | None. | Tooltip rewording. |
| 2 | Marginal. Paper-cut prevention. | Centralize a duplicated transaction format. |
| 3 | Modest. Fixes a known nuisance or enables one trivial future item. | Normalize a transaction-type enum. |
| 5 | Real. Audit/security finding or unblocks one future item. | Add SAR-triggering audit logging. |
| 8 | Substantial. Eliminates an incident class or unblocks 2+ items. | Build an idempotency-key service for payment retries. |
| 13 | Large. Systemic risk reduction or enables a product line of work. | Stand up shared ledger / double-entry runway used by 3+ products. |
| 21 | Portfolio-wide. Existential risk reduction or platform unlock. | Replace the core ledger or core card-processing platform. |

## Job Size (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | < 1 week, 1 person, no schema/integration changes. | Tweak a fee calculation constant. |
| 2 | 1–2 weeks, 1 person, minor schema or 1 integration. | Surface a new field from card network webhook payload. |
| 3 | 2–3 weeks, 1–2 people, contained to a single service. | Build a new ops screen on existing transaction APIs. |
| 5 | One sprint, full pod, 1–2 internal integrations. | Add a new transaction type end-to-end. |
| 8 | Two sprints, full pod, multiple integrations or modest runway. | Launch a new partner API reusing existing services. |
| 13 | 3–4 sprints, full pod, significant runway, partner coordination (card network, banking-as-a-service partner). | Build a new ledger product with new schema + new bank partner. |
| 21 | Multi-pod or multi-quarter. **Split first.** | "Replace the card-processing platform" — a program. |

---

## Recalibration triggers

- After 10 use cases scored.
- On regulatory change (CFPB rulemaking, OCC bulletin, BSA/AML update, PCI-DSS major version, PSD2/MiCA evolution).
- On adverse exam finding.
- On material drift between scoring expectations and actual delivery.
