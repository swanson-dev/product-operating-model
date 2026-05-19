# PHI handling — healthcare starter standard

> HIPAA Privacy + Security Rules baseline. Customise to your portfolio type (covered entity vs. business associate), state-specific overlays (e.g., CA CMIA, NY SHIELD), and operational context.

## Scope

Every system that creates, receives, maintains, or transmits PHI. PHI = individually identifiable health information held by a covered entity or business associate. Includes derived PHI (e.g., a risk score linked to an identified patient is PHI).

## Baseline rules

### PH1 — Minimum necessary
PHI access at every layer (system → API → user → field) is the minimum necessary for the intended purpose. Access tiers documented; reviewed at least annually.

### PH2 — BAA scope verified for every third party
Every vendor, processor, or partner with access to PHI has a current Business Associate Agreement. The BAA scope matches actual data flow. New flows require BAA review BEFORE launch.

### PH3 — De-identification: which method, which use
Where the portfolio de-identifies data for secondary use (research, analytics, ML training): the method (Safe Harbor or Expert Determination) is recorded per dataset, and the re-identification risk threshold is documented. Re-identification attempts are prohibited and detected.

### PH4 — Audit logging of PHI access
Access to PHI is logged in a tamper-evident store with: actor, time, target patient(s), purpose. Audit retention meets the longest applicable retention requirement (typically 6 years).

### PH5 — Breach-readiness
The portfolio has a documented Breach Notification Rule readiness: detection mechanism, risk-assessment process, notification triggers, timeline owners. Tested at least annually.

### PH6 — Patient access rights operational
HIPAA's right of access (review and amend records) is implementable. The portfolio has a process to fulfil access requests within HIPAA's required timeframes (typically 30 days, with one 30-day extension).

### PH7 — Encryption posture for PHI
PHI is encrypted in transit and at rest per HIPAA Security Rule technical safeguards. Encryption keys are managed (separation of duties between key owner and data custodian).

### PH8 — Workforce training
Workforce members with PHI access complete privacy and security training at hire and annually. Training records are retained.

## Open questions for the standard owner

- BAA inventory: where does the current list live, who owns refreshes?
- For Expert Determination paths: who is the named expert (internal/external)?
- Audit-log retention: 6 years HIPAA, but some states require more — codify per portfolio state mix.
- Breach detection: integrated with security monitoring, or a separate workflow?

## How this interacts with Q4

Healthcare H1 (PHI handling) and H5 (consent) read this. Cross-ref generic G2 (data minimisation) and G4 (reversibility — PHI exposed cannot be unlearned).
