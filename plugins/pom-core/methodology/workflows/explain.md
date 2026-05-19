<purpose>
Produce a single-line live summary of any POM artifact, given its ID or path. Read-only.

The line is paste-ready: it can be dropped into Teams, email, or a PR comment without editing.
</purpose>

<process>

<step name="parse_args">
Parse `$ARGUMENTS`:
- First positional: `$ARTIFACT` (required) — either an artifact ID (`UC-0042`, `DISC-2026-05-claims-doc-ai`, `ADR-0003`, etc.) or a full path to a markdown file.

If `$ARTIFACT` missing, abort with:
```
Provide an artifact ID or path.

Examples:
  /pom-explain UC-0042
  /pom-explain DISC-2026-05-claims-doc-ai
  /pom-explain products/claims/runway/ADR-0003-drift-detection.md
```
</step>

<step name="precondition_checks">
Verify cwd is a POM repo (`intake/` and `products/` both exist).

If `$ARTIFACT` looks like a path (contains `/` or `\` or ends in `.md`), use it directly. Skip to `parse_artifact`.
</step>

<step name="resolve_id_to_path">
Detect artifact type from the prefix (case-insensitive). For each type, glob the canonical location:

- `UC-NNNN` → `intake/use-case-backlog/$ARTIFACT-*.md`
- `DISP-` → `intake/dispositions/$ARTIFACT.md` (exact filename match — DISP filenames already include the date and UC-ID)
- `DISC-` → `products/*/discovery-backlog/$ARTIFACT*.md`
- `PB-` → `products/*/product-backlog/$ARTIFACT*.md`
- `ADR-NNNN` → `products/*/runway/$ARTIFACT-*.md`
- `DEC-YYYY-MM-DD-` → `enabling/*/decision-log/$ARTIFACT.md`

Expected: exactly one match. If 0 → abort with "Artifact $ARTIFACT not found." If > 1 (only possible for `ADR-NNNN` across products) → list candidates and report "$ARTIFACT is ambiguous — multiple products have this ADR number" then summarize each match one per line.
</step>

<step name="parse_artifact">
Read the resolved file. Extract from the YAML frontmatter (if any) and the leading metadata table:

**For all types:**
- Title (first `# ` heading)
- File creation date (from the filename if dated, or from git log via `git log --follow --diff-filter=A --format=%ai -- "$PATH"`; fall back to filesystem mtime)

**Type-specific fields:**

UC:
- `Status` field (e.g., `scored`, `routed → claims`, `killed`, `parked`)
- WSJF score if computed (look for `WSJF` row in the metadata table)
- Submitted-by (first row of Stakeholders table, if present)

DISP:
- `Decision` (kill / park / route / split)
- `Deciders`
- For route: receiving product
- For park: trigger condition

DISC:
- Promotion Readiness verdict (READY / NOT READY) and the four Q states (Q1/Q2/Q3/Q4 = ❌/🟡/✅)
- Trio members (read from the DISC file's metadata table, NOT from trio.md — DISC carries its own snapshot)
- If NOT READY: the first un-done item from the critical-path sequence

PB:
- Status (shaped / pulled / in-flight / shipped)
- Outcome (one-line)
- Owning pod, if assigned

ADR:
- Status (Accepted / Superseded by X / Proposed)
- Decision title (chosen option)
- Deciders (TL + others)
- Product slug (from path)

DEC:
- Concern (from path: `enabling/<concern>/`)
- Precedent scope (UC-only / product-only / portfolio-wide)
- Expiration / re-review trigger

Compute age in days from creation date to today.
</step>

<step name="compose_line">
Build the single-line summary using these templates. **Stay under 160 characters when possible** — readability over completeness.

UC:
```
$ARTIFACT: Use Case, {{status}}{{ (WSJF=N) if scored}}, submitted by {{submitter}}, {{age}}d old
```

DISP:
```
$ARTIFACT: {{decision}}{{ → product if route}}{{; trigger: ... if park}}, by {{deciders}}, {{age}}d old
```

DISC:
```
$ARTIFACT: Discovery, {{Q1}}/{{Q2}}/{{Q3}}/{{Q4}}{{ blocked on <phrase> if NOT READY}}, Trio {{trio}}, {{age}}d old
```

PB:
```
$ARTIFACT: Product Backlog, {{status}}, {{outcome}}{{, pod: <pod> if assigned}}, {{age}}d old
```

ADR (include product slug for disambiguation):
```
$ARTIFACT ({{product}}): {{title}}, {{status}}, TL {{tl}}, {{age}}d old
```

DEC:
```
$ARTIFACT: {{concern}}, scope: {{precedent scope}}, re-review: {{trigger}}, {{age}}d old
```

Substitute `unknown` for any field that cannot be parsed — do not invent values.
</step>

<step name="report">
Print the single composed line. No headers, no blank lines, no dashes, no surrounding prose.

If the artifact was ambiguous (rare — only ADR-NNNN across products), print one line per match.
</step>

</process>

<guardrails>
- **Read-only.** Never write, edit, or modify any file.
- **Single line only.** No multi-line reports, no headers, no commentary.
- **Never invent values.** If a field cannot be parsed, print `unknown`. Do not guess.
- **No follow-up suggestions.** The line is the value. The user knows where to look next.
- Age calculation uses git log creation date if available; else filesystem mtime. Filename dates (DISP / DISC / PB / DEC) are advisory, not authoritative for age.
</guardrails>

<success_criteria>
- [ ] Single line output, under 160 chars where possible
- [ ] Type-appropriate fields surfaced (status, owners, age, blocker)
- [ ] `unknown` used for unparseable fields — no invented values
- [ ] No file mutations
- [ ] Ambiguous IDs (ADRs across products) handled with one line per match
</success_criteria>
