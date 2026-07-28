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

## Changelog
- 2026-07-28: Added hide_from_cart section covering the variant/template-option cart filter and its two silent limitations (editable-cart branch, child orderlines). Source: claude-chat.
