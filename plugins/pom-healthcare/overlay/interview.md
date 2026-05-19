# Healthcare overlay — bootstrap interview

Capture answers via `AskUserQuestion`; append to `<pom-repo-path>/intake/scoring/healthcare-calibration.md`.

## H-Q1 — Portfolio type
"Primary type: (a) Payer (health insurance, including Medicaid/Medicare Advantage). (b) Provider (hospital, health system, ambulatory). (c) Digital health / consumer health. (d) Life sciences (pharma, devices, diagnostics). (e) Health IT / data platform. (f) Hybrid (explain)."

## H-Q2 — Patient population scale
"Approximately how many distinct patients (or members) does the portfolio serve / cover? Order of magnitude is fine."

## H-Q3 — PHI exposure
"PHI handling: (a) Direct (the portfolio creates/holds PHI in normal operation). (b) Indirect (PHI flows through systems we operate as a business associate). (c) De-identified only (no PHI by design). (d) Mixed."
**→** Drives H1 default state.

## H-Q4 — Clinical workflow integration
"Does the portfolio integrate with clinical workflow systems (EHR, ePrescribing, lab, imaging)? (Yes — bidirectional / Yes — read-only / Yes — write-only via standards / No)."
**→** Drives H6, H7 defaults.

## H-Q5 — FDA-regulated software
"Does the portfolio include or interact with FDA-regulated software (SaMD, software in a medical device)? (Yes — we own SaMD / Yes — we integrate with regulated devices / No, but we monitor)."
**→** Drives H8 + clinical-safety standard scope.

## H-Q6 — ML in clinical context
"ML model usage: (a) Clinical decision support (recommendation, risk score). (b) Operational (admin, claims, scheduling). (c) Patient-facing personalization. (d) None currently. Multi-select."
**→** Drives ai-ethics-healthcare overlay defaults.

## H-Q7 — Interoperability participation
"Are you participating in CMS Interoperability rules (Patient Access API, Provider Directory API, Payer-to-Payer API)? Or TEFCA / QHIN?"
**→** Drives interoperability standard customisation.

## H-Q8 — Equity stance
"Does the portfolio have an active health-equity initiative or stated commitment? (Yes — board-level / Yes — operational team / Aspirational / Not yet)."
**→** Drives H4 framing depth.

## After the interview

Answers are written to `intake/scoring/healthcare-calibration.md` and inform downstream rubric and enabling-standards customisation.
