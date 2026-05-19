# Insurance overlay — bootstrap interview

A short questionnaire `pom-insurance-init` walks the user through after the core POM repo is in place. Answers customize the WSJF rubric anchors and pre-seed the insurance-specific enabling standards.

> Render each question with `AskUserQuestion`. Capture the answer and append to `<pom-repo-path>/intake/scoring/insurance-calibration.md`.

## I-Q1 — Lines of business
**Question:** "Which lines of business does this portfolio cover? Multi-select: (a) Life. (b) Property & Casualty (Personal). (c) Property & Casualty (Commercial). (d) Health. (e) Specialty (e.g., cyber, marine, surety). (f) Annuities."

## I-Q2 — Regulated markets
**Question:** "Which US states are you licensed in, OR — at a high level — 'all 50 + DC', 'top 10 by GWP', or 'specific list'? International markets?"
**→** Drives the state-filing matrix in `enabling/state-filings/` and the I1 sub-question scope.

## I-Q3 — Distribution model
**Question:** "Primary distribution channel: (a) Direct-to-consumer. (b) Captive agents. (c) Independent agents/brokers. (d) Wholesale/MGAs. (e) Embedded (partner-led). (f) Hybrid (explain)."
**→** Drives I5 sub-question framing.

## I-Q4 — Ceded business
**Question:** "Do you cede business to reinsurers? (Yes — material / Yes — incidental / No.)"
**→** If incidental or no, I6 sub-question is largely N/A. If material, additional review steps activate.

## I-Q5 — Regulatory deadlines on the horizon
**Question:** "Any known regulator-driven deadlines in the next 12 months that your portfolio should plan around? (e.g., NAIC model law adoption, state-specific effective date, CFPB ruling.) Type 'none' or list."
**→** Pre-seeds Time-Criticality scoring anchors and recorded as a watchlist in `enabling/regulatory-reporting/README.md`.

## I-Q6 — Actuarial team relationship
**Question:** "Is the actuarial team: (a) Embedded in the product organisation. (b) A separate function with formal review handoffs. (c) Outsourced to a consulting actuary."
**→** Drives I2 review-cadence default in `enabling/actuarial-review/README.md`.

## I-Q7 — Existing model governance
**Question:** "Do you have an existing model risk governance framework (often SR 11-7 inspired, even for insurance)? (Yes — comprehensive / Yes — emerging / No.)"
**→** Pre-seeds AI-ethics enabling standard customisation per I4 + generic G6.

## I-Q8 — Catastrophe exposure (P&C only)
**Question (skip if no P&C in I-Q1):** "Material exposure to catastrophe (hurricane, wildfire, earthquake, severe convective storm)?"
**→** Drives grace-period and surge-claim handling considerations in I3.

## After the interview

`pom-insurance-init` writes the answers to `intake/scoring/insurance-calibration.md` and uses them to:
- Tune the rubric anchors in `intake/scoring/wsjf-rubric.md`
- Seed the watch-list of regulatory deadlines in `enabling/regulatory-reporting/`
- Pre-populate the producer-impact section in `enabling/policyholder-communications/`

The interview is additive to the generic interview already in `methodology/generic/interview.md`. Answers don't overlap; each plays its part.
