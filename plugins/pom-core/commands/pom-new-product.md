---
description: "Stand up a new product end-to-end: spin up the product directory, form the first pod, draft an initial runway plan. Stops if any prerequisite is missing."
argument-hint: "<product-slug> [--pm=<name>] [--pd=<name>] [--tl=<name>] [--sponsor=<name>] [--originating-uc=<UC-ID>] [--pod-name=<slug>] [--sm=<name>] [--po=<name>] [--members=<name,name,…>]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Skill
---

# /pom-new-product

Orchestrate three skills in sequence to take a portfolio from "we should have a Payments product" to "Payments has its first pod with a runway plan."

## Pre-flight

1. Resolve cwd to the active POM repo (`intake/`, `products/`, etc. exist).
2. Verify `products/<product-slug>/` does NOT already exist. If it does, abort with: `Product '<slug>' already exists. Use /pom-form-pod or /pom-runway-add-adr for incremental changes.`
3. Confirm slug is kebab-case lowercase.

## Pipeline

### Stage 1 — Spin up the product (writes products/<slug>/...)
Invoke `pom-spinup-product` skill with the slug and any provided Trio names. The skill:
- Copies `products/_template/` to `products/<slug>/`
- Fills `product.md` with placeholders (Vision, Target customer, etc.) — prompts the user for each
- Fills `trio.md` with PM/PD/TL names (or `_TBD_` if not provided; prompts the user)
- Updates `products/README.md` (portfolio index)
- If `--originating-uc` is provided, adds the UC reference to `product.md`

**Human gates:** Trio assignments, vision statement, outcomes. Do not auto-fill these.

### Stage 2 — Form the first pod (writes products/<slug>/pods/<pod-name>/...)
Only proceeds if the user opts in (Ask: "Form the first pod for this product now? You can also wait until a PB item is ready to pull.").

If yes, invoke `pom-form-pod` with the product slug, the pod name, and any provided SM/PO/members. The skill:
- Copies `products/_template/pods/_template/` to `products/<slug>/pods/<pod-name>/`
- Fills `pod.md` with roster (SM, PO, 2-5 members) — prompts for any missing
- Enforces the 2-7 composition rule
- Surfaces the cadence default from the bootstrap interview

**Human gate:** roster confirmation, especially member list (the 2-7 rule is non-negotiable).

### Stage 3 — Draft the initial runway plan (writes products/<slug>/runway/runway-plan.md)
Only proceeds if the user opts in (Ask: "Draft an initial runway-plan.md for this product? You can refresh it later with /pom-runway-plan.").

If yes, invoke `pom-runway-plan` with the product slug. The skill:
- Writes a fresh `runway-plan.md` from template
- Auto-fills the "Recent decisions" table (empty — no ADRs yet)
- Prompts the user for the user-authored sections (commitments, investments, assumptions, open questions)

**Human gate:** the user-authored sections require real input. Don't fabricate "TBD."

## After the run

Print a summary:

```
Product '<slug>' stood up:
  ✓ products/<slug>/product.md
  ✓ products/<slug>/trio.md
  ✓ products/README.md updated
  ✓ Pod '<pod-name>' formed (<N> members)            [if stage 2 ran]
  ✓ products/<slug>/runway/runway-plan.md drafted    [if stage 3 ran]

Next steps:
  - Run /pom-route-to-discovery <UC-ID> <slug> to route Use Cases here
  - Run /pom-runway-add-adr <slug> <decision-slug> when the TL records the first architectural decision
  - Run /pom-status to see the portfolio with the new product
```

## Guardrails

- Each stage is opt-in past the first. The orchestrator must not silently form a pod or draft a runway without user confirmation.
- If `pom-spinup-product` fails (e.g., template missing, products/ not present), do NOT proceed to stages 2–3.
- Trio names default to `_TBD_` if not provided — that is acceptable; the validator will WARN but not ERROR (per R3.5). The orchestrator should surface this WARN explicitly so the user can fill it now if they want.

## References

- `${CLAUDE_PLUGIN_ROOT}/methodology/workflows/spinup-product.md`
- `${CLAUDE_PLUGIN_ROOT}/methodology/workflows/form-pod.md`
- `${CLAUDE_PLUGIN_ROOT}/methodology/workflows/runway-plan.md`
- `${CLAUDE_PLUGIN_ROOT}/methodology/references/pod-composition-rules.md`
- `${CLAUDE_PLUGIN_ROOT}/methodology/references/runway-rules.md`
