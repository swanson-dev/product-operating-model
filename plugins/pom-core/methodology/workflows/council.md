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
