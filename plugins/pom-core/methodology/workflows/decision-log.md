<purpose>
Append a precedent-setting decision to an enabling concern's decision log. For exceptions, threshold-setting, standards-level deviations, or conflicts with authoritative policy.

Append-only: never edits prior decision files. Filename collisions are disambiguated with a suffix.

For per-UC dispositions, use `pom-disposition` instead. For runway architectural decisions, use `pom-runway-add-adr` instead.
</purpose>

<process>

<step name="parse_args">
Parse:
- `--target=<path>` (required) — relative path to the enabling concern folder (e.g., `enabling/ai-ethics`)
- `--slug=<kebab-case-slug>` (required; prompted if missing) — short descriptor of the decision

If either missing, ask via `AskUserQuestion`.
</step>

<step name="validate_target">
`$TARGET` must match the pattern `enabling/<concern>` (allowed roots).

If `$TARGET` is under `products/*/runway/`, abort:
```
Runway decisions belong in ADRs, not decision logs.

Use `/pom-runway-add-adr <product> <slug>` instead.
```

If `$TARGET` is `intake/dispositions/`, abort:
```
Per-UC dispositions use `pom-disposition`, not `pom-decision-log`.

Use `/pom-disposition <UC-ID> --decision=<kill|park|route|split>` instead.
```

Other paths: abort with a list of allowed targets.
</step>

<step name="validate_slug">
`$SLUG` must be kebab-case. Abort with corrected suggestion if invalid.
</step>

<step name="precondition_checks">
Verify cwd is a POM repo.

Verify `$TARGET/` exists. If not, abort:
```
Target $TARGET does not exist.

If this concern is new, run `/pom-seed-enabling-standard --concern=<concern>` first.
```

Verify `$TARGET/decision-log/` exists. If not, this is a structural issue — abort with the missing path.
</step>

<step name="gather_fields">
Use `AskUserQuestion` (chain multiple questions or batch them) to collect:

1. **Deciders** — names + roles (Trio members, Compliance, Risk, Enablement reps)
2. **Context** — what was being decided and why. What triggered this decision (a UC's Discovery Q4, a regulatory change, an audit finding, etc.)?
3. **Decision** — exactly what was decided. Be specific about scope.
4. **Rationale** — why this decision; what alternatives were considered.
5. **Precedent scope** — UC-only / product-only / portfolio-wide
6. **Expiration / re-review** — date, event, or condition that triggers revisiting this decision

If the user provides any of these via flags (future: --deciders, etc.), skip the corresponding prompt.
</step>

<step name="compute_filename">
$DATE = today (YYYY-MM-DD).
$FILENAME = `DEC-$DATE-$SLUG.md`.

If a file with this exact name already exists in `$TARGET/decision-log/`:
- Add a suffix `-2`, `-3`, etc. to disambiguate. Loop until a non-colliding name is found.
</step>

<step name="write_decision_file">
Generate the file content:

```markdown
# DEC-$DATE-$SLUG — {{decision-summary-one-line}}

| Field | Value |
|---|---|
| **Decision ID** | DEC-$DATE-$SLUG |
| **Date** | $DATE |
| **Concern** | {{concern from $TARGET}} |
| **Deciders** | {{names + roles}} |
| **Precedent scope** | {{UC-only / product-only / portfolio-wide}} |
| **Expiration / re-review** | {{date, event, or condition}} |

## Context

{{What was being decided and why; what triggered this decision}}

## Decision

{{Exactly what was decided. Specific about scope.}}

## Rationale

{{Why; what alternatives were considered}}

## Implications

{{What changes for future UCs / products / pods as a result of this precedent}}

## Related decisions

{{Links to any prior decisions this builds on or supersedes}}
```

Write to `$TARGET/decision-log/$FILENAME`.
</step>

<step name="report">
```
✅ Decision logged: DEC-$DATE-$SLUG

File: $TARGET/decision-log/$FILENAME

Precedent scope: {{scope}}

This decision is now precedent. Future UCs touching this concern should:
1. Read this folder before scoring/shaping.
2. Reference this DEC by ID when relevant.
3. Re-evaluate when the expiration condition fires: {{condition}}.
```
</step>

</process>

<guardrails>
- **APPEND-ONLY.** Never edit a prior decision file. Reversal = new DEC that supersedes the old.
- **NEVER write to `intake/dispositions/`** — that's `pom-disposition`'s job.
- **NEVER write to `products/*/runway/`** for ADRs — that's `pom-runway-add-adr`'s job.
- Disambiguate filename collisions automatically; never overwrite.
- ALL required fields must be filled — no skipping deciders, scope, or expiration.
</guardrails>

<success_criteria>
- [ ] Decision file written in correct `$TARGET/decision-log/` location
- [ ] Filename matches `DEC-YYYY-MM-DD-<slug>.md` (R6.3)
- [ ] All required fields populated
- [ ] No prior file overwritten
- [ ] `pom-validate` passes (R6.3, R6.4)
</success_criteria>
