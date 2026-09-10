# 22 — Option & Variant Rendering

**Authority Scope:** Variant rendering behavior only.

_Last updated: 2026-09-09_

---

# Option / Variant Types and how Shopper renders them

This doc explains **what option/variant “types” exist in Pixfizz** and **how the Shopper template renders them** (based on the `product/px-options` and `product/px-option-cart` snippets you shared).

> Terminology note: Pixfizz UI calls these “Variants”, but the Liquid/snippets often treat them as generic `options` and render them via the same machinery.

---

## 1) The core `option.type` values (admin dropdown)

From the admin UI, these are the supported types:

- **Multiple Choice**
- **Text**
- **Number**
- **Color**
- **Font**
- **Image Upload**
- **File Upload**

In Shopper Liquid, those appear as `option.type` values (e.g. `'text'`, `'number'`, `'color'`, `'font'`, `'image_upload'`, `'file_upload'`). **Multiple Choice** is the default “else” path (typically rendered as radio buttons / tiles) unless an `option.custom.selector` overrides it.

---

## 2) Shopper entry points: where options are rendered

Shopper commonly renders options in two places:

- **On product pages** (static + design products):
	- Design product flow often renders:
		- Template options: `options: design.template_options` with `parameter_name: 'template_options'`
		- Product variants: `options: product.variants` with `parameter_name: 'variants'`
	- Static product flow renders product variants only.

- **In cart** (cart line item editing / display):
	- Cart uses a cart-specific snippet (commonly `product/px-option-cart`) to render option controls more compactly.

---

## 3) Global display gating and nesting

### 3.1 Kiosk-only options
Options can be conditionally displayed based on kiosk mode:

- If `option.custom.kiosk_mode_only` is truthy:
	- Shopper checks kiosk mode (`helpers/is-kiosk-mode`)
	- Only displays the option if `is_kiosk_mode == 'TRUE'`

### 3.2 Triggered / child options (conditional logic)
Options can have children (`option.children`), and child options can be shown based on a trigger:

- Parent option may include `option.trigger_value`
- Child option rendering can include `trigger="{{ child_option.trigger_value.code }}"`

Shopper renders children recursively, e.g.:

```liquid
{% snippet 'product/px-options',
	options: option.children,
	chosen_options: chosen_options,
	parameter_name: parameter_name,
	design: design %}
```

---

## 4) The important “selector” overrides (`option.custom.selector`)

Even when `option.type` is the same, Shopper can render **very different UI** via `option.custom.selector`.

These are the selectors explicitly handled in the snippet you provided:

### 4.1 `textarea` (Text option rendered as textarea)
- Applies when `option.type == 'text'` and `option.custom.selector == 'textarea'`
- Renders `<textarea ... rows="4">`

### 4.2 `color` (Multiple choice rendered as swatches)
- Applies when `option.custom.selector == 'color'`
- Renders radio buttons with SVG swatch tiles
- Uses `value.custom.hex` for the swatch color

> Separate from `option.type == 'color'`, which can render either a palette picker (`option.color_palette`) or an `<input type="color">`.

### 4.3 `checkbox` (Multiple choice rendered as checkbox)
- Applies when `option.custom.selector == 'checkbox'`
- Uses the **first value** for checked/unchecked logic
- Renders `<input type="checkbox" ...>`

### 4.4 `dropdown` (Multiple choice rendered as select)
- Applies when `option.custom.selector == 'dropdown'`
- Renders `<select>...</select>`
- Can display:
	- `value.custom.price_label`, else
	- `value.price` formatted via `currency`
	- Supports negative pricing display (minus sign formatting)

### 4.5 `slider` (Multiple choice rendered as range input)
- Applies when `option.custom.selector == 'slider'`
- Renders `<input type="range" ...>`
- Uses a JS map of `value.code -> value.name` to show a friendly label

### 4.6 `quick-quantity` (Multiple choice rendered as per-value quantity inputs)
- Applies when `option.custom.selector == 'quick-quantity'` **and** `support_quick_quantity` is true
- Renders a grid of numeric inputs (one per `option.values`)
- Uses `data-parameter-name` and `data-value-code` so JS can transform them into the expected param structure

