# Products

The portfolio. Each subfolder is one product with its own Trio, backlogs, roadmap, runway, and pods.

## Portfolio index

> No products created yet. To spin one up, run `pom-spinup-product <product-slug>` or copy [`_template/`](_template/) to `<product-slug>/` and fill in the placeholders.

| Product | Trio | Pods | Status |
|---|---|---|---|
| _(none yet)_ | | | |

## Conventions

- **Folder name** — kebab-case product slug, short and stable (rename is expensive). Avoid version numbers and codenames that may change.
- **One Trio per product.** PM + PD + TL named in `<product>/trio.md`. If a person holds the same role across multiple products, that's noted in each `trio.md` — they appear in both rosters.
- **Pods belong to exactly one product** (default). If you need a Platform Pod serving multiple products, raise it for portfolio discussion — pods then live at `platform/pods/` rather than under one product.

## Product folder shape

Every product folder mirrors [`_template/`](_template/):

```
<product>/
├── README.md               # Orientation for this specific product
├── product.md              # Charter: vision, customer, outcomes, scope
├── trio.md                 # PM + PD + TL roster
├── roadmap.md              # Now / Next / Later
├── discovery-backlog/      # Opportunities being shaped (one file each)
├── product-backlog/        # Groomed items ready for pod pull (one file each)
├── runway/                 # Architectural Runway artifacts (ADRs, patterns, runway-plan)
└── pods/                   # 1+ pod folders, mirroring pods/_template/
```
