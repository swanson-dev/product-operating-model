# `/pom-council` — Design Spec

**Date:** 2026-05-19
**Status:** Approved (brainstorming complete; pending implementation plan)
**Release target:** v0.3.0 (first of three v0.3 items; followed by validator `--fix` and `/pom-foresight`)
**Plugin:** `pom-core`
**Source strategy:** [what-do-you-think-structured-rose.md](C:\Users\CodeAssassin\.claude\plans\what-do-you-think-structured-rose.md), §4 Trio Mock Council

---

## 1. Purpose

`/pom-council` runs the existing role-perspective agents (PM, PD, TL, and optionally SM, PO) in parallel against a draft Discovery item, then synthesises their outputs into a single Council artifact that surfaces **where the roles disagree** before the humans meet.

The 10x value the strategy doc claims: "Trio meetings today are often 'read the doc out loud → react.' Council pre-stages the disagreements. Humans walk in already knowing where PM and TL diverge."

Council is **strictly consultative**. It produces no verdict, gates nothing, and mints no work items. Authority to grade a DISC against Q1–Q4 remains exclusively with `pom-discovery-gate`.

## 2. Scope

**In scope (v0.3):**
- New orchestrator skill: `pom-council` in `pom-core`
- New synthesis agent: `pom-council-synthesizer` in `pom-core`
- Extension to existing `pom-trio` agent: a `council_mode` flag producing structured JSON output
- Two new validator rules: R-NN (filename + frontmatter ERROR) and R-MM (4/4 ✅ DISC without Council WARN)
- Co-located, append-only Council files: `products/<product>/discovery-backlog/<disc-slug>.council-<ISO-ts>.md`
- Test contracts: `tests/council.test.md` (15 scenarios), `tests/validate-council.test.md` (4 scenarios)

**Out of scope (deferred or excluded):**
- Council emitting epics, features, user stories, tasks, branches, or PR bodies. **Open question for v0.4** — the boundary is intentional: pre-promotion artifacts don't have a PB-ID handle for ADO sync.
- Council producing any aggregate verdict (✅/🟡/❌). Forbidden by design and enforced by defense-in-depth (forbidden-token stripping).
- Sequential or multi-pass role dialogue. Roles run in parallel only.
- Auto-invocation of Council by other skills (e.g., gate or promote). Manual invocation only.
- Re-using or mutating existing Council files. Every run produces a new file.

## 3. Architecture

```
User invokes  ─►  /pom-council DISC-2026-05-foo  [--roles=pm,pd,tl,sm,po]
                          │
                          ▼
                  ┌───────────────────┐
                  │  pom-council      │  resolves DISC path, reads DISC frontmatter
                  │  skill (orch)     │  + body, reads runway ADRs cited by DISC,
                  └─────────┬─────────┘  reads industry overlay Q4 framing
                            │
                            │  fan-out (parallel, 3 by default, up to 5)
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
      ┌─────────────┐ ┌──────────┐ ┌──────────┐
      │ pom-trio    │ │ pom-trio │ │ pom-trio │  (existing agent,
      │  role=pm    │ │ role=pd  │ │ role=tl  │   invoked with
      │  +council   │ │+ council │ │+ council │   council_mode: true)
      │  mode flag  │ │ mode flag│ │ mode flag│
      └──────┬──────┘ └────┬─────┘ └────┬─────┘
             │             │            │
             └──────┬──────┴────────────┘  per-role JSON outputs
                    ▼ (confidence per Q1–Q4 + one-paragraph rationale)
            ┌───────────────────────────────┐
            │   pom-council-synthesizer     │  reads all role outputs,
            │   agent (NEW)                 │  renders tension narrative +
            │   guardrail prompt: AMPLIFY,  │  paste-ready block
            │   do not soften disagreements │
            └───────────────┬───────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────────┐
        │ Write file: products/<product>/           │
        │  discovery-backlog/                       │
        │  <disc-slug>.council-<ISO-ts>.md          │
        │ Frontmatter + Q1–Q4 table + per-role      │
        │ rationale + synthesis + paste block       │
        └───────────────────┬───────────────────────┘
                            │
                            ▼
                    CLI report
                    (echoes paste-ready block)
```

