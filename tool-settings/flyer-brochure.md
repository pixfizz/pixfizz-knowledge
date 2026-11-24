# Flyer / Brochure: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
1.0.4 live (10 Nov 2026); 1.1.0 built, not released.

As recorded on 3 Nov 2026: Status: live, version 1.0.3 (per the CHANGELOG; 1.0.3 changes the asset only, the snippet is the 1.0.1/1.0.2 one). Snippet: `product/flyer-brochure`. Mount option: `px_mount`. Prefix: `ff_`.

## Changes since 3 November 2026

- 1.1.0, not released: two-level paper (a paper finish that opens a paper weight choice) shows a Weight row. It comes from your variants, not from a mount setting.

*Checked against the CHANGELOG on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper uploads a print-ready flyer, postcard or folded brochure, sees it trimmed (and folded in 3D for brochures) before ordering, and picks paper, color, sides, finish and quantity. The tool builds the print file.

Example mount (tri-fold, inches):
```liquid
{% snippet 'product/flyer-brochure',
	ff_fold: 'c',
	ff_units: 'in',
	ff_trim_w: '11in',
	ff_trim_h: '8.5in',
	ff_panels: '3.6250in,3.6875in,3.6875in',
	ff_bleed: '0.125in',
	ff_safe: '0.125in',
	ff_min_dpi: '300',
	ff_max_mb: '200',
	ff_paper: 'paper',
	ff_color: 'print_color',
	ff_qtys: '50,100,250,500,1000',
	ff_qty_default: '250',
	ff_word: 'brochures|brochure',
	ff_title: 'Tri-fold brochures',
	ff_intro: 'Upload your print-ready PDF: page 1 is the outside, page 2 the inside.',
	ff_trust: 'Checked before you order|Folded preview',
	ff_look: 'pxt-flyer'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `ff_fold` (required) | The fold | `flat` (flyers, postcards), `h` (half fold), `c` (tri-fold, C-fold), `z` (Z-fold) | none (required) | **With care**: must match the product and your folding |
| `ff_units` (required) | Units shown to the shopper | `in` or `mm` | `in` | **Yes** |
| `ff_trim_w`, `ff_trim_h` (required) | Flat trimmed sheet size (unfolded) | Lengths, e.g. `11in` or `297mm` | none (required) | **With care**: must match the template and your cutter |
| `ff_panels` (required unless `flat`) | Panel widths of page 1, left to right; must add up to `ff_trim_w` | Lengths separated by commas | none (required) | **With care**: must match your fold positions |
| `ff_bleed`, `ff_safe` (required) | Bleed around the trim, and safe margin inside it | Lengths | none (required) | **With care**: match your production |
| `ff_min_dpi` (required) | Resolution of the print file | A number | none (required) | **With care**: your print quality bar |
| `ff_low_dpi` | Below this the shopper is warned the print may look soft | A number | `100` | **Yes** |
| `ff_max_mb` (required) | Largest file a shopper can upload | A number in MB | none (required) | **Yes** |
| `ff_paper` (required) | Variant (or template option) that carries the paper | A code | none (required) | **No**: must match your code exactly |
| `ff_color`, `ff_sides`, `ff_finish` | Variants for print color, printed sides (flat pieces) and lamination. Blank hides that step | Codes, or blank | blank | **No**: must match your codes. Blank is fine if you do not offer that choice |
| `ff_sides_fixed` | For a flat piece with no sides choice: always one side or both | `1` or `2`, or blank | blank (one side, unless `ff_sides` is set) | **With care**: must match what the product prints |
| `ff_hide` | Variant values the tool must not offer | `CODE:VALUE` pairs separated by spaces | blank | **Yes**, with exact codes |
| `ff_qtys` | Quantity choices, only when quantity is not already a list on the product | Numbers separated by commas | blank | **With care**: must match your pricing |
| `ff_qty_default` | Starting quantity | A number | blank | **Yes** |
| `ff_sizes` | Size switch linking sibling size pages | `WxH=url` or `WxH Name=url`, separated by `\|` | blank (no switch) | **Yes** |
| `ff_design_url` | Adds a "Design it online" link | A URL | blank | **Yes** |
| `ff_hard_proof` | Shows a "send me a printed proof" checkbox | `true` or `false` | `false` | **Yes** |
| `ff_hard_proof_code` | Text option that receives yes or no for the printed proof | An option code | blank | **No**: must match the option on your template |
| `ff_word` | What the product is called in the tool | `plural\|singular`, e.g. `brochures\|brochure` | `pieces\|piece` for flat, brochures for folded | **Yes** |
| `ff_title`, `ff_intro` | Heading and intro line | Text, no apostrophes | `Print-ready flyers and brochures` and a standard intro | **Yes** |
| `ff_trust` | Short reassurance lines in the header | Lines separated by `\|` | `Checked before you order\|Proof before you order` | **Yes** |
| `ff_look` | Extra style classes on the tool | Class names, e.g. `pxt-flyer` | blank | **Yes** |
| `ff_owns_page` | Whether the tool takes over the product page layout | `true` or `false` | `true` | **Yes** |
| `ff_file_prefix` | Start of the print file name | Text | `flyer` (flat) or `brochure` (folded) | **With care**: your production may sort by file name |
| `ff_print_code`, `ff_preview_code`, `ff_original_code`, `ff_spec_code` | Template options the print file, preview, original and job spec are written to | Option codes | `px_print_file`, `px_preview`, `px_original`, `px_spec` | **No**: your fulfillment reads these |
| `ff_add_to_cart_selector` | Which button counts as Add to Cart, on a heavily customized page | A CSS selector | blank | **No**, unless Pixfizz support asks you to |

Lengths accept inches and millimeters (`11in`, `297mm`). From 1.0.3, a mm mount shows mm everywhere.

Legacy flyer-fold 0.5.x (`product/flyer-fold`, mounted on `flyer_mount`, `flyer_` prefix) is a separate older tool being retired. Its settings are `flyer_fold` (`c`, `z` or `h`), `flyer_trim_w`, `flyer_trim_h`, `flyer_panels` (mm), `flyer_bleed`, `flyer_safe`, `flyer_units`, `flyer_variants`, plus optional `flyer_size`, `flyer_fold_name`, `flyer_title`, `flyer_intro`, `flyer_options_title`, `flyer_trust`, `flyer_crease_gsm`, `flyer_qty_default`, `flyer_qtys`, `flyer_select_url`, `flyer_select_label`, `flyer_file_code`, `flyer_preview_code`, `flyer_spec_code`. Ask Pixfizz about moving to 1.0.x rather than tuning it.

Limits of 1.0.x (tool-level)
- **No triggered child variant trees.** 1.0.x drives one code per step (`ff_paper`, `ff_color`, `ff_sides`, `ff_finish`). It cannot drive a parent value that shows a child variant (for example a paper type value that triggers a child variant holding the paper weights). 0.5.x could, through `flyer_variants` with `CODE.*` patterns. A site with a paper tree stays on 0.5.x until the tool gets a two-level paper step. *Verified by reading source (1.0.3 has no trigger or child handling) and a live lab page, 2026-11-02.*
- **File carriers are not hidden by the tool.** 1.0.x hides the store controls of its own steps, but not the file carriers. When a mount reuses a visible store file option as the print carrier (for example `ff_print_code: 'document_file_upload'`), add a style line after the snippet call in the same `custom_script` that hides that option and the preview and spec carriers. 0.5.x hid the file option in script, so a straight swap shows an extra File Upload box. *Verified by reading source; carrier swap tested in the 1.0.3 test harness, 2026-11-02.*
- **`ff_original_code` is optional.** With no such option on the template the tool skips the originals zip, so there is no edit-from-cart restore of the original file. *Verified by reading source, 2026-11-02.*

Things that are not set here
- Paper, color, sides, finish and quantity are variants on the product: they carry the shopper's choice and the price.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
