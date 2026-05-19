---
name: pom-form-pod
description: "Use when a product is ready to stand up a new pod (typically its first, when a PB item is ready to pull). Copies pods/_template/, fills roster (SM, PO, 2-5 members), enforces 2-7 composition rule, documents surface area and cadence."
argument-hint: "<product-slug> <pod-slug> [--sm=<name>] [--po=<name>] [--members=<name,name,…>] [--surface=<text>] [--force]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Stand up a new pod for a product. Copies the canonical pod template, fills the roster, documents what the pod owns, sets cadence defaults.

**Composition rules enforced** (per `pod-composition-rules.md`):
- 2–7 total members (SM + PO + 0-5 contributors). Outside this range refuses without `--force`.
- Exactly one SM.
- Exactly one PO.

**Creates:**
- `products/<product>/pods/<pod-slug>/pod.md`
- `products/<product>/pods/<pod-slug>/pod-backlog/` (empty)
- `products/<product>/pods/<pod-slug>/slices/` (empty)

**Updates:**
- `products/README.md` portfolio index (Pods count)

**Refuses** to form a pod for a non-existent product (suggests `pom-spinup-product` first). **Refuses** to overwrite an existing pod.

**After this command:**
1. PO Sync orders the Product Backlog. Pod pulls from the top at its cadence.
2. Pod documents working agreements (pairing defaults, code review SLA, on-call) in `pod.md`.
3. First vertical slice goes into `slices/<slice-slug>/`.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/form-pod.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/pod-composition-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/products/_template/pods/_template/pod.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (slug validation, product existence check, composition rule enforcement, no overwrite).
</process>