**What does NOT change:**
- `pom-discovery-gate` is untouched. It does not read Council files. The gate remains the sole grader.
- `pom-promote-to-backlog` is untouched. It does not check for Council files.
- All existing artifact types (UC, DISP, DISC, PB, ADR, DEC) are untouched.

The architecture deliberately isolates Council from the gate-and-promote pipeline. If Council is later dropped or replaced, no other skill breaks.

## 4. Components

### 4.1 `/pom-council` skill — `plugins/pom-core/skills/pom-council/SKILL.md`

**Invocation:**
```
/pom-council DISC-2026-05-policyholder-portal
/pom-council DISC-2026-05-policyholder-portal --roles=all
/pom-council DISC-2026-05-policyholder-portal --roles=pm,pd,tl,po
/pom-council DISC-2026-05-policyholder-portal --dry-run
```

**Args:**

| Arg | Required | Default | Notes |
|---|---|---|---|
| `<disc-id>` | yes | — | Positional. Resolved to a file via glob `products/*/discovery-backlog/<disc-id>.md` across all products. Errors if ambiguous (mirrors `pom-explain` ID-resolution behaviour). |
| `--roles=<list>` | no | `pm,pd,tl` | Comma-separated list or the literal `all`. Validates against `{pm,pd,tl,sm,po}`. |
| `--dry-run` | no | false | Zero file writes. Prints rendered markdown + intended filename to stdout. |

**Behaviour (deterministic orchestration, no LLM in the orchestrator itself):**

1. Parse args. Validate `disc-id` format and `--roles` values.
2. Resolve `<disc-id>` to a file via glob over Discovery-backlog directories. Error if missing or ambiguous.
3. Read DISC frontmatter + body. Read every runway ADR cited in the DISC body (best-effort; warn-and-continue on missing). Read the active industry overlay's Q4 framing file (if any overlay plugin is installed; otherwise generic).
4. Build a single context payload: DISC content + cited ADR contents + overlay Q4 framing.
5. Fan out to `pom-trio` for each role in `--roles`, in parallel, passing the context payload and `council_mode: true`.
6. Wait for all role outputs. Validate each against the role-output schema (§4.3). On schema mismatch from any role, abort the run with a clear error — do not partial-write.
7. Dispatch `pom-council-synthesizer` agent with all role outputs and the DISC context.
8. Receive synthesis (`tensions: []`, `synthesis_prose: ""`). Strip forbidden tokens from `synthesis_prose` (defense-in-depth — see §4.2).
9. Compute cached counts: `tensions_flagged` (length of `tensions[]`), `low_confidence_count` (count of `low` cells across the role outputs).
10. Mint UTC ISO-8601 timestamp `YYYY-MM-DDTHH-MM-SSZ` (colons replaced with dashes for Windows filesystem safety).
11. Render the markdown file (frontmatter → Tension Map → Synthesis → Per-Role Detail → Paste-ready block).
12. Write atomically (write-to-temp-then-rename) to `<disc-dir>/<disc-slug>.council-<ts>.md` where `<disc-dir>` is the resolved DISC's parent directory (i.e. `products/<product>/discovery-backlog/`). On filename collision (two simultaneous runs hitting the same UTC second), append `-2`, `-3`, etc. to the timestamp suffix.
13. Echo the paste-ready block to the CLI report.

**Read-only on:** every artifact in the repo *except* the new Council file it writes.

### 4.2 `pom-council-synthesizer` agent — `plugins/pom-core/agents/pom-council-synthesizer.md`

Single-purpose agent. Reads role outputs, produces synthesis. Critical guardrails in its system prompt:

