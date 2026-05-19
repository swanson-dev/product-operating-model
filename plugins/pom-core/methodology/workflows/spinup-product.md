<purpose>
Stand up a new product folder in an existing POM repo by copying from the canonical `products/_template/` and filling seed-state placeholders.

Run when a UC has routed to a product that doesn't yet exist, or proactively when a new product line is being initiated. After completion, run `pom-route-to-discovery` if a UC is routing here.
</purpose>

<process>

<step name="parse_args">
Parse `$ARGUMENTS`:
- First positional: `$PRODUCT_SLUG` (required) — kebab-case product slug
- `--pm=<name>` (optional) — proposed Product Manager
- `--pd=<name>` (optional) — proposed Product Designer
- `--tl=<name>` (optional) — proposed Technical Leader
- `--executive-sponsor=<name>` (optional)
- `--originating-uc=<UC-NNNN>` (optional) — link the new product to a routed UC

If `$PRODUCT_SLUG` missing, ask:
```
What's the product slug? (kebab-case, lowercase, no spaces)
```
</step>

<step name="validate_slug">
Validate `$PRODUCT_SLUG`:
- Lowercase only
- Kebab-case (a-z, 0-9, hyphens; no leading/trailing hyphen)
- No spaces, no special characters
- ≥ 2 characters

If invalid:
```
Invalid product slug: '$PRODUCT_SLUG'

Slugs must be kebab-case (lowercase, hyphens for separators, no spaces).

Examples: claims, claims-operations, underwriting-auto

Suggested correction: $SUGGESTED_SLUG
```

**Stop. Do not modify anything.**
</step>

<step name="precondition_checks">
Verify cwd is a POM repo:
- `products/` directory exists
- `products/_template/` exists
- `intake/` directory exists

If not, abort:
```
Current directory is not a POM repo (missing products/, intake/, or products/_template/).

Run `pom-bootstrap <dir>` first.
```

Verify `products/$PRODUCT_SLUG/` does not already exist. If it does:
```
Product '$PRODUCT_SLUG' already exists at products/$PRODUCT_SLUG/.

Refusing to overwrite. If you intend to update this product, edit the files directly.
```

**Stop. Do not modify anything.**
</step>

<step name="copy_template">
Copy `products/_template/` → `products/$PRODUCT_SLUG/`. Preserve all subfolder structure:
- `README.md`
- `product.md`
- `trio.md`
- `roadmap.md`
- `discovery-backlog/README.md` and `DISC-template.md`
- `product-backlog/README.md` and `PB-template.md`
- `runway/README.md` and `ADR-template.md`, `PATTERN-template.md`, `runway-plan-template.md`
- `pods/_template/pod.md`

**Do NOT** copy `discovery-backlog/DISC-template.md` to a new DISC file in the product — it stays as a template.
</step>

<step name="fill_seed_values">
In `products/$PRODUCT_SLUG/`, replace placeholders:

**`README.md`:**
- `{{product-name}}` → title-cased product name (e.g., `claims` → `Claims`, `claims-operations` → `Claims Operations`)
- Add creation date

**`product.md`:**
- `{{product-name}}` → title-cased name
- Status `Created` field → today's date (ISO 8601)
- Vision, outcomes, scope sections: leave as seed-state placeholders with `_seed_` markers — do NOT invent content

**`trio.md`:**
- Title row keeps `{{product-name}}` shape
- If `--pm` provided, fill PM name field with `<name> (proposed, awaiting confirmation)`
- If `--pd` provided, fill PD name
- If `--tl` provided, fill TL name
- If `--executive-sponsor` provided, fill that row
- Unassigned roles remain `_TBD_` and a "Blocking risks" section lists them as Discovery-gate blockers
- Add operating cadence section with recommended defaults

**`roadmap.md`:**
- `{{product-name}}` → title-cased name
- Now: if `--originating-uc` provided, add a row pointing to the upcoming Discovery item path; otherwise empty
- Next, Later, Won't: leave empty (Trio populates)
- Open horizon questions: include the generic 3 (scope boundary, runway-vs-platform investment, resourcing horizon)

**Subdirectories** (`discovery-backlog/`, `product-backlog/`, `runway/`, `pods/`):
- Leave their README.md files (carried from template)
- Leave them empty otherwise — no Discovery items, no PB items, no pods yet
</step>

<step name="update_portfolio_index">
Open `products/README.md`. Locate the portfolio index table. Add a row for `$PRODUCT_SLUG`:

| Product | Trio status | Pods | Discovery items | Product Backlog | Notes |
|---|---|---|---|---|---|
| [**$PRODUCT_SLUG**]($PRODUCT_SLUG/) | 🟡 PM proposed; PD/TL unassigned (or ✅ complete) | 0 | 0 (or 1 if --originating-uc provided) | empty | Created YYYY-MM-DD |

If the table is "_(none yet)_" or empty, replace the empty-state placeholder with the first real row.
</step>

<step name="verify">
Run pom-validate-lite on `products/$PRODUCT_SLUG/`. Expect:
- R3.3: required files/dirs all present → ERROR if not
- R3.4: product.md required fields present → ERROR if not
- R3.5: Trio assignments → WARN if unassigned roles (expected and OK)
- All other R3 checks: PASS (empty backlogs and pods are valid)

If R3.3 or R3.4 fail, abort with file-list of what's missing.
</step>

<step name="report">
Output:

```
✅ Product '$PRODUCT_SLUG' created.

Files:
- products/$PRODUCT_SLUG/README.md
- products/$PRODUCT_SLUG/product.md (seed charter)
- products/$PRODUCT_SLUG/trio.md (Trio incomplete: {{unassigned roles}})
- products/$PRODUCT_SLUG/roadmap.md (Now-only; Next/Later empty)
- products/$PRODUCT_SLUG/discovery-backlog/ (empty)
- products/$PRODUCT_SLUG/product-backlog/ (empty)
- products/$PRODUCT_SLUG/runway/ (empty)
- products/$PRODUCT_SLUG/pods/ (empty)
- products/README.md (portfolio index updated)

Next steps:
1. Confirm Trio assignments — PD and TL MUST be named before Discovery items can promote.
2. {{If --originating-uc:}} Run `pom-route-to-discovery $ORIGINATING_UC $PRODUCT_SLUG` to create the Discovery file.
3. Ratify product charter (vision + outcomes) in product.md — currently seed-state.
4. Set Trio operating cadence in trio.md.
```
</step>

</process>

<guardrails>
- NEVER overwrite an existing `products/<slug>/` directory.
- NEVER skip slug validation — invalid slugs break cross-references later.
- ALWAYS update `products/README.md` portfolio index after creating a new product.
- Do NOT invent vision/outcomes/scope content — those are Trio decisions; leave seed markers.
- Do NOT fill PD/TL fields if not provided — make the gap visible.
</guardrails>

<success_criteria>
- [ ] `products/$PRODUCT_SLUG/` directory exists with required structure (R3.3)
- [ ] `product.md` has required fields filled with seed/known values (R3.4)
- [ ] `trio.md` reflects any provided assignments
- [ ] Portfolio index in `products/README.md` lists the new product
- [ ] `pom-validate` passes (R3.5 WARN-only if Trio incomplete is acceptable)
</success_criteria>
