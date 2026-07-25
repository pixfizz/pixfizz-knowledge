# 22 — Option & Variant Rendering

**Authority Scope:** Variant rendering behavior only.

_Last updated: 2026-06-19_

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

## Changelog
- 2026-06-19: Added section 4.8 `toggle` selector (2-value animated CSS-only switch on `product/px-options`), including the `toggle_hide_labels` bare-switch option, guard/fallback behavior, and primary-colour sourcing. Added cart-context note (7) that toggle is product-page only. Added `toggle` and `toggle_hide_labels` to the recognize-and-document list (8).