- **Amplify disagreements, never soften them.** If three roles say "high" and one says "low," lead with the "low" — that's the signal.
- **Quote, do not paraphrase, role rationales** when calling out tensions. Reader must trust the synthesis is grounded in what the roles actually said.
- **Cite gate question by ID** (Q1/Q2/Q3/Q4) on every tension. No fuzzy references.
- **Never produce a verdict.** Forbidden tokens in output: `✅`, `🟡`, `❌`, "should promote", "ready for gate", "promote", "block", "do not promote". The orchestrator (step 8) strips any forbidden tokens that slip through and logs a WARN.

**Output schema (structured):**
```json
{
  "tensions": [
    {
      "question": "Q2",
      "summary": "<one sentence: who diverges and on what>",
      "quoted_rationales": [
        {"role": "pm", "quote": "<verbatim from PM rationale>"},
        {"role": "tl", "quote": "<verbatim from TL rationale>"}
      ]
    }
  ],
  "synthesis_prose": "<1-3 paragraphs narrating the tension landscape>"
}
```

### 4.3 `pom-trio` agent — council mode extension

Existing agent gets a `council_mode` flag in its input contract. When `council_mode: true`:

**Output must be JSON matching:**
```json
{
  "role": "pm",
  "confidence": {
    "Q1": {"level": "high|medium|low", "rationale": "<one paragraph, ≤750 chars>"},
    "Q2": {"level": "high|medium|low", "rationale": "<one paragraph, ≤750 chars>"},
    "Q3": {"level": "high|medium|low", "rationale": "<one paragraph, ≤750 chars>"},
    "Q4": {"level": "high|medium|low", "rationale": "<one paragraph, ≤750 chars>"}
  },
  "open_question": "<one sentence, optional>"
}
```

- Per-question rationale is **bounded at 750 chars** (hard cap). Forces commitment over hedging. Rationale longer than 750 chars is truncated to 750 + `…` by the orchestrator, with a WARN to stderr.
- `open_question` lets a role flag something the DISC does not address.
- All five roles (pm/pd/tl/sm/po) get council_mode in the same way — symmetric contract.

**No existing pom-trio behaviour changes** outside `council_mode: true`. Existing skills that invoke `pom-trio` are unaffected.

### 4.4 Validator rules

POM's existing validator uses `R<section>.<rule>` numbering (e.g., `R2.1`, `R3.3`). Council adds two rules; their final numbers will be assigned during implementation based on the current rule table. Provisional IDs `R-NN` and `R-MM` are placeholders here; the implementation plan will pin them to the appropriate section (likely `R3.X` since they concern product-directory contents).

- **R-NN (ERROR)** — Council filename must match `^<disc-slug>\.council-\d{4}-\d{2}-\d{2}T\d{2}-\d{2}-\d{2}Z(-\d+)?\.md$` and reside in the same directory as a DISC file with matching slug. Required frontmatter fields: `id`, `disc`, `ran_at`, `roles`, `synthesizer`, `tensions_flagged`, `low_confidence_count`. The `disc` field must point to an existing DISC file.
- **R-MM (WARN)** — A DISC with all four gate questions ✅ AND zero `.council-*.md` siblings in the same directory emits a WARN with: `"DISC promoted to 4/4 ✅ without any Council run; consider /pom-council before promotion"`. Never an ERROR — Council remains non-gating.

The validator does **not** parse Council file bodies. The cached counts in frontmatter (`tensions_flagged`, `low_confidence_count`) are what `pom-pulse` and `pom-status` consume; the body is for humans.

## 5. File format

A complete example, then the rules.

