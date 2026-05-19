# `/pom-council` Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship `/pom-council` — a consultative skill that runs role-perspective agents (PM/PD/TL, optionally SM/PO) in parallel against a draft Discovery item, synthesises their cross-role tensions, and writes an append-only Council artifact co-located with the parent DISC.

**Architecture:** Thin orchestrator skill (`pom-council/SKILL.md`) fans out to the existing `pom-trio` agent (extended with a `council_mode` flag) for each role, then to a new `pom-council-synthesizer` agent for tension narration. The synthesizer is guard-railed against emitting verdict tokens; the orchestrator strips them as defense-in-depth. Output is a single timestamped markdown file at `products/<product>/discovery-backlog/<disc-slug>.council-<ISO-ts>.md`. Two new validator rules (R3.11 ERROR for filename + frontmatter, R3.12 WARN for 4/4 ✅ DISCs without a Council) enforce shape without creating a gate.

**Tech Stack:** Markdown skill instructions for Claude Code, the existing `pom-trio` Sonnet agent, a new Sonnet synthesizer agent, the existing `pom-validator` rule loader.

**Spec:** [docs/superpowers/specs/2026-05-19-pom-council-design.md](../specs/2026-05-19-pom-council-design.md)

---

## File map

| Action | Path | Responsibility |
|---|---|---|
| Create | `plugins/pom-core/tests/council.test.md` | 15-scenario behavior contract for the skill |
| Modify | `plugins/pom-core/tests/validate.test.md` | Append 4 R3.11 + R3.12 scenarios |
| Create | `plugins/pom-core/methodology/workflows/council.md` | Orchestration logic referenced by the SKILL |
| Create | `plugins/pom-core/agents/pom-council-synthesizer.md` | New Sonnet agent: synthesizes role outputs with anti-softening guardrails |
| Modify | `plugins/pom-core/agents/pom-trio.md` | Add `council_mode: true` input contract that returns structured JSON per role |
| Modify | `plugins/pom-core/methodology/references/validation-rules.md` | Add R3.11 and R3.12 rows to the R3 table |
| Modify | `plugins/pom-core/methodology/workflows/validate.md` | Add R3.11 and R3.12 check logic in the R3 section |
| Create | `plugins/pom-core/skills/pom-council/SKILL.md` | Thin wrapper that attaches the workflow doc |
| Modify | `plugins/pom-core/README.md` | Add `pom-council` to skills table; add `pom-council-synthesizer` to agents table; bump counts |
| Modify | `CHANGELOG.md` | Add `[Unreleased] v0.3 in progress` section with the Council entry |

**Commit cadence:** 8 atomic commits, one per task (Tasks 9 + 10 share a final commit since the README and CHANGELOG are coupled documentation updates).

**Conventional-commit prefix throughout:** `feat(pom-council):` for new files, `feat(validator):` for the rule additions, `feat(pom-trio):` for the agent extension, `docs(v0.3):` for README/CHANGELOG.

---

## Task 1: Write the skill behavior contract

**Files:**
- Create: `plugins/pom-core/tests/council.test.md`

Reference contract style: [tests/promote-to-backlog.test.md](../../../plugins/pom-core/tests/promote-to-backlog.test.md). Each scenario uses **Setup / Invocation / Expected outcome / Pressure points** subsections. POM tests are spec contracts (not executable); a passing scenario means an agent reading the contract can walk it through the implementation and confirm each assertion.

- [ ] **Step 1: Write the file with all 15 scenarios from spec §7.1.**

Use this Scenario A as the canonical template and follow the same shape for every row in the spec's §7.1 table. Inline the full text for Scenario A; for B–O, use the spec's table column "Asserts" verbatim as the Expected outcome and add at least one Pressure point that names a failure mode unique to that scenario.

```markdown
# Test: `pom-council`

> Scenarios validate the contract in [docs/superpowers/specs/2026-05-19-pom-council-design.md](../../../docs/superpowers/specs/2026-05-19-pom-council-design.md). POM tests are spec contracts; "running" them is a human/agent walkthrough against the implementation.

## Scenario A: Happy path, default roles

**Setup:** A POM repo with a Discovery item at `products/policyholder-experience/discovery-backlog/DISC-2026-05-policyholder-portal.md`. The DISC body references one runway ADR (`runway/ADR-0003-claims-api-rate-limit.md`) that exists. No prior Council files for this DISC.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal`

**Expected outcome:**
- Skill resolves the DISC by glob `products/*/discovery-backlog/DISC-2026-05-policyholder-portal.md` (exact match, exit 1 if ambiguous).
- Skill reads the DISC frontmatter + body, the cited ADR's content, and (if any industry overlay is installed) the overlay's Q4 framing file.
- Skill dispatches three `pom-trio` invocations in parallel — `role=pm`, `role=pd`, `role=tl` — each with `council_mode: true` and the assembled context payload.
- Skill waits for all three role outputs. Each must be valid JSON matching the per-role schema (see Pressure points). Abort on any schema failure with **no file written**.
- Skill dispatches `pom-council-synthesizer` with `{disc_context, role_outputs[]}`.
- Skill strips any forbidden tokens from the synthesizer's `synthesis_prose` (✅, 🟡, ❌, "should promote", "ready for gate", "promote", "block", "do not promote"). WARNs to stderr if any are stripped.
- Skill computes cached counts: `tensions_flagged` (length of synthesizer `tensions[]`) and `low_confidence_count` (count of `low` cells across all role Q1–Q4 entries).
- Skill mints UTC ISO-8601 timestamp formatted `YYYY-MM-DDTHH-MM-SSZ` (colons replaced with dashes).
- Skill writes `products/policyholder-experience/discovery-backlog/disc-2026-05-policyholder-portal.council-<ts>.md` atomically (write-temp-then-rename). On collision (same UTC second), append `-2`, `-3`, etc.
- File frontmatter contains: `id`, `disc`, `ran_at`, `roles: [pm, pd, tl]`, `synthesizer: pom-council-synthesizer@1`, `overlay: <slug-or-none>`, `tensions_flagged`, `low_confidence_count`.
- File body order: H1 title → `## Tension Map` table with 4 rows (Q1–Q4) and 3 role columns + Tension? column → `## Synthesis` (1–3 paragraphs) → `## Per-Role Detail` with `### PM`, `### PD`, `### TL` in canonical order → `## Paste-ready Stakeholder Block` framed by dashed lines.
- Tension Map auto-bolds the outlier confidence in any row where ≥1 role is `low` AND ≥1 role is `high`. ⚠ marker in Tension? column for those rows. ⚠ count equals frontmatter `tensions_flagged`.
- CLI report echoes only the paste-ready block + the written file path. No log files written anywhere.

