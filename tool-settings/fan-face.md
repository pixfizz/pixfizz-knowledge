# Fan Face (face fan designer): tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
1.3.1 live on Hite Photo only (23 Nov 2026). Older versions on other installs. 2.0.0 built and tested, not released.

As recorded on 3 Nov 2026: Status: beta, version 1.2.0 (built, being rolled out to sites; older installs still use the pre-1.0 settings page, see below). Snippet: `product/face-fan-designer`. Mount option: `facefan_mount`. Prefix: `facefan_`.

## Changes since 3 November 2026

- `facefan_noun`, `facefan_noun_plural` (from 1.3.0): the word the tool uses for the product, for example `big head` and `big heads`. **Yes.**
- `facefan_custom_min`, `facefan_custom_max` (from 1.3.0): the smallest and largest custom sheet size the shopper may type (up to 36 in on CT Print). **With care**: the size must fit your press and pricing. Value format not confirmed: copy it from your installed mount.
- **New in 2.0.0, not released:** cut mode `auto` in `facefan_cut_modes` (for example `head:auto|oval:oval`, where `head` is a value of your cut variant) adds "Cut around the head": the platform AI removes the background round the person the shopper picks, and the cut follows them with a white edge. **AI is a switch, off by default**: it exists only when `auto` is listed, and it hides itself on a site where AI is off. AI tools need a signed-in shopper; the tool signs them in on the page.
- 2.0.0 settings for the `auto` cut, all optional, defaults shown: `facefan_edges` (`none:0|thin:0.125in|thick:0.25in`), `facefan_edge_default` (`thick`), `facefan_ring_colors` (outline swatches, `#D8272A|#1E4FA8|#F2B705|#1F8A4C|#0F0F0F`, blank = no outline), `facefan_ring_any` (`true` adds an any-color swatch), `facefan_ring_width` (`0.125in`), `facefan_ai_tool` (`remove_background`, **No**), `facefan_ai_max_px` (`3000`, **No**), `facefan_register_url` (`/site/register`), `facefan_forgot_url` (`/site/forgotten-password`). Edges, rings and URLs are **Yes**.

*Checked against docs/releases/1.3.1-unreleased and 2.0.0-unreleased on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper uploads a face photo, positions it in an oval (or a hand-cut shape) on a big-head fan sheet, and approves a proof. The tool writes the print file with a cut path for the cutter.

Example mount:
```liquid
{%- assign facefan_xml_dpi = '' -%}
{%- if template.dpi -%}{%- assign facefan_xml_dpi = template.dpi -%}{%- endif -%}
{% snippet 'product/face-fan-designer',
	facefan_display: 'inline',
	facefan_sizes: '8.5x11:8.5x11in|12x18:12x18in',
	facefan_size_variant: 'size',
	facefan_cut_variant: 'cut',
	facefan_cut_modes: 'oval:oval|head:none',
	facefan_upscale_variant: 'upscale',
	facefan_bleed: '0.125in',
	facefan_min_dpi: '150',
	facefan_good_dpi: facefan_xml_dpi,
	facefan_max_mb: '40',
	facefan_zoom_max: '1000',
	facefan_cut_spot: 'CutContour',
	facefan_cut_file: 'embedded',
	facefan_block_low_dpi: 'false',
	facefan_title: 'Design your face fan',
	facefan_trust: 'Printed and assembled in our shop|Ready in 2 business days' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `facefan_display` | Layout | `inline`, `modal` | `inline` | Yes |
| `facefan_sizes` | Sheet size for each size variant value; the print file is the sheet, the oval is the sheet less the bleed | `value:WxHin` pairs split by `|`, for example `8.5x11:8.5x11in|12x18:12x18in` | none, required | With care: values must match your size variant values, sizes must match the XML template |
| `facefan_size_variant` | Code of the size variant the tool keeps selected | variant code | none, required | No, unless you renamed it |
| `facefan_bleed` | Photo past the cut on every side | length with unit, for example `0.125in` | none, required | With care: must match your production spec |
| `facefan_min_dpi` | Below this the photo is called blurry | number, for example `150` | none, required | Yes |
| `facefan_good_dpi` | Print resolution and the sharp threshold; passed from the XML template dpi | `template.dpi` as in the example, or a number | none, required (falls back to 300 with a console error) | With care: must match the dpi in the template XML; keep the `template.dpi` lines |
| `facefan_max_mb` | Largest photo accepted | number, MB | none, required | Yes |
| `facefan_zoom_max` | Highest zoom, percent (100 = the photo just fills the oval) | number | none, required | Yes |
| `facefan_cut_spot` | Spot color name of the cut path | text, for example `CutContour` | none, required | With care: must be the name your cutter or RIP expects |
| `facefan_cut_file` | Where the cut path goes | `embedded` (inside the print file), `separate` (its own file on `px_print_cut`) | none, required | With care: must match how your cutter takes files |
| `facefan_block_low_dpi` | Whether a blurry photo can still be approved | `true` (cannot approve), `false` (warning only) | none, required | Yes |
| `facefan_cut_variant` | Code of the cut variant (oval or hand cut) | variant code, or blank | blank (oval only) | No, unless renamed |
| `facefan_cut_modes` | What each cut value means | `value:oval` or `value:none` pairs split by `|`; `none` = no cut path, cut by hand | blank | With care: values must match your cut variant values |
| `facefan_upscale_variant` | Code of the upscale variant offered when the photo is under `facefan_min_dpi` | variant code, or blank | blank (not offered) | No, unless renamed |
| `facefan_upscale_yes`, `facefan_upscale_no` | Value codes of the upscale variant | value code | `yes`, `no` | No, unless renamed |
| `facefan_title`, `facefan_intro` | Heading and intro line | text | blank | Yes |
| `facefan_trust` | Short reassurance lines under the tool | lines split by `|` | blank | Yes |
| `facefan_print_code`, `facefan_cut_code`, `facefan_preview_code`, `facefan_original_code`, `facefan_place_code`, `facefan_spec_code` | Template options the files and job record are written to | option code | `px_print_file`, `px_print_cut`, `px_preview`, `facefan_original`, `facefan_place`, `px_spec` | No, unless renamed |
| `facefan_trim_w`, `facefan_trim_h` | Older single-size form (1.0.0 mounts) | length with unit | blank | No; use `facefan_sizes` on new mounts |

Things that are not set here
- Prices: product and variant pricing.
- Installs older than 1.0.0 do not use a mount: they read product custom fields `facefan_*` and a "Fan Face Settings" checklist under Custom Admin. Those settings stop being read once the site moves to 1.2.0.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
