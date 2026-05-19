---
name: pom-seed-enabling-standard
description: "Use when a new enabling concern needs to be bootstrapped in a POM repo — AI ethics, security, data quality, accessibility, etc. For known concerns (ai-ethics) ships full v0.1 standards; for new concerns generates a minimal scaffold + decision log."
argument-hint: "--concern=<slug> [--force-update]"
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
Bootstrap a new enabling concern (Layer 3 standard) in a POM repo with v0.1 scaffolding. Run when a UC's Discovery Q4 (Ethical/Compliant) requires a standard that doesn't exist yet in the portfolio.

**Concerns with full templates available:**
- `ai-ethics` — Model Card template, PII handling standards, bias audit framework

**Concerns with generic scaffolding** (README + decision-log + optional stub files):
- Any other concern slug — `security`, `data-quality`, `accessibility`, `coaching-metrics`, etc.

**Creates:**
- `enabling/<concern>/README.md` (v0.1 — references authoritative org policy)
- For full templates: concern-specific standards files (Model Card, PII checklist, bias audit framework)
- For generic: optional stub `checklist.md` and `framework.md` (asks via AskUserQuestion)
- `enabling/<concern>/decision-log/README.md` (append-only precedent log)

**Refuses** to overwrite an existing concern without `--force-update` AND explicit confirmation.

**After this command:**
1. Link authoritative organizational policies in `enabling/<concern>/README.md`.
2. Apply the per-UC checklists during Discovery Q4 of relevant UCs.
3. Log standards-level decisions in `decision-log/` as they arise.
4. Re-calibrate after first 3 UCs apply the standards.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/seed-enabling-standard.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/enabling/README.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/enabling/ai-ethics/README.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (slug validation, existence check, force-update confirmation, mode branching).
</process>