This selector is crucial for “matrix style” purchasing (e.g., multiple sizes/finishes at once) without forcing the customer to add multiple separate line items manually.

**The chosen size never reaches `px-option-selector`, so the button price is wrong.**
Verified live, 20 August 2026, on an apparel client's product. The branch renders its
number inputs with **no `name` attribute** by design — `handleAddToCart` synthesises
`variants[<code>]` onto the FormData at submit time. `px-option-selector` only tracks
named form controls, so the size is never part of its selection set, and
`px-product-price` therefore prices the product as though no size were chosen: base
× quantity, with no value-price adjustment, whatever the shopper picks.

**The cart is unaffected.** The project is created with `variants[<code>]` set, so the
orderline prices correctly. Verified on that site: adult sizes billed at the higher rate
while the button showed the base. Say this plainly to a client before it reads as a
revenue bug — it is a display fault, not a revenue bug.

**Three hardening rules for any quick-quantity add-to-cart handler.** Verified by
debugging, 20 August 2026.

1. **`evt.currentTarget`, never `evt.target`.** `<px-product-price>` sits inside the
   button, so clicking the price makes `evt.target` that element and `evt.target.form`
   `undefined`. `new FormData(undefined)` throws *after* `preventDefault()` has already
   fired, so the click silently does nothing.
2. **Scope the input query to `button.form`,** not `document`. A duplicate button or a
   sticky bar otherwise collects the wrong set of inputs.
3. **Post the cart adds sequentially (`for … await`), never `Promise.all`.**
   Simultaneous `/cart/add_print_product` calls are a read-modify-write race on the cart
   session. Cost is N serial round trips instead of N parallel, and it cannot lose a
   line. Also guard on `book.id` being present and re-enable the button in a `catch`, so
   a failure is visible rather than a silent redirect.

**The cloned-button trap.** A sticky add-to-cart bar produced with `cloneNode` carries
the `add-to-cart-button` class and the live `px-product-price` element but **not the
click listener** — listeners are not cloned. Clicking it submits the form natively:
one orderline, quantity equal to the grid total, and the size falling back to the option
default.

**Diagnostic.** DevTools console only — `getEventListeners` is not available to page
scripts:

```js
[...document.querySelectorAll('button')]
	.filter(b => /add to cart/i.test(b.textContent))
	.map(b => ({ cls: b.className, inForm: !!b.form, listeners: Object.keys(getEventListeners(b)) }))
```

More than one entry means a second add-to-cart button exists, typically a sticky bar.
One entry with no `click` in `listeners` means binding never ran, or a fragment reload
replaced the button after `bindCartButtons`. One entry with `click` present means the
handler is fine and the parallel posts are racing.

**Translation gap.** The quick-quantity branch prints `{{ value.name | escape }}` while
every other selector uses `{{ value.name | t: ns: 'variants' | escape }}`. Size names do
not translate on a multilingual site. Verified by reading source.

### 4.7 Text option input constraints (`min_length`, `max_length`, `pattern`)

Applies when `option.type == 'text'` (and `option.custom.selector` is not `textarea`).

The platform exposes three validation properties on the `OptionType` object that map directly to HTML input attributes:

| Liquid property | HTML attribute | Notes |
|---|---|---|
| `option_type.min_length` | `minlength` | `nil` if not set |
| `option_type.max_length` | `maxlength` | `nil` if not set |
| `option_type.pattern` | `pattern` | `nil` if not set; value is a regex string |

These are configured in the admin under the option type's settings (Max Length field + Pattern field). The Pattern field tooltip confirms it is applied as an HTML `pattern` attribute, which triggers native browser validation.

**Pattern format notes:**
- The value is a standard HTML pattern regex (anchored implicitly to the full input value by the browser).
- Do not wrap in `/` delimiters — the value is used directly as the `pattern` attribute string.
- Example allowing upper and lowercase letters, digits, whitespace, and common punctuation: `[A-Za-z0-9\s&,.']*`
- The original parent template default (uppercase only) is: `[A-Z0-9\s&,.']*`

**Rendering note:** Whether `min_length`, `max_length`, and `pattern` are passed through as attributes on the rendered `<input>` in cart context (`product/px-option-cart`) has not been confirmed — verify against the live snippet if this matters for a specific implementation.

