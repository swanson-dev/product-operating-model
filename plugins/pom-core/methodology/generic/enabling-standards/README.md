# Generic enabling-standards starter pack

When `pom-bootstrap` (or `pom-seed-enabling-standard`) runs with the generic baseline selected and the user opts in (Q7 of `interview.md`), these starter standards get copied into the user's POM repo at `enabling/<concern>/README.md` with a companion empty `decision-log/`.

These are **starters**, not finished standards. Each one defines:
- **Scope** — what this concern covers and (importantly) does not cover
- **Baseline rules** — the minimum bar all Discovery items must meet
- **Open questions** — what the standard owner needs to decide before raising the bar

The expectation is that the standard owner reviews the starter, customises it to the portfolio's actual context, and uses `/pom-decision-log` to record material decisions over time.

## Files in this pack

| File | Concern | Bootstrap target | When to install |
|------|---------|------------------|-----------------|
| `ai-ethics.md` | AI ethics | `enabling/ai-ethics/README.md` | Any portfolio building anything ML-adjacent. Pairs with the richer scaffold in `methodology/templates/enabling-standards/ai-ethics/`. |
| `security-baseline.md` | Security | `enabling/security/README.md` | Always — every portfolio handles data of some kind. |
| `accessibility-baseline.md` | Accessibility | `enabling/accessibility/README.md` | Any portfolio with a user interface (web, mobile, internal tools). |
| `data-quality.md` | Data quality | `enabling/data-quality/README.md` | Any portfolio with reporting, analytics, ML, or data exchange with other systems. |

## Industry overlays

Industry plugins (`pom-insurance`, `pom-fintech`, `pom-healthcare`, `pom-retail`) ship their own enabling standards that **extend, not replace**, this generic baseline. For example, `pom-healthcare` adds `enabling/clinical-safety/README.md` and `enabling/phi-handling/README.md`, while leaving the generic `security/` and `accessibility/` baselines in place.

## See also

- `methodology/templates/enabling-standards/` — the canonical scaffold (heavier, fully fleshed for ai-ethics). Lives in its own namespace so `pom-bootstrap` cannot accidentally copy it; only `pom-seed-enabling-standard` reads from here.
- `methodology/references/four-question-gate.md` — explains how Q4 reads enabling standards
- `pom-seed-enabling-standard` — skill that lays a new enabling concern into a user's POM repo
- `pom-decision-log` — append-only log for precedent-setting decisions on each concern