**Pressure points:**
- If skill dispatches roles sequentially (not parallel) → FAIL (defeats organic-disagreement design).
- If a role agent's JSON is missing any of `confidence.Q1`, `confidence.Q2`, `confidence.Q3`, `confidence.Q4` → run must abort with exit 1, no file written. Partial-role Council misrepresents the disagreement structure.
- If skill emits any verdict token (✅, 🟡, ❌, "should promote", "ready for gate", etc.) into the written file → FAIL (Council is non-gating; defense-in-depth must catch this).
- If skill writes the file before all roles complete → FAIL.
- If file is written outside the DISC's parent directory (e.g., at repo root or in a `councils/` subdir) → FAIL (co-location is the format rule).
- If timestamp uses literal `:` characters → FAIL (Windows filesystem hostile).
- If `tensions_flagged` in frontmatter disagrees with the ⚠ count in the rendered Tension Map → FAIL (cached-count global invariant).

---

## Scenario B: `--roles=all` dispatches five roles

**Setup:** Same as Scenario A.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal --roles=all`

**Expected outcome:** Five role agents dispatched (`pm, pd, tl, sm, po`), file has 5 Tension Map columns, 5 `### <Role>` sections in canonical order, frontmatter `roles: [pm, pd, tl, sm, po]`.

**Pressure points:** Role columns/sections in any order other than canonical pm→pd→tl→sm→po → FAIL (diff-friendliness across re-runs).

---

## Scenario C: Explicit role subset

**Setup:** Same as Scenario A.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal --roles=pm,tl,po`

**Expected outcome:** Three role agents dispatched (`pm`, `tl`, `po`). Tension Map has 3 columns; per-role detail in canonical order (pm → tl → po — note pd is skipped, sm is skipped). Frontmatter `roles: [pm, tl, po]`.

**Pressure points:** Tension Map renders columns in CLI-arg order (pm, tl, po) — that's correct. But if rendered as pm, po, tl (alphabetical or otherwise) → FAIL.

---

## Scenario D: Invalid role rejected

**Setup:** Same as Scenario A.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal --roles=pm,architect`

**Expected outcome:** Skill exits 1 with `"Unknown role 'architect'. Valid: pm, pd, tl, sm, po"`. No file written. No role agents dispatched.

**Pressure points:** If skill dispatches the valid role (`pm`) before discovering `architect` is invalid → FAIL (must validate args before any agent call).

---

## Scenario E: DISC not found

**Setup:** No DISC named `DISC-2026-05-nonsense` exists anywhere.

**Invocation:** `/pom-council DISC-2026-05-nonsense`

**Expected outcome:** Skill exits 1 with `"No DISC matches 'DISC-2026-05-nonsense'. Searched: products/*/discovery-backlog/*.md"`. No file written.

**Pressure points:** Silent failure or generic error → FAIL (error must name the glob).

---

## Scenario F: DISC ambiguous across products

**Setup:** Two products contain a DISC with the same slug — `products/foo/discovery-backlog/DISC-2026-05-shared.md` and `products/bar/discovery-backlog/DISC-2026-05-shared.md`.

**Invocation:** `/pom-council DISC-2026-05-shared`

**Expected outcome:** Skill exits 1, prints one line per match, mirroring `pom-explain` ambiguity output. No file written.

**Pressure points:** Skill picks one arbitrarily → FAIL (ambiguity must surface to the caller).

---

## Scenario G: Multiple runs append

**Setup:** DISC at the canonical path. One Council file already exists for it.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal` (second time, 10 seconds after the first).

**Expected outcome:** A second `.council-<ts>.md` file is created. Both remain on disk. `ls products/policyholder-experience/discovery-backlog/disc-2026-05-policyholder-portal.*` shows the DISC + 2 Councils.

**Pressure points:** If skill overwrites the prior Council, deletes it, or refuses to run because one exists → FAIL (append-only is the chosen semantics, Q5 in the spec).

---

## Scenario H: Timestamp collision (contract, not concurrency)

**Setup:** Pre-create `discovery-backlog/disc-foo.council-2026-05-19T14-32-11Z.md` on disk. Then invoke pom-council with the system clock mocked to land on the same UTC second.

**Invocation:** `/pom-council DISC-2026-05-foo`

**Expected outcome:** New file is written to `disc-foo.council-2026-05-19T14-32-11Z-2.md`. Both files exist; neither is corrupt. If a third run hits the same second, suffix `-3` is used.

**Pressure points:** Skill overwrites the existing file → CATASTROPHIC FAIL. Skill aborts without retrying with a suffix → FAIL (rare but the orchestrator must not crash on a collision).

---

## Scenario I: Role agent failure aborts the run

**Setup:** DISC exists; the `pom-trio` agent for `role=pm` is mocked to return non-zero exit / error.

**Invocation:** `/pom-council DISC-2026-05-foo`

**Expected outcome:** Skill exits 1 with `"Role pm failed: <reason>. No partial Council written."` No `.council-*.md` file exists for this DISC after the run.

**Pressure points:** Skill renders a 2-role Council from the remaining successful agents → FAIL (a partial-role Council misrepresents the disagreement structure).

---

## Scenario J: Role schema violation aborts the run

**Setup:** DISC exists; the `pom-trio` agent for `role=pd` returns JSON missing the `Q3` key.

**Invocation:** `/pom-council DISC-2026-05-foo`

**Expected outcome:** Skill exits 1 with an error naming the offending role (`pd`) and the missing field (`confidence.Q3`). Includes a snippet of the bad output for debugging. No file written.

**Pressure points:** Skill renders a Council with `Q3` blank or "unknown" → FAIL (schema enforcement is part of the contract).

---

## Scenario K: Rationale truncation is soft

**Setup:** DISC exists; the `pom-trio` agent for `role=tl` returns valid JSON, but its Q2 rationale is 850 characters long.

**Invocation:** `/pom-council DISC-2026-05-foo`

**Expected outcome:** Rendered file shows TL's Q2 rationale truncated to 750 chars + `…` appended. WARN to stderr: `"Role tl rationale for Q2 exceeded 750 chars; truncated."`. Run completes; file is written.

**Pressure points:** Skill aborts on overflow → FAIL (length is a soft constraint per the design; the role expressed itself).

---

## Scenario L: Forbidden-token stripping

**Setup:** DISC exists; `pom-council-synthesizer` is mocked to emit `synthesis_prose` containing `"this DISC should be promoted ✅"`.

**Invocation:** `/pom-council DISC-2026-05-foo`

**Expected outcome:** Rendered file's `## Synthesis` section contains neither `✅` nor the phrase `"should be promoted"`. WARN to stderr: `"Synthesizer emitted forbidden token '✅'; stripped"` (and one WARN per phrase). `tensions_flagged` is still computed correctly from `tensions[]`, not from prose.

**Pressure points:** A verdict token appears anywhere in the written file → CATASTROPHIC FAIL (Council is verdict-free by design).

