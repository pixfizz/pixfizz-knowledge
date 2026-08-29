# 20 — Shopper Cart Rules

**Authority Scope:** Cart behavior only.

_Last updated: 2026-02-26_

---

# 04 — Shopper Cart Rules

## Options visibility/editability
- Options visible unless image/file upload type.
- Editable in cart only if site-wide cart setting allows; otherwise change via project-edit/editor/photo prints UI.

## Hiding options and variants from the cart line

A boolean custom field `hide_from_cart` exists on both **variants** and **template options**. Where it is set, `product/px-option-cart` skips that entry when rendering the cart line. Use it for machine-set or internal values — tool settings, cut offsets, injected file references — that should not be shown to the shopper.

Two limitations apply, and both fail silently:

- **The editable cart branch ignores it.** When `admin/checklist/cart-editable-options` is `TRUE`, the cart renders options through `px-option-selector` / `px-option-selector-alt` rather than the loop, and those components do not honour `hide_from_cart`. A value correctly hidden in read-only mode reappears the moment a site turns on editable cart options.
- **Child orderlines are not filtered at all.** Options on child orderlines render unconditionally regardless of the field.

`hide_from_cart` must exist as a custom field on the specific site — custom fields do not inherit parent to child (see **13_TEMPLATE_BOUNDARIES**). Before adding a new "hide this from the cart" field, check whether `hide_from_cart` already covers the case; creating a duplicate field is a common and avoidable mistake.

## Photo prints
- Quantity not editable in cart.
- Quantity is per-photo in Photo Prints UI; orderline priced via `cut_print_quantity`.

## Pricing display
- Pricing generally visible; tiered pricing may show strikethrough.

## Digital-only
- No special cart behavior.

## The Cart Fly-Out Preview Block Has Never Rendered for a Custom Tool

Found 2026-08-27. **Live parent bug, not a documentation error.** Affects every
custom design tool, in the cart fly-out only.

`modals/shopping-cart` assigns the preview code list with an **underscore** and
tests it with a **hyphen**:

```liquid
{%- assign flat_preview_codes = 'sticker_preview,gangup_preview,bc_preview,facefan_preview,pu_preview,cvs_preview' | split: ',' -%}
{%- assign flat-preview-url = '' -%}
{%- assign flat-preview-class = 'img-fluid' -%}
{%- for opt in orderline.chosen_template_options -%}
    {%- if flat-preview-codes contains opt.template_option.code and opt.uploaded_file.url != blank -%}
```

`flat-preview-codes` is never assigned in this snippet. Liquid has no arithmetic
operators, so a hyphen is a legal identifier character — `flat-preview-codes` is a
valid variable name that happens to be nil. `contains` against nil is false for
every code, `flat-preview-url` stays blank, and **every line falls through to
`px-project-preview`**, which has nothing to draw for a custom-tool product.

The other two variables in the same block are hyphenated and internally consistent.
Only the codes list is mismatched.

**Blast radius:** every tool in the list shows an empty preview well in the cart
fly-out. The cart page itself is fine — it assigns `flat-preview-codes` with a
hyphen, matching its own read.

**Fix — one character, in `modals/shopping-cart`:** change the **assign** to
`flat-preview-codes`, not the read. The read is consistent with the two other
hyphenated variables in the block, so this is the smaller and safer edit.

Verified by extracting the block verbatim and rendering it through a Liquid engine
before and after, against three orderline shapes: a custom-tool line with an
uploaded preview code rendered `px-project-preview` (blank) before and an `<img>`
with the uploaded URL after, and an ordinary project line with no matching code was
**byte-identical** in both — which is the property that matters for a shared file.

### Two corrections that follow from it

1. **The fly-out snippet path.** Several build specs name it `shopper/cart-flyout`.
   On this parent it is **`modals/shopping-cart`**.
2. **The variable spelling is not a reliable identifier.** Those specs record cart
   and cart fly-out as using `flat-preview-codes` (hyphen) and projects/gallery as
   using `flat_preview_codes` (underscore). That is wrong here, and the deeper point
   is that the spelling is **per file and per line**: the `assign` and the
   `contains` must be read as a pair in the file being edited.

### The method lesson

Every spec that touched these files verified the edit the same way — render before
and after across many orderline shapes, assert byte-identical output for every shape
that is not the tool's own. **That test passes on a dead block.** Adding a token to a
list that is never read is byte-identical for every shape, including the tool's own.
Four tools were installed through this check and none of them caught it.

**Rule: when extending a preview-code list, assert that the tool's own line now
renders an `<img>` with the expected URL.** The no-regression assertion is necessary
and is not sufficient — assert the customer-visible outcome, not the input.

**Second, smaller finding:** `pu_preview` is in the cart and fly-out lists but not in
`account/v2/projects` or `product/gallery`. Whichever tool owns that code shows a
thumbnail in the cart and an empty well in saved projects. Worth adding to both
lists, or confirming it is deliberate.

## Changelog
- 2026-07-28: Added hide_from_cart section covering the variant/template-option cart filter and its two silent limitations (editable-cart branch, child orderlines). Source: claude-chat.
- 2026-08-29: Added the cart fly-out preview block defect — `modals/shopping-cart` assigns `flat_preview_codes` (underscore) and tests `flat-preview-codes` (hyphen), which Liquid accepts as a distinct nil variable, so every custom-tool line falls through to `px-project-preview` and shows an empty preview well; includes the one-character fix, the before/after verification, the correction that the fly-out is `modals/shopping-cart` rather than `shopper/cart-flyout`, the rule that preview-code variable spelling must be read per file as an assign/read pair, and the method lesson that a byte-identical no-regression test passes on a dead block. Source: claude-chat.
