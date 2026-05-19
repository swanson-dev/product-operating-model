---
name: pom-retail-init
description: "Apply the retail / e-commerce industry overlay to an existing POM repo. Copies the retail WSJF rubric, Q4 framing, starter enabling standards (PCI-retail, accessibility-regulatory, omnichannel data, fulfillment integrity), and a worked example. Idempotent."
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
Install the retail overlay into an existing POM repo.

**Read-from:** `${CLAUDE_PLUGIN_ROOT}/overlay/`
**Writes-to:** the user's POM repo's `intake/scoring/`, `enabling/`, `methodology/industries/retail/`.

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
Resolve `<pom-repo-path>` (required positional argument).

Verify POM repo using the **Glob tool** for each required directory (do NOT use Bash conditionals — Bash on Windows runs Git Bash POSIX and PowerShell `Test-Path { }` syntax will fail). Glob:
- `<pom-repo-path>/intake/`
- `<pom-repo-path>/products/`
- `<pom-repo-path>/methodology/`

If any does not match, abort with `Not a POM repo. Run /pom-bootstrap first.`
</step>

<step name="run_interview">
Walk through `${CLAUDE_PLUGIN_ROOT}/overlay/interview.md`. Append answers to `<pom-repo-path>/intake/scoring/retail-calibration.md`.
</step>

<step name="install_rubric">
Copy `${CLAUDE_PLUGIN_ROOT}/overlay/rubric.md` to `<pom-repo-path>/intake/scoring/wsjf-rubric.md` with the standard merge-prompt pattern.
</step>

<step name="install_q4_framing">
Write `${CLAUDE_PLUGIN_ROOT}/overlay/discovery-q4-framing.md` to `<pom-repo-path>/methodology/industries/retail/discovery-q4-framing.md`.
</step>

<step name="install_enabling_standards">
Copy each starter standard from `${CLAUDE_PLUGIN_ROOT}/overlay/enabling-standards/` to `<pom-repo-path>/enabling/<concern>/README.md`:
- `pci-retail.md` → `enabling/pci-retail/`
- `accessibility-regulatory.md` → `enabling/accessibility-regulatory/` (companion to generic accessibility-baseline)
- `omnichannel-data.md` → `enabling/omnichannel-data/`
- `fulfillment-integrity.md` → `enabling/fulfillment-integrity/`

Create `decision-log/` for each. Prompt before overwriting existing.
</step>

<step name="install_example">
Copy `${CLAUDE_PLUGIN_ROOT}/overlay/examples/UC-sample.md` to `<pom-repo-path>/methodology/industries/retail/examples/UC-sample.md`. Skip if present.
</step>

<step name="report">
Print a summary; recommend `/pom-validate <pom-repo-path>`.
</step>

</process>

<guardrails>
- NEVER silently overwrite user-edited files.
- Idempotent.
- The generic accessibility-baseline from pom-core stays in place; this overlay adds the regulatory layer (ADA, EAA, state-specific obligations) as a companion concern.
</guardrails>
