# Security — generic baseline standard

> **This is a starter.** It is deliberately minimal. Your security team owns the real standards; this exists so a Discovery item can't pass Q4 without considering security at all.

## Scope

Every change in the portfolio. There is no "this doesn't apply to me" exception — a UI tweak still has session handling; a back-office tool still has access control; a script still has secrets.

What this baseline does **not** cover (industry overlays or your own standards do):
- PCI-DSS scope (use `pom-fintech` or `pom-retail` overlays)
- HIPAA / PHI handling (use `pom-healthcare` overlay)
- State-regulated insurance data (use `pom-insurance` overlay)
- Government-classified data (out of scope for this plugin)

## Baseline rules

### S1 — Identity and access
Every user-facing action runs under an authenticated identity. Every privileged action (admin, support impersonation, data export) is logged with actor + reason + target.
- **Evidence at Q4:** auth flow diagram; admin-action audit log inspection.

### S2 — Secrets and credentials
No secrets in source control. No secrets in CI environment variables that aren't explicitly secret-managed. Rotation policy exists per credential type.
- **Evidence at Q4:** `.gitignore` entry; secret-manager reference; rotation schedule.

### S3 — Transport encryption
All network traffic between the portfolio's services, and between the portfolio and third parties, is encrypted in transit. No exceptions for "internal" or "trusted" networks.
- **Evidence at Q4:** transport summary in the runway plan; certificate management owner identified.

### S4 — Data at rest
PII, customer data, and anything classified as sensitive is encrypted at rest. The key-management approach is documented (KMS, HSM, vendor-managed) per data store.
- **Evidence at Q4:** data-store inventory with encryption + key-management columns.

### S5 — Dependency hygiene
Production dependencies (libraries, container images, vendor SDKs) are scanned for known vulnerabilities. The portfolio has a documented response time for critical CVEs.
- **Evidence at Q4:** scanner output in CI; SLA recorded somewhere (recommend `enabling/security/sla.md`).

### S6 — Logging and observability
Security-relevant events (auth failures, privilege changes, admin actions, anomalous data access) are logged in a tamper-evident store and reviewable. The retention period is recorded.
- **Evidence at Q4:** log inventory; retention statement.

### S7 — Incident response
There is a documented incident-response playbook covering: detection, triage, customer notification thresholds, post-mortem cadence. A new launch confirms the playbook covers the new surface area.
- **Evidence at Q4:** `enabling/security/incident-response.md` exists; new launch's surface area is named in it.

## Open questions for the standard owner

- What is the company's customer-notification threshold for a security incident (1 customer affected? 100? 10000?)? Codify per regulatory exposure.
- What is the critical-CVE SLA — 24h, 72h, 7 days? Document.
- Who owns the secrets-management rotation policy and what tooling enforces it?
- Is there a bug-bounty program? If yes, where do reports land?

## How this standard interacts with Q4

The `pom-discovery-gate` and `pom-ethics-reviewer` read this standard when grading Q4 sub-questions G2 (data minimisation), G4 (reversibility), G5 (dual use / misuse), and G7 (third-party / supply chain).

## See also

- `methodology/generic/discovery-q4-framing.md`
- `methodology/references/validation-rules.md` rule R6 (enabling concern structural integrity)
- Industry overlay plugins for regulated-data-specific rules
