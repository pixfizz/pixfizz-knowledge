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
**Trap: a missing boolean definition hides the option on the website.** On a site with no boolean custom field definitions for the option flags (`kiosk_mode_only`, `edit_from_cart`, `hide_label`, `hide_pricing`, `hide_value_labels`), an imported option stores them as the text `"false"`, which Liquid reads as truthy, so `product/px-options` drops the option with no error. Seen on a custom-script mount option that was stored (visible in `/v1/themes/<id>`) and never rendered. Check: `JSON.stringify(option.custom.kiosk_mode_only)` must return `false`, not `"false"` in quotes. Fix: create the boolean definitions on the site's option object; the values then read as booleans. Platform-level (definitions per site) meeting template-level (Shopper 24 `product/px-options`). *Verified by query and by reading source (Shopper 24 parent), 2026-09-28.* See § Unset Booleans Export as the String `'false'`.

**The same trap arrives with a template export from another site.** An export carries the source site's option custom keys (seen: `kiosk_mode_only`, `hide_label`, `hide_pricing`, `edit_from_cart`, `hide_value_labels`, `fill_placeholder_text`, all `false`). A target site with no definitions for those keys stores each as the string `"false"`, so the option, including a `custom_script` mount, renders in kiosk mode only and the tool never mounts on the website, with no error anywhere. Two ways out:

- **Clear the keys:** post the option's custom form with each key blank (`template_option_type[custom][<key>]=`), plus `_method=patch` and the page token. A blank value deletes the key; in the verified case the other keys (`custom_script`) were kept. Unconfirmed: `18_ADMIN_NAVIGATION.md` § Bulk Update Tools says fields left out of this form are blanked, so read the option back after posting.
- **Prevent it:** before importing a template or option export, strip every option custom key the target site has no definition for. The site's definitions are listed at `/site/<site>/admin/custom_field_definitions?type=TemplateOptionType`.

*Verified by query on a shopper24 child, 2026-10-05.*

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
- **`admin/checklist/hide-color-label` did not reach this branch** until a parent fix on 2026-10-08. The checklist capture ran only inside the `option.type == 'color'` branch; this branch tested the same variable without setting it, so value names always showed. The parent fix repeats the capture right after `{% elsif option.custom.selector == 'color' %}`. A child with its own override of `product/px-option` keeps the defect: list the child's snippets before calling a parent fix done. `option.custom.hide_value_labels` does not reach this branch. Hover names already work (`title` plus `data-toggle="tooltip"` on the swatch). Template-level (Shopper 24). *Verified by reading source and by query, 2026-10-08.*

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

### 4.9 `hide_value_labels` (hide the text under image thumbnails)

Template-level (Shopper 24 `product/px-options`). *Verified by reading source, 2026-09-10.*

`product/px-options` renders a value label under every thumbnail on an image-based multiple choice option. The per-option boolean custom field `hide_value_labels` (Variant Options object) hides those labels. Ticked hides them; unticked (default) shows them. No effect on options whose values have no image: there the label is the control.

- `option.custom.hide_label` hides the option title, not the value labels. `option.custom.hide_pricing` hides the price overlay only.
- `admin/checklist/hide-color-label` (site-wide, `TRUE`) covers only the two color branches (`option.color_palette` swatches and `option.custom.selector == 'color'`; the second only since the 2026-10-08 parent fix, and not on a child that overrides `product/px-option`, § 4.2). An option whose code contains "color" but whose values are asset images renders through the generic image branch, inherits the color font sizing, and is not reached by the color checklist. Use `hide_value_labels` there.
- Read it as `option.custom.hide_value_labels and option.custom.hide_value_labels != 'false'` (§ Unset Booleans Export as the String `'false'`).
- The custom field definition must exist on the site's option object before it can be ticked. An Override Snippet of `product/px-options` on a child site pins the old snippet and never receives the parent change.