```markdown
---
id: disc-2026-05-policyholder-portal.council-2026-05-19T14-32-11Z
disc: DISC-2026-05-policyholder-portal
ran_at: 2026-05-19T14:32:11Z
roles: [pm, pd, tl]
synthesizer: pom-council-synthesizer@1
overlay: insurance
tensions_flagged: 2
low_confidence_count: 3
---

# Council — DISC-2026-05-policyholder-portal — 2026-05-19

## Tension Map

| Question | PM | PD | TL | Tension? |
|---|---|---|---|---|
| Q1 — Desirable | high | high | medium | — |
| Q2 — Viable | high | medium | **low** | ⚠ split |
| Q3 — Feasible | high | medium | **low** | ⚠ split |
| Q4 — Ethical (insurance overlay) | medium | high | high | — |

## Synthesis

PM and TL diverge sharply on **Q2 and Q3**. PM cites the broker-channel demand
signal (quoted from DISC §Stakeholders) as sufficient viability evidence; TL
counters that the existing claims-API rate limit makes the proposed slice
unbuildable inside one pod's runway without an ADR-grade decision on caching.
PD sits in the middle on both.

Recommended pre-gate action: surface the rate-limit constraint to the Trio
explicitly before grading Q3. The DISC body does not currently reference the
claims-API limit.

## Per-Role Detail

### PM (high / high / high / medium)

**Q1 — Desirable (high).** Three stakeholder interviews in DISC §Evidence and
the broker NPS dataset (cited 2026-04) make demand unambiguous.

**Q2 — Viable (high).** [...]
**Q3 — Feasible (high).** [...]
**Q4 — Ethical (medium).** [...]

### PD (high / medium / medium / high)

[four question rationales]

### TL (medium / low / low / high)

**Q3 — Feasible (low).** The slice as written assumes the policyholder-portal
can call the claims API at session-load. The claims API is rate-limited at
5 req/s per tenant (runway ADR-0003); the proposed slice would hit that
limit at ~40 concurrent sessions. A caching ADR would change my answer to high.

**Open question (TL):** Does the slice include a caching decision, or is that
deferred to runway?

## Paste-ready Stakeholder Block

────────────────────────────────────────────────────────────────────
Council ran on DISC-2026-05-policyholder-portal (2026-05-19, roles: PM/PD/TL).

Two tensions surfaced:
• Q2 (Viable): TL low-confidence — claims-API rate limit not in DISC scope
• Q3 (Feasible): TL low-confidence — caching decision required

Recommend reviewing the rate-limit constraint with the Trio before
running /pom-discovery-gate.

Full council: products/policyholder-experience/discovery-backlog/disc-2026-05-policyholder-portal.council-2026-05-19T14-32-11Z.md
────────────────────────────────────────────────────────────────────
```

**Format rules:**

- **Filename:** `<disc-slug>.council-<ISO-ts>.md` in the **same directory** as the parent DISC (i.e. `products/<product>/discovery-backlog/`). Co-location means `ls products/<product>/discovery-backlog/disc-foo.*` shows the DISC and all its Councils together.
- **Frontmatter fields (all required, validator R-NN):** `id`, `disc`, `ran_at`, `roles`, `synthesizer`, `tensions_flagged`, `low_confidence_count`. `overlay` field present when an industry overlay is installed; value is the overlay slug (`insurance`, `fintech`, `healthcare`, `retail`) or `none`.
- **Tension Map table:** always 4 rows (Q1–Q4), one column per role that ran. "Tension?" column flags any question where ≥1 role is `low` AND ≥1 role is `high` on the same question. The outlier confidence value is bolded.
- **Synthesis section:** prose from the synthesizer agent. 1–3 paragraphs. No verdict tokens (orchestrator strips them as defense-in-depth).
- **Per-Role Detail:** one `### <Role>` section per role that ran, in canonical order `pm, pd, tl, sm, po` (regardless of CLI order) for diff-friendliness across re-runs. Each: per-question rationale (≤750 chars) and one optional **Open question**.
- **Paste-ready Stakeholder Block:** dashed-line-framed, tension-first, references the file path. Mirrors the v0.2 pattern from `pom-disposition` and `pom-promote-to-backlog`. Generated deterministically by the orchestrator from `tensions[]`, not from prose.
- **No imperative directives:** Council suggests, never decides. Recommended action sentences are fine ("Recommend reviewing X with the Trio"); imperative directives are not ("Promote the DISC"). Forbidden tokens enforced.

## 6. Data flow and error handling

### 6.1 Happy path

