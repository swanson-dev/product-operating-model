# PB-YYYY-MM-{{slug}} — {{title}}

| Field | Value |
|---|---|
| **PB ID** | PB-YYYY-MM-{{slug}} |
| **Originating Discovery item** | [`../discovery-backlog/DISC-YYYY-MM-{{slug}}.md`](../discovery-backlog/) |
| **Promoted on** | YYYY-MM-DD |
| **Promoted by** | Trio (PM={{name}}, PD={{name}}, TL={{name}}) |
| **Current state** | shaped / pulled-by-{{pod}} / in-flight / shipped |
| **Pulled by pod** | {{pod-slug}} (filled when pulled) |

## Outcome

{{The measurable customer-facing change this item delivers. One sentence. Refers to the product's ratified outcomes in product.md.}}

## Vertical slice definition

A thin end-to-end customer-journey increment. **Not a horizontal task list.**

```
{{Describe the slice: what the user does start-to-finish, what data flows, what systems touch.
Be specific about what is and is NOT in the slice.}}
```

## Acceptance signals

How the Trio will know the slice delivered the outcome:

- {{measurable signal — e.g., "% of cohort completing X rises by Y"}}
- {{behavioral signal — e.g., "support tickets in category Z drop by W"}}
- {{operational signal — e.g., "manual step count goes from N to M"}}

## Runway prerequisites

What must exist in [`../runway/`](../runway/) *before* a pod can pull this:

- {{ADR or pattern this depends on}}
- {{integration that must be in place}}

If runway is not yet in place, this item is NOT pullable — even if scored. Link to the runway investment item.

## Out of scope

What this slice explicitly does NOT include — useful to prevent scope creep when the pod is mid-flight.

- {{exclusion}}
- {{exclusion}}

## Dependencies on other PB items

- {{PB-ID and what it must deliver first}}

## Pull history (append-only)

> Updated each time the slice is pulled by a pod.

- **YYYY-MM-DD** — pulled by {{pod-slug}}; expected ship: {{date or sprint}}
