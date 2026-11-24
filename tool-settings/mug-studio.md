# Mug Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.2.4, built 19 Nov 2026, not yet installed on a site. Snippet: `product/mug-studio`. Mount option: `ms_mount`. Prefix: `ms_`.

Make your own mug: the shopper adds photos, words and drawings and sees them on a live 3D mug. One product and one template per mug size.

Example mount (11 oz):
```liquid
{% snippet 'product/mug-studio',
	ms_size: '11oz',
	ms_trim_w: '8in',
	ms_trim_h: '3.5in',
	ms_page: 'fullwrap',
	ms_bleed: '0in',
	ms_model: 'mug-11oz',
	ms_diameter: '3.25in',
	ms_edge: '0.75in',
	ms_min_dpi: '150',
	ms_element: 'wrap',
	ms_dpi: '300',
	ms_money: '$1,234.50',
	ms_other_url: '',
	ms_other_label: '15 oz' %}
```

| Setting | What it controls | Accepted values | Default if missing | Can you change it? |
|---|---|---|---|---|
| `ms_size` | The size of this product and template | `11oz`, `15oz` | `11oz` (logged) | **No**: one template per size |
| `ms_trim_w`, `ms_trim_h` | Print width and height, the page size of the XML template | length with unit, for example `8in`, `3.5in` | none, required | **With care**: must equal the XML template page |
| `ms_page` | The template page the print goes on | the page type in the XML template | `fullwrap` (logged) | **No** |
| `ms_bleed` | Bleed of that page | length with unit | `0in` (logged) | **With care**: must match the XML template |
| `ms_model` | The 3D mug model | `mug-11oz`, `mug-15oz` (a `p3d-model-<name>.js` asset on the parent) | none, required | **No** |
| `ms_diameter` | Mug body diameter, checked against the model | length with unit, for example `3.25in` | none, required | **No** |
| `ms_edge` | Area kept clear each side of the handle, shaded on the flat print | length with unit | `0.75in` (logged) | **With care**: match your press |
| `ms_min_dpi` | Below this a photo gets a low resolution warning | number | `150` (logged) | **Yes** |
| `ms_element` | Name of the full-bleed image placeholder the print fills | element name in the template | `wrap` (logged) | **No** |
| `ms_dpi` | Fallback print resolution, used only when the XML template dpi cannot be read | number | `300` (logged) | **No** |
| `ms_money` | Money format sample | for example `$1,234.50` | `$1,234.50` (logged) | **Yes** |
| `ms_other_url` | Path of the other size's product page, for a size switch | path, for example `/site/product/mugs/mug-studio-15oz`, or blank | blank (no size switch) | **Yes** |
| `ms_other_label` | Label of that other size | text, for example `15 oz` | blank | **Yes** |

The tool writes the hidden template options `ms_preview` (cart picture) and `ms_spec` (design record). Prices and options come from the product's own variants.

*Verified by reading `custom-design-tools/mug-studio/src/product__mug-studio.liquid` and `config/MOUNT-mug-11oz.liquid`, 24 Nov 2026. Not yet verified on a live page.*

## Changelog

- 2026-11-24: Page created.
