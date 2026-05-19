---
name: pom-spinup-product
description: "Use when standing up a new product in an existing POM repo — copies products/_template/ to products/<slug>/, fills seed-state placeholders, updates portfolio index. Typically run when a UC has routed to a product that doesn't exist yet."
argument-hint: "<product-slug> [--pm=<name>] [--pd=<name>] [--tl=<name>] [--executive-sponsor=<name>] [--originating-uc=<UC-NNNN>]"
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
Stand up a new product folder in an existing POM repo by copying the canonical `products/_template/` to `products/<slug>/` and filling seed-state placeholders with provided values.

**Creates:**
- `products/<slug>/README.md`
- `products/<slug>/product.md` (seed charter — Trio must ratify)
- `products/<slug>/trio.md` (with provided Trio assignments; unassigned roles flagged as blocking risks)
- `products/<slug>/roadmap.md`
- `products/<slug>/discovery-backlog/`, `product-backlog/`, `runway/`, `pods/` (empty)
- Updates `products/README.md` portfolio index

**Validates** the product slug (kebab-case, lowercase, no spaces). **Refuses** to overwrite an existing product directory.

**After this command:**
1. Trio ratifies the product charter.
2. If `--originating-uc` provided, run `/pom-route-to-discovery <UC> <slug>` to create the Discovery file.
3. Set Trio operating cadence.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/spinup-product.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/pod-composition-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/products/_template/product.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/products/_template/trio.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/products/_template/roadmap.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (slug validation, precondition checks, portfolio index update).
</process>
