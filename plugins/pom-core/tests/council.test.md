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

**Setup:** A POM repo with a Discovery item at `products/policyholder-experience/discovery-backlog/DISC-2026-05-policyholder-portal.md`. The DISC body cites one runway ADR that exists. No prior Council files. Invoker wants the full five-role Council.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal --roles=all`

**Expected outcome:**
- Skill resolves the DISC by the same glob as Scenario A and assembles the same context payload.
- Skill dispatches five `pom-trio` invocations in parallel — `role=pm`, `role=pd`, `role=tl`, `role=sm`, `role=po` — each with `council_mode: true`.
- Skill waits for all five role outputs. Schema validation applies identically to every role. Any failure aborts the run with no file written.
- Skill dispatches `pom-council-synthesizer` with all five role outputs in `role_outputs[]`.
- Forbidden-token stripping, cached-count computation, timestamp minting, and atomic write all behave per Scenario A.
- File frontmatter records `roles: [pm, pd, tl, sm, po]` in that exact canonical order.
- File body's `## Tension Map` table has five role columns plus the Tension? column, with column headers in canonical order PM → PD → TL → SM → PO.
- File body's `## Per-Role Detail` section contains `### PM`, `### PD`, `### TL`, `### SM`, `### PO` in canonical order with no other role sections present.
- CLI report echoes only the paste-ready block + the written file path.

**Pressure points:**
- If role columns or per-role detail sections appear in any order other than canonical pm→pd→tl→sm→po → FAIL (diff-friendliness across re-runs depends on stable ordering).
- If skill dispatches roles sequentially → FAIL (same organic-disagreement rule as Scenario A).
- If any of the five role JSONs is schema-invalid and the skill renders a four-role Council anyway → FAIL.

---

## Scenario C: Explicit role subset `--roles=pm,tl,po`

**Setup:** A POM repo with the same DISC as Scenario A. The invoker wants Council input only from PM, TL, and PO — deliberately skipping PD and SM for this run.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal --roles=pm,tl,po`

**Expected outcome:**
- Skill validates the CLI-provided role list against the canonical set `{pm, pd, tl, sm, po}` before dispatching anything.
- Skill dispatches exactly three role agents in parallel — `pm`, `tl`, `po` — and no others.
- Skill waits for all three role outputs and proceeds through synthesizer dispatch, token stripping, cached counts, and atomic write per Scenario A.
- File frontmatter records `roles: [pm, tl, po]` — preserving the order requested at the CLI for this scenario, since the canonical order with omitted roles collapsed is `pm → tl → po`.
- Tension Map table has three role columns in canonical order PM → TL → PO (PD and SM are absent, not blank-filled).
- Per-role detail renders `### PM`, `### TL`, `### PO` in canonical order. No `### PD` or `### SM` section appears.
- CLI report echoes only the paste-ready block + the written file path.

**Pressure points:**
- If Tension Map renders columns alphabetically (e.g., PM, PO, TL) → FAIL (canonical order with gaps collapsed is the contract).
- If Tension Map renders all five canonical columns with PD and SM blank-filled → FAIL (absent roles must be absent, not empty).
- If frontmatter `roles:` includes `pd` or `sm` → FAIL (the run did not consult them).
- If skill silently expands `--roles=pm,tl,po` to include any other role → FAIL.

---

## Scenario D: Invalid role rejected

