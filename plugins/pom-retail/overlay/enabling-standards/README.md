# pom-retail enabling-standards

Four starter standards installed by `pom-retail-init`:

| Concern | Why |
|---|---|
| `pci-retail` | Retail-specific PCI scope: POS, e-commerce, BOPIS pickup, returns refund-to-card. |
| `accessibility-regulatory` | Layer over the generic accessibility-baseline: ADA Title III, EAA, state-specific obligations, retail-surface enumeration. |
| `omnichannel-data` | Cross-channel data consistency (inventory, pricing, customer, promotion) — the biggest source of retail customer failure. |
| `fulfillment-integrity` | Order capture → pick → pack → ship → deliver chain integrity; reverse logistics for returns. |

## Interaction with generic baseline

The generic `pom-core` standards stay in place. `accessibility-regulatory` is a companion to the generic `accessibility-baseline.md` — the generic is the WCAG floor; this overlay adds the regulatory enforcement layer.
