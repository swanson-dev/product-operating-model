# PCI handling — retail starter standard

> Customise to your channel mix and processor relationships.

## Scope

Every system that handles cardholder data in the retail context — e-commerce checkout, POS terminals, mobile POS, manual phone orders (call-centre PCI), kiosk / self-checkout, BOPIS pickup, returns-refunds-to-card, gift cards if linked to underlying funding.

## Baseline rules

### PR1 — Scope minimisation by design
New changes default to *out of PCI scope*. Tokenisation (network or processor) is the default. Raw PAN handling requires explicit security review.

### PR2 — POS device inventory and lifecycle
Physical POS terminals: inventoried, tamper-evidence checks (PCI-DSS Req 9.9), lifecycle managed (provisioning to decommissioning), firmware updates tracked.

### PR3 — E-commerce iframe / hosted-fields approach
E-commerce checkout uses processor-hosted iframes or hosted-fields where possible to keep the merchant page out of the cardholder-data environment.

### PR4 — Call-centre PCI handling
For phone orders: documented procedure for taking card data (e.g., DTMF masking via a vendor, agent pause-and-resume). Agents do not write card data on paper.

### PR5 — Tokenisation reuse across channels
Where a customer uses the same card across channels (e.g., e-commerce, in-store, app), tokens are unified or mapping is documented — to enable refund-to-original-tender without exposing PAN.

### PR6 — Returns refund-to-card
Refunds-to-card use the stored token, not user input of card number at return. POS return flows are designed accordingly.

### PR7 — Annual attestation + processor responsibility matrix
SAQ-A / SAQ-D depending on scope; processor responsibility matrices documented per relationship.

## Open questions for the standard owner

- POS device fleet size and management vendor?
- Call-centre PCI vendor (DTMF masking) if applicable — Aspect/Genesys/Sycurio/other?
- Token unification strategy across channels — is one in place, or are tokens channel-specific today?

## How this interacts with Q4

Retail R1 (PCI scope) reads this. Cross-ref generic G2 (data minimisation), G7 (third-party — processors / vendors).