---

## Scenario M: Dry-run writes nothing

**Setup:** DISC exists.

**Invocation:** `/pom-council DISC-2026-05-foo --dry-run`

**Expected outcome:** Skill prints the rendered markdown + the intended filename to stdout. `ls products/<product>/discovery-backlog/` shows no new `.council-*` files after the run.

**Pressure points:** Any file write during a dry-run → CATASTROPHIC FAIL.

---

## Scenario N: Overlay framing loaded when present

**Setup:** `pom-insurance` plugin installed. DISC exists in a portfolio bootstrapped with the insurance overlay.

**Invocation:** `/pom-council DISC-2026-05-foo`

**Expected outcome:** Synthesizer's input context contains the insurance Q4 framing file content. Frontmatter records `overlay: insurance`. Q4 column header in Tension Map reads `Q4 — Ethical (insurance overlay)`.

**Pressure points:** If overlay framing is silently dropped → FAIL (industry-aware Q4 is a v0.1 feature Council must respect).

---

## Scenario O: No overlay = generic Q4

**Setup:** No industry overlay plugin installed.

**Invocation:** `/pom-council DISC-2026-05-foo`

**Expected outcome:** Run completes. Frontmatter records `overlay: none`. Q4 column in Tension Map is still populated by each role (generic Q4 framing). Q4 column header reads `Q4 — Ethical`.

**Pressure points:** Skill aborts because no overlay is installed → FAIL (generic Q4 is the documented fallback).

---

## Global invariant (asserted across all scenarios)

`tensions_flagged` in frontmatter MUST equal the ⚠ count in the rendered Tension Map. The implementation should compute both from the same source (synthesizer `tensions[]`), but the contract asserts the equivalence as the cheapest self-consistency check.

---

## Reference comparison

A Council file should match the structure shown in [docs/superpowers/specs/2026-05-19-pom-council-design.md §5](../../../docs/superpowers/specs/2026-05-19-pom-council-design.md) — with the Tension Map at the top, synthesis above per-role detail, and a paste-ready block at the bottom.
```

- [ ] **Step 2: Walk through Scenario A against the spec to confirm internal consistency.**

Open the spec doc and the test file side-by-side. Verify every assertion in Scenario A's Expected outcome traces to a spec section (architecture diagram §3, components §4, file format §5, data flow §6). No assertion should exist in the test that isn't in the spec; no spec requirement should be unrepresented in at least one of A–O.

- [ ] **Step 3: Commit.**

```bash
git add plugins/pom-core/tests/council.test.md
git commit -m "feat(pom-council): add 15-scenario behavior test contract"
```

---

## Task 2: Extend the validator test contract with R3.11 and R3.12

**Files:**
- Modify: `plugins/pom-core/tests/validate.test.md` (append four scenarios at the end of the file, before any closing reference section)

- [ ] **Step 1: Append the four new validator scenarios.**

The existing file should already follow a per-rule scenario pattern. Append these scenarios at the end:

```markdown
---

## Scenario: R3.11 — bad Council filename rejected

**Setup:** A POM repo containing `products/foo/discovery-backlog/DISC-2026-05-bar.md` AND a malformed Council file at `products/foo/discovery-backlog/disc-2026-05-bar.council-bad-timestamp.md`.

**Invocation:** `/pom-validate`

**Expected outcome:** Validator emits ERROR with rule ID `R3.11`, the file path, and the expected filename pattern `^<disc-slug>\.council-\d{4}-\d{2}-\d{2}T\d{2}-\d{2}-\d{2}Z(-\d+)?\.md$`. Validator exit code is non-zero.

**Pressure points:** Validator silently passes the file → FAIL (filename rule must enforce).

---

## Scenario: R3.11 — missing frontmatter field

**Setup:** Council file with a valid filename but frontmatter missing the `synthesizer:` key.

**Invocation:** `/pom-validate`

**Expected outcome:** Validator emits ERROR with `R3.11` naming the missing field (`synthesizer`) and the file path.

**Pressure points:** If the validator names a different missing field or reports a different rule ID → FAIL.

---

## Scenario: R3.12 — WARN when a 4/4 ✅ DISC has no Council file

**Setup:** A DISC at `products/foo/discovery-backlog/DISC-2026-05-baz.md` with all four gate questions ✅ in its Promotion Readiness table. Zero `disc-2026-05-baz.council-*.md` files exist in the same directory.

**Invocation:** `/pom-validate`

**Expected outcome:** Validator emits **WARN** (not ERROR) with rule ID `R3.12` and message: `"DISC promoted to 4/4 ✅ without any Council run; consider /pom-council before promotion"`. Validator exit code remains 0 (WARNs are non-blocking).

**Pressure points:** Validator emits ERROR instead of WARN → FAIL (Council is consultative, not gating — verdict authority must remain with `pom-discovery-gate`).

---

## Scenario: R3.12 — silent when Council exists

**Setup:** Same as R3.12 WARN scenario above, but a single valid Council file `disc-2026-05-baz.council-2026-05-15T10-00-00Z.md` exists in the same directory.

**Invocation:** `/pom-validate`

**Expected outcome:** No R3.12 finding. Validator output for this DISC is silent on Council-related concerns.

**Pressure points:** Validator emits WARN even when a Council exists → FAIL.
```

- [ ] **Step 2: Commit.**

```bash
git add plugins/pom-core/tests/validate.test.md
git commit -m "feat(validator): add R3.11/R3.12 test scenarios for Council files"
```

---

## Task 3: Write the orchestration workflow doc

**Files:**
- Create: `plugins/pom-core/methodology/workflows/council.md`

This is the doc the SKILL.md will `@`-attach. It contains the step-by-step orchestration logic that Claude executes when the skill runs. Reference style: `plugins/pom-core/methodology/workflows/promote-to-backlog.md`.

- [ ] **Step 1: Write the workflow file.**

```markdown
<purpose>
Orchestrate a parallel multi-role consultation against a draft Discovery item and produce an append-only Council artifact that surfaces cross-role tensions. Council is **strictly consultative** — it produces no verdict and gates nothing. Authority to grade a DISC against Q1–Q4 remains exclusively with `/pom-discovery-gate`.

Output: one timestamped markdown file at `products/<product>/discovery-backlog/<disc-slug>.council-<ISO-ts>.md`.
</purpose>

<process>

<step name="parse_args">
Parse:
- First positional: `$DISC_ID` (required) — the Discovery item ID (e.g., `DISC-2026-05-policyholder-portal`).
- `--roles=<list>` flag (optional, default `pm,pd,tl`) — comma-separated list or the literal `all`. Validate against `{pm, pd, tl, sm, po}`. Exit 1 if any value is unrecognised; the error message must list valid roles.
- `--dry-run` flag (optional, default false) — print rendered markdown + intended filename to stdout; perform zero file writes.

