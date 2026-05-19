# Dispositions

Append-only log of decisions made on use cases. Each disposition is one file: `DISP-YYYY-MM-DD-UC-NNNN.md`.

See [`DISP-template.md`](DISP-template.md) for the canonical template.

## The four dispositions

| Disposition | Meaning |
|---|---|
| **kill** | Will not be pursued. Reason must be documented. Use case file marked `killed`. |
| **park** | Held for future re-evaluation. Trigger condition for re-scoring must be documented (date, event, dependency). |
| **route** | Sent to a specific product's discovery backlog. Receiving product and Trio must be named. |
| **split** | Use case is decomposed into 2+ smaller use cases. Original is killed; new ones are scored individually. |

## Required fields

- **Disposition ID** — matches filename
- **Use case ID** — `UC-NNNN`
- **Decision date** — ISO 8601
- **Deciders** — who was in the room (Trio members, Enablement Team, PMs)
- **Decision** — one of the four above
- **Rationale** — why
- **Next action** — for `park`: trigger condition; for `route`: receiving product + Trio; for `kill`: nothing; for `split`: new use case IDs
- **Stakeholder-facing summary** — paste-ready paragraph

## Rules

1. **Append-only.** Never edit a past disposition. If a decision is reversed, write a *new* disposition referencing the old one.
2. **Every use case must have a disposition before it leaves intake.** No silent routing.
3. **Dispositions are visible to stakeholders.** Write the rationale assuming the stakeholder will read it.
