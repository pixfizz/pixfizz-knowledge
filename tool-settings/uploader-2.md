# Uploader 2.0 (documents, booklets, forms, flat prints): tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: live, version 2.6.0 (per the release notes; this folder has no CHANGELOG). Replaces Document Uploader and Booklet Uploader on new products. Snippet: `product/uploader`. Mount option: `px_mount`. Prefix: `up_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper uploads a PDF, chooses size, printing, paper, finishing (and binding, cover or parts, depending on the mode), sees the price and adds it to the cart. One asset with four modes.

Example mount (the rate card block sits above the snippet call in the same field):
```liquid
<script type="application/json" data-px-rates>{ ...your rate card, on one line... }</script>
{% snippet 'product/uploader',
	up_mode: 'booklet',
	up_size: '',
	up_bleed: '0.125',
	up_crop_marks: 'false',
	up_max_mb: '100',
	up_min_dpi: '150',
	up_full_width: 'true',
	up_single_cart: 'true',
	up_write_preview: 'true',
	up_max_qty: '5000' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `up_mode` | What the product is | `flat` (documents, copies, flyers), `booklet` (saddle-stitched booklets), `forms` (carbonless NCR forms), `prints` (flat prints imposed on a press sheet) | `flat` | **With care**: must match the rate card in the same mount; a card for another mode stops pricing |
| `up_size` | Flat mode only: the one size this product sells | A size code from your rate card, or blank | blank (offers every priced size as a picker) | **With care**: must match a size code in your rate card and the template |
| `up_bleed` | Bleed added on every page of the print file | A number in inches | `0.125` | **With care**: match your production |
| `up_crop_marks` | Adds crop marks to the print file | `true` or `false` | `false` | **With care** |
| `up_max_mb` | Largest file a shopper can upload | A number in MB | `100` | **Yes** |
| `up_min_dpi` | Minimum resolution for images inside the PDF | A number | `150` | **With care**: your print quality bar |
| `up_full_width` | Tool uses the full page width | `true` or `false` | `true` | **Yes** |
| `up_single_cart` | Hides the theme Add to Cart so only the tool button shows | `true` or `false` | `true` | **Yes** |
| `up_write_preview` | Saves a preview image for the cart and order | `true` or `false` | `true` | **Yes** |
| `up_max_qty` | Highest quantity a shopper can order | A number | `5000` | **Yes** |
| `up_print_file_code`, `up_print_cover_code`, `up_original_code`, `up_original_cover_code`, `up_preview_code`, `up_spec_code` | Template options for the print file, cover print file, original uploads, preview and job spec | Option codes | `px_print_file`, `px_print_cover`, `px_original`, `px_original_cover`, `px_preview`, `px_spec` | **No**: your fulfillment reads these. Pass only if your template renamed one |
| `up_price_code`, `up_pages_code`, `up_sheets_code`, `up_size_code`, `up_paper_code`, `up_color_code`, `up_sides_code`, `up_finishing_code`, `up_binding_code`, `up_cover_src_code`, `up_cover_print_code`, `up_cover_paper_code`, `up_blanks_code`, `up_fit_code`, `up_notes_code` | Variants (and the notes option) the shopper's choices and the price are written to | Variant or option codes | the `px_` standard: `px_price`, `px_pages`, `px_sheets`, `px_size`, `px_paper`, `px_color`, `px_sides`, `px_finishing`, `px_binding`, `px_cover_src`, `px_cover_print`, `px_cover_paper`, `px_blanks`, `px_fit`, `px_notes` | **No**: must match your product variants; your POS and OrderHub read them. Pass only if you renamed one |

All ten main settings should be passed every time; any left out is reported in the browser console. Bleed is a plain number in inches.

The rate card is a one-line JSON block, `<script type="application/json" data-px-rates>...</script>`, in the same `px_mount` field above the snippet call. It holds the sizes, papers, colors, finishing, bindings, covers or form parts, and every price. Change prices there, never in the snippet arguments. Starter cards for documents, letter booklets and NCR forms exist; ask Pixfizz for one. From 2.6.0 the tool reads the product custom field `px_rates` first and the mount card second, so a generic template can carry a zero-price card in its mount while each site prices on its own product. *Verified by reading source (2.6.0), 2026-11-02.*

**Linking to a size (flat mode, from 2.6.6).** The tool reads `?size=<size code>` on the page link (codes from the rate card `sizes[].code`, for example `85x11`, `85x14`, `11x17`; `8.5x11` is also accepted) and pre-selects that size, unless the mount fixes the size. The shopper can still change it, and an uploaded PDF of another size still wins. A size shop page whose cards all link to one multi-size uploader product adds `&size=<code>` to each card link; without it every card opens on the default size. An unknown code logs `[uploader] the link asks for size ...` in the console and the default size is used. This is a link, not a setting. Tool-level. *Verified live on a client site, 2026-11-17.*

Things that are not set here
- Every shopper choice and the price are product variants, so the order, POS and OrderHub can read them.
- Products still on Document Uploader (`du_`) or Booklet Uploader (`bu_`) keep their own settings until moved.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
