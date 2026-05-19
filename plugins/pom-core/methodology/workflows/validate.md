<purpose>
Check structural conformance of a POM repo against the rules in the plugin's bundled validation ruleset (`methodology/references/validation-rules.md`).

Reports ERROR / WARN / INFO findings grouped by rule ID with file paths. Exits non-zero if any ERROR.

Use:
- After `pom-bootstrap` to confirm clean output
- Periodically as a quality gate
- Before merging changes to a POM repo
- As a TDD harness during `pom-*` skill development
- For Voyager retro-validation in Phase 6
</purpose>

<process>

<step name="parse_args">
Parse:
- First positional: `$TARGET_DIR` (optional, defaults to cwd) — POM repo to validate. Resolve to absolute path.
- `--verbose` flag → include INFO-level findings in output (otherwise INFO is suppressed)
- `--rule=<id>` flag (optional) → only check a specific rule (e.g., `--rule=R2.5`)
- `--strict` flag (optional) → treat WARNs as ERRORs (non-zero exit)

If `$TARGET_DIR` doesn't exist, abort.
</step>

<step name="load_rules">
Read the plugin's bundled `methodology/references/validation-rules.md` (already attached via the calling SKILL.md's `<execution_context>`). Parse into a structured rule set keyed by rule ID:

```
R1.1: { description: "...", severity: ERROR, check_fn: r1_1 }
R2.5: { description: "...", severity: ERROR, check_fn: r2_5 }
...
```

