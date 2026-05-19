---
name: pom-render
description: "Use when a POM portfolio needs a glanceable visual cockpit — generates a single self-contained portfolio.html at the repo root showing the funnel (UCs → DISPs → DISCs → PBs → Pods), aging heatmap, four-question gate radar per Discovery item, WSJF distribution, Trio assignment matrix, and enabling-concern decision activity. Insurance-overlay-aware (surfaces regulatory risk indicators when present). Single file, no server, no build step, opens in any browser, pasteable anywhere."
argument-hint: "[target-dir] [--output=<path>]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
---

<objective>
Generate a single self-contained `portfolio.html` file at the POM repo root that visualises portfolio state. The file is glanceable — leaders open it and see funnel health, stuck items, and product status in one view, no further navigation needed.

**Why single-file HTML:** POM's "no infra" philosophy. The file drops into Confluence, SharePoint, OneDrive, email attachments, or any team wiki. No server, no build step, no dependencies on a renderer. Inline CSS, inline minimal JavaScript only for interactivity (none required for first read).

**What it shows:**
1. **Header**: portfolio name (from top-level README), date, total artifact counts, "last activity" timestamp
2. **Funnel**: horizontal Sankey-style flow from `UC submitted → UC scored → Disposition (route/park/kill/split) → Discovery → Promotion → Pod → In-flight → Shipped`. Width of each band reflects count.
3. **Products grid**: one card per product with Trio status (red/yellow/green), Discovery count by gate state (0/4 through 4/4), PB count, Pod count
4. **Aging heatmap**: a per-product × per-stage matrix where cell color encodes the oldest artifact's age at that stage (green < 7d, yellow 7-14d, orange 14-30d, red > 30d)
5. **Four-question gate radar**: small radar per Discovery item showing Q1/Q2/Q3/Q4 completion (❌=0, 🟡=0.5, ✅=1)
6. **WSJF distribution**: histogram of UC WSJF scores, segmented by Status (pending/scored/dispositioned)
7. **Trio assignment matrix**: per-product rows × PM/PD/TL columns; cells are names or `_TBD_` flagged in red
8. **Recent activity stream**: most recent 10 events from dispositions / DISCs advanced / ADRs / decisions (last activity dates pulled from filenames + git)
9. **Open blockers** (mirrors the `pom-status` blocker section)
10. **Industry-overlay panel** (if the portfolio has an industry overlay applied): surfaces overlay-specific risk indicators. For insurance: state-filing deadlines, actuarial-review status across DISCs, regulatory-reporting risk flags.

**Read-only on artifacts.** The only file written is `$TARGET_DIR/portfolio.html` (and `$TARGET_DIR/.pom/render-cache/` if intermediate scratch files are useful for future iterations — start without it).

**After this command:** Open `portfolio.html` in any browser. Re-run after material changes; commit the file to git so the cockpit is versioned alongside the artifacts.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/render.md
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/status.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
</execution_context>

<process>
Execute end-to-end.
Read-only on artifacts. Write `portfolio.html` (and optional render cache under `.pom/`).
Output is the file path of the generated HTML plus a one-line summary of what was rendered.
</process>
