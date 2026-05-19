# WSJF Rubric — Healthcare (providers, payers, digital health)

Calibrated for healthcare entities operating under HIPAA, HITECH, FDA, and state health regulators. **Re-calibrate after 10 use cases.**

For formula, scale, and rules, see [`../references/wsjf-formula.md`](../references/wsjf-formula.md).

---

## User-Business Value (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | Trivial. Internal QoL; no measurable patient or workflow impact. | Add a sort to an internal staff dashboard. |
| 2 | Small. One internal team benefits; saves minutes/week. | Filter a member-services queue by plan type. |
| 3 | Modest. Productivity gain for one role *or* small patient-experience bump. | Pre-fill a prior-authorization form from EHR. |
| 5 | Moderate. Material change to a workflow's KPI (handle time, escalation rate, clinician keystrokes). | Cut PA review handle time by 30%. |
| 8 | Substantial. Material lift on a product-line KPI (member retention, clinician satisfaction, claims accuracy). | Lift first-pass claim acceptance rate from 80% to 92%. |
| 13 | Large. Shifts measurable clinical outcomes, MLR, or retention numbers leadership reports on. | Reduce hospital readmission rate by 15% on a chronic-care product. |
| 21 | Transformational. Repositions a product line or opens a new segment. | Launch a new value-based care product. |

## Time Criticality (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | Stable. Same value in 12+ months. | Refactor an internal benefits-summary tool. |
| 2 | Mostly stable. Minor decay over 6–12 months. | Modernize a denials-management workflow. |
| 3 | Soft decay. Internal goal or soft deadline in 6–9 months. | Ship before annual open enrollment. |
| 5 | Real decay. Quarterly business cycle, customer commitment, competitor window. | Match a payer competitor's portal feature pre-renewal. |
| 8 | Significant. Partner-facing commitment or downstream-pod blocker. | Deliver an FHIR integration a hospital partner is scheduling around. |
| 13 | Hard deadline within 6 months. **HIPAA / HITECH update, FDA pre-cert, CMS interoperability rule (TEFCA), USCDI revision**, contractual SLA. | Implement a CMS-mandated interoperability rule by effective date. |
| 21 | Imminent cliff. HIPAA breach trajectory, FDA enforcement, accreditation risk. | Remediate a HIPAA finding risking accreditation status this quarter. |

## Risk Reduction & Opportunity Enablement (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | None. | Tooltip rewording. |
| 2 | Marginal. Paper-cut prevention. | Centralize a duplicated clinical-note template. |
| 3 | Modest. Fixes a known nuisance or enables one trivial future item. | Normalize a diagnosis-code mapping. |
| 5 | Real. Audit/security finding or unblocks one future item. | Add PHI access logging to satisfy a HIPAA gap. |
| 8 | Substantial. Eliminates an incident class or unblocks 2+ shaped items. | Build a de-identification service for analytics consumers. |
| 13 | Large. Systemic risk reduction or enables a product line of future work. | Stand up shared FHIR runway consumed by multiple products. |
| 21 | Portfolio-wide. Existential risk reduction or platform unlock. | Replace a core claims-adjudication or EHR-integration platform. |

## Job Size (1–21)

| Score | Definition | Example |
|---|---|---|
| 1 | < 1 week, 1 person, no schema/integration changes. | Tweak a benefit-rule constant. |
| 2 | 1–2 weeks, 1 person, minor schema or 1 integration. | Add a new field from an EHR webhook payload. |
| 3 | 2–3 weeks, 1–2 people, contained to a single service. | Build a new staff screen on existing FHIR APIs. |
| 5 | One sprint, full pod, 1–2 internal integrations. | Add a new claim sub-type end-to-end. |
| 8 | Two sprints, full pod, multiple integrations or modest runway. | Launch a new provider-facing API reusing existing services. |
| 13 | 3–4 sprints, full pod, significant runway, partner coordination (clearinghouse, EHR vendor, HIE). | Build a new prior-auth API with new schema + EHR partner. |
| 21 | Multi-pod or multi-quarter. **Split first.** | "Replace the claims-adjudication platform" — a program. |

---

## Recalibration triggers

- After 10 use cases scored.
- On regulatory change (HIPAA evolution, CMS interoperability rule update, USCDI revision, FDA Software-as-Medical-Device guidance).
- On adverse audit finding.
- On material drift between scoring expectations and actual delivery.
