# Service Intake — {{service-name}}

How a product requests use of this service.

## Process

1. **Trio reviews fit.** Does this service genuinely solve the product's need, or would rolling our own be better? Document the decision.
2. **Open a request.** {{e.g., file an issue in this repo, send a Slack message, open a ticket}}
3. **Onboarding conversation.** Platform owner walks the product team through:
   - Capabilities and current limits
   - SLA / SLO commitments
   - Integration patterns (see [`consumption.md`](consumption.md))
   - Cost model (if any)
4. **Onboarding ticket created.** Tracks the onboarding work to completion.
5. **First successful integration.** Product team confirms the service is integrated and consuming as expected. README's "Products currently consuming" updated.

## Lead time

{{Expected time from initial request to first successful consumption}}

## What the platform team needs from you

- **Use case description** — what you're using the service for
- **Expected load** — req/s, data volume, etc.
- **Critical dependencies** — what fails for your users if this service is down
- **Contact for on-call coordination** — who do we page if your usage breaks something

## When NOT to onboard

- {{anti-condition — when this service is the wrong choice}}
- {{anti-condition}}
