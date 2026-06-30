# 41 --- Implementation Patterns

**Authority Scope:** Reusable architectural patterns for Pixfizz
storefronts.

*Last updated: 2026-06-30*

------------------------------------------------------------------------

# Pixfizz Implementation Patterns

This document captures **real-world patterns used repeatedly when
building Pixfizz storefronts**.

These patterns represent practical knowledge beyond API documentation.

------------------------------------------------------------------------

# Email Rendering Patterns

## Project Preview Image

For design products:

``` liquid
<img src="https://{{ website.hostname }}{{ orderline.project | preview_url: height: 300, share: orderline.project.share_code }}">
```

Key rule:

Project previews in email must include the **share code**.

------------------------------------------------------------------------

## Product Image Fallback

Static products should fall back to the product image.

``` liquid
<img src="{{ orderline.product.image | asset_url:200, cdn:false }}">
```

------------------------------------------------------------------------

# Cart Rendering Patterns

## Looping Orderlines

Correct loop structure:

``` liquid
{% for orderline in cart.orderlines %}
...
{% endfor %}
```

Orderlines are the **atomic commerce unit** in Pixfizz.

------------------------------------------------------------------------

# Inventory UI Pattern

Example low-inventory badge:

``` html
<span class="px-stock-badge px-stock-low">
Only <b>{{ current_inventory }}</b> left
</span>
```

Recommended logic:

-   show badge when inventory < threshold
-   hide when inventory unlimited

------------------------------------------------------------------------

# Dynamic UI Trigger Pattern

Avoid inline JS triggers.

Instead:

1.  Render a DOM marker
2.  Let global JS detect the marker

Example marker:

``` html
<div id="px-rush-sameday-marker"></div>
```

This pattern works reliably with AJAX snippet updates when the snippet
only needs to signal presence to a global script.

For snippets that need to carry Liquid-baked values into JS, use the
`style onload` pattern below instead.

------------------------------------------------------------------------

# `style onload` Re-injection Pattern

## When to use

Use this pattern for any JS that must run on **both** initial page load
and every subsequent AJAX re-injection — for example:

-   Checkout snippets inside `async: true` forms
-   Snippets that re-render when the user switches delivery options
-   Any snippet where `<script>` tags are not reliably re-executing

`<script>` tags inside AJAX-injected HTML do not re-execute. The
`style onload` attribute fires every time the element is inserted into
the DOM, including after re-injection.

## Pattern

``` liquid
{% assign saved_value = form.values.cart.custom.my_field | default: cart.custom.my_field %}

{% capture my_init_script %}
(function() {
	var savedValue = '{{ saved_value | escape }}';

	// setup and event listeners here

	// Always call init directly — DOMContentLoaded does not re-fire on AJAX:
	initMyThing();
})();
{% endcapture %}

<style onload="{{ my_init_script | escape }}"></style>
```

## Rules

-   The `{% capture %}` and `<style onload>` blocks must sit **outside**
    any `<script>` tag. Placing them inside a `<script>` block causes a
    syntax error and breaks all JS on the page.
-   Always include a **direct call** to the init function at the end of
    the IIFE. Do not rely solely on `DOMContentLoaded`, `pageshow`, or
    `px.fragmentsReloaded` event listeners — they are not sufficient for
    AJAX re-injection.
-   Bake Liquid server values into the IIFE via `{{ variable | escape }}`
    so they are available without additional API calls.
-   Use `removeEventListener` before `addEventListener` on any element
    that persists across re-injections, to avoid stacking duplicate
    handlers.

## Separation from once-only logic

Keep logic that should run **once** on full page load (e.g. postcode
lookup, checkbox validation) in a separate `<script>` block with
`DOMContentLoaded`. Do not merge it into the `style onload` capture.

``` liquid
{% capture my_init_script %}
(function() {
	// Runs on every injection
})();
{% endcapture %}
<style onload="{{ my_init_script | escape }}"></style>

<script>
document.addEventListener('DOMContentLoaded', function() {
	// Runs once on full page load only
});
</script>
```

## Once-Per-Load Guard Pattern (`window.__flag`)

