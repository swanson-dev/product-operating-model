---
name: pom-bootstrap
description: "Use when initializing a new Product Operating Model (POM) repository from scratch — scaffolds the intake funnel, product/platform/enabling layers, and a WSJF rubric (interactive or industry preset: insurance, fintech, healthcare, retail, generic)."
argument-hint: "[target-dir] [--rubric=interactive|insurance|fintech|healthcare|retail|generic]"
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
Initialize a new POM repository from canonical templates. Scaffolds intake, products, platform, enabling, methodology, and use-cases directories with all required template files. Sets up the WSJF rubric either interactively (via AskUserQuestion) or from a pre-calibrated industry preset.

**Creates:**
- `<target>/README.md`
- `<target>/intake/` (use-case-backlog, dispositions, scoring/wsjf-rubric.md)
- `<target>/products/_template/` (full subtree)
- `<target>/platform/` (with _service-template)
- `<target>/enabling/README.md` (concerns seeded separately via pom-seed-enabling-standard)
- `<target>/methodology/README.md` (pointer)
- `<target>/use-cases/README.md` (pointer)

**Refuses** to operate on a non-empty target directory.

**After this command:** Run `/pom-validate <target>` to confirm structure, then stakeholders submit raw materials to `<target>/use-cases/` to begin the funnel.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/bootstrap.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/wsjf-formula.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/README.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (precondition checks, rubric calibration, verification).
</process>
