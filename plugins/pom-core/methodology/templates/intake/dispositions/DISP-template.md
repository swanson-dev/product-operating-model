# DISP-YYYY-MM-DD-UC-NNNN — {{decision}} → {{target}}

| Field | Value |
|---|---|
| **Disposition ID** | DISP-YYYY-MM-DD-UC-NNNN |
| **Use case** | [`../use-case-backlog/UC-NNNN-{{slug}}.md`](../use-case-backlog/) |
| **Decision date** | YYYY-MM-DD |
| **Deciders** | {{names + roles}} |
| **Decision** | **{{kill | park | route | split}}** |

## Rationale

{{why this decision — what alternatives were considered}}

## Next action

### If kill
Nothing. Use case file status set to `killed`.

### If park
- **Trigger to re-score:** {{date, event, or dependency that warrants re-evaluation}}
- Use case file status set to `parked`.

### If route
- **Receiving product:** {{product-slug}}
- **Receiving Trio:** PM={{name}}, PD={{name}}, TL={{name}}
- **Discovery file to be created at:** `products/{{product-slug}}/discovery-backlog/DISC-YYYY-MM-{{slug}}.md`
- Use case file status set to `routed → {{product-slug}}`.

### If split
- **New use cases created:** UC-NNNN, UC-NNNN, …
- **Original UC status:** `killed-by-split`
- **Split rationale per new UC:** {{briefly, why the split shape}}

## Stakeholder-facing summary (paste-ready)

> {{One paragraph the stakeholder can read. State the decision, the reason, and the next step. If there are gaps that will be closed during Discovery, name them.}}

## Notes for future calibration

- {{any insight worth capturing for future scoring or routing decisions}}
