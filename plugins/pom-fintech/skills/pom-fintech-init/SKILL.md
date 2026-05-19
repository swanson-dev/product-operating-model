---
name: pom-fintech-init
description: "Apply the fintech industry overlay to an existing POM repo. Copies the fintech WSJF rubric, Q4 framing, starter enabling standards (PCI, KYC/AML, consumer protection, model risk), and a worked example into the repo with merge prompts on conflict. Idempotent."
argument-hint: "<pom-repo-path>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Install the fintech overlay shipped with this plugin into an existing POM repo. Layers regulatory and product-specific framing onto the generic baseline already present in `pom-core`.

**Read-from:** `${CLAUDE_PLUGIN_ROOT}/overlay/`
**Writes-to:** the user's POM repo's `intake/scoring/`, `enabling/`, `methodology/industries/fintech/`.

**Idempotent.** Prompts on conflict; never silently overwrites user-edited content.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/overlay/rubric.md
@${CLAUDE_PLUGIN_ROOT}/overlay/discovery-q4-framing.md
@${CLAUDE_PLUGIN_ROOT}/overlay/interview.md
@${CLAUDE_PLUGIN_ROOT}/overlay/enabling-standards/README.md
</execution_context>

<process>

<step name="pre_flight">
1. Resolve `<pom-repo-path>` (required positional). Abort if missing.
2. Verify POM repo structure via the **Glob tool** (do NOT use Bash conditionals — Bash on Windows is Git Bash POSIX and PowerShell `Test-Path { }` syntax will fail). Glob each:
   - `<pom-repo-path>/intake/`
   - `<pom-repo-path>/products/`
   - `<pom-repo-path>/methodology/`
   If any does not match, abort with `Not a POM repo. Run /pom-bootstrap first.`
</step>

<step name="run_interview">
Read `${CLAUDE_PLUGIN_ROOT}/overlay/interview.md`. Walk the user through each question via AskUserQuestion. Append answers to `<pom-repo-path>/intake/scoring/fintech-calibration.md`.
</step>

<step name="install_rubric">
Read `${CLAUDE_PLUGIN_ROOT}/overlay/rubric.md`. Target: `<pom-repo-path>/intake/scoring/wsjf-rubric.md`. Apply merge-prompt rules (see pom-insurance-init for full pattern — same logic here): if user-edited, ask before overwriting.
</step>

<step name="install_q4_framing">
Write `${CLAUDE_PLUGIN_ROOT}/overlay/discovery-q4-framing.md` to `<pom-repo-path>/methodology/industries/fintech/discovery-q4-framing.md`.
</step>

<step name="install_enabling_standards">
For each starter standard in `${CLAUDE_PLUGIN_ROOT}/overlay/enabling-standards/` (excluding README.md):
- `pci-handling.md` → `<pom-repo-path>/enabling/pci-handling/README.md`
- `kyc-aml-bsa.md` → `<pom-repo-path>/enabling/kyc-aml-bsa/README.md`
- `consumer-protection.md` → `<pom-repo-path>/enabling/consumer-protection/README.md`
- `model-risk.md` → `<pom-repo-path>/enabling/model-risk/README.md`

Create `decision-log/` subdirectory for each concern. Prompt before overwriting existing.
</step>

<step name="install_example">
Write `${CLAUDE_PLUGIN_ROOT}/overlay/examples/UC-sample.md` to `<pom-repo-path>/methodology/industries/fintech/examples/UC-sample.md`. Skip if already present.
</step>

<step name="report">
Print a summary of files written / updated and a recommendation: `/pom-validate <pom-repo-path>`.
</step>

</process>

<guardrails>
- NEVER silently overwrite user-edited files.
- Idempotent — repeated runs produce no diffs unless user has edited the targets.
- Read-only on `<pom-repo-path>/methodology/generic/` — generic baseline stays in place.
</guardrails>
