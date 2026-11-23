# Puzzle Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.6.0 live on the Shopper parent since 5 Nov 2026.

As recorded on 3 Nov 2026: Status: live, version 0.2.0 on the platform; 0.4.0 built and not yet released (as of the CHANGELOG). Snippet: `product/puzzle-studio`. Mount option: `px_mount` (hidden Text template option on every puzzle template of the range). Prefix: `pz_`.

## Changes since 3 November 2026

- From 0.3.0 the cart picture is stored in a hidden File Upload template option `pz_preview`. Each puzzle template needs that option, or the cart shows a sideways or missing picture. Ask Pixfizz to add it.

*Not re-checked setting by setting on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper picks a puzzle size and shape, adds one photo or a collage of up to eight, chooses pieces and material, sees the finished puzzle in Live Finish, and adds it to the cart.

The same mount also calls `product/live-finish`, before the Puzzle Studio call. Those `lf_` settings are documented under Live Finish.

Example mount (Puzzle Studio part):
```liquid
{% snippet 'product/puzzle-studio',
	pz_items: '5x7=00000 8x10=00000 11x14=00000 16x20=00000 circle=00000 hexagon=00000',
	pz_collection: 'puzzles',
	pz_max_photos: '8',
	pz_min_dpi: '100',
	pz_material_notes: 'on',
	pz_pickup: 'Ready for pickup in store.',
	pz_range: 'Photo Puzzle',
	pz_units: 'in' %}
```
Replace each `00000` with the design id (or design code) of that puzzle on your site.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `pz_items` | The puzzle range: which sizes and shapes the studio offers, and the design behind each | `key=design id or design code`, space separated. The key names the size (`5x7`, `8x10`, `circle`, `hexagon`; with `pz_units: 'cm'` a paper name such as `A4`) | required: none. Missing is listed in `data-config-missing` | **With care**: ids must be your real puzzle designs. Removing an entry removes that size |
| `pz_collection` | Collection holding the range | A collection name | the collection each design is published in | **Yes**: must exist on your site |
| `pz_max_photos` | Most photos in one collage | `1` to `8` (circle and hexagon take at most 4) | `8` | **Yes** |
| `pz_min_dpi` | Low resolution warning below this | A number | `100` | **Yes** |
| `pz_material_notes` | One-line notes under each material | `on`, `off` | `on` | **Yes** |
| `pz_pickup` | One sentence under About, for example where to pick the order up | Text, no apostrophe | empty | **Yes** |
| `pz_range` | Word after the size in the title | Text | `Photo Puzzle` | **Yes** |
| `pz_units` | Size display unit. With `cm`, a paper size key (`A4`, `A3`) is used as the size name (0.4.0) | `in`, `cm` | `in` | **Yes**: match how your sizes are named |

Only `pz_items` is required; the other settings have defaults set in the snippet. Never put an apostrophe in a value or in the mount.

Things that are not set here:
- Product, trim, bleed, resolution, prices and production time: read live from each puzzle design and product.
- Pieces and materials: the template options `pieces` (values `<n>pieces`) and `puzzlematerial` on each size product.
- The finished-puzzle preview (looks, light, room): the `product/live-finish` call in the same mount.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
