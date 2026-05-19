# Platform Services (Layer 2)

Portfolio-shared services that every product and pod consumes. The point of Platform is to **lay runway under every product so vertical slices are actually possible**.

If a capability is needed by two or more products, it belongs here — not duplicated inside each product.

## Standard areas

| Concern | Examples |
|---|---|
| **provisioning/** | Environment templates, infra-as-code modules, sandbox provisioning |
| **templates-generators/** | Service scaffolding, repo templates, reusable code generators |
| **governance/** | Built-in controls, policy-as-code, audit hooks |
| **data-quality/** | Lineage, quality monitoring, schema registries |

Additional areas (e.g., `observability/`, `mlops/`) may be added if portfolio-wide.

## Adding a service

Run `pom-platform-add-service --area=<area> --service=<slug>` or copy `_service-template/` into the correct area folder.

Each service directory must contain:

- `README.md` — what the service does, owner, status, consuming products
- `intake.md` — how products request use of the service
- `consumption.md` — how products integrate (patterns, SLAs, fallbacks)

## Operating principle

Platform services are **consumed, not mandated**. A product chooses to depend on a platform service because it's better than rolling its own. If a platform service becomes a bottleneck or a tax, that's a defect to fix, not a policy to enforce.

The Product Enablement Team (Vendor Manager + Data Architect) coordinates platform investment but does not "own" the services in isolation — platform work is done by pods (sometimes dedicated platform pods, under `pods/`) and surfaced here.

For the full rules see `the pom-core plugin's methodology/references/platform-service-rules.md`.
