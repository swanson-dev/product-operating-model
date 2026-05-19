# Test: `pom-status`

## Scenario A: full portfolio status (Voyager)

**Setup:** Voyager repo at `e:\Projects\Protective\voyager\` — 1 UC, 1 product (claims, Trio incomplete), 1 Discovery item.

**Invocation:** `/pom-status e:\Projects\Protective\voyager`

**Expected outcome:**
- Skill scans the repo and produces a human-readable summary.
- Output sections:
  1. **Header**: target path, date, last activity (most recent commit or file change)
  2. **Intake**: counts of UCs by status (pending / scored / dispositioned / killed / parked / routed)
  3. **Products**: one row per product showing Trio status, Discovery count, PB count, Pod count
  4. **Recent dispositions**: last N (default 5)
  5. **Recent decisions**: last N decision-log entries across all enabling concerns
  6. **Recent ADRs**: last N across all products
  7. **Open blockers**: prominent surfaces (e.g., "Trio incomplete in 1 product", "Q4 not closeable for 1 DISC")
- Format: text-first, copy-pasteable into a stakeholder update. Tables OK, but no HTML or color codes.

**Pressure points:**
- If skill reports counts that don't match reality → FAIL (must be derived from filesystem)
- If skill fabricates an "open blocker" not supported by file contents → FAIL
- If output uses ANSI color codes or HTML → FAIL (must be plaintext-friendly)

---

## Scenario B: product-scoped summary

**Setup:** Voyager repo.

**Invocation:** `/pom-status e:\Projects\Protective\voyager --product=claims`

**Expected outcome:**
- Same shape as Scenario A but scoped to one product.
- Skip the "Intake" section (intake is portfolio-wide; not relevant to a single product).
- Add product-specific detail: roadmap state (Now / Next / Later counts), runway artifacts count, pods (with PO names).

---

## Scenario C: empty bootstrap

**Setup:** A fresh `pom-bootstrap` output with no UCs and no products.

**Invocation:** `/pom-status`

**Expected outcome:** Skill reports "Portfolio empty. Next step: ingest a UC via `pom-ingest-use-case`". Does not error.

---

## Scenario D: not a POM repo

**Setup:** Random directory.

**Invocation:** `/pom-status`

**Expected outcome:** Skill detects missing structure, suggests `pom-bootstrap`, exits cleanly.

---

## Scenario E: first `--since=last` (no prior snapshot)

**Setup:** Voyager repo with NO `.pom/snapshots/` directory yet.

**Invocation:** `/pom-status e:\Projects\Protective\voyager --since=last`

**Expected outcome:**
- Standard status report renders normally.
- Skill notes "No prior snapshot found. This run will write the first snapshot — no diff to show this time."
- Skill creates `.pom/snapshots/<ISO-timestamp>.json` with the current portfolio state.
- Snapshot JSON is pretty-printed (2-space indent).
- No CHANGES SINCE section is rendered.

**Pressure points:**
- If skill aborts on missing baseline → FAIL (first run must succeed and seed the snapshot dir)
- If skill writes the snapshot somewhere other than `.pom/snapshots/` → FAIL
- If skill modifies any portfolio artifact during this run → CATASTROPHIC FAIL (only `.pom/` is writable)

---

## Scenario F: second `--since=last` (baseline exists, real changes)

**Setup:** Voyager repo with a snapshot from yesterday at `.pom/snapshots/2026-05-18T10-00-00Z.json`. Since then: 1 new UC added, 1 DISC promoted to PB, 1 new ADR.

**Invocation:** `/pom-status e:\Projects\Protective\voyager --since=last`

**Expected outcome:**
- Standard status report renders normally.
- **CHANGES SINCE 2026-05-18T10:00:00Z** section appears with:
  - Added: 1 new UC (ID listed since count ≤ 5)
  - Promotions: 1 (DISC-ID → PB-ID)
  - Added: 1 new ADR (ID listed)
- New snapshot written for today's run.

**Pressure points:**
- If diff misses a real change → FAIL
- If diff fabricates a change not present in filesystem → FAIL
- If new snapshot is not written → FAIL (`--since` runs always seed the next diff)

---

## Scenario G: `--since=<specific-date>`

**Setup:** Voyager repo with snapshots from `2026-05-15`, `2026-05-17`, `2026-05-18`.

**Invocation:** `/pom-status e:\Projects\Protective\voyager --since=2026-05-16`

**Expected outcome:**
- Skill picks the newest snapshot with `taken_at <= 2026-05-16T23:59:59Z` → uses the `2026-05-15` snapshot as baseline.
- CHANGES SINCE section reflects diff from that baseline.
- New snapshot written.

**Pressure points:**
- If skill picks the newest snapshot regardless of date → FAIL (must honor the date cap)
- If skill picks the `2026-05-17` snapshot when user said `--since=2026-05-16` → FAIL

---

## Output format reference

Status output must be plaintext, < 80 columns wide, copy-pasteable. Structure mirrors the disposition file's "Stakeholder-facing summary" pattern — proven readable. The CHANGES SINCE block, when present, is part of the paste-ready output.
