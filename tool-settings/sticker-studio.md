# Sticker Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
6.3.0 live on the Shopper parent since 3 Nov 2026; 6.4.0 built, not released.

As recorded on 3 Nov 2026: Status: live, version 6.2.2 (per the CHANGELOG). Snippet: `product/sticker-studio`. Mount option: `px_mount`. Prefix: `stk_`.

## Changes since 3 November 2026

- **6.4.0, not released:** the built-in `stk_bleed` default becomes `0.125` (1/8 in, what Sticker Designer 5.30 printed), so a die cut print PDF grows by 1/8 in each side. A kiss cut still stops at its backer margin. Mounts that pass `stk_bleed` keep their own value. Note: the 6.4.0 snippet's own comment still says 0.0625; the script uses 0.125.

*Checked against docs/releases/6.4.0-unreleased on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper uploads artwork, the tool traces the cut line around it (contour or a chosen shape), shows a proof, and builds the print file and cut file for a print-and-cut workflow.

Example mount:
```liquid
{% snippet 'product/sticker-studio',
	stk_size_variant: 'size',
	stk_finish_variant: 'finish',
	stk_border_min: '0.04',
	stk_border_max: '0.25',
	stk_border_default: '0.08',
	stk_max_mb: '50',
	stk_xml_min_dpi: template.minimum_dpi,
	stk_cut_spot: 'CutContour',
	stk_cut_output: 'combined',
	stk_title: 'Custom die-cut stickers',
	stk_trust: 'Cut line traced for you|Proof before you order'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `stk_size_variant` (required) | The variant that carries the sticker size | A variant code whose values are W x H in inches, e.g. `3x3` | `size` | **No**: must match your size variant code |
| `stk_cut_spot` (required) | Spot color name your RIP or cutter reads as the cut line | A name, letter for letter, for example `CutContour` or `CutPath` | none (required) | **With care**: must match the cut spot name in your RIP exactly, or the line prints instead of cutting |
| `stk_cut_output` (required) | Cut line inside the print file, or as a separate cut file | `combined` or `split` | `combined` | **With care**: match your production |
| `stk_max_mb` (required) | Largest file a shopper can upload | A number in MB | `50` if unreadable | **Yes** |
| `stk_border_min`, `stk_border_max`, `stk_border_default` (required) | Smallest, largest and starting white border around the artwork | Numbers in inches | `0.04`, `0.25`, `0.08` if unreadable | **With care**: within what your cutter can hold |
| `stk_xml_min_dpi` | Minimum dpi read from the template XML; wins over `stk_min_dpi` | Leave as `template.minimum_dpi` | blank | **No**: leave as shown |
| `stk_min_dpi` | Minimum dpi when the XML gives none | A number | `300` | **With care**: your print quality bar |
| `stk_low_dpi` | Below this the shopper is warned the print may look soft | A number | `100` | **Yes** |
| `stk_backing_variant` | Variant that carries kiss cut, die cut with border, or die cut without border (read from the value names). May list several codes as `a\|b`; the first one the product has is used | Variant code(s) | blank | **No**: must match your variant code(s) |
| `stk_backing_kind` | Cut style used when the product has no backing variant | `border`, `noborder` or `kiss` | `border` | **With care**: match the product you sell |
| `stk_finish_variant` | Variant that carries the finish (e.g. matte or gloss) | A variant code | blank | **No**: must match your variant code |
| `stk_cut_variant`, `stk_cut_value` | A fixed variant the tool sets for every order, e.g. a cut type of custom shape | A variant code and one of its value codes | blank | **No**: must match your variant and value codes |
| `stk_hide_variants` | Other variants to hide on the page because the tool handles them | Variant codes, space separated | blank | **Yes**, as long as the codes are right |
| `stk_kiss_margin` | Margin around the artwork on a kiss cut | A number in inches | `0.0625` | **With care** |
| `stk_no_border` | Cut distance on a die cut with no border | A number in inches | `0.02` | **With care**: match your cutter tolerance |
| `stk_cut_stroke` | Cut line stroke width | A number in points | `0.5` | **With care** |
| `stk_title`, `stk_intro` | Heading and intro line | Text, no apostrophes | `Custom stickers` and a standard intro | **Yes** |
| `stk_trust` | Short reassurance lines in the header | Lines separated by `\|` | `Cut line traced for you\|Proof before you order` | **Yes** |
| `stk_look` | Extra style classes on the tool | Class names | blank | **Yes** |
| `stk_owns_page` | Whether the tool takes over the product page layout | `true` or `false` | `true` | **Yes** |
| `stk_print_code`, `stk_cut_code`, `stk_preview_code`, `stk_original_code`, `stk_spec_code` | Template options the print file, cut file, preview, original and job spec are written to | Option codes | `px_print_file`, `px_print_cut`, `px_preview`, `px_original`, `px_spec` | **No**: your fulfillment reads these |
| `stk_add_to_cart_selector` | Which button counts as Add to Cart, on a heavily customized page | A CSS selector | blank | **No**, unless Pixfizz support asks you to |

Settings added in 6.1 and 6.2 (read by the tool; confirm with Pixfizz that your site has the 6.2 snippet before relying on them):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `stk_shape_variant` | Variant that carries the shape (contour, circle, square, oval, rectangle, heart, star, diamond) | A variant code | blank | **No**: must match your variant code |
| `stk_corner_radius` | Corner radius for Rounded square and rectangle shapes | A number in inches | `0.125` | **With care**: match your die or cutter |
| `stk_bleed` | How far a Fill artwork bleeds past the cut line | A number in inches | `0.0625` | **With care** |
| `stk_custom_size` | Typed sizes (width and height fields) when the product has a sheet rate card | `off` to turn off; on otherwise | on | **Yes** |
| `stk_width_variant`, `stk_height_variant`, `stk_price_variant` | Number variants the typed width, height and unit price are written to | Variant codes | `px_width_in`, `px_height_in`, `px_price` | **No**: must match your variant codes |

Border, margin and cut lengths are plain numbers in inches.

Things that are not set here
- Size, finish, backing and shape choices are variants on the product: they carry the shopper's choice and the price.
- Typed-size prices come from the product's sheet rate card (`px_rates`), not the mount.
- Sticker Designer 5.30 (`product/sticker-designer`) is a separate, older tool.

Not yet confirmed: The 6.1 and 6.2 settings are read by the tool script; whether every installed snippet passes them through is not confirmed. Ask Pixfizz before relying on them.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
