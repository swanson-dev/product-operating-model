# Changelog

All notable changes to the **product-operating-model** marketplace are recorded here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versions follow [SemVer](https://semver.org/).

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

### Known limitations / open questions
- Plugin auto-discovery of `skills/` directory has been validated for `pom-core` via the canary test; not yet smoke-tested for the industry overlay plugins individually. Phase E smoke tests cover this.
- Industry init skills (`pom-<industry>-init`) describe merge-prompt semantics; the prompt logic is invoked at runtime by Claude Code and not unit-tested in this release.
- See `plugins/pom-core/README.md` "Architectural notes" for the read-vs-write skill split and append-only semantics.
