# AI Ethics Decision Log

Append-only log of standards-level decisions — exceptions, precedents, threshold-setting, conflicts with authoritative policy. Future use cases inherit these decisions as precedents.

## Why this exists

Standards rot when teams make exceptions silently. An append-only log means the *third* AI use case inherits not just the rules but the **precedents**: "we accepted a 6pp accuracy gap on cohort X for UC-0002 because Y; future UCs use that as a comparable, not a green light."

## File format

`DEC-YYYY-MM-DD-<short-slug>.md` — one decision per file.

Each file contains:

- **Decision ID** _(matches filename)_
- **Date**
- **Deciders** _(Trio members, Compliance, Risk, Enablement reps)_
- **Context** — what was being decided and why
- **Decision** — exactly what was decided
- **Rationale** — why; what alternatives were considered
- **Precedent scope** — UC-only, product-only, or portfolio-wide
- **Expiration / re-review** — when this decision needs to be revisited

## Rules

1. **Append-only.** Never edit a past decision. To reverse one, write a new decision that supersedes it.
2. **No silent decisions.** Anything the Trio decides about PII handling, bias audits, or Model Cards that **deviates from this folder's defaults** must be logged here.
3. **Read this folder before scoring a new AI/ML use case.** Precedents shape what's acceptable.
