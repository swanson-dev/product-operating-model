# Bias Audit Framework

Framework for **pre-deployment and periodic** bias audits of AI/ML models. Companion to [model-card-template.md](model-card-template.md) and [pii-handling.md](pii-handling.md). Authoritative regulatory definitions live with the organization's Compliance team — this framework *operationalizes* the audit.

## When a bias audit is required

Required when **any** of the following is true:

- The model influences a decision about a person (claim, underwriting, pricing, fraud flag, eligibility, customer treatment).
- The model uses any sensitive attribute as a feature, label, or strong proxy.
- The model's training data is materially imbalanced across protected groups.
- The use case is in a jurisdiction with explicit AI bias rules (e.g., Colorado SB21-169, NY DFS, EU AI Act).
- The Trio determines an audit is warranted even if the above don't trigger.

**Default: assume an audit is required unless the Trio documents why not** (logged in [`decision-log/`](decision-log/)).

## Protected groups and sensitive attributes

The organization's Compliance list takes precedence. Baseline groups Voyager audits against:

- Race / ethnicity
- Color
- National origin
- Religion
- Sex (incl. pregnancy, sexual orientation, gender identity per applicable jurisdictions)
- Age (especially 40+ for ADEA)
- Disability
- Marital status
- Genetic information (GINA)
- Veteran status (per applicable state law)
- ZIP code / geography (redlining proxy)
- Income bracket (proxy concerns)

**Important:** sensitive attributes may not be in the model as *features* but can be *proxied* by correlated variables. The audit must check for proxy effects.

## What an audit measures

For each protected group, compute and compare:

| Metric | What it tests | Default flag threshold (v0.1) |
|---|---|---|
| **Accuracy** per group | Equal performance? | > 5pp difference |
| **False positive rate** | Wrongly flagging one group more? | > 5pp difference |
| **False negative rate** | Wrongly missing one group more? | > 5pp difference |
| **Calibration error** | "70% confident" actually 70% across groups? | > 5pp difference |
| **4 / 5ths rule** (selection rate) | Any group's positive rate < 80% of highest? | Any failure |
| **Equal opportunity** | TPR equal across groups? | > 5pp difference |
| **Equalized odds** | Both TPR and FPR equal? | > 5pp difference on either |

Thresholds are v0.1 defaults. **Compliance to set authoritative thresholds** — log changes in [`decision-log/`](decision-log/).

## Audit process

1. **Define groups.** Trio + Data Architect identify testable protected groups; document gaps.
2. **Build the audit dataset.** Held-out, representative, labeled with group membership.
3. **Compute metrics overall and per group.**
4. **Run a proxy audit.** Train a sidecar model that tries to predict each sensitive attribute from features alone. If sidecar succeeds well above baseline → proxy problem.
5. **Document findings** in an audit report linked from Model Card §7.
6. **Decide** for each flagged disparity:
   - **Accept** — document why; rare; requires Risk + Legal sign-off if regulated.
   - **Mitigate** — rebalance, reweight, remove feature, postprocessing thresholds, defer-to-human.
   - **Block** deployment until resolved.
7. **Log the decision** in [`decision-log/`](decision-log/) as precedent.

## Periodic re-audit

- **Quarterly** for regulated-decision models.
- **On drift event** — material shift in performance or training distribution.
- **On data source change** — training data composition changes.
- **On regulatory change** — new rule → review framework + thresholds.

## What this framework does *not* do

- It does not **define** what bias means. That is Compliance's domain.
- It does not **certify** a model as unbiased. No audit can.
- It does not **replace** Trio + Compliance judgment on disparate impact.

## Per-UC checklist (must be complete before promotion to Product Backlog)

- [ ] Audit required? Yes/No documented with rationale.
- [ ] If yes: groups identified; testability gaps documented.
- [ ] Audit dataset built and documented.
- [ ] Metrics computed overall and per group.
- [ ] Proxy audit run.
- [ ] Findings documented.
- [ ] For each flagged disparity: accept / mitigate / block decided and logged.
- [ ] Audit report linked from Model Card §7.
- [ ] Periodic re-audit cadence set.