### 4.8 `toggle` (2-value Multiple choice rendered as an animated switch)

- Applies when `option.custom.selector == 'toggle'` **and** `option.values.size == 2`.
- Renders two visually-hidden radio inputs (first value = off side, second value = on side) plus a CSS-only switch control. No JavaScript is used, so it survives AJAX re-injection without a `style onload` re-init.
- Radios always submit a value, so the off side is never an empty submission (a lone checkbox would submit nothing in its off state).
- The active state is driven entirely by `:checked ~` sibling rules in CSS:
	- knob slides via `transform: translateX(...)`
	- track border and knob recolour to the site primary
	- the active flanking label is emphasised
- ON-state colour comes from `{% snippet 'style/color-primary' %}` referenced inside `style/custom.css` (custom.css is Liquid-processed, so the snippet resolves there). Note the snippet is `style/color-primary`, not `style/primary-color`.
- Click-to-flip is achieved with two overlapping empty `<label>` click targets that swap `pointer-events` by state — no script needed.
- Each radio carries an `aria-label` from its value name, so accessibility is preserved even when visible labels are hidden.

**Fallback / guard:**
- If values != 2, the branch is skipped and the option falls through to the default radio rendering. It cannot break the form.
- Mark one of the two values as **default** in admin so the initial state is predictable.

**Optional bare switch (no flanking labels):**
- Boolean custom field `option.custom.toggle_hide_labels`: when set, adds the `px-toggle-no-labels` class to the wrapper, which hides the value names (`.px-toggle-name`) and shows just the switch.
- Test as a real boolean (`{% if option.custom.toggle_hide_labels %}`), not a string comparison.
- Custom fields are site-specific and do not inherit parent → child, so set this on the site where the option lives.

**Pricing display note:** unlike `dropdown`, this selector does not show price deltas next to the values by default. If a toggle value carries a price, append it to the value name in the branch (or use `dropdown`).

**Where it lives:** `toggle` is a branch added to the Shopper snippet `product/px-options` (template layer). Add it on the parent (`shopper24`) to make it available everywhere, or override `product/px-options` on a single site to scope it. The CSS belongs in that site's `style/custom.css`.

---

## 5) Upload-specific behaviors

### 5.1 `image_upload` (`option.type == 'image_upload'`)
Shopper renders a `<px-image-upload>` web component with:
- sources: `local galleries qr` (varies; multi-upload group uses `local qr`)
- optional crop: `crop-aspect-ratio="{{ option.crop_aspect_ratio }}"`
- optional DPI: `minimum-dpi="{{ design.template.minimum_dpi }}"`
- optional accept: `accept="{{ option.custom.accept }}"`
- optional “no element substitutions”: `data-px-no-element-substitutions`
- optional “no pricing”: `data-px-no-pricing`
- optional image adjustments driven by admin checklist:
	- `enable-image-filters-image-upload`
	- `enable-image-color-image-upload`

### 5.2 `file_upload` (`option.type == 'file_upload'`)
Shopper renders a `<px-file-upload>` component.
- Default accept is `image/*,.pdf` unless overridden.
- Can display existing uploaded file name / URL.
- **Programmatic injection:** `px-file-upload` exposes a real
  `input[type=file]`, so a script can inject a File via a `DataTransfer`
  object (`input.files = dt.files; input.dispatchEvent(new Event('change'))`).
  `px-image-upload` does **not** — it is built for interactive gallery/QR
  selection, exposes only a hidden `input[type=hidden]` and an "Upload"
  button, and creates its file input lazily inside the dialog. To attach a
  script-generated file to an option, use `file_upload`, not `image_upload`.
- **Read-only vs hidden:** for a script-injected upload the option must be
  `hidden: true` (invisible to the customer) with `read_only: false`. A
  `read_only: true` upload renders a hidden value input plus a read-back chip
  with no file input to inject into.
### 5.2b Targeting a specific upload by option code
Every option is wrapped in `<px-option code="...">`. When a product has more
than one upload option, scope any DOM query to the wrapper —
`document.querySelector('px-option[code="X"] px-file-upload')` — rather than
grabbing the first upload on the page. When a code is given and no match is
found, return null rather than falling back to the first upload, or an
injected file lands in the wrong option.

