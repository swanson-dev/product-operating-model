<purpose>
Produce a paste-ready stuck-artifact digest for a POM portfolio. Read-only on artifacts.

Output is shaped for direct paste into Teams or email. The receiver sees the stuck-list and knows what to chase, with no further parsing needed.
</purpose>

<process>

<step name="parse_args">
Parse:
- First positional: `$TARGET_DIR` (optional, defaults to cwd)
- `--uc-sla=<days>` (optional, default 7)
- `--disc-sla=<days>` (optional, default 14)
- `--pb-sla=<days>` (optional, default 14)
- `--adr-sla=<days>` (optional, default 7)
- `--trio-sla=<days>` (optional, default 30)
</step>

<step name="precondition_checks">
Verify `$TARGET_DIR` is a POM repo (`intake/` and `products/` exist).

If `$TARGET_DIR/.pom/snapshots/` has prior snapshots, the newest snapshot's `taken_at` is "yesterday's baseline" for cross-run trend detection (used in Section 7 closing line, e.g., "3 of these were also stuck yesterday").
</step>

<step name="scan_stuck_ucs">
Glob `intake/use-case-backlog/UC-*.md` (exclude template).

For each UC, parse the metadata table. An UC is stuck if:
- `Status` is `pending` or `scored` (no disposition yet)
- AND age (today − `Submitted` field date, falling back to git creation date) > `$UC_SLA`

For each stuck UC, capture: ID, age days, responsible role (PM/PD/TL from the Stakeholders table — usually a sponsor or PM noted there).
</step>

<step name="scan_stuck_discs">
Glob `products/*/discovery-backlog/DISC-*.md` (exclude template).

For each DISC, parse the Promotion Readiness table and the shaping log. A DISC is stuck if:
- Verdict is `NOT READY` (not 4/4 ✅)
- AND age since the most recent shaping log entry > `$DISC_SLA`

Capture: ID, current gate state (e.g., `Q1✅/Q2🟡/Q3✅/Q4❌`), days since last shaping log entry, trio (read from the DISC's trio snapshot), and the next blocker phrase from the critical-path sequence.
</step>

<step name="scan_stuck_pbs">
Glob `products/*/product-backlog/PB-*.md` (exclude template).

For each PB, parse the status field. A PB is stuck if:
- Status is `shaped` (promoted but not yet pulled by any pod)
- AND age (today − promotion date in filename `PB-YYYY-MM-`, or git creation date) > `$PB_SLA`

Capture: ID, age days, product slug, owning pod (if any).
</step>

<step name="scan_proposed_adrs">
Glob `products/*/runway/ADR-*.md`.

For each ADR, parse its Status field. An ADR is stuck if:
- Status is `Proposed` (not yet `Accepted`)
- AND age (git creation date) > `$ADR_SLA`

Capture: ID, age days, product slug, TL.
</step>

<step name="scan_trio_gaps">
For each subdirectory in `products/` (exclude `_template`):
- Read `trio.md`. Identify unassigned roles (`_TBD_`, `Unassigned`, blank).
- Count Discovery items currently in that product.
- A Trio gap is stuck if:
  - Any Trio role is unassigned
  - AND the product has ≥ 1 Discovery item (so the gap is blocking active work, not dormant)
  - AND product age (git creation date of `product.md`) > `$TRIO_SLA`

Capture: product slug, unassigned roles, count of Discovery items affected.
</step>

<step name="compute_trend">
**Only if `.pom/snapshots/*.json` exists with ≥ 1 prior snapshot.**

Load the newest prior snapshot. For each stuck artifact identified above, check whether it was also stuck in the baseline (same ID, same `status` or `verdict`, age then was already > SLA).

Compute: how many of today's stuck items were also stuck yesterday (`carry_over_count`). New-this-period = today's stuck − carry-over.

This is the only field that needs cross-snapshot reasoning. If no snapshots exist, skip silently and omit the trend line from the closing.
</step>

<step name="render_digest">
Compose the paste-ready digest. Use this exact shape (omit any section with zero items):

```
**POM Pulse — {{date}}**
Portfolio: {{target dir name}}
Stuck total: {{N}}{{ ({{carry_over_count}} carried over from {{baseline date}}) if applicable}}

{{If stuck UCs > 0:}}
**🟠 Use Cases past disposition SLA ({{uc-sla}}d)**
- {{UC-NNNN}}: {{title}} — {{age}}d old, owner: {{role}}
- ...

{{If stuck DISCs > 0:}}
**🟡 Discovery items aging at gate ({{disc-sla}}d)**
- {{DISC-ID}} ({{product}}): {{Q1}}/{{Q2}}/{{Q3}}/{{Q4}}, {{age}}d since last shaping, blocked on {{phrase}}, Trio {{names}}
- ...

{{If stuck PBs > 0:}}
**🔵 Product Backlog shaped but not pulled ({{pb-sla}}d)**
- {{PB-ID}} ({{product}}): {{age}}d shaped{{, pod: <pod> if owned else no pod yet}}
- ...

{{If Proposed ADRs > 0:}}
**🟣 Proposed ADRs awaiting decision ({{adr-sla}}d)**
- {{ADR-NNNN}} ({{product}}): {{title}}, {{age}}d Proposed, TL {{name}}
- ...

{{If Trio gaps > 0:}}
**🔴 Trio gaps blocking active Discovery ({{trio-sla}}d)**
- {{product}}: {{unassigned roles}} — {{N}} Discovery item(s) affected
- ...

**Next steps**
- Each owner: `/pom-explain <ID>` for context, then either move the artifact forward or update its status.
- Run `/pom-status --since=last` to see what shipped since yesterday.
```

The block is markdown so Teams renders it natively. Use only emoji indicators (🟠🟡🔵🟣🔴), bold headers, bullet lists — no tables (Teams pastes them poorly).
</step>

<step name="report">
Print the rendered digest directly. No preamble, no postamble, no commentary outside the digest block.

If NOTHING is stuck across all five sections, print:
```
**POM Pulse — {{date}}**
Portfolio: {{target dir name}}

✅ No stuck artifacts past current SLAs. The portfolio is moving.
```
</step>

</process>

<guardrails>
- **READ-ONLY on portfolio artifacts.** Never modifies UC, DISP, DISC, PB, ADR, DEC, pod, product, platform, or enabling files.
- **Never invent stuck items.** Every line must trace to a real file past its SLA.
- **No tables in the digest** — Teams renders them poorly. Bullets and bold only.
- **Omit empty sections.** Readability > consistency.
- **SLAs are advisory, not load-bearing.** If `$ARGUMENTS` provides custom thresholds, use them.
- The trend line only appears when ≥ 1 prior snapshot exists. Otherwise silently omit it.
</guardrails>

<success_criteria>
- [ ] Each stuck artifact in the output traces to a real file past its SLA
- [ ] Empty sections are omitted entirely
- [ ] Output is valid markdown that renders correctly in Teams
- [ ] Zero modifications to portfolio artifacts
- [ ] Trend line ("X carried over from <baseline date>") appears iff `.pom/snapshots/` has prior data
- [ ] If nothing is stuck, the "all clear" message is rendered
</success_criteria>
