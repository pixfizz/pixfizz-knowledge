# Business Cards: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
3.7.2 is the newest release (Playpoint name cards, Nov 2026). Which version each site runs is not checked here: the console start-up line shows it.

As recorded on 3 Nov 2026: Status: live, version 3.6.0 (latest built, per the CHANGELOG; 3.x is the current tool). Snippet: `product/business-cards`. Mount option: `px_mount`. Prefix: `bc_`.

*Not re-checked against 3.7.x on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper uploads print-ready card artwork or starts from one of the designs, types their details, and sees a trimmed proof (front and back) before ordering. The tool builds the production print file.

Example mount:
```liquid
{% snippet 'product/business-cards',
	bc_units: 'in',
	bc_trim_w: '3.5in',
	bc_trim_h: '2in',
	bc_bleed: '0.125in',
	bc_safe: '0.125in',
	bc_xml_w: template.width,
	bc_xml_h: template.height,
	bc_xml_min_dpi: template.minimum_dpi,
	bc_min_dpi: '300',
	bc_max_mb: '100',
	bc_sides_variant: 'bc_sides',
	bc_stock_variant: 'bc_stock',
	bc_corners_variant: 'bc_corners',
	bc_corner_radius: '0.125in',
	bc_designs: 'true',
	bc_title: 'Business cards',
	bc_intro: 'Upload your print-ready file, or start from one of our designs and type your details.',
	bc_trust: 'Bleed added for you|Proof before you order'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `bc_units` (required) | Units shown to the shopper, and the unit of any length typed without one | `in` or `mm` | `in` | **Yes** |
| `bc_trim_w`, `bc_trim_h` (required) | Finished card width and height | A length, e.g. `3.5in` or `89mm` | none (required) | **With care**: must match the XML template page size and your cutter. Ignored when `bc_xml_w` / `bc_xml_h` are set |
| `bc_bleed` (required) | Bleed added around the trim | A length, e.g. `0.125in` or `3mm` | none (required) | **With care**: must match your production bleed |
| `bc_safe` (required) | Safe margin inside the trim (the dashed line) | A length | none (required) | **With care**: match your production spec |
| `bc_xml_w`, `bc_xml_h`, `bc_xml_min_dpi` | Reads size and minimum dpi straight from the template XML; when set they win over the trim and dpi settings | Leave as `template.width`, `template.height`, `template.minimum_dpi` | blank | **No**: leave as shown so the tool always follows the XML |
| `bc_min_dpi` (required) | Minimum resolution for an uploaded print file | A number, e.g. `300` | none (required) | **With care**: your print quality bar |
| `bc_low_dpi` | Below this, a photo or logo placed in a design gets a "may look soft" warning | A number | `100` | **Yes** |
| `bc_max_mb` (required) | Largest file a shopper can upload | A number in MB | none (required) | **Yes** |
| `bc_sides_variant`, `bc_stock_variant`, `bc_corners_variant` (required) | Which product variant codes carry printed sides, paper and corners | Variant codes | none (required) | **No**: must match your variant codes exactly. Change only if you named those variants differently |
| `bc_corner_radius` (required) | Corner radius used for a corners value that does not state one (e.g. `rounded`) | A length | none (required) | **With care**: match your rounding die |
| `bc_designs` (required) | Shows the "start from one of our designs" library | `true` or `false` | none (required) | **Yes** |
| `bc_designs_file` | Which designs file to load | An asset name | `business-cards-designs.json` (square cards use `business-cards-designs-25x25.json` or `business-cards-designs-2x2.json`) | **With care**: the file must match the card size. A designs file is per trim size: from 3.2.0 the tool refuses a file whose width x height does not match the card trim, and a finished file is never scaled to another size (the bleed extension is applied per size, so each size is generated from the source design). *Verified by reading source, 2026-11-02.* |
| `bc_designs_more_file` | Your own designs file, listed before the standard designs | An asset name on your site | blank | **Yes** |
| `bc_designs_more_assets` | The image assets your own designs file uses | Asset names separated by `\|` | blank | **Yes** (must list every asset that file uses) |
| `bc_size_links` | Shows a Card size switch linking sibling products | `3.5x2=<url>\|2.5x2.5=<url>` (path or product url_path) | blank (no switch) | **Yes** |
| `bc_qtys` | Quantity choices, only when quantity is not already a list on the product | Numbers separated by commas or spaces | blank | **With care**: must match your pricing |
| `bc_title` | Heading at the top of the tool | Text, no apostrophes | `Business cards` | **Yes** |
| `bc_intro` | Intro line under the heading | Text, no apostrophes | A standard intro | **Yes** |
| `bc_trust` | Short reassurance lines in the header | Lines separated by `\|` | `Bleed added for you\|Proof before you order` | **Yes** |
| `bc_look` | Extra style classes on the tool | Class names, e.g. `pxt-flyer` | blank | **Yes** |
| `bc_owns_page` | Whether the tool takes over the product page layout | `true` or `false` | `true` | **Yes** |
| `bc_print_code`, `bc_preview_code`, `bc_original_code`, `bc_spec_code` | Template options the print file, preview, original upload and job spec are written to | Option codes | `px_print_file`, `px_preview`, `px_original`, `px_spec` | **No**: your fulfillment reads these. Change only if your template uses other codes |
| `bc_add_to_cart_selector` | Which button counts as Add to Cart, on a heavily customized page | A CSS selector | blank (standard button) | **No**, unless Pixfizz support asks you to |

Lengths accept inches and millimeters (`3.5in`, `89mm`); a bare number uses `bc_units`.

Legacy Business Cards 1.x (`product/business-card`, singular, mounted on the `bc_artwork_front` option) is a separate upload-only tool still on some sites and being retired. Its settings are `bc_sides`, `bc_units`, `bc_trim_w`, `bc_trim_h`, `bc_bleed`, `bc_safe`, `bc_min_dpi`, `bc_max_mb`, `bc_output` (`NORMALISED` or `PASSTHROUGH`), `bc_placement` (`AUTO`, `FILL`, `FIT`, `TRIM`, `NONE`), `bc_show_bleed`, `bc_preview`, `bc_reposition`, `bc_price_footer` (`off`, `total`, `each`), `bc_requires_design`, `bc_block_on_error`, `bc_designer_link`, `bc_human_check`, `bc_display` (`modal` or `inline`), plus the code settings `bc_sides_code`, `bc_shape_code`, `bc_corners_code`, `bc_stock_code`, `bc_print_code`. If your site still runs 1.x, ask Pixfizz about moving to 3.x rather than tuning it.

Things that are not set here
- Paper, sides, corners and quantity are variants on the product: they carry the shopper's choice and the price.
- Prices live in the product's pricing, not the mount.
- Page size and minimum dpi live in the template XML (the mount reads them through `bc_xml_*`).

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
