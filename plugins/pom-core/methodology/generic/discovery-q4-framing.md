# Q4 (Ethical / Responsible) — Generic framing

The Four-Question Discovery Gate's fourth question asks: **"Is it ethical, safe, and responsible to build?"** This file defines the *generic* baseline framing. Industry overlay plugins (`pom-insurance`, `pom-fintech`, `pom-healthcare`, `pom-retail`) layer industry-specific sub-questions on top.

> **How this file is used.** `pom-discovery-gate` reads this framing when grading Q4. `pom-ethics-reviewer` reads it for the independent audit. Both expect evidence (not assertions) before marking Q4 as ✅.

## What Q4 is — and isn't

Q4 is **not** a generic "is this a good idea" check. It is a hard gate that asks whether building this would cause harm to users, third parties, or the broader public — and whether building it would meet the bar set by enabling standards (security, privacy, accessibility, ethics) that the portfolio has committed to.

A Q4 ❌ does not necessarily kill a Discovery item. It means the item cannot promote until either (a) the harm/standard concern is mitigated or (b) an exception is recorded in the appropriate enabling concern's `decision-log/`.

## Generic sub-questions

A Discovery item answers Q4 ✅ when **every applicable** sub-question below is answered with cited evidence:

### G1 — Vulnerable users
Could the proposed change put any subset of users at materially worse risk than today? Consider: minors, financially distressed users, accessibility needs, language access, low-literacy users, users in coercive contexts.
- **Evidence patterns:** user research segment definitions; audit of who the change affects vs. ignores; explicit deprioritization rationale if any group is excluded.

### G2 — Data minimisation
Does the change collect, store, or transmit only the data strictly required for the stated user outcome? Each new field collected, retained, or shared must be justified against the outcome.
- **Evidence patterns:** data-flow diagram with sources/sinks; field-level retention justification; deletion/rotation policy.

### G3 — Consent and transparency
Are users informed about what is happening with their data and choices in plain language, at the moment of decision (not buried in a policy)? Can they change their mind later?
- **Evidence patterns:** UX flow with consent surface; copy review by a non-technical reader; opt-out path verified end-to-end.

### G4 — Reversibility
If the change turns out to cause harm we did not anticipate, can we roll back without consequence to users? "Consequence" includes data that cannot be unlearned (e.g., shared to a third party) or actions that cannot be unwound (e.g., money moved).
- **Evidence patterns:** rollback plan; identified data egress points; a "kill switch" the operator can use without engineering involvement.

### G5 — Dual use and misuse
Could a competent adversary use this feature for purposes other than the intended one (e.g., to harm other users, evade fraud detection, generate spam, scrape personal data)? Have we modelled the abuse paths explicitly?
- **Evidence patterns:** abuse-cases document; rate-limiting / friction surfaces; monitoring of abuse signals planned at launch.

### G6 — Algorithmic accountability (if AI/ML involved)
Does the change involve a model that influences a decision about a person? If yes: is there a model card; are there documented bias audits across reasonable demographic strata; is there a human-in-the-loop path for high-stakes outcomes?
- **Evidence patterns:** `runway/model-cards/MC-<slug>.md`; bias-audit results referenced from `enabling/ai-ethics/decision-log/`; human-review path documented.

### G7 — Third-party / supply chain
If the change depends on a third-party service (SaaS, API, model provider), have we vetted the third party against the same Q4 standards? What changes if they change their terms?
- **Evidence patterns:** vendor due-diligence note; data-processing agreement reference; contingency if the dependency fails or pivots.

### G8 — Enabling-standard alignment
For each applicable enabling concern (`enabling/<concern>/`), does the change comply with the current standards version? Where it deviates, is there a recorded exception in `enabling/<concern>/decision-log/`?
- **Evidence patterns:** `enabling/<concern>/decision-log/DEC-YYYY-MM-DD-<slug>.md`; explicit "no deviation" statement if everything aligns.

## How to mark Q4

Use the same 4-state marker as the other three questions:
- **✅ Strong evidence** — every applicable sub-question above has cited evidence; no unmitigated concerns
- **🟡 Partial / mixed** — some sub-questions answered, others not; or evidence is weak; or one mitigation depends on work outside this Discovery item's scope
- **❌ No evidence / serious concern** — at least one sub-question has a credible unmitigated concern; cannot promote
- **N/A** — only if no users, no data, and no decisions affecting people are involved (very rare; default to ✅/🟡/❌)

Default is **❌** until evidence is cited. Confirmation bias is the failure mode this gate exists to prevent.

## When to bring in an enabling-standard owner

If G6, G7, or G8 requires deviating from an existing enabling standard (e.g., the AI-ethics standard says "no PHI in model training" and the proposed change wants an exception for de-identified PHI), do **not** record the exception in the Discovery shaping log. Instead invoke `/pom-decision-log` to record it in `enabling/<concern>/decision-log/` — that is the immutable, precedent-setting decision the standard owner is accountable for.

## See also

- `methodology/references/four-question-gate.md` — full gate rubric and rationale
- `methodology/templates/enabling-standards/ai-ethics/` — canonical AI-ethics scaffolding (model card, bias audit, PII handling)
- `pom-ethics-reviewer` agent — independent audit of a Q4 answer
- Industry overlay packs (`pom-<industry>`) for regulatory and domain-specific sub-questions
