# UC-0001 — Accelerate first-notice-of-loss for high-frequency auto claims

> **Worked example — insurance overlay.** Sample Use Case scored with the insurance WSJF rubric and graded with the generic + insurance Q4 framing.

| Field | Value |
|---|---|
| **ID** | UC-0001 |
| **Submitted** | 2026-05-04 |
| **Status** | Scored — awaiting disposition |
| **Source** | VP Claims memo + supporting MMR analysis (`use-cases/claims-fnol-2026-04-29.pdf`) |

## Problem

> "Our auto claims department's first-notice-of-loss (FNOL) intake is taking an average of 18 minutes per claim — competitor benchmarks show 6-8 minutes is achievable with structured guided intake. Of our 220k auto claims annually, ~70% are simple physical damage (no injury, single-vehicle or non-disputed multi-vehicle). For these, our adjusters spend disproportionate time on data entry that could be self-service. Claim cycle time correlates strongly with NPS and retention." — VP Claims, 2026-04-29

## Stakeholders

| Stakeholder | Volume | Evidence | Source |
|---|---|---|---|
| Auto policyholders filing physical-damage claims | ~154k/year (70% of 220k) | Evidence | Memo p.2 |
| Claims adjusters | ~120 FTEs in this queue | Evidence | Memo p.3 |
| Customer support (triaging "where is my claim") | ~3 FTEs absorbing FNOL questions | Inferred | Memo + dashboard cite |
| State insurance regulators (DOIs in licensed states) | Indirect | Implicit | Standard regulatory exposure for claims communications |

## Desired outcomes

- Reduce FNOL handle time from 18 min to 8 min for simple physical-damage claims within 6 months of launch (Evidence: memo target)
- No regression in claim accuracy or fraud indicators (Inferred constraint)
- Maintain or improve NPS on the FNOL touchpoint (Evidence: memo cites NPS-cycle-time correlation)
- Comply with all state-specific claim communication rules (Implicit per I1 / I7)

## WSJF scoring — insurance rubric

| Dimension | Score | Rationale |
|---|---|---|
| User-Business Value | **13** | Material P&L impact via cycle-time and adjuster capacity; NPS lever leadership reports on (rubric anchor: "shifts P&L/NPS"). |
| Time-Criticality | **5** | Competitive window — peer carriers shipping similar in 2026 (rubric anchor: "competitor window"). No hard regulator deadline. |
| Risk Reduction / Opportunity Enablement | **8** | Reduces operational risk (overworked adjusters in surge); enables future automated triage and SLA improvements (rubric anchor: "enables a strategic capability"). |
| Job Size | **5** | Effort estimate: 1 backend pod 8 weeks (intake routing, structured form), 1 design pod 4 weeks (mobile-first FNOL), 1 actuarial week (fraud detection signal review). |
| **WSJF** | **(13 + 5 + 8) / 5 = 5.2** | Strong score. |

## Insurance-specific Q4 considerations pre-Discovery

These will surface in the Discovery gate (DISC-2026-05-fnol-acceleration):

- **I1 — Regulatory exposure**: state-by-state notice and acknowledgement requirements differ — some states require claim acknowledgement within 15 days of FNOL; ensure structured-intake confirmation meets this. Compliance owner: TBD.
- **I3 — Policyholder vulnerable populations**: catastrophe-impacted policyholders (hurricane, wildfire) typically need human-touch FNOL, not self-service. Must have a routing rule that escalates these.
- **I5 — Producer / distribution chain**: agent-assisted FNOL still needs to function; producers must not lose visibility into their book's claims.
- **I7 — Claim-experience transparency**: structured intake outputs must be reflected in the policyholder's view; appeals/contact path surfaced clearly if intake routes them to a longer adjustment process.

## Recommendation

- **Disposition: route** to the Claims product.
- Discovery priorities: I3 (catastrophe routing rule) and I1 (state notification compliance) are the gating sub-questions. Q2 viability is straightforward; Q3 feasibility is mostly about the integration with the policy-of-record system.

## Open questions before scoring confidence

- Memo cites "competitor benchmarks" but doesn't name them — request specificity for I-Q5 / Time-Criticality.
- "70% are simple" — verify against the actual claim mix; the wrong distribution shifts the lift.
- Has Legal reviewed the state-by-state notice timing yet, or is that surfacing here for the first time?