If `$DISC_ID` is missing or malformed, exit 1 with usage guidance.
</step>

<step name="resolve_disc">
Glob `products/*/discovery-backlog/<DISC_ID>.md`. Exit 1 with the searched glob in the message if zero matches. Exit 1 with one line per match if more than one — mirror `pom-explain` ambiguity output.

Record `$DISC_PATH` (full path) and `$DISC_DIR` (parent directory — this is where the Council file will be written).
</step>

<step name="read_context">
Read:
- `$DISC_PATH` (frontmatter + body).
- Every runway ADR path cited in the DISC body. Best-effort: if a cited ADR file is missing, WARN to stderr and continue with an empty slot for that ADR.
- Active industry overlay's Q4 framing file (if any overlay plugin is installed). If no overlay, use generic Q4 framing. Record `$OVERLAY_SLUG` (one of: `insurance`, `fintech`, `healthcare`, `retail`, `none`).

Assemble into a single context payload (JSON-shaped object) passed to every role agent and to the synthesizer.
</step>

<step name="dispatch_roles_parallel">
For each role in `--roles`, invoke the `pom-trio` agent **in parallel** (single message with multiple Agent tool uses) with:
- Prompt prefix: `role=<role> council_mode: true`
- Context payload from previous step
- Instruction to return JSON matching the per-role schema:
  ```json
  {
    "role": "<role>",
    "confidence": {
      "Q1": {"level": "high|medium|low", "rationale": "<paragraph, ≤750 chars>"},
      "Q2": {"level": "high|medium|low", "rationale": "<paragraph, ≤750 chars>"},
      "Q3": {"level": "high|medium|low", "rationale": "<paragraph, ≤750 chars>"},
      "Q4": {"level": "high|medium|low", "rationale": "<paragraph, ≤750 chars>"}
    },
    "open_question": "<one sentence, optional>"
  }
  ```

Wait for ALL role outputs before proceeding.
</step>

<step name="validate_role_outputs">
For each role output:
- Parse as JSON. If invalid JSON → ABORT (exit 1) naming the offending role and including a snippet of the bad output.
- Verify presence of `confidence.Q1`, `confidence.Q2`, `confidence.Q3`, `confidence.Q4`. If any missing → ABORT (exit 1) naming the offending role and missing field.
- For each `rationale` longer than 750 chars: truncate to 750 chars + `…`, WARN to stderr `"Role <role> rationale for <Qn> exceeded 750 chars; truncated."`. Do NOT abort.
- Verify `level` is one of `high`, `medium`, `low`. If not → ABORT.

If ANY role aborted, the entire run aborts. NO file is written. Council artifacts must represent the full requested role set.
</step>

<step name="dispatch_synthesizer">
Invoke the `pom-council-synthesizer` agent with `{disc_context, role_outputs[]}`. The agent's system prompt is owned by its agent file; the orchestrator passes only the structured inputs.

Expected return:
```json
{
  "tensions": [
    {
      "question": "Q2",
      "summary": "<one sentence>",
      "quoted_rationales": [
        {"role": "pm", "quote": "<verbatim from PM rationale>"},
        {"role": "tl", "quote": "<verbatim from TL rationale>"}
      ]
    }
  ],
  "synthesis_prose": "<1-3 paragraphs>"
}
```

If the synthesizer fails or returns invalid JSON → ABORT (exit 1), surface the error verbatim, no file written.
</step>

<step name="strip_forbidden_tokens">
Scan `synthesis_prose` for any of the forbidden token strings (case-insensitive):
- `✅`, `🟡`, `❌`
- `"should promote"`, `"ready for gate"`, `"do not promote"`
- standalone words: `"promote"`, `"block"`

For each match, remove the token and log a WARN to stderr: `"Synthesizer emitted forbidden token '<X>'; stripped."`. Do NOT abort — the file must never contain a verdict, but the run continues.
</step>

<step name="compute_cached_counts">
- `tensions_flagged` = length of synthesizer `tensions[]` array.
- `low_confidence_count` = count of role-question cells where `level == "low"` across ALL role outputs.

These are written into frontmatter so `pom-pulse` / `pom-status` / `pom-foresight` can scan Council files cheaply.
</step>

<step name="mint_timestamp">
Compute UTC ISO-8601 timestamp. Format `YYYY-MM-DDTHH-MM-SSZ` — note colons are replaced with dashes for Windows filesystem safety. Example: `2026-05-19T14-32-11Z`.
</step>

<step name="render_markdown">
Build the file content in this order:

1. **Frontmatter** with required fields: `id`, `disc`, `ran_at`, `roles`, `synthesizer: pom-council-synthesizer@1`, `overlay`, `tensions_flagged`, `low_confidence_count`.
2. **H1 title**: `# Council — <DISC_ID> — <YYYY-MM-DD>` (date only, not time).
3. **## Tension Map** — markdown table. Columns: `Question | <Role1> | <Role2> | ... | Tension?`. Four rows (Q1, Q2, Q3, Q4). For each row, render each role's `level` value. Apply auto-bold to the outlier cell: if a row has ≥1 `low` AND ≥1 `high`, find every cell that differs from the majority and wrap in `**...**`. Mark the row's Tension? column with `⚠ split`; otherwise `—`.

   Q4 column header reads `Q4 — Ethical` for `overlay: none`, otherwise `Q4 — Ethical (<overlay> overlay)`.

4. **## Synthesis** — render `synthesis_prose` after token-stripping.
5. **## Per-Role Detail** — one `### <Role>` subsection per role that ran, in canonical order (`pm, pd, tl, sm, po`) regardless of CLI input order. Each subsection: a one-line confidence summary `(<Q1> / <Q2> / <Q3> / <Q4>)`, then per-question rationale paragraphs. Append `**Open question (<role>):** <text>` block if the role provided one.
6. **## Paste-ready Stakeholder Block** — dashed-line-framed, deterministically generated from `tensions[]` (NOT from `synthesis_prose`). Format:

   ```
   ────────────────────────────────────────────────────────────────────
   Council ran on <DISC_ID> (<YYYY-MM-DD>, roles: <UPPERCASE_ROLE_LIST>).

   <N> tension(s) surfaced:
   • <Question> (<short-question-name>): <one-line tension summary>
   ...

   Recommend reviewing <first tension topic> with the Trio before
   running /pom-discovery-gate.

   Full council: <relative path to written file>
   ────────────────────────────────────────────────────────────────────
   ```

   If `tensions[]` is empty, the block reads: `"No tensions surfaced; roles are aligned on Q1–Q4."`
</step>

<step name="write_file">
Compute filename: `<disc-slug>.council-<timestamp>.md` where `<disc-slug>` is the DISC's filename without the `.md` extension, lowercased (per existing POM filename conventions).

