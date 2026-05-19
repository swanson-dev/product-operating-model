# pom-core

The core plugin for the [Product Operating Model marketplace](../../README.md). Ships everything you need to operate a POM portfolio: intake, four-question Discovery gate, WSJF scoring, product/platform/enabling layers, runway management, and structural validation — plus the **generic industry baseline**.

## Install

```text
/plugin marketplace add swansoncreativestudios/product-operating-model
/plugin install pom-core@product-operating-model
```

Optional industry overlays (install after `pom-core`):
- `pom-insurance` — regulatory Q4 framing, actuarial / state-filing standards
- `pom-fintech` — consumer-protection Q4 framing, PCI / KYC standards
- `pom-healthcare` — HIPAA / clinical-safety Q4 framing, PHI / interoperability standards
- `pom-retail` — PCI / accessibility Q4 framing, omnichannel standards

## Quickstart

```text
# 1. Scaffold a new POM portfolio with the generic industry baseline
/pom-bootstrap /path/to/portfolio --rubric=generic

# 2. Funnel your first stakeholder source end-to-end
/pom-funnel /path/to/portfolio/use-cases/first-memo.pdf

# 3. Periodic health check
/pom-status /path/to/portfolio
/pom-validate /path/to/portfolio
/pom-review                       # full audit
```

## Commands (4 orchestrators)

| Command | What it does |
|---|---|
| `/pom` | Top-level router. Inspects POM repo state and recommends the next step. Read-only. |
| `/pom-funnel <source>` | Runs research → ingest → disposition → route → gate → promote for one or more sources. Stops at every human gate. |
| `/pom-new-product <slug>` | Stands up a new product: spinup-product → form-pod → runway-plan. Each stage opt-in past the first. |
| `/pom-review` | Coordinated review: validate + portfolio-auditor + ethics-reviewer. Produces a stakeholder-ready summary. Read-only. |

## Skills (15)

All skills are directly invocable (`/<skill-name> <args>`).

| Skill | Purpose | Writes? |
|---|---|---|
| `pom-bootstrap` | Scaffold a new POM repo from canonical templates with a WSJF rubric | Yes |
| `pom-ingest-use-case` | Convert a stakeholder doc into a scored Use Case | Yes (intake/) |
| `pom-disposition` | Record kill/park/route/split for a scored UC. Append-only | Yes (intake/dispositions/) |
| `pom-route-to-discovery` | Land a routed UC in a product's Discovery Backlog | Yes (products/<x>/discovery-backlog/) |
| `pom-discovery-gate` | Walk a Discovery item through the four-question gate | Yes (DISC shaping log append) |
| `pom-promote-to-backlog` | Promote a 4/4 ✅ DISC to the Product Backlog | Yes (products/<x>/product-backlog/) |
| `pom-spinup-product` | Stand up a new product from `products/_template/` | Yes (products/<x>/) |
| `pom-form-pod` | Stand up a pod with 2-7 composition rule enforced | Yes (products/<x>/pods/<pod>/) |
| `pom-runway-add-adr` | Record an Architecture Decision Record. Immutable | Yes (products/<x>/runway/) |
| `pom-runway-plan` | Maintain a product's living runway-plan.md | Yes (products/<x>/runway/runway-plan.md) |
| `pom-platform-add-service` | Scaffold a portfolio-shared platform service | Yes (platform/<area>/<service>/) |
| `pom-seed-enabling-standard` | Bootstrap an enabling concern (AI ethics, security, etc.) | Yes (enabling/<concern>/) |
| `pom-decision-log` | Append precedent-setting decisions to an enabling concern. Append-only | Yes (enabling/<concern>/decision-log/) |
| `pom-status` | Read-only portfolio snapshot for stakeholder updates | No |
| `pom-validate` | Read-only structural validation against the v0.1 ruleset | No |

## Agents (7)

Agents are invoked by skills and commands; you don't usually call them directly.

### Workflow agents (3) — heavy lifting, protect main context

| Agent | What it does |
|---|---|
| `pom-uc-researcher` | Converts a raw stakeholder source (PDF/deck/transcript) into a structured pre-UC dossier. Read-only. |
| `pom-wsjf-scorer` | Proposes WSJF scores from a dossier + rubric, with cited anchors and confidence levels. Does NOT write. |
| `pom-validator` | Runs the full validation ruleset in isolation; returns compact JSON findings. |

### Role agent (1) — Trio / pod-leadership framing

| Agent | What it does |
|---|---|
| `pom-trio` | Single agent with role-branching (`role=pm\|pd\|tl\|sm\|po`) — applies one role's lens at a time. Explicitly stays in lane. |

### Audit agents (3) — confirmation-bias mitigation

| Agent | What it does |
|---|---|
| `pom-discovery-auditor` | Independently grades a DISC against the four-question gate, forms verdict before reading original. |
| `pom-portfolio-auditor` | Scans the portfolio for flow-level issues (stuck UCs, Discovery rot, orphan ADRs, …). 12 flow checks. |
| `pom-ethics-reviewer` | Q4-only deep audit. Reads cited evidence independently. Industry-aware. |

## Methodology library

Bundled inside the plugin at `methodology/`:

```
methodology/
├── references/        ← 8 authoritative reference files (validation-rules, four-question-gate, wsjf-formula, …)
├── workflows/         ← 15 per-skill workflow docs (one per skill above)
├── templates/         ← canonical scaffold copied into user POM repos by pom-bootstrap
└── generic/           ← the generic industry baseline (rubric, Q4 framing, interview, enabling-standards, examples)
```

Skills reference these via `${CLAUDE_PLUGIN_ROOT}/methodology/...` paths so they resolve correctly regardless of where Claude Code installs the plugin.

## Tests

`tests/` contains 15 behavioural tests (`*.test.md`) — one per skill. These define the expected behaviour of each skill in given/when/then form and serve as the regression harness.

## Architectural notes

- **Strict separation of read vs write skills.** `pom-status` and `pom-validate` never write. Every other skill that writes does so via clearly-scoped, idempotent operations (append-only logs, single-file creates, in-place updates with confirmation).
- **Append-only semantics on**: `pom-decision-log`, `pom-disposition` (reversals create new DISP files that supersede), `pom-runway-add-adr` (supersession creates new ADR), `pom-discovery-gate` (shaping log).
- **Gate enforcement**: `pom-promote-to-backlog` refuses to promote unless `pom-discovery-gate` shows 4/4 ✅. `pom-ingest-use-case` refuses to write a UC without WSJF scores. `pom-form-pod` enforces 2–7 composition.
- **Agents are opt-in for skills.** A skill can use AskUserQuestion directly or delegate to an agent (workflow agent for heavy reads; audit agent for independence). Both patterns are supported.

## License

MIT — see [../../LICENSE](../../LICENSE).
