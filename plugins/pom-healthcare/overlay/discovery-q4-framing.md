# Q4 — Healthcare industry framing

Extends the generic Q4 sub-questions in `pom-core`. Healthcare adds H1–H8 below.

## Healthcare-specific sub-questions

### H1 — PHI handling
Does the change collect, store, transmit, transform, or display Protected Health Information? Does it create new PHI (e.g., derived diagnoses, predictive risk scores) where none existed?
- **Evidence:** PHI data-flow map; minimum-necessary justification per field; BAA scope verified for every third party in the path.
- **Default:** ❌ until the PHI flow is mapped and the minimum-necessary determination is documented.

### H2 — Clinical safety risk
Could a malfunction or misuse cause clinical harm — to a patient, an operator, or a population? Severity (none / inconvenience / harm / serious harm / death) × probability informs the response.
- **Evidence:** clinical-safety hazard log (`enabling/clinical-safety/`); mitigations per identified hazard; sign-off appropriate to severity.
- **For SaMD candidates:** FDA classification determination on record before launch.

### H3 — Vulnerable patient populations
Healthcare disproportionately affects vulnerable populations — pediatrics, elderly, disabled, low-income, low-literacy, limited-English-proficient, mental-health users, end-of-life care.
- **Evidence:** UX walkthrough for each affected vulnerable cohort; clear-language test on patient-facing content; readability bar at lowest reading-grade level practical (Flesch-Kincaid 6 for patient consumer copy is a reasonable target).
- **Cross-ref:** generic G1.

### H4 — Health equity and disparate impact
Does the change improve or worsen health-equity outcomes? Specifically: does it close or widen gaps across race, ethnicity, gender, geography, payer source, SDOH proxies?
- **Evidence:** equity-impact analysis using disaggregated data; mitigation plan if the change worsens any cohort's outcomes.
- **Cross-ref:** generic G6 (algorithmic accountability), F5 (fair-lending pattern adapted), and the AI-ethics-healthcare standard.

### H5 — Informed consent and patient communication
For changes affecting clinical decisions, communications, or data sharing: is informed consent (or its waiver per HIPAA's TPO/research/etc.) appropriate, recorded, and revocable? Are patient communications timed correctly per applicable rules (e.g., breach notification per HIPAA Breach Notification Rule)?
- **Evidence:** consent flow walk-through; revocation path verified; breach-notification readiness documented (the obligation arises post-incident; readiness is what's required pre-launch).

### H6 — Interoperability and data exchange standards
For data exchange (with another EHR, payer, HIE, patient-access app under CMS Interoperability rules): does the change preserve or improve standards compliance — FHIR R4, USCDI version currency, terminology bindings (SNOMED CT / LOINC / RxNorm)?
- **Evidence:** standards compliance check; downstream consumer impact assessment.

### H7 — Clinician workflow and burden
For provider-facing changes: does the change reduce or add clinical documentation burden, alert fatigue, click count? Does it interrupt the clinical workflow at a high-cognitive moment?
- **Evidence:** clinician interview / observation; time-and-motion estimate; alert frequency / signal-to-noise calculation.

### H8 — FDA/regulatory boundary
For software-as-a-medical-device (SaMD) candidates or software in a regulated device: where does the change sit relative to FDA classification (Class I/II/III)? Does the change cross from non-SaMD into SaMD scope, or escalate the class?
- **Evidence:** classification determination; quality-system applicability; pre-submission strategy if needed.

## Grading

Same ✅/🟡/❌/N/A scale. Q4 ✅ requires applicable generic G1–G8 AND healthcare H1–H8 ✅.

## Recorded exceptions

Deviations from any healthcare enabling standard require recorded entries in the relevant `enabling/<concern>/decision-log/` via `/pom-decision-log` BEFORE Q4 ✅.

## See also

- `methodology/generic/discovery-q4-framing.md`
- `enabling/phi-handling/README.md`, `enabling/clinical-safety/README.md`, `enabling/interoperability/README.md`, `enabling/ai-ethics-healthcare/README.md`
- `pom-ethics-reviewer` agent
