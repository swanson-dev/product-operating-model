<purpose>
Scaffold a new portfolio-shared platform service under one of the canonical areas (provisioning, templates-generators, governance, data-quality) or a new area. Copies the `_service-template/` and fills README + intake + consumption files.

Services are **consumed, not mandated** — this skill provides the structural scaffolding, not the implementation.
</purpose>

<process>

<step name="parse_args">
Parse:
- `--area=<slug>` (required) — one of: provisioning, templates-generators, governance, data-quality, OR a new area slug
- `--service=<slug>` (required) — kebab-case service slug
- `--owner=<text>` (optional; prompted) — pod or individual; portfolio-level if cross-product
- `--status=<value>` (optional, default `proposed`) — proposed / building / available / deprecated

If `--area` or `--service` missing, ask via `AskUserQuestion`.
</step>

<step name="validate_slugs">
Both must be kebab-case. Abort with corrected suggestion if invalid.
</step>

<step name="precondition_checks">
Verify cwd is a POM repo (`platform/` exists, `platform/README.md` present).
</step>

<step name="check_area">
Canonical areas: `provisioning`, `templates-generators`, `governance`, `data-quality`.

If `--area` is in the canonical four:
- Verify `platform/$AREA/` exists. If not, create it with a stub README (read the parent platform/README.md to understand the area's purpose; write a 2-3 line README for the new area folder).

If `--area` is NOT in the canonical four:
- Use `AskUserQuestion`:
  - "Area '$AREA' is not in the canonical four (provisioning, templates-generators, governance, data-quality). Create as a new portfolio-level area? (Yes / Use canonical area instead)"
- If yes: create `platform/$AREA/` with a brief README documenting the new area's purpose (user provides one paragraph).
- If no: prompt for which canonical area instead.
</step>

<step name="check_service_collision">
Verify `platform/$AREA/$SERVICE/` does NOT exist. If it does, refuse:
```
Service platform/$AREA/$SERVICE/ already exists.

To update, edit the existing files directly. To deprecate, set Status field to 'deprecated' and document a retirement timeline.
```
</step>

<step name="copy_template">
Copy `the pom-core plugin's methodology/templates/platform/_service-template/` → `platform/$AREA/$SERVICE/`:
- `README.md`
- `intake.md`
- `consumption.md`
</step>

<step name="gather_fields">
Use `AskUserQuestion` to collect:

1. **Service description** — one paragraph: capability provided + problem solved
2. **SLO targets** — availability (e.g., 99.9%), latency (p95 target), throughput
3. **Onboarding lead time** — expected time from request to first consumption
4. **Integration pattern** — high-level shape (client library, API, message queue, etc.)
5. **Failure modes** — at least 2 with consumer-side fallback
</step>

<step name="fill_files">
**README.md:**
- `{{service-name}}` → title-cased service slug
- Service slug, Area, Owner, Status fields filled
- Created = today
- Last reviewed = today
- "What this service does" → user-provided description
- "Products currently consuming" → empty table (filled as products onboard)
- SLO targets table filled

**intake.md:**
- `{{service-name}}` → title-cased
- Lead time filled
- "When NOT to onboard" → at least one anti-condition (prompt user if not provided)

**consumption.md:**
- `{{service-name}}` → title-cased
- Integration pattern documented (1-2 paragraphs)
- Example code/config block (placeholder for now if not yet implemented)
- Failure modes table filled with user-provided modes
- Versioning section → placeholder ("TBD as the service comes online")
</step>

<step name="update_platform_readme">
Edit `platform/README.md`. If there's a section or table listing services per area, add a row/bullet for the new service. If not, append a brief "Active services" section at the bottom.

Reference: the parent `platform/README.md` template lists the four canonical areas but not specific services — adding services is each repo's evolving record.
</step>

<step name="report">
```
✅ Platform service '$SERVICE' scaffolded under '$AREA'.

Files:
- platform/$AREA/$SERVICE/README.md
- platform/$AREA/$SERVICE/intake.md
- platform/$AREA/$SERVICE/consumption.md
- platform/README.md (services list updated)

Status: $STATUS

Next steps:
1. Implement the service — the scaffolded files document the intended contract, not the implementation.
2. As products onboard, add them to README.md's "Products currently consuming" table.
3. Fill consumption.md with real integration examples once the service is in use.
4. Update Status field to 'building' → 'available' as the service matures.
```
</step>

</process>

<guardrails>
- **NEVER overwrite an existing service folder.** Updates are manual edits.
- ALWAYS update `platform/README.md` to surface the new service.
- For non-canonical areas, require explicit user confirmation — don't proliferate areas silently.
- Initial Status defaults to `proposed`. The user advances it as the service matures.
- Do NOT pretend the service is implemented — files document the **contract** for consumers to plan against.
</guardrails>

<success_criteria>
- [ ] Service folder created with README, intake, consumption
- [ ] README has all required fields (R5.2, R5.3)
- [ ] platform/README.md surfaces the new service
- [ ] `pom-validate` continues to pass
</success_criteria>
