# UC-0001 — Reduce no-show rate for primary-care visits via proactive reminders

> **Worked example — healthcare overlay.** Sample Use Case scored with the healthcare WSJF rubric and graded with generic + healthcare Q4 framing.

| Field | Value |
|---|---|
| **ID** | UC-0001 |
| **Submitted** | 2026-05-04 |
| **Status** | Scored — awaiting disposition |
| **Source** | Chief Medical Officer memo + scheduling analytics dashboard (`use-cases/no-shows-2026-q1.md`) |

## Problem

> "Our primary-care no-show rate is 18.2% across the network, with significant variation by clinic (range 9% to 27%). Each no-show carries an opportunity cost of ~$210 in unfilled clinician time and contributes to access delays for other patients. Our current reminder approach (text 24h before, voice call same-day) hasn't moved meaningfully in 2 years. Patient interviews suggest reminders aren't reaching the right patients in the right form — younger patients ignore voice calls; older patients miss texts; patients with chronic conditions need different reminder framings." — CMO memo, 2026-04-29

## Stakeholders

| Stakeholder | Volume | Evidence | Source |
|---|---|---|---|
| Patients with upcoming primary-care visits | ~180k visits/month | Evidence | Scheduling dashboard |
| PCPs and nursing teams | ~340 PCPs in network | Evidence | Network roster |
| Patients in chronic-condition cohorts | ~22% of visits | Evidence | Dashboard segmentation |
| Care-access coordinators | 18 FTEs (small team) | Implicit | Org chart |
| Patient-access-API consumers (CMS rule) | External | Implicit | Interop. standard |

## Desired outcomes

- Reduce no-show rate by ≥4pp (18.2% → ≤14%) within 6 months of launch (Evidence: memo target)
- No widening of equity gap in no-show rate across reported race/ethnicity or payer-source strata (Implicit constraint — must monitor per HE1)
- Maintain or improve patient experience metric on the reminder touchpoint (Evidence: memo)

## WSJF scoring — healthcare rubric

| Dimension | Score | Rationale |
|---|---|---|
| User-Business Value | **13** | Material clinical-throughput and access lift; pulls a measurable network-wide KPI leadership reports on. Anchor: "shifts measurable clinical outcomes / retention." |
| Time-Criticality | **5** | Real decay — access wait times compounding; competitor health systems shipping similar. No regulatory deadline. |
| Risk Reduction / Opportunity Enablement | **8** | Reduces care-access risk (delayed care correlates with downstream higher acuity); enables future personalised-engagement capabilities. |
| Job Size | **8** | Estimated 8-10 weeks: ML for cohort prediction, clinically-appropriate copy creation, EHR integration for reminder triggers, bias audit per HE1, equity monitoring buildout. |
| **WSJF** | **(13 + 5 + 8) / 8 = 3.25** | Solid but not top-of-stack; competes with other portfolio priorities. |

## Healthcare-specific Q4 considerations pre-Discovery

- **H1 — PHI handling**: reminders reference upcoming visits, which is PHI. Channel choice (SMS, email, voice) must meet minimum-necessary; SMS at minimum metadata only ("you have a visit tomorrow"), not condition or specialty in plain text.
- **H3 — Vulnerable patient populations**: elderly patients (text reminder ineffective), low-literacy patients, limited-English-proficient patients. Multi-modal reminder strategy required.
- **H4 — Health equity**: if reminder personalisation uses ML, must audit for disparate impact across race/ethnicity and payer source per HE1.
- **H5 — Informed consent**: reminders are within TPO; granular consent for channel (text vs. voice vs. portal) must be honoured; revocation must propagate.
- **H6 — Interoperability**: if reminder system integrates with the EHR's scheduling module, FHIR Appointment + Communication resources should be used.
- **H7 — Clinician workflow**: nursing teams should NOT receive new alert burden as a result of this — reminders are patient-facing; clinician dashboards stay as-is.

## Recommendation

- **Disposition: route** to the Patient Engagement product.
- Discovery priorities: H3 (multi-modal strategy for vulnerable cohorts) and H4 (equity audit baseline established BEFORE personalisation ships). H1 (PHI minimum-necessary in channel choice) is gating for Q4 ✅.

## Open questions before scoring confidence

- Memo references "ML for cohort prediction" — is this prediction-only (which patients to remind more aggressively) or also targeting framing? Affects HE1 bias-audit scope.
- Clinic-level variation 9% to 27% — is the variation driven by patient mix or by clinic operations? May change the lever entirely.
- EHR integration scope: which EHR(s) in the network — Epic, Cerner, others? Affects feasibility and integration cost.
