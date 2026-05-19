# Architectural Runway Rules

The Runway is the **technical foundation laid down ahead of where pods are about to ship**. Without it, "vertical slice" devolves into "every pod hand-rolls auth, CI, observability, data plumbing" — the most common failure mode of Product Operating Model adoption.

## Ownership

The **Technical Leader (TL)** in the Product Trio owns the runway. The TL does **not** build the runway alone — pods build runway artifacts as part of their slice work, with the TL ensuring coherence.

## What lives in `runway/`

| Artifact | File pattern | Purpose |
|---|---|---|
| **ADR** (Architecture Decision Record) | `ADR-NNNN-<slug>.md` | A single binding decision: context, options, decision, consequences. Numbered sequentially; immutable once recorded. |
| **Pattern** | `PATTERN-<slug>.md` | A reusable design pattern documented for the product's pods. Versioned by edit history. |
| **Runway Plan** | `runway-plan.md` (single file) | Living document. Lists current ADRs, patterns, capacity assumptions, runway investments in flight. Updated continuously. |
| **Model Cards** | `model-cards/MC-<slug>.md` | For AI/ML use cases — see [`../templates/enabling/ai-ethics/model-card-template.md`](../templates/enabling/ai-ethics/model-card-template.md). |

## ADR rules

- **Numbered sequentially** within a product (ADR-0001, ADR-0002, …). Never reused.
- **Immutable.** A reversed decision gets a new ADR superseding the old, not an edit.
- **Required fields:** Context, Decision Drivers, Considered Options, Decision Outcome (with rationale), Consequences (positive + negative).
- **Linked to runway-plan.md:** every ADR must be referenced from the runway plan within 1 working day of being recorded.

## Pattern rules

- **One concern per pattern file.** Don't bundle.
- **Show example usage** in the same file — patterns without examples rot.
- **Patterns are not policy.** They're "here's the shape we've found works." If a pod has reason to deviate, they document why in an ADR.

## Runway plan rules

- **Single file** (`runway-plan.md`) per product. Always current.
- **Sections:** Current commitments, Recent decisions (linked ADRs), Active patterns (linked), Upcoming runway investments, Open questions.
- **Reviewed at each Trio sync** (recommended weekly).
- **Never branches** to a versioned-history file — git is the history.

## Anti-patterns

- **Runway-as-ivory-tower** — the TL writes ADRs alone, never consulted by pods. Sign: pods reinvent things already documented. Fix: TL pairs with pods on first use of each pattern.
- **ADR avalanche** — every micro-decision gets an ADR. Sign: > 50 ADRs in 6 months for one product. Fix: ADRs are for decisions that *bind* future work; routine choices stay in PRs.
- **Stale runway plan** — runway-plan.md hasn't been touched in a month. Sign: pods can't tell what's in flight. Fix: Trio sync agenda includes "runway plan check" each week.
