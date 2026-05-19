---
name: pom-discovery-gate
description: "Use when a Discovery item needs to be walked through the four-question gate (desirable / viable / feasible / ethical) — produces a structured assessment with evidence, blockers, promotion-readiness verdict, and critical-path unblock sequence. Append-only on shaping log."
argument-hint: "<DISC-ID-or-path> [--question=Q1|Q2|Q3|Q4]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Walk a Discovery item through the four-question gate (per `four-question-gate.md`). For each question:
1. State the question and its owner role (PD / PM / TL / Trio+Compliance)
2. Check if owner is assigned in the product's `trio.md` — surface blocker if not
3. Prompt user for current state, evidence, and remaining work
4. Update the question's status marker (❌ / 🟡 / ✅) and write detailed answer prose
5. Update the Promotion Readiness table and critical-path unblock sequence
6. Append a dated shaping log entry

**Required for ✅:** evidence captured in the file. Not "we know" — the answer must be in the document.

**Upstream blockers detected and reported:**
- Unassigned Trio roles (PD for Q1, PM for Q2, TL for Q3)
- Product outcomes still seed-state (blocks Q2)
- Missing `enabling/<concern>/` for the UC's relevant standards (blocks Q4)

**Updates** (never overwrites prior content):
- The DISC file's four question sections
- Promotion Readiness table
- Critical-path unblock sequence
- Shaping log (append-only)

**Refuses** to promote items — promotion is `pom-promote-to-backlog`'s job.

**After this command:**
- If all 4 ✅ + Trio-Note gaps ✅ → run `/pom-promote-to-backlog <DISC-ID>`
- If not ready → address the critical-path items, then re-run this skill
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/discovery-gate.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/four-question-gate.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/products/_template/discovery-backlog/DISC-template.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (owner-assignment checks, evidence-required for ✅, append-only shaping log).
</process>
