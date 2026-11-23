# Custom Sized Prints: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: live, version 0.4.1 on the shopper24 parent; 0.5.0 (metric) built, not yet released. Snippet: `product/custom-print`. Mount option: `px_mount` (hidden Text template option, one per template). Prefix: `cp_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper types any width and height inside your limits, uploads a photo or PDF, sees it on a wall or a scene, and gets an area-based price. One tool covers paper prints, metal, acrylic, canvas and banners; `cp_kind` picks which.

Example mount (paper prints, inches with a cm option):
```liquid
{% snippet 'product/custom-print',
	cp_display: 'inline',
	cp_kind: 'paper',
	cp_units: 'in,cm',
	cp_unit_default: 'in',
	cp_fractions: 'true',
	cp_step: '0.25in',
	cp_step_cm: '0.5',
	cp_min_w: '4in',
	cp_min_h: '4in',
	cp_max_w: '60in',
	cp_max_h: '60in',
	cp_roll_width: '44in',
	cp_max_long: '60in',
	cp_bleed: '0in',
	cp_safe: '0.125in',
	cp_good_dpi: '150',
	cp_min_dpi: '100',
	cp_out_dpi: '300',
	cp_max_mb: '200',
	cp_pdf: 'true',
	cp_stock_label: 'Paper',
	cp_notes: 'lustre:Soft sheen|glossy:Deep blacks, bright color|matte:No glare, soft finish' %}
```
A canvas mount adds `cp_kind: 'canvas'`, `cp_wrap: '1.5:2in|0.75:1in'` and the depth and edge variant codes. A banner mount uses `cp_kind: 'banner'`, `cp_units: 'ft,in'` and `cp_grommets_variant: 'px_grommets'`.

Lengths always carry their unit: `in`, `ft`, `cm` or `mm` (for example `0.25in`, `5ft`, `1100mm`). A bare number is read as inches.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `cp_display` | Layout of the tool | `inline` (the only layout built) | none, required | No |
| `cp_kind` | Product family: drives the preview scene, canvas wrap, grommets and wording | `paper`, `metal`, `acrylic`, `canvas`, `banner` | none, required | With care: must match the product |
| `cp_units` | Units the shopper can switch between | comma list, for example `in,cm`, `ft,in`, `cm` | none, required | Yes |
| `cp_unit_default` | Unit shown first | one of the units in `cp_units`, for example `in`, `cm`, `ft` | none, required | Yes |
| `cp_fractions` | Show sizes as fractions (17 1/4) or decimals (17.25) | `true`, `false` | none, required | Yes |
| `cp_step` | Size step when working in inches or feet | length with unit, for example `0.25in`, `1in` | none, required | Yes |
| `cp_step_cm` | Size step when working in cm (a plain number) | number, for example `0.5` | none; required when `cp_units` includes `cm` | Yes |
| `cp_min_w`, `cp_min_h` | Smallest width and height | length with unit | none, required | With care: must be a size your lab can produce and your pricing card covers |
| `cp_max_w`, `cp_max_h` | Largest width and height | length with unit | none, required | With care: as above |
| `cp_roll_width` | The longest the short side can be (roll or sheet width) | length with unit | none, required | With care: must match your printer or media |
| `cp_max_long` | The longest the long side can be | length with unit | none, required | With care: must match your printer or media |
| `cp_bleed` | Print past the cut on every side | length with unit, `0in` for prints | none, required | With care: must match your production spec |
| `cp_safe` | Keep-clear margin inside the cut, shown on the layout | length with unit | none, required | Yes |
| `cp_wrap` | Canvas only: print bleed per frame depth | `depth:bleed` pairs split by `|`, for example `1.5:2in|0.75:1in` | none; required when `cp_kind` is `canvas` | With care: depth values must match your depth variant values |
| `cp_good_dpi` | At or above this the photo is called sharp | number, for example `150` | none, required | Yes |
| `cp_min_dpi` | Below this the photo is called soft (a warning only) | number, for example `100` | none, required | Yes |
| `cp_out_dpi` | Resolution of the print file | number | none, required | With care: must match the dpi in the template XML |
| `cp_max_mb` | Largest upload accepted | number, MB | none, required | Yes |
| `cp_pdf` | Accept a PDF as well as photos | `true`, `false` | none, required | Yes |
| `cp_size_unit` | 0.5.0 and later. Unit written to the size variants, job spec and file name | `in`, `mm` | `in` | With care: `mm` needs the `px_width_mm` / `px_height_mm` variants and a metric pricing card |
| `cp_title` | Heading above the tool | text | the product name | Yes |
| `cp_intro` | Short intro line | text | blank | Yes |
| `cp_stock_label` | Label for the material choice | text, for example `Paper`, `Finish`, `Canvas`, `Vinyl` | blank | Yes |
| `cp_notes` | One line under a choice | `value:text` pairs split by `|`; prefix `depth.`, `edge.` or `grommets.` for those choices | blank | Yes |
| `cp_templates` | Artwork template downloads offered to the shopper | `label=url` pairs split by `|` | blank | Yes |
| `cp_grommets_when` | Banner stocks that take grommets; on any other stock the grommet choice is hidden | comma list of stock values | blank (every stock) | Yes |
| `cp_width_variant`, `cp_height_variant` | Variant codes the size is written to | variant code | `px_width_in` / `px_height_in` (`px_width_mm` / `px_height_mm` with `cp_size_unit: 'mm'`) | No, unless you renamed those variants |
| `cp_stock_variant` | Variant code for the material | variant code | `px_stock` | No, unless renamed |
| `cp_price_variant` | Hidden Number variant that carries the unit price | variant code | `px_price` | No, unless renamed |
| `cp_hanging_variant`, `cp_depth_variant`, `cp_edge_variant`, `cp_edgecolor_variant`, `cp_grommets_variant` | Variant codes for hanging, canvas depth, edge, edge color and grommets | variant code | blank (choice not offered) | With care: set only if the product has that variant, and use its exact code |
| `cp_print_code`, `cp_preview_code`, `cp_original_code`, `cp_place_code`, `cp_spec_code` | Template options the print file, preview, original upload, placement and job spec are written to | option code | `px_print_file`, `px_preview`, `px_original`, `px_place`, `px_spec` | No, unless renamed |

Things that are not set here
- Prices: they come from the `px_rates` card on the product (area pricing), not the mount.
- The materials, depths, edges and grommet choices themselves: those are variant values on the product.

Not yet confirmed: A bare number is read as inches for some length settings; not confirmed for every one, so always write the unit.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