### When to use

Use when a snippet lives inside an async-reinjected section (e.g. `selectors: 'section.checkout-page'`) but contains logic that must fire **only once per full page load** — not on every re-injection. Examples:

- Advisory or welcome modals
- One-time analytics events
- Initialisation that would be disruptive if repeated

The `style onload` pattern fires on every DOM injection by design. Without a guard, the logic runs every time the user switches delivery options.

### Pattern
```liquid
{% capture px_once_init %}
(function() {
	if (window.__pxMyThingShown) return;
	window.__pxMyThingShown = true;

	// logic that should run only once per page load
	setTimeout(function() {
		var el = document.getElementById('my-element');
		if (!el) return;
		$(el).modal('show');
	}, 3000);
})();
{% endcapture %}
<style onload="{{ px_once_init | escape }}"></style>
```

### Rules

- Use a flag name specific to the feature (`window.__pxAdvisoryShown`, `window.__pxWelcomeShown`) — avoid generic names that could collide across snippets.
- The guard must be the **first line** of the IIFE so nothing runs before the check.
- The flag persists for the lifetime of the page session and is cleared on full page reload — which is the intended behaviour.
- Do not use this pattern for logic that genuinely needs to re-run on re-injection (state restore, dropdown population, etc.) — use the plain `style onload` IIFE for those.

### Variant: `sessionStorage` for tab-session persistence

`window.__flag` is cleared on full page reload. If the guard needs to persist across page loads within the same browser tab (e.g. a modal that should not re-show even if the user navigates away and returns during the same session), use `sessionStorage` instead:

```javascript
(function() {
	if (sessionStorage.getItem('pxMyThingShown')) return;
	sessionStorage.setItem('pxMyThingShown', '1');

	// logic here
})();
```

`sessionStorage` is cleared when the tab is closed but persists across page navigations within that tab.

------------------------------------------------------------------------

# Defensive Snippet Architecture

When editing widely reused snippets:

-   preserve variable names
-   preserve JS hooks
-   add logic without breaking existing behavior

This keeps sites stable across updates.

------------------------------------------------------------------------

# `px-project-preview` — Shadow DOM Image Styling

The `px-project-preview` web component renders its image inside a shadow DOM. Setting `height: 100%` or `object-fit` on the component element itself has no effect on the internal image.

Use the `::part(img)` CSS pseudo-element, which the component exposes as a named part:

```css
px-project-preview::part(img) {
	object-fit: cover;
	height: 100%;
	width: 100%;
}
```

This is the correct approach whenever the component's image does not fill its container as expected.

------------------------------------------------------------------------

# Hero Background Overlay — Preferred Pattern

Do not use `background-blend-mode: multiply` combined with Bootstrap's `bg-dark` class for hero image overlays. The blend mode cannot be tuned and produces excessively dark results.

Use a semi-transparent `rgba` overlay `<div>` with `position: absolute` instead:

```html
<div style="position: absolute; inset: 0; background: rgba(0,0,0,0.35); z-index: 1;"></div>
```

- Starting point: `rgba(0, 0, 0, 0.35)`. Adjust the alpha value to taste.
- The hero container must be `position: relative`.
- Ensure text content has a higher `z-index` than the overlay.

------------------------------------------------------------------------

# Skip Cart Redirect Pattern (`product.custom.skip_cart_redirect`)

## When to use

Use when a customer wants the shopper to stay on the product page after
adding an item to cart, instead of being redirected to `/site/cart`.

Enabled by setting the **boolean** custom field `skip_cart_redirect` on the
product. Works for both static and design products.

## How it works

### Static products (`cart_add_product` form)

The `page: 'cart'` form parameter renders as
`<input type="hidden" name="target" value="/site/cart">`. The button
`onclick` calls `enableSkipCartRedirect(form)` which overwrites the
`target` input value with the current page URL +
`?product_added_to_cart=t`, so the platform redirects back to the
product page.

### Design products (`project_create` form)

A disabled hidden `<input name="target">` is added inside the form. The
button `onclick` (inside the `btn_add_to_cart` conditional) calls
`enableSkipCartRedirect(form)` which enables the input and sets its
value. The platform picks up the `target` field and redirects back.