**Setup:** A POM repo with the same DISC as Scenario A. The invoker mistypes a role name, intending `tl` but passing `architect`.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal --roles=pm,architect`

**Expected outcome:**
- Skill validates the CLI role list against the canonical set before any agent dispatch.
- Skill exits 1 immediately with the error message: `"Unknown role 'architect'. Valid: pm, pd, tl, sm, po"`.
- No role agents are dispatched. No synthesizer call is made. No `.council-*.md` file is written anywhere on disk.
- stderr carries the error message; stdout is empty (no partial report).

**Pressure points:**
- If skill dispatches the valid role (`pm`) before discovering `architect` is invalid → FAIL (argument validation must precede any agent call to avoid wasted token spend and partial state).
- If the error message omits the valid-roles list → FAIL (a typo-correcting error must show the legal options).
- If skill silently drops the invalid role and proceeds with the valid subset → FAIL (must be explicit, not lenient).

---

## Scenario E: DISC not found

**Setup:** A POM repo with one or more products and discovery-backlog directories. No DISC anywhere in the repo whose filename matches the slug the invoker provides.

**Invocation:** `/pom-council DISC-2026-05-nonsense`

**Expected outcome:**
- Skill computes the resolution glob `products/*/discovery-backlog/DISC-2026-05-nonsense.md` and finds zero matches.
- Skill exits 1 immediately with the error message: `"No DISC matches 'DISC-2026-05-nonsense'. Searched: products/*/discovery-backlog/*.md"`.
- No role agents are dispatched. No synthesizer call is made. No `.council-*.md` file is written anywhere on disk.
- stderr carries the error message; stdout is empty.

**Pressure points:**
- If skill emits a generic `"file not found"` or `"DISC missing"` without naming the glob → FAIL (error must name the glob so the operator can verify their slug or directory layout).
- If skill exits 0 with an empty Council file → CATASTROPHIC FAIL (silent success on missing input).
- If skill prompts interactively to choose a similar DISC → FAIL (resolution is exact-match-or-error in v0.3).

---

## Scenario F: DISC ambiguous across products

**Setup:** A POM repo where two products each contain a discovery-backlog entry with the same slug — e.g., `products/policyholder-experience/discovery-backlog/DISC-2026-05-portal.md` and `products/agent-experience/discovery-backlog/DISC-2026-05-portal.md`. Both are valid DISCs; the slug was reused.

**Invocation:** `/pom-council DISC-2026-05-portal`

**Expected outcome:**
- Skill computes the resolution glob and finds two matches.
- Skill exits 1 with one line per match, mirroring the `pom-explain` ambiguity output shape — e.g., `"Ambiguous DISC 'DISC-2026-05-portal'. Matches:"` followed by each absolute or repo-relative file path on its own line.
- No role agents are dispatched. No synthesizer call is made. No `.council-*.md` file is written anywhere on disk.
- The error tells the operator how to disambiguate (typically by passing the product-qualified path).

**Pressure points:**
- If skill picks one match arbitrarily and runs against it → CATASTROPHIC FAIL (silent ambiguity resolution writes a Council against the wrong DISC).
- If the error message lists only one of the two matches → FAIL (operator can't choose without seeing all candidates).
- If the output deviates materially from `pom-explain`'s ambiguity shape → FAIL (cross-skill consistency is a v0.2 expectation).

---

## Scenario G: Multiple runs append

**Setup:** A POM repo with the same DISC as Scenario A. The trio invokes Council, reviews, makes no edits, then invokes Council a second time ten seconds later to capture a re-pass.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal` (run once); `/pom-council DISC-2026-05-policyholder-portal` (run again ~10s later).

**Expected outcome:**
- First invocation writes `products/policyholder-experience/discovery-backlog/disc-2026-05-policyholder-portal.council-<ts1>.md` per Scenario A's rules.
- Second invocation, ten seconds later, mints a fresh UTC timestamp `<ts2>` (distinct from `<ts1>`) and writes a second file `disc-2026-05-policyholder-portal.council-<ts2>.md` next to the first.
- After both runs, `ls products/policyholder-experience/discovery-backlog/` shows the DISC file plus both `.council-*.md` files, co-located.
- Neither Council file is modified or deleted by the second run.
- Frontmatter of each file independently records its own `ran_at`, cached counts, and synthesizer output.

**Pressure points:**
- If skill overwrites the prior Council on the second invocation → CATASTROPHIC FAIL (append-only is the chosen semantics; history would be lost).
- If skill deletes the prior Council before writing the new one → CATASTROPHIC FAIL.
- If skill refuses to run because a Council already exists → FAIL (Council is consultative and re-runnable as evidence accumulates).
- If the second file is written outside the DISC's parent directory → FAIL (co-location rule from Scenario A applies to every run).

---

## Scenario H: Timestamp collision

**Setup:** A POM repo with the same DISC as Scenario A. Pre-create `products/policyholder-experience/discovery-backlog/disc-2026-05-policyholder-portal.council-2026-05-19T14-32-11Z.md` with arbitrary valid Council content. The implementation's clock is mocked to return exactly `2026-05-19T14:32:11Z` for the next invocation.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal` (with mocked clock).

**Expected outcome:**
- Skill resolves the DISC, runs roles + synthesizer per Scenario A, and mints the timestamp string `2026-05-19T14-32-11Z`.
- Before writing, skill detects that `disc-2026-05-policyholder-portal.council-2026-05-19T14-32-11Z.md` already exists.
- Skill appends `-2` to the timestamp suffix and writes the new file as `disc-2026-05-policyholder-portal.council-2026-05-19T14-32-11Z-2.md`.
- On a hypothetical third collision in the same second, the skill would write `-3`, and so on.
- Both files exist on disk after the run; neither is corrupt; the pre-existing file is byte-identical to its pre-run state.

**Pressure points:**
- If skill overwrites the existing file → CATASTROPHIC FAIL (silent data loss on what was intended to be append-only history).
- If skill aborts with `"file exists"` rather than appending `-2` → FAIL (the contract specifies append-on-collision, not error).
- If the resulting filename contains literal `:` characters → FAIL (Windows filesystem hostility, same rule as Scenario A's timestamp format).
- If the collision suffix is something other than `-N` (e.g., `.2`, `_2`, `(2)`) → FAIL (contract specifies `-N`).

---

## Scenario I: Role agent failure aborts the run

**Setup:** A POM repo with the same DISC as Scenario A. The PM role agent is mocked to fail mid-execution (e.g., raises an exception, times out, or returns a non-zero exit code).

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal`

**Expected outcome:**
- Skill dispatches PM, PD, and TL in parallel per Scenario A.
- PD and TL return valid JSON; PM fails.
- Skill exits 1 with the error message: `"Role pm failed: <reason>. No partial Council written."` where `<reason>` is the underlying failure summary.
- No `.council-*.md` file is created for this DISC (neither a partial file nor a placeholder).
- stderr carries the error; stdout is empty.
- The PD and TL outputs are discarded; no caching, no retry.

**Pressure points:**
- If skill renders a two-role Council from the remaining successful agents → FAIL (every Council must reflect every role the operator asked for; a silent two-role result misrepresents the disagreement structure).
- If skill writes a placeholder file with PM marked "unavailable" → FAIL (partial Council violates Scenario A's atomicity rule).
- If skill retries the failing role automatically without operator opt-in → FAIL (v0.3 leaves retry to the operator).
- If the error message omits the failing role name or the reason → FAIL.

---

## Scenario J: Role schema violation aborts the run

**Setup:** A POM repo with the same DISC as Scenario A. The PD role agent is mocked to return well-formed JSON that omits the required `confidence.Q3` field. PM and TL return valid, complete JSON.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal`

**Expected outcome:**
- Skill dispatches PM, PD, and TL in parallel.
- All three return; skill validates each against the per-role schema.
- PD's JSON fails validation because `confidence.Q3` is missing.
- Skill exits 1 with an error message that names the offending role (`pd`), the missing field (`confidence.Q3`), and includes a short snippet of the offending JSON to aid debugging.
- No `.council-*.md` file is written. PM and TL outputs are discarded.
- stderr carries the error.

**Pressure points:**
- If skill renders the Council with `Q3` blank or filled with `"unknown"` for PD → FAIL (Q1–Q4 are load-bearing; a blank cell silently weakens the Tension Map).
- If the error names a role but not the missing field, or names the field but not the role → FAIL (operator needs both to fix the agent).
- If the JSON snippet is omitted → FAIL (debug ergonomics).
- If schema validation happens after the file is written → CATASTROPHIC FAIL (writing an invalid Council to disk corrupts the artifact set).

---

## Scenario K: Rationale truncation is soft

**Setup:** A POM repo with the same DISC as Scenario A. The TL role agent is mocked to return valid JSON in which the Q2 rationale string is 850 characters long — past the 750-char soft cap defined in spec §9.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal`

**Expected outcome:**
- Skill validates the role JSON (schema-valid) and proceeds.
- During render, skill truncates the TL Q2 rationale to 750 characters and appends a Unicode ellipsis (`…`) so the visible rationale ends at exactly 750 chars + 1 ellipsis char.
- Skill WARNs to stderr: `"Role tl rationale for Q2 exceeded 750 chars; truncated."` Exactly one WARN per truncated field.
- The run completes; the `.council-*.md` file is written per Scenario A's rules.
- All other rationales (within the cap) appear at full length.

**Pressure points:**
- If skill aborts the run on overflow → FAIL (length is a soft constraint; truncation is the contract).
- If the truncated rationale does not end in `…` → FAIL (operator must see that truncation occurred at-a-glance in the file).
- If no WARN is emitted to stderr → FAIL (silent truncation loses information without notice).
- If truncation alters or summarizes the content rather than hard-cutting at 750 chars → FAIL (no LLM-summarization at render time).

---

## Scenario L: Forbidden-token stripping

**Setup:** A POM repo with the same DISC as Scenario A. The `pom-council-synthesizer` agent is mocked to emit `synthesis_prose` containing the sentence `"this DISC should be promoted ✅"` — both a verdict token (`✅`) and a verdict phrase (`"should be promoted"`).

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal`

**Expected outcome:**
- Skill receives the synthesizer output and immediately strips forbidden tokens from `synthesis_prose` before rendering.
- The rendered file's `## Synthesis` section contains neither the `✅` character nor the substring `"should be promoted"`. Stripping is character-level for emoji tokens and substring-level for phrases.
- Skill WARNs to stderr once per stripped token category — e.g., `"Stripped verdict token '✅' from synthesis_prose"` and `"Stripped verdict phrase 'should be promoted' from synthesis_prose"`.
- `tensions_flagged` in frontmatter is unchanged (computed from `tensions[]`, not from `synthesis_prose`), so the stripping does not silently desync cached counts from the Tension Map.
- The run completes; the file is written.

**Pressure points:**
- If any verdict token (`✅`, `🟡`, `❌`, `"should promote"`, `"ready for gate"`, `"promote"`, `"block"`, `"do not promote"`) appears anywhere in the written file → CATASTROPHIC FAIL (Council is non-gating; a verdict in the file would be operationally read as gate output).
- If stripping is performed but no WARN is emitted → FAIL (operator must know the synthesizer is producing forbidden output so the prompt can be corrected).
- If `tensions_flagged` changes as a side effect of stripping → FAIL (counts come from `tensions[]`, not prose).
- If the stripping pass also accidentally removes a non-verdict use of the word `"promote"` in quoted context → acceptable, but FAIL if it removes substantive content elsewhere in the file (Synthesis only).

---

## Scenario M: Dry-run writes nothing

**Setup:** A POM repo with the same DISC as Scenario A. The operator wants to preview the Council before deciding whether to keep it.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal --dry-run`

**Expected outcome:**
- Skill resolves the DISC, dispatches roles, runs the synthesizer, and assembles the rendered Council markdown identically to Scenario A.
- Skill prints the full rendered markdown to stdout, followed by a separator and the intended filename — e.g., `# would write: products/policyholder-experience/discovery-backlog/disc-2026-05-policyholder-portal.council-<ts>.md`.
- `ls products/policyholder-experience/discovery-backlog/` shows no new `.council-*` files. No temp files, no atomic-rename half-state.
- stderr carries any WARNs that would have fired in a real run (truncation, stripped tokens), so the dry-run faithfully previews stderr behavior too.

**Pressure points:**
- If any file is written to disk during a dry-run — even a temp file in the same directory — → CATASTROPHIC FAIL (dry-run is a hard read-only contract).
- If the intended filename is omitted from the stdout output → FAIL (operator can't predict where the real run would land).
- If the rendered preview differs structurally from what a real run would write → FAIL (dry-run must be a faithful preview).
- If WARNs are suppressed during dry-run → FAIL (operator needs the same diagnostics they would see in a real run).

---

## Scenario N: Overlay framing loaded when present

**Setup:** A POM repo with the same DISC as Scenario A, and the `pom-insurance` industry overlay plugin installed. The overlay ships a Q4 framing file (e.g., `plugins/pom-insurance/methodology/framings/q4-ethical.md`) that reframes the ethical question for insurance-domain DISCs.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal`

**Expected outcome:**
- Skill detects the installed overlay during context assembly and reads the overlay's Q4 framing file content.
- The synthesizer input context contains the insurance Q4 framing file content as a labelled section (so the synthesizer can cite it).
- Each role agent's payload also includes the overlay framing, so role rationales for Q4 are anchored in the insurance framing rather than generic POM phrasing.
- Frontmatter records `overlay: insurance`.
- The Tension Map's Q4 column header reads `Q4 — Ethical (insurance overlay)`, making the overlay visible in the rendered artifact.
- Everything else behaves per Scenario A.

**Pressure points:**
- If skill silently drops the overlay framing and runs with generic Q4 even though `pom-insurance` is installed → FAIL (industry-aware Q4 is a v0.1 feature Council must respect).
- If `overlay: insurance` is missing from frontmatter → FAIL (the artifact must record which framing influenced the run for Foresight to interpret correctly).
- If the Q4 column header omits the overlay suffix → FAIL (visibility rule from spec §5).
- If two overlays are installed and the skill picks one silently → FAIL (out-of-scope for v0.3 but worth a defensive error rather than ambiguous behavior).

---

## Scenario O: No overlay = generic Q4

**Setup:** A POM repo with the same DISC as Scenario A. No industry overlay plugin is installed — no `pom-insurance`, no `pom-fintech`, no `pom-healthcare`, etc.

**Invocation:** `/pom-council DISC-2026-05-policyholder-portal`

**Expected outcome:**
- Skill scans for installed overlays during context assembly and finds none.
- Skill proceeds with generic POM Q4 framing — no overlay framing file is loaded or passed to roles or the synthesizer.
- Frontmatter records `overlay: none`.
- Each role's Q4 confidence cell is populated normally using generic framing; the column is not blank.
- The Tension Map's Q4 column header reads `Q4 — Ethical` (no overlay suffix).
- Everything else behaves per Scenario A.

**Pressure points:**
- If skill aborts because no overlay is installed → FAIL (generic Q4 is the documented fallback; the overlay is optional).
- If frontmatter records `overlay:` as empty string, `null`, or missing entirely → FAIL (the field must explicitly state `none` so operators and Foresight can distinguish "no overlay installed" from "field forgotten").
- If the Q4 column appears with the `(... overlay)` suffix despite no overlay being installed → FAIL (header must match the actual context).
- If Q4 cells are left blank because the implementation assumes an overlay is required → FAIL.

---

## Global invariant (asserted across all scenarios)

`tensions_flagged` in frontmatter MUST equal the ⚠ count in the rendered Tension Map. The implementation should compute both from the same source (synthesizer `tensions[]`), but the contract asserts the equivalence as the cheapest self-consistency check.

---

## Reference comparison

A Council file should match the structure shown in [docs/superpowers/specs/2026-05-19-pom-council-design.md §5](../../../docs/superpowers/specs/2026-05-19-pom-council-design.md) — with the Tension Map at the top, synthesis above per-role detail, and a paste-ready block at the bottom.
