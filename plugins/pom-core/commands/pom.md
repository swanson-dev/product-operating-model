---
description: "Top-level router for the Product Operating Model toolkit — shows what's available, recommends the next step based on the current POM repo's state, and routes to the appropriate skill or command."
argument-hint: "[target-dir]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

# /pom

The `/pom` command is a router. It does NOT perform mutations on the user's POM repo; it inspects state and recommends the right next step.

## Pre-flight

1. Resolve `$TARGET_DIR` — first positional argument; if absent, use cwd.
2. Determine whether `$TARGET_DIR` is:
   - **Not a POM repo** (no `intake/`, `products/`, etc.) → recommend `/pom-bootstrap`.
   - **An empty / fresh POM repo** (bootstrapped but no UCs yet) → recommend `/pom-ingest-use-case` or `/pom-funnel`.
   - **An active POM repo** (UCs exist; some scored; products exist) → show portfolio status and the most actionable next step.

## What to show

Always print:

```
Product Operating Model — /pom
──────────────────────────────
Target: <path>
State: <not-a-pom-repo | fresh | active>

Skills available:
  /pom-bootstrap                Scaffold a new POM repo
  /pom-ingest-use-case          Score a stakeholder doc into a UC
  /pom-disposition              Kill/park/route/split a scored UC
  /pom-route-to-discovery       Land a UC in a product's Discovery
  /pom-discovery-gate           Walk the 4-question gate on a DISC
  /pom-promote-to-backlog       Promote a 4/4 ✅ DISC to PB
  /pom-spinup-product           Stand up a new product
  /pom-form-pod                 Stand up a pod for a product
  /pom-runway-add-adr           Add an ADR to a product's runway
  /pom-runway-plan              Refresh a product's runway-plan.md
  /pom-platform-add-service     Scaffold a portfolio-shared platform service
  /pom-seed-enabling-standard   Bootstrap an enabling concern
  /pom-decision-log             Append to an enabling concern's decision log
  /pom-status                   Read-only portfolio snapshot
  /pom-validate                 Read-only structural validation

Orchestrator commands:
  /pom-funnel <source>          Run ingest → score → disposition → route → gate
  /pom-new-product <slug>       Spin up product + first pod + initial runway plan
  /pom-review                   Validate + portfolio audit + ethics review

Industry overlays (install separately):
  pom-insurance, pom-fintech, pom-healthcare, pom-retail
```

## State-aware recommendation

Based on the state detected in pre-flight, end with one specific recommendation:

- **Not a POM repo:** `Run /pom-bootstrap <dir> [--rubric=generic|insurance|fintech|healthcare|retail] to scaffold.`
- **Fresh repo, no UCs:** `Run /pom-funnel <path-to-first-source> to ingest your first Use Case end-to-end, or /pom-ingest-use-case <source> if you want to stop at scoring.`
- **Active repo, has scored UCs awaiting disposition:** `N UC(s) awaiting disposition: <list>. Run /pom-disposition <UC-ID>.`
- **Active repo, has 4/4 ✅ DISC items not yet promoted:** `M DISC(s) ready to promote. Run /pom-promote-to-backlog <DISC-ID>.`
- **Active repo, has products with 0 pods:** `K product(s) without pods. Run /pom-form-pod <product> when ready to pull from PB.`
- **Active repo, no obvious next step:** `Run /pom-status for a portfolio snapshot, or /pom-review for a quality audit.`

## Guardrails

- This command is **read-only**. It must not write any files.
- If `$TARGET_DIR` doesn't exist, ask the user (AskUserQuestion) whether to bootstrap there or pick a different directory.
- Never recommend a skill that requires content the repo doesn't have (e.g., don't recommend `/pom-disposition` if no scored UCs exist).