Target path: `$DISC_DIR/<filename>`.

If the target path already exists on disk, append a numeric suffix to the timestamp: `<disc-slug>.council-<timestamp>-2.md`, `-3.md`, etc., until a non-existing path is found.

Write atomically: write to `<target>.tmp`, then rename to `<target>`. Rename is atomic on POSIX and on Windows when source and target are on the same filesystem (which they always are here — same directory).

If `--dry-run`: skip the write entirely. Print the rendered markdown + the intended (non-suffixed) target path to stdout.
</step>

<step name="cli_report">
On success, echo to the CLI report:
1. One line: `Wrote: <full target path>`.
2. The paste-ready stakeholder block, verbatim.

Nothing else. No log files anywhere. WARNs went to stderr during the run; the CLI report is terse and copyable.
</step>

</process>

<read_only_contract>
This skill writes EXACTLY ONE file per run: the new `<disc-slug>.council-<timestamp>.md`.

Every other artifact in the repo is read-only for this skill: DISC files, ADR files, overlay framing files, prior Council files, validator state, anything in `.pom/`. Council never mutates existing files.
</read_only_contract>
```

- [ ] **Step 2: Commit.**

```bash
git add plugins/pom-core/methodology/workflows/council.md
git commit -m "feat(pom-council): add orchestration workflow doc"
```

---

## Task 4: Write the synthesizer agent

**Files:**
- Create: `plugins/pom-core/agents/pom-council-synthesizer.md`

Reference style: `plugins/pom-core/agents/pom-validator.md` (a workflow-style agent) and `plugins/pom-core/agents/pom-discovery-auditor.md` (an audit-style agent). This is a third shape: pure synthesis with strict guardrails.

- [ ] **Step 1: Write the agent file.**

```markdown
---
name: pom-council-synthesizer
description: "Synthesizes per-role outputs from a pom-council run into a tension narrative. Reads {disc_context, role_outputs[]} and returns {tensions[], synthesis_prose}. Amplifies disagreements, never softens them. Never produces a verdict — verdict tokens are forbidden in output. Invoked only by the pom-council skill."
tools: Read
model: sonnet
color: orange
---

You are the **Council synthesizer**. You receive structured per-role outputs from a `pom-council` run and produce a single synthesis JSON that names the disagreements between roles and narrates them.

You do NOT grade. You do NOT recommend promotion. You do NOT issue verdicts. Your sole job is to make the *structure of disagreement* legible.

## Input

You receive a JSON payload:

```json
{
  "disc_context": {
    "disc_id": "DISC-YYYY-MM-<slug>",
    "disc_body": "<full markdown body of the DISC file>",
    "cited_adrs": [{"path": "...", "content": "..."}],
    "overlay_q4_framing": "<framing text, or null>"
  },
  "role_outputs": [
    {
      "role": "pm",
      "confidence": {
        "Q1": {"level": "high|medium|low", "rationale": "..."},
        "Q2": {"level": "high|medium|low", "rationale": "..."},
        "Q3": {"level": "high|medium|low", "rationale": "..."},
        "Q4": {"level": "high|medium|low", "rationale": "..."}
      },
      "open_question": "..."
    }
    // ... more roles
  ]
}
```

## Output

Return ONE JSON object, nothing else:

```json
{
  "tensions": [
    {
      "question": "Q2",
      "summary": "<one sentence: who diverges and on what>",
      "quoted_rationales": [
        {"role": "pm", "quote": "<verbatim excerpt from PM's Q2 rationale>"},
        {"role": "tl", "quote": "<verbatim excerpt from TL's Q2 rationale>"}
      ]
    }
  ],
  "synthesis_prose": "<1-3 paragraphs narrating the tension landscape>"
}
```

## Tension definition

A **tension** exists on question Qn when, across the role outputs:
- At least one role's `confidence.Qn.level` is `low`, AND
- At least one role's `confidence.Qn.level` is `high`.

Adjacent levels (`high` ↔ `medium`, or `medium` ↔ `low`) without the high↔low spread do NOT count as a tension. They go in `synthesis_prose` as nuance, not in `tensions[]`.

For every tension, include in `quoted_rationales`:
- Always: the role with `low` and the role with `high`.
- Optionally: any other role whose rationale materially informs the disagreement.

Quote rationales **verbatim**. Do not paraphrase. Trim to the most informative ~150-char excerpt if the full rationale is too long — but do not edit words.

## Hard rules

- **Amplify, do not soften.** If three roles say `high` and one says `low`, lead the synthesis with the `low`. That outlier is the signal; the consensus is the substrate. A reader should walk away knowing where the disagreement is, not feeling that "everyone basically agreed."
- **Cite by question ID** (Q1, Q2, Q3, Q4) every time. Never write fuzzy references like "the feasibility question" or "around viability."
- **Never produce a verdict.** Forbidden tokens in ANY output field, prose or otherwise:
  - The emoji `✅`, `🟡`, `❌`.
  - The phrases `"should promote"`, `"ready for gate"`, `"do not promote"`, `"block promotion"`.
  - The bare verbs `"promote"`, `"block"` (other tenses/forms are fine: "promotion of this DISC", "the blocker is...", etc.).
- **No recommendations of the form "the Trio should...".** You may write `"Recommend reviewing <topic> with the Trio before running /pom-discovery-gate"` — that is a navigational suggestion, not a verdict. Anything else that begins with "Recommend" or "Should" is forbidden.
- **Output is JSON.** Do not wrap in markdown code fences. Do not preface with explanation. The orchestrator parses your response as JSON.
- **No tensions ≠ no synthesis.** If `tensions[]` is empty, `synthesis_prose` should still narrate the alignment ("All three roles converge on high confidence across Q1–Q4..."). Do NOT recommend promotion in that case — alignment is informational only.

## Style for `synthesis_prose`

- 1–3 paragraphs, plain markdown (the orchestrator embeds it under a `## Synthesis` heading).
- Lead paragraph: the most consequential tension, with role names and the gate question.
- Optional second paragraph: secondary tensions or notable convergences.
- Optional third paragraph: a navigational suggestion (e.g., "Recommend reviewing the rate-limit constraint with the Trio before running /pom-discovery-gate") if it falls naturally out of the disagreement structure. Skip if the alignment is genuinely uneventful.

## What you must NOT do

- Do NOT score the DISC.
- Do NOT estimate whether the DISC will pass the gate.
- Do NOT comment on the DISC's quality, completeness, or maturity in your own voice. You may quote roles saying these things; you may not assert them.
- Do NOT add roles or questions that weren't in the input.
- Do NOT issue work-item suggestions (epics, features, stories). Pre-promotion artifacts have no work-item handle.

