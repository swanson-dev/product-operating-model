# {{portfolio-name}}

Portfolio of products operating under a shared **Product Operating Model**. This repository is the source-of-truth filesystem for how work flows from stakeholder request to customer delivery across every product in the portfolio.

## How work flows

```
Stakeholders
     │
     ▼
intake/use-case-backlog/        ← WSJF-scored, single funnel for the portfolio
     │
     ▼
intake/dispositions/            ← kill / park / route / split
     │
     ▼
products/<product>/discovery-backlog/    ← Trio shapes the opportunity
     │
     ▼
products/<product>/product-backlog/      ← Trio + PO Sync grooms
     │
     ▼
products/<product>/pods/<pod>/pod-backlog/   ← Pod pulls work
     │
     ▼
products/<product>/pods/<pod>/slices/        ← Vertical slices ship
```

## Top-level layout

| Path | Purpose | Layer |
|---|---|---|
| [`intake/`](intake/) | Portfolio-wide funnel: stakeholder requests scored with WSJF, dispositioned, routed | Layer 0 — Value Stream |
| [`products/`](products/) | One folder per product; each owns its Trio, backlogs, roadmap, runway, and pods | Layer 1 — Pods |
| [`platform/`](platform/) | Shared platform services: provisioning, templates, governance, data quality | Layer 2 |
| [`enabling/`](enabling/) | Shared enabling functions: standards, security, AI ethics, coaching, metrics | Layer 3 |
| [`methodology/`](methodology/) | Reference diagrams — the contract this repo enforces | — |
| [`use-cases/`](use-cases/) | Raw stakeholder submissions (PDFs, decks); referenced by intake entries | — |

## Roles

- **Product Trio** (per product): Product Manager + Product Designer + Technical Leader — owns the Product Backlog, Roadmap, and Runway.
- **Product Enablement Team** (portfolio-shared): Vendor Manager + Data Architect.
- **Pod**: 2–7 cross-functional members + Scrum Master + Product Owner. POs execute Trio decisions; they do not unilaterally re-prioritize.

## Vocabulary

WSJF · Use Case Backlog · Discovery Backlog · Product Backlog · Product Trio · Architectural Runway · Vertical Slice · Pod · Release Planning · PO Sync · Platform Services. When in doubt, use this methodology's vocabulary, not generic Agile/Scrum terminology.
