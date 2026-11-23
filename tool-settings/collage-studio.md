# Collage Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: beta, version 1.1.0 built 3 Nov 2026 (as of the release notes; the repo marks every release folder unreleased, so live status is not verified). Snippet: `product/collage-studio`. Mount: not a template option. The studio sits on its own Pages page, whose page content is only the snippet tag. Prefix: `col_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

One collage poster product instead of one per size and photo count. The shopper adds photos first, the studio generates a layout for them, and the shopper picks size, material, background and title before adding it to the cart.

Example mount (page content of the studio page):
```liquid
{% snippet 'product/collage-studio',
	col_units: 'in',
	col_materials: 'none',
	col_products: '8x10:00000,11x14:00000,16x20:00000,20x30:00000',
	col_collection: 'collage-studio',
	col_design: 'col-blank',
	col_finish_variant: 'finish',
	col_mount_variant: 'none',
	col_min_cell: '2.5in',
	col_min_dpi: '150',
	col_max_photos: '60',
	col_layouts: 'fit,rows,grid,hero,instant,film,heart',
	col_backgrounds: 'white,ivory,kraft,sage,navy,black,custom',
	col_room: 'sofa',
	col_fonts: 'serif:Fraunces Regular,script:Caveat Medium,sans:DM Sans SemiBold,line:DM Sans Medium',
	col_login_url: '/site/login',
	col_exit_url: '/',
	col_exit_label: 'Shop' %}
```
Replace each `00000` with the product id of that size on your site.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `col_units` | How sizes and lengths are shown, and the unit of a size written without one | `in`, `mm` | `in` | **Yes** |
| `col_materials` | Materials offered, in order; the first is the default. With more than one, the shopper gets a Material step | `none`, or `<code>:<label>[:<kind>[ <depth>]]`, comma separated. Kind `print`, `canvas`, `metal` or `acrylic` (for example `poster:Poster,cv20:Canvas 2 cm:canvas 20mm`) | `none` | **With care**: every material needs its own size products in `col_products` |
| `col_products` | The size products the collage can be ordered on | `[material/]size:product id` pairs, comma separated. A size is `<w>x<h>` with an optional unit (`in`, `mm`, `cm`) or a name such as `A3`, sized by its template. With materials, every entry starts with its material code | none: no sizes | **With care**: ids must be your real products. The template definition always wins over the size you write |
| `col_collection` | Design collection(s) holding the blank collage designs | Collection name(s), comma separated | `collage-studio` | **With care**: must name collections on your site holding those designs |
| `col_design` | Design code suffix. The studio uses `<template code>-<col_design>` on each product | A code suffix | `col-blank` | **No**: must match your blank design codes |
| `col_finish_variant` | Variant type code(s) of the paper or finish on the size products | A code, several codes comma separated (each product uses the first it has), or `none` | `none` | **No**: must match your variant type codes |
| `col_mount_variant` | Variant type code(s) of the mounting | A code, several codes, or `none` | `none` | **No**: must match your variant type codes |
| `col_min_cell` | Smallest a photo may be printed, on its short side | A length, `in` or `mm` (also `cm`), for example `2.5in` or `60mm` | `2.5in` | **Yes** |
| `col_min_dpi` | Below this, a photo is flagged as low resolution | A number, minimum 50 | `150` | **Yes** |
| `col_max_photos` | Most photos in one collage | A number from 2 to 120 | `60` | **Yes** |
| `col_layouts` | Layouts offered, in this order | Any of `fit`, `rows`, `grid`, `hero`, `instant`, `film`, `heart`, comma separated | all seven | **Yes** |
| `col_backgrounds` | Background swatches offered | Any of `white`, `ivory`, `kraft`, `sage`, `navy`, `black`, `custom`, comma separated | all seven | **Yes** |
| `col_room` | Room scene for the On the wall preview | `sofa`, or `none` | `sofa` | **Yes** |
| `col_fonts` | Title fonts, by site font name | `serif:<name>,script:<name>,sans:<name>,line:<name>` | `serif:Fraunces Regular,script:Caveat Medium,sans:DM Sans SemiBold,line:DM Sans Medium` | **With care**: each name must be a font available on your site, or the print will not match |
| `col_login_url` | Sign-in page used by the upload window | A site path | `/site/login` | **Yes** |
| `col_exit_url` | Where the breadcrumb goes | A site path | `/` | **Yes** |
| `col_exit_label` | Breadcrumb text | Text | `Shop` | **Yes** |

Every setting should be in the mount. One that is left out is listed in `data-config-missing` on `#col-root` and in a console error, and the default is used. A value the studio does not recognize is reported in the console and left out. Never put an apostrophe in a value. Lengths accept in and mm.

Things that are not set here:
- Print size, bleed and unit: the template XML definition of each size product.
- Finish and mounting names and prices, and collage prices: the product variants and pricing.
- `?material=<code>` on the page link opens the studio on that material; that is a link, not a mount setting.

Known issues
- **The studio can be wider than header and footer on some themes.** The studio frame uses a fixed 1400 px maximum width with 16 px padding at every width. On a theme whose `.container` follows the Bootstrap breakpoints (540 / 720 / 960 / 1400 px), the studio is up to 24 px wider than the header and footer between 576 and 1199 px. Site fix used, in the studio's Pages page: `@media (min-width:576px){body .col-root .wrap{max-width:540px}}`, and the same with 720, 960 and 1400 px at 768, 992 and 1200 px. A tool fix (size the frame from the site's `.container`) is wanted. Tool-level. *Verified by query on a lab site, 2026-11-04.*
- **Hide the studio's design collection as soon as you create it.** A new collection (for example the one holding the studio's blank designs) shows at once as a card in collection grids. Set the collection's `unpublished` custom field right after creating it; the studio still reads the collection through `/v1/theme_categories`. Template-level (Shopper). *Verified by query on a lab site, 2026-11-04.*

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
