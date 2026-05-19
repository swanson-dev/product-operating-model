# Fulfillment integrity — retail starter standard

> Customise to your fulfillment model (own warehouse / 3PL / drop-ship / store-fulfillment / hybrid).

## Scope

Order capture → pick → pack → ship → deliver chain, plus reverse logistics (returns) and customer communication at each transition. Includes BOPIS pickup integrity, ship-from-store integrity, and 3P seller fulfillment integrity (for marketplace operators).

## Baseline rules

### FI1 — Order-state authoritative source
The order state machine has a single authoritative source. Every channel surfacing order state reads from (or replicates from) it with defined freshness. (Pairs with omnichannel-data OC6.)

### FI2 — Picking accuracy monitoring
Pick-accuracy rate is measured (overall and by location). Materially low locations are investigated. Pick errors trigger downstream review (was this a mislabel, mispick, or substitution?).

### FI3 — Shipping promise accuracy
For each shipping promise tier offered (standard, expedited, same-day, etc.), actual achievement rate is measured. Material under-promise/over-promise gaps are flagged.

### FI4 — Carrier handoff and tracking
Carrier handoffs are tracked. Lost packages investigated. Customer tracking visibility is consistent with internal visibility (no "shipped" status when the carrier hasn't picked up).

### FI5 — BOPIS pickup integrity (if applicable)
Buy-online-pickup-in-store flows guarantee inventory at the pickup store at the time of order. Curbside / locker variants meet documented SLA for ready-for-pickup notification.

### FI6 — Returns processing integrity
Returns are received, inspected per documented policy, refunded per documented timeline. Restocked items return to inventory accurately. Damaged or fraudulent returns follow a documented path.

### FI7 — 3P seller fulfillment integrity (marketplace operators only)
3P sellers meet documented fulfillment standards (ship-by SLA, valid tracking, accurate item delivery). Underperforming sellers flagged. Buyer protection covers documented gaps.

### FI8 — Communication at each transition
Customer communications fire at each material transition (order confirmed, picked, shipped, out-for-delivery, delivered, returned). Failures of communication generate operational signals.

## Open questions for the standard owner

- FI3: shipping promise tracking — vendor (e.g., ShipperHQ, ShipBob native) or homegrown?
- FI5: BOPIS inventory-hold mechanism — soft-allocation, hard-allocation, both?
- FI7 (if applicable): 3P seller scorecard structure — what tiers, what consequences?

## How this interacts with Q4

Retail R5 (returns / chargebacks fairness), R6 (peak-window resilience), R7 (marketplace seller integrity) read this. Cross-ref generic G2 (data quality — order state), G4 (reversibility — wrong fulfillment is hard to undo).
