# pom-healthcare enabling-standards

Four starter standards installed by `pom-healthcare-init`:

| Concern | Why |
|---|---|
| `phi-handling` | HIPAA-aligned PHI handling: minimum necessary, BAA scope, de-identification methods (Safe Harbor / Expert Determination), breach-readiness. |
| `clinical-safety` | Clinical-risk classification (ISO 14971-inspired lite), hazard log, clinician-in-the-loop requirements for high-stakes decisions. |
| `interoperability` | FHIR R4 baseline, USCDI version-currency, terminology bindings, data-exchange contract expectations. |
| `ai-ethics-healthcare` | Healthcare overlay on the generic ai-ethics: clinical-stratified bias audits, dataset shift monitoring, clinically-aware model cards, GMLP alignment. |

Each is a starter — customise to your portfolio type, regulatory exposure, and operational context.

## Interaction with generic baseline

The generic `pom-core` standards stay in place. `ai-ethics-healthcare` specifically *adds* clinical-context rules on top of the generic `ai-ethics.md` — read both for AI/ML changes in healthcare.
