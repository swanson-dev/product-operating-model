# Generic baseline — bootstrap interview

A short questionnaire `pom-bootstrap --rubric=generic` walks the user through after the canonical directory tree is scaffolded. The answers customize the WSJF rubric anchors, calibrate the four-question gate framing, and pre-populate optional fields in `intake/scoring/wsjf-rubric.md` and `enabling/<concern>/README.md`.

> **Use this file with `AskUserQuestion`.** Each block below is one question. Render the question, capture the answer, and write the result to the target file noted in `→ writes`.

---

## Q1 — Portfolio scope
**Question:** "In one sentence, what is the portfolio's reason for existing? (e.g., 'a B2B SaaS for small-business invoicing,' 'an internal data platform for analysts,' 'a consumer marketplace for second-hand goods.')"
**→ writes** `README.md` Portfolio Description line at repo root.
**Used by:** rubric calibration prompts (Q3, Q6), Q4 generic framing context.

## Q2 — Primary customers
**Question:** "Who are the *primary* customer segments? Pick up to three. (e.g., 'small-business owners,' 'finance analysts,' 'individual sellers.')"
**→ writes** `intake/scoring/wsjf-rubric.md` "Anchor — Customer base" line in the User-Business Value dimension.

## Q3 — Revenue posture
**Question:** "Which best describes the portfolio's relationship to revenue right now? (a) Direct customer revenue. (b) Indirect — supports internal teams who generate revenue. (c) Pre-revenue / strategic investment. (d) Cost centre / efficiency play."
**→ writes** rubric Time-Criticality anchor + the README "Revenue posture" field.
**Why it matters:** Time-Criticality scoring is calibrated very differently for revenue-direct vs. efficiency portfolios.

## Q4 — Risk tolerance
**Question:** "On a scale of 1–5, how risk-tolerant is this portfolio? (1 = highly regulated, customer harm is unrecoverable; 5 = startup mode, can ship and fix forward.)"
**→ writes** Q4 (ethical) gate default-state and rubric Risk-Reduction anchor.
**Why it matters:** A risk-tolerant portfolio's WSJF naturally underweights Risk-Reduction; a risk-averse one overweights it. Calibrating up front prevents argument later.

## Q5 — Average size of a vertical slice
**Question:** "Looking at typical features your teams ship today, what's a realistic median *vertical slice* size? (a) ≤2 person-weeks. (b) 2–4 person-weeks. (c) 4–8 person-weeks. (d) >8 person-weeks — slices are too big, we'll work on splitting them."
**→ writes** rubric Job Size anchor (the Job Size dimension's "1 point ≈ X person-weeks" calibration).
**Why it matters:** WSJF Job Size is meaningless without a calibrated scale. If users answer (d), `pom-ingest-use-case` warns when scoring a UC with Job Size > 20 (rule of thumb: it should be split first).

## Q6 — User population scale
**Question:** "Approximately how many distinct users (people, not events) does the portfolio serve today? (Order of magnitude is fine: tens, hundreds, thousands, tens-of-thousands, hundreds-of-thousands, millions.)"
**→ writes** rubric Customer-base anchor (informs User-Business Value scoring).
**Why it matters:** A change affecting 50 of 50 users scores differently than one affecting 50 of 500,000.

## Q7 — Enabling concerns to seed
**Question:** "Which of these enabling concerns should be seeded with starter standards now? Multi-select: (a) AI ethics. (b) Security baseline. (c) Accessibility. (d) Data quality. (e) None — we'll add as needed."
**→ writes** Drives which `enabling/<concern>/` directories get populated; for each selected concern, copies the corresponding `enabling-standards/<concern>.md` from this generic pack into the user's repo and writes a `decision-log/README.md`.

## Q8 — Trio assignments (optional)
**Question:** "Are there already-known names for the first product's Trio (PM, PD, TL)? Type 'skip' if not."
**→ writes** Pre-fills the seed `products/_template/trio.md` placeholders — but ONLY for the first product spun up; user can always edit later. Skipping leaves the placeholders unchanged.

## Q9 — Pod cadence
**Question:** "What sprint cadence do you expect pods to run on? (1-week, 2-week, 3-week, or 'continuous' — no fixed cadence.)"
**→ writes** `products/_template/pods/_template/pod.md` "Cadence" line default.
**Why it matters:** Just a default that pods can override; surfaces the expectation early.

## Q10 — Methodology source
**Question:** "Is there an existing methodology doc, playbook, or wiki that this POM repo should reference? Type 'none' or paste a URL/path."
**→ writes** README "References" section.
**Why it matters:** Lets new pod members trace back to the source of truth for organisation-specific practices the plugin doesn't capture.

---

## After the interview

`pom-bootstrap` then:
1. Writes a summary of all answers to a new file `intake/scoring/calibration.md` (so the calibration decisions are auditable).
2. Recommends running `/pom-validate <target-dir>` to confirm a clean baseline.
3. Suggests next steps: `/pom-ingest-use-case <first source>` or `/pom-spinup-product <slug>` depending on what the user has ready.

## Industry overlays

If the user later installs `pom-<industry>` and runs `/pom-<industry>-init`, that plugin's own `interview.md` runs as an **additive** questionnaire — only asking questions the generic interview didn't already cover. Answers extend, never overwrite (per Open Question #3 resolution in the design plan: merge with explicit prompts, never silently overwrite user-edited content).
