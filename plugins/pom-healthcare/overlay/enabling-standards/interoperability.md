# Interoperability — healthcare starter standard

> Customise to your participation in CMS Interoperability rules, TEFCA, and your provider/payer integration partners.

## Scope

Every interface where clinical or administrative health data crosses an organisational boundary — to or from EHRs, HIEs, payer systems, patient-access apps, devices, public-health reporting, partner labs/pharmacies.

## Baseline rules

### IO1 — FHIR R4 as baseline
New data exchange uses FHIR R4 resources where applicable. Legacy formats (HL7 v2, X12, proprietary) require justification documented per interface.

### IO2 — USCDI version currency
For data exchange in scope of CMS Interoperability rules, data elements meet the current USCDI version (v3 at time of writing — update as new versions land). Plan for upcoming versions in the runway.

### IO3 — Terminology bindings
Coded values use the canonical terminologies: SNOMED CT for clinical findings, LOINC for observations/labs, RxNorm for medications, ICD-10-CM for diagnoses (administrative), CPT/HCPCS for procedures.

### IO4 — Patient-matching reliability
For interfaces that match patients across systems, the matching method (deterministic, probabilistic, hybrid) is documented with false-match and false-non-match rate targets. Monitored.

### IO5 — Consent and data-sharing legal basis
Each interface has a documented legal basis for data sharing (TPO under HIPAA, patient consent, BAA, public-health, etc.). Patient revocation of consent (where applicable) propagates.

### IO6 — Interface SLAs and degradation behaviour
Each interface has documented SLAs (availability, throughput) and degradation behaviour. Failure modes are communicated to consuming clinicians — silent failure is unacceptable.

### IO7 — Versioning and deprecation cadence
Breaking changes to interfaces follow a documented deprecation cadence with consumer notification. No surprise breaking changes.

## Open questions for the standard owner

- IO4 false-match thresholds: clinically acceptable rates differ by use (medication reconciliation vs. care-team broadcast).
- IO5 consent: how is granular consent (e.g., omit substance-use records per 42 CFR Part 2) handled in practice?
- IO7 deprecation cadence: 90 days, 180 days, longer for partner-facing?

## How this interacts with Q4

Healthcare H6 (interoperability standards) reads this. Cross-ref generic G7 (third-party — interface partners) and G8 (enabling-standard alignment).