### Quick-quantity products

The quick-quantity JS checks `window.pxSkipCartRedirect`. If true, it
calls `showPxCartToast()` instead of redirecting to `/site/cart` after
the fetch calls complete.

### Visual feedback

A Bootstrap 4.6 toast notification with a green checkmark appears when
the `product_added_to_cart=t` URL parameter is detected on page load.
The script also suppresses the platform's built-in flash message via a
MutationObserver. The URL parameter is cleaned via
`history.replaceState` so a page refresh does not re-trigger the toast.

## Key implementation details

-   `enableSkipCartRedirect(form)` finds `input[name="target"]` by name
    (not ID) — works for both static (platform-generated) and design
    (manually added) target inputs.
-   The toast HTML and script sit outside `{% unless product.is_static %}`
    so they render for both product types.
-   `window.pxSkipCartRedirect` is set in the inventory flags script block.
-   The toast uses an IIFE for immediate flash suppression + URL cleanup,
    with `$(function() { ... })` for the actual toast show (requires
    Bootstrap JS to be loaded).

## Dependencies

-   Requires `btn_add_to_cart` (product or collection level) for design
    products — otherwise the button goes into Design Now mode and the
    `enableSkipCartRedirect` onclick never fires.
-   Not required for static products — they are always add-to-cart.

------------------------------------------------------------------------

# Custom Type Collection: Sort + Filter Pattern

When you need to both **sort** and **filter** Custom Type instances (e.g. date-based filtering, unpublished flags), the Liquid `sort` filter cannot be applied to the push-built filtered array — it silently fails on plain arrays with nested dot-notation keys.

**Correct pattern:** sort at initial collection assignment, then push-filter preserving that order.
```liquid
{% assign all_items = website.custom_types['my_type'] | sort: 'custom.my_date_field' | reverse | page_size: 9999 %}
{% assign today = 'now' | date: '%Y-%m-%d' %}
{% assign filtered_items = '' | split: ',' %}

{% for item in all_items %}
  {% assign item_date = item.custom.my_date_field | date: '%Y-%m-%d' %}
  {% if item_date <= today %}
    {% assign filtered_items = filtered_items | push: item %}
  {% endif %}
{% endfor %}

{% for item in filtered_items %}
  {# filtered_items is already sorted — insertion order is preserved from all_items #}
{% endfor %}
```

The `sort` and `reverse` on `all_items` execute in the database. The push loop inserts items in that already-sorted order. `filtered_items` inherits the sort without needing a second `sort` call.

**Note:** Date fields used as sort keys must be in ISO 8601 format (`YYYY-MM-DD`) for sort and string comparison to work correctly.

**Reference implementations:** blog listing page (`blog_post` Custom Type), promotions modal (`promotions` Custom Type).

# Collection Filters: Conditional Default Values

The `collection_filters` custom field on a Collection supports Liquid conditionals. This allows a filter's default value (or even its entire rendering) to change based on another filter's current selection via `request.params`.

## Pipe-Delimited Syntax

Each filter is one line in the `collection_filters` field:
Label | param_name | attribute | default_value | key: value options

| Field | Description |
|---|---|
| Label | Display label shown to the shopper |
| param_name | URL query parameter name (e.g. `orientation`, `size`, `group`) |
| attribute | Product attribute to filter on (e.g. `product.custom.size`) |
| default_value | Pre-selected value on page load |
| options | Key-value pairs: `control_type: radio`, `control_type: dropdown`, `asset_images: true` |

## Conditional Default Pattern

Wrap a filter line in Liquid `{% if %}` / `{% elsif %}` blocks, reading the parent filter's current selection from `request.params['param_name'][0]`.

**Example — Size default changes based on Orientation selection:**

```liquid
Orientation | orientation | product.custom.orientation_img | portrait.jpg | control_type: radio | asset_images: true
{%- if request.params['orientation'][0] == 'landscape.jpg' or request.params['orientation'][0] == 'portrait.jpg' or request.params['orientation'][0] == blank %}
Size | size | product.custom.size | 16x20 | control_type: dropdown |
{%- elsif request.params['orientation'][0] == 'square.jpg' %}
Size | size | product.custom.size | 12x12 | control_type: dropdown |
{%- endif %}
```