### 5.2c Input names differ between the product page and project-edit
The same variant or option is submitted under a different input name depending
on which page the shopper is on:

- **Product page:** `variants[<code>]`
- **Project-edit:** `book[options][<code>]`

Any script that reads or writes an option value must therefore **suffix-match**
the input name (`name.endsWith('[' + code + ']')`) rather than matching the full
string. An exact match on `variants[<code>]` works on the product page and
silently finds nothing on project-edit. The user-visible symptom is a saved
project opening with its settings reset when the customer edits it.

Hidden inputs count. A read-back routine that only inspects checked radios and
selects will miss values that project-edit renders as `input[type=hidden]`.

### 5.3 Multi-upload groups (`option.custom.multi_upload_group`)
This is a key advanced behavior.

If `option.custom.multi_upload_group` is set, Shopper:
- Opens a wrapper `<px-multi-image-upload ...>`
- Groups multiple upload options under a single “Upload Photos” experience
- Starts the wrapper when the group changes vs `previous_option.custom.multi_upload_group`
- Closes the wrapper when the group changes vs `next_option.custom.multi_upload_group`

This allows “upload many images” flows while still storing each uploaded image against a distinct option code.

---

## 6) Pricing and substitutions flags (important for rendering + behavior)

Shopper passes these flags into many inputs/components:

- `option.has_element_substitutions`
	- If false: `data-px-no-element-substitutions`
- `option.has_pricing`
	- If false: `data-px-no-pricing`

Meaning: **an option can exist purely for substitutions**, purely for pricing, both, or neither, and Shopper can explicitly tell Px components to ignore certain behaviors.

---

## 7) Cart rendering differences (`product/px-option-cart`)

In cart context, the rendering is simplified:

- `option.type == 'text'` becomes `<input type="text">`
- Otherwise, defaults to `<select>` for values
- Value price display in the `<option>` label:
	- e.g. `{{ value.price | currency }}` if non-zero
- Still supports nested options (children) via recursion:
	- `option.children` + `trigger` relationship

> Note: the `toggle` selector is implemented in `product/px-options` (product-page context). It is not wired into `product/px-option-cart`, where a 2-value option falls back to the default `<select>`. Add it there separately if a toggle is needed in cart.

---

## 8) Practical “recognize and document” list (what matters when reading a site)

When you see a variant/option behaving “special” in Shopper, check these first:

- `option.type` (text/number/color/font/image_upload/file_upload vs multiple choice default)
- `option.custom.selector` (textarea, color swatch, checkbox, dropdown, slider, quick-quantity, toggle)
- `option.custom.toggle_hide_labels` (bare switch with no flanking labels, toggle selector only)
- `option.custom.multi_upload_group` (multi-image wrapper)
- `option.custom.kiosk_mode_only` (kiosk-only display)
- `option.custom.hidden` (completely hidden)
- `option.trigger_value` + `option.children` (conditional display / nesting)
- `option.custom.accept` (upload constraints)
- Admin checklist toggles affecting image upload adjustments
- `option.custom.custom_script` (inline scripts—dangerous but real)

That’s the set that needs to be “muscle memory” when debugging option rendering in Shopper.

---

## A Single-Value Variant Type Renders as a Selected Button

Found 2026-08-24 on a print product page.

When a Shopper variant type has exactly one value, that value is auto-selected, so
it picks up the theme's **selected** state:

```css
.variant-selector label.field-label input:checked + .label-text { background:#3b3b3b; color:#fff; }
```

The result is a large dark pill that looks like an interactive choice, cannot be
changed, and is not in the site palette. On a title with four fixed specifications
(cover paper, content paper, cover finish, binding) that is four stacked blocks of
roughly 190 px each, which pushed quantity and Add to order below the fold on a
1512 px viewport.

**This hits any print product whose specification is fixed per SKU, which is most
web-to-print.**

The markup:

```
div.select-box
  div.px-title
    label                      "Select Cover Paper"
  div.row
    div.col-4.col-md-3         one per value
      label.field-label
        input                  radio, visually zero-sized
        div.label-text         "230 gsm"
```