```
1. Parse args                    →  validate disc-id, --roles values
2. Resolve DISC                  →  glob products/*/discovery-backlog/*.md, match by frontmatter id
3. Read context                  →  DISC body + cited ADR paths + overlay Q4 framing
4. Build payload                 →  single JSON object passed to each role agent
5. Fan out (parallel)            →  N pom-trio invocations with council_mode: true
6. Collect role outputs          →  wait for all; validate each against role schema
7. Dispatch synthesizer          →  pass {disc_context, role_outputs[]}
8. Receive synthesis             →  {tensions: [], synthesis_prose: ""}
9. Strip forbidden tokens        →  if synthesis_prose contains ✅/🟡/❌/etc., strip + WARN
10. Compute cached counts        →  tensions_flagged, low_confidence_count
11. Render markdown              →  frontmatter → table → synthesis → per-role → paste block
12. Mint timestamp + filename    →  UTC, ISO-8601, collision check
13. Write file                   →  single atomic write
14. Echo paste-ready block       →  CLI report only
```

### 6.2 Failure paths

| Failure | Detection | Response |
|---|---|---|
| DISC ID not found | Step 2 — glob returns nothing | Exit 1: `"No DISC matches '<id>'. Searched: products/*/discovery-backlog/*.md"`. No file written. |
| DISC ID ambiguous (multi-product) | Step 2 — glob returns >1 file | Exit 1, one line per match. Same shape as `pom-explain` ambiguity handling. No file written. |
| Cited runway ADR path missing | Step 3 — file read fails | WARN to stderr, continue with empty ADR slot. |
| No industry overlay installed | Step 3 — overlay path doesn't exist | Continue with generic Q4 framing. Frontmatter records `overlay: none`. |
| Role agent times out or errors | Step 6 — agent invocation fails | **Abort the run.** Exit 1: `"Role <pm> failed: <reason>. No partial Council written."` No file written. A partial-role Council would misrepresent the disagreement structure. |
| Role agent returns invalid JSON | Step 6 — schema validation fails | Abort, exit 1, no file written. Error names offending role and missing/invalid field, includes snippet of bad output. |
| Role agent rationale exceeds 750 chars | Step 6 — schema check | **Truncate to 750 chars + append `…`**, log WARN. Do not abort. The role expressed itself; length is a soft constraint. |
| Synthesizer fails | Step 7 — agent invocation fails | Abort, exit 1, no file written. Surface synthesizer error verbatim. |
| Synthesizer emits forbidden tokens | Step 9 — regex scan | Strip tokens, log WARN to stderr (`"Synthesizer emitted forbidden token <X>; stripped"`). Continue rendering. Defense-in-depth — the file never contains a verdict. |
| Filesystem write fails | Step 13 — IO error | Abort, exit 1, surface OS error. No partial file. |
| Timestamp collision | Step 12 — file already exists | Append `-2`, `-3`, etc. to the timestamp suffix before write. |
| `--dry-run` requested | Steps 12–13 skipped | Print rendered markdown + intended filename to stdout. Zero file writes. |

### 6.3 Idempotency and concurrency

- **Each Council is timestamped on creation.** Two invocations on the same DISC produce two files. There is no "update existing Council" path.
- **No file locks.** Two simultaneous runs on the same DISC could collide on filename if they share the same UTC-second; the suffix appender handles it. Otherwise they're independent.
- **No retries inside the orchestrator.** A failed run leaves no artifact. The user re-invokes.

### 6.4 What gets logged (and where)

- **CLI report:** paste-ready block + the file path. Terse, copyable.
- **stderr:** WARNs (truncated rationale, missing ADR reference, stripped forbidden tokens) and ERRORs (abort reasons). Never on stdout.
- **No log files written anywhere.** Council itself is the only artifact.

## 7. Testing

POM uses spec-style markdown tests (`*.test.md`). No executable framework; the tests are the contract that implementation must satisfy. Matches the v0.2 pattern (`tests/promote-to-backlog.test.md`, `tests/render.test.md`).

### 7.1 `plugins/pom-core/tests/council.test.md` — skill behaviour (15 scenarios)

