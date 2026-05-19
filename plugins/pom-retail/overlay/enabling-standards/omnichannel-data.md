# Omnichannel data consistency — retail starter standard

> The biggest source of retail customer failure: "the website said it was in stock but the store didn't have it" / "I saw a price online and the store charged different." Customise to your channel mix and fulfillment model.

## Scope

Every dataset that's read by multiple channels: inventory (per location + aggregate), pricing (base + promotional), product master, customer master, order state, loyalty balance, shipping promises.

## Baseline rules

### OC1 — Source of truth per data type
Each shared dataset has exactly one source-of-truth system. Other channels are consumers, not parallel masters. Documented.

### OC2 — Cross-channel freshness SLA
Each shared dataset has a documented freshness SLA per channel (e.g., inventory: e-commerce ≤ 2 min, store ≤ 30 sec local, mobile ≤ 5 min). Misses surface, not silent.

### OC3 — Reconciliation between channels
At a documented cadence, reconciliation pipelines compare channel views against the source-of-truth. Discrepancies investigated; chronic discrepancies become PB items.

### OC4 — Pricing consistency by channel
Where pricing differs by channel (online vs in-store), the difference is intentional, documented, and disclosed where regulation requires. Anti-bait-and-switch: an honoured price doesn't depend on which side of the transaction sees first.

### OC5 — Customer-data consistency
Customer profile (name, address, loyalty status, preferences) is consistent across channels. Source-of-truth designation per field. Updates propagate within SLA.

### OC6 — Order state consistency
Order state transitions (placed → confirmed → picked → shipped → delivered → returned) are visible identically across all channels the customer can interact with (web, mobile, email, store, call centre).

### OC7 — Promotion eligibility consistency
A promotion's eligibility (e.g., "20% off, customers in loyalty tier X, on category Y") evaluates the same way across all channels offering the promotion.

## Open questions for the standard owner

- OC1: Inventory source-of-truth — WMS, ERP, dedicated OMS?
- OC3: Reconciliation tooling — built or vendor?
- OC4: Is there a documented "channel pricing strategy" — usually yes, but often informal?

## How this interacts with Q4

Retail R2 (omnichannel data consistency) reads this. Cross-ref generic G2 (data quality), G4 (reversibility — silent inconsistencies are hard to roll back), G8 (enabling-standard alignment).
