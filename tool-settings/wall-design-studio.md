# Wall Design Studio (metal and acrylic walls): tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.8.9 live on the Shopper parent since 9 Nov 2026.

As recorded on 3 Nov 2026: Status: live, version 0.8.7 (snippet 0.8.5). Snippet: `product/wall-design-studio`. Mount: a Pages page (usually `/site/wall-design-studio`) whose content calls the snippet; it can also sit in the `custom_script` of a template option. Prefix: `wds_`.

*Not re-checked setting by setting on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper picks one of 12 ready-made wall layouts (2 to 9 panels), a material and size, drags photos into the panels and sees the wall in a room. The studio adds one cart line per panel on your own single metal or acrylic products, plus a hang guide. Not the same tool as TilePix Wall Studio (`product/wall-designer`).

How values are written: plain text values as written; structured values (lists, prices, product maps, layouts, backings) as JSON with double quotes. Images are asset names on your site.

Example mount (shape only: product ids, paths and codes come from your own catalog):
```liquid
{% snippet 'product/wall-design-studio',
	wds_photo_option: 'photo',
	wds_hang_option: 'px_print_hang',
	wds_products: '{"metal":{"12x18":101,"16x24":102},"acrylic":{"12x18":201}}',
	wds_paths: '{"metal":"/site/shop/metal-prints","acrylic":"/site/shop/acrylic-prints"}',
	wds_mount: '{"metal":{"code":"mounting","value":"float","label":"Float mount"}}',
	wds_discount: '{"label":"Wall discount","tiers":{"2":5,"3":10,"4":15,"5":20}}',
	wds_dpi: '{"min":100,"good":150}',
	wds_room_sofa: 'room-sofa.webp',
	wds_room_bedroom: 'room-bedroom.webp',
	wds_room_hallway: 'room-hallway.webp',
	wds_samples: 'Beach:sample-1.webp|Forest:sample-2.webp' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `wds_photo_option` | Image template option on each panel product that receives the photo | option code | none, required (the tool falls back to `photo`) | No, unless your panel products use another code |
| `wds_hang_option` | File option on panel 1 that receives the hang guide PDF | option code | `px_print_hang` | No, unless renamed |
| `wds_products` | Which of your products is each panel size, per material | JSON: material to size to product id | none, required unless `wds_demo` is `true` | With care: every id must be a live product of that material and size |
| `wds_paths` | Shop page (collection) per material where each size's add-to-cart form is found; can carry per-size query strings | JSON: material to collection path | none | With care: must resolve to the right product for every size |
| `wds_mount`, `wds_finish`, `wds_fixed` | Variant code and value the studio sets on every panel (mounting, finish, other fixed choices such as corners) | JSON per material, with `code` and `value` (and an optional `label`) | none | With care: codes and values must match your products' variant types exactly |
| `wds_discount` | Wall discount shown in the studio | JSON with `label` and `tiers` (panel count to percent) | none (no discount shown) | With care: tiers must equal your Automatic Discount, or the studio shows a price the cart does not charge |
| `wds_prices` | Panel prices used only when a size has no product id (demo sites) | JSON: material to size to price | none | Yes, on a demo site |
| `wds_materials` | Which materials are offered | JSON list, `metal` and/or `acrylic` | both | Yes |
| `wds_mat_text` | Replacement wording for a material | JSON per material | built-in wording | Yes |
| `wds_dpi` | Photo sharpness thresholds | JSON `{"min":..,"good":..}` | min 100, good 150 | Yes |
| `wds_hang` | Hanging point per material (inches down from the top edge) for the hang guide | JSON per material | none | With care: must match the hardware you ship |
| `wds_size_advice` | Size advice notes on the wall | `false` turns them off | on | Yes |
| `wds_ship_box` | Largest shipping box, short and long side in inches | JSON `[short, long]` | none | With care: must match your shipping |
| `wds_backings`, `wds_frame_colors`, `wds_hardware`, `wds_start_back` | Optional backing, frame color and hanging hardware choices (each can carry a per-panel price) and the backing selected first | JSON lists; `wds_start_back` is a backing id | none (not offered) | With care: prices must match your product pricing |
| `wds_layouts` | Restrict or reorder the built-in layouts | JSON list of layout ids | all 12 | Yes |
| `wds_auto` | Layout picked automatically for a given number of photos | JSON: photo count to layout id | built-in map | Yes |
| `wds_start_layout`, `wds_start_opt`, `wds_start_scene` | Layout, size option and room shown first | layout id; size option; `sofa`, `bedroom`, `hallway` | `tri`; first available; `sofa` | Yes |
| `wds_room_sofa`, `wds_room_bedroom`, `wds_room_hallway`, `wds_room_sofa_front` | Room images | asset names | `wds_room_sofa` required | Yes |
| `wds_samples` | Sample photos shown before the shopper adds their own | `Name:file.webp` pairs split by `|` | none | Yes |
| `wds_max_samples` | How many samples to use | number | `4` | Yes |
| `wds_title`, `wds_title_tool`, `wds_facts_line`, `wds_facts`, `wds_facts_tool`, `wds_product_name` | Headings and short fact lines | text | built-in wording ("Wall Design Studio") | Yes |
| `wds_gallery_id` | Customer gallery the upload window opens | gallery id | none | With care: must be a gallery on your site |
| `wds_upload_sources` | Upload sources offered | any of `local`, `galleries`, `qr`, space separated | `local galleries qr` | Yes |
| `wds_demo`, `wds_demo_note`, `wds_demo_done` | Demo mode with no real cart, and its messages | `true` / `false`; text | off | Yes, demo sites only |

Things that are not set here
- Panel prices: they are your own products' prices and variant surcharges. The wall discount is an Automatic Discount on your site.
- Brand colors and the wall texture: theme tokens (`--pxt-*`, `--wdx-gallery`) in your site CSS.
- Canvas and wood walls are not supported yet; metal and acrylic only.

Not yet confirmed: The inner field names of the JSON settings are not confirmed here; copy the shape from your installed mount.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
