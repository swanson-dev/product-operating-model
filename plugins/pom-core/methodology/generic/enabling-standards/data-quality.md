# Data quality — generic baseline standard

> **This is a starter.** It governs the *trustworthiness* of data the portfolio produces and consumes. It assumes you have data flowing somewhere — analytics, ML, reporting, partner exchange. If you don't, this is N/A today.

## Scope

Every dataset the portfolio:
- Produces and exposes to other systems (events, exports, ETL targets, model features)
- Consumes from other systems for production decisions (third-party APIs, internal data products)
- Reports on (operational dashboards, executive metrics, regulatory filings)

This is **not** about database administration (uptime, replication, backups — those are operational). It's about whether the *meaning and accuracy* of the data are preserved.

## Baseline rules

### D1 — Source of truth per concept
For every important business concept (e.g., "active customer," "completed order," "monthly revenue"), there is exactly one source-of-truth dataset/system, and the definition is recorded. Multiple "active customer" definitions across teams is the failure mode this rule exists to prevent.
- **Evidence at Q4:** glossary or data-catalog entry with the canonical definition referenced.

### D2 — Schema contracts
Every dataset exposed to other systems has a recorded schema with field-level meaning, types, nullability, and breaking-change policy. Breaking changes follow a documented deprecation cadence.
- **Evidence at Q4:** schema file in source control; subscriber notification path documented.

### D3 — Freshness SLA per dataset
Every dataset has a documented freshness SLA (e.g., "updated daily by 06:00 UTC," "real-time within 60 seconds"). Consumers know what they can rely on. Misses are surfaced, not silent.
- **Evidence at Q4:** SLA recorded; freshness monitoring referenced.

### D4 — Quality monitoring
For each dataset above a documented importance threshold, automated quality checks run (row counts, null rates, distribution shifts, referential integrity). Failures alert.
- **Evidence at Q4:** monitor inventory referenced; alert routing.

### D5 — Lineage where it matters
For datasets used in customer-facing decisions or regulatory filings, the full lineage (raw source → transformations → exposed surface) is documented at a coarse-grained level. Not every column needs lineage, but every regulatory metric does.
- **Evidence at Q4:** lineage diagram or catalog entry.

### D6 — Personal data is classified
Every field containing personal data is classified (PII, PHI, payment, etc.) per the security baseline. Classification drives retention, encryption, and access decisions.
- **Evidence at Q4:** classified field inventory; classification owner identified.

### D7 — Data deletion paths
For every dataset containing personal data, there is a documented path to delete a specific person's data on request (regardless of regulatory mandate). The path is exercised periodically, not just documented.
- **Evidence at Q4:** deletion runbook; last drill date.

### D8 — ML feature lineage and training-set documentation (if applicable)
If the portfolio trains models, every production feature's source is documented. Training set composition is recorded (snapshot date, filters applied, exclusions). Pairs with `ai-ethics.md` A4.
- **Evidence at Q4:** feature catalog; training set manifest.

## Open questions for the standard owner

- What is the importance threshold for D4 — every dataset, or only those above some usage/exposure bar?
- Where does the data catalog live? Is it integrated with the source-of-truth definition (D1)?
- What is the deprecation cadence for breaking schema changes (D2) — 30 days, 90 days, per-consumer negotiation?
- Who owns the classification taxonomy (D6) — security, legal, or data team?
- For consumed third-party data, do we have the same standards expectations, or a lower bar with explicit risk acceptance?

## How this standard interacts with Q4

The `pom-discovery-gate` and `pom-ethics-reviewer` read this standard when grading Q4 sub-questions G2 (data minimisation), G6 (algorithmic accountability — training data), G7 (third-party — consumed data quality), and G8 (enabling-standard alignment).

## See also

- `methodology/generic/enabling-standards/ai-ethics.md` for ML-specific data rules
- `methodology/generic/enabling-standards/security-baseline.md` for data-at-rest and access controls
- `methodology/generic/discovery-q4-framing.md` G2
