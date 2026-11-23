# Stationery Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.4.0 live on the Shopper parent since 19 Nov 2026.

As recorded on 3 Nov 2026: Status: live, version 0.3.3 (as of the CHANGELOG; the snippet header reads 0.3.0 and loads the 0.3.3 asset). Snippet: `product/stationery-studio`. Mount option: `ss_mount` (hidden Text template option, one per stationery template). Prefix: `ss_`.

*Not re-checked setting by setting on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

Notepads, notebooks and postcards. The shopper picks a design, adds photos or a logo, changes the words and cover color, picks binding, pages and quantity, approves a proof and adds it to the cart, or customizes further in the design tool. On notebooks, Make your own opens Creative Studio inside the same page.

Example mount (one per template; this one is an 8.5 x 11 notebook):
```liquid
{% snippet 'product/stationery-studio',
	ss_family: 'notebook',
	ss_size: '8.5x11',
	ss_dpi: '300',
	ss_min_dpi: '150',
	ss_collection: '',
	ss_editor: 'true',
	ss_step: '1',
	ss_money: '$1,234.50',
	ss_creative: 'true',
	ss_bleed: '0.125' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `ss_family` | Kind of stationery this template is | `notepad`, `notebook`, `postcard` | required: none | **No**: must match the template and its design codes |
| `ss_size` | Size of this template, as written in the design codes | `4x5`, `5x5`, `4x10`, `5.5x8.5`, `8.5x11`, `4x6-2s` | required: none | **No**: must match the template and its design codes |
| `ss_dpi` | Resolution of the template XML definition | A number, the dpi in the XML definition | required: none | **With care**: must equal the dpi in the template XML definition |
| `ss_min_dpi` | Photo low resolution warning below this | A number | `150` | **Yes** |
| `ss_collection` | Collection path holding the designs | A collection path, or blank to use the collection in the page URL | blank | **Yes**: must exist on your site |
| `ss_editor` | Offer Customize more in the design tool | `true`, `false` | `true` | **Yes** |
| `ss_step` | Quantity step, for example 4 for postcards sold in fours. The minimum is the product minimum | A number | `1` | **With care**: must match how your pricing sells the product |
| `ss_money` | Money format sample for prices shown in the studio | A formatted amount, for example `$1,234.50` | `$1,234.50` | **Yes**: match your store currency format |
| `ss_creative` | Notebooks only: add the Make your own card (Creative Studio) | `true`, `false` | `false` | **With care**: the collection must hold the design `pxt-notebook-in-<size>-creative` |
| `ss_bleed` | Bleed of the XML definition in inches, for the Make your own print file | A number in inches | `0.125` | **With care**: must equal the bleed in the template XML definition |

`ss_family`, `ss_size`, `ss_dpi`, `ss_min_dpi`, `ss_editor` and `ss_step` must be in every mount; one that is left out is listed in `data-config-missing` on `#ss-root`. Never put an apostrophe in a value or in the mount.

Things that are not set here:
- Designs: found in the collection by code `pxt-<family>-in-<size>-<design>`.
- Binding, pages, paper and their prices: the product variants and pricing.
- Cover colors: built into the designs.
- Print size and bleed: the template XML definition. `ss_dpi` and `ss_bleed` only repeat it for the studio.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
