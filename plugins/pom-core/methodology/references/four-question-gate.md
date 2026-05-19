# The Four-Question Discovery Gate

A Discovery item promotes to the Product Backlog **only when all four questions are fully answered**. This is the load-bearing quality gate of the methodology.

## The questions

### 1. Desirable (PD owns)

**Do customers want it?** Validated through journey-mapping, interviews, or other PD-led discovery. Persona pain alone is *signal*, not validation.

### 2. Viable (PM owns)

**Does it advance the product's outcomes?** Outcomes must be defined in the product's `product.md` first. Viability is a check against those outcomes — not against the use case in isolation.

### 3. Feasible (TL owns)

**Can pods build it on the existing runway?** If not, identify the runway investment required first (and decide whether that's in scope or a separate Discovery item). POC evidence counts but is not sufficient — production-grade feasibility includes integration, scale, and operations.

### 4. Ethical / compliant

**Does it pass the standards in `enabling/`?** PII handling, bias audit, security, regulatory considerations. The Trio runs the per-UC checklists; Compliance signs off where required.

## Status values

A Discovery file marks each question with one of:

| Marker | Meaning |
|---|---|
| ❌ | Not answered. Specific blocker named. |
| 🟡 | Partial. Some evidence; specific remaining work named. |
| ✅ | Fully answered. Promotion-eligible on this dimension. |

## Promotion criteria

**ALL FOUR must be ✅** before a Discovery item promotes to the Product Backlog. No exceptions, no overrides — if an override is genuinely needed, it must be a logged decision in [`enabling/<concern>/decision-log/`](../templates/enabling/) with explicit Trio + Compliance sign-off.

## Owner accountability

| Question | Owner | What "fully answered" looks like |
|---|---|---|
| Desirable | PD | Journey map + at least 2 user conversations (interviews / shadowing) with affected persona. |
| Viable | PM | Product outcomes ratified; this item maps explicitly to at least one outcome with a stated direction. |
| Feasible | TL | Runway gaps identified and scoped (in-scope or split out); integration touch-points confirmed; SLO/SLA targets set. |
| Ethical/compliant | Trio + Compliance | Per-UC checklists in `enabling/<concern>/` completed; sign-off recorded. |

## Anti-patterns

- **PO unilaterally promotes.** The PO is part of pod execution, not Trio shaping. Promotion is a Trio decision.
- **"Mostly answered" promotion.** 3-of-4 ✅ is not promotion-eligible. The unanswered question is the future incident.
- **Skipping the gate "because we already know."** The gate is the record. If the answer is genuinely obvious, writing it down takes one sentence per question.
