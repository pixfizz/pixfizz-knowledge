# TilePix Wall Studio (Wall Designer): tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.6.12 live on the Shopper parent since 9 Nov 2026. Asset-only releases: mounts unchanged.

As recorded on 3 Nov 2026: Status: live, version 0.6.10 asset (snippet 0.6.6; any 0.6.x mount runs on the current asset). Snippet: `product/wall-designer`. Mount option: `wd_mount`. Prefix: `wd_`.

*Not re-checked setting by setting on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper builds a wall of up to 16 pieces (TilePix classic tiles, glass prints, skateboard decks or a mix), picks a layout and spacing, adds photos (or one photo across the whole wall) and sees it in a room before ordering. The tool writes one print file per piece plus a hang guide.

Example mount:
```liquid
{% snippet 'product/wall-designer',
	wd_version: '0.6.3',
	wd_display: 'inline',
	wd_max_pieces: '16',
	wd_families: 'classic,glass,skate,mix',
	wd_gaps: '1,2,3',
	wd_default_gap: '2',
	wd_min_dpi: '100',
	wd_good_dpi: '150',
	wd_output: 'jpeg',
	wd_ppi: '300',
	wd_glass_pad: '12-pad',
	wd_code_classic_black: 'TP_8x8_BLK',
	wd_code_classic_white: 'TP_8x8_WHT',
	wd_code_classic_gray: '',
	wd_code_g825: 'TP_GLASS_8x8',
	wd_code_g1114: 'GLASS_11x14',
	wd_code_g1616: 'GLASS_16x16',
	wd_code_g1624: 'GLASS_16x24',
	wd_code_skate: 'DTC_SBD_Skateboard',
	wd_family_code: 'wd_family',
	wd_layout_code: 'wd_layout',
	wd_frame_code: 'wd_frame',
	wd_gap_code: 'wd_gap',
	wd_qty_prefix: 'wd_qty_',
	wd_print_code: 'px_print_file',
	wd_print_prefix: 'px_print_p',
	wd_preview_code: 'px_preview',
	wd_spec_code: 'px_spec',
	wd_hang_code: 'wd_hang_guide',
	wd_src_code: 'wd_src_photo',
	wd_requires_design: 'true',
	wd_title: 'Wall Studio',
	wd_intro: 'Peel, stick, hang. No nails, no holes.' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `wd_version` | Mount version, compared with the tool version | version number, for example `0.6.3` | none, required | No |
| `wd_display` | Layout | `inline` (the tool owns the product page) | none, required | No |
| `wd_max_pieces` | Most pieces on one wall | number up to `16` (one print file option per piece) | none, required | With care: the template needs a print file option for every piece |
| `wd_families` | Which product families the shopper can pick | any of `classic`, `glass`, `skate`, `mix`, comma separated | none, required | Yes, within what your template's variants support |
| `wd_gaps` | Spacing choices between pieces, inches | any of `1`, `2`, `3`, comma separated | none, required | Yes |
| `wd_default_gap` | Spacing on the first view, inches | one of the `wd_gaps` values | none, required | Yes |
| `wd_min_dpi` | Below this a piece is flagged as printing soft | number | none, required | Yes |
| `wd_good_dpi` | At or above this a piece is called sharp | number | none, required | Yes |
| `wd_output` | Print file format | `jpeg` | none, required | No |
| `wd_ppi` | Print file resolution | number, for example `300` | none, required | With care: must match the supplier's file spec |
| `wd_glass_pad` | Glass mounting pad written to the job record | supplier pad code, for example `12-pad` | none, required | With care: must match the supplier spec |
| `wd_code_classic_black`, `wd_code_classic_white`, `wd_code_classic_gray` | Supplier item code for each classic tile frame color; blank means not offered | supplier item code, or blank | blank (not offered) | With care: must be the supplier's exact item code; blank switches that frame off |
| `wd_code_g825`, `wd_code_g1114`, `wd_code_g1616`, `wd_code_g1624` | Supplier item code for each glass size; blank means not offered | supplier item code, or blank | blank (not offered) | With care: as above |
| `wd_code_skate` | Supplier item code for skateboard decks; blank means not offered | supplier item code, or blank | blank (not offered) | With care: as above |
| `wd_family_code`, `wd_layout_code`, `wd_frame_code`, `wd_gap_code` | Variant codes for family, layout, frame and spacing | variant code | none, required (standard: `wd_family`, `wd_layout`, `wd_frame`, `wd_gap`) | No, unless you renamed those variants |
| `wd_qty_prefix` | Prefix of the per-family count variants | text, standard `wd_qty_` | none, required | No, unless renamed |
| `wd_print_code` | First print file option | option code, standard `px_print_file` | none, required | No, unless renamed |
| `wd_print_prefix` | Prefix of print file options 2 to 16 | text, standard `px_print_p` (gives `px_print_p02` and so on) | none, required | No, unless renamed |
| `wd_preview_code` | Cart preview option | option code, standard `px_preview` | none, required | No, unless renamed |
| `wd_spec_code` | Job record option | option code, standard `px_spec` | none, required | No, unless renamed |
| `wd_hang_code` | Hang guide PDF option | option code, standard `wd_hang_guide` | `wd_hang_guide` (warns if missing) | No, unless renamed |
| `wd_src_code` | Original photo option when one photo spans the wall | option code, standard `wd_src_photo` | `wd_src_photo` (warns if missing) | No, unless renamed |
| `wd_requires_design` | Add to cart only through the tool | `true`, `false` | none, required | With care: `false` lets an empty wall reach the cart |
| `wd_title` | Heading above the tool | text | blank | Yes |
| `wd_intro` | Short intro line | text | blank | Yes |
| `wd_set` | Scene and sample set | `home`, `business` | `home` (blank) | Yes |
| `wd_font_heading`, `wd_font_body` | Font stacks for a studio look | CSS font stack | blank (your site theme fonts) | Yes; leave out to keep your theme fonts |
| `wd_font_css` | Stylesheet that loads those fonts | `https://fonts.googleapis.com/...` URL only | blank | Yes |
| `wd_studio`, `wd_studio_tag` | Studio bar label and small tag above the tool | text | blank (no bar) | Yes |
| `wd_by_logo`, `wd_by_name` | "Fulfilled by" supplier logo and name | image URL; text | blank (text only) | Yes |
| `wd_price_label` | Label for the page price on catalog or demo sites where the page price is what the lab pays | text, for example `Lab price` | blank (no lab pricing lines) | Yes; leave out on a retail site |
| `wd_retail_classic`, `wd_retail_g825`, `wd_retail_g1114`, `wd_retail_g1616`, `wd_retail_g1624`, `wd_retail_skate` | Suggested retail per piece shown with the lab markup (only with `wd_price_label`) | number, for example `19.99` | blank | Yes |

Things that are not set here
- Prices: they come from the product's variants and pricing, not the mount.
- Which families, layouts and frames exist as choices: those are variant values on the product; the mount only switches them on or off.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
