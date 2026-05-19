# DISC-2026-05-card-expiry-notifications

| Field | Value |
|---|---|
| **Originating UC** | [UC-0001 — Reduce checkout abandonment from saved-payment expiry](../../../../intake/use-case-backlog/UC-0001-card-expiry.md) |
| **Routed to** | products/payments/ |
| **Created** | 2026-05-12 |
| **Status** | Discovery — Q1 ✅, Q2 🟡, Q3 ✅, Q4 ✅ — **not ready to promote (3.5/4)** |
| **Discovery owners** | PM: J. Park · PD: M. Acharya · TL: D. Chen |

> **Worked example.** This DISC exists in `methodology/generic/examples/` as reference content. A real Discovery item lives under `products/<product>/discovery-backlog/DISC-YYYY-MM-<slug>.md` in the user's POM repo.

## What we're investigating

Whether (and how) to notify customers proactively about saved-card expiry, with the goal of recovering the 8pp abandonment delta observed in March (see UC-0001).

## Four-question gate

### Q1 — Is it desirable? **✅**
**Evidence:**
- 12 customer interviews (sampled across active cohorts); 11 of 12 said they would prefer pro-active notification over in-checkout failure (`evidence/interviews-2026-05-08.md`).
- Concierge test (1 week, ~200 customers sent test notification via existing CRM): 73% updated their card within 7 days; 5% opted out of further notifications; 22% no action.
- Net Promoter Score impact: +9 NPS in the notified cohort vs. control over the test window.

### Q2 — Is it viable? **🟡**
**Evidence — green:**
- Revenue model: expected lift ~$160k/month based on test cohort behaviour extrapolated to full population (`evidence/business-case.md`).
- Operating cost: ~$1.2k/month additional CRM volume.

**Evidence — yellow (why not green):**
- The test used the marketing CRM, but the right long-term channel is the transactional notification system, which has a different rate-limit posture. Need confirmation from Platform that transactional volume can absorb ~340k notifications/month spread over expiry dates.
- **Blocker for ✅:** Platform sign-off on transactional channel capacity.

### Q3 — Is it feasible? **✅**
**Evidence:**
- Architecture review by TL Chen (`evidence/arch-review-2026-05-10.md`): nightly job already exists and identifies expiring cards 30/14/3 days ahead. Notification call adds one HTTP call per identified card. No new infrastructure needed.
- Effort estimate: 2 weeks for a 3-engineer pod (matches UC-0001 Job Size of 5).

### Q4 — Is it ethical, safe, responsible? **✅**
Per `methodology/generic/discovery-q4-framing.md`:

| Sub-question | Status | Evidence |
|---|---|---|
| G1 — Vulnerable users | ✅ | Notification respects existing comms preferences; users who opted out of marketing get no notification (designed deliberately to avoid pressure on financially distressed users). |
| G2 — Data minimisation | ✅ | Existing data (expiry date) — no new collection. |
| G3 — Consent & transparency | ✅ | Notification copy reviewed with Legal; opt-out one-click in the notification itself. |
| G4 — Reversibility | ✅ | Feature flag controls rollout; CRM kill switch available to ops. |
| G5 — Dual use / misuse | ✅ | Internal job, no user-controllable input. |
| G6 — Algorithmic accountability | N/A | No model. |
| G7 — Third-party | ✅ | Existing CRM vendor, already in scope; no new vendor. |
| G8 — Enabling-standard alignment | ✅ | Security-baseline S6 (logging) and S4 (data at rest) already covered; data-quality D3 (freshness SLA on the nightly job) recorded. |

## Promotion readiness

**3.5/4 — not promotable yet.** Blocker: Q2 🟡 — awaiting Platform team's confirmation that the transactional notification channel can absorb the expected monthly volume. Once confirmed, Q2 → ✅ and this item is promotable.

Once 4/4 ✅, run `/pom-promote-to-backlog DISC-2026-05-card-expiry-notifications` to create the PB item with vertical-slice definition and acceptance signals.

## Shaping log (append-only)

| Date | Author | Note |
|---|---|---|
| 2026-05-12 | J. Park (PM) | Discovery opened; interviews scheduled. |
| 2026-05-14 | M. Acharya (PD) | Q1 ✅ — interview synthesis posted. |
| 2026-05-15 | D. Chen (TL) | Q3 ✅ — arch review posted. |
| 2026-05-18 | J. Park (PM) | Q4 ✅ — Legal review done; Q2 awaiting Platform sign-off. |
