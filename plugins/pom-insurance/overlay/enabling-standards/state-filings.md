# State filings — insurance starter standard

> Customise to your licensed state footprint.

## Scope

Every state-specific filing: rate filings, form filings, advertising filings, agent licensing renewals, market-conduct exam submissions, data-call responses. Covers all 50 states + DC where the portfolio is licensed.

Tightly coupled with `regulatory-reporting.md` — that standard covers federal + NAIC + non-state; this one is the state-by-state operational layer.

## Baseline rules

### F1 — Filing matrix maintained
A current filing matrix (`enabling/state-filings/matrix.md` — starter scaffold) lists, per state: licensed lines, current filing status (rate, form, advertising), SERFF tracking number (if open), effective date of last filing.

### F2 — Pre-filing review
Any rate or form change going to a state filing follows a pre-filing review process: legal, actuarial (if rate), compliance. Sign-off recorded.

### F3 — SERFF user roster current
SERFF user roster is reviewed quarterly (state filings are made under user identities; offboarded users must be removed promptly to avoid filings under stale identities).

### F4 — Objection-handling SLA
Each state has an internal SLA for responding to a regulator objection (typical: 15 days; some states require 10). Misses are tracked and escalated.

### F5 — Multi-state coordination
For products filed in multiple states, the matrix tracks state-specific deviations from a base filing. A single source of truth identifies which variation is master vs. derivative.

### F6 — Withdrawn/discontinued products
When a product is discontinued in a state, the state-specific run-off (continued servicing of in-force policies, claim payments, regulatory reporting on closed business) is tracked. This doesn't close the filing matrix entry; it transitions it to "run-off."

## Open questions for the standard owner

- What is the materiality bar for F2 (do form-only minor changes need full review, or just rate changes)?
- Who owns the multi-state coordination role in F5 — a state-relations team, a compliance generalist, or product-level ownership?
- How are SERFF correspondences mirrored internally (email-based, shared inbox, ticketing)?

## How this interacts with Q4

Insurance-specific Q4 sub-question I1 (regulatory exposure) reads this standard. Cross-references the generic G8 (enabling-standard alignment) for any state-specific deviation, which must be in `enabling/state-filings/decision-log/`.
