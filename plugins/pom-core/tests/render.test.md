# Test: `pom-render`

## Scenario A: render a populated portfolio

**Setup:** A POM repo with:
- 5 UCs (3 scored, 2 pending)
- 3 dispositions (1 route, 1 park, 1 kill)
- 2 products, each with a Trio and 2 Discovery items
- 1 PB item shaped
- 1 ADR with status `Accepted`
- 1 decision-log entry under `enabling/ai-ethics/`

**Invocation:** `/pom-render`

**Expected outcome:**
- Skill writes `portfolio.html` to the repo root.
- File is valid HTML5 (opens in a browser without console errors).
- All sections present: Header, Funnel, Products grid, Aging heatmap, Gate radars, WSJF distribution, Trio matrix, Activity stream, Blockers.
- Industry overlay panel is OMITTED (no overlay applied in this setup).
- Counts match what `/pom-status` would report exactly.
- File size ≤ 250 KB.

**Pressure points:**
- If skill writes ANY portfolio artifact → CATASTROPHIC FAIL (read-only on artifacts)
- If skill emits external `<link>` or `<script src>` references → FAIL (must be self-contained)
- If counts disagree with `pom-status` → FAIL (single source of truth)
- If file exceeds 250 KB without paginating → FAIL

---

## Scenario B: insurance overlay detected

**Setup:** Same as Scenario A but with the insurance overlay applied: `intake/scoring/wsjf-rubric.md` references insurance anchors, `enabling/state-filings/` exists, and one DISC's Q4 cites "NY DFS filing deadline 2026-06-30".

**Invocation:** `/pom-render`

**Expected outcome:**
- All sections from Scenario A.
- Industry overlay panel is rendered between Blockers and end of document.
- Insurance panel shows: open state-filing deadlines, actuarial-review status counts, regulatory-reporting risk flags — each row traces to a real DISC/UC.

**Pressure points:**
- If overlay panel shows when no overlay is detected → FAIL
- If overlay row references a DISC that doesn't exist → FAIL (must trace to real files)

---

## Scenario C: empty portfolio

**Setup:** Fresh `pom-bootstrap` output with no UCs, no products.

**Invocation:** `/pom-render`

**Expected outcome:**
- File is generated but small.
- Each section renders with explicit "(none yet)" placeholders.
- Header explains "Portfolio not yet populated — see `pom-ingest-use-case` to start."

---

## Scenario D: print preview

**Setup:** Any populated portfolio.

**Action:** Open `portfolio.html` and trigger print preview in the browser.

**Expected outcome:** All collapsible sections expanded; interactive controls hidden; page breaks fall between sections, not mid-table; colors degrade to printable grayscale-friendly hatching.

---

## Scenario E: accessibility

**Action:** Open the file and inspect with an accessibility tool (axe, Lighthouse).

**Expected outcome:**
- All color-coded states have textual labels (e.g., vacant Trio cells have `aria-label="vacant"` AND visible text).
- WCAG AA contrast passes in both light and dark schemes.
- Semantic landmarks present (`<header>`, `<main>`, `<section>`).

---

## Pressure points (cross-scenario)

- Single-file contract: no external CSS, no external JS, no CDN, no images.
- Read-only-on-artifacts: any modification to UC / DISP / DISC / PB / ADR / DEC / pod / product / platform / enabling = FAIL.
- Count-consistency: every numeral in the cockpit must match what `pom-status` would report for the same repo.
- Overlay-honesty: industry panel renders only when overlay is genuinely present.