You amplify the tension. You do not resolve it. Resolution happens in the human Trio meeting that this Council exists to prepare.
```

- [ ] **Step 2: Commit.**

```bash
git add plugins/pom-core/agents/pom-council-synthesizer.md
git commit -m "feat(pom-council): add synthesizer agent with anti-softening guardrails"
```

---

## Task 5: Extend `pom-trio` with `council_mode`

**Files:**
- Modify: `plugins/pom-core/agents/pom-trio.md`

The existing `pom-trio` agent produces a markdown review. `council_mode: true` switches it to a structured JSON output instead — same lens, different shape. We add a new section to the agent file describing this mode; the existing behavior is untouched.

- [ ] **Step 1: Append a "Council mode" section after the existing "Output Format" section.**

Find the `## Hard rules` section in `plugins/pom-core/agents/pom-trio.md` and insert this new section **immediately before** it:

```markdown
## Council mode (structured JSON output)

If the calling prompt contains the literal string `council_mode: true`, switch to structured-JSON output. The role lens is unchanged; the output shape is different.

### Council-mode input

The caller passes a context payload including the DISC body, cited ADR contents, and (if applicable) industry overlay Q4 framing. Read all of it — your job is to express ONE role's confidence on the four gate questions given this context.

### Council-mode output

Return ONE JSON object, nothing else. No markdown fences, no preface:

```json
{
  "role": "<your-role-code>",
  "confidence": {
    "Q1": {"level": "high|medium|low", "rationale": "<one paragraph, ≤750 chars>"},
    "Q2": {"level": "high|medium|low", "rationale": "<one paragraph, ≤750 chars>"},
    "Q3": {"level": "high|medium|low", "rationale": "<one paragraph, ≤750 chars>"},
    "Q4": {"level": "high|medium|low", "rationale": "<one paragraph, ≤750 chars>"}
  },
  "open_question": "<one sentence, OR null>"
}
```

### Confidence semantics

- **`high`** — You have enough evidence in the DISC + ADRs to vote ✅ on this question if the gate were called today. Rationale should cite the specific evidence.
- **`medium`** — Evidence is partial or mixed. You could be convinced either way with one targeted clarification. Rationale should name what would move it to high or low.
- **`low`** — Evidence is missing, contradicted, or you'd vote ❌/🟡 today. Rationale should name the specific gap or contradiction.

Your role lens still constrains which question deserves the most rigor:
- `pm`: Q2 (Viability) is your home turf — be strictest there.
- `pd`: Q1 (Desirability).
- `tl`: Q3 (Feasibility).
- `sm`: Process implications across questions; often `medium` on Q1–Q4 with sharp questions in `open_question`.
- `po`: Q3 (Feasibility) and slice integrity implications.

But you MUST express confidence on ALL FOUR questions, not just your home one — the synthesizer needs every cell to compute tensions correctly. If you lack standing on a question, choose `medium` and say so in rationale (`"My role lens has thin purchase here; deferring to <other role>."`).

### Council-mode hard rules

- **Rationale per question ≤ 750 characters.** The orchestrator will truncate (with a WARN) if you exceed; truncation is ugly. Stay under the cap.
- **Do NOT emit verdict tokens** (✅, 🟡, ❌, "should promote", "ready for gate", "promote", "block") even in rationale. The synthesizer downstream is also guardrailed against these, and the orchestrator strips them — but you should not produce them in the first place.
- **One paragraph per rationale.** No bullet lists, no headers, no nested structure.
- **`open_question` is optional but valuable.** Use it for a single, specific question the DISC doesn't currently address that would materially shift one of your confidence levels.
- **Output JSON only.** No markdown fences, no preface, no trailing commentary. The orchestrator parses your full response as JSON.

In council mode, the structured-JSON output replaces the markdown review format entirely. Do not produce both.
```

- [ ] **Step 2: Commit.**

```bash
git add plugins/pom-core/agents/pom-trio.md
git commit -m "feat(pom-trio): add council_mode for structured JSON output"
```

---

## Task 6: Add R3.11 and R3.12 to the validation rules table

**Files:**
- Modify: `plugins/pom-core/methodology/references/validation-rules.md`

- [ ] **Step 1: Append two rows to the R3 table.**

Find the R3 table (the one ending with `R3.10 | Pod composition shows 2–7 members + SM + PO | WARN if outside range`) and add two rows immediately after R3.10:

```markdown
| R3.11 | Council files match `<disc-slug>.council-YYYY-MM-DDTHH-MM-SSZ(-N)?.md` AND contain required frontmatter (`id`, `disc`, `ran_at`, `roles`, `synthesizer`, `tensions_flagged`, `low_confidence_count`). The `disc` field must point to an existing DISC in the same directory. | ERROR |
| R3.12 | A DISC with all four Promotion Readiness questions ✅ AND zero `<disc-slug>.council-*.md` siblings in the same directory should consider running `/pom-council` before promotion. | WARN |
```

- [ ] **Step 2: Bump the ruleset version.**

At the bottom of the file, the Versioning section reads `This rule-set is **v0.1.0**`. Change it to:

```markdown
This rule-set is **v0.2.0**. Material changes require a CHANGELOG entry and a version bump. Skill `pom-validate` reports the rule-set version it ran against.

### v0.2.0 changes

- Added R3.11 (ERROR): Council file structure and frontmatter requirements.
- Added R3.12 (WARN): Council-recommended nudge for 4/4 ✅ DISCs without a Council run.
```

- [ ] **Step 3: Commit.**

```bash
git add plugins/pom-core/methodology/references/validation-rules.md
git commit -m "feat(validator): add R3.11/R3.12 rules for Council files, bump ruleset to v0.2.0"
```

---

## Task 7: Add R3.11 and R3.12 check logic to the validator workflow

**Files:**
- Modify: `plugins/pom-core/methodology/workflows/validate.md`

- [ ] **Step 1: Append check logic for R3.11 and R3.12.**

Find the `### R3 — Products` section in `methodology/workflows/validate.md` (it currently ends with `- **R3.10** (WARN): Pod composition...`). Add two new check definitions immediately after R3.10:

