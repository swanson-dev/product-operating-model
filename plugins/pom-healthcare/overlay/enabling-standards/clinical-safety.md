# Clinical safety — healthcare starter standard

> ISO 14971-inspired lite. For full SaMD development, this is a starter and the team's QMS should govern.

## Scope

Every change that could affect a clinical decision, a clinical workflow, a patient outcome, or operator action. Includes both software-as-a-medical-device and software that interacts with clinical workflow but isn't classified as SaMD.

## Baseline rules

### CS1 — Hazard analysis at design
For each change touching clinical workflow or patient-facing output, a hazard analysis identifies: what could go wrong, who could be harmed, severity, probability. The result lives in `enabling/clinical-safety/hazards/HAZ-<slug>.md`.

### CS2 — Severity-stratified mitigation
Mitigations are designed proportional to severity. "Death" or "serious harm" risks require independent review; "inconvenience" risks may need only basic mitigation.

### CS3 — Clinician-in-the-loop for high-stakes
High-stakes clinical decisions (diagnosis, treatment recommendation, automated medication decision) include a clinician-in-the-loop path. The path is documented and used in production.

### CS4 — Change-control gating
Material changes to clinical-impact software receive change-control review. "Material" includes algorithm changes, threshold changes, new data sources, new patient populations, new languages.

### CS5 — Post-deployment surveillance
Clinical-impact software is monitored post-deployment for safety signals (adverse-event reports, near-miss tracking, performance drift). Findings feed back into hazard analysis.

### CS6 — FDA classification determination
For any candidate SaMD or software-in-a-device, FDA classification is determined and documented BEFORE launch. Pre-submission strategy if the path involves FDA review.

### CS7 — Incident response for clinical-safety events
Clinical-safety incidents have a documented response path: triage, immediate-risk reduction, root-cause analysis, corrective action, follow-up. Aligned with the security incident-response playbook but with clinical-specific roles (e.g., chief medical officer involvement).

## Open questions for the standard owner

- For CS1: hazard-analysis template — adapt from ISO 14971, or use a lighter internal format?
- CS3: who is the named clinical reviewer (medical director, clinical SMEs by specialty)?
- CS6: who owns FDA classification determinations — regulatory affairs, an external partner?
- CS5: how is post-deployment surveillance integrated with the operational monitoring stack?

## How this interacts with Q4

Healthcare H2 (clinical safety) and H8 (FDA boundary) read this. Cross-ref generic G6 (algorithmic accountability — model-driven clinical decisions).
