# Sticker Designer (legacy): tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
5.31 (4 Nov 2026). Being retired: sites move to Sticker Studio.

As recorded on 3 Nov 2026: Status: live on older installs, version 5.30 (per the CHANGELOG). New installs use Sticker Studio. Snippet: `product/sticker-designer`. Mount option: the `custom_script` field on the `sticker_artwork` template option. Prefix: `stk_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper uploads artwork in a pop-up (or inline) editor, picks shape and size, and the tool builds a print PDF with a contour cut line.

Example mount:
```liquid
{% snippet 'product/sticker-designer',
	stk_display: 'modal',
	stk_cut_spot: 'CutContour',
	stk_cut_stroke: '0.5',
	stk_cut_layer: 'true',
	stk_cut_overprint: 'true',
	stk_cut_split: 'false',
	stk_max_mb: '50',
	stk_target_dpi: '300',
	stk_price_footer: 'each',
	stk_quantity_tiers: '',
	stk_type_toggle: 'false',
	stk_artwork_check: 'true',
	stk_requires_design: 'true'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `stk_display` | Pop-up editor behind a button, or open on the page | `modal` or `inline` | `modal` | **Yes** |
| `stk_cut_spot` | Spot color name your RIP or cutter reads as the cut line | A name, letter for letter, for example `CutContour` or `CutPath` | `CutPath` | **With care**: must match the cut spot name in your RIP exactly, or the line prints instead of cutting |
| `stk_cut_stroke` | Cut line stroke width | A number in points | `0.5` | **With care** |
| `stk_cut_layer` | Puts the cut line on its own PDF layer | `true` or `false` | `true` | **With care**: match your RIP |
| `stk_cut_overprint` | Sets the cut line to overprint | `true` or `false` | `true` | **With care**: match your RIP |
| `stk_cut_split` | Cut line as a separate file instead of inside the print file | `true` or `false` | `false` | **With care**: match your production |
| `stk_max_mb` | Largest file a shopper can upload | A number in MB | `50` | **Yes** |
| `stk_target_dpi` | Resolution of the print file the tool renders | A number from 72 to 1200 | `300` | **With care**: your print quality bar |
| `stk_price_footer` | Price line in the tool footer | `off`, `total` or `each` | `off` | **Yes** |
| `stk_quantity_tiers` | Quantity buttons shown in the tool | Numbers separated by commas, or empty for none | empty (no buttons) | **With care**: should match the quantities your pricing expects |
| `stk_qty_min`, `stk_qty_max` | Limits for the quantity buttons and typed quantity | Numbers | the product min and max quantity | **Yes** |
| `stk_type_toggle` | Shows a switch between sticker formats (uses the collection filter) | `true` or `false` | `false` | **Yes** |
| `stk_artwork_check` | Runs the artwork check and records the result | `true` or `false` | `true` | **Yes** |
| `stk_artwork_check_code` | Template option the artwork check result is written to | An option code | `artwork_check` | **No**: must match the option on your template |
| `stk_requires_design` | Blocks Add to Cart until artwork is added | `true` or `false` | `true` | **Yes** |
| `stk_nominal_w`, `stk_nominal_h` | Starting size shown, in inches (the shopper still picks the real size) | Numbers | `3` and `3` | **Yes** |
| `stk_units` | Unit label only. Figures stay in inches, so `mm` shows the wrong label | `in` | `in` | **No**: leave at `in` |

Flags also accept `yes`/`no`, `on`/`off` and `1`/`0`. Sizes are inches only.

Things that are not set here
- Sticker size, shape and quantity prices come from the product's variants and pricing.
- Older installs may still read some of these from a product custom field or the Sticker Settings page in Custom Admin. The mount wins when it sets a value; move every setting into the mount.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
