# Booklet Uploader: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: live, version 1.5.2 (per the CHANGELOG). To be merged into Uploader 2.0. Snippet: `product/booklet-uploader`. Mount option: `bu_mount`. Prefix: `bu_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper uploads a booklet PDF, confirms the page count, chooses cover, paper, color and binding, sees the price and adds it to the cart. Always full width on the page, always double-sided. One product, one template, one trim size per mount.

Example mount (the rate card block sits above the snippet call in the same field; see below):
```liquid
{% snippet 'product/booklet-uploader',
	bu_size: '55x85',
	bu_bleed: '0.125',
	bu_safe: '0.125',
	bu_crop_marks: 'false',
	bu_max_mb: '100',
	bu_min_dpi: '150',
	bu_full_width: 'true',
	bu_single_cart: 'true',
	bu_omit_empty: 'true',
	bu_merge_size_into_paper: 'true',
	bu_thumbs_max: '24',
	bu_brand: '',
	bu_write_preview: 'true',
	bu_confirm_pages: 'true',
	bu_block_on_error: 'false',
	bu_file_code: 'bu_file',
	bu_preview_code: 'bu_preview',
	bu_price_code: 'bu_price',
	bu_pages_code: 'bu_pages',
	bu_sheets_code: 'bu_sheets',
	bu_spec_code: 'bu_spec',
	bu_size_code: 'bu_size',
	bu_paper_code: 'bu_paper',
	bu_color_code: 'bu_color',
	bu_binding_code: 'bu_binding',
	bu_finishing_code: 'bu_finishing',
	bu_notes_code: 'bu_notes' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `bu_size` (required) | Which size from the rate card this product prints | The code of a size entry in your rate card, e.g. `55x85` | none (reported as missing) | **With care**: must match a size code in your rate card and the template trim |
| `bu_bleed` | Bleed added on every page of the print file | A number in inches | `0.125` | **With care**: match your production |
| `bu_safe` | Safe margin inside the trim | A number in inches | `0.125` | **With care** |
| `bu_crop_marks` | Adds crop marks (only drawn when bleed is 0.25 or more) | `true` or `false` | `false` | **With care** |
| `bu_max_mb` | Largest file a shopper can upload | A number in MB | `100` | **Yes** |
| `bu_min_dpi` | Minimum resolution for images inside the document | A number | `150` | **With care**: your print quality bar |
| `bu_full_width` | Tool uses the full page width | `true` | `true` | **No**: keep `true`; the tool is built full width |
| `bu_single_cart` | Hides the theme Add to Cart so only the tool button shows | `true` or `false` | `true` | **Yes** |
| `bu_omit_empty` | Leaves empty hidden options out of the cart, emails and order | `true` or `false` | `true` | **Yes** |
| `bu_merge_size_into_paper` | Writes size and paper together on one order line | `true` or `false` | `true` | **Yes** |
| `bu_thumbs_max` | Most page thumbnails shown | A number | `24` | **Yes** |
| `bu_brand` | Accent color override. Empty uses your store button color | A color, or empty | empty | **Yes** |
| `bu_write_preview` | Saves a preview image for the cart and order | `true` or `false` | `true` | **Yes** |
| `bu_confirm_pages` | Page-count confirmation step | `true` or `false` | `true` | **Yes** |
| `bu_block_on_error` | Blocks the order when the file check finds an error | `true` or `false` | `false` | **Yes** |
| `bu_file_code`, `bu_preview_code`, `bu_price_code`, `bu_pages_code`, `bu_sheets_code`, `bu_spec_code`, `bu_size_code`, `bu_paper_code`, `bu_color_code`, `bu_binding_code`, `bu_finishing_code`, `bu_notes_code` | Template options the tool writes the file, preview, price, pages, sheets, job spec and each choice to | Option codes | the setting name without `_code`, e.g. `bu_file`, `bu_binding` | **No**: must match the options on your template; your fulfillment reads them |

There is no sides setting: a booklet is always double-sided.

The rate card (prices, papers, bindings, covers, volume ladder) is a JSON block in the same `bu_mount` field, written as `<script type="application/json" data-bu-rates>...</script>` above the snippet call. Edit it with the Booklet Settings editor in Custom Admin: paste the current mount, change the card, copy the whole field back. The editor stores nothing itself. If the JSON is not valid, pricing stops (it does not fall back). An older site may still read rates from the `product/booklet-uploader-rates` snippet; the tool logs that as legacy.

Things that are not set here
- Covers, papers and bindings offered, and their prices, are in the rate card, not the snippet arguments.

Not yet confirmed: `bu_full_width: 'false'`: the exact effect is not confirmed.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
