---
name: pom-promote-to-backlog
description: "Use when a Discovery item has passed the four-question gate (4/4 ✅) and is ready to enter the Product Backlog as a pullable vertical slice. Writes the PB file with slice definition, acceptance signals, runway prerequisites; updates DISC status. Refuses to promote if any question still ❌ or 🟡."
argument-hint: "<DISC-ID-or-path>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Promote a 4/4 ✅ Discovery item to its product's Product Backlog. Writes a `PB-YYYY-MM-<slug>.md` file containing:
- Outcome (referencing the product's ratified outcomes)
- **Vertical slice definition** — thin end-to-end customer journey (not a task list)
- Acceptance signals (≥ 2 measurable signals)
- Runway prerequisites (links to ADRs, patterns, platform services)
- Out of scope (explicit exclusions to prevent scope creep)
- Dependencies on other PB items

**Refuses** to promote items that haven't passed the four-question gate (per `four-question-gate.md`) or have open Trio-Note gaps. The gate is the load-bearing rule of the methodology.

**Updates:**
- DISC file's status → "Promoted to Product Backlog YYYY-MM-DD"
- DISC shaping log (append-only) → promotion entry
- `products/README.md` portfolio index → Product Backlog count

**Emits engineering affordances in the CLI report** (paste-ready, never written to disk):
- An ADO-compatible branch name (`pb/PB-YYYY-MM-<slug>`)
- A PR title (`PB-YYYY-MM-<slug>: <outcome>`)
- A PR description body in markdown referencing the slice contract, acceptance signals, out-of-scope, and linked runway/ADRs

These thread the PB-ID through every commit and PR — strategic decisions become traceable from `git log`. `/pom-sync-ado` will later use the same fields to push the PB into Azure DevOps.

**After this command:**
- If no pods exist for this product → run `/pom-form-pod <product> <pod-slug>`
- Otherwise, the PO Sync orders the Product Backlog; pods pull at their cadence.
- When a pod pulls, use the suggested branch name and PR template.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/promote-to-backlog.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/four-question-gate.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/products/_template/product-backlog/PB-template.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (promotion readiness check, vertical-slice requirement, append-only DISC log).
</process>
