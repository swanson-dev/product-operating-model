# Validation Rules — Source of Truth for `pom-validate`

`pom-validate` reads this file at runtime and enforces every rule below against a target POM repo. **This file is the only source of truth for validation.** If a rule isn't here, `pom-validate` does not enforce it.

Severity levels:

- **ERROR** — must fix. `pom-validate` exits non-zero.
- **WARN** — should fix. Reported but does not fail validation.
- **INFO** — nice to have. Reported only with `--verbose`.

## R1 — Repo structure

| ID | Rule | Severity |
|---|---|---|
| R1.1 | Top-level dirs must exist: `intake/`, `products/`, `platform/`, `enabling/`, `methodology/`, `use-cases/` | ERROR |
| R1.2 | Top-level `README.md` must exist | ERROR |
| R1.3 | `methodology/` should contain at least one reference asset (PDF, image, or markdown) | WARN |

## R2 — Intake

| ID | Rule | Severity |
|---|---|---|
| R2.1 | `intake/README.md` must exist | ERROR |
| R2.2 | `intake/scoring/wsjf-rubric.md` must exist | ERROR |
| R2.3 | `intake/use-case-backlog/README.md` must exist | ERROR |
| R2.4 | `intake/dispositions/README.md` must exist | ERROR |
| R2.5 | Use Case files match `UC-NNNN-<slug>.md` (NNNN zero-padded, slug kebab-case) | ERROR |
| R2.6 | Each UC file contains required fields: ID, Submitted, Status, Source, WSJF scores, Disposition link | ERROR |
| R2.7 | Disposition files match `DISP-YYYY-MM-DD-UC-NNNN.md` | ERROR |
| R2.8 | Each disposition file references an existing UC file | ERROR |
| R2.9 | Disposition files have not been modified since creation (best-effort via git log) | WARN |
| R2.10 | The disposition decision matches one of: kill / park / route / split | ERROR |

## R3 — Products

| ID | Rule | Severity |
|---|---|---|
| R3.1 | `products/README.md` exists | ERROR |
| R3.2 | `products/_template/` exists with all template subfolders | ERROR |
| R3.3 | Each non-template product directory contains: `product.md`, `trio.md`, `roadmap.md`, `discovery-backlog/`, `product-backlog/`, `runway/`, `pods/` | ERROR |
| R3.4 | `product.md` contains: Vision, Target customer, Outcomes, In scope, Out of scope, Status (with Created date) | ERROR |
| R3.5 | `trio.md` lists the three Trio roles (PM, PD, TL). Unassigned roles are flagged as blocking risks. | WARN if any role unassigned |
| R3.6 | Discovery files match `DISC-YYYY-MM-<slug>.md` | ERROR |
| R3.7 | Each Discovery file references an originating UC file | ERROR |
| R3.8 | Product Backlog files match `PB-YYYY-MM-<slug>.md` | ERROR |
| R3.9 | Each pod directory (excluding `_template/`) contains `pod.md` | ERROR |
| R3.10 | Pod composition shows 2–7 members + SM + PO | WARN if outside range |
| R3.11 | Council files match `<disc-slug>.council-YYYY-MM-DDTHH-MM-SSZ(-N)?.md` AND contain required frontmatter (`id`, `disc`, `ran_at`, `roles`, `synthesizer`, `tensions_flagged`, `low_confidence_count`). The `disc` field must point to an existing DISC in the same directory. | ERROR |
| R3.12 | A DISC with all four Promotion Readiness questions ✅ AND zero `<disc-slug>.council-*.md` siblings in the same directory should consider running `/pom-council` before promotion. | WARN |

## R4 — Runway

| ID | Rule | Severity |
|---|---|---|
| R4.1 | ADR files match `ADR-NNNN-<slug>.md` (NNNN zero-padded, sequential within product) | ERROR |
| R4.2 | ADR numbering is contiguous (no gaps within a product's ADRs) | WARN |
| R4.3 | If `runway-plan.md` exists, it references every ADR in the product's `runway/` | WARN |
| R4.4 | Pattern files match `PATTERN-<slug>.md` | ERROR |

## R5 — Platform

| ID | Rule | Severity |
|---|---|---|
| R5.1 | `platform/README.md` exists | ERROR |
| R5.2 | Each service directory (excluding `_service-template/`) contains `README.md` | ERROR |
| R5.3 | Service `README.md` contains: name, owner, status, products consuming | WARN |

## R6 — Enabling

| ID | Rule | Severity |
|---|---|---|
| R6.1 | `enabling/README.md` exists | ERROR |
| R6.2 | Each enabling concern directory contains `README.md` and `decision-log/` | ERROR |
| R6.3 | Decision log files match `DEC-YYYY-MM-DD-<slug>.md` | ERROR |
| R6.4 | Decision log files have not been modified since creation (best-effort via git log) | WARN |

## R7 — Cross-references

| ID | Rule | Severity |
|---|---|---|
| R7.1 | Every UC with a non-empty Disposition field links to an existing disposition file | ERROR |
| R7.2 | Routed UCs (disposition = route) have a corresponding Discovery file in the receiving product | ERROR |
| R7.3 | Every Discovery file references its originating UC | ERROR |
| R7.4 | Promoted Discovery items (4/4 ✅) reference a Product Backlog item | ERROR |
| R7.5 | All relative markdown links resolve to existing files | WARN |

## R8 — WSJF rubric

| ID | Rule | Severity |
|---|---|---|
| R8.1 | `intake/scoring/wsjf-rubric.md` contains the formula and Fibonacci scale references | ERROR |
| R8.2 | Rubric anchors (the 1/2/3/5/8/13/21 rows) are defined for all four dimensions OR the rubric is marked as a scaffold | WARN |

## R9 — Vocabulary

| ID | Rule | Severity |
|---|---|---|
| R9.1 | Product Backlog item files refer to "vertical slice" (not "story" / "epic") | INFO |
| R9.2 | Pod files refer to "Pod" (not "team" / "squad") | INFO |

## Overrides

A repo may override severity levels via a `.pom-validate.yaml` file in the repo root. Format TBD; out of scope for Phase 1.

## Versioning

This rule-set is **v0.2.0**. Material changes require a CHANGELOG entry and a version bump. Skill `pom-validate` reports the rule-set version it ran against.

### v0.2.0 changes

- Added R3.11 (ERROR): Council file structure and frontmatter requirements.
- Added R3.12 (WARN): Council-recommended nudge for 4/4 ✅ DISCs without a Council run.
