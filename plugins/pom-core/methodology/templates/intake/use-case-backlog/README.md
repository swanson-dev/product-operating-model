# Use Case Backlog

The single, portfolio-wide list of scored stakeholder requests. One markdown file per use case.

## Naming

`UC-NNNN-short-slug.md` where `NNNN` is a zero-padded incrementing integer (UC-0001, UC-0002, …). Slug is kebab-case.

## File format

See [`UC-template.md`](UC-template.md) for the canonical template. Every use case file must contain:

- **ID** — matches filename
- **Title** — one line
- **Stakeholder** — name, role, contact
- **Source material** — relative path into [`../../use-cases/`](../../use-cases/)
- **Submitted date** — ISO 8601
- **Problem statement** — what's broken / missing / opportunity, in stakeholder's words
- **Desired outcome** — measurable if possible
- **WSJF scores** — User-Business Value, Time Criticality, Risk Reduction & Opportunity Enablement, Job Size, computed WSJF
- **WSJF rationale** — why these scores; reference comparable use cases
- **Disposition** — pending / killed / parked / routed-to-`<product>`
- **Disposition link** — relative path into [`../dispositions/`](../dispositions/)

## Lifecycle

```
new           → file created, fields filled except WSJF + disposition
scored        → WSJF computed by Trio per ../scoring/wsjf-rubric.md
dispositioned → decision logged in ../dispositions/, file updated
routed        → referenced from products/<product>/discovery-backlog/
```

After routing, the use case file remains here as the canonical source. The product's discovery-backlog references back to this file rather than copying it.
