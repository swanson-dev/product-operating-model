# Platform Service Rules

Platform Services are **portfolio-shared capabilities consumed by products at runtime**. Distinct from Enabling Functions, which are practices and standards (Layer 3).

## Operating principle

**Platform services are consumed, not mandated.** A product chooses to depend on a platform service because it's better than rolling its own. If a platform service becomes a bottleneck or a tax, that's a defect to fix — not a policy to enforce.

## Standard areas

Four canonical areas under `platform/`:

| Area | Examples |
|---|---|
| `provisioning/` | Environment templates, infra-as-code modules, sandbox provisioning |
| `templates-generators/` | Service scaffolding, repo templates, reusable code generators |
| `governance/` | Built-in controls, policy-as-code, audit hooks |
| `data-quality/` | Lineage, quality monitoring, schema registries |

A POM repo may define additional areas if needed (e.g., `observability/`, `mlops/`). Areas are top-level under `platform/`; services live within an area.

## Service file layout

Each platform service is a directory under one of the areas:

```
platform/<area>/<service-slug>/
├── README.md           # What the service does + status
├── intake.md           # How products request use of this service
└── consumption.md      # How products integrate with this service
```

`pom-platform-add-service` scaffolds this layout.

## Required content per service

### `README.md`

- **Service name** + one-line description
- **Owner** (Pod or individual; portfolio-level if cross-product)
- **Status** (proposed / building / available / deprecated)
- **Products currently consuming** (list with link to consuming product)

### `intake.md`

- **How to request use** — process, contact, expected lead time
- **What information the platform team needs** from the consuming product

### `consumption.md`

- **Integration pattern** — code examples, configuration
- **SLAs / SLOs** — what consumers can expect (latency, uptime, throughput)
- **Failure modes & fallbacks** — what consumers should do when the service is degraded

## Shared pods

If a platform service is significant enough to warrant a dedicated pod (rather than being a side-project of a product pod), the pod lives at `platform/pods/<pod-slug>/` and references the area(s) it serves. See [`pod-composition-rules.md`](pod-composition-rules.md).

## Anti-patterns

- **Platform-as-policy** — "you must use the platform" instead of "the platform is the easy choice." Sign: products copy-paste around the platform. Fix: improve the platform, don't mandate.
- **Platform-as-archeology** — services accumulate without retirement. Sign: README says "available" but nobody is consuming. Fix: deprecate explicitly; document retirement timeline.
- **Service sprawl** — every pod's reusable utility becomes a platform service. Sign: > 30 services across 4 areas, mostly unused. Fix: promote to platform only when ≥ 2 products genuinely need it.
