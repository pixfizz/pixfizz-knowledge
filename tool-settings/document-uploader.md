# Document Uploader: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: live, version 1.3.3 (per the CHANGELOG). To be replaced by Uploader 2.0. Snippet: `product/document-uploader`. Mount option: `du_mount`. Prefix: `du_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper uploads a document (PDF), confirms its page count, picks paper, color, sides and finishing, sees the price, and adds it to the cart. One product, one template, one trim size per mount.

Example mount:
```liquid
{% snippet 'product/document-uploader',
	du_size: '85x11',
	du_bleed: '0.125',
	du_safe: '0.125',
	du_crop_marks: 'false',
	du_max_mb: '100',
	du_min_dpi: '150',
	du_full_width: 'true',
	du_single_cart: 'true',
	du_omit_empty: 'true',
	du_merge_size_into_paper: 'true',
	du_thumbs_max: '24',
	du_brand: '',
	du_write_preview: 'true',
	du_confirm_pages: 'true',
	du_block_on_error: 'false',
	du_file_code: 'du_file',
	du_preview_code: 'du_preview',
	du_price_code: 'du_price',
	du_pages_code: 'du_pages',
	du_sheets_code: 'du_sheets',
	du_spec_code: 'du_spec',
	du_size_code: 'du_size',
	du_paper_code: 'du_paper',
	du_color_code: 'du_color',
	du_sides_code: 'du_sides',
	du_finishing_code: 'du_finishing',
	du_notes_code: 'du_notes' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `du_size` (required) | Which size from the rate card this product prints | The code of a size entry in your rate card, e.g. `85x11` | none (reported as missing) | **With care**: must match a size code in your rate card and the template trim |
| `du_bleed` | Bleed added on every side of the print file | A number in inches | `0.125` | **With care**: match your production |
| `du_safe` | Safe margin inside the trim | A number in inches | `0.125` | **With care** |
| `du_crop_marks` | Adds crop marks to the print file (only drawn when bleed is 0.25 or more) | `true` or `false` | `false` | **With care**: match your production |
| `du_max_mb` | Largest file a shopper can upload | A number in MB | `100` | **Yes** |
| `du_min_dpi` | Minimum resolution for images inside the document | A number | `150` | **With care**: your print quality bar |
| `du_full_width` | Hides the product image column so the tool uses the full page width | `true` or `false` | `true` | **Yes** |
| `du_single_cart` | Hides the theme Add to Cart so the shopper sees only the tool button | `true` or `false` | `true` | **Yes** |
| `du_omit_empty` | Leaves empty hidden options out of the cart, emails and order | `true` or `false` | `true` | **Yes** |
| `du_merge_size_into_paper` | Writes size and paper together on one order line, e.g. "8.5 x 11 in, Gloss" | `true` or `false` | `true` | **Yes** |
| `du_thumbs_max` | Most page thumbnails shown | A number | `24` | **Yes** |
| `du_brand` | Accent color override. Empty uses your store button color | A color, or empty | empty | **Yes** |
| `du_write_preview` | Saves a preview image for the cart and order | `true` or `false` | `true` | **Yes** |
| `du_confirm_pages` | Page-count confirmation step | `true` or `false` | `true` | **Yes** |
| `du_block_on_error` | Blocks the order when the file check finds an error | `true` or `false` | `false` | **Yes** |
| `du_file_code`, `du_preview_code`, `du_price_code`, `du_pages_code`, `du_sheets_code`, `du_spec_code`, `du_size_code`, `du_paper_code`, `du_color_code`, `du_sides_code`, `du_finishing_code`, `du_notes_code` | Template options the tool writes the file, preview, price, pages, sheets, job spec and each choice to | Option codes | the same name as the setting without `_code`, e.g. `du_file`, `du_price` | **No**: must match the options on your template; your fulfillment reads them |

Bleed and safe are plain numbers in inches.

Things that are not set here
- Rates, papers, finishing and the volume ladder live in the rate card snippet `product/document-uploader-rates`, which your site overrides with its own. Never in the mount.
- The rate card also sets how double-sided is priced (`duplex_mode`) and what the volume ladder counts (`tier_driver`).

Not yet confirmed: `du_confirm_pages` and `du_block_on_error`: the exact effect on the page is not confirmed.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
