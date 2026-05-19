---
name: pom-healthcare-init
description: "Apply the healthcare industry overlay to an existing POM repo. Copies the healthcare WSJF rubric, Q4 framing, starter enabling standards (PHI handling, clinical safety, interoperability, AI-ethics-healthcare), and a worked example. Idempotent — prompts before overwriting user-edited content."
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
Install the healthcare overlay into an existing POM repo.

**Read-from:** `${CLAUDE_PLUGIN_ROOT}/overlay/`
**Writes-to:** the user's POM repo's `intake/scoring/`, `enabling/`, `methodology/industries/healthcare/`.

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
Resolve `<pom-repo-path>` (the required positional argument).

To verify it's a POM repo, use the **Glob tool** (not Bash conditionals — Bash on Windows runs Git Bash POSIX and PowerShell-style `Test-Path { }` syntax will fail). Run these Glob calls:
- `<pom-repo-path>/intake/` — must match
- `<pom-repo-path>/products/` — must match
- `<pom-repo-path>/methodology/` — must match

If any does not match, abort with: `Not a POM repo. Run /pom-bootstrap first.`
</step>

<step name="run_interview">
Walk the user through `${CLAUDE_PLUGIN_ROOT}/overlay/interview.md` via AskUserQuestion. Append answers to `<pom-repo-path>/intake/scoring/healthcare-calibration.md`.
</step>

<step name="install_rubric">
Copy `${CLAUDE_PLUGIN_ROOT}/overlay/rubric.md` to `<pom-repo-path>/intake/scoring/wsjf-rubric.md` with merge prompts per the standard overlay-install pattern (prompt if user-edited).
</step>

<step name="install_q4_framing">
Write `${CLAUDE_PLUGIN_ROOT}/overlay/discovery-q4-framing.md` to `<pom-repo-path>/methodology/industries/healthcare/discovery-q4-framing.md`.
</step>

<step name="install_enabling_standards">
Copy each starter standard from `${CLAUDE_PLUGIN_ROOT}/overlay/enabling-standards/` to `<pom-repo-path>/enabling/<concern>/README.md`:
- `phi-handling.md` → `enabling/phi-handling/`
- `clinical-safety.md` → `enabling/clinical-safety/`
- `interoperability.md` → `enabling/interoperability/`
- `ai-ethics-healthcare.md` → `enabling/ai-ethics-healthcare/` (companion to the generic `enabling/ai-ethics/`)

Create empty `decision-log/` for each new concern. Prompt before overwriting existing.
</step>

<step name="install_example">
Write `${CLAUDE_PLUGIN_ROOT}/overlay/examples/UC-sample.md` to `<pom-repo-path>/methodology/industries/healthcare/examples/UC-sample.md`. Skip if present.
</step>

<step name="report">
Print a summary; recommend `/pom-validate <pom-repo-path>`.
</step>

</process>

<guardrails>
- NEVER silently overwrite user-edited files.
- Idempotent — re-runs are no-ops absent user edits.
- The `enabling/ai-ethics/` directory installed by pom-core remains in place; this overlay adds the `ai-ethics-healthcare` companion concern.
</guardrails>
