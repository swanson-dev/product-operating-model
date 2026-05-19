---
description: "Run a full portfolio review — structural validation, portfolio audit (stuck UCs, stale runways, orphan ADRs), and ethics review of recent Discovery items. Read-only. Produces a stakeholder-ready summary."
argument-hint: "[target-dir] [--since=<YYYY-MM-DD>] [--strict] [--product=<slug>]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Skill
---

# /pom-review

Run a coordinated review of the POM portfolio. This command is **read-only** and never mutates the repo.

## Pre-flight

1. Resolve `$TARGET_DIR` — first positional, default cwd.
2. Verify it looks like a POM repo (`intake/`, `products/`, `methodology/`). Abort if not.
3. Resolve `--since` if provided (defaults to "no time bound").
4. Resolve `--product` if provided (scope audits and ethics review to a single product).

## Pipeline (parallel where possible)

### Stage A — Structural validation
Invoke `pom-validate` against `$TARGET_DIR`. Capture ERROR/WARN/INFO counts and the full report.

### Stage B — Portfolio audit
Dispatch the `pom-portfolio-auditor` agent with `$TARGET_DIR`, `--since`, `--product`. The agent independently scans for:
- Stuck UCs (scored, no disposition for >14 days)
- Routed UCs without DISC follow-up (>7 days)
- DISC items at 3/4 with no shaping-log update for >14 days (Discovery rot)
- Promoted DISCs without an active PB pull within 30 days
- Products with no pods >30 days after spinup
- ADRs not referenced in `runway-plan.md`
- Open questions in runway plans untouched >60 days

Returns a structured findings JSON.

### Stage C — Ethics review (sampled)
Dispatch the `pom-ethics-reviewer` agent on each DISC item that has Q4 ✅ marked within the `--since` window (default: all). The agent independently re-grades Q4 against the current `discovery-q4-framing.md` (industry-aware if an overlay is installed). Returns its own ✅/🟡/❌ verdict per DISC, with rationale.

Findings where the auditor disagrees with the original grading (e.g., original ✅ but auditor 🟡 because evidence is weak) are surfaced.

## Output

Produce a single review report with these sections:

```
POM Portfolio Review — <target>
────────────────────────────────
Date: YYYY-MM-DD
Scope: <full | --product=<slug>> | <since YYYY-MM-DD>

1. Structural validation
   Result: PASS / FAIL (<E> ERROR, <W> WARN, <I> INFO)
   [Top 5 errors, file-path:rule-ID]

2. Portfolio audit
   <findings grouped by category>

3. Ethics review
   <N> DISC items re-audited.
   <K> disagreements with original Q4 grading.
   [List of disagreements with rationale]

4. Stakeholder-ready summary
   <one paragraph, paste-able into a leadership update>

5. Recommended next actions (top 3)
```

## Guardrails

- **Strictly read-only.** This command never modifies any file in the target repo.
- If Stage A fails with structural ERRORs that prevent Stages B/C from running (e.g., `intake/` missing entirely), report stage A and skip B/C; don't crash.
- Agents in Stages B/C run independently — do NOT pass the original grading or stage-A findings to the ethics reviewer (that contaminates its independence). Each agent gets only the raw repo state.
- `--strict` propagates to `pom-validate` (treats WARNs as ERRORs).

## References

- `${CLAUDE_PLUGIN_ROOT}/methodology/workflows/validate.md`
- `${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md`
- `${CLAUDE_PLUGIN_ROOT}/methodology/generic/discovery-q4-framing.md`
- `${CLAUDE_PLUGIN_ROOT}/agents/pom-portfolio-auditor.md`
- `${CLAUDE_PLUGIN_ROOT}/agents/pom-ethics-reviewer.md`
