---
name: pom-status
description: "Use when a POM portfolio's current state needs a human-readable summary — UCs in intake, products and Trio status, Discovery/Product Backlog depth, pod count, recent dispositions/decisions/ADRs, open blockers. Read-only. Plaintext output, copy-pasteable into stakeholder updates."
argument-hint: "[target-dir] [--product=<slug>] [--recent=<N>]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
---

<objective>
Produce a human-readable portfolio summary of a POM repo. Read-only.

**Output sections (full mode):**
- Intake (UC counts by status)
- Products table (per-product Trio status, Discovery count, PB count, Pod count)
- Recent dispositions (last N)
- Recent decisions across enabling concerns (last N)
- Recent ADRs across all products (last N)
- Platform services (by area, with status)
- **Open blockers** — surfaced prominently

**Product-scoped mode (`--product=<slug>`):**
- Trio (named members + unassigned flagged)
- Roadmap state (Now/Next/Later counts)
- Discovery backlog by gate state (0/4, 1/4, 2/4, 3/4, 4/4-READY)
- Product backlog by state (shaped / pulled / in-flight / shipped)
- Runway (ADR count, patterns count, runway-plan freshness)
- Pods (composition summary per pod)
- Product-scoped open blockers

**Format:** plain text, < 80 columns, copy-pasteable into stakeholder updates. No ANSI color, no HTML.

**Read-only.** Never modifies the target repo.

**After this command:** Use the output as a status update. For structural integrity, run `/pom-validate`.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/status.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
</execution_context>

<process>
Execute end-to-end.
Read-only. Output is the report.
Counts derived from filesystem; blockers correspond to real states.
</process>
