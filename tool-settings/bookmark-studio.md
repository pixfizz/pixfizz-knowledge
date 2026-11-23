# Bookmark Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: live, version 0.2.1 (as of the CHANGELOG). Snippet: `product/bookmark-studio`. Mount option: `bs_mount` (hidden Text template option, custom_script field). Prefix: `bs_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

A live preview of a metal bookmark: close-up, in a book, at actual size, the back, and a hanging tassel that follows the tassel option. It writes nothing to the order.

Example mount:
```liquid
{% snippet 'product/bookmark-studio',
	bs_source: 'design',
	bs_place: 'gallery',
	bs_page: 'bookmark',
	bs_size: '1.5x6.63in',
	bs_product_size: '1.375x6.5x0.03in',
	bs_finish: 'aluminum-gloss-silver-back',
	bs_option_selector: 'auto',
	bs_tassel_options: 'tassel',
	bs_tassel_colors: 'no tassel=none|gold=gold|black=black|cream=cream|white=white',
	bs_tassel_assets: 'gold=bs-tassel-gold.webp|black=bs-tassel-black.webp|cream=bs-tassel-cream.webp|white=bs-tassel-white.webp',
	bs_scene: 'bs-scene-book.webp',
	bs_views: 'close,book,actual',
	bs_start: 'close',
	bs_labels: 'Close-up|In a book|Actual size|Drag to tilt and catch the shine|Tilt with your phone|Close to real size on most screens|See the back|See the front' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `bs_source` | What is previewed: the design on the product page, or a saved project | `design`, `project` | none, reported | **No**: `design` on a product page mount |
| `bs_place` | Where the preview shows | `gallery` (over the first gallery image), `inline` (where the snippet renders) | `gallery` for a design, `inline` for a project | **Yes** |
| `bs_page` | The page rendered: a page name for a design, a 1-based page number for a project | page name or number | none, reported | **With care**: must name the page in the XML template |
| `bs_size` | Page size including bleed | `WxHin`, for example `1.5x6.63in` | none, reported | **With care**: must match the XML template page |
| `bs_product_size` | Finished bookmark size and thickness | `WxHxDin`, for example `1.375x6.5x0.03in` | none, reported | **With care**: must match the real product |
| `bs_finish` | Surface and back. Words in the value set the look: `matte`, `satin` or `semi`, otherwise gloss; `metal` or `alumin` draws metal edges; `silver` or `white-back` sets the back | text such as `aluminum-gloss-silver-back` | `metal-gloss` | **Yes**: match the real product |
| `bs_option_selector` | How the tassel option is found on the page | `auto`, or an element id | none, reported | **Yes**: leave `auto` |
| `bs_tassel_options` | The tassel option code | option code | none, reported | **No**: must match the option code on the template |
| `bs_tassel_colors` | Tassel color per option value | `value=color` pairs separated by `|`; `none` for no tassel | none, reported | **With care**: values must match your option values |
| `bs_tassel_assets` | Tassel image per color | `color=asset` pairs separated by `|` | none, reported | **With care**: each image must be uploaded to the site |
| `bs_scene` | Background photo for the in a book view | asset name | none, reported | **Yes**: upload your own asset first |
| `bs_views` | Views offered, in order | `close`, `book`, `actual` (comma list) | `close,book,actual` | **Yes** |
| `bs_start` | View shown first | `close`, `book`, `actual` | `close` | **Yes** |
| `bs_labels` | The 8 interface labels, in this order: close-up, in a book, actual size, drag hint, phone tilt hint, actual size note, see the back, see the front | 8 texts separated by `|` | none, reported | **Yes**: keep all 8 in order |
| `bs_project_id`, `bs_design_id`, `bs_product_id`, `bs_timestamp` | Identify the saved project on an edit page. Project source only | Liquid values from the page | blank | **No**: set by the page, not by hand |

Things that are not set here
- Production. The platform renders and fulfills from the template as before.

Not yet confirmed: The meaning of each of the 8 labels is confirmed only by their order in the example.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
