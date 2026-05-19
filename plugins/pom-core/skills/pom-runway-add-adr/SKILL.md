---
name: pom-runway-add-adr
description: "Use when a product's TL needs to record an architectural decision (ADR). Auto-numbers sequentially within the product, fills the canonical ADR template (context, drivers, options, decision, consequences), updates runway-plan.md. Immutable content — supersession via new ADR, not edit."
argument-hint: "<product-slug> <adr-slug> [--supersedes=ADR-NNNN]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Record an Architecture Decision Record in a product's runway. Sequential numbering per product. Immutable content once written — supersession creates a NEW ADR (and updates the prior ADR's Status field only).

**Required content per ADR:**
- Context and problem statement
- Decision drivers (2-5 quality attributes / constraints)
- Considered options (≥ 2, each with pros + cons)
- Decision outcome with rationale
- Consequences (positive + negative + neutral/follow-up)

**Creates:**
- `products/<product>/runway/ADR-NNNN-<slug>.md` (NNNN = next available, zero-padded)

**Updates:**
- `products/<product>/runway/runway-plan.md` "Recent decisions" table (if runway-plan exists)
- The superseded ADR's Status field only (content preserved) if `--supersedes` provided

**Emits** a paste-ready decision summary in the CLI report — one paragraph for Teams/email/PR comments naming the chosen option, rationale, and top trade-off.

**Refuses** to skip considered options, skip negative consequences, or modify a prior ADR's content.

**After this command:**
- If runway-plan.md doesn't exist → run `/pom-runway-plan <product>` to create it
- Mention the ADR at the next Trio sync
- If implementation requires runway investment, add it to runway-plan's "Upcoming runway investments"
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/runway-add-adr.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/runway-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/products/_template/runway/ADR-template.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (sequential numbering, ≥ 2 options required, immutability of prior ADRs, runway-plan sync).
</process>
