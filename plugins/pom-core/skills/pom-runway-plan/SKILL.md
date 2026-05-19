---
name: pom-runway-plan
description: "Use when a product's runway-plan.md needs to be created or refreshed. Re-scans runway/ for ADRs and patterns, updates auto-generated tables, preserves user-authored sections (commitments, investments, assumptions, open questions). Surfaces orphan references."
argument-hint: "<product-slug> [--force-regenerate]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Maintain a product's living runway plan — `products/<product>/runway/runway-plan.md`. This is THE document Trios review at every sync to understand current architectural commitments and upcoming investments.

**Auto-generated sections (refreshed every run):**
- Recent decisions table → re-scans `runway/` for ADR files
- Active patterns table → re-scans `runway/` for PATTERN files
- Last updated date

**User-authored sections (preserved on refresh):**
- Current commitments
- Upcoming runway investments
- Capacity assumptions
- Open questions
- Retired items

**`--force-regenerate`** wipes user-authored content and rebuilds from scratch. Requires explicit confirmation via `AskUserQuestion`.

**Orphan detection:** if the plan references ADRs/patterns that no longer exist in `runway/`, surfaces them as warnings (does NOT auto-remove — user decides).

**Creates** runway-plan.md if it doesn't exist.

**After this command:**
1. Trio reviews the plan at the next sync.
2. Add new ADRs via `/pom-runway-add-adr <product> <slug>` — they'll surface here automatically on next refresh.
3. Re-run this skill periodically (recommend monthly or after any ADR addition).
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/runway-plan.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/runway-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/products/_template/runway/runway-plan-template.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (preserve user-authored sections on refresh, orphan-detection warnings, no silent destructive operations).
</process>
