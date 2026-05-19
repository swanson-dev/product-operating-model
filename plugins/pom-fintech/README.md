# pom-fintech

Fintech industry overlay for the Product Operating Model. Layers on top of `pom-core` (install pom-core first).

## What it adds

Run `/pom-fintech-init <pom-repo-path>` after `pom-core` bootstrap to install:

- **Fintech-calibrated WSJF rubric** — anchored to fintech market signals (CFPB rules, Reg E/Z, BSA/AML, PCI-DSS, PSD2 SCA, partner-led embedded windows).
- **Industry-aware Q4 framing** — adds fintech-specific sub-questions (consumer-protection regs, KYC/AML applicability, fair-lending exposure, payment-rails compliance) on top of generic G1–G8.
- **Starter enabling standards** — PCI handling, KYC/AML/BSA, consumer protection & disclosure, model risk (SR 11-7-inspired).
- **Interactive interview** — fintech-specific questions (chartered vs partner-chartered, customer types, payment rails, lending products).
- **Worked example** — a sample UC tailored to a lending / banking / payments scenario.

## Install

```text
/plugin install pom-core@product-operating-model
/plugin install pom-fintech@product-operating-model
/pom-fintech-init /path/to/your/pom-repo
```

Or apply at bootstrap: `/pom-bootstrap /path --rubric=fintech` (pom-core routes the overlay invocation).

## Compatibility

Co-installable with other industry overlays. The generic baseline standards in `pom-core` stay in place — this overlay extends, doesn't replace.

## License

MIT — see [../../LICENSE](../../LICENSE).
