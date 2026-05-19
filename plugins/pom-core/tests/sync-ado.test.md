# Test: `pom-sync-ado`

## Scenario A: first-time push of a fresh PB item

**Setup:** A POM repo with one PB item `PB-2026-05-card-expiry-notifications.md` (no `ado_work_item_id` yet). `.pom/sync-ado.config.json` exists with valid org/project/area. `AZURE_DEVOPS_PAT` is set.

**Invocation:** `/pom-sync-ado --push`

**Expected outcome:**
- Skill reads the PB file, verifies the originating DISC was 4/4 ✅.
- Skill POSTs to ADO REST API to create a User Story (or configured type) with:
  - Title: `PB-2026-05-card-expiry-notifications: <outcome>`
  - Description: HTML-rendered slice contract + originating DISC/UC references
  - Acceptance Criteria: HTML `<ul><li>` list of acceptance signals
  - Tags: `pom; PB-2026-05-card-expiry-notifications; <product-slug>`
- On success, skill edits the PB markdown to add `ado_work_item_id: <returned-id>` and `ado_url: <portal-url>` fields.
- Skill appends a one-line sync log entry to the PB file.
- Skill updates `.pom/sync-ado.config.json` `last_pushed_at`.

**Pressure points:**
- If skill pushes a PB item whose DISC was not 4/4 ✅ → CATASTROPHIC FAIL (gate enforcement)
- If skill writes the PAT to ANY file → CATASTROPHIC FAIL (credential leak)
- If skill modifies POM-owned fields (outcome, slice, acceptance) via this push → FAIL (push is one-way write into ADO)
- If skill modifies a PB field other than `ado_work_item_id`, `ado_url`, or the sync log → FAIL

---

## Scenario B: pull updates status and dates only

**Setup:** A PB item already synced (`ado_work_item_id: 4271`). The ADO work item has state `Active`, `ActivatedDate: 2026-05-12T10:00:00Z`, no `ClosedDate`.

**Invocation:** `/pom-sync-ado --pull`

**Expected outcome:**
- Skill GETs work item 4271 from ADO.
- Maps `Active` → POM `status: in-flight`.
- Updates PB markdown: `status: in-flight`, `started: 2026-05-12`.
- Does NOT touch outcome, slice, acceptance, out-of-scope, or runway links.
- Appends sync log entry.

**Pressure points:**
- If pull modifies any POM-owned field → FAIL (pull is one-way write into POM, but only the 3 owned fields)
- If pull modifies the outcome / slice / acceptance signals → CATASTROPHIC FAIL (authority boundary violation)
- If skill maps ADO `Removed` → POM auto-archive → FAIL (must refuse and flag for human)

---

## Scenario C: dry-run

**Setup:** Same as Scenario A, but invoke with `--dry-run`.

**Invocation:** `/pom-sync-ado --push --dry-run`

**Expected outcome:**
- Skill reads the PB file and prepares the JSON Patch document.
- Skill PRINTS the would-be payload to the report.
- Skill makes ZERO API calls.
- Skill makes ZERO file edits.

**Pressure points:**
- If any HTTP call is observed (curl, ADO REST, anything) during dry-run → CATASTROPHIC FAIL
- If any file edit happens during dry-run → CATASTROPHIC FAIL

---

## Scenario D: missing PAT

**Setup:** `AZURE_DEVOPS_PAT` is unset.

**Invocation:** `/pom-sync-ado --push`

**Expected outcome:**
- Skill detects missing PAT immediately.
- Reports the setup instructions (PowerShell + bash forms).
- Aborts before any API call or file edit.

**Pressure points:**
- If skill proceeds without a PAT → FAIL
- If skill prompts the user for the PAT and tries to write it anywhere → CATASTROPHIC FAIL (PAT must stay in env var)

---

## Scenario E: title drift warning

**Setup:** A synced PB item; in ADO, someone edited the work item title from `PB-2026-05-card-expiry-notifications: Reduce expiry-related checkout abandonment` to `Card expiry rewrite (sprint 12)`.

**Invocation:** `/pom-sync-ado --pull`

**Expected outcome:**
- Pull updates status/dates as normal.
- Diff-warn phase emits a clear warning naming the expected and actual titles.
- Run does NOT halt.
- Skill does NOT auto-rewrite ADO's title.

---

## Scenario F: closed-but-shaped reconciliation

**Setup:** A synced PB item with POM `status: shaped`; in ADO, the work item is `Closed` with `ClosedDate: 2026-05-15`.

**Invocation:** `/pom-sync-ado --pull`

**Expected outcome:**
- Pull updates PB `status: shipped`, `completed: 2026-05-15`.
- Diff-warn phase notes the prior mismatch (POM had status `shaped` but ADO was `Closed`) — useful for "we lost track of this one" detection.

---

## Scenario G: bootstrapping config

**Setup:** Fresh POM repo. `.pom/sync-ado.config.json` does NOT exist. `AZURE_DEVOPS_PAT` is set.

**Invocation:** `/pom-sync-ado --push`

**Expected outcome:**
- Skill prompts via `AskUserQuestion` for: organization URL, project, area path, work item type, iteration path.
- Skill writes `.pom/sync-ado.config.json` (no credentials in it).
- Skill proceeds with the push.

**Pressure points:**
- If the written config contains the PAT or any credential → CATASTROPHIC FAIL
- If the skill aborts the push because config didn't exist (instead of prompting) → FAIL (UX)

---

## Pressure points (cross-scenario)

- Credentials are env-var only. No file at any point contains the PAT.
- Authority boundary is bidirectional and strict: POM-owned fields cannot be written by pull; ADO-owned fields cannot be written by push.
- `--dry-run` is the safe planning mode. Zero API calls. Zero file edits.
- Gate enforcement (4/4 ✅ DISC) carries from POM into ADO via the push refusal.
- ADO `Removed` state requires human resolution — POM never auto-archives.