The value columns are the light-DOM children of `px-option-selector`, whose shadow
root is only a `<slot>`, so `style/custom.css` reaches them normally. No `::part()`
needed.

**The fix** — detect the single-value case with `:only-child` and render the group
as a spec row instead of a button:

```css
.select-box:has(.row > div:only-child) {
	display: flex; align-items: baseline; justify-content: space-between;
	padding: 11px 0; border-bottom: 1px solid var(--brand-line);
}
.select-box:has(.row > div:only-child) .label-text,
.select-box:has(.row > div:only-child) .field-label input:checked + .label-text {
	background: transparent !important; color: var(--brand-ink) !important;
	border: 0 !important; border-radius: 0 !important; padding: 0 !important;
	font-size: 14px !important; font-weight: 600 !important; text-align: right;
}
```

Measured: each group drops from about 190 px to 54 px. Two properties worth
keeping: **it reverts itself** — add a second value and `:only-child` stops
matching, so the buttons come back with no code change — and **it fails in the
right direction**, since a browser without `:has()` keeps the old appearance rather
than a broken one.

**Not fixable in CSS.** The variant type names read "Select Cover Paper". For a
fixed specification the verb is wrong; rename the variant types in admin. The Add
to cart label is literally uppercase in the theme markup, not `text-transform`, so
it cannot be sentence-cased from `style/custom.css`.

## Unset Booleans Export as the String `'false'`

An unset boolean in a YAML export comes back as the **quoted string** `'false'`,
which Liquid reads as truthy. This affects `hidden`, `read_only` and
`hide_from_cart` — all of which live **inside** `custom`, not at the top level.

**After importing any option archive, re-check both flags in admin.** An option
intended as `read_only: false` can import as effectively read-only, and
`read_only: true` renders a hidden value input plus a read-back chip with no file
input, so a script-injected upload lands nowhere.

The `'false'` trap bites **string** fields only. A genuine boolean custom field is
safe with `{% if collection.custom.x %}`.

## A Required File-Upload Option Behind a Trigger Silently Kills Add to Cart

**Class: platform bug, not site misconfiguration.** Verified in-browser and independently
reproduced in admin, 8 September 2026, on a copy-shop client's product. Affects any
product with a `required` file or image upload option behind `trigger_value`.

### Symptom

Add to Cart does nothing. **No network request is issued.** Other static products on the
same site add to cart normally. The console shows unrelated noise — on the site where
this was found, a `gtag is not defined` ReferenceError — which sends you chasing
analytics. Uploading a valid file for the chosen size does not help.

### Cause

Three `Upload File` options, one per paper size, each `required: true`, each gated by
`trigger_value_code`. Only one is displayed at a time.

The upload component sets a **custom validity message** and **does not clear it when the
option is hidden by its trigger**. The two branches the shopper did not choose stay
permanently invalid. The browser refuses to submit, and because those controls sit inside
a `display: none` `PX-OPTION`, Chrome cannot focus them to show a validation bubble.
Silent failure.

### Evidence

Clean page load, single pass, resolving each invalid control to its enclosing
`PX-OPTION`:

| Stage | Form valid | Invalid options |
|---|---|---|
| Fresh load | false | `file-11x17` (hidden), `file-8.5x14` (hidden), `file-8.5x11` (visible) |
| After satisfying only the **visible** option — what uploading a file does | **false** | `file-11x17` (hidden), `file-8.5x14` (hidden) |

**Independently confirmed from the other direction:** uploading a file to *all three*
sizes allowed Add to Cart to succeed. That is also the shape of the bug — a shopper
would have to upload their document once per size they are not buying.

### `disable_required_form` does NOT fix this

**Correction to the obvious assumption.** The `disable_required_form` boolean product
custom field was ticked and Add to Cart was still dead.

The field is documented as bypassing HTML5 required-field validation, and it does what it
says — it deals with the `required` **attribute**. This fault is not a `required`
attribute. It is a `setCustomValidity()` string set by the upload component, which
survives the field being ticked. A `required`-attribute switch cannot clear a custom
validity string.

So `disable_required_form` is not a workaround here, and any text implying it is a
general "unblock Add to Cart" lever needs qualifying: it covers `required`, not custom
validity.

### Workarounds, in order

