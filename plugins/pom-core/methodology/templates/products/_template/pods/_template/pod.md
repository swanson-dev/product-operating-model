# Pod: {{pod-name}}

> Copy this template to `pods/<pod-slug>/` to spin up a new pod, or run `pom-form-pod <product> <pod-slug>`.

## Composition

A pod is **2–7 cross-functional members** plus an SM and a PO. Smaller than 2 is not a pod; larger than 7 starts losing the small-team properties this model relies on.

| Role | Name | Notes |
|---|---|---|
| **Scrum Master (SM)** | _TBD_ | Process and impediment removal. |
| **Product Owner (PO)** | _TBD_ | Executes Trio decisions; runs pod ceremonies; does **not** unilaterally re-prioritize the Product Backlog. |
| **Member** | _TBD_ | _e.g., engineer, designer, data scientist_ |
| **Member** | _TBD_ | |
| _add rows as needed (≤ 7 members)_ | | |

## What this pod owns

A specific surface area of the product. Be explicit — overlap with other pods is the source of most cross-pod thrash.

- {{e.g., "Claims triage end-to-end"}}
- {{e.g., "Underwriter referral workflows"}}

## Folders

| Path | Purpose |
|---|---|
| `pod-backlog/` | Items this pod has pulled from the Product Backlog. One file each. |
| `slices/` | Vertical slices in flight or shipped. One folder per slice with its artifacts. |

## Cadence

- **Standup:** {{daily / async}}
- **Planning:** {{per pod's chosen cadence — 1 or 2 weeks}}
- **Review + demo:** {{end of cadence}}
- **Retro:** {{end of cadence}}
- **PO ↔ Trio:** continuous; pod-level prioritization within pulled items only

## Working agreements

Document anything non-obvious about how this pod chooses to operate (e.g., pairing defaults, code review SLA, on-call rotation). Keep it short and current.

- {{agreement}}
- {{agreement}}
