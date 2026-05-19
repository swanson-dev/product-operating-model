---
name: pom-pulse
description: "Use when the portfolio needs a daily/weekly digest of stuck artifacts — UCs past disposition SLA, DISCs aging at a gate state, PBs shaped but never pulled, Proposed ADRs awaiting decision. Output is a paste-ready Teams/email message. Read-only on artifacts; may read `.pom/snapshots/` for age detection."
argument-hint: "[target-dir] [--uc-sla=<days>] [--disc-sla=<days>] [--pb-sla=<days>] [--adr-sla=<days>] [--trio-sla=<days>]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
---

<objective>
Produce a paste-ready stuck-artifact digest for a POM portfolio. Read-only on artifacts.

**SLAs (defaults; all overridable via flags):**
- UC pending disposition: **7 days** since `Submitted` field date
- DISC at 0/4, 1/4, 2/4, 3/4 gate state: **14 days** since last shaping log entry
- PB shaped but not pulled: **14 days** since promotion date
- Proposed ADR: **7 days** since file creation
- Trio incomplete on a product with active Discovery items: **30 days** since product creation

**Output shape:** A single markdown block, paste-ready into Teams or email. Sections:
1. Header (target, date, total stuck count)
2. Stuck UCs (one bullet per item with ID, age, responsible role)
3. Stuck DISCs (with gate state and trio)
4. Stuck PBs (with owning pod or "no pod yet")
5. Proposed ADRs (with product and TL)
6. Trio gaps blocking active work
7. Closing line with recommended next steps

If a section has zero stuck items, it's omitted entirely (Teams readability > consistency here).

**Read-only on artifacts.** Never modifies UC, DISP, DISC, PB, ADR, DEC, pod, product, platform, or enabling files. May read `.pom/snapshots/*.json` for cross-snapshot age detection.

**After this command:** Paste the output into the team channel. For each named artifact, the responsible role uses `/pom-explain <ID>` for one-line context or opens the file for full detail.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/pulse.md
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/explain.md
</execution_context>

<process>
Execute end-to-end.
Read-only on artifacts. Output is the digest.
If nothing is stuck, the report says so plainly — do not invent stuck items.
</process>