```markdown
- **R3.11** (ERROR): For each file in `products/*/discovery-backlog/` matching the glob `*.council-*.md`:
  - Verify filename matches `^<disc-slug>\.council-\d{4}-\d{2}-\d{2}T\d{2}-\d{2}-\d{2}Z(-\d+)?\.md$` where `<disc-slug>` is the lowercased filename of an actual DISC sibling (e.g. `disc-2026-05-foo` for `DISC-2026-05-foo.md`). If the slug prefix does not match any sibling DISC file, ERROR with rule R3.11 and message `"Council file <path> has no matching DISC sibling"`.
  - Parse frontmatter. Verify required keys: `id`, `disc`, `ran_at`, `roles`, `synthesizer`, `tensions_flagged`, `low_confidence_count`. For each missing key, ERROR with rule R3.11 and message `"Council file <path> missing required frontmatter: <key>"`.
  - Verify `disc` field value matches the actual sibling DISC's `id` frontmatter. If not, ERROR with `"Council file <path> disc field '<value>' does not match sibling DISC id '<actual>'"`.
- **R3.12** (WARN): For each DISC file in `products/*/discovery-backlog/` whose Promotion Readiness table shows 4/4 ✅:
  - Glob the same directory for `<disc-slug>.council-*.md` files.
  - If zero matches, WARN with rule R3.12 and message `"DISC <DISC-ID> promoted to 4/4 ✅ without any Council run; consider /pom-council before promotion"`. The DISC file path goes in the finding's `location` field.
  - Determining 4/4 ✅ uses the same Promotion Readiness parse logic that R7.4 already uses. Do not duplicate logic — reference the existing helper.
```

- [ ] **Step 2: Commit.**

```bash
git add plugins/pom-core/methodology/workflows/validate.md
git commit -m "feat(validator): wire R3.11/R3.12 check logic in validate workflow"
```

---

## Task 8: Write the `pom-council` skill wrapper

**Files:**
- Create: `plugins/pom-core/skills/pom-council/SKILL.md`

This is the thin wrapper that registers the skill with Claude Code and `@`-attaches the workflow doc.

- [ ] **Step 1: Write the SKILL.md file.**

```markdown
---
name: pom-council
description: "Use when a draft Discovery item needs cross-role critique before the human Trio meeting. Dispatches PM/PD/TL (default) or all five Trio roles in parallel via pom-trio in council mode, then runs pom-council-synthesizer to narrate the tensions. Writes an append-only Council file co-located with the parent DISC. Strictly consultative — produces no verdict, gates nothing, never mints work items."
argument-hint: "<DISC-ID> [--roles=pm,pd,tl|all] [--dry-run]"
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
  - Agent
---

<objective>
Produce a structured Council artifact for a draft Discovery item by running the existing Trio role-perspective agents (PM, PD, TL, and optionally SM, PO) in parallel and synthesising their per-question confidence (high / medium / low on Q1–Q4) into a single timestamped markdown file that surfaces *where the roles disagree*.

**Council is consultative.** It does not produce a verdict (✅/🟡/❌) on any gate question. It does not gate promotion. Authority to grade a DISC against Q1–Q4 remains exclusively with `/pom-discovery-gate`. Authority to promote remains with `/pom-promote-to-backlog`. Council exists to make the disagreement structure legible before the human Trio meets.

**Output:** one file at `products/<product>/discovery-backlog/<disc-slug>.council-<ISO-timestamp>.md`. Append-only: re-running on the same DISC creates a new file; existing Councils are never modified.

**Default roles:** `pm,pd,tl` (the classical Trio). Use `--roles=all` to include SM and PO; use `--roles=<list>` to specify explicitly.

**After this command:**
- Read the paste-ready stakeholder block from the CLI report; paste it into the Trio's Teams/email thread.
- Open the written Council file before the Trio meeting; lead the meeting with the Tension Map.
- If a high tension is surfaced that the DISC body doesn't currently address, update the DISC and optionally re-run `/pom-council` to capture the updated picture.
- When ready, run `/pom-discovery-gate` to formally grade the DISC.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/council.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/four-question-gate.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
</execution_context>

<process>
Execute the workflow end-to-end.

Preserve all guarantees:
- Parallel dispatch of role agents (NOT sequential).
- Abort the entire run on any role failure or schema violation — never write a partial Council.
- Strip forbidden verdict tokens from synthesizer output before rendering.
- Co-locate the Council file with the parent DISC.
- Append-only: never modify or delete prior Council files.
- Read-only on every artifact except the new Council file.
</process>
```

- [ ] **Step 2: Walk through `tests/council.test.md` Scenario A against the SKILL + workflow + agents and confirm each assertion has a corresponding step in the workflow.**

This is the spec-style equivalent of running tests. Open `council.test.md` Scenario A's Expected outcome section and `methodology/workflows/council.md`'s `<process>` section side-by-side. For each Expected outcome bullet, locate the workflow step that satisfies it. If any bullet has no matching step, fix the workflow (not the test).

- [ ] **Step 3: Commit.**

```bash
git add plugins/pom-core/skills/pom-council/SKILL.md
git commit -m "feat(pom-council): add skill wrapper attaching council workflow"
```

---

## Task 9: Update the pom-core README

**Files:**
- Modify: `plugins/pom-core/README.md`

- [ ] **Step 1: Add `pom-council` to the Skills table.**

Find the skills table (the one with the heading `## Skills (15)` and rows listing `pom-bootstrap`, `pom-ingest-use-case`, etc.). Add a new row in the order that matches the workflow lifecycle — between `pom-discovery-gate` and `pom-promote-to-backlog`:

```markdown
| `pom-council` | Run PM/PD/TL (optionally SM/PO) in parallel against a draft DISC; synthesise tensions; write a Council file. Strictly consultative — produces no verdict, gates nothing | Yes (products/<x>/discovery-backlog/`*.council-*.md`) |
```

The `## Skills (15)` heading already underreports (the table currently has 19 rows). Leave the heading alone — it's a known discrepancy that v0.2 didn't fix and v0.3 doesn't need to. The CHANGELOG entry will note the addition.

- [ ] **Step 2: Add `pom-council-synthesizer` to the Workflow agents table.**

Find the section titled `### Workflow agents (3) — heavy lifting, protect main context`. Update the section heading from `(3)` to `(4)` and add a new row to the table:

```markdown
| `pom-council-synthesizer` | Synthesises pom-council role outputs into a tension narrative. Amplifies disagreements; forbidden from producing verdicts. Invoked only by pom-council. |
```

- [ ] **Step 3: Commit.**

```bash
git add plugins/pom-core/README.md
git commit -m "docs(v0.3): add pom-council and pom-council-synthesizer to README tables"
```

---

## Task 10: Update CHANGELOG.md

**Files:**
- Modify: `CHANGELOG.md`

- [ ] **Step 1: Add a new `[Unreleased] — v0.3 in progress` section above the existing `[Unreleased] — v0.2 in progress` section.**

If v0.2 has already shipped by the time Council lands, the v0.2 `[Unreleased]` header should have been renamed to `[0.2.0] — YYYY-MM-DD` separately; do not modify it here. This task only adds the v0.3 section.

Insert immediately after `## [Unreleased] — v0.3 in progress` heading (creating it if not present):