Note the ruleset version (from the file's "Versioning" section, currently v0.1.0) for the report header.

If `--rule=<id>` was provided, filter to only that rule.
</step>

<step name="run_checks">
For each rule in the (possibly filtered) ruleset, perform the check against `$TARGET_DIR`.

### R1 — Repo structure
- **R1.1** (ERROR): For each required top-level dir (`intake/`, `products/`, `platform/`, `enabling/`, `methodology/`, `use-cases/`), verify it exists.
- **R1.2** (ERROR): Top-level `README.md` exists.
- **R1.3** (WARN): `methodology/` contains at least one file (not just a stub README).

### R2 — Intake
- **R2.1–R2.4** (ERROR): required files in intake/.
- **R2.5** (ERROR): Every file in `intake/use-case-backlog/` (excluding `README.md` and `UC-template.md`) matches `UC-NNNN-<slug>.md` regex.
- **R2.6** (ERROR): Each UC file contains required fields. Parse markdown for: ID, Submitted, Status, Source, WSJF scores, Disposition link. Use heuristic search for table rows or labeled lines.
- **R2.7** (ERROR): Every file in `intake/dispositions/` (excluding `README.md` and `DISP-template.md`) matches `DISP-YYYY-MM-DD-UC-NNNN.md`.
- **R2.8** (ERROR): For each disposition file, extract the referenced UC-NNNN and verify a matching UC file exists in `intake/use-case-backlog/`.
- **R2.9** (WARN): Best-effort git check — if `.git/` exists, verify disposition files have no commits modifying them after their initial creation. Skip silently if no git.
- **R2.10** (ERROR): Each disposition file contains a "Decision" field with value matching `kill | park | route | split`.

### R3 — Products
- **R3.1** (ERROR): `products/README.md` exists.
- **R3.2** (ERROR): `products/_template/` exists with all required subfolders.
- **R3.3** (ERROR): For each `products/<slug>/` (excluding `_template`), required files all present.
- **R3.4** (ERROR): `product.md` contains required fields (Vision, Target customer, Outcomes, In scope, Out of scope, Status with Created date).
- **R3.5** (WARN): `trio.md` lists PM, PD, TL. If any is `_TBD_` or `Unassigned`, WARN.
- **R3.6** (ERROR): Discovery files match `DISC-YYYY-MM-<slug>.md`.
- **R3.7** (ERROR): Each Discovery file references an existing UC.
- **R3.8** (ERROR): PB files match `PB-YYYY-MM-<slug>.md`.
- **R3.9** (ERROR): Each pod directory (excluding `_template`) has `pod.md`.
- **R3.10** (WARN): Pod composition (member count from pod.md table) is between 2-7.
- **R3.11** (ERROR): For each file in `products/*/discovery-backlog/` matching the glob `*.council-*.md`:
  - Verify filename matches `^<disc-slug>\.council-\d{4}-\d{2}-\d{2}T\d{2}-\d{2}-\d{2}Z(-\d+)?\.md$` where `<disc-slug>` is the lowercased filename of an actual DISC sibling (e.g. `disc-2026-05-foo` for `DISC-2026-05-foo.md`). If the slug prefix does not match any sibling DISC file, ERROR with rule R3.11 and message `"Council file <path> has no matching DISC sibling"`.
  - Parse frontmatter. Verify required keys: `id`, `disc`, `ran_at`, `roles`, `synthesizer`, `overlay`, `tensions_flagged`, `low_confidence_count`. For each missing key, ERROR with rule R3.11 and message `"Council file <path> missing required frontmatter: <key>"`.
  - Verify `disc` field value matches the actual sibling DISC's `id` frontmatter. If not, ERROR with `"Council file <path> disc field '<value>' does not match sibling DISC id '<actual>'"`.
- **R3.12** (WARN): For each DISC file in `products/*/discovery-backlog/` whose Promotion Readiness table shows 4/4 ✅:
  - Glob the same directory for `<disc-slug>.council-*.md` files.
  - If zero matches, WARN with rule R3.12 and message `"DISC <DISC-ID> promoted to 4/4 ✅ without any Council run; consider /pom-council before promotion"`. The DISC file path goes in the finding's `location` field.
  - Determining 4/4 ✅ uses the same Promotion Readiness parse logic that R7.4 already uses. Do not duplicate logic — reference the existing helper.

### R4 — Runway
- **R4.1** (ERROR): ADR files match `ADR-NNNN-<slug>.md`.
- **R4.2** (WARN): ADR numbering is contiguous per product.
- **R4.3** (WARN): If `runway-plan.md` exists, it references every ADR in the product's `runway/`.
- **R4.4** (ERROR): Pattern files match `PATTERN-<slug>.md`.

### R5 — Platform
- **R5.1** (ERROR): `platform/README.md` exists.
- **R5.2** (ERROR): Each service directory (excluding `_service-template`) has `README.md`.
- **R5.3** (WARN): Service README contains required fields (name, owner, status, products consuming).

### R6 — Enabling
- **R6.1** (ERROR): `enabling/README.md` exists.
- **R6.2** (ERROR): Each enabling concern dir contains `README.md` and `decision-log/`.
- **R6.3** (ERROR): Decision log files match `DEC-YYYY-MM-DD-<slug>.md`.
- **R6.4** (WARN): Best-effort git check for append-only on decision logs.

### R7 — Cross-references
- **R7.1** (ERROR): Every UC with non-empty Disposition links to an existing disposition file.
- **R7.2** (ERROR): Routed UCs (disposition = route) have a corresponding Discovery file in the named receiving product.
- **R7.3** (ERROR): Every Discovery file references an existing UC.
- **R7.4** (ERROR): Promoted Discovery items (Promotion Readiness shows 4/4 ✅) link to a PB item.
- **R7.5** (WARN): All relative markdown links resolve to existing files.

### R8 — WSJF rubric
- **R8.1** (ERROR): `intake/scoring/wsjf-rubric.md` contains the formula and Fibonacci scale references.
- **R8.2** (WARN): Rubric anchors defined for all four dimensions OR the rubric is explicitly marked as a scaffold.

### R9 — Vocabulary
- **R9.1** (INFO): PB files use "vertical slice" not "story"/"epic".
- **R9.2** (INFO): Pod files use "Pod" not "team"/"squad".

For each violation found, collect: rule ID, severity, file path, descriptive message.
</step>

<step name="format_report">
Output a structured report:

```
POM Validation Report
─────────────────────
Target:   $TARGET_DIR
Ruleset:  v0.1.0
Date:     YYYY-MM-DD HH:MM

Summary: N ERROR, M WARN, K INFO
Overall: PASS / FAIL

─────────────────────────────────────────────

ERRORS ({{N}})

[ERROR R2.5] intake/use-case-backlog/uc1.md
  Filename does not match pattern UC-NNNN-<slug>.md

[ERROR R7.3] products/claims/discovery-backlog/DISC-2026-05-bad.md
  References UC-9999, but no such UC exists in intake/use-case-backlog/

WARNINGS ({{M}})

[WARN R3.5] products/claims/trio.md
  PD and TL unassigned — Discovery items in this product cannot pass Q1/Q3 until assigned

[WARN R3.10] products/claims/pods/giant-pod/pod.md
  12 members listed; outside 2-7 range

{{If --verbose:}}
INFO ({{K}})

[INFO R9.1] products/claims/product-backlog/PB-2026-05-batch.md
  Uses "story" in body text; prefer "vertical slice"

─────────────────────────────────────────────

{{If errors:}}
Result: FAIL — fix the {{N}} ERROR(s) above.

{{If only warnings:}}
Result: PASS — no errors. {{M}} WARN(s) flagged for attention but do not block merge.

{{If --strict and warns present:}}
Result: FAIL (--strict mode treats WARNs as errors) — address warnings or remove --strict.
```

Exit code:
- **Non-zero** if any ERROR (or any WARN under `--strict`).
- **Zero** otherwise.
</step>

</process>

<guardrails>
- NEVER modify the target repo. This is a strictly read-only skill.
- NEVER fabricate findings — every reported violation must point to a real file and a real rule.
- ALWAYS include the file path in each finding (actionability).
- ALWAYS report the ruleset version in the report header for traceability.
- NEVER suppress ERRORs — if a rule says ERROR, it reports as ERROR.
- If parsing a UC/DISC/PB file's required fields is ambiguous (e.g., heuristic search fails), report a finding rather than silently passing.
</guardrails>

<success_criteria>
- [ ] Every rule in `validation-rules.md` is checked
- [ ] Each finding names the offending file path and the rule ID
- [ ] Exit code matches the severity: zero if no errors, non-zero if any error
- [ ] Voyager repo passes with zero ERRORs (the load-bearing test in `tests/validate.test.md`)
</success_criteria>
