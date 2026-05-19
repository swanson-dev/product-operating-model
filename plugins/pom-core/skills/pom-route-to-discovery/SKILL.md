---
name: pom-route-to-discovery
description: "Use when a routed UC needs to land in its receiving product's Discovery Backlog. Creates a DISC file as a reference (NOT a copy) to the originating UC, updates the UC's Discovery entry field, updates the portfolio index."
argument-hint: "<UC-ID> <product-slug>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Land a routed Use Case in its receiving product's Discovery Backlog. Creates a `DISC-YYYY-MM-<slug>.md` file as a **reference** to the originating UC (not a copy of UC content). Initializes the four-question gate scaffold with all questions ❌. Reflects current Trio state in the critical-path sequence.

**Preconditions:**
- UC has been dispositioned as `route → <product-slug>`
- `products/<product-slug>/` exists (run `pom-spinup-product` first if not)
- No duplicate DISC already exists for this UC in this product

**Creates:**
- `products/<product-slug>/discovery-backlog/DISC-YYYY-MM-<UC-slug>.md`

**Updates:**
- UC file's `Discovery entry` field (for R7.2 cross-reference integrity)
- `products/README.md` portfolio index (Discovery item count)

**Refuses** to copy UC content into the DISC file. The DISC must be a reference.

**After this command:** Run `/pom-discovery-gate <DISC-ID>` to walk through the four-question shaping gate.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/route-to-discovery.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/four-question-gate.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/products/_template/discovery-backlog/DISC-template.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (UC disposition check, product existence check, duplicate-DISC check, reference-not-copy rule).
</process>
