# Publication Upload: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
1.16.0 released 4 Nov 2026 (upload standard).

As recorded on 3 Nov 2026: Status: live on publishing-program sites, version 1.15.1 (per the CHANGELOG). Snippet: `product/publication-upload`. Mount option: `pu_mount` from 1.15.0 (older installs: the `custom_script` on the `pu_text` option). Prefix: `pu_`.

*Not re-checked setting by setting on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

For publishers who reprint their own titles: the shopper uploads the book block (and optionally a new cover) for a title, the tool checks page count, trim size, bleed, resolution and ISBN against that title, and holds the order on a hard fail.

Example mount (one template holds many titles, so the per-title values are read from design custom fields):
```liquid
{%- assign pu_asset = design.custom.pu_file -%}
{% snippet 'product/publication-upload',
	pu_isbn: design.custom.pu_isbn,
	pu_file: pu_asset | asset_url,
	pu_file_name: pu_asset,
	pu_pages: design.custom.pu_pages,
	pu_trim_width: design.custom.pu_trim_width,
	pu_trim_height: design.custom.pu_trim_height,
	pu_cover_mode: 'separate',
	pu_bleed: '3',
	pu_min_dpi: '300',
	pu_soft_dpi_floor: '150',
	pu_max_file_mb: '150',
	pu_block_on_fail: 'true',
	pu_partner: '',
	pu_generic: 'false',
	pu_upload_required: 'true'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `pu_pages` | Exact page count for the title. Blank turns the page count check off | A number, or blank | blank | **With care**: must match the title; this is the tool's only hard fail |
| `pu_trim_width`, `pu_trim_height` | Trim size of the title. Both blank turns the size check off | Numbers in mm, or blank | blank | **With care**: must match the title and your production |
| `pu_cover_mode` | Cover supplied as separate front and back, or as one spread | `separate` or `spread` | `separate` | **With care**: match how your production expects covers |
| `pu_bleed` | Expected bleed | A number in mm | `3` | **With care**: match how your artwork is built |
| `pu_min_dpi` | Images below this are flagged | A number | `300` | **With care** |
| `pu_soft_dpi_floor` | Images below this are always flagged | A number | `150` | **With care** |
| `pu_max_file_mb` | Largest file a shopper can upload | A number in MB, at most `150` (the platform limit) | `250` in 1.15.1 (being lowered to 150); set `150` | **Yes**, up to 150 |
| `pu_block_on_fail` | Holds Add to Order on a hard fail | `true` or `false` | `true` | **Yes** |
| `pu_upload_required` | `false` offers the uploader but never blocks the order | `true` or `false` | `true` | **Yes** |
| `pu_partner` | Production partner named in the shopper-facing text. Blank names nobody | Text, no apostrophes, or blank | blank | **Yes** |
| `pu_generic` | Marks the title as deliberately unspecified (reads differently from an unfinished setup) | `true` or `false` | `false` | **Yes** |
| `pu_isbn` | The title ISBN, shown in the header and matched against the uploaded file | An ISBN | blank (falls back to the ISBN in the product code) | **With care**: must be the title ISBN |
| `pu_sku` | Production SKU for the title | Text | the product code | **No**: leave unset unless your production SKU differs from the product code |
| `pu_file`, `pu_file_name` | The contents PDF already on file for the title, and the name shown for it. Blank means the shopper uploads | An asset URL (`{{ 'name.pdf' \| asset_url }}`) and a file name | blank | **With care**: must point at the right title file |
| `pu_cover_file`, `pu_cover_file_name` | The cover PDF already on file, and the name shown for it | An asset URL and a file name | blank (or the product's stored cover) | **With care** |
| `pu_cover_required` | Requires a cover file as well as the book block | `true` or `false` | `false` | **Yes** |
| `pu_cart_image` | Image shown in the cart, checkout and emails: page 1 of the upload, or the title's catalog image | `content` or `product` | `content` | **Yes** |
| `pu_support_email` | Address a shopper is pointed to when a file carries an ISBN no title has | An email address | the first email link on the page | **Yes** |
| `pu_text_code`, `pu_cover_code`, `pu_preview_code`, `pu_preflight_code`, `pu_state_code`, `pu_changes_code` | Template options the tool writes to | Option codes | the tool's standard `pu_` codes | **No**: pass only if your template renamed one |

Pass every setting every time; any left out is named in the browser console. Trim and bleed are in millimeters.

Things that are not set here
- Per-title values (ISBN, page count, trim, file) live in design custom fields and are passed in by the mount; those field definitions must exist on your site.
- A stored cover can also come from the product custom field `px_stored_cover`, which the edition roll-forward maintains.
- Older installs may still read some settings from the Publication Upload checklist keys; the mount wins.

Not yet confirmed: The built-in defaults of `pu_text_code` and the other code settings are not confirmed; keep the values your install came with.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