| # | Scenario | Asserts |
|---|---|---|
| 1 | Happy path, default roles | DISC resolves via `products/*/discovery-backlog/` glob, PM/PD/TL dispatched in parallel, synthesizer runs, file written next to the DISC with required frontmatter, Tension Map has 3 columns, per-role detail in canonical order, paste-ready block at bottom. |
| 2 | `--roles=all` | 5 role agents dispatched, file has 5 Tension Map columns, 5 `### <Role>` sections, frontmatter `roles: [pm, pd, tl, sm, po]`. |
| 3 | Explicit role subset | `--roles=pm,tl,po` dispatches exactly those 3, others absent. Role list rendered in canonical order regardless of CLI order. |
| 4 | Invalid role rejected | `--roles=pm,architect` exits 1: `"Unknown role 'architect'. Valid: pm, pd, tl, sm, po"`. No file written. |
| 5 | DISC not found | `/pom-council DISC-nonsense` exits 1, lists glob searched, no file written. |
| 6 | DISC ambiguous (multi-product) | Exits 1, one line per match. Same shape as `pom-explain` ambiguity output. |
| 7 | Multiple runs append | Two invocations 10s apart produce two `.council-*.md` files. Both remain on disk. `ls` on the parent DISC's directory shows DISC + both Councils, co-located. |
| 8 | Timestamp collision (contract, not concurrency test) | The implementation contract states: when the orchestrator's chosen filename already exists on disk, it must append `-2`, `-3`, etc. to the timestamp suffix before writing. Spec-test scenario: pre-create `<slug>.council-<fixed-ts>.md`, invoke with the same clock value mocked, verify the new run writes to `<slug>.council-<fixed-ts>-2.md`. Neither file corrupt. |
| 9 | Role agent failure | Mock PM to fail. Run aborts: `"Role pm failed: <reason>. No partial Council written."` No `.council-*.md` file exists for that DISC. |
| 10 | Role schema violation | Mock PD to return JSON missing `Q3`. Same abort behaviour, error names offending role and missing field. |
| 11 | Rationale truncation | Mock TL with 850-char Q2 rationale. Rendered file shows `…` at char 750. WARN to stderr. Run completes; file is written. |
| 12 | Forbidden-token stripping | Mock synthesizer to emit `"this DISC should be promoted ✅"`. Rendered Synthesis section contains neither `✅` nor `"should be promoted"`. WARN to stderr. `tensions_flagged` count still correct (computed from `tensions[]`, not prose). |
| 13 | Dry-run writes nothing | `--dry-run` prints rendered markdown + filename to stdout. `ls products/<product>/discovery-backlog/` shows no new `.council-*` files. |
| 14 | Overlay framing loaded | With `pom-insurance` installed, run on a DISC. Synthesizer input contains insurance Q4 framing. Frontmatter records `overlay: insurance`. |
| 15 | No overlay = generic Q4 | Without any overlay plugin, same run completes. Frontmatter records `overlay: none`. Q4 column in Tension Map still populated. |

**Global invariant across all scenarios:** rendered Tension Map's ⚠ count must equal frontmatter `tensions_flagged`.

### 7.2 `plugins/pom-core/tests/validate-council.test.md` — validator rules (4 scenarios)

| # | Scenario | Asserts |
|---|---|---|
| 1 | R-NN bad filename rejected | File `disc-foo.council-bad-timestamp.md` exists. Validator emits ERROR with rule ID and expected pattern. |
| 2 | R-NN missing frontmatter field | Council file missing `synthesizer:`. ERROR with missing field named. |
| 3 | R-MM WARN on missing Council | DISC has `gate_status: 4/4 ✅` and zero `.council-*.md` siblings. Validator emits WARN (not ERROR) with recommendation message. |
| 4 | R-MM silent when Council exists | Same DISC + one Council file. No WARN. |

### 7.3 Dogfood gate (release verification, not a test file)

Before tagging v0.3.0, mirror the v0.2 verification pattern. Run `/pom-council` on **one real insurance DISC** from Josh's current team. Confirm:

