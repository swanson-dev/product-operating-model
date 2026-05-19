# Accessibility — generic baseline standard

> **This is a starter.** It assumes a digital portfolio (web/mobile/internal tools). For non-UI portfolios (purely backend, APIs, ETL), this standard is N/A but you still own accessibility for any operator-facing tools.

## Scope

Every user-facing surface the portfolio ships: customer products, internal admin tooling, support consoles, marketing pages, status pages, error pages, emails.

This standard is the **functional accessibility bar**, not a legal-compliance checklist. Legal compliance (ADA, EAA, Section 508) lives in your legal team's standards; this baseline ensures the bar isn't actively below WCAG 2.2 AA.

## Baseline rules

### X1 — WCAG 2.2 AA as the floor
New surfaces ship at WCAG 2.2 Level AA conformance. Existing surfaces have a documented gap list with prioritised remediation, not just "AAA is aspirational."
- **Evidence at Q4:** WCAG audit (automated + manual); remediation backlog references real PB items.

### X2 — Keyboard accessibility
Every interactive element is reachable and operable from a keyboard alone — no mouse, no touch. Focus order is logical; focus is visible.
- **Evidence at Q4:** keyboard walkthrough recorded for new surfaces (screen recording is fine).

### X3 — Screen-reader semantics
Every interactive element has an accessible name. Headings reflect document structure. Images have alt text or are marked decorative. Forms have associated labels.
- **Evidence at Q4:** screen-reader walkthrough recorded; semantic markup audit.

### X4 — Colour and contrast
Text meets WCAG AA contrast minimums (4.5:1 for body, 3:1 for large text). Information is never conveyed by colour alone.
- **Evidence at Q4:** contrast audit; redundancy check for colour-coded states.

### X5 — Motion and timing
Auto-playing motion can be paused/stopped. Timed interactions either have no time limit, or can be extended, or warn the user before timeout. No content flashes more than 3 times per second.
- **Evidence at Q4:** timing inventory; reduced-motion preference respected.

### X6 — Language and copy
UI copy is written at the lowest reading level that still preserves meaning. Error messages explain what happened and what to do next. The page's language is declared correctly.
- **Evidence at Q4:** readability check (e.g., Flesch-Kincaid grade ≤8 for consumer copy, ≤12 for professional tools); plain-language error messages reviewed.

### X7 — Assistive-tech testing in CI
Where feasible, automated accessibility checks (axe-core, Lighthouse a11y, equivalent) run in CI for the portfolio's UI builds. Failures break the build.
- **Evidence at Q4:** CI configuration referenced; baseline scan exists.

## Open questions for the standard owner

- Where do we host the prioritised remediation backlog — in the product's PB or in `enabling/accessibility/`?
- Do we maintain a per-product VPAT? At what cadence?
- For internal tools, is the bar the same as customer-facing, or lower (e.g., AA-best-effort)?
- Who runs the manual screen-reader walkthroughs and at what trigger (every PR? every release? every major surface change)?

## How this standard interacts with Q4

The `pom-discovery-gate` and `pom-ethics-reviewer` read this standard when grading Q4 sub-question G1 (vulnerable users — accessibility is the most common form of "user we underserve").

## See also

- WCAG 2.2 reference: https://www.w3.org/TR/WCAG22/
- `methodology/generic/discovery-q4-framing.md` G1 (vulnerable users)
- `pom-retail` overlay for accessibility-as-regulatory-exposure framing (e.g., EAA, ADA Title III)