1. **`required: false` on every one of those upload options.** Most likely to work,
   because the component's custom validity is presumably raised off the option's required
   flag. It must be **all** of them — leaving any one required re-creates the fault
   whenever that branch is not selected. Cost: no upload enforcement on any branch, so a
   shopper can reach the cart with no file. Test before relying on it.
2. **The platform fix, and the correct one:** the option renderer must clear custom
   validity, or disable the control, whenever a triggered option is not displayed.
   Benefits every lab.
3. **Collapse the per-branch uploads into one always-visible upload.** Removes the
   precondition and keeps enforcement. **Check fulfillment first** — the file
   currently lands on a size-specific option code and production may key off it. Option
   codes resolve outside the tar and nothing can detect the break.

### Two debugging techniques worth keeping

**When Add to Cart does nothing and no network request is issued, it is form validation.**
Not JavaScript, not pricing. Go straight to `form.checkValidity()` and enumerate
`[...form.elements].filter(el => el.willValidate && !el.checkValidity())`, resolving each
to its enclosing `PX-OPTION`. Console errors at that moment are usually a red herring.

**`setCustomValidity()` mutations persist for the life of the page.** A first pass that
clears the hidden options' validity poisons the next test, which then shows only one
invalid control and appears to exonerate them. Reload before every re-test and run the
whole before/after comparison in a single call.

### Related cart-side defect on the same orderline

The cart line rendered the same child option label once per hidden per-branch child. Those
children carry `custom: {hidden: true}`, which hides them on the product page but **not**
in the cart. They need `hide_from_cart` set. Customer-facing quality issue, unrelated to
the validation bug, fix it in the same variant pass.

---

## `value.price` Exports Blank, Not Zero

**Verified by reading source — a variant export, 20 August 2026.**

A variant export carries `price: '2'` on priced values and **`price: ''`** on unpriced
ones — an empty string, not `0`.

The idiom used throughout `product/px-options` for the dropdown, checkbox, swatch and
segmented branches is:

```liquid
{% if value.price != 0 %}+{{ value.price | currency }}{% endif %}
```

A blank string is not equal to `0`, so on any option whose unpriced values export as `''`
this test passes and renders `+$0.00` against every free value. Normalise first:

```liquid
{% assign qq_value_price = value.price | plus: 0 %}
{% if qq_value_price > 0 %}+{{ qq_value_price | currency }}{% endif %}
```

`| plus: 0` coerces both `''` and nil to `0`.

**Not verified — pending confirmation:** whether the Liquid `value.price` accessor
returns the raw `''` or coerces it to `0` before the template sees it. If it returns
`''`, every existing `!= 0` branch in `product/px-options` renders `+$0.00` on free
values across every site, which would be a visible and long-standing fault nobody has
reported — so coercion somewhere is likely. Confirm on a dropdown-selector option
with a mix of priced and free values before rewriting the other branches.

---

## Grouped Value Bands Must Be Order-Independent and Opt-In

**Verified by test, 8 September 2026**, while porting a grouped size grid onto the Shopper
parent.

A grouped value grid — bands of values under a group heading, driven by a value-level
custom field — has two rules, and both were learned by breaking them:

- **Collect distinct group names on first appearance.** Do not open a new band whenever
  the group name differs from the previous value. That version is order-dependent: values
  ordered Youth, Adult, Youth produce three bands, two of them labelled Youth. Sizes get
  reordered in admin routinely and nothing warns you.
- **Stay on the flat grid until at least one value carries the group field.** A version
  that fires with nothing grouped opens one band on every existing quick-quantity product,
  which on a parent snippet is a visual change to every apparel site at once.

**Record of the worse variant, caught by test rather than by reading.** An earlier draft
tested `value.custom.group == blank` and **silently dropped ungrouped values while still
rendering bands** — a size visible on the product but impossible to order, with no
error anywhere.

Values left ungrouped while others are grouped belong in a **final untitled band** —
visibly odd rather than invisibly missing.

---

## `collection_filters` Has Two Syntaxes

**Verified live, 8 September 2026.** Neither syntax was previously recorded.

| Page | Consumed by | Fields |
|---|---|---|
| Standard shop page | `collection/collection-filters` | three |
| `pdp_layout` | `product/details-filter-dual-mode` | five |

