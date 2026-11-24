# T-shirt Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.2.3, built 23 Nov 2026, not yet installed on a customer site. Snippet: `product/tshirt-studio` (product page) and `product/tshirt-studio-page` (a studio page). Mount option: `ts_mount`. Prefix: `ts_`.

The shopper designs a shirt on a 3D garment: front and back placements, photos, school or club art, a back name and number, sizes and colors. One garment per template and product.

Example mount (Gildan 980 tee):
```liquid
{% snippet 'product/tshirt-studio',
	ts_version: '0.1.0',
	ts_garment: 'tee',
	ts_garment_name: 'T-shirt, Gildan 980',
	ts_photo_prefix: 'ts-g980-',
	ts_board_w: '12in',
	ts_board_h: '18in',
	ts_dpi: '300',
	ts_min_dpi: '150',
	ts_place_left_chest: '8,0.5,3.5,3.5',
	ts_place_right_chest: '0.5,0.5,3.5,3.5',
	ts_place_large_front: '0.5,0.5,11,14',
	ts_place_back_tag: '1.75,0.5,8.5,3.25',
	ts_place_back_medium: '1.75,2.75,8.5,9',
	ts_place_back_large: '1.5,2.75,9,13',
	ts_place_back_name: '0.5,0.5,11,3',
	ts_place_back_number: '1,4,10,12',
	ts_pack_places: 'left-chest,right-chest,back-tag,back-medium',
	ts_font: 'Graduate',
	ts_name_max: '12',
	ts_number_max: '2',
	ts_money: '$1,234.50' %}
```

| Setting | What it controls | Accepted values | Default if missing | Can you change it? |
|---|---|---|---|---|
| `ts_garment` | The garment of this template | for example `tee` | `tee` (logged) | **No** |
| `ts_garment_name` | The garment as the shopper reads it | text | `T-shirt` (logged) | **Yes** |
| `ts_photo_prefix` | Garment photo asset prefix on the parent | for example `ts-g980-` | none, required | **No** |
| `ts_board_w`, `ts_board_h` | Print artboard size | length with unit, for example `12in`, `18in` | none, required | **With care**: must match your printer's platen |
| `ts_dpi` | Resolution of the print PNG | number | `300` (logged) | **With care** |
| `ts_min_dpi` | Below this a photo gets a low resolution warning | number | `150` (logged) | **Yes** |
| `ts_place_left_chest`, `ts_place_right_chest`, `ts_place_large_front`, `ts_place_back_tag`, `ts_place_back_medium`, `ts_place_back_large`, `ts_place_back_name`, `ts_place_back_number` | Print box of each placement | `x,y,w,h` in inches from the artboard top left | none, required | **With care**: must match where your press prints |
| `ts_pack_places` | Placements school or club pack art may print on | placement names, comma separated | `left-chest,right-chest,back-tag,back-medium` (logged) | **Yes** |
| `ts_font` | First lettering choice for the back name and number | a Google font name | `Graduate` (logged) | **Yes** |
| `ts_fonts` | All lettering choices | Google font names, comma separated | `ts_font`, then Graduate, Bebas Neue, Oswald, Alfa Slab One, Pacifico | **Yes** |
| `ts_name_max` | Longest back name | number | `12` (logged) | **Yes** |
| `ts_number_max` | Longest back number | number | `2` (logged) | **Yes** |
| `ts_money` | Money format sample | for example `$1,234.50` | `$1,234.50` (logged) | **Yes** |
| `ts_model` | 3D model | a `p3d-model-<name>.js` asset on the parent | `tee-classic` | **No** |
| `ts_three` | three.js library URL | URL | cdnjs three.js r128 | **No** |
| `ts_version` | The mount version, for your reference | text | blank | **Yes**, informational |

Studio page: a Pages page whose content is only `{% snippet 'product/tshirt-studio-page', ts_product: '<product id>', ts_theme: '<design id>' %}` (optional `ts_collection`). It reads the mount from the product page, so the mount is never repeated there.

What the tool writes (do not rename): template options `px_print_file`, `px_print_back`, `px_preview`, `px_spec`, `ts_photo`, and variants `size`, `ShirtColor`, `ts_sides`, `ts_front`, `ts_back`, `ts_back_name`, `ts_back_number`, `ts_design`. One cart line per size and per name. Prices come from your variants.

*Verified by reading `custom-design-tools/tshirt-studio/src/product__tshirt-studio.liquid`, `product__tshirt-studio-page.liquid` and `config/MOUNT-gildan-980.liquid`, 24 Nov 2026. Not yet verified on a customer site.*

## Changelog

- 2026-11-24: Page created.