Snippet Description (for the custom field): "Hides the text label under each thumbnail on an image-based multiple choice option. Ticked hides the labels, unticked (default) shows them. Has no effect on options whose values have no image."

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
**Value and crop (platform-level, the `px-image-upload` web component):**
- The upload lives in the component's `value` attribute as `db:<image id>`, optionally followed by crop data `@{l:..,t:..,r:..,z:..}`; no crop is plain `db:<id>`. `px-multi-image-upload` carries it the same way. The option code differs per template (`photo`, `upload-image`), so find the control by tag, scoped to its `px-option` where there are several. `px-option-selector` fires a bubbling `change` after an upload. *Verified by query and by reading source, 2026-09-26 and 2026-09-29.*
- Setting `value` to `db:<existing image id>` from script updates the hidden input and fires that `change`, which tests an upload flow without the dialog. *Verified by query, 2026-09-26.*
- **`crop-aspect-ratio` is an observed attribute.** Setting it on the page (`8in/10in`, `10in/8in`) changes the Adjust dialog's crop box at once, with no reload. One upload option can therefore serve several placeholder shapes (orientation, presentation) with the ratio set from the page; one upload option per shape loses the photo when the customer switches. A crop made for the old ratio stays in the value: reset it to `db:<id>` when the shape changes. *Verified live, 2026-09-29.*
- **With no Crop Aspect Ratio on the option, Adjust shows no crop box at all**: the customer can only filter, rotate and reset, and the photo fills the placeholder centered. *Verified live, 2026-09-29.*
- The Adjust dialog copies every `crop-*` attribute of the upload onto `px-image-adjust-tool` (prefix stripped) when it opens. `crop-rotation-mode` is also observed; its values are unknown.
- Not verified: a crop moved by hand after a ratio change, end to end into the production file.
- **An image upload option crops the customer's image to fill, whatever the element says.** With no crop props on the `db:<id>` value, the server crops to fill even when the page XML element has `crop="false"`. To keep a logo whole, add a hidden multiple-choice option whose single default value carries an `image_crop_flag` element substitution with cropping off; image upload options have no substitution panel of their own. Whether the option's Crop Aspect Ratio field also drives this is not tested. Platform-level. *Verified on baseline.pixfizz.com, 2026-10-05.* Details: `17_DESIGN_TOOL.md` § Image crop flag.

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

### 6.1 `data-px-no-element-substitutions` keeps an input out of preview URLs

When the attribute is on an option input, the preview widget does not use that input to build `/preview.svg` URLs. Shopper adds it when `option.has_element_substitutions` is false. To force it per option (for example long text the preview pages do not render), check a boolean custom field on the template option first, in `product/px-option`:

```liquid
{% if option.custom.skip_for_previews %}
	data-px-no-element-substitutions
{% else %}
	{% unless option.has_element_substitutions %}data-px-no-element-substitutions{% endunless %}
{% endif %}
```

- Create the boolean `skip_for_previews` definition on Template Options on the site first. The field name in admin must match the Liquid exactly: a near-miss name stores a value nothing reads.
- Prefer this to intercepting the preview request to strip parameters.

Template-level (Shopper `product/px-option`) over platform preview widget behavior. *Stated by the core developer, 2025-05; not re-tested by query.*

### 6.2 Selected options are written into the page URL

`<px-option-selector onchange="storeOptionSelectionIntoURL(this)">` in `product/design-now` writes every selected option into the page URL on change. This is what carries values across a product switch on the same product form.

- Long text values (for example generated story paragraphs) make the URL too long: the product page crashes and the project can be saved corrupted. Short values are harmless (`27_LIVE_FINISH_AND_3D_PREVIEWS.md` observes it with `lf_mount`).
- Do not remove the handler if shoppers switch products on the form. Exclude the long-value options from it (an ignore list in `product/design-now`), keep those values in a client store or in project custom fields (`50_LIQUID_REFERENCE.md` § Projects), and refill the inputs on `pageshow` and `px.fragmentsReloaded` (`17_DESIGN_TOOL.md` § Restoring Input Values After a Product Switch).
- A text option value is capped at 1,024 characters regardless (`51_CUSTOM_FIELDS_REFERENCE.md` Key Notes).

Template-level (Shopper `product/design-now`). *Stated by the core developer, 2025-05; not re-tested by query.*

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

## Template Option Substitutions: Target Elements, Types and Limits