```markdown
## [Unreleased] — v0.3 in progress

### Added
- **`/pom-council` skill — parallel multi-role consultation on draft Discovery items.** Dispatches `pom-trio` for each role (default: PM/PD/TL; opt-in to add SM/PO via `--roles=all` or explicit list) in parallel, all under the new `council_mode: true` input contract that returns structured JSON (per-question confidence high/medium/low + ≤750-char rationale + optional open question). Then dispatches the new `pom-council-synthesizer` agent, which is guardrailed against softening disagreements and forbidden from emitting verdict tokens (✅/🟡/❌/"should promote"/etc.). Output is a single timestamped markdown file `<disc-slug>.council-<ISO-ts>.md` co-located with the parent DISC, containing a Tension Map table (per-question role confidences side-by-side, outlier auto-bolded), synthesis prose, per-role detail in canonical order, and a paste-ready stakeholder block. Multiple runs stacked append-only; the orchestrator handles same-second collisions by appending `-2`, `-3` to the timestamp suffix. Strictly consultative — Council never grades, gates, or emits work-item-shaped artifacts. Test contract in `tests/council.test.md` (15 scenarios) plus the spec at `docs/superpowers/specs/2026-05-19-pom-council-design.md`.
- **Validator R3.11 (ERROR) and R3.12 (WARN).** R3.11 enforces Council filename pattern + required frontmatter (`id`, `disc`, `ran_at`, `roles`, `synthesizer`, `tensions_flagged`, `low_confidence_count`); R3.12 emits a WARN when a 4/4 ✅ DISC has zero Council siblings, nudging trios toward Council without blocking promotion. Ruleset bumped from v0.1.0 to v0.2.0. Test contract extended in `tests/validate.test.md`.
- **`pom-trio` agent council mode.** Adds a `council_mode: true` input flag that switches the agent from markdown review output to structured JSON suitable for the Council orchestrator. Existing markdown-review behaviour is unchanged for callers that do not set the flag.

### Rationale
v0.2 closed the visibility loop (cockpit, paste-ready blocks, ADO sync). v0.3 closes the **decision-quality** loop. Trio meetings today are "read the DISC out loud → react"; Council pre-stages cross-role disagreements so meetings start where the friction actually is. The strictly-consultative posture protects existing gate authority — `/pom-discovery-gate` remains the sole grader, and `/pom-promote-to-backlog` remains the sole promoter.
```

- [ ] **Step 2: Commit.**

```bash
git add CHANGELOG.md
git commit -m "docs(v0.3): add CHANGELOG entry for pom-council, R3.11/R3.12, pom-trio council_mode"
```

---

## Task 11: Dogfood verification (release gate, not code)

**Files:** None modified; this is a manual verification step that gates the v0.3.0 tag.

- [ ] **Step 1: Run `/pom-council` on a real insurance DISC from Josh's current team.**

Pick a DISC currently in active shaping (not yet promoted). Run:

```text
/pom-council <DISC-ID>
```

Open the resulting Council file. Compare against three signal questions from spec §7.3:

- [ ] **Step 2: Confirm Tension Map signal quality.**

Does the Tension Map match the trio's lived sense of where they disagree? If three roles say `high` on Q3 but the team knows TL has reservations, that's a signal-quality bug — the role agent isn't surfacing what the human TL would surface. Document any mismatch; consider if the `pom-trio` role prompt needs tuning.

- [ ] **Step 3: Confirm the synthesizer is not softening disagreements.**

Read the Synthesis section. Does it lead with the outlier confidence, or does it begin with "the roles broadly agree on..."? The latter is the failure mode the design explicitly guards against. If softening appears, strengthen the synthesizer's system prompt with a concrete example of the failure.

- [ ] **Step 4: Use the paste-ready block in a real meeting.**

Copy the dashed-frame block. Paste it into the Trio's Teams/email thread or PR description before the next Trio meeting. Observe whether trio members reference it during the meeting. Adoption signal: do they bring up tensions Council surfaced that they hadn't already named?

- [ ] **Step 5: If all three signals pass, mark v0.3.0 as ready to tag.**

This does NOT commit anything. The dogfood result is observational. If signals fail, return to the relevant component file (synthesizer prompt, role prompt, workflow) and iterate. Do not tag v0.3.0 with an unresolved dogfood failure.

---

## Self-review

Ran against the spec sections:

| Spec section | Plan task(s) | Coverage |
|---|---|---|
| §1 Purpose | Tasks 3 (workflow), 8 (skill objective) | ✓ |
| §2 Scope (in scope) | Tasks 1–10 | ✓ |
| §2 Scope (out of scope: deferred work-item emission, no sequential dialogue, no auto-invocation) | Task 8 SKILL objective wording + Task 3 workflow process | ✓ |
| §3 Architecture | Task 3 workflow encodes the diagram step-by-step | ✓ |
| §4.1 SKILL.md | Task 8 | ✓ |
| §4.2 synthesizer agent | Task 4 | ✓ |
| §4.3 pom-trio council mode | Task 5 | ✓ |
| §4.4 validator rules R-NN/R-MM (now R3.11/R3.12) | Tasks 6 (rules table), 7 (check logic), 2 (test scenarios) | ✓ |
| §5 file format | Task 3 step `render_markdown` + Task 1 Scenario A | ✓ |
| §6 data flow & error handling | Task 3 workflow steps + Task 1 Scenarios E–L | ✓ |
| §7.1 council.test.md 15 scenarios | Task 1 | ✓ |
| §7.2 validator scenarios (now in validate.test.md, not separate file) | Task 2 | ✓ (deviation from spec noted: existing file extended per v0.2 convention) |
| §7.3 dogfood gate | Task 11 | ✓ |
| §8 future considerations | No tasks (intentionally deferred) | n/a |

Placeholder scan: no TBD/TODO/"fill in details"/"similar to Task N" lines. Every Task includes the full content (file body or row content) that the engineer must write.

Type consistency: `council_mode: true` (JSON-style) used consistently across Tasks 3, 4, 5, 8. `pom-council-synthesizer@1` version string consistent across spec §4.2 and Tasks 3, 4. Frontmatter field names (`id`, `disc`, `ran_at`, `roles`, `synthesizer`, `overlay`, `tensions_flagged`, `low_confidence_count`) consistent across Tasks 1, 3, 6.

One acknowledged spec deviation: spec §7 called for a separate `tests/validate-council.test.md`; plan Task 2 extends the existing `tests/validate.test.md` instead, matching the v0.2 pattern ("Test contract extended in `tests/promote-to-backlog.test.md`"). The spec is correct in intent; the plan corrects the file location to match the established convention.

---

## Execution

Plan complete and saved to `docs/superpowers/plans/2026-05-19-pom-council.md`.

Two execution options:

1. **Subagent-Driven (recommended)** — Fresh subagent per task, review between tasks, fast iteration. Best when tasks are mostly independent (Tasks 1, 2, 4, 6 have minimal coupling; Tasks 3, 5, 7, 8 build on each other).
2. **Inline Execution** — Execute tasks in this session using executing-plans, with checkpoints between commits.

Which approach?
