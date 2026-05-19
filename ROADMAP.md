# ROADMAP

Forward-looking view of where this marketplace is going. For *what shipped*, see [CHANGELOG.md](./CHANGELOG.md). For the durable vision and scope, see [PROJECT.md](./PROJECT.md).

This file is intentionally short — it covers the current release, the in-flight version, and the next one or two themes. We don't pre-commit to features more than one minor version ahead.

## Released

| Version | Date | Theme |
|---------|------|-------|
| [v0.1.0](https://github.com/swanson-dev/product-operating-model/releases/tag/v0.1.0) | 2026-05-18 | Initial release — marketplace + 5 plugins, core methodology, 15 skills, 7 agents, 4 orchestrators, ~30-rule validator |
| [v0.2.0](https://github.com/swanson-dev/product-operating-model/releases/tag/v0.2.0) | 2026-05-19 | Visibility loop — `pom-render` cockpit, `pom-explain`, `pom-pulse`, `pom-status --since` diffs, `pom-sync-ado`, paste-ready stakeholder summaries across artifact-producing skills |

## In flight — v0.3

**Theme: closing the decision-quality loop.** v0.2 made decisions visible; v0.3 makes them better.

- `pom-council` — parallel multi-role consultation on draft Discovery items. Dispatches `pom-trio` per role under a structured `council_mode`, then `pom-council-synthesizer` produces a Tension Map without softening disagreements. Strictly consultative — never grades, never gates.
- Validator rules **R3.11** (ERROR: Council file structure) and **R3.12** (WARN: 4/4 ✅ DISC with zero Council siblings). Ruleset bumped to v0.2.0.
- `pom-trio` `council_mode: true` input flag — structured JSON output for orchestrator consumption.

See [`docs/superpowers/specs/2026-05-19-pom-council-design.md`](./docs/superpowers/specs/2026-05-19-pom-council-design.md) and [`docs/superpowers/plans/2026-05-19-pom-council.md`](./docs/superpowers/plans/2026-05-19-pom-council.md). Status: merged to `main`, untagged.

## Candidate themes — v0.4 and beyond

These are *candidate* directions, not commitments. They move into "In flight" once a spec lands in `docs/superpowers/specs/`. Listed in rough priority order based on the project rationale; the maintainer makes the call.

- **Dogfood the insurance overlay on a live portfolio.** The insurance plugin shipped in v0.1.0 and ADO sync shipped in v0.2.0; v0.4 would run both against a real product team and feed surprises back as fixes. The first overlay to graduate from "shipped" to "validated in production."
- **Tracker integration parity.** ADO Boards sync exists; Jira and Linear are obvious next targets given the audience. Same asymmetric contract (POM owns slice / acceptance / outcome; tracker owns state / dates).
- **Pod health beyond composition.** The 2–7 rule is structural. Pod health metrics — WIP, cycle time, runway burn rate, ADR cadence — would round out the pod-level operating loop.
- **AI-assisted Q1–Q4 drafting.** Today the Discovery gate is human-graded with optional Council pre-staging. A pre-fill assistant that drafts each answer from the underlying DISC / UC context (still graded by a human) would cut Discovery cycle time.
- **Additional industry overlays.** Energy / utilities, public sector, education — driven by demand, not pre-emption.

> The strategic call on v0.4 scope is the maintainer's. To lock in a theme, replace the "Candidate themes" section with an "In flight — v0.4" section once a spec exists.

## How this file is maintained

- Update on every release: move the in-flight section to "Released", promote a candidate theme to "In flight".
- Keep it short. If it grows past one screen, the candidate list is too speculative — prune.
- Don't duplicate CHANGELOG. The CHANGELOG owns *what shipped in detail*; ROADMAP owns *the one-line theme and the link*.
