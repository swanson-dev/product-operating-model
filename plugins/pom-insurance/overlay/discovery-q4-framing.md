# Q4 — Insurance industry framing

This file **extends** (not replaces) the generic Q4 sub-questions in `pom-core`'s `methodology/generic/discovery-q4-framing.md`. When `pom-insurance` is installed and active, `pom-discovery-gate` and `pom-ethics-reviewer` read BOTH the generic G1–G8 and the insurance-specific I1–I7 below.

## Insurance-specific sub-questions

### I1 — Regulatory exposure (state DOI, NAIC, federal)
Does the change touch anything that requires state Department of Insurance filing (rates, forms, advertising, agent licensing) or NAIC model-law compliance? If yes, is the filing path identified and the timeline realistic against the regulator's review windows?
- **Evidence patterns:** state-by-state filing matrix; SERFF tracking number(s); legal/compliance sign-off.
- **Default state:** ❌ until filing scope is mapped and a compliance owner is named.

### I2 — Actuarial soundness
Does the change affect pricing, reserving, or loss-cost projections? If yes, has an actuarial review been completed for the proposed change, and is the actuarial memorandum on file?
- **Evidence patterns:** signed actuarial memorandum; reserving model impact analysis; sensitivity test results.
- **Default:** ❌ for any pricing/reserving change; ✅ N/A for purely operational improvements.

### I3 — Policyholder vulnerable populations
Insurance disproportionately affects vulnerable populations at moments of crisis (claim, lapse, denial, rate increase). Does the change improve or degrade outcomes for these moments? Specifically:
- Seniors and adults with reduced cognitive capacity (lapse and re-shopping interactions)
- Beneficiaries (claim adjudication)
- First-time policyholders (renewal communications)
- Policyholders in catastrophe-impacted areas (claim handling and grace periods)
- **Evidence patterns:** UX flow under degraded conditions; clear-language test on legal notices; grace-period extension analysis.

### I4 — Discrimination and disparate impact
Insurance is regulated against unfair discrimination. Does the change introduce, perpetuate, or measurably amplify disparate outcomes across protected classes (and additional state-specific classes — e.g., Colorado SB21-169 for life insurance, NY 187, CA Prop 103)?
- **Evidence patterns:** disparate-impact analysis on the affected rating/underwriting/claims dimension; mitigation plan if uplifts beyond regulatory thresholds.
- **Cross-references:** generic G6 (algorithmic accountability) — both apply when a model is involved.

### I5 — Producer / distribution chain
If the change affects agents/brokers/wholesalers, does it preserve their disclosure obligations to policyholders? Does it affect commission, suitability assessment, or producer licensing requirements?
- **Evidence patterns:** producer-impact summary; compliance training plan if disclosures change; commission impact analysis.

### I6 — Reinsurance and treaty implications
If the change affects ceded business or treaty terms, has the reinsurance team confirmed alignment with current treaties? Does it require treaty amendment?
- **Evidence patterns:** reinsurance review note; treaty amendment scope if needed.
- **Default:** N/A for changes that don't touch underwriting limits or covered perils.

### I7 — Claim-experience transparency
For consumer-facing changes that affect claim outcomes (e.g., new fraud-detection model, automated triage), is the claim communication clear about what was decided and why? Is the appeals/escalation path documented and surfaced?
- **Evidence patterns:** claim communication copy review; appeals flow walk-through; escalation owner identified.
- **Cross-reference:** generic G3 (consent & transparency) — I7 is the insurance specialisation.

## Grading

Use the same ✅/🟡/❌/N/A scale as the generic gate. The DISC's Q4 ✅ requires **all applicable** generic G1–G8 AND insurance-specific I1–I7 to be ✅ — partial coverage is 🟡.

## Recorded exceptions

Where the change deviates from any insurance enabling standard (regulatory reporting, actuarial review, policyholder communications, state filings), the exception MUST be recorded in `enabling/<concern>/decision-log/` via `/pom-decision-log` BEFORE Q4 can be marked ✅.

## See also

- `methodology/generic/discovery-q4-framing.md` — generic G1–G8 (always applies)
- `enabling/regulatory-reporting/README.md` — installed by this overlay
- `enabling/actuarial-review/README.md` — installed by this overlay
- `enabling/state-filings/README.md` — installed by this overlay
- `pom-ethics-reviewer` agent — Q4-only deep audit (reads both generic and insurance framing)
