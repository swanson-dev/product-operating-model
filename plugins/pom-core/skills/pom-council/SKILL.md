---
name: pom-council
description: "Use when a draft Discovery item needs cross-role critique before the human Trio meeting. Dispatches PM/PD/TL (default) or all five Trio roles in parallel via pom-trio in council mode, then runs pom-council-synthesizer to narrate the tensions. Writes an append-only Council file co-located with the parent DISC. Strictly consultative — produces no verdict, gates nothing, never mints work items."
argument-hint: "<DISC-ID> [--roles=pm,pd,tl|all] [--dry-run]"
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
  - Agent
---

<objective>
Produce a structured Council artifact for a draft Discovery item by running the existing Trio role-perspective agents (PM, PD, TL, and optionally SM, PO) in parallel and synthesising their per-question confidence (high / medium / low on Q1–Q4) into a single timestamped markdown file that surfaces *where the roles disagree*.

**Council is consultative.** It does not produce a verdict (✅/🟡/❌) on any gate question. It does not gate promotion. Authority to grade a DISC against Q1–Q4 remains exclusively with `/pom-discovery-gate`. Authority to promote remains with `/pom-promote-to-backlog`. Council exists to make the disagreement structure legible before the human Trio meets.

**Output:** one file at `products/<product>/discovery-backlog/<disc-slug>.council-<ISO-timestamp>.md`. Append-only: re-running on the same DISC creates a new file; existing Councils are never modified.

**Default roles:** `pm,pd,tl` (the classical Trio). Use `--roles=all` to include SM and PO; use `--roles=<list>` to specify explicitly.

**After this command:**
- Read the paste-ready stakeholder block from the CLI report; paste it into the Trio's Teams/email thread.
- Open the written Council file before the Trio meeting; lead the meeting with the Tension Map.
- If a high tension is surfaced that the DISC body doesn't currently address, update the DISC and optionally re-run `/pom-council` to capture the updated picture.
- When ready, run `/pom-discovery-gate` to formally grade the DISC.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/council.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/four-question-gate.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
</execution_context>

<process>
Execute the workflow end-to-end.

Preserve all guarantees:
- Parallel dispatch of role agents (NOT sequential).
- Abort the entire run on any role failure or schema violation — never write a partial Council.
- Strip forbidden verdict tokens from synthesizer output before rendering.
- Co-locate the Council file with the parent DISC.
- Append-only: never modify or delete prior Council files.
- Read-only on every artifact except the new Council file.
</process>
