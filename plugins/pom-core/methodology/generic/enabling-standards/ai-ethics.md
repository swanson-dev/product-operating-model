# AI ethics — generic starter standard

> **This is a starter.** Customise it to your portfolio's actual context, then use `/pom-decision-log` to record material decisions over time.

## Scope

This standard covers any change that:
- Trains a machine-learning model on user, customer, or operational data
- Embeds a model output into a user-facing decision (recommendation, ranking, scoring, classification, generation)
- Uses a third-party model provider for the same

It does **not** cover analytics, dashboards, or rule-based automation that contains no learned behaviour. Those are governed by `data-quality.md`.

## Baseline rules

### A1 — Model card required
Every model used in production has a model card recorded at `products/<product>/runway/model-cards/MC-<slug>.md`. The template at `methodology/templates/enabling-standards/ai-ethics/model-card-template.md` defines the required fields. New deployment without a model card is blocked at the Discovery gate (Q4 ❌, G6).

### A2 — Bias audit before launch
For any model that influences a decision about a person, a bias audit runs against the deployed model before launch. Strata to include at minimum: any protected demographic the portfolio is aware of, plus any segment material to the product (e.g., user tenure, geographic region, account size).
- The audit produces a report committed to `enabling/ai-ethics/audits/AUDIT-YYYY-MM-DD-<slug>.md`.
- A failing audit (statistically significant disparity beyond a documented threshold) blocks launch.

### A3 — Human-in-the-loop for high-stakes outcomes
"High-stakes" = the model output materially affects an individual's economic, legal, social, or safety outcome. For these cases, a human review path exists for adverse outcomes. The Discovery gate Q4 ✅ requires this path to be documented in the product backlog item.

### A4 — No production training on customer data without recorded consent
If customer-identifiable data is used for training (not inference) a production model, the consent and legal basis is recorded in `enabling/ai-ethics/decision-log/` before training begins.

### A5 — Drift monitoring
For any production model, drift monitoring is in place at launch — both for input distribution and output distribution. The thresholds and response plan are recorded in the model card.

### A6 — Third-party model providers
Any third-party model used (commercial API, open-weights model hosted by a vendor) is recorded with: name, version, terms-of-service review date, data residency claim. If the provider trains on submitted prompts, that fact and its mitigation (e.g., enterprise plan opt-out) is recorded in the model card.

## Open questions for the standard owner

- What is "statistically significant disparity" for A2 — a specific p-value threshold, a confidence interval, an effect-size minimum? Calibrate to portfolio context and record the decision.
- What is the SLA for human review (A3)? 24 hours? 7 days? Record the decision per affected product.
- What is the bar for "production model" (A1, A5)? Any model serving any traffic? Or only models above a request-volume threshold? Record per portfolio.
- Where does third-party model approval live in the procurement process? Pre-procurement (security review), post-procurement (one-pager filed)?

## How this standard interacts with Q4

The `pom-ethics-reviewer` agent and the `pom-discovery-gate` skill both read this standard when grading Q4 sub-question G6 (algorithmic accountability) and G7 (third-party / supply chain). Any Discovery item touching an ML model needs to show alignment with A1–A6 above, OR show a recorded exception in `enabling/ai-ethics/decision-log/`.

## See also

- `methodology/templates/enabling-standards/ai-ethics/model-card-template.md`
- `methodology/templates/enabling-standards/ai-ethics/bias-audit-framework.md`
- `methodology/templates/enabling-standards/ai-ethics/pii-handling.md`
- `methodology/generic/discovery-q4-framing.md` — how Q4 reads this standard
