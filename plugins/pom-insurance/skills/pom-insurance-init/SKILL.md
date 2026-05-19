---
name: pom-insurance-init
description: "Apply the insurance industry overlay to an existing POM repo. Copies the insurance WSJF rubric, Q4 framing, starter enabling standards, and a worked example into the repo with merge prompts on conflict. Idempotent — re-running prompts before overwriting any user-edited content."
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
Install the insurance overlay shipped with this plugin into an existing POM repo. The overlay layers regulatory Q4 framing, an insurance-calibrated WSJF rubric, starter enabling standards (regulatory reporting, actuarial review, policyholder communications, state filings), and a worked example.

**Read-from:** `${CLAUDE_PLUGIN_ROOT}/overlay/` (this plugin's bundled overlay content)
**Writes-to:** the user's POM repo's `intake/scoring/`, `enabling/`, and `methodology/industries/insurance/` if present.

**Idempotent:** if any target file already exists with different content from the source, prompts the user (merge / overwrite / skip). Never silently overwrites user-edited content.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/overlay/rubric.md
@${CLAUDE_PLUGIN_ROOT}/overlay/discovery-q4-framing.md
@${CLAUDE_PLUGIN_ROOT}/overlay/interview.md
@${CLAUDE_PLUGIN_ROOT}/overlay/enabling-standards/README.md
</execution_context>

<process>

<step name="pre_flight">
1. Resolve `<pom-repo-path>` (required positional arg). Abort if not provided.
2. Verify the path looks like a POM repo using the **Glob tool** for each required directory (do NOT use Bash conditionals — Bash on Windows runs Git Bash POSIX and PowerShell-style `Test-Path { }` syntax will fail). Glob:
   - `<pom-repo-path>/intake/`
   - `<pom-repo-path>/products/`
   - `<pom-repo-path>/methodology/`
   If any does not match, abort with `Not a POM repo. Run /pom-bootstrap first.`
3. Note whether the generic baseline is currently installed (`intake/scoring/wsjf-rubric.md` exists; `methodology/generic/` may also be present if bootstrap copied it) — check via Glob, same reason.
</step>

<step name="run_interview">
Read `${CLAUDE_PLUGIN_ROOT}/overlay/interview.md`. Walk the user through each question via AskUserQuestion. The interview asks about:
- Lines of business (life, P&C, health, specialty)
- Regulated markets (which states; international)
- Distribution model (direct, agent, broker, embedded)
- Reinsurance involvement
- Existing regulatory deadlines on the horizon

Save answers to `<pom-repo-path>/intake/scoring/insurance-calibration.md` (appended if file already exists; never overwrite).
</step>

<step name="install_rubric">
Read `${CLAUDE_PLUGIN_ROOT}/overlay/rubric.md`. The target is `<pom-repo-path>/intake/scoring/wsjf-rubric.md`.

If the target doesn't exist: write the rubric directly.

If it exists and matches the generic rubric content: confirm with user, then replace with the insurance rubric.

If it exists and is user-edited (diff against generic shows changes): present the user with three options via AskUserQuestion — (a) overlay insurance anchors but keep your edits, (b) overwrite with insurance rubric, (c) save insurance rubric as `wsjf-rubric-insurance.md` alongside and leave the current rubric untouched.
</step>

<step name="install_q4_framing">
Read `${CLAUDE_PLUGIN_ROOT}/overlay/discovery-q4-framing.md`. Write to `<pom-repo-path>/methodology/industries/insurance/discovery-q4-framing.md` (creating the directory if needed). This sits alongside the generic Q4 framing — both are read by `pom-discovery-gate` and `pom-ethics-reviewer` when this overlay is active.

Note: the methodology/ subtree inside the user's repo is a local read-only mirror; the plugin's bundled methodology/ stays canonical. We write a copy here so that the user can audit what their portfolio is grading against, and so an industry overlay can be removed by deleting the directory.
</step>

<step name="install_enabling_standards">
For each standard file in `${CLAUDE_PLUGIN_ROOT}/overlay/enabling-standards/` (excluding README.md):
- Determine target concern (e.g., `regulatory-reporting.md` → `<pom-repo-path>/enabling/regulatory-reporting/README.md`)
- If the concern directory doesn't exist: create it, write the starter standard as README.md, create empty `decision-log/` with a starter README.
- If the concern directory exists with a README.md from a prior bootstrap or seed: prompt the user (merge / overwrite / skip).

Always create `<pom-repo-path>/enabling/<concern>/decision-log/README.md` if missing, so `/pom-decision-log` can append to it immediately.
</step>

<step name="install_example">
Read `${CLAUDE_PLUGIN_ROOT}/overlay/examples/UC-sample.md`. Write to `<pom-repo-path>/methodology/industries/insurance/examples/UC-sample.md`. Skipped if already present.
</step>

<step name="report">
Print a summary:

```
pom-insurance overlay installed at: <pom-repo-path>

Files written / updated:
  ✓ intake/scoring/wsjf-rubric.md
  ✓ intake/scoring/insurance-calibration.md (interview record)
  ✓ methodology/industries/insurance/discovery-q4-framing.md
  ✓ methodology/industries/insurance/examples/UC-sample.md
  ✓ enabling/regulatory-reporting/README.md
  ✓ enabling/actuarial-review/README.md
  ✓ enabling/policyholder-communications/README.md
  ✓ enabling/state-filings/README.md
  (decision-logs created for each)

Recommended next: /pom-validate <pom-repo-path>
```
</step>

</process>

<guardrails>
- NEVER overwrite user-edited content silently. When in doubt, prompt.
- Idempotent — re-running this skill against the same repo should produce no diffs if no user edits have occurred.
- The user's POM repo's `methodology/industries/insurance/` is a local audit copy; the canonical content lives in the plugin's `overlay/`. Deleting the local copy is non-destructive.
- This skill does NOT modify any file under `<pom-repo-path>/methodology/generic/` — the generic baseline stays in place.
</guardrails>
