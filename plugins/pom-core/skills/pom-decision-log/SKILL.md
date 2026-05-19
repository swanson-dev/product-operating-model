---
name: pom-decision-log
description: "Use when a precedent-setting decision needs to be appended to an enabling concern's decision log (AI ethics, security, etc.) — exceptions, threshold-setting, standards-level deviations, or conflicts with authoritative policy. Append-only. Not for UC dispositions or runway ADRs."
argument-hint: "--target=enabling/<concern> --slug=<kebab-case-slug>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Append a precedent-setting decision to an enabling concern's `decision-log/`. Use for:
- **Standards-level exceptions** (e.g., "accept 6pp accuracy gap on cohort X for UC-0002")
- **Threshold-setting** (e.g., "set bias audit accuracy-disparity threshold to 5pp for insurance products")
- **Conflicts with authoritative policy** (logged for traceability)
- **Calibration changes** to a v0.1 scaffold

**Writes:**
- `enabling/<concern>/decision-log/DEC-YYYY-MM-DD-<slug>.md` (append-only; filename collisions disambiguated automatically)

**Required fields collected via `AskUserQuestion`:**
- Deciders (names + roles)
- Context (what was decided and why; what triggered this)
- Decision (exactly what was decided; specific scope)
- Rationale (why; alternatives considered)
- Precedent scope (UC-only / product-only / portfolio-wide)
- Expiration / re-review condition

**Emits** a paste-ready precedent summary in the CLI report — one paragraph for Teams/email/PR comments naming the decision, concern, precedent scope, and re-review trigger.

**Refuses to write to non-decision-log paths:**
- `intake/dispositions/` → use `/pom-disposition` instead
- `products/*/runway/` → use `/pom-runway-add-adr` instead

**Refuses** to overwrite prior decisions. Reversal = new DEC file that supersedes the old.

**After this command:** This decision is now precedent. Future UCs touching the same concern should read this folder before scoring/shaping.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/decision-log.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/enabling/ai-ethics/decision-log/README.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (target-path validation, append-only rule, required-field collection, collision disambiguation).
</process>
