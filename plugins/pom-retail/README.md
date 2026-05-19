# pom-retail

Retail / e-commerce industry overlay for the Product Operating Model. Layers on top of `pom-core` (install first).

## What it adds

Run `/pom-retail-init <pom-repo-path>` after `pom-core` bootstrap to install:

- **Retail-calibrated WSJF rubric** — anchored to retail signals (holiday windows, inventory cycles, PCI-DSS deadlines, accessibility regulation, omnichannel partner commitments).
- **Industry-aware Q4 framing** — adds retail-specific sub-questions (PCI scope, omnichannel data consistency, accessibility regulatory exposure, sales-tax obligations, return-and-refund fairness) on top of generic G1–G8.
- **Starter enabling standards** — PCI handling (retail-specific), accessibility (regulatory layer over generic baseline), omnichannel data consistency, fulfillment integrity.
- **Interactive interview** — retail-specific questions (channel mix, fulfillment model, payment processors, peak windows).
- **Worked example** — sample UC tailored to an e-commerce / brick-and-mortar / omnichannel scenario.

## Install

```text
/plugin install pom-core@product-operating-model
/plugin install pom-retail@product-operating-model
/pom-retail-init /path/to/your/pom-repo
```

Or apply at bootstrap: `/pom-bootstrap /path --rubric=retail`.

## Compatibility

Co-installable with other industry overlays (e.g., a retail-fintech BNPL hybrid might install `pom-retail` + `pom-fintech`). The generic `pom-core` baseline stays in place.

## License

MIT — see [../../LICENSE](../../LICENSE).
