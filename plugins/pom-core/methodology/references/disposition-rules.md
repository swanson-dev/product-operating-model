# Disposition Rules

Every Use Case must be **dispositioned before leaving intake**. No silent routing, no items in limbo.

## The four dispositions

| Disposition | Meaning | Required fields |
|---|---|---|
| **kill** | Will not be pursued. Documented and closed. | Rationale. |
| **park** | Held for future re-evaluation. | Trigger condition for re-scoring (date, event, dependency). |
| **route** | Sent to a specific product's Discovery Backlog. | Receiving product slug + Trio members named. |
| **split** | Decomposed into 2+ smaller use cases. Original is killed; new ones are scored individually. | List of new UC IDs created. |

## File naming

`DISP-YYYY-MM-DD-UC-NNNN.md` — one file per disposition. Date is the decision date.

## Required fields in every disposition file

- **Disposition ID** (matches filename)
- **Use Case ID** referenced
- **Decision date** (ISO 8601)
- **Deciders** (who was in the room — Trio members, Enablement Team, PMs of candidate products)
- **Decision** (one of the four above)
- **Rationale** (why; what alternatives were considered)
- **Next action** (varies by disposition — see table above)
- **Stakeholder-facing summary** (paste-ready)

## Append-only rule

**Dispositions are immutable.** Never edit a past disposition. If a decision is reversed:

1. Write a **new** disposition file referencing the old one.
2. Update the UC status field to reflect the new disposition.
3. Do **not** delete or edit the original — it remains the historical record.

This rule is enforced by `pom-validate` (best-effort via git history when available).

## Stakeholder visibility

Dispositions are **visible to stakeholders**. Write rationale assuming the stakeholder will read it. The "Stakeholder-facing summary" field is a paste-ready paragraph for status updates.

## Routing dependencies

- **Routing to a non-existent product** is allowed (and expected during portfolio bootstrap). The disposition file documents the routing decision AND lists the precondition (product must be created before the UC lands in Discovery).
- **Splitting a UC** is allowed. Original UC is marked `killed-by-split`; the disposition lists the new UC IDs and assigns each a brief rationale for the split shape.
