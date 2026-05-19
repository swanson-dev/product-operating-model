# Changelog

All notable changes to the **product-operating-model** marketplace are recorded here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versions follow [SemVer](https://semver.org/).

## [Unreleased] — v0.2 in progress

### Added
- **Paste-ready stakeholder summary in every artifact-producing skill.** `pom-discovery-gate`, `pom-runway-add-adr`, and `pom-decision-log` now generate a dashed-line-framed paragraph in the CLI report — matching the existing pattern in `pom-disposition` and `pom-promote-to-backlog`. The block is for Teams/email/PR comments; it lives only in the report, never in the artifact file. Test contracts (`*.test.md`) updated with the assertion in each affected scenario.
- **`pom-explain` skill.** Single-line live summary of any POM artifact by ID (UC, DISP, DISC, PB, ADR, DEC). Read-only. Output is one paste-ready line — type, status, owners, age, and the next blocker if any. Resolves IDs to files via canonical-location globs; handles `ADR-NNNN` ambiguity across products with one line per match. Test contract in `tests/explain.test.md` (7 scenarios).
- **`pom-status --since=<date|last>` diff mode.** Adds a `CHANGES SINCE` section showing promotions, new dispositions/ADRs/decisions, Discovery gate advances, and Trio/Pod changes between a baseline snapshot and now. Writes a fresh snapshot to `.pom/snapshots/<ISO-timestamp>.json` on every `--since` run. Snapshots are pretty-printed JSON, committed to git intentionally — they are the portfolio's heartbeat. The `pom-status` read-only contract now scopes to portfolio artifacts only; `.pom/` metadata is the sole allowed write. Test contract extended with 3 new scenarios (E/F/G).

### Rationale
v0.1.0 produced well-structured immutable artifacts but provided no canonical way to communicate each decision back to stakeholders — leaders had to re-summarize from the markdown. The paste-ready block closes that gap with zero new state and zero validator changes.

## [0.1.0] — 2026-05-18

Initial release.

### Marketplace
- Marketplace manifest at `.claude-plugin/marketplace.json` listing 5 plugins.
- Top-level README with install instructions, overview diagram, design principles.
- MIT license, .gitignore, .gitattributes (LF line endings).

### pom-core (the required plugin)
- **15 skills** (`pom-bootstrap`, `pom-ingest-use-case`, `pom-disposition`, `pom-route-to-discovery`, `pom-discovery-gate`, `pom-promote-to-backlog`, `pom-spinup-product`, `pom-form-pod`, `pom-runway-add-adr`, `pom-runway-plan`, `pom-platform-add-service`, `pom-seed-enabling-standard`, `pom-decision-log`, `pom-status`, `pom-validate`).
- **7 agents**: 3 workflow (`pom-uc-researcher`, `pom-wsjf-scorer`, `pom-validator`), 1 combined role (`pom-trio` with role=pm|pd|tl|sm|po), 3 audit (`pom-discovery-auditor`, `pom-portfolio-auditor`, `pom-ethics-reviewer`).
- **4 orchestrator commands**: `/pom`, `/pom-funnel`, `/pom-new-product`, `/pom-review`.
- **Methodology library**: 8 references, 15 workflows, ~30 templates, generic industry baseline (rubric, Q4 framing, interview, 4 enabling-standards starter, 2 worked examples).
- **15 spec-style tests** in `plugins/pom-core/tests/`.
- All path references migrated from `$HOME/.claude/pom-methodology/` to `${CLAUDE_PLUGIN_ROOT}/methodology/` so installs work on any machine. Canary test (see commit `43c55df`) validated `${CLAUDE_PLUGIN_ROOT}` resolution inside SKILL.md `<execution_context>` `@`-attachments.

### Industry overlay plugins
Each ships: plugin.json, README, single `pom-<industry>-init` skill, overlay/ content (calibrated rubric, industry-specific Q4 framing, bootstrap interview, 4 starter enabling standards, worked example UC).

- **pom-insurance** — Insurance overlay (regulatory reporting, actuarial review, policyholder communications, state filings; Q4 sub-questions I1–I7).
- **pom-fintech** — Fintech overlay (PCI handling, KYC/AML/BSA, consumer protection, model risk; Q4 sub-questions F1–F7).
- **pom-healthcare** — Healthcare overlay (PHI handling, clinical safety, interoperability, AI-ethics-healthcare; Q4 sub-questions H1–H8). Richest content per design.
- **pom-retail** — Retail / e-commerce overlay (PCI-retail, accessibility-regulatory, omnichannel data, fulfillment integrity; Q4 sub-questions R1–R7).

### Fixed during smoke testing (pre-tag)
Five spec-vs-implementation gaps surfaced during Phase E smoke tests and were fixed before tagging:
1. **`pluginRoot` not honored** by Claude Code's installer — switched to inline `./plugins/<name>` source paths in `marketplace.json`.
2. **`repository` field must be a string, not an object** — converted all 5 `plugin.json` files from `{type, url, directory}` to plain URL string.
3. **PowerShell syntax in Bash on Windows** — industry `init` skills now prescribe the `Glob` tool for pre-flight existence checks (cross-shell, no shell at all).
4. **R8.1 / R7.5: rubrics not self-contained** — all 5 industry-style rubrics (generic + 4 industry overlays) now inline the WSJF formula + Fibonacci scale instead of relying on a relative link to `wsjf-formula.md` that bootstrap doesn't create.
5. **GitHub URL ownership** — corrected from `swansoncreativestudios` to `swanson-dev` across all manifests and READMEs.

All five fixes are mechanical / metadata. No skill behaviour changed.

### Known limitations / open questions
- Plugin auto-discovery of `skills/` directory has been validated for `pom-core` (canary) and `pom-healthcare` (smoke test 5). Other industry plugins follow the same shape but are not individually verified.
- Industry init skills describe merge-prompt semantics; the prompt logic is invoked at runtime by Claude Code and not unit-tested in this release.
- See `plugins/pom-core/README.md` "Architectural notes" for the read-vs-write skill split and append-only semantics.
