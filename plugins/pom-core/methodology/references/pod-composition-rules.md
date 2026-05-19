# Pod Composition Rules

A Pod is the unit of delivery. Composition rules below are enforced by `pom-validate`.

## Hard constraints

- **2–7 members.** Smaller than 2 isn't a pod; larger than 7 loses the small-team properties this model relies on.
- **Exactly one SM** (Scrum Master).
- **Exactly one PO** (Product Owner).
- **Owned by exactly one product** (default). Shared pods (e.g., a Platform Pod serving multiple products) are an explicit exception requiring portfolio-level discussion and a different folder location — see "shared pods" below.

## Role boundaries

| Role | Owns | Does NOT own |
|---|---|---|
| **PO** | Pod-level prioritization within already-pulled items; running pod ceremonies; clarifying acceptance signals | Product Backlog ordering (Trio + PO Sync owns); promotion decisions (Trio owns) |
| **SM** | Process facilitation; impediment removal; ceremony cadence | Technical decisions; prioritization |
| **Members** | Cross-functional delivery: engineering, design, data, etc. | Pod-level prioritization (PO owns); ceremony facilitation (SM owns) |

## Critical guardrail

**Pod POs execute Trio decisions. They do not unilaterally re-prioritize the Product Backlog.** This boundary is the most common drift point in Product Operating Model adoption — if violated, the Trio becomes ceremonial and the methodology collapses to PO-decides-everything.

The Product Backlog can be re-ordered only during **PO Sync** (Trio + all pod POs together). Between syncs, pods pull in the order set at the last sync.

## File location

- **Product-owned pods** live at `products/<product>/pods/<pod-slug>/`.
- **Shared pods** (rare) live at `platform/pods/<pod-slug>/` and reference the products they serve in `pod.md`.

## Required files

Every pod directory must contain:

- `pod.md` — composition (members + SM + PO), what surface area the pod owns, cadence, working agreements.
- `pod-backlog/` — items pulled from the Product Backlog (one file each).
- `slices/` — vertical slices in flight or shipped (one folder per slice).

## Lifecycle

- **Forming:** pod.md created; members named; first item pulled. SM and PO assigned.
- **Active:** delivering vertical slices.
- **Dissolved:** archived to `products/<x>/pods/_archived/<pod-slug>/` with a final retrospective.

Pods should not be dissolved silently — the archive directory preserves history (members, slices shipped, lessons).
