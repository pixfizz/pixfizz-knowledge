# 3D Preview: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.7.4 live on the Shopper parent since 10 Nov 2026 (includes 0.7.3); 0.7.5 built, not released.

As recorded on 3 Nov 2026: Status: live, version 0.7.1 (as of the CHANGELOG; 0.7.2 built, not live). Snippet: `product/preview-3d`. Mount option: `p3d_mount` (hidden Text template option, custom_script field). Prefix: `p3d_`.

## Changes since 3 November 2026

- `p3d_inner_by_product` (from 0.7.3): one template shared by several products, the inside color per product, `productid:#rrggbb` pairs, comma separated. **With care.**
- `p3d_off_products` (from 0.7.3): product ids the 3D preview cannot draw (heart handle, printed inside); those keep the normal gallery. **Yes.**
- **0.7.5, not released:** `p3d_inner_field` (option or variant code the shopper picks the inside color with) and `p3d_inner_colors` (`value:#rrggbb` pairs). Order of use: the shopper's pick, then `p3d_inner_by_product`, then `p3d_inner_color`.

*Checked against src/product__preview-3d.liquid on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

Shows the shopper's design on a rotatable 3D product (mugs, pillows), either as the main product image or behind a View in 3D button. The artwork is the platform's own render of the production page. It writes nothing to the order.

Example mount (11 oz mug as the main image):
```liquid
{% snippet 'product/preview-3d',
	p3d_shape: 'mug',
	p3d_model: 'mug-11oz',
	p3d_page: 'front',
	p3d_units: 'in',
	p3d_trim_w: '8in',
	p3d_trim_h: '3.5in',
	p3d_color: '#f1f1ef',
	p3d_inner_color: '#f1f1ef',
	p3d_label: '11 oz ceramic mug',
	p3d_placement: 'inline',
	p3d_render_width: '1200' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `p3d_shape` | Which product shape is drawn | `mug`, `pillow` | `mug` | **With care**: must match the product |
| `p3d_model` | The 3D model drawn. Optional for a mug, required for a pillow | `mug-11oz`, `mug-15oz`, `mug-15oz-875`, `mug-20oz`, `pillow-linen-16` | none (mug built in code) | **With care**: must match the real product; each model is a Pixfizz asset on the parent |
| `p3d_page` | Template page rendered as the artwork | a page name in the template | `front` | **With care**: must name the page in the XML template |
| `p3d_units` | Unit for bare numbers | `in`, `mm` | `in` | **Yes** |
| `p3d_trim_w` | Print area width | a length | none, preview off | **With care**: must equal the page size in the XML template (checked at load) |
| `p3d_trim_h` | Print area height | a length | none, preview off | **With care**: must equal the page size in the XML template |
| `p3d_diameter` | Mug body diameter. Only without a model | a length | none | **With care**: must match the real mug |
| `p3d_height` | Mug body height. Only without a model | a length | none | **With care**: must match the real mug |
| `p3d_print_bottom` | Mug base to the bottom of the print area. Only without a model | a length | none | **With care**: must match the real mug |
| `p3d_color` | Product color (ceramic, or fabric for a pillow) | hex | `#ffffff` | **Yes** |
| `p3d_inner_color` | Inside of the mug | hex | same as `p3d_color` | **Yes** |
| `p3d_label` | Viewer title | text | blank | **Yes** |
| `p3d_placement` | Where the 3D preview appears | `inline` (replaces the main image), `image` (View in 3D button on the image), `below` (button under the gallery) | `image` | **Yes** |
| `p3d_render_width` | Artwork render width in px | number, platform caps at 1200 | `1200` | **Yes** |
| `p3d_accent_variant` | Variant whose value colors the handle and rim (accent mugs). Needs a mug model | variant code | off | **With care**: must match the variant code |
| `p3d_accent_colors` | Accent color per variant value; unlisted values stay plain | `value:#hex` pairs | none | **With care**: values must match your variant |
| `p3d_accent_parts` | Which parts take the accent | `handle`, `rim`, `handle,rim` | `handle,rim` | **Yes**: match the real mug |
| `p3d_rim_depth` | Glaze depth below the top edge | a length | `0.04in` | **Yes** |
| `p3d_handle_stub` | White stub where the handle joins the mug | a length | `0.15in` | **Yes** |
| `p3d_handle_field` | Form field (variant code or field name) that picks a sport handle. Needs a model with a sport handle | code or field name | off | **With care**: must match the field on the product |
| `p3d_handle_styles` | Handle style per field value | `value:style` pairs; styles `plain`, `football`, `baseball` | none | **With care**: values must match your field |

Things that are not set here
- The 3D models. They are made by Pixfizz and uploaded to the parent.
- The artwork. It is always the platform render of `p3d_page`, so the preview shows what prints.

Not yet confirmed: `mm` as a unit value is not confirmed; use inches.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
