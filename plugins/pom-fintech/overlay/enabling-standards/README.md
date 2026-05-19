# pom-fintech enabling-standards

Four starter standards installed by `pom-fintech-init`:

| Concern | Why |
|---|---|
| `pci-handling` | Minimise PCI scope, define tokenisation expectations, segregate cardholder-data environments. |
| `kyc-aml-bsa` | Onboarding KYC integrity, AML monitoring, sanctions screening, SAR filing pipeline. |
| `consumer-protection` | CFPB, Reg E/Z, UDAAP, fair lending — the baseline bar for any consumer-facing change. |
| `model-risk` | SR 11-7-inspired model governance for any production model affecting customer outcomes. |

Each is a starter — customise to your specific charter, licensing footprint, and product mix.

## Interaction with generic baseline

The generic `pom-core` standards (ai-ethics, security-baseline, accessibility-baseline, data-quality) stay in place. These four are fintech-specific additions; some overlap with the generic ai-ethics (model-risk extends it).
