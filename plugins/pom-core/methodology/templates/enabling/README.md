# Enabling Functions (Layer 3)

Portfolio-shared capabilities that **make pods effective** without being part of the product surface area itself. Distinct from [`../platform/`](../platform/) — Platform services are *consumed at runtime*; enabling functions are *practices and standards*.

## Standard concerns (seeded as needed)

| Concern | Examples |
|---|---|
| **standards-compliance** | Coding standards, regulatory checklists, audit templates |
| **security** | Threat modeling templates, security review process, secret-handling policy |
| **ai-ethics** | AI use guidelines, model risk policies, Model Cards, PII handling, bias audit framework |
| **coaching-metrics** | Agility coaching artifacts, product coaching, flow metrics, DORA-style measures |

Additional concerns may be added as needed. Use `pom-seed-enabling-standard --concern=<slug>` to scaffold a new concern with v0.1 standards and a decision log.

## Operating principle

Enabling functions exist to **make the right thing the easy thing**. If a standard creates more friction than value, the standard is broken — not the team violating it. Standards documented here should be enforced at the runway level (linters, templates, automated checks) whenever possible, not via human gatekeeping.

## Decision logs

Every concern has its own append-only decision log. This is where exceptions, precedents, and standards-level rulings are recorded so future use cases inherit not just the rules but the *precedents*.
