---
name: pom-explain
description: "Use when a single POM artifact ID (UC-NNNN, DISP-..., DISC-..., PB-..., ADR-..., DEC-...) needs a one-line live status summary — current state, owners, age, and the next blocker if any. Read-only. Output is a single paste-ready line."
argument-hint: "<ARTIFACT-ID-or-path>"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
---

<objective>
Produce a single-line summary of any POM artifact, given its ID or path. Read-only.

**Recognized artifact prefixes** (case-insensitive):
- `UC-NNNN` — Use Case (in `intake/use-case-backlog/`)
- `DISP-YYYY-MM-DD-UC-NNNN` — Disposition (in `intake/dispositions/`)
- `DISC-YYYY-MM-<slug>` — Discovery item (in `products/*/discovery-backlog/`)
- `PB-YYYY-MM-<slug>` — Product Backlog item (in `products/*/product-backlog/`)
- `ADR-NNNN` — Architecture Decision Record (in `products/*/runway/`)
- `DEC-YYYY-MM-DD-<slug>` — Enabling-concern decision (in `enabling/*/decision-log/`)

**Output shape (one line, no headers):**
```
<ID>: <type>, <current state>, <key owner(s)>, <age>d old{{, blocker: <one phrase>}}
```

Examples:
- `UC-0042: Use Case, scored (WSJF=21), submitted by J. Park, 14d old`
- `DISC-2026-05-claims-doc-ai: Discovery, Q2 🟡 blocked on actuarial sign-off, Trio M. Park / J. Lee / D. Chen, 7d old`
- `ADR-0003 (claims): drift-detection-cadence, Accepted, TL D. Chen, 5d old`
- `DISP-2026-05-12-UC-0001: route → claims, by Beth Carter + Carrie Lines, 7d old`

**Read-only.** Never writes, edits, or modifies any file.

**After this command:** No follow-up needed. The output is the value. For deeper inspection, the user reads the artifact file directly.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/explain.md
</execution_context>

<process>
Execute end-to-end.
Read-only. The single-line output IS the result.
If the artifact is not found, say so plainly — do not guess.
</process>