Platform-level (Pixfizz CMS). Applies to template options and design options alike.

- **`target_element_name` is a 255-character column.** A longer comma list fails the **whole template import** with `Mysql2::Error: Data too long for column 'target_element_name'`. Keep element names short when one upload fills many elements (`t1,t2,...`). *Verified by query, 2026-10-05.* A failed import leaves a partial template behind; see `16_PRODUCT_HIERARCHY.md` § Import Behavior.
- **An image upload's Target Element Name accepts several names separated by commas** (admin help text).
- **Substitution keys can target tags:** `name@[tag1,tag2]`, with the name optional. Elements carry `tags="a,b"` in page XML. Works in the storefront editor (verified 2026-10-08, `17_DESIGN_TOOL.md` § Substitutions bind by element name); the production render is not verified.
- **A substitution that names a missing element does nothing, silently.** If an option value's substitution targets an element name found on no page of the design, the option renders, can be chosen and is saved on the order, and nothing changes: not the product preview, not the editor, not the print file (renders for two values were byte-identical). Before any preview work, check every design option's substitution targets against the element names on that design's pages. *Verified by query, 2026-10-07.*
- **A variant and a template option on the same product must not share a code.** `variants[x]` and `template_options[x]` both end in `[x]`, so a script matching input names by suffix finds either. Template-level (Shopper 24 form names).
- **Copy design options between sites with their values:** `GET /site/<a>/admin/templates/<t>/options/export_all?print_theme_id=<design>`, then POST the tar as `exported_file` to `/site/<b>/admin/templates/<t2>/options/import?print_theme_id=<design2>`. Values, value custom fields and element substitutions travel; the custom field definitions must already exist on the target site. *Verified by query on 60 designs, 2026-10-08.*
- **Element substitution types offered by the admin form:** `image`, `image_mask`, `image_color`, `image_effect`, `image_crop_flag`, `image_border_width`, `image_border_color`, `image_border_radius`, `text`, `text_color`, `text_font`, `text_font_size`, `qrcode_content`, `shape_color`, `shape_border_width`, `shape_border_color`, `shape_border_radius`, `inline_page_mask`, `inline_page_border_radius`, `element_opacity`, `element_blend_mode`, `background_color`, `layout`, `page_mask`.
- **A `color` option applies the customer's color to every substitution it carries**; the substitution's own content is not used (read from the editor bundle). Color substitutions on a color option import with their color intact, not reset to black. *Verified by query, 2026-10-05.*

---

## Template-Options Import: Blank Ids Create New Records

**Verified live, 9 September 2026 — two templates on one site, first attempt, no
error.**

The standalone `__template_options.yml` archive — the export produced from a
template's options alone, not the whole `__print_product.yml` — accepts a blank
`id:` and creates new records. The two traps in reusing an options export taken from
another site, and the still-untested duplicate question, are in
`51_CUSTOM_FIELDS_REFERENCE.md`.
**Option trees with layout substitutions import in one go.** A `__template_options.yml` archive imported at the template's options import can hold a parent option with `children` (each with `trigger_value_code`) and values carrying `element_substitutions` of type layout; the whole tree is created. The option value edit page in admin has no element substitution UI, so import is the route for adding layout substitutions to template option values. The import answered with a 500 error but created everything: check the result with the option's own export, not the response. *Verified by use and by export, 2026-09-29.*

---

## Variant Type Exports

**Verified by reading source — a Finish variant type export, 2 September 2026.**

**Corrected 2026-09-20.** A variant type belongs to **one Product Attribute** and is **not shared** across products. Two products on the same site can carry the same variant code at different prices. It is created at **Products Attributes → Product: `<name>` → New Variant** (`/admin/products/<product_id>/variant_types/new`). Fields: Name, Code, Description, Image, Type, Page Names, Required, Published. Types: Multiple Choice, Text, Number, Color, Font, Image Upload, File Upload. Hidden, Read only and Hide from cart are set in admin. A product's variant types export (`/admin/products/<id>/variant_types/export_all`, returns `__variant_types.yml`) **does** carry them, in `custom` (`hidden`, `read_only`, `hide_from_cart`). *Verified by query, 2026-09-29.* (Corrected 2026-10-06: previously said they do not appear in the export.) A variant type set imports onto one product at `/admin/products/<id>/variant_types/import`; blank ids create new records.

