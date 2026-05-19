---
name: pom-validate
description: "Use when checking a POM repo for structural conformance — file naming, required fields, cross-references, four-question-gate completeness, append-only rules, etc. Read-only. Reports ERROR/WARN/INFO findings grouped by rule ID with file paths."
argument-hint: "[target-dir] [--verbose] [--strict] [--rule=<id>]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
---

<objective>
Validate a POM repository against the plugin's bundled validation ruleset (`methodology/references/validation-rules.md`). Produces a structured report grouped by rule ID with file paths. Exits non-zero if any ERROR.

**Use cases:**
- After `pom-bootstrap` to confirm clean output
- Periodically as a quality gate
- Before merging changes to a POM repo
- As a TDD harness during `pom-*` skill development
- For Voyager retro-validation in Phase 6 of the rollout

**Output sections:**
1. Header (target path, ruleset version, date)
2. Summary line (`N ERROR, M WARN, K INFO`; overall PASS/FAIL)
3. Grouped findings by severity — each names the offending file and rule ID
4. Result line

**Flags:**
- `--verbose` — include INFO-level findings (suppressed by default)
- `--strict` — treat WARNs as ERRORs (fails on either)
- `--rule=<id>` — check only one rule (e.g., `--rule=R2.5`)

**Read-only.** This skill never modifies the target repo.

**After this command:** If ERRORs reported, fix the files named in each finding and re-run.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/validate.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
</execution_context>

<process>
Execute end-to-end.
Read-only — never modify the target repo.
Always include file paths in findings.
</process>
