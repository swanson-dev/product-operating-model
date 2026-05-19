# AI Ethics — Enabling Standards

Standards every AI/ML use case must comply with before promoting from a product's Discovery Backlog to its Product Backlog.

These standards exist to **make the right thing the easy thing** — the operating principle of Layer 3 enabling functions. If a standard creates more friction than value, the standard is broken — raise it for revision rather than working around it.

**Version: v0.1** — seeded from `pom-seed-enabling-standard`. Refine as actual UCs apply them.

## What's here

| File | Purpose | Owner |
|---|---|---|
| [model-card-template.md](model-card-template.md) | Per-model documentation contract — copied and filled for every deployed model | Trio's TL |
| [pii-handling.md](pii-handling.md) | Standards + checklist for PII in AI pipelines (training, inference, logging, retention) | Trio + Data Architect (Enablement) |
| [bias-audit-framework.md](bias-audit-framework.md) | Framework + checklist for pre-deployment and periodic bias audits | Trio + Compliance |
| [decision-log/](decision-log/) | Append-only log of standards-level decisions, exceptions, and precedents | Product Enablement Team |

## Authoritative references (NOT in this folder)

These standards are scaffolding. The authoritative versions of each organization-specific policy live elsewhere — link them here as identified:

- **Legal & Compliance** policies on PII, GLBA, state regulations — *link TBD*
- **Information Security** policy on data classification, encryption, retention — *link TBD*
- **Risk Management** model risk management framework (SR 11-7 / NAIC Model Bulletin aligned) — *link TBD*
- **Regulatory guidance** for jurisdictions in scope (NAIC, NY DFS, Colorado SB21-169, EU AI Act, etc.) — *link TBD*

The files in this folder distill those policies into AI-pipeline-specific actions. **They do not override the authoritative policies.** If a conflict appears, the authoritative policy wins — log the conflict in [`decision-log/`](decision-log/) and escalate to the Product Enablement Team.

## How a UC applies these standards

1. While shaping in Discovery, the Trio runs through the per-UC checklists in `pii-handling.md` and `bias-audit-framework.md`.
2. Findings and decisions become entries in the receiving product's runway folder (Model Card draft, bias audit report).
3. Before promotion to the Product Backlog, the four-question Discovery gate (Q4) confirms all checklist items are closed.
4. Any deviation from defaults is logged in [`decision-log/`](decision-log/).

## Calibration triggers

- **After 3 AI/ML UCs have applied them** — Product Enablement Team reviews for friction, gaps, or over-prescription.
- **After the first regulatory examination** touching an AI model — review against examiner feedback.
- **On material regulatory change** — new state DOI rule, NAIC update, federal action, EU AI Act guidance.