The earlier text here called a variant type a shared object with one price list for every product using it. That was read from the shape of an export file and was wrong: an export's shape is not proof of how the platform uses it. Confirm against the admin screen. *Verified by reading source (admin) and stated by Alex, 2026-09-20.*

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

**Settled 2026-09-20: a variant-type import never updates in place.** A code collision appends `-1` to every code and value code in the imported set, the same create-only behavior as every other import (`01_CODE_GOVERNANCE_UPDATED.md` § Never Re-Import to Update). An export → edit → re-import round trip is not a bulk price-editing route. *Stated by Alex.*
**Variant codes are not guaranteed to match across a size range.** Because variant types are per product, one range can use one set of codes on most sizes and different codes on another (seen: 54 sizes on one set, one size on different codes, one size with no mounting variant). A script, preview or mount keyed on variant codes must read the codes of every product in the range, not one sample. *Verified by query, 2026-09-28.*

In a template export (`__print_product.yml`), each product's variant types carry `published` at type and value level, and `custom.hidden`; unpublished types and values travel with the rest. *Verified by reading source, 2026-09-29.*

**Conditional (child) variant types in a per-product archive.** A variant type that shows only for one value of another type sits **inside its parent type's `children:` list**, with `trigger_value_code: <parent value code>` placed after `custom: {}` and before `variant_values:`. `order` counts across parents and children together, and a value with no price is `price: ''`. Read from the storefront, the child carries `parent_id` and `trigger_value_id`, and `price_forecast` adds the parent value's price and the child value's price. Generated archives with children import cleanly (8 products, 111 price checks, 0 mismatches). Platform-level. *Verified by query against a real export, 2026-10-05.*

**A priced number variant must ship with a default value.** A number variant's price formula sits on the variant type (`price: value * 20.99`, `variant_values: []`). With a formula and no `default_value`, a template import and a product-only import both answered 500 and left a partial product holding only the variants before the first number variant. The same types with `default_value: '0'` import cleanly, through `variant_types/import` and inside a full template tar. Likely cause: the formula is validated with the default, and `nil * 20.99` is invalid. Platform-level. *Verified by query, 2026-10-06.* Pricing behavior: `30_PRICING_ENGINE.md` § Number Variant Price Formulas.

**Rolling a changed template option or variant set across a live range:** configure it on one template, then copy it to the others with the admin Bulk Update Tools (Advanced tab). Never hand-edit dozens of templates and never delete and re-import. See `18_ADMIN_NAVIGATION.md`. *Stated by Alex, 2026-09-26 and 2026-09-29.*

## Pricing and POS-Relevant Choices Belong on Variants, Not Template Options

Anything that affects price, needs a customer choice, or would have to be entered by hand on a point of sale belongs on a **Product Attribute variant**. Manual and POS orders placed through the Order API pull **product variants only, never template options**, so a custom tool built only on template options cannot be re-created at a counter. Template options remain the right place for design-side inputs (photos, text) that do not change the price. *Stated by Alex, 2026-09-21 and 2026-09-22.*

A Product Attribute links to **one** template; one template can be used by **many** Product Attributes. *Stated by Alex, 2026-09-22.*
OrderHub custom orders read the same way: variants only. Production choices a counter has to re-create, such as canvas edge or wrap and mounting, must therefore be Product Attribute variants, not design or template options. *Stated by Alex on a call, 2026-09-29.*

