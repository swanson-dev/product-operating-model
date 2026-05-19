# pom-insurance

Insurance industry overlay for the Product Operating Model. Layers on top of `pom-core` (which must be installed first).

## What it adds

When you run `/pom-insurance-init <pom-repo-path>`, this plugin copies the following into your POM repo (with merge prompts — never silent overwrite):

- **Calibrated WSJF rubric** — anchored to insurance market signals (NAIC deadlines, state DOI filings, distribution-partner commitments, regulatory windows). Replaces or augments the generic rubric.
- **Industry-aware Q4 framing** — adds insurance-specific sub-questions (regulatory exposure, state-of-filing scope, actuarial soundness, policyholder vulnerable populations) on top of the generic G1–G8.
- **Starter enabling standards** — regulatory reporting, actuarial review, policyholder communications, state filings.
- **Interactive interview** — additive to the generic bootstrap interview; asks insurance-specific questions (lines of business, regulated markets, distribution model).
- **Worked example** — a sample UC tailored to a P&C / life carrier scenario.

## Install

```text
# Step 1 (one-time): install pom-core
/plugin install pom-core@product-operating-model

# Step 2: install this overlay
/plugin install pom-insurance@product-operating-model

# Step 3: apply the overlay to your POM repo
/pom-insurance-init /path/to/your/pom-repo
```

If you have not bootstrapped a POM repo yet, run `/pom-bootstrap /path/to/your/pom-repo --rubric=generic` first, then `/pom-insurance-init`. The overlay can also be applied at bootstrap time via `/pom-bootstrap --rubric=insurance` (pom-core handles the routing).

## Compatibility

- Co-installable with other industry overlays — if your portfolio crosses lines (e.g., insurance + fintech for an insurtech-lending hybrid), install both. Each `init` skill operates on independent files and prompts on conflicts.
- The generic baseline standards from `pom-core` stay in place. This overlay extends, doesn't replace.

## License

MIT — see [../../LICENSE](../../LICENSE).
