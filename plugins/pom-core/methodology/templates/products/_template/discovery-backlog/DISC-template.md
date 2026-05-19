# DISC-YYYY-MM-{{slug}} — {{title}}

| Field | Value |
|---|---|
| **Discovery ID** | DISC-YYYY-MM-{{slug}} |
| **Originating UC** | [`../../../intake/use-case-backlog/UC-NNNN-{{slug}}.md`](../../../intake/use-case-backlog/) |
| **Routing disposition** | [`../../../intake/dispositions/DISP-YYYY-MM-DD-UC-NNNN.md`](../../../intake/dispositions/) |
| **Routed to Discovery** | YYYY-MM-DD |
| **Shaping status** | In progress — **0 / 4 questions fully answered** |

> This file is a **reference**, not a copy. The canonical source-of-truth for problem, outcomes, stakeholders, and scoring is the originating UC file.

---

## The four-question gate

A Discovery item promotes to the Product Backlog only when all four questions are fully answered. See `the pom-core plugin's methodology/references/four-question-gate.md`.

### Q1 — Desirable (PD owns)

**Status: ❌ Not validated** | 🟡 Partial | ✅ Fully answered

{{What signal exists today. What journey-mapping / interviews still need to happen. Who blocks the answer.}}

### Q2 — Viable (PM owns)

**Status: ❌ Not answerable** | 🟡 Partial | ✅ Fully answered

{{Does this advance the product's ratified outcomes? If outcomes not yet ratified, that's the blocker.}}

### Q3 — Feasible (TL owns)

**Status: ❌ Not assessed** | 🟡 Partial | ✅ Fully answered

{{Technical feasibility. Runway gaps. Integration realism. Data availability.}}

### Q4 — Ethical / compliant

**Status: ❌ Gaps** | 🟡 Partial | ✅ Fully answered

{{Which enabling standards apply. Which checklists have been run. What has Compliance signed off.}}

---

## Trio-Note gaps (if any)

{{If the originating UC's Trio Note flagged execution-readiness gaps that are NOT Discovery-blockers but must close before pod pull, list them here.}}

1. {{gap}}
2. {{gap}}

---

## Promotion readiness

| Item | Status |
|---|---|
| Q1 Desirable | ❌ / 🟡 / ✅ |
| Q2 Viable | ❌ / 🟡 / ✅ |
| Q3 Feasible | ❌ / 🟡 / ✅ |
| Q4 Ethical / compliant | ❌ / 🟡 / ✅ |
| Trio-Note gaps closed | ❌ / 🟡 / ✅ |

**Overall: {{READY | NOT READY}} for promotion to Product Backlog.**

**Smallest critical-path unblock sequence:**

```
1. {{first blocker}}
2. {{second blocker}}
N. Promote to product-backlog/
```

---

## Shaping log (append-only)

> Add dated entries as Discovery work happens. Append-only — never edit prior entries.

- **YYYY-MM-DD** — {{what happened}}
