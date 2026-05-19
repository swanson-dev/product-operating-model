# Test: `pom-platform-add-service`

## Scenario A: add an MLOps service in provisioning

**Setup:** A POM repo with `platform/` standing.

**Invocation:** `/pom-platform-add-service --area=provisioning --service=mlops --owner="platform-pod"`

**Expected outcome:**
- Skill validates the area against the canonical four (provisioning, templates-generators, governance, data-quality). For non-canonical areas, prompts user to confirm via `AskUserQuestion` (e.g., `mlops/` might be a new area).
- Copies `the pom-core plugin's methodology/templates/platform/_service-template/` → `platform/provisioning/mlops/`:
  - README.md (filled with name, owner, status)
  - intake.md (filled with default process; user can refine via AskUserQuestion)
  - consumption.md (filled with placeholder integration pattern; user refines)
- Skill prompts via `AskUserQuestion` for:
  - Service description (one paragraph)
  - Initial SLO targets (availability, latency, throughput)
  - Lead time for onboarding
- Updates `platform/README.md` to mention the new service in the provisioning area.
- Reports: file path; reminds the user to fill consumption.md examples as the service comes online.

---

## Scenario B: new area (not in canonical four)

**Invocation:** `/pom-platform-add-service --area=observability --service=metrics-pipeline`

**Expected outcome:**
- Skill detects "observability" is not in canonical four.
- Prompts via `AskUserQuestion`: "observability is a new area. Create it and add metrics-pipeline as the first service? (Yes / No — use a canonical area instead)"
- If yes: creates `platform/observability/` and proceeds.

---

## Scenario C: service already exists

**Setup:** `platform/provisioning/mlops/` exists.

**Invocation:** `/pom-platform-add-service --area=provisioning --service=mlops`

**Expected outcome:** Skill refuses to overwrite. Suggests editing the existing service files directly.

---

## Scenario D: invalid slug

**Invocation:** `/pom-platform-add-service --area=provisioning --service="MLOps Platform"` (capitals, spaces)

**Expected outcome:** Skill rejects the slug. Suggests `mlops-platform`.

---

## Reference comparison

Output service folder should match the `_service-template/` shape with README.md + intake.md + consumption.md.
