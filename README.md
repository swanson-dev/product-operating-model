# Product Operating Model — Claude Code marketplace

A Claude Code marketplace that operationalises the **Product Operating Model** (POM) — a target operating model for product organisations covering intake, four-question Discovery gating, WSJF prioritisation, product/platform/enabling layers, runway management, and structural validation.

> **Status:** v0.1.0 in development. Install instructions and full docs follow shortly.

## What ships

| Plugin | Required? | What it gives you |
|--------|-----------|-------------------|
| `pom-core` | **Yes** | 15 skills, 7 agents, 4 orchestrator commands, the methodology library, and the generic industry baseline. |
| `pom-insurance` | Optional | Insurance overlay: rubric, regulatory Q4 framing, starter enabling standards. |
| `pom-fintech` | Optional | Fintech overlay: rubric, consumer-protection Q4 framing, PCI/KYC starter standards. |
| `pom-healthcare` | Optional | Healthcare overlay: rubric, HIPAA/clinical-safety Q4 framing, PHI/interoperability starter standards. |
| `pom-retail` | Optional | Retail overlay: rubric, PCI/accessibility Q4 framing, omnichannel starter standards. |

## Install (after v0.1.0 publishes)

```text
/plugin marketplace add swansoncreativestudios/product-operating-model
/plugin install pom-core@product-operating-model
# optional, after pom-core is bootstrapped:
/plugin install pom-healthcare@product-operating-model
```

## Quickstart

```text
/pom-bootstrap /path/to/new-pom-repo --rubric=generic
/pom-status /path/to/new-pom-repo
/pom-validate /path/to/new-pom-repo
```

Full quickstart, command reference, and methodology overview ship in `plugins/pom-core/README.md`.

## License

MIT — see [LICENSE](./LICENSE).
