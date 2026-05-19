<purpose>
Render a single self-contained `portfolio.html` cockpit from a POM portfolio's markdown corpus. Glanceable, leadership-ready, single-file.

The cockpit reuses the same parsing semantics as `pom-status` and `pom-explain` so all three skills agree on field names and counts.
</purpose>

<process>

<step name="parse_args">
Parse:
- First positional: `$TARGET_DIR` (optional, defaults to cwd)
- `--output=<path>` (optional, default `$TARGET_DIR/portfolio.html`)
</step>

<step name="precondition_checks">
Verify `$TARGET_DIR` is a POM repo. If not, abort and suggest `pom-bootstrap`.
</step>

<step name="scan_portfolio">
Reuse the scanning logic from `status.md` (`scan_intake`, `scan_products`, `scan_enabling`, `scan_platform`, `detect_blockers`, and `snapshot_current_state` — but do NOT write a snapshot here; the cockpit only reads).

Augment with per-DISC question state extraction:
- For each DISC file, parse the four `## Q1` / `## Q2` / `## Q3` / `## Q4` sections to get their current `Status: ❌|🟡|✅` line.
- Compute the radar coordinates (Q1=0/0.5/1, Q2=0/0.5/1, Q3=0/0.5/1, Q4=0/0.5/1).

For each UC with a WSJF score, capture the numeric value for the histogram.

For each artifact, compute age (days since creation) for the aging heatmap.
</step>

<step name="detect_industry_overlay">
Check for industry overlay markers:
- Read `intake/scoring/wsjf-rubric.md` for header text indicating a calibrated industry rubric.
- Glob `enabling/*/README.md` for industry-specific concerns (e.g., `state-filings/`, `actuarial-review/`, `clinical-safety/`, `phi-handling/`).
- Check for an `industry:` line in the top-level README or `methodology/README.md`.

If insurance overlay is detected, capture:
- Open state-filing deadlines (parse DISC files for date-mentioning blocker phrases like "state filing", "NAIC", "NY DFS")
- Actuarial-review status counts across DISCs (Q4 sub-question states)
- Regulatory reporting risk flags

If no overlay detected, skip the industry panel entirely.
</step>

<step name="compose_html">
Generate a single HTML document with:

- `<!DOCTYPE html>` and minimal HTML5 boilerplate
- `<style>` block in `<head>` containing all CSS (no external stylesheets)
- `<script>` block at end of `<body>` for any minimal interactivity (collapse/expand sections only — no charts library)
- All data rendered as semantic HTML (tables, lists, divs with class names) — not as images, not as canvas

**CSS principles:**
- System font stack (`-apple-system, "Segoe UI", Roboto, sans-serif`) so the file opens identically on Mac, Windows, mobile.
- Light theme by default with a `prefers-color-scheme: dark` block for dark mode.
- Max content width 1200px, centered.
- Print-friendly (`@media print` rules: hide navigation, expand all collapsibles).
- Color palette aligned with POM gate semantics: green (`#22a06b`), yellow (`#d97706`), red (`#dc2626`), blue (`#2563eb`) for in-flight, gray (`#6b7280`) for inactive.

**Section composition (in order):**

1. **Header** — `<h1>` portfolio name; subline with date, total artifact counts, last-activity timestamp.
2. **Funnel** — A horizontal strip of labeled bands using flexbox; width of each band is `flex: <count>`. Bands: `UCs pending → UCs scored → Dispositions → Discovery (NOT READY) → Discovery (READY) → PB shaped → PB pulled → PB in-flight → PB shipped`. Each band shows count as large numeral.
3. **Products grid** — CSS grid of cards. Each card: product name, Trio status badges, mini funnel (DISC by state, PB by state, Pods), open blockers count badge.
4. **Aging heatmap** — `<table>` with products as rows, stages as columns; cell background-color encodes oldest age in that cell (CSS-only, no JS).
5. **Discovery gate radar** — Each Discovery item gets a small SVG radar (4 axes Q1/Q2/Q3/Q4). Polygon fill green if 4/4, yellow if 3/4, red otherwise. SVG inline.
6. **WSJF distribution** — A CSS-only horizontal histogram of WSJF buckets (≤3, 5, 8, 13, 21, 34+). Each bucket bar segmented by status color.
7. **Trio assignment matrix** — `<table>` of products × PM/PD/TL. `_TBD_` cells get a red dot and `aria-label="vacant"`.
8. **Recent activity stream** — `<ul>` of last 10 events sorted desc by date. Each `<li>`: date, event type icon, artifact ID linked to its file path (relative).
9. **Open blockers** — `<ul>` mirroring the `pom-status` blocker section, items ordered by criticality.
10. **Industry overlay panel** (conditional) — Section only rendered if an overlay was detected. For insurance: 3 sub-panels (Filing deadlines, Actuarial review, Regulatory reporting risk).

**Accessibility:**
- Semantic HTML (`<header>`, `<main>`, `<section>`, `<nav>`).
- All color-coded states ALSO have text labels (e.g., a red Trio dot has text "vacant" beside it).
- Color choices pass WCAG AA on both light and dark schemes.

**File-size discipline:**
- Target ≤ 250 KB total. No images. No fonts. No external links to CDNs.
- If portfolio is large (> 200 artifacts), paginate the Discovery radar section with collapsible groups.
</step>

<step name="write_file">
Write the composed HTML to `$OUTPUT` (default `$TARGET_DIR/portfolio.html`).

If a prior `portfolio.html` exists at the path, overwrite it (it is a derived artifact, not part of the immutable record).

Compute the file size after write. Report it.
</step>

<step name="report">
```
✅ Portfolio cockpit rendered.

File: portfolio.html ({{size_kb}} KB)
Rendered: {{total artifacts}} artifacts across {{N products}} product(s).
{{If industry overlay detected:}}
Industry overlay surfaced: {{overlay name}}

Open it:
  - Windows:  start portfolio.html
  - macOS:    open portfolio.html
  - Linux:    xdg-open portfolio.html

Commit it alongside artifacts to version the cockpit with the portfolio.

Next:
- Re-run /pom-render after material changes.
- /pom-status --since=last to see what changed in the underlying corpus.
```
</step>

</process>

<guardrails>
- **READ-ONLY on portfolio artifacts.** Never modifies UC, DISP, DISC, PB, ADR, DEC, pod, product, platform, or enabling files.
- The cockpit is the ONLY allowed write target (default `portfolio.html` at repo root).
- **Single self-contained file.** No external CSS, no external JS, no CDN links, no image files. The HTML must open and render identically with the machine offline.
- **No invented values.** Every count, age, owner, blocker, and event must trace to a real file. If parsing fails for a field, render `unknown` or omit the row — do not guess.
- **File-size budget ~250 KB.** Paginate or collapse if the portfolio is too large.
- Industry overlay panel renders only when an overlay is actually detected — never speculative.
- The cockpit must be print-friendly. `@media print` rules expand collapsibles and hide interactive controls.
</guardrails>

<success_criteria>
- [ ] `portfolio.html` exists at the configured output path
- [ ] Opens in Chrome, Edge, Firefox, Safari without errors
- [ ] All sections render with counts matching `pom-status` exactly
- [ ] Trio vacancies, stuck artifacts, and blockers are visible without scrolling on a 1080p screen
- [ ] Insurance overlay panel renders when overlay is detected, omitted otherwise
- [ ] File size ≤ 250 KB
- [ ] No portfolio artifact modified
- [ ] Print preview shows all sections expanded
</success_criteria>
