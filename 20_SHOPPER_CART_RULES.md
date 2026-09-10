# 20 — Shopper Cart Rules

**Authority Scope:** Cart behavior only.

_Last updated: 2026-09-09_

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

## Cart custom fields promote to order custom fields at checkout

A custom field written on the cart as `cart[custom][x]` becomes `order.custom.x` when the cart
converts to an order. It is then available in order management, in exports and in email
templates with no extra work and no mapping step.

**Order and Cart are the same custom-field object.** There is no separate Cart object to
register a field against — register the field once and it serves both.

This is the mechanism a cart-level Extra Fee already relies on: a storefront snippet writes
`cart[custom][rush]`, the Extra Fee formula reads `cart.custom.rush` at pricing time (see
`30_PRICING_ENGINE.md`), and the completed order carries `order.custom.rush` for fulfillment
and reporting.

Verified by reading source (a live production fee and the storefront field that feeds it),
2026-09-09.

## A required upload on an unselected variant branch silently blocks Add to Cart

A `required` file-upload option sitting on a variant branch the customer did **not** select
still blocks Add to Cart. There is no visible error, nothing near the button, and **no network
request is issued at all** — the browser refuses to submit an invalid form and stops there.

**Diagnostic:** upload a file to *every* branch, including the ones the customer would never
choose. If the line then adds, that is the cause.

`disable_required_form` does **not** fix it. It clears the `required` attribute; it does not
clear a `setCustomValidity()` string, so the element stays invalid and the form stays
unsubmittable.

Verified live, 2026-09-08, and independently reproduced on a second occasion. The full
detail — the option-rendering mechanism and what does clear it — lives in
`22_OPTION_VARIANT_RENDERING.md` and is not repeated here.

## Add to Cart does nothing and no network request is issued

Treat this as **form validation**, immediately. It is not a JavaScript error, an AJAX failure
or a cart-service problem: none of those can prevent the request from being made in the first
place. No request means the browser never got as far as submitting.

Go straight to the form. From any option input, `el.form` gives the handle:

```js
form.checkValidity();
Array.prototype.filter.call(form.elements, function (el) { return !el.checkValidity(); })
	.forEach(function (el) {
		console.log(el.name, el.validationMessage, el.closest('px-option'));
	});
```

Enumerate the invalid elements and resolve each one back to its enclosing `PX-OPTION` — that
names the option the shopper cannot see and cannot fill in.

Console errors present at that moment are usually a red herring. Unrelated third-party noise
is normal on a storefront page, and it reliably attracts the debugging effort away from the
actual cause.

Verified live, 2026-09-08.

## A custom-tool product must be a design product, not a static product

Static products cannot carry template options, and every cart, cart fly-out and gallery
preview block loops `orderline.chosen_template_options` to find something to draw. A static
product therefore gives those blocks nothing to iterate, and gives a custom tool no route to
attach its output.

Any product that mounts a custom design tool is a **design product** — even when the customer
never opens the editor and the artwork arrives entirely by upload.

The `chosen_template_options` loop is verified by reading source (see the cart fly-out preview
block above). That static products cannot carry template options is stated in a build spec and
is **not independently verified**.

## Gate Add to Cart with `data-requires-design`, never `requires_design: true`

To hold Add to Cart shut until a custom tool has attached artwork, put `data-requires-design`
on the **tool root** element.

**Never set `requires_design: true` on the product.** It disables the button and never
re-enables it, because it waits on a *design record* — and a tool that attaches file uploads
never creates one. The result is a store whose Add to Cart button is permanently dead, with
nothing in the console to explain it.

**The gate must fail open.** Release the button whenever the tool cannot tell whether artwork
is attached. Letting one unconfigured line through is recoverable; holding a live store's Add
to Cart shut is not.

The `requires_design: true` failure is verified live. The `data-requires-design` gate is
stated in a build spec as the established pattern.

## Changelog
- 2026-07-28: Added hide_from_cart section covering the variant/template-option cart filter and its two silent limitations (editable-cart branch, child orderlines). Source: claude-chat.
- 2026-08-29: Added the cart fly-out preview block defect — `modals/shopping-cart` assigns `flat_preview_codes` (underscore) and tests `flat-preview-codes` (hyphen), which Liquid accepts as a distinct nil variable, so every custom-tool line falls through to `px-project-preview` and shows an empty preview well; includes the one-character fix, the before/after verification, the correction that the fly-out is `modals/shopping-cart` rather than `shopper/cart-flyout`, the rule that preview-code variable spelling must be read per file as an assign/read pair, and the method lesson that a byte-identical no-regression test passes on a dead block. Source: claude-chat.
- 2026-09-09: Added that cart custom fields promote to order custom fields at checkout — `cart[custom][x]` becomes `order.custom.x`, and Order and Cart are the same custom-field object with no separate Cart object. Source: claude-chat.
- 2026-09-09: Added the silent Add to Cart blocker — a `required` file-upload option on a variant branch the customer did not select stops the form submitting with no error and no network request, and `disable_required_form` does not clear it; cross-referenced to 22_OPTION_VARIANT_RENDERING.md for the full detail. Added the diagnostic rule that Add to Cart doing nothing with no network request is form validation, to be resolved with `form.checkValidity()` and by resolving each invalid element to its enclosing `PX-OPTION`. Source: claude-chat + fireflies-call.
- 2026-09-09: Added that a custom-tool product must be a design product, because static products cannot carry template options and every preview block loops `chosen_template_options`; and that Add to Cart must be gated with `data-requires-design` on the tool root and never `requires_design: true` on the product, with the gate failing open. Source: claude-chat.
