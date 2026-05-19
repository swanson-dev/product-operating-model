<purpose>
Bootstrap a new enabling concern (Layer 3 standard) in a POM repo with v0.1 scaffolding.

Standard concerns with canonical templates: `ai-ethics`. Other common concerns: `security`, `data-quality`, `accessibility`, `coaching-metrics` — for these, a generic scaffold is generated.

Run when a UC's Discovery Q4 (Ethical/Compliant) requires a standard that doesn't yet exist in the portfolio.
</purpose>

<process>

<step name="parse_args">
Parse:
- `--concern=<slug>` (required) — kebab-case concern slug
- `--force-update` (optional, default false) — if the concern already exists, overwrite

If `--concern` is missing, use `AskUserQuestion` to prompt with options:
- **ai-ethics** — Model Cards, PII handling, bias audit framework (full template available)
- **security** — Threat modeling, security review process, secret handling
- **data-quality** — Lineage, quality monitoring, schema registry standards
- **accessibility** — WCAG conformance, screen-reader testing, accessibility review
- **coaching-metrics** — Flow metrics, DORA-style measures, coaching artifacts
- **Other** (custom slug)
</step>

<step name="validate_slug">
Validate `$CONCERN`:
- Lowercase only
- Kebab-case
- ≥ 2 characters

If invalid, report violation and suggest correction. **Stop.**
</step>

<step name="precondition_checks">
Verify cwd is a POM repo (`enabling/` directory exists; `enabling/README.md` present).

If not, abort:
```
Current directory is not a POM repo (missing enabling/).

Run `pom-bootstrap <dir>` first.
```

Check if `enabling/$CONCERN/` exists:
- If yes AND `--force-update` is not set:
  ```
  Concern 'enabling/$CONCERN/' already exists.

  Refusing to overwrite. To replace, re-run with --force-update (NOT recommended; standards files often contain organization-specific edits).

  Better: edit the existing files directly, or use `pom-decision-log --concern=$CONCERN` to log changes.
  ```
  **Stop.**
- If yes AND `--force-update` is set:
  - Use `AskUserQuestion` to confirm: "Overwrite enabling/$CONCERN/? This will replace all v0.1 standards files." (Yes / No)
  - If No, abort silently.
</step>

<step name="check_template_availability">
Check if `the pom-core plugin's methodology/templates/enabling/$CONCERN/` exists.

- **Yes** → Set `$MODE = "full"`. The full template will be copied.
- **No** → Set `$MODE = "generic"`. A minimal scaffold will be generated.
</step>

<step name="seed_full_template">
If `$MODE = "full"`:

Copy `the pom-core plugin's methodology/templates/enabling/$CONCERN/` → `enabling/$CONCERN/`. Preserve subfolder structure:
- `README.md`
- Concern-specific standards files (e.g., for `ai-ethics`: `model-card-template.md`, `pii-handling.md`, `bias-audit-framework.md`)
- `decision-log/README.md`

Replace any placeholder dates with today's date.
</step>

<step name="seed_generic_scaffold">
If `$MODE = "generic"`:

Create `enabling/$CONCERN/README.md`:

```markdown
# {{ConcernTitleCased}} — Enabling Standards

> **v0.1 scaffold — seeded YYYY-MM-DD.** Awaiting concern-specific standards. This file is a placeholder so the structure exists; the actual standards must be authored by the team that owns this concern.

## What's here

| File | Purpose | Owner |
|---|---|---|
| [decision-log/](decision-log/) | Append-only log of standards-level decisions | Product Enablement Team |

> Add standards files (checklists, frameworks, templates) as they're authored.

## Authoritative references (NOT in this folder)

These standards (when written) are scaffolding. The authoritative versions of organizational policy live elsewhere — link them here as identified:

- _Authoritative policy on $CONCERN_ — *link TBD*
- _Regulatory references_ — *link TBD*

## How a UC applies these standards

1. While shaping in Discovery, the Trio checks whether this concern applies to the UC.
2. If yes, the Trio applies whatever checklists exist in this folder (or improvises if standards are still being authored).
3. Decisions are logged in [`decision-log/`](decision-log/).

## Calibration triggers

- **After 3 UCs have applied these standards** — review and refine.
- **On material regulatory change** — review whether scaffolding needs updating.
```

Create `enabling/$CONCERN/decision-log/README.md` — copy verbatim from `the pom-core plugin's methodology/templates/enabling/ai-ethics/decision-log/README.md`.

Then use `AskUserQuestion` to ask the user:
- **Generate stub standards files** (empty `checklist.md`, `framework.md` with section headers but no content) for the team to populate later?
- **Or leave README-only** for the team to author from scratch?

If "stub", create:
- `enabling/$CONCERN/checklist.md` with a per-UC checklist scaffold (no actual checks — sections like "Data inventory", "Stakeholder sign-off")
- `enabling/$CONCERN/framework.md` with section headers (When required, What it measures, Process, Checklist)
</step>

<step name="verify">
Verify `enabling/$CONCERN/` satisfies validation rules:
- R6.2: has `README.md` AND `decision-log/`
- R6.3: decision-log README format matches the canonical format

If verification fails, report and abort.
</step>

<step name="report">
Output:

```
✅ Enabling concern '$CONCERN' seeded at enabling/$CONCERN/

{{If $MODE = "full":}}
v0.1 standards files:
- README.md
- <concern-specific files>
- decision-log/README.md

{{If $MODE = "generic" with stubs:}}
v0.1 scaffold:
- README.md (placeholder; authoritative policy links TBD)
- checklist.md (stub — team to populate)
- framework.md (stub — team to populate)
- decision-log/README.md

{{If $MODE = "generic" without stubs:}}
v0.1 scaffold:
- README.md (placeholder; authoritative policy links TBD)
- decision-log/README.md

Next steps:
1. Link authoritative organizational policies in README.md.
2. {{If generic:}} Author the concern-specific standards files.
3. Apply the per-UC checklists during Discovery Q4 of relevant UCs.
4. Log standards-level decisions in decision-log/ as they arise.
5. Re-calibrate after first 3 UCs apply the standards.
```
</step>

</process>

<guardrails>
- NEVER overwrite an existing enabling concern without `--force-update` AND explicit user confirmation.
- NEVER invent organization-specific policy content. Generic scaffolds reference where authoritative policy *should* live.
- ALWAYS create `decision-log/` subdirectory and its README (R6.2 requirement).
- For generic concerns, do NOT pretend to be domain expert — the README explicitly says "awaiting concern-specific standards."
</guardrails>

<success_criteria>
- [ ] `enabling/$CONCERN/` exists with `README.md` and `decision-log/README.md` (R6.2)
- [ ] If full template available, standards files match canonical templates
- [ ] If generic, README explicitly marked as v0.1 scaffold awaiting authorship
- [ ] `pom-validate` passes R6 rules
</success_criteria>
