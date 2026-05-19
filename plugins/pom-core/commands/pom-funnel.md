---
description: "Run one end-to-end intake funnel pass for one or more stakeholder source documents — research → ingest (score) → disposition → route (if routed) → optional Discovery gate → optional promote. Stops at human gates."
argument-hint: "<source-path-or-glob> [--rubric=auto|generic|insurance|fintech|healthcare|retail] [--gate=auto] [--dry-run]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
  - Skill
---

# /pom-funnel

Orchestrate the intake-to-promotion funnel for one or more stakeholder source documents.

## Pre-flight

1. Resolve cwd to the active POM repo. If `intake/`, `products/`, `methodology/` are missing, abort with: `Not a POM repo. Run /pom-bootstrap first.`
2. Resolve `<source-path-or-glob>` to a concrete list of source files. If the glob matches zero files, abort.
3. Determine the active WSJF rubric. If `--rubric=auto` (default), read `intake/scoring/wsjf-rubric.md` to detect the calibrated rubric (or fall back to generic). If an explicit rubric is passed, use it for this run.

## Per-source pipeline

For each resolved source file, in sequence:

### Stage 1 — Research
Dispatch the `pom-uc-researcher` agent with the source path. Receive back a structured dossier (problem, stakeholders, outcomes, what's already true, decisions already made, open questions). Persist the dossier path for downstream stages but do NOT write a UC yet.

### Stage 2 — Score (writes intake/use-case-backlog/UC-NNNN-<slug>.md)
Invoke the `pom-ingest-use-case` skill, passing the dossier as input. The skill prompts the user for WSJF scores using the active rubric. If `--dry-run`, the skill *proposes* scores and prints them but does NOT write the UC file; otherwise it writes the file and returns the new UC-ID.

**Human gate:** scoring requires user confirmation per dimension. Do not auto-confirm.

### Stage 3 — Disposition
Invoke the `pom-disposition` skill on the new UC. The skill prompts the user for kill/park/route/split.

**Human gate:** disposition decision is the user's. Do not auto-pick.

### Stage 4 — Route (only if disposition was `route`)
If the disposition was `route` with a target product:
1. Check whether the target product exists in `products/<slug>/`. If not, halt with: `Target product '<slug>' does not exist. Run /pom-new-product <slug> first, then re-run /pom-funnel.`
2. If the product exists, invoke `pom-route-to-discovery` to land a DISC reference. Capture the new DISC path.

### Stage 5 — Discovery gate (only if `--gate=auto` or user opts in)
If `--gate=auto` is set, OR the user is asked and opts in:
Invoke `pom-discovery-gate` on the new DISC. The skill walks the four questions and records evidence in the shaping log.

**Human gate:** each question requires evidence, not assertions. Do not auto-fill.

### Stage 6 — Promote (only if DISC reaches 4/4 ✅)
If the Discovery gate finished with all four questions ✅, ask the user whether to promote now. If yes, invoke `pom-promote-to-backlog`. If no, leave the DISC at 4/4 ✅ for explicit later promotion.

## Stop conditions (per source)

The pipeline stops cleanly at the first of:
- User cancels at any human gate
- Routing target product missing (stage 4)
- Disposition was kill / park / split (stages 5–6 don't apply)
- Discovery gate finished with any ❌ or 🟡 (stage 6 doesn't apply)
- Any underlying skill returns an error

## After the run

Print a one-paragraph summary:

```
Processed N source(s) → M UC(s) written, K dispositioned, J routed, I promoted.
Stopped: <list of stop reasons, with source filename and stage>.
```

Then a paste-ready stakeholder update line, e.g.:

```
Stakeholder update: 3 stakeholder inputs ingested this week. 2 routed to Payments product
(DISC-2026-05-card-expiry, DISC-2026-05-receipts-pdf); 1 killed (UC-0014, scope was
covered by existing capability). 1 promoted to PB (PB-2026-05-card-expiry-notifications).
```

## Guardrails

- Always confirm at every human gate. This command's value is in *sequencing*, not in *deciding*.
- `--dry-run` propagates only to stage 2 (scoring). Stages 3+ require committed UC files and are skipped under dry-run.
- If a glob matches >5 sources at once, ask the user to confirm before starting — funnel runs are intentionally serial and a 20-source batch is a 2-hour session.
- If any stage fails, do NOT skip ahead to the next stage. Stop and report which stage failed and why.

## References

- `${CLAUDE_PLUGIN_ROOT}/methodology/workflows/ingest-use-case.md`
- `${CLAUDE_PLUGIN_ROOT}/methodology/references/wsjf-formula.md`
- `${CLAUDE_PLUGIN_ROOT}/methodology/references/four-question-gate.md`
- `${CLAUDE_PLUGIN_ROOT}/methodology/references/disposition-rules.md`
