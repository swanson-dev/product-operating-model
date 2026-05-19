# PII Handling in AI/ML Pipelines

Standards for handling personal information across AI/ML pipelines: **training, inference, logging, retention**. Companion to [model-card-template.md](model-card-template.md) and the organization's authoritative Legal / Compliance / Security policies (linked in [README.md](README.md)).

## Data classifications used here

> Illustrative — your organization's authoritative data classification wins where it differs.

| Class | Examples | Default treatment |
|---|---|---|
| **PII** (Personally Identifying) | Name, address, SSN, DOB, account number, claim ID, policy number | Minimize; encrypt at rest and in transit; access-logged; never sent to 3rd-party LLMs without contractual protection. |
| **PHI** (Protected Health) | Medical history, diagnoses, prescriptions, lab results | PII rules + HIPAA-equivalent contractual protections + stricter retention. |
| **Sensitive non-PII** | Income, employment, occupation, ZIP-level demographics | Minimize; access-logged. **ZIP is a redlining proxy** — see [bias-audit-framework.md](bias-audit-framework.md). |
| **Confidential business** | Internal pricing, models, codes | Confidentiality, not privacy. Separate concern; not the subject of this file. |

## Hard rules (no exceptions without Product Enablement Team approval, logged in `decision-log/`)

1. **PII / PHI never leaves the controlled environment** without a contract explicitly covering AI use — *especially* third-party LLM APIs.
2. **Logs do not contain raw PII.** Log identifiers (claim ID, salted hash), not data.
3. **Inference requests are not retained as training data** without explicit Legal sign-off and a documented retention period.
4. **Sensitive attributes** (race, religion, national origin, disability, genetic info, sex, age 40+, marital status) are **not features** in any model without an affirmative legal basis and a bias audit.
5. **Data minimization is the default.**

## Per-UC checklist (must be complete before promotion to Product Backlog)

- [ ] **Data inventory** — every field used in training and inference listed and classified.
- [ ] **Legal basis confirmed** for each PII/PHI field. Cite the policy or written approval.
- [ ] **Minimization review** — for each PII/PHI field, document why this model needs it.
- [ ] **De-identification path documented** — technique named (FPE, k-anonymity, tokenization, salted hashing).
- [ ] **Re-identification risk assessed** — can outputs be combined with public data to re-identify?
- [ ] **Logging review** — logs reviewed line-by-line; no raw PII appears.
- [ ] **Third-party processor list** — every external service that may see data is named; contracts confirmed.
- [ ] **Retention policy documented** — training data, inference logs, deletion path.
- [ ] **Cross-jurisdiction transfer reviewed** — if data crosses borders or state lines.
- [ ] **Incident response defined** — trigger conditions; notification chain; window.

## Decision tree — "can I use this field?"

```
Is the field PII or PHI per the table?
├─ No  → Is it sensitive non-PII?
│        ├─ No  → OK to use. Minimize anyway.
│        └─ Yes → Bias audit required (see bias-audit-framework.md).
└─ Yes → Does the model genuinely need this field?
         ├─ No  → REMOVE. Document removal in Model Card §3.
         └─ Yes → Is the legal basis documented and current?
                  ├─ No  → STOP. Engage Legal/Compliance.
                  └─ Yes → Document in Model Card §3 and §7.
```

## When this file is wrong

If applying this checklist would cause a delivery to violate an authoritative policy or regulation, **the authoritative policy wins** — this file is scaffolding, not law. Log the conflict in [`decision-log/`](decision-log/).