**`pos_hidden`** is the standard boolean custom field on the VariantType for hiding a variant in the POS: choice variants leave it unchecked, tool-written values (a tool's price, page count, spec and mount) have it checked. A site needs the `pos_hidden` and `hidden` definitions on VariantType before it can take a product built this way. Detail in `26_CUSTOM_DESIGN_TOOLS.md` § Every customer choice on a variant. *Stated by Alex, 2026-09-26.*

## Changelog
- 2026-06-19: Added section 4.8 `toggle` selector (2-value animated CSS-only switch on `product/px-options`), including the `toggle_hide_labels` bare-switch option, guard/fallback behavior, and primary-colour sourcing. Added cart-context note (7) that toggle is product-page only. Added `toggle` and `toggle_hide_labels` to the recognize-and-document list (8).
- 2026-07-28: Added 5.2c — option input names differ between the product page (`variants[code]`) and project-edit (`book[options][code]`); scripts must suffix-match and must handle hidden inputs. Source: claude-chat.
- 2026-08-29: Added the single-value variant type gotcha — one value is auto-selected and inherits the theme's selected-button styling, producing a large fixed pill that costs roughly 190 px per group; includes the markup tree, the `:only-child` CSS fix that reverts itself when a second value is added, and the two things that need an admin change rather than CSS. Added: unset booleans export as the quoted string `'false'` and read truthy in Liquid, affecting `hidden`, `read_only` and `hide_from_cart` inside `custom` — re-check both flags in admin after importing any option archive. Source: claude-chat.
- 2026-09-09: Added the platform bug where a required file-upload option behind a trigger silently kills Add to Cart — symptom, cause, evidence table, the independent confirmation, the correction that `disable_required_form` does not fix it, the three workarounds and the two debugging techniques. Extended §4.6 `quick-quantity` with the no-`name` consequence for `px-option-selector` and `px-product-price` (display fault, cart correct), the three add-to-cart handler hardening rules, the cloned-button trap, the `getEventListeners` diagnostic and the missing `t: ns: 'variants'` translation filter. Added that `value.price` exports blank rather than zero, so `!= 0` renders `+$0.00` on free values, and the `| plus: 0` normalisation. Added the two rules for grouped value bands (order-independent collection, opt-in on the group field) and the recorded test failure where `== blank` dropped ungrouped values. Added the two `collection_filters` syntaxes. Added that blank ids in a standalone template-options import create new records. Added the variant type export shape and the shared-object price constraint, with the unverified update-in-place claim flagged. Source: claude-chat, fireflies-call.
- 2026-09-24: Corrected "Variant Type Exports": variant types belong to one Product Attribute and are not shared; variant-type imports never update in place. Added "Pricing and POS-Relevant Choices Belong on Variants" and the one-template-per-Product-Attribute rule. Source: claude-chat, fireflies-call, slack-message.
- 2026-09-29: `px-image-upload` value format, live crop-aspect-ratio, no crop box without a ratio. Kiosk_mode_only stored as text hides the option. Option trees with layout substitutions via options import. Variant codes differ per product in a range; template export carries published/hidden; bulk update pointer. OrderHub custom orders read variants only. Source: claude-chat, fireflies-call.
- 2026-10-06: § 3.1: the string "false" trap arriving with a template export from another site, the blank-key clearing fix and the strip-undefined-keys prevention. New § Template Option Substitutions: Target Elements, Types and Limits (255-character `target_element_name`, comma-separated targets, tag targeting, substitution type list, color option behavior). § Variant Type Exports: conditional child variant types in a per-product archive. New § 4.9 `hide_value_labels`. § 5.1: an image upload crops to fill regardless of `crop="false"`, with the hidden `image_crop_flag` option fix. CORRECTED § Variant Type Exports: a variant types export does carry `hidden`, `read_only` and `hide_from_cart`; added the per-product variant types export and import routes. § Pricing and POS-Relevant Choices: `pos_hidden` pointer. Source: claude-chat, vault-doc.
- 2026-10-06 (later): § 6.1 what `data-px-no-element-substitutions` does (keeps an input out of preview URLs) and the per-option `skip_for_previews` override. § 6.2 options written into the page URL by `storeOptionSelectionIntoURL`, the long-value crash and the ignore-list fix. Source: gmail (core developer, 2025).
- 2026-10-09: § 4.2 and § 4.9 the hide-color-label checklist did not reach `selector == 'color'` swatches until the 2026-10-08 parent fix; child overrides keep the defect. Template Option Substitutions: tag targeting works in the editor; a substitution naming a missing element does nothing silently; a variant and a template option must not share a code; copying design options between sites. Variant Type Exports: a priced number variant needs `default_value: '0'` to import. Source: claude-chat.
