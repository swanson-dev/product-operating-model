# Intake

The single, portfolio-wide funnel from stakeholder request to product routing. Every product in [`../products/`](../products/) is fed from here — there is no side-door.

## The five steps

1. **Capture** — stakeholder submits raw material (deck, PDF, conversation notes) into [`../use-cases/`](../use-cases/).
2. **Score** — a new entry is added to [`use-case-backlog/`](use-case-backlog/) using the WSJF rubric in [`scoring/wsjf-rubric.md`](scoring/wsjf-rubric.md).
3. **Disposition** — a decision is logged in [`dispositions/`](dispositions/): **kill**, **park**, **route**, or **split**.
4. **Route** — for routed items, the entry is referenced from the receiving product's `discovery-backlog/`.
5. **Discovery** — the Product Trio shapes the opportunity into a candidate for the Product Backlog (or sends it back / parks it).

Steps 1–4 are intake's responsibility. Step 5 belongs to the Product Trio of the receiving product.

## What does NOT belong here

- Product-specific roadmap items → [`../products/<product>/roadmap.md`](../products/)
- Pod-level tasks or stories → `pods/<pod>/pod-backlog/`
- Architectural decisions → `products/<product>/runway/`
- Stakeholder *delivery* communication (status, demos) → the receiving product

Intake's only job is: **score, disposition, route**.

## Files

| Path | Purpose |
|---|---|
| [`scoring/wsjf-rubric.md`](scoring/wsjf-rubric.md) | The scoring contract — every use case is scored using this rubric, nothing else |
| [`use-case-backlog/`](use-case-backlog/) | One markdown file per use case (`UC-NNNN-slug.md`) |
| [`dispositions/`](dispositions/) | Append-only log of disposition decisions |
