<purpose>
Stand up a new pod for a product. Copies the pod template, fills the roster (SM + PO + 2-5 members), confirms surface area, sets cadence defaults.

Enforces composition rules from `pod-composition-rules.md`: 2-7 members, exactly one SM, exactly one PO. Refuses non-compliant pods without `--force`.
</purpose>

<process>

<step name="parse_args">
Parse:
- First positional: `$PRODUCT_SLUG` (required)
- Second positional: `$POD_SLUG` (required)
- `--sm=<name>` (required; prompted if missing)
- `--po=<name>` (required; prompted if missing)
- `--members=<comma-separated-names>` (required; prompted if missing)
- `--surface=<text>` (optional; prompted if missing — what the pod owns)
- `--force` (optional) — allow non-compliant composition (≤ 1 or > 7 members)
</step>

<step name="validate_slugs">
Validate both `$PRODUCT_SLUG` and `$POD_SLUG` are kebab-case. Abort with corrected suggestion if invalid.
</step>

<step name="precondition_checks">
Verify `products/$PRODUCT_SLUG/` exists. If not, abort:
```
Product products/$PRODUCT_SLUG/ does not exist.

Run `/pom-spinup-product $PRODUCT_SLUG` first.
```

Verify `products/$PRODUCT_SLUG/pods/$POD_SLUG/` does NOT exist. If it does, abort with conflict message.

Verify `products/$PRODUCT_SLUG/pods/_template/` or `the pom-core plugin's methodology/templates/products/_template/pods/_template/` exists for copying.
</step>

<step name="validate_composition">
Count members: parse `--members` by comma. Compute total = 1 (SM) + 1 (PO) + member count.

If total < 2 or total > 7:
- Without `--force`: abort:
  ```
  Pod composition violates R3.10: $TOTAL members (SM + PO + N members).

  Allowed range: 2-7 total per `pod-composition-rules.md`.

  To proceed anyway: re-run with `--force`. The pod.md will include a WARN annotation about the violation.
  ```
- With `--force`: warn and proceed.
</step>

<step name="copy_pod_template">
Copy the pod template tree to `products/$PRODUCT_SLUG/pods/$POD_SLUG/`:
- `pod.md`
- `pod-backlog/` (empty)
- `slices/` (empty)

Source: `products/$PRODUCT_SLUG/pods/_template/` if exists, else `the pom-core plugin's methodology/templates/products/_template/pods/_template/`.
</step>

<step name="fill_pod_md">
Edit `products/$PRODUCT_SLUG/pods/$POD_SLUG/pod.md`:

- `{{pod-name}}` → title-cased pod slug
- SM row → `$SM_NAME`
- PO row → `$PO_NAME`
- Member rows → expand `$MEMBERS` list, one row each
- "What this pod owns" section → fill with `$SURFACE` (prompted via `AskUserQuestion` if not provided)
- Cadence defaults filled:
  - Standup: daily
  - Planning: 2 weeks
  - Review/retro: end of cadence
- Working agreements: leave as "[document anything non-obvious about how this pod operates]"

If composition was forced out of range, add a "Composition exception" note at the top:
```
> ⚠️ Composition exception: $TOTAL members. Per `pod-composition-rules.md` R3.10, recommended range is 2-7. Justification (to fill): _______
```
</step>

<step name="update_portfolio_index">
Edit `products/README.md`. Update the row for `$PRODUCT_SLUG`: increment Pods count.
</step>

<step name="report">
```
✅ Pod '$POD_SLUG' formed in product '$PRODUCT_SLUG'.

Files:
- products/$PRODUCT_SLUG/pods/$POD_SLUG/pod.md
- products/$PRODUCT_SLUG/pods/$POD_SLUG/pod-backlog/ (empty)
- products/$PRODUCT_SLUG/pods/$POD_SLUG/slices/ (empty)
- products/README.md (portfolio index updated)

Composition:
  SM: $SM_NAME
  PO: $PO_NAME
  Members: $MEMBERS
  Total: $TOTAL {{(WARN: outside 2-7 range)}}

Surface: $SURFACE

Next steps:
1. PO Sync orders the Product Backlog. Pod pulls from the top at its cadence.
2. Pod documents working agreements in pod.md (pairing defaults, code review SLA, on-call rotation).
3. First vertical slice goes into `slices/<slice-slug>/`.
```
</step>

</process>

<guardrails>
- **NEVER create a pod with > 7 members without `--force` + warning annotation.** The size cap is load-bearing per the methodology.
- **NEVER skip the SM or PO assignment.** Pods cannot function without both.
- **NEVER overwrite an existing pod folder.** Conflicts must be explicit.
- Always update `products/README.md` portfolio index.
</guardrails>

<success_criteria>
- [ ] Pod folder created with `pod.md`, `pod-backlog/`, `slices/`
- [ ] SM + PO + members filled in pod.md
- [ ] Surface area documented
- [ ] Cadence defaults set
- [ ] Portfolio index updated
- [ ] `pom-validate` passes (R3.9 met; R3.10 OK if 2-7 members)
</success_criteria>
