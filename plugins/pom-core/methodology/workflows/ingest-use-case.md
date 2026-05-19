<purpose>
Process a raw stakeholder doc (PDF, markdown, deck) into a scored Use Case Backlog entry:
- Extract problem, stakeholders, outcomes from the source
- Prompt the user for WSJF scores using the rubric's anchors as context
- Compute WSJF
- Write the UC file with all required fields
- Recommend a disposition based on the score + extracted content

Run after a stakeholder submits raw material to `use-cases/`. Output: a scored UC file ready for `pom-disposition`.
</purpose>

<process>

<step name="parse_args">
Parse `$ARGUMENTS`:
- First positional: `$SOURCE_PATH` (required) — path to the raw stakeholder doc (relative to cwd or absolute)
- `--next-uc=<NNNN>` (optional) — force a specific UC number (use sparingly; auto-increment is the default)
- `--slug=<slug>` (optional) — force a specific UC slug

If `$SOURCE_PATH` missing, ask:
```
What's the source doc path?
```
</step>

<step name="precondition_checks">
Verify cwd is a POM repo (`intake/use-case-backlog/` exists, `intake/scoring/wsjf-rubric.md` exists).

If not, abort:
```
Current directory is not a POM repo (missing intake/use-case-backlog/ or intake/scoring/wsjf-rubric.md).

Run `pom-bootstrap <dir>` first.
```

Verify `$SOURCE_PATH` exists and is readable. If not, abort with a clear "Source file not found: $SOURCE_PATH".
</step>

<step name="determine_uc_number">
List existing files in `intake/use-case-backlog/` matching `UC-NNNN-*.md`. Extract the numbers; take max + 1 as `$UC_NUMBER` (zero-padded to 4 digits).

If `--next-uc=<NNNN>` was passed, use that, but verify no file `UC-<NNNN>-*.md` already exists. If it does, abort with collision message.
</step>

<step name="read_source">
Read `$SOURCE_PATH`. Use the `Read` tool — it handles PDFs, markdown, and other text formats. For PDFs, the tool returns extracted text.

If the source is binary and not a supported format (.pdf, .md, .txt, .docx via Read), prompt the user to extract content manually.
</step>

<step name="extract_fields">
From the source content, extract heuristically:

- **Title** — usually a header or first prominent line
- **Stakeholders** — names + roles mentioned as sponsors, PM, SMEs, affected persona
- **Problem statement** — narrative description of what's broken / missing
- **Desired outcomes** — measurable targets (e.g., "50% reduction", "95% accuracy", "improve cycle time")
- **What's already true** — POC results, existing data, prior work, dependencies
- **Implied receiving product** — based on persona/systems mentioned

Build an extraction summary. Present to the user:

```
Extracted from $SOURCE_PATH:

Title: {{title}}
Stakeholders:
  - Executive sponsor: {{name or unknown}}
  - Proposed PM: {{name or unknown}}
  - SMEs: {{names or unknown}}
  - Affected persona: {{persona}}

Problem: {{2-3 sentence summary}}
Outcomes: {{bullet list of measurable targets}}
Already true: {{POC results or prior work}}
Implied receiving product: {{slug or "unclear"}}

Does this look right? Anything missing or wrong?
```

Use `AskUserQuestion` to confirm or correct. If user corrects, incorporate the corrections.
</step>

<step name="determine_slug">
If `--slug=<slug>` was provided, use it (validate kebab-case).

Otherwise, derive from the title:
- Lowercase
- Replace non-alphanumerics with hyphens
- Collapse consecutive hyphens
- Trim leading/trailing hyphens
- Truncate to ≤ 40 chars (cut at word boundary)

Show the proposed slug to the user; ask to confirm or override via `AskUserQuestion`.

Final filename: `UC-$UC_NUMBER-$SLUG.md`.
</step>

<step name="score_wsjf">
Read `intake/scoring/wsjf-rubric.md` to get the rubric anchors.

For each of the 4 dimensions in turn:
1. Display the dimension's anchor table from the rubric
2. Use `AskUserQuestion` to prompt for the score (offer: 1, 2, 3, 5, 8, 13, 21)
3. Use `AskUserQuestion` (or chat) to capture a brief rationale referencing the rubric anchor

Dimensions in order:
1. User-Business Value
2. Time Criticality
3. Risk Reduction & Opportunity Enablement
4. Job Size

Compute:
- Cost of Delay = UBV + TC + RR
- WSJF = CoD / Job Size (round to 2 decimals)

If Job Size = 21, warn the user immediately:
```
Job Size = 21 is hard-prohibited per the rubric. This UC must be SPLIT before scoring.

Suggested next step: identify 2+ smaller use cases that compose this one, then ingest each separately.
```
**Stop scoring.** Do not write the UC file. User can re-invoke after splitting.
</step>

<step name="recommend_disposition">
Based on the score and extracted content, recommend a disposition for the user to consider on the next step:

- If WSJF is high (top-third of items already in backlog, or > 1.0 in an empty backlog) → recommend **route** to the implied receiving product
- If WSJF is low → recommend **park** with a future trigger
- If the source flags clear duplication of an existing UC → recommend **kill** (split or de-dupe)
- If `pom-validate`-style checks find the source describes work that's clearly multi-quarter → recommend **split**

This is *advisory only* — `pom-disposition` makes the actual decision.
</step>

<step name="write_uc_file">
Generate the UC file from `the pom-core plugin's methodology/templates/intake/use-case-backlog/UC-template.md`:
- Replace `{{title}}` with the extracted title
- Fill ID, Submitted (today), Scored (today), Status (`scored`), Source (relative path to `$SOURCE_PATH`), Disposition (left blank, pending)
- Fill all extracted stakeholder rows
- Fill problem statement, outcomes, what's already true (from extraction)
- Fill WSJF table with user-provided scores + rationale; compute CoD and WSJF
- Fill "Why this routes to {{product}}" if the receiving product is clear; otherwise leave a "(awaiting disposition)" note
- Fill "What the receiving Trio must close in Discovery" with placeholders the Trio will fill later

Write to `intake/use-case-backlog/UC-$UC_NUMBER-$SLUG.md`.
</step>

<step name="report">
```
✅ Use Case ingested.

File: intake/use-case-backlog/UC-$UC_NUMBER-$SLUG.md

Scores:
  User-Business Value:    {{UBV}}
  Time Criticality:       {{TC}}
  Risk Reduction:         {{RR}}
  Cost of Delay:          {{CoD}}
  Job Size:               {{JS}}
  WSJF:                   {{WSJF}}

Recommended next step:
  /pom-disposition UC-$UC_NUMBER --decision={{recommended}} {{--product=<slug>}}

Notes:
- {{Any caveats: low-confidence extraction, missing stakeholders, etc.}}
```
</step>

</process>

<guardrails>
- NEVER invent WSJF scores — always prompt the user.
- NEVER invent stakeholder names that aren't in the source.
- NEVER overwrite an existing UC file.
- If extraction is ambiguous (e.g., problem statement unclear), prompt the user rather than confabulating.
- If Job Size = 21, refuse to write the UC; require split first.
- ALWAYS reference the rubric's anchor table when prompting for each dimension's score.
</guardrails>

<success_criteria>
- [ ] UC file exists with sequential ID and valid kebab-case slug
- [ ] All required fields per R2.6 populated
- [ ] WSJF scores backed by user-provided rationale
- [ ] Source file path correctly referenced
- [ ] User given a concrete next-step suggestion
</success_criteria>
