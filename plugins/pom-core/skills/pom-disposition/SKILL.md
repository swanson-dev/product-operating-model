---
name: pom-disposition
description: "Use when a scored UC needs a formal kill/park/route/split decision. Writes a DISP file with required fields, paste-ready stakeholder summary, updates UC status. Append-only — reversals create new DISP that supersedes the old."
argument-hint: "<UC-ID> --decision=<kill|park|route|split> [--product=<slug>] [--rationale=<text>] [--trigger=<text>] [--new-ucs=<UC-NNNN,UC-NNNN>]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Record a formal disposition decision on a scored Use Case. Writes the DISP file with all required fields per `disposition-rules.md`, generates a paste-ready stakeholder-facing summary, and updates the UC's Status + Disposition fields.

**Decisions:**
- `kill` — UC will not be pursued. Documented and closed.
- `park` — Held for future re-evaluation. Re-score trigger required.
- `route` — Sent to a specific product's Discovery Backlog. Receiving product + Trio named.
- `split` — Decomposed into 2+ smaller UCs. Original killed; new UCs to be ingested.

**Creates:**
- `intake/dispositions/DISP-YYYY-MM-DD-<UC-ID>.md`

**Updates:**
- `intake/use-case-backlog/<UC-ID>-*.md` (Status + Disposition link)

**Append-only:** A reversal of a prior disposition creates a NEW DISP file that supersedes the old one. The old file is never edited or deleted.

**After this command:**
- If route → run `/pom-spinup-product <slug>` (if needed) then `/pom-route-to-discovery <UC-ID> <slug>`
- If split → run `/pom-ingest-use-case` for each new UC fragment
- If kill or park → no further action; for park, re-invoke when trigger fires
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/disposition.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/disposition-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/intake/dispositions/DISP-template.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (decision validation, supersession handling, required-field collection, append-only rule).
</process>
