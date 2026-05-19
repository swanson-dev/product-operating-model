# pom-healthcare

Healthcare industry overlay for the Product Operating Model. Layers on top of `pom-core` (install first).

## What it adds

Run `/pom-healthcare-init <pom-repo-path>` after `pom-core` bootstrap to install:

- **Healthcare-calibrated WSJF rubric** — anchored to clinical-impact signals (HIPAA/HITECH deadlines, CMS interoperability rules, USCDI revisions, payer/provider/digital-health distinctions).
- **HIPAA + clinical-safety Q4 framing** — adds healthcare-specific sub-questions (PHI handling, clinical-safety risk, equity of access, vulnerable patient populations, FDA SaMD applicability) on top of generic G1–G8.
- **Starter enabling standards** — PHI handling, clinical safety, interoperability (FHIR/USCDI), AI-ethics-healthcare overlay.
- **Interactive interview** — payer/provider/life-sciences split, PHI exposure, clinical workflow integration, FDA-regulated software boundaries.
- **Worked example** — a sample UC tailored to a payer/provider/digital-health scenario.

## Install

```text
/plugin install pom-core@product-operating-model
/plugin install pom-healthcare@product-operating-model
/pom-healthcare-init /path/to/your/pom-repo
```

Or apply at bootstrap: `/pom-bootstrap /path --rubric=healthcare`.

## Compatibility

Co-installable with other industry overlays. Healthcare-specific enabling concerns extend, never replace, the generic baseline in `pom-core`.

## License

MIT — see [../../LICENSE](../../LICENSE).
