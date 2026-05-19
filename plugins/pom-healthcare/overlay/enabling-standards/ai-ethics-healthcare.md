# AI ethics — healthcare overlay starter standard

> Healthcare-specific addendum to `pom-core`'s generic `ai-ethics.md`. **Read both** for any ML model used in healthcare context. The generic A1–A6 still apply; this overlay adds clinical-context rules.

## Scope

Every ML model used in a healthcare context where outputs inform clinical or care decisions, patient-facing personalisation, or operational decisions affecting access to care. Includes vendor models embedded in clinical workflow.

## Healthcare-specific rules (in addition to generic A1–A6)

### HE1 — Clinically-stratified bias audit
For any model with clinical influence, bias audits include clinical strata where appropriate: by reported race/ethnicity, sex/gender, age cohort, payer source, geography, and condition severity. Beyond demographics: audit across clinically-relevant cohorts (e.g., comorbidity profiles, prior care setting).

### HE2 — Dataset shift monitoring
Beyond generic drift monitoring (generic A5), healthcare models monitor for clinically-meaningful dataset shift: changes in case mix, coding patterns (e.g., post-ICD update), care setting distribution, payer mix. Shift triggers re-validation, not silent acceptance.

### HE3 — Clinically-aware model cards
Model cards (generic A1 baseline) for healthcare models additionally include: intended clinical setting and patient population, contraindications / scope exclusions, training data composition by clinical strata, clinical performance metrics (sensitivity / specificity / PPV / NPV at chosen threshold), known limitations and failure modes.

### HE4 — Human-in-the-loop for clinical-impact outputs
Reinforces generic A3. For healthcare: "high-stakes" defaults to ANY output that influences diagnosis, treatment selection, medication dosing, care escalation, or denial of care. Override paths and overrides themselves are logged and reviewable.

### HE5 — GMLP alignment cues
For SaMD candidates: align with FDA's Good Machine Learning Practice principles. Reference: predetermined change-control plan (PCCP) considerations; locked-vs-adaptive distinction; post-market real-world performance.

### HE6 — Training-data PHI hygiene
For training data containing PHI: documented under the PHI-handling standard (PH3 de-identification). Training on identified data requires recorded patient consent or appropriate legal basis under HIPAA. Audit-logged per PHI access requirements.

### HE7 — Clinically-meaningful explanations where decisions affect patients
For patient-facing model outputs (or clinician-facing where the output substitutes for clinical reasoning), the model output is accompanied by a clinically-meaningful explanation — not just a confidence score. The clinician (or patient) understands the reasoning at the level needed to accept or override.

## Open questions for the standard owner

- HE1 strata selection — clinical advisors should weigh in per clinical domain.
- HE5: are any current products SaMD candidates, or might they become so?
- HE7: who defines "clinically-meaningful explanation" — clinical leadership, model developers, or jointly?

## How this interacts with Q4

Healthcare H2 (clinical safety), H4 (health equity), H8 (FDA boundary). Cross-ref generic G6 (algorithmic accountability) and the generic `ai-ethics.md` (A1–A6 all still apply).
