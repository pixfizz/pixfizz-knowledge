# Pet Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: live, version 0.4.1 (as of the CHANGELOG; snippet 0.4.1, asset 0.4.0). Snippet: `product/pet-studio`. Mount option: `px_mount` (hidden Text template option, one per template). Prefix: `ps_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

Personalized pet products fulfilled by a print-on-demand supplier: bandanas (four sizes), dog and cat bowls, collars and tags. The shopper picks a design and breed, adds a photo and name, sees a 3D preview and adds it to the cart.

Example mount (a medium bandana template; design codes and ids are placeholders):
```liquid
{%- assign ps_xml_dpi = '' -%}{%- if template.dpi -%}{%- assign ps_xml_dpi = template.dpi -%}{%- endif -%}
{% snippet 'product/pet-studio',
	ps_product: 'bandana',
	ps_size: 'M',
	ps_items: 'S=BANDANA-S#00000 M=BANDANA-M#00000 L=BANDANA-L#00000 XL=BANDANA-XL#00000 dogbowl=dog-bowl#00000 catbowl=cat-bowl#00000 collar=dog-collar#00000 tag=circle-pet-tag#00000',
	ps_size_s: 'pet-bandana-small:7.25x4.125:0.39:7.8-11.8:0.339',
	ps_size_m: 'pet-bandana-medium:9.25x5.5:0.59:10.2-17.3:0.311',
	ps_size_l: 'pet-bandana-large:12.5x6.63:0.79:12.6-19.7:0.303',
	ps_size_xl: 'pet-bandana-xl:15x7.75:0.79::0.291',
	ps_spec_code: 'px_spec',
	ps_dpi: ps_xml_dpi,
	ps_trim: '0.25in',
	ps_safe: '0.5in',
	ps_min_dpi: '150',
	ps_title: 'Pet Bandana with Collar',
	ps_trust: 'bandana:Printed edge to edge on soft white fabric|tag:Glossy aluminum, printed on both sides' %}
```
The first line reads the dpi from the template XML definition, so `ps_dpi` always matches it.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `ps_product` | Which product this template is | `bandana`, `dogbowl`, `catbowl`, `collar`, `tag` | `bandana` | **No**: must match the template |
| `ps_items` | The whole range: the design behind every product and bandana size, so the shopper can switch between them | `key=design code#design id`, space separated. Keys `S`, `M`, `L`, `XL`, `dogbowl`, `catbowl`, `collar`, `tag` | blank: bandana sizes found by `url_path` | **With care**: codes and ids must be the designs on your site. Removing a key removes that product from the switcher |
| `ps_size` | Bandana size of this template | `S`, `M`, `L`, `XL` | required on bandana templates | **No**: must match the template |
| `ps_size_s`, `ps_size_m`, `ps_size_l`, `ps_size_xl` | Each bandana size: page url_path, print size, collar width, fit and band. Put all four on every template of the range | `url_path:WxH:collar:fit:band`, or blank when that size is not offered | blank: size not offered | **With care**: sizes must match the supplier spec and the templates |
| `ps_dpi` | Resolution of the template XML definition | A number. Pass `template.dpi` as in the example | required | **No**: leave it reading the template |
| `ps_trim` | Production buffer inside the page edge | A length, for example `0.25in` | required | **With care**: must match the supplier production spec |
| `ps_safe` | Safe line from the page edge | A length, for example `0.5in` | required | **With care**: must match the supplier production spec |
| `ps_collection` | Collection path holding the other sizes | A collection path | the collection in the page URL | **Yes**: must exist on your site |
| `ps_min_dpi` | Photo low resolution warning below this | A number | `150` | **Yes** |
| `ps_spec_code` | Template option that receives the job record for fulfillment | A template option code | `px_spec` | **No**: must match the option on the template |
| `ps_title` | Heading shown above the studio | Text | the page heading | **Yes** |
| `ps_maker` | Made by line in the product details | Text | empty: line hidden | **Yes** |
| `ps_trust` | Short reassurance lines. A line can name its product (`bandana:...`); unnamed lines show on every product | Lines separated by a vertical bar | empty | **Yes** |
| `ps_money` | Money format sample for prices shown in the studio | A formatted amount, for example `$1,234.50` | `$1,234.50` | **Yes**: match your store currency format |
| `ps_fulfiller` | Supplier name in the strip under the header | Text, or `none` to hide the strip | `Navitor` | **Yes** |
| `ps_fulfiller_logo` | Supplier logo in that strip | Asset name of an image in your site assets, or `none` to show the name as text | `pod-logo-navitor.webp` | **Yes** |
| `ps_range` | Chip next to Pet Studio in the strip | Text | `Bandana` | **Yes** |
| `ps_display` | Display mode | `inline` | `inline` | **No**: read but has no other mode in this version |

`ps_size` (bandana templates), `ps_dpi`, `ps_trim` and `ps_safe` must be in every mount; one that is left out is listed in `data-config-missing` on `#ps-root`. Never put an apostrophe in a value or in the mount. Lengths are written in inches.

Things that are not set here:
- Print size and bleed of each product: the template XML definition (supplier spec).
- Prices: the products and their pricing.
- Designs and breeds: the design collection and the tool itself.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