**Key rules:**
- `request.params['param_name'][0]` — bracket notation with `[0]` to get the first (or only) selected value
- Include `or request.params['param_name'][0] == blank` in the default branch to cover initial page load (no selection yet)
- The param values compared must exactly match the values stored in the product's custom field
- Use `{%- -%}` whitespace-stripping tags to avoid blank lines in the rendered output
- The same filter line (e.g. Size) appears in each conditional branch — only the default_value differs
- This pattern also works for rendering entirely different filters based on selection (not just changing defaults)

RATIONALE: The collection_filters conditional default pattern is a reusable Shopper technique for dependent filter defaults. Not documented anywhere in the KB despite being in active use.
SOURCE: Current conversation — collection_filters conditional default for orientation/size
SOURCE TYPE: claude-chat
------------------------------------------------------------------------

# Fulfillment Transformation: Bulk Copy Across Templates

Fulfillment transformation settings (color profile conversion, bleed adjustments, output format, etc.) can be **bulk-copied from one template to multiple other templates** in a single operation.

Use this when deploying the same transformation rules to a set of related templates — for example, a family of photo book sizes where all variants share the same production pipeline configuration. Doing this manually per template is both slow and error-prone.

This preserves consistent pricing and design logic across a product range without having to re-enter settings for each template individually.

# data-method="post" and Rails UJS Event Bubbling

Links using `data-method="post"` (for example the copy/duplicate-project link
`/v1/books/{{ project.id }}/copy`) rely on a delegated Rails UJS handler attached
at the document level. The click must bubble up to `document` for the POST to
fire.

`onclick="event.stopPropagation()"` kills the event before it reaches that
handler, so the POST never fires and the browser falls back to a plain GET. The
action silently does the wrong thing (no error).

Fixes:
- Remove the `stopPropagation`. If a parent card click handler must be prevented,
  handle that in the parent (bail out when the click target is inside an `<a>`).
- Or drop `data-method` and submit a small inline `<form method="post">` with a
  submit button. A native form POST does not depend on UJS, so `stopPropagation`
  on the form is safe.

------------------------------------------------------------------------

# Conditional-Skip Comma Handling in JSON Loops

When building a JSON array in Liquid by looping a collection, do not use `forloop.last` to decide where commas go if any item can be skipped mid-loop (for example an `{% if %}` guard that omits some rows). When the final iterated item is the one skipped, the comma logic produces invalid JSON (a trailing or missing comma).

Use a `first_row` flag instead, and prepend a comma before every emitted row except the first:

```liquid
{%- assign first_row = true -%}
[
{%- for item in collection -%}
	{%- if item.custom.hide_from_search -%}{%- continue -%}{%- endif -%}
	{%- unless first_row %},{% endunless -%}
	{ "code": "{{ item.code }}" }
	{%- assign first_row = false -%}
{%- endfor -%}
]
```

This is robust regardless of which items are skipped. It applies to any Liquid JSON emitter — search index pages, fulfillment payloads, product feeds.

## Changelog

- 2026-03-12: Added `style onload` Re-injection Pattern section. Updated Dynamic UI Trigger Pattern.
- 2026-03-21: Added `sessionStorage` variant, `px-project-preview` shadow DOM styling, hero background overlay pattern.
- 2026-03-23: Added Skip Cart Redirect Pattern.
- 2026-03-26: Added Custom Type Collection: Sort + Filter Pattern.
- 2026-04-23: Added Fulfillment Transformation bulk copy pattern.
- 2026-05-13: Added Collection Filters: Conditional Default Values pattern.
- 2026-06-01: Added data-method/Rails UJS stopPropagation pattern. Source: claude-chat.
- 2026-06-30: Added conditional-skip comma handling pattern for JSON loops (first_row flag) — forloop.last breaks when items are skipped mid-loop. Source: claude-chat (search index work).
