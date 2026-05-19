---
name: pom-platform-add-service
description: "Use when a new portfolio-shared platform service needs scaffolding (provisioning, templates-generators, governance, data-quality, or a new area). Creates README + intake + consumption files documenting the service contract for consumers to plan against."
argument-hint: "--area=<area-slug> --service=<service-slug> [--owner=<text>] [--status=<proposed|building|available|deprecated>]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Scaffold a new portfolio-shared platform service. Platform services are **consumed, not mandated** — this skill creates the structural shell that documents the service contract so products can plan their integration. Implementation is separate work.

**Canonical areas:** `provisioning`, `templates-generators`, `governance`, `data-quality`. New areas allowed with explicit confirmation.

**Creates:**
- `platform/<area>/<service-slug>/README.md` (name, owner, status, products consuming, SLO targets)
- `platform/<area>/<service-slug>/intake.md` (how products request use)
- `platform/<area>/<service-slug>/consumption.md` (integration pattern, SLAs, failure modes, fallbacks)

**Updates:**
- `platform/README.md` to surface the new service in its area

**Refuses** to overwrite an existing service. To update, edit files directly.

**Status lifecycle:** `proposed` → `building` → `available` → `deprecated`. Default on creation: `proposed`.

**After this command:**
1. Implement the service — scaffolded files document the **contract**, not the implementation.
2. As products onboard, add them to README.md's "Products currently consuming" table.
3. Fill `consumption.md` with real integration examples as the service comes online.
4. Advance Status field through the lifecycle.
</objective>

<execution_context>
@${CLAUDE_PLUGIN_ROOT}/methodology/workflows/platform-add-service.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/platform-service-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/references/validation-rules.md
@${CLAUDE_PLUGIN_ROOT}/methodology/templates/platform/_service-template/README.md
</execution_context>

<process>
Execute end-to-end.
Preserve all workflow gates (area validation, slug validation, collision check, new-area confirmation).
</process>
