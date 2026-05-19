<purpose>
Walk a Discovery item through the four-question gate (desirable / viable / feasible / ethical), updating the Discovery file with current state, evidence, and blockers. Compute the promotion-readiness verdict.

Each question has an owner role (PD / PM / TL / Trio+Compliance). If the role isn't assigned, the question cannot close. The workflow surfaces this explicitly.

Append-only on the shaping log. Updates question-section content in place.
</purpose>

<process>

<step name="parse_args">
Parse:
- First positional: `$DISC_ID` (required) — Discovery ID (e.g., `DISC-2026-05-UC0001-claims-ai-doc-classification`) OR a full path to the DISC file
- `--question=<Q1|Q2|Q3|Q4>` (optional) — only walk a specific question
- `--all` (optional, default true if `--question` not set) — walk all four questions

If `$DISC_ID` missing, ask via `AskUserQuestion`.
</step>

<step name="locate_disc_file">
If `$DISC_ID` looks like a path, use it directly.

Otherwise, glob `products/*/discovery-backlog/$DISC_ID*.md`. Expect exactly one match.

If 0 matches: abort with "Discovery file $DISC_ID not found."
If > 1 match: list candidates and ask via `AskUserQuestion`.

Set `$DISC_PATH` and infer `$PRODUCT_SLUG` from the path.
</step>

<step name="precondition_checks">
Verify cwd is a POM repo.

Read the DISC file. Extract:
- Originating UC link (validate it resolves)
- Routing disposition link (validate it resolves)
- Receiving product slug ($PRODUCT_SLUG)
- Current 4-question states (parse the four `## Q1 / Q2 / Q3 / Q4` sections for their `Status:` line)
- Current Promotion Readiness table state

Read `products/$PRODUCT_SLUG/trio.md`. Identify which Trio roles are assigned vs `_TBD_` / `Unassigned`.

Read `products/$PRODUCT_SLUG/product.md`. Check whether vision and outcomes are still seed-state (presence of `_seed_` or `Status: seed` markers).
</step>

<step name="check_upstream_blockers">
If ALL Trio roles are unassigned:
```
Upstream blocker: Trio in products/$PRODUCT_SLUG/trio.md has no assignments.

Cannot proceed with Discovery gate — none of the four questions has an owner.

Next step: Beth Carter (or Executive Sponsor) ratifies the Trio.
```

Append a shaping log entry noting the blocker. **Stop.**
</step>

<step name="walk_question">
For each question to be walked (per `--question` or all four):

### Q1 — Desirable (PD owns)

Check PD assignment in trio.md.
- **If PD unassigned:** State the blocker. Set Q1 status to ❌. Skip the rest of Q1.
- **If PD assigned:** Use `AskUserQuestion` to ask:
  - Current state of evidence for customer desirability (journey-map done? user interviews? other signals?)
  - What specific work remains?
  - Are SMEs available and consulted?
- Based on answers, set Q1 status: ❌ (no validation yet) / 🟡 (signal but not validated) / ✅ (fully validated per `four-question-gate.md`)
- Update the Q1 section in the DISC file with current state + evidence + remaining work.

### Q2 — Viable (PM owns)

Check PM assignment AND product-level outcomes state.
- **If PM unassigned:** Blocker. Q2 = ❌. Skip.
- **If product outcomes still seed-state:** Blocker — "Q2 cannot close: product.md outcomes still seed-state, must be ratified by Trio first." Q2 = ❌.
- **Otherwise:** `AskUserQuestion`:
  - Which of the product's ratified outcomes does this Discovery item advance? (specific outcome from product.md)
  - What's the magnitude of advancement?
  - Has the PM confirmed viability with the Trio?
- Set status; update Q2 section.

### Q3 — Feasible (TL owns)

Check TL assignment.
- **If TL unassigned:** Blocker. Q3 = ❌. Skip.
- **If TL assigned:** `AskUserQuestion`:
  - POC / spike status?
  - Runway gaps: what does the runway need to provide before this can be built? Is that investment in scope of this item or a separate runway item?
  - Integration realism: what systems are touched? Are timelines and partner availability confirmed?
  - Data availability + quality?
- Set status; update Q3 section.

### Q4 — Ethical / compliant