The five-field form is:

```
label | url_name | filter_attribute | default_value | snippet_args
```

`snippet_args` is `key: value` pairs joined by pipes, consumed by
`product/filter-controls`. With `asset_images: true` the field value must be an **asset
filename**, not a label.

---

## Template-Options Import: Blank Ids Create New Records

**Verified live, 9 September 2026 — two templates on one site, first attempt, no
error.**

The standalone `__template_options.yml` archive — the export produced from a
template's options alone, not the whole `__print_product.yml` — accepts a blank
`id:` and creates new records. The two traps in reusing an options export taken from
another site, and the still-untested duplicate question, are in
`51_CUSTOM_FIELDS_REFERENCE.md`.

---

## Variant Type Exports

**Verified by reading source — a Finish variant type export, 2 September 2026.**

A variant type is a **shared** object attached to many Product Attributes, and a
`variant_value` carries a single `price`. One price list therefore covers every product
using that variant type, which does not fit a per-size price ladder. Plan the shape before
building the export, not after.

The export format is its own shape, not the template shape:

```
./assets/  ./fonts/  ./glb_files/  ./images/  ./pdfs/
./__variant_types.yml
```

with `__asset_map`, `__image_map`, `__pdf_map` and `__font_map` at the end. The file holds
one `variant_types` list; each entry carries `variant_values`. Type-level keys seen:
`id`, `name`, `code`, `value_type`, `required`, `order`. Value-level keys seen: `id`,
`name`, `code`, `default`, `order`, `price`, `image`. Unpriced values export `price: ''`
— see the blank-price section above.

**Stated, not independently verified:** that a variant-type import updates in place by ID
rather than creating duplicates, which would make an export → edit → re-import
round trip safe for bulk price editing. This is **in tension** with the auto-suffix
duplicate behavior recorded for Templates, Product Attributes and Designs in
`16_PRODUCT_HIERARCHY.md`, and with the create-only per-product archive in
`51_CUSTOM_FIELDS_REFERENCE.md`. A bench test on a test site settles it. Neither behavior
should be written as a general import rule until it does.

## Changelog
- 2026-06-19: Added section 4.8 `toggle` selector (2-value animated CSS-only switch on `product/px-options`), including the `toggle_hide_labels` bare-switch option, guard/fallback behavior, and primary-colour sourcing. Added cart-context note (7) that toggle is product-page only. Added `toggle` and `toggle_hide_labels` to the recognize-and-document list (8).
- 2026-07-28: Added 5.2c — option input names differ between the product page (`variants[code]`) and project-edit (`book[options][code]`); scripts must suffix-match and must handle hidden inputs. Source: claude-chat.
- 2026-08-29: Added the single-value variant type gotcha — one value is auto-selected and inherits the theme's selected-button styling, producing a large fixed pill that costs roughly 190 px per group; includes the markup tree, the `:only-child` CSS fix that reverts itself when a second value is added, and the two things that need an admin change rather than CSS. Added: unset booleans export as the quoted string `'false'` and read truthy in Liquid, affecting `hidden`, `read_only` and `hide_from_cart` inside `custom` — re-check both flags in admin after importing any option archive. Source: claude-chat.
- 2026-09-09: Added the platform bug where a required file-upload option behind a trigger silently kills Add to Cart — symptom, cause, evidence table, the independent confirmation, the correction that `disable_required_form` does not fix it, the three workarounds and the two debugging techniques. Extended §4.6 `quick-quantity` with the no-`name` consequence for `px-option-selector` and `px-product-price` (display fault, cart correct), the three add-to-cart handler hardening rules, the cloned-button trap, the `getEventListeners` diagnostic and the missing `t: ns: 'variants'` translation filter. Added that `value.price` exports blank rather than zero, so `!= 0` renders `+$0.00` on free values, and the `| plus: 0` normalisation. Added the two rules for grouped value bands (order-independent collection, opt-in on the group field) and the recorded test failure where `== blank` dropped ungrouped values. Added the two `collection_filters` syntaxes. Added that blank ids in a standalone template-options import create new records. Added the variant type export shape and the shared-object price constraint, with the unverified update-in-place claim flagged. Source: claude-chat, fireflies-call.
