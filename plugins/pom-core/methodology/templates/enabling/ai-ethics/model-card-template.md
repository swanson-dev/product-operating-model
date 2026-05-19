# Model Card — Template

> Copy this template to `<product>/runway/model-cards/MC-<short-slug>.md` and fill in every section. Sections marked **REQUIRED** must be completed before promotion to the Product Backlog. Sections marked **AT DEPLOYMENT** are completed at production cutover.

Based on the Mitchell et al. (2019) Model Card structure, adapted for regulated AI/ML.

---

## 1. Model details (REQUIRED)

- **Model name:** 
- **Model version:** _(semantic versioning — e.g., 1.0.0)_
- **Model type:** _(e.g., LLM-based classifier, gradient-boosted tree, NER, etc.)_
- **Owners:** Trio TL; responsible pod
- **Date trained:** _YYYY-MM-DD_
- **Date deployed:** _YYYY-MM-DD (filled at deployment)_
- **Upstream model / weights provenance:** _(e.g., "fine-tuned from open-source model X v2.1"; "trained from scratch on internal data")_
- **License / contractual terms** on any external model providers, if applicable
- **Originating UC + Discovery item:** _(links)_

## 2. Intended use (REQUIRED)

- **Primary intended use:** _(1–3 sentences; specific about workflow and decision boundary)_
- **Primary intended users:** _(e.g., "Claims processors during initial document intake")_
- **Out-of-scope uses:** _(explicit list of misuses)_
- **Decision boundary:** Does this model **make** an automated decision, or does a human make the decision **with** the model as input? Document the human-in-the-loop design.

## 3. Training data (REQUIRED)

- **Source(s):** systems of record, vintage, volume
- **Data classification:** see [pii-handling.md](pii-handling.md) — what classes are present?
- **Legal basis for use:** documented clearance from Legal/Compliance _(cite)_
- **Preprocessing:** redaction, anonymization, normalization, augmentation
- **Known limitations:** class imbalance, label noise, coverage gaps
- **Sensitive attributes present:** _(race, age, sex, disability, geography proxies, etc.; see [bias-audit-framework.md](bias-audit-framework.md))_

## 4. Evaluation data (REQUIRED)

- **Source(s):** held-out split, separate sample, etc.
- **Representativeness:** how was it verified?
- **Stratification:** what groups was evaluation sliced by?

## 5. Performance metrics (REQUIRED)

Report overall AND per-group metrics. Groups per [bias-audit-framework.md](bias-audit-framework.md).

| Metric | Overall | Group A | Group B | … |
|---|---|---|---|---|
| Accuracy | | | | |
| Precision | | | | |
| Recall | | | | |
| False positive rate | | | | |
| False negative rate | | | | |
| Calibration error | | | | |

## 6. Limitations & known failure modes (REQUIRED)

- Edge cases where the model is known to perform poorly
- Cohorts or input distributions outside training
- Failure modes documented during evaluation

## 7. Ethical considerations (REQUIRED)

- **Bias audit summary** _(link to audit artifact)_ — pass / partial / fail
- **PII handling summary** _(link to PII checklist)_
- **Disparate impact assessment** if the model influences a regulated decision
- **Mitigations applied** in response to ethical findings

## 8. Deployment & monitoring (AT DEPLOYMENT)

- **Production serving environment:** infra, version pin, rollback plan
- **Live monitoring metrics:** what is watched in production?
- **Drift detection thresholds and alert routing:** who gets paged?
- **Retraining cadence:** scheduled or triggered?
- **Latency SLA:** target + measured
- **Fallback behavior:** what happens if the model is unavailable or low-confidence?

## 9. Caveats & recommendations (REQUIRED)

- When to retrain
- When to retire the model
- When to escalate to manual review

## 10. Approval chain (AT DEPLOYMENT)

- [ ] Trio (PM, PD, TL) — sign-off
- [ ] Compliance — sign-off
- [ ] Risk Management — sign-off _(if regulated decision)_
- [ ] Information Security — sign-off
- [ ] Product Enablement Team — review

---

**Model Card last updated:** _YYYY-MM-DD_ by _[name]_
**Card version:** _v_
