# 22 — Option & Variant Rendering

**Authority Scope:** Variant rendering behavior only.

_Last updated: 2026-04-19_

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

---

## 8) Practical “recognize and document” list (what matters when reading a site)

When you see a variant/option behaving “special” in Shopper, check these first:

- `option.type` (text/number/color/font/image_upload/file_upload vs multiple choice default)
- `option.custom.selector` (textarea, color swatch, checkbox, dropdown, slider, quick-quantity)
- `option.custom.multi_upload_group` (multi-image wrapper)
- `option.custom.kiosk_mode_only` (kiosk-only display)
- `option.custom.hidden` (completely hidden)
- `option.trigger_value` + `option.children` (conditional display / nesting)
- `option.custom.accept` (upload constraints)
- Admin checklist toggles affecting image upload adjustments
- `option.custom.custom_script` (inline scripts—dangerous but real)

That’s the set that needs to be “muscle memory” when debugging option rendering in Shopper.
