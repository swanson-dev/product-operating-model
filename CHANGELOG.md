# Changelog

All notable changes to the **product-operating-model** marketplace are recorded here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versions follow [SemVer](https://semver.org/).

## [Unreleased] — v0.3 in progress

### Added
- **`/pom-council` skill — parallel multi-role consultation on draft Discovery items.** Dispatches `pom-trio` for each role (default: PM/PD/TL; opt-in to add SM/PO via `--roles=all` or explicit list) in parallel, all under the new `council_mode: true` input contract that returns structured JSON (per-question confidence high/medium/low + ≤750-char rationale + optional open question). Then dispatches the new `pom-council-synthesizer` agent, which is guardrailed against softening disagreements and forbidden from emitting verdict tokens (✅/🟡/❌/"should promote"/etc.). Output is a single timestamped markdown file `<disc-slug>.council-<ISO-ts>.md` co-located with the parent DISC, containing a Tension Map table (per-question role confidences side-by-side, outlier auto-bolded), synthesis prose, per-role detail in canonical order, and a paste-ready stakeholder block. Multiple runs stacked append-only; the orchestrator handles same-second collisions by appending `-2`, `-3` to the timestamp suffix. Strictly consultative — Council never grades, gates, or emits work-item-shaped artifacts. Test contract in `tests/council.test.md` (15 scenarios) plus the spec at `docs/superpowers/specs/2026-05-19-pom-council-design.md`.
- **Validator R3.11 (ERROR) and R3.12 (WARN).** R3.11 enforces Council filename pattern + required frontmatter (`id`, `disc`, `ran_at`, `roles`, `synthesizer`, `overlay`, `tensions_flagged`, `low_confidence_count`); R3.12 emits a WARN when a 4/4 ✅ DISC has zero Council siblings, nudging trios toward Council without blocking promotion. Ruleset bumped from v0.1.0 to v0.2.0. Test contract extended in `tests/validate.test.md`.
- **`pom-trio` agent council mode.** Adds a `council_mode: true` input flag that switches the agent from markdown review output to structured JSON suitable for the Council orchestrator. Existing markdown-review behaviour is unchanged for callers that do not set the flag.

### Rationale
v0.2 closed the visibility loop (cockpit, paste-ready blocks, ADO sync). v0.3 closes the **decision-quality** loop. Trio meetings today are "read the DISC out loud → react"; Council pre-stages cross-role disagreements so meetings start where the friction actually is. The strictly-consultative posture protects existing gate authority — `/pom-discovery-gate` remains the sole grader, and `/pom-promote-to-backlog` remains the sole promoter.

## [Unreleased] — v0.2 in progress

### Added
- **Paste-ready stakeholder summary in every artifact-producing skill.** `pom-discovery-gate`, `pom-runway-add-adr`, and `pom-decision-log` now generate a dashed-line-framed paragraph in the CLI report — matching the existing pattern in `pom-disposition` and `pom-promote-to-backlog`. The block is for Teams/email/PR comments; it lives only in the report, never in the artifact file. Test contracts (`*.test.md`) updated with the assertion in each affected scenario.
- **`pom-explain` skill.** Single-line live summary of any POM artifact by ID (UC, DISP, DISC, PB, ADR, DEC). Read-only. Output is one paste-ready line — type, status, owners, age, and the next blocker if any. Resolves IDs to files via canonical-location globs; handles `ADR-NNNN` ambiguity across products with one line per match. Test contract in `tests/explain.test.md` (7 scenarios).
- **`pom-status --since=<date|last>` diff mode.** Adds a `CHANGES SINCE` section showing promotions, new dispositions/ADRs/decisions, Discovery gate advances, and Trio/Pod changes between a baseline snapshot and now. Writes a fresh snapshot to `.pom/snapshots/<ISO-timestamp>.json` on every `--since` run. Snapshots are pretty-printed JSON, committed to git intentionally — they are the portfolio's heartbeat. The `pom-status` read-only contract now scopes to portfolio artifacts only; `.pom/` metadata is the sole allowed write. Test contract extended with 3 new scenarios (E/F/G).
- **`pom-pulse` skill.** Paste-ready Teams/email digest of stuck artifacts past their SLAs — UCs past disposition, DISCs aging at gate, PBs shaped-but-not-pulled, Proposed ADRs, Trio gaps blocking active Discovery. Default SLAs (7/14/14/7/30 days) overridable via flags. Emoji-bolded sections, bullet lists only (no tables — Teams renders them poorly). Empty sections omitted. Trend line ("N carried over from <baseline date>") appears when `.pom/snapshots/` has prior data. Read-only on artifacts. Test contract in `tests/pulse.test.md` (5 scenarios).
- **PB → ADO branch / PR engineering affordances.** `pom-promote-to-backlog` now emits (in the CLI report only — never written to disk) an ADO-compatible branch name `pb/PB-YYYY-MM-<slug>`, a PR title `PB-YYYY-MM-<slug>: <outcome>`, and a markdown PR description body referencing the originating DISC, vertical slice, acceptance signals (as checkboxes), out-of-scope list, and linked runway/ADRs. Threads the PB-ID through every commit so `git log` traces back to strategic decisions. Sets up the field surface that `pom-sync-ado` will consume in the next task. Test contract extended in `tests/promote-to-backlog.test.md`.
- **`pom-render` skill — Live Portfolio Lens cockpit.** Generates a single self-contained `portfolio.html` at the POM repo root from the markdown corpus. Sections: header, funnel (UCs → DISPs → DISCs → PBs → Pods), products grid, aging heatmap, four-question gate radars per Discovery item, WSJF distribution, Trio assignment matrix, recent activity stream, open blockers, and an industry-overlay panel when an overlay is detected (insurance: state-filing deadlines, actuarial review, regulatory reporting risk). No external CSS/JS/CDN — fully self-contained. Print-friendly via `@media print`. Light + dark color schemes via `prefers-color-scheme`. WCAG AA contrast. ≤ 250 KB. Test contract in `tests/render.test.md` (5 scenarios including overlay detection and print preview).
- **`pom-sync-ado` skill — bi-directional Azure DevOps Boards sync.** POM is authoritative for slice/acceptance/outcome; ADO is authoritative for state/dates. Push creates ADO work items with PB-ID in title, slice contract in description, acceptance signals as Acceptance Criteria checklist, tags including the PB-ID and product slug. Pull maps ADO state → POM `status` (Active→in-flight, Closed→shipped, etc.) and writes `started`/`completed` dates. Diff-warn surfaces title drift, scope edits, and closed-but-shaped reconciliation cases without auto-overwriting. `AZURE_DEVOPS_PAT` env var required; never written to disk. `.pom/sync-ado.config.json` holds org/project/area (no credentials). `--dry-run` makes zero API calls and zero file edits. `--push`, `--pull`, `--product=<slug>` flags supported. Gate enforcement carries through (refuses to push PBs whose DISC was not 4/4 ✅). Test contract in `tests/sync-ado.test.md` (7 scenarios including credential safety, dry-run, drift, and config bootstrap).
- **`.gitignore` belt-and-suspenders.** Added patterns under `.pom/` blocking accidental commits of `*.pat`, `*.secret`, `credentials*`, and `.env*` files. The PAT contract is env-var-only; these patterns catch human mistakes.

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
