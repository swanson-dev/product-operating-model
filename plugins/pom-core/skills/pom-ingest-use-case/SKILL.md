---
name: pom-ingest-use-case
description: "Use when a raw stakeholder doc (PDF, deck, markdown) needs to be processed into a scored Use Case Backlog entry. Extracts problem/stakeholders/outcomes, prompts for WSJF scores using rubric anchors, computes WSJF, writes the UC file, recommends a disposition."
argument-hint: "<source-path> [--next-uc=NNNN] [--slug=<slug>]"
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
Process a raw stakeholder doc into a scored Use Case Backlog entry. Reads the source (PDF/markdown/text), extracts problem statement, stakeholders, outcomes, and what's already true. Prompts the user for WSJF scores per dimension using the current rubric's anchors. Computes Cost of Delay and WSJF. Writes the UC file with all required fields per `disposition-rules.md` and `validation-rules.md`.

**Creates:**
- `intake/use-case-backlog/UC-NNNN-<slug>.md` (auto-incrementing NNNN unless overridden)

**Refuses** to invent WSJF scores — always prompts. **Refuses** to write a UC with Job Size = 21 — splits required first.

**After this command:** Run `/pom-disposition <UC-ID> --decision=<kill|park|route|split>` to formally disposition the scored UC.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/ingest-use-case.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/wsjf-formula.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/disposition-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/intake/use-case-backlog/UC-template.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (extraction confirmation, WSJF scoring, slug validation, Job Size = 21 guard).
</process>
