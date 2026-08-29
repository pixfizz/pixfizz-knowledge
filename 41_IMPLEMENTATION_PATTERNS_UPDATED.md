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

### Gotchas: Bootstrap modals built inside a `style onload` IIFE

Two failure modes come up when a Bootstrap 4.6 modal is created or shown from a `style onload` IIFE on a Shopper/Shopify page:

- **jQuery is not available at script-load time.** The `style onload` attribute fires very early in page parsing, before the theme's jQuery has loaded. Any `jQuery(...)` / `$(...)` reference evaluated at load time fails silently. Only reference jQuery *inside* a user-interaction handler (click/change), which runs long after load — never at the top level of the IIFE.
- **`position: fixed` breaks under a transformed ancestor.** A modal inside a container that has a CSS `transform` (common in sidebars/option panels) renders inline/contained rather than as a full-screen overlay, because a transformed ancestor becomes the containing block for `position: fixed`. Fix: on first show, move the modal element to `document.body` (e.g. `document.body.appendChild(modalEl)`) so it escapes the transformed ancestor. Also wire close buttons with explicit delegated click handlers rather than relying on Bootstrap's `data-dismiss` auto-wiring, which may be absent in a trimmed Bootstrap build. Source: claude-chat (test-print charge modal).
- **`filter` on `<body>` breaks `position: fixed` and appending to body does not fix it.** `transform` is not the only property that creates a containing block for fixed positioning — `filter`, `backdrop-filter`, `perspective`, `contain`, and `will-change` do the same. A site whose `<body>` carries even a no-op `filter: blur(0px)` will contain every fixed element on the page, so the standard `document.body.appendChild(modalEl)` fix has nothing to escape to. Diagnose by checking computed style on `<body>` and every ancestor for these properties, not just `transform`. Where the offending rule cannot be removed (it is often part of a theme's page-transition effect), the fallback is a JavaScript viewport anchor that repositions the overlay against `window.scrollY` on scroll and resize. Source: claude-chat (custom designer modal).
- **`position: sticky` traps a modal below the backdrop.** This is a different mechanism from the containing-block problems above. `position: sticky` (and `position: fixed`) creates a **stacking context** unconditionally, even at `z-index: auto`. A modal rendered inside a sticky container therefore cannot rise above the body-level `.modal-backdrop` at z-index 1040, however high the modal's own z-index is. The symptom is a modal that is fully visible but completely unclickable. Confirm it with `document.elementFromPoint(x, y)` at the centre of the modal: a return of `DIV.modal-backdrop` proves the trap, and a manual `document.body.appendChild(modalEl)` in the console will unblock it. Fix it in CSS rather than JavaScript, by dropping sticky only while a modal is open: `body.modal-open .product-details.sticky { position: static; }`. When writing a diagnostic that scans ancestors for this class of bug, do **not** gate the check on `z-index !== auto`, because that filter skips every `sticky` and `fixed` element and is the reason the cause gets missed. Source: claude-chat (custom design tool on a child site).

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
# Liquid Parses Before CSS/HTML Comments

A Liquid-rendered snippet is parsed as Liquid **before** the host language's
comments mean anything. A `{% ... %}` tag inside a CSS `/* */` block, an HTML
`<!-- -->` comment, or a JS comment is still parsed and can still throw
(`Invalid snippet tag syntax`) or execute. Two consequences for any snippet
delivered as CSS or JS via `{% snippet %}`:
- Never write Liquid tag syntax inside a comment, even as documentation. A
  banner comment reading `{% snippet 'x' %}` errors the whole file.
- A snippet must never `{% snippet %}`-include itself.
A CSS snippet should contain exactly one Liquid tag — the `asset_url` for its
font/asset — and nothing else. Worth a pre-handover check.

------------------------------------------------------------------------

# Canvas and Form Patterns (customer-facing tools)

- **Render at devicePixelRatio.** A canvas sized only in CSS pixels is
  upscaled by the browser on retina displays and looks soft. Set
  `canvas.width = cssW * dpr` and `ctx.setTransform(dpr,0,0,dpr,0,0)`;
  resizing a canvas resets its transform, so reapply each frame.
- **`form.requestSubmit()` vs `form.submit()`.** To add to cart from a custom
  UI, prefer clicking the page's real Add-to-Cart control. As a fallback,
  `form.requestSubmit()` fires the form's own handlers and native validation;
  `form.submit()` bypasses both. Find the product form via
  `uploadHost.closest('form')` — `input.form` is null when the input sits in a
  shadow root.
- **Cart preview of a generated image.** The default cart option loop skips
  `option.template_option.type == 'image_upload'` (and `text`, `font`,
  `hide_from_cart`, `edit_from_cart`). A generated preview must therefore be
  attached as `file_upload`, and displaying it in the cart depends on whether
  an uploaded file's asset resolves to a URL in orderline Liquid — confirm
  before relying on it.

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

# Custom Tool Dependency Loading

A custom design tool delivered as a site asset (its own bundled build, plus any
vendored libraries such as a ZIP writer) needs `<script>` tags on the page. The
obvious host is `integrations/custom-body-scripts`, and it is the wrong one.

`integrations/custom-body-scripts` is a snippet child sites routinely override to
carry their own analytics and third-party tags. An include added to the parent
copy is therefore invisible on every child that has an override, and the failure
is silent — the product page renders, the tool never initialises, no error.

**Pattern:** have the tool's own product snippet load its dependencies.

-   The product snippet (e.g. `product/my-tool`) is the only file guaranteed to
    be present wherever the tool is actually used.
-   Guard the load so repeated renders do not double-inject: test for the global
    the library exposes, and only create a `<script>` element if it is absent.
-   Chain initialisation off the injected script's `onload`, not off document
    ready — the dependency resolves after the page has already loaded.

Deployment on a new site then reduces to two steps: upload the assets, add the
product snippet. No shared-include edit, and no parent-level change that has to
survive every child override.

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

## Measured platform behaviour

Evidence-backed on a live child site, not inference. Recorded because each one
either removes a constraint people assume exists, or is a constraint people
assume does not.

**`parse_json` is cheap at scale.** Parsing a ~20,700-character JSON string 25
times in a single page render produced no measurable TTFB change (10-run
medians: baseline ~0.79s, 1× parse ~0.72s, 25× parse ~0.70s — all inside network
noise). Large-JSON runtime composition is a safe pattern. This corrects any
assumption that `parse_json` must be rationed.

**Redirects capture dotted root paths.** A redirect rule such as
`[["^/llms\\.txt$", "<asset-url>"]]` fires as a 301 (x-runtime ~6ms) and serves
the CDN asset. Root-level "file" URLs are servable with no platform work.

**The asset store is extension-filtered.** The uploader **rejects `.txt` and
`.md`** (greyed in the picker) and **accepts `.json`** alongside images, js,
css, fonts and pdf. Two consequences: text-content root files must upload under
a `.json` name and will serve as `application/json`, which is functionally fine
for AI crawlers and cosmetically imperfect; and any generated creation bundle
must not put `.txt`/`.md` in `asset_files/` without verifying the importer path
separately.

**Image pipeline: the `format:` filter is WebP-capped; AVIF ships as an
uploaded asset.** *(Corrected 2026-08-29 — the previous text read "WebP only, no
AVIF support", which conflated a filter parameter with a format ban.)*

The `asset_url` filter's `format:` parameter accepts `jpeg`, `png` or `webp`
only. The platform will not **transcode** an asset to AVIF at request time, and
`format: 'webp'` is an accurate description of that ceiling. It does not follow
that AVIF cannot be used: pre-encoded AVIF files upload to the asset store and
serve through `<picture>` normally, and have shipped in production bundles since
16 August 2026.

Current rule: **AVIF + WebP, delivered through `<picture>`, WebP as the `<img>`
fallback, no JPEG copies for new imagery.** AVIF must be uploaded pre-encoded as
its own asset — never ask the pipeline for a format it does not emit.

**AVIF is not automatically smaller.** Against JPEG it wins by 40–65%. Against an
already-optimised small WebP it has measured **44–70% larger** (one 4,704-byte
WebP became 7,985 bytes at q62). Keep an AVIF only where it measures smaller than
its WebP, and record the measurement.

Where a rule is derived from a narrower fact, record the fact rather than the
generalisation. "The `format:` parameter accepts jpeg/png/webp" is durable; "no
AVIF support" was an inference that outlived its evidence and stayed quotable for
three weeks after three shipped builds contradicted it.

**Admin-only custom type instances are platform-hidden**, visible to logged-in
admins and no one else. This is platform behaviour, not template behaviour, and
is the mechanism for preview-and-approve workflows.

---

## `!= blank` is not portable, and treats nil as blank

Two portability bugs that were invisible to reading the Liquid and only showed
up on execution:

- `{% if product.custom.field != blank %}` — in Ruby Liquid, nil **is** blank, so
  this behaves as intended on Pixfizz but not in other Liquid engines. Anything
  ported or tested outside the platform will diverge.
- `{% if product.id != blank %}` reported a nil product as present.

**Portable pattern:** normalise, then compare against an empty string.

```liquid
{% assign v = product.custom.field | default: '' | strip %}
{% if v != '' %}
	...
{% endif %}
```

For plain existence, `{%- if product.id -%}` is correct and unambiguous.

Related, from `50_LIQUID_REFERENCE.md`: `nil == false` evaluates as `false`, so
use `!= true` rather than `== false` for boolean checks; and a boolean custom
field is a real boolean, never the string `'true'`.

---

## Browser-generated print files carry no physical size

`canvas.toBlob()` writes **no pHYs chunk**. A PNG produced from a canvas therefore
declares no physical size at all, and anything opening it falls back to 72 dpi.

The pixels can be perfectly correct and the file will still print at the wrong size.
Measured on a real gang sheet, 10 Aug 2026: 1800 x 9000 px for an ordered 12 x 60 in
sheet — exactly 150 dpi, geometrically correct — with zero pHYs chunks, so it opens as
**25 x 125 in**.

This is invisible on screen and invisible in any proof. It presents to the lab as
"the printables are not coming in the right size".

**Inject the chunk after `toBlob`.** It is 9 bytes of data — X and Y pixels-per-metre as
big-endian uint32, then a unit byte of 1 — placed immediately after IHDR, which is always
the first chunk and always 25 bytes, so the insertion point is a constant offset of 33
bytes. CRC32 over the type and data.

Three rules for the implementation:

- **Tag with the ACHIEVED dpi, not the requested one.** Browsers clamp large canvases, so
  the two can differ. A file that lies about its own resolution is worse than one that
  admits being soft.
- **Replace any existing chunk rather than appending**, or the winner is undefined.
- **Never throw.** Return the original blob unchanged on any anomaly. A missing DPI tag is
  bad; failing the export outright is worse.

Note pHYs stores pixels-per-*metre*, so 150 dpi round-trips as 150.0124 and a 72 in sheet
reads back as 71.99 in. That is inherent and harmless.

**PDF output does not need this** — page boxes already declare physical size.

---

# Canvas Export Requires `crossOrigin = 'anonymous'`

Site assets are served from a **different origin** than the storefront. Any image
drawn into a canvas that will later be exported must be loaded with
`img.crossOrigin = 'anonymous'` before `src` is set. Without it the canvas is
tainted and `canvas.toBlob()` throws `SecurityError`.

The failure is silent until export: the preview looks perfect on screen and no
file is ever written. This was the single cause of every missing preview in one
custom design tool across four consumers — the saved-cover card, Saved Projects,
the cart and the gallery — all of which looked like four separate bugs.

# iOS Safari Canvas Pixel Ceiling — 16,777,216

iOS Safari refuses to allocate a canvas larger than **16,777,216 pixels**
(width × height). This decides whether a product can be built as a browser-side
print file at all.

An 11×17 inch panel at 300 dpi is 5100 × 3300 = 16.83 M px. It misses the ceiling
by 0.3% — it passes every desktop QA run and fails on an iPhone. Run this
arithmetic as a go/no-go test before choosing the custom-tool archetype over the
standard XML archetype for any large-format product.

# Writing to the Cart from a Custom Tool

There is no client-side cart API. The write is a form submission.

Render one `{% form 'cart_add_product', product: p, page: 'cart' %}` per product
with a hidden input for every variant type on that product, **all disabled**. A
disabled input is not submitted, so enable only the axes whose parent-trigger
chain the current selection satisfies and leave the rest absent, letting the
platform apply its own defaults. **Posting empty strings instead clears required
variants.**

Multi-add is sequential, one navigation per product:

- set the form's `target` to the host page plus `?product_added_to_cart=t`
- keep the queue in `sessionStorage` so it survives the navigation
- on return, **confirm `cart.orderlines_total` actually grew before advancing** —
  do not treat the navigation itself as success

`skip_cart_redirect` on the product is not required if the engine writes `target`
itself.

Order-level values go to `cart.custom` via `cart_update`. **Unverified:** whether
a partial `cart_update` post preserves the other cart custom fields or clears
them. Establish that before relying on more than one.

# Collection Filter Params Are Arrays

The collection filter snippet tests `request.params[<url_name>].size`, so a scalar
query parameter fails the guard and the page renders **unfiltered, with no error**.

```
/site/shop/stickers?type=Roll      -> silently ignored
/site/shop/stickers?type[]=Roll    -> filters
```

This is a deep-linking trap for every collection carrying `collection_filters`.
Any campaign URL, email link or nav item that pre-filters a collection must use
the array form.

# A Snippet's Own `data-*-mount` Default Is a Label, Not Evidence

Mount-hook names written into a snippet's own default are written by whoever built
the snippet and are never checked against where the snippet is actually called
from. Trace the customer's route and grep for the hook.

Worked example — the photo-prints flow has two hooks on different routes:

| Page | Renders | Hook available |
|---|---|---|
| `/site/photo-prints`, collection-level prints pages | `product/product-details-prints` | `product/custom-prints-code` |
| `/site/prints?collection=…` (`pages/prints`) | `product/photo-prints` | `product/extra-prints-code` |

`custom-prints-code` sits on the prints **collection landing page**, the one whose
CTA is ORDER PRINTS. The actual flow — the only page carrying `.px-photo-prints`
and the `.px-btn-cart` button — is `pages/prints`, and its hook is
`extra-prints-code`. Anything that has to observe the prints component or its cart
button belongs on `extra-prints-code`.

# A Closing `</style>` Inside an Inlined CSS Snippet Breaks the Page

Some snippets carry their own CSS in a companion snippet inlined by the markup
snippet rather than going into `style/custom.css`:

```liquid
<style>{% snippet 'style/publication-upload' %}</style>
```

That is the right shape for a self-contained tool — the CSS travels with the
snippet, is not loaded on pages that do not use it, and a site can still retune it
from `style/custom.css` by redefining the custom properties.

**The trap:** the HTML parser ends a `<style>` element at the **first literal
`</style>` in its text**, and does not care that the string is inside a CSS
comment. So a CSS snippet whose header comment documents how it is included:

```css
/* Included from account/draft-order as
   <style>{% snippet 'style/draft-order' %}</style> */
```

ends the style element on that line. Everything after it is parsed as body
content, so the rest of the stylesheet renders on the page as visible text and
none of the CSS applies.

**The rule: a CSS snippet inlined inside a `<style>` element must contain no
literal `</style>` anywhere, comments included.** The same applies to `</script>`
inside anything inlined into a script element.

The failure is loud once you look at the page and invisible if you only read the
file — no import error, no Liquid error, and the snippet content is exactly what
was written. The signature is a page that appears to begin partway through a CSS
comment.

# Browser PDF Preflight — Rules That Only Surface on Real Artwork

Established against real InDesign and print-house files after synthetic fixtures
passed and real files failed on every page.

1. **pdf.js exposes only the CropBox.** `PDFPageProxy.view` is CropBox ∩ MediaBox;
   there is no TrimBox in the pdf.js public API (checked at 3.11.174). Read boxes
   with pdf-lib instead:
   `PDFDocument.load(bytes, { ignoreEncryption: true, updateMetadata: false })`
   then `getMediaBox` / `getTrimBox` / `getBleedBox`. Cost 0.07 s on a 3.5 MB file,
   1.2 s on 29.5 MB.
2. **Never infer trim from page size.** Real exports carry crop marks,
   registration marks and colour bars *outside* the bleed — measured 5.3–7.4 mm of
   mark area per edge on top of trim plus 5 mm bleed. Any "page size = trim +
   bleed" rule rejects real artwork, in code and in customer-facing copy.
3. **pdf.js needs `cMapUrl` and `cMapPacked: true`** (plus `standardFontDataUrl`),
   pinned to the same CDN version as the library, or CID fonts with predefined
   CMaps lose their glyphs in the preview.
4. **Visual thumbnail fingerprints do not identify pages in a text-heavy book.** A
   24×32 luminance fingerprint scored 0.99 on synthetic fixtures and picked the
   wrong page on a real 84-page book. Character trigrams of `getTextContent()`
   compared by Jaccard got 6/6 at rank 1 (0.79–0.94).
5. **`pdf-lib copyPages` is trustworthy** — 0 mean absolute pixel difference
   across 84 pages, with subset CID fonts, the PDF/X OutputIntent, tagged
   structure, TrimBox and BleedBox all surviving. Caveat: pdf-lib writes a PDF 1.7
   header, so a PDF/X-1a:2001 file keeps an XMP conformance claim a strict
   validator will reject.

pdf-lib self-hosts safely as a Pixfizz asset (380 KB, no worker, does not rewrite
its own script URL to find siblings), unlike pdf.js.

**Verifying a CDN file's SRI hash from a sandbox:** assert HTTP status and byte
count before trusting any digest. Hashing an empty body returns a value that looks
exactly like a hash, so a blocked request produces a confident wrong answer.


## Changelog

- 2026-03-12: Added `style onload` Re-injection Pattern section. Updated Dynamic UI Trigger Pattern.
- 2026-03-21: Added `sessionStorage` variant, `px-project-preview` shadow DOM styling, hero background overlay pattern.
- 2026-03-23: Added Skip Cart Redirect Pattern.
- 2026-03-26: Added Custom Type Collection: Sort + Filter Pattern.
- 2026-04-23: Added Fulfillment Transformation bulk copy pattern.
- 2026-05-13: Added Collection Filters: Conditional Default Values pattern.
- 2026-06-01: Added data-method/Rails UJS stopPropagation pattern. Source: claude-chat.
- 2026-06-30: Added conditional-skip comma handling pattern for JSON loops (first_row flag) — forloop.last breaks when items are skipped mid-loop. Source: claude-chat (search index work).
- 2026-07-04: Added Bootstrap-modal gotchas for `style onload` IIFEs (jQuery not available at load time; append modal to `document.body` to escape transformed-ancestor `position: fixed` containment). Source: claude-chat.
- 2026-07-25: Extended the fixed-positioning gotcha — `filter` (including a no-op `filter: blur(0px)`), `backdrop-filter`, `perspective`, `contain`, and `will-change` also create a containing block, and when the property sits on `<body>` the append-to-body fix does not work. Source: claude-chat.
- 2026-07-28: Added Custom Tool Dependency Loading — load tool dependencies from the tool's own product snippet, never from `integrations/custom-body-scripts`, which child sites override. Source: claude-chat.
- 2026-08-05: Added the `position: sticky` stacking-context modal trap, distinct from the containing-block gotchas, with the elementFromPoint confirmation, the `body.modal-open` CSS fix, and the diagnostic-script flaw of gating ancestor checks on `z-index !== auto`. Source: claude-chat.
- 2026-08-11: Added Measured platform behaviour — `parse_json` is cheap at scale (25 parses of a 20KB payload per render, no measurable TTFB change); redirects capture dotted root paths so `llms.txt` and similar are servable from an asset; the asset uploader is extension-filtered (.txt/.md rejected, .json accepted); WebP is the image-pipeline ceiling with no AVIF (AVIF under discussion, pending). Added the `!= blank` nil trap and the `| default: '' | strip` portable comparison. Source: claude-chat (Shopper v2 verification kit).
- 2026-08-29: **Corrected the image-pipeline rule** — the `format:` filter is WebP-capped, which is not a format ban; pre-encoded AVIF uploads and serves through `<picture>` and has shipped since 2026-08-16. Current rule is AVIF + WebP with WebP as the `<img>` fallback, keeping AVIF only where it measures smaller. Added: canvas export requires `crossOrigin = 'anonymous'` or `toBlob` throws `SecurityError` after a perfect-looking preview; the iOS Safari 16,777,216-pixel canvas ceiling as a go/no-go test for browser-built print files; writing to the cart from a custom tool (`cart_add_product` per product, disabled inputs as the mechanism, sequential queue in `sessionStorage`, assert `cart.orderlines_total` grew); collection filter params are arrays, so `?type=Roll` silently no-ops; a snippet's own `data-*-mount` default is a label rather than evidence, with the two photo-prints routes; a literal `</style>` inside an inlined CSS snippet ends the element early and dumps the stylesheet onto the page; and five browser PDF preflight rules (pdf.js exposes only the CropBox, never infer trim from page size, cMap config, text-trigram page matching, `pdf-lib copyPages` fidelity). Source: claude-chat.
