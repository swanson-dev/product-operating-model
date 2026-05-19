# Architectural Runway

The technical foundation that must exist **ahead of** pod work so vertical slices can ship without each pod reinventing plumbing.

## What lives here

| Artifact | File pattern | Purpose |
|---|---|---|
| **ADR** | `ADR-NNNN-<slug>.md` | A single binding decision: context, options, decision, consequences. Numbered sequentially; immutable. |
| **Pattern** | `PATTERN-<slug>.md` | A reusable design pattern documented for the product's pods. |
| **Runway Plan** | `runway-plan.md` | Living document listing current ADRs, patterns, capacity assumptions, runway investments. |
| **Model Cards** | `model-cards/MC-<slug>.md` | For AI/ML use cases — see `the pom-core plugin's methodology/templates/enabling-standards/ai-ethics/model-card-template.md`. |

## Ownership

The **Technical Leader** in [`../trio.md`](../trio.md) owns the runway, but it is built by Pods as part of their slice work. The TL prevents the runway from becoming a separate-team ivory tower.

## Why this matters

Vertical slicing only works if there is runway laid down ahead of where pods are about to ship. Without it, "vertical slice" devolves into "every pod hand-rolls auth, CI, observability, data plumbing" — the #1 failure mode of Product Operating Model adoption.

For the full rules see `the pom-core plugin's methodology/references/runway-rules.md`.
