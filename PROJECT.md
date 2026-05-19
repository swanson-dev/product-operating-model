# PROJECT — Product Operating Model marketplace

The durable view of this repo: what it is, who it's for, the principles that govern changes, and how releases are run. For *what shipped*, see [CHANGELOG.md](./CHANGELOG.md). For *what's next*, see [ROADMAP.md](./ROADMAP.md).

## Vision

A Claude Code marketplace that operationalises the **Product Operating Model** (Marty Cagan / SVPG / Voyager lineage) as executable, append-only markdown — so a product organisation can *operate* the model from day one instead of diagramming it for six months.

Success is measured by whether a team can:

1. Capture stakeholder inputs as scored Use Cases without a separate SaaS.
2. Make kill / park / route / promote decisions that survive an audit twelve months later.
3. Walk Discovery items through the four-question gate (desirable / viable / feasible / ethical) with the gate actually enforced.
4. Ship vertical slices through pods whose composition, runway, and decisions are all traceable to a single repo.

## Scope

**In scope.** The methodology surface a CPO / product-ops leader needs to *run* the operating model:

- Intake → WSJF scoring → disposition → routing → Discovery → backlog → pod
- ADR-driven product runway and runway plans
- Platform-service contracts (intake + consumption)
- Enabling concerns (AI ethics, security, accessibility, data quality) with industry overlays
- Read-only views: `pom-status`, `pom-pulse`, `pom-explain`, `pom-render` (HTML cockpit)
- Bi-directional sync to engineering trackers — Azure DevOps Boards today; others as demand surfaces
- Structural validation ruleset (~30 rules, runs in seconds)

**Out of scope.** Things deliberately not solved here, because doing them well requires a different surface:

- Roadmapping tools, Gantt charts, capacity planning calculators
- Sprint mechanics (standups, retros, story-point estimation, velocity)
- Anything that requires a database or background service — the whole methodology is plain markdown for a reason
- Replacing the existing engineering tracker — POM is upstream of Jira / ADO / Linear, never a substitute

## Audiences

Listed in the [README.md](./README.md) "Who this is for" section. Briefly: CPOs and product-ops leaders, product trios, portfolio / platform leaders, regulated-industry teams (insurance / fintech / healthcare / retail), and solo founders who want the discipline without a transformation consultancy.

## Design principles

The non-negotiables that shape every PR.

- **Append-only history.** UCs, dispositions, DISC shaping logs, ADRs, and decision-logs are immutable. Reversals create new artifacts that supersede; nothing is silently rewritten. This is the audit trail.
- **Gates that actually gate.** `pom-promote-to-backlog` refuses 4/4 ✅ failures; `pom-form-pod` enforces 2–7 composition; `pom-ingest-use-case` refuses unscored UCs. Methodology is enforced in code, not vibes.
- **Read-only views are read-only.** `pom-status`, `pom-pulse`, `pom-explain`, `pom-render`, `pom-validate` never touch portfolio artifacts. The single sanctioned exception is `.pom/snapshots/` and `.pom/sync-ado.config.json` metadata.
- **Strict consultative-vs-authoritative separation.** `pom-trio` and `pom-council` advise; only `pom-discovery-gate` grades; only `pom-promote-to-backlog` promotes. Advisory tools never emit verdict tokens.
- **Industry overlays are additive content, not replacement code.** A healthcare portfolio uses the generic ai-ethics standard *plus* the healthcare AI-ethics overlay — never one in place of the other.
- **Pure markdown, no runtime.** No compiled binaries, no databases, no scripts that require a runtime beyond Claude Code's Bash / Read / Write. `${CLAUDE_PLUGIN_ROOT}` is the only path indirection.
- **Plays well with engineering trackers.** POM is upstream of Jira / ADO / Linear. The sync direction is asymmetric on purpose: POM owns slice / acceptance / outcome; the tracker owns state / dates.
- **Surgical changes per PR.** Every changed line traces back to the user-facing rationale in the CHANGELOG entry that ships it.

## Versioning and releases

- **Git tags are the release contract.** `v0.1.0`, `v0.2.0`, etc. mark marketplace-wide releases — what users install and reference. CI and install consumers should pin to tags, not branches.
- **Two version surfaces, not always in lockstep.** Each `plugins/*/.claude-plugin/plugin.json` and the top-level `marketplace.json` carry a `version` field that bumps when *that surface* changes meaningfully. These per-plugin versions can lag behind marketplace tags when a tagged release's changes are confined to other plugins — the manifest version field is per-plugin SemVer, not a mirror of the marketplace tag.
- **One CHANGELOG, sectioned by version.** Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). `[Unreleased]` rolls forward; tagged versions become headed sections with ISO dates. The CHANGELOG is the single source of truth for what's in each release, across both version surfaces.
- **Backwards compatibility for one minor version per plugin.** Breaking changes get a major bump on the affected plugin manifest and a migration note in CHANGELOG.

## Repository layout

```
.
├── .claude-plugin/marketplace.json    marketplace manifest (5 plugins)
├── README.md                          user-facing entry point
├── PROJECT.md                         this file (durable vision/scope/principles)
├── ROADMAP.md                         forward-looking version themes
├── CHANGELOG.md                       what shipped, per version
├── LICENSE                            MIT
├── docs/superpowers/                  design specs and implementation plans per feature
└── plugins/
    ├── pom-core/                      required plugin (skills, agents, orchestrator commands)
    ├── pom-insurance/                 industry overlay
    ├── pom-fintech/                   industry overlay
    ├── pom-healthcare/                industry overlay
    └── pom-retail/                    industry overlay
```

## Working agreements

- **PRs are atomic and CHANGELOG-first.** A PR ships one feature, one CHANGELOG entry, and the matching test contract — or it doesn't ship.
- **Specs before plans, plans before code.** Non-trivial features land a design spec in `docs/superpowers/specs/` and an implementation plan in `docs/superpowers/plans/` before the feature PR opens. `pom-council` is the canonical example.
- **Test contracts are specs, not coverage.** Each skill has a `*.test.md` enumerating scenarios with expected behaviour. They describe *what the skill must do*, executed by reading + reasoning, not a runner.
- **Smoke-test gaps become CHANGELOG entries.** When pre-tag smoke tests surface bugs, the fixes are listed under "Fixed during smoke testing" in the same release section — see v0.1.0 for the canonical pattern.
- **Never push to `main`.** All work lands via PR. The maintainer merges.

## License

MIT — see [LICENSE](./LICENSE). The methodology references (four-question gate, WSJF, vertical slicing) draw on widely-published industry practice; this packaging is original.