Check whether the relevant enabling concerns exist.
- For AI/ML UCs, check `enabling/ai-ethics/`. If missing, flag as a blocker — "Q4 cannot close: enabling/ai-ethics/ does not exist. Run `pom-seed-enabling-standard --concern=ai-ethics` first."
- For other concerns relevant to the UC (security, privacy, accessibility), check their `enabling/<concern>/` existence.
- For each relevant concern, check whether the per-UC checklist has been applied (look for a runway/model-cards/ entry, a logged bias audit, etc.).
- `AskUserQuestion`:
  - Which enabling concerns apply?
  - Have the per-UC checklists been applied? Status?
  - Has Compliance signed off (if required)?
- Set status; update Q4 section.
</step>

<step name="update_promotion_readiness">
Recompute the Promotion Readiness table at the bottom of the DISC file:
- Each question status (❌ / 🟡 / ✅) based on the just-walked state
- Trio-Note gaps state (carry over from prior or update if user provides info)

Overall verdict:
- **READY** for promotion only when all 4 questions are ✅ AND Trio-Note gaps are ✅
- **NOT READY** otherwise

Update the "Overall: ..." line accordingly.
</step>

<step name="update_critical_path">
Regenerate the critical-path unblock sequence based on current state. Order:
1. Trio formation (if any role unassigned)
2. Product charter ratification (if vision/outcomes still seed)
3. Trio-Note gaps (closure with executive sponsor)
4. Runway / Platform decisions (Q3 dependencies)
5. PD journey map (Q1 closure)
6. Data Architect quality assessment (Q3)
7. Per-UC PII / bias / standards checklists (Q4)
8. Model Card or other deployment-prerequisite artifacts (Q4)
9. Promote to product-backlog/

Steps already complete are marked `(done)`. Items not applicable are removed. Active step is the first un-done one.
</step>

<step name="append_shaping_log">
Append a new dated entry to the Shaping log:
```
- **YYYY-MM-DD** — Discovery gate walkthrough run. Q1: {{new status}}; Q2: {{new status}}; Q3: {{new status}}; Q4: {{new status}}. {{Brief summary of what was decided or what blockers were surfaced.}}
```

**Never edit prior log entries. Always append.**
</step>

<step name="draft_stakeholder_summary">
Generate a draft **stakeholder-facing summary** (one paragraph the stakeholder can read directly):
- State the current gate result (READY / NOT READY).
- Name the closed questions (✅) and the open ones (🟡 / ❌) with their owners.
- If NOT READY: name the top blocker and who closes it.
- If READY: name the next action (`/pom-promote-to-backlog`).

Show the draft via chat and use `AskUserQuestion` to confirm or refine.
</step>

<step name="report">
```
✅ Discovery gate walkthrough complete for $DISC_ID.

Status:
  Q1 Desirable:           {{❌ / 🟡 / ✅}}
  Q2 Viable:              {{❌ / 🟡 / ✅}}
  Q3 Feasible:            {{❌ / 🟡 / ✅}}
  Q4 Ethical/compliant:   {{❌ / 🟡 / ✅}}
  Trio-Note gaps:         {{❌ / 🟡 / ✅}}

Overall: {{READY / NOT READY}} for promotion to Product Backlog.

Critical-path next steps:
{{Numbered list of remaining items}}

Stakeholder summary (paste-ready):
─────────────────────────────────
{{stakeholder-facing summary}}
─────────────────────────────────

{{If READY:}}
🎯 Recommended next step: `/pom-promote-to-backlog $DISC_ID`

{{If NOT READY:}}
Address the critical-path items above. Re-run `/pom-discovery-gate $DISC_ID` after each material change.
```
</step>

</process>

<guardrails>
- **NEVER mark a question ✅ without evidence captured in the DISC file.** "Fully answered" must include the answer.
- **NEVER promote an item from this skill.** Promotion is `pom-promote-to-backlog`'s exclusive job.
- **NEVER edit prior shaping log entries.** Append-only.
- If any owner role is unassigned, the corresponding question cannot close — be explicit about why.
- If product outcomes are seed-state, Q2 cannot close — be explicit.
- If a required enabling concern doesn't exist, Q4 cannot close — be explicit and suggest `pom-seed-enabling-standard`.
- Update the critical-path sequence based on current state, not stale state.
</guardrails>

<success_criteria>
- [ ] Each walked question has an updated status + evidence in the DISC file
- [ ] Promotion Readiness table reflects current state
- [ ] Critical-path sequence reflects current blockers
- [ ] Shaping log has a new dated entry summarizing the walkthrough
- [ ] Stakeholder-facing summary is paste-ready in the CLI report
- [ ] User has a clear next step (specific blocker or `pom-promote-to-backlog`)
</success_criteria>
