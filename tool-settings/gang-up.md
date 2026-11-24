# Gang Up (DTF gang sheet): tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
2.2.0 live; 2.3.0 built, not released (24 Nov 2026).

As recorded on 3 Nov 2026: Status: live, version 2.2.0 (latest built, per the CHANGELOG). Snippet: `product/gang-up`. Mount option: `px_mount`. Prefix: `gu_`.

## Changes since 3 November 2026

- **New in 2.3.0, not released:** `gu_cut_shape`: `contour` (default, the traced outline) or `rect` (each design cut as a rectangle round its trimmed artwork, at `gu_cut_offset`). **With care**: match how your cutter should cut. Mounts without it keep the contour cut.

*Checked against docs/releases/2.3.0-unreleased on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper uploads their designs, sets size and copies for each, and the tool packs them onto a roll of film. The shopper pays for the length used, and the tool builds the print file (and an optional cut file).

Example mount:
```liquid
{% snippet 'product/gang-up',
	gu_roll_width: '22in',
	gu_margin: '0.25in',
	gu_gap: '0.125in',
	gu_min_length: '12in',
	gu_max_length: '120in',
	gu_length_variant: 'length',
	gu_min_dpi: '300',
	gu_max_mb: '50',
	gu_accept: 'png jpg webp svg',
	gu_trim: 'true',
	gu_units: 'in',
	gu_view: 'auto',
	gu_output: 'pdf',
	gu_cut: 'false',
	gu_cut_spot: 'CutContour',
	gu_title: 'Custom DTF Gang Sheet',
	gu_intro: 'Upload your designs and set the size and copies. You pay only for the length you use.',
	gu_trust: 'Full white underbase|Pay only for the length you use|File checked before you order'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `gu_roll_width` (required) | Printable width of the film roll. Its presence also tells the platform this is a 2.x mount | A length, e.g. `22in` or `559mm` | none (a mount without it runs the old 1.x tool) | **With care**: must match the roll on your printer |
| `gu_margin` (required) | Empty edge left on each side of the sheet | A length | `0.25in` if unreadable | **With care**: match your printer |
| `gu_gap` (required) | Space between designs | A length | `0.125in` if unreadable | **With care**: enough to cut or weed between designs |
| `gu_min_length`, `gu_max_length` (required) | Shortest and longest sheet a shopper can order | Lengths | `12in` and `120in` if unreadable | **With care**: must match the lengths your pricing covers |
| `gu_length_variant` (required) | The product variant whose value is the priced sheet length | A variant code | `length` | **No**: must match your length variant code exactly |
| `gu_min_dpi` (required) | Minimum resolution for an uploaded design | A number | `300` if unreadable | **With care**: your print quality bar |
| `gu_max_mb` (required) | Largest file a shopper can upload | A number in MB | `50` if unreadable | **Yes** |
| `gu_accept` (required) | File types the shopper may upload | Any of `png jpg webp svg`, space separated | `png` if unreadable | **Yes** (transparent PNG is the safe choice for DTF) |
| `gu_trim` (required) | Automatically trims empty transparent space around each design | `true` or `false` | `true` | **Yes** |
| `gu_units` (required) | Units shown to the shopper | `in` or `mm` | none (required) | **Yes** |
| `gu_view` (required) | Background the sheet is shown on | `auto`, `checker`, `white`, `grey`, `charcoal`, `black` | `auto` | **Yes** |
| `gu_view_options` | Which backgrounds the shopper can switch between | Any of `checker white grey charcoal black custom`, space separated | all six | **Yes** |
| `gu_output` (required) | Print file format | `pdf` or `png` | `pdf` | **With care**: must be what your RIP accepts |
| `gu_cut` (required) | Also builds a contour cut line around each design | `true` or `false` | `false` | **With care**: only for a print-and-cut workflow |
| `gu_cut_offset` | How far the cut line sits outside each design | A number in inches | `0.0625` (max `0.5`) | **With care**: match your cutter |
| `gu_cut_clearance` | Extra space between neighboring cut lines | A number in inches | `0.0625` (max `1`) | **With care** |
| `gu_cut_spot` | Spot color name your cutter reads as the cut line | A name, letter for letter, for example `CutContour` or `CutPath` | `CutContour` | **With care**: must match the cut spot name in your RIP exactly, or the line prints instead of cutting |
| `gu_cut_output` | Cut line in the print file, or in a separate file | `combined` or `split` | `combined` | **With care**: match your production |
| `gu_cut_stroke` | Cut line stroke width in points | A number, 0.1 to 5 | `0.5` | **With care** |
| `gu_cut_page_units` | How the cut file page size is written | `points` or `userunit` | `points` | **With care**: change only if your RIP reads the size wrong |
| `gu_samples` | Shows a sample design before the first upload | `true` or `false` | `true` | **Yes** |
| `gu_title`, `gu_intro` | Heading and intro line | Text, no apostrophes | standard text | **Yes** |
| `gu_trust` | Short reassurance lines in the header | Lines separated by `\|` | `Full white underbase\|Pay only for the length you use` | **Yes** |
| `gu_look` | Extra style classes on the tool | Class names | blank | **Yes** |
| `gu_owns_page` | Whether the tool takes over the product page layout | `true` or `false` | `true` | **Yes** |
| `gu_print_code`, `gu_cut_code`, `gu_preview_code`, `gu_original_code`, `gu_spec_code` | Template options the print file, cut file, preview, originals and job spec are written to | Option codes | `px_print_file`, `px_print_cut`, `px_preview`, `px_original`, `px_spec` | **No**: your fulfillment reads these |
| `gu_check_code` | A text option that receives the artwork check result | An option code, e.g. `artwork_check` | blank (off) | **With care**: the option must exist on the template |
| `gu_add_to_cart_selector` | Which button counts as Add to Cart, on a heavily customized page | A CSS selector | blank | **No**, unless Pixfizz support asks you to |

Lengths accept inches and millimeters (`22in`, `559mm`). Gap and margin are raised automatically when the cut line is on, so cut lines never overlap.

Things that are not set here
- The price per length is the product pricing on the length variant, not the mount.
- Older Gang Up 1.x mounts (on the `gangup_artwork` option, without `gu_roll_width`) still work but only read the cut and trim settings; ask Pixfizz about moving to 2.x.

Not yet confirmed: `gu_cut_offset` and `gu_cut_clearance` are read as plain numbers in inches; a unit suffix is not confirmed to work.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