1. The Tension Map matches the trio's lived sense of where they disagree (signal quality).
2. The synthesizer does not smooth over disagreements (the most likely failure mode).
3. The paste-ready block is actually copy-pasted into Teams during a real meeting (adoption signal).

If the dogfood Council surfaces tensions the trio *didn't* already know about, that's a strong positive signal. If it only surfaces tensions everyone already named in the room, the value is "captures what we already said" — still useful for async portfolio review, but a weaker case.

## 8. Future considerations (deferred, not in v0.3)

- **Work-item emission (v0.4 open question).** Whether Council should emit epic/feature/story/task shapes for ADO is intentionally deferred. The current boundary — work-item shape is minted at PB-promotion, never before — protects the v0.2 ADO sync invariant. Revisit once v0.3 dogfooding reveals whether trios want Council outputs to flow into ADO.
- **Foresight integration (later v0.3 item).** `/pom-foresight` (the third v0.3 item) will read Council files and surface patterns across DISCs: "Q3 (Feasible) is low-confidence in 4 of the last 6 DISCs in this product → suggests a Platform service is missing." The cached `tensions_flagged` and `low_confidence_count` frontmatter fields exist specifically to make this Foresight pass cheap.
- **Per-product overlay-aware role weighting.** Today all roles get equal weight. A future option could let an overlay declare "Q4 is owned by PO, weight their confidence 1.5x" for industries where the role-question mapping is asymmetric. Out of scope until Foresight reveals whether it's needed.
- **Sequential or multi-pass dialogue mode.** Parallel-only is the v0.3 default. If dogfooding shows roles producing genuinely independent positions misses important cross-role critique, add a `--dialogue` flag in v0.3.x that triggers a Round-2 pass.
- **Council-driven DISC revision suggestions.** Currently the synthesizer cites tensions; a future enhancement could emit a structured `recommended_disc_edits[]` array. Held back because it would push Council closer to "advisor with authority" — exactly what the verdict-free design guards against.

## 9. Decisions locked (this brainstorming session)

| # | Decision | Rationale |
|---|---|---|
| Q1 | Sidecar artifact per run (file on disk) | Matches POM's "every meaningful artifact is a file" principle; supports re-running as evidence accumulates. |
| Q2 | Per-role confidence high/medium/low on Q1–Q4; no aggregate verdict | Preserves gate-authority isolation. `pom-discovery-gate` is the sole grader. |
| Q3 | 5 role agents in parallel + 1 LLM synthesizer agent (6 calls total) | Parallel preserves organic disagreement; synthesizer narrates with guardrails. |
| Q4 | Default `pm,pd,tl`; opt-in to add `sm,po` via `--roles=all` or explicit list | "Trio Mock Council" name honoured; SM/PO add value at later DISC maturity. |
| Q5 | Multiple runs stacked, append-only: `<disc-slug>.council-<ISO-ts>.md` | Mirrors POM's append-only ethos (DISP supersession, `.pom/snapshots/`). |
| Q6 | Manual invocation; validator R-MM WARN when DISC is 4/4 ✅ but has zero Council files | Council is consultative, not gating. WARN nudges without forcing rubber-stamp Councils. |
| Q7 | Tension Map (per-question) first, then Synthesis, then Per-Role Detail, then paste-ready block | Tensions visible at a glance; Foresight can parse the table deterministically; synthesis above per-role detail prevents anchoring. |
| — | 750-char hard cap per role rationale | Forces commitment over hedging; bounds file size. Truncation is soft (warn-and-continue). |
| — | Co-location in `discovery-backlog/` (not a separate `councils/` subdir) | `git log discovery-backlog/disc-foo.*` and `ls discovery-backlog/disc-foo.*` show the complete artifact set. |
| — | Tension-map auto-bolding rule: bold the outlier when ≥1 role low AND ≥1 role high on same question | Simple, deterministic. Refine if v0.3.1 shows it's too coarse. |
| — | Work-item emission deferred to v0.4 (open question, not no) | Pre-promotion artifacts lack a PB-ID; mixing would re-couple gate authority. |
