# Keepsake Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: live, version 0.6.3 (as of the CHANGELOG; snippet 0.6.0). Snippet: `product/keepsake-studio`. Mount option: `ks_mount` (hidden Text template option, custom_script field). Prefix: `ks_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

Replaces the main product image with a real-time 3D acrylic keepsake (a plate on a stand, or a thick acrylic block) that follows every choice the shopper makes. The artwork is the platform render of the design page, so it matches the print. It writes nothing to the order.

Example mount (acrylic plate on a stand):
```liquid
{% snippet 'product/keepsake-studio',
	ks_page: 'preview',
	ks_plate_w: '5.25in',
	ks_plate_h: '7.25in',
	ks_thickness: '0.125in',
	ks_photo_option: 'image-upload',
	ks_base_option: 'base',
	ks_bases: 'bent:bent,stand:stand,wood:wood',
	ks_hosts: '*',
	ks_fulfiller: '',
	ks_label: 'Acrylic keepsake 5 x 7' %}
```

Example mount (acrylic block):
```liquid
{% snippet 'product/keepsake-studio',
	ks_model: 'block',
	ks_page: 'print',
	ks_sample_page: 'sample',
	ks_plate_w: '5.13in',
	ks_plate_h: '7.13in',
	ks_trim: '0.065in',
	ks_thickness: '1in',
	ks_front: 'variant:printing-style,on:front,layer:Text',
	ks_photo_option: 'photo',
	ks_bases: '',
	ks_base_option: '',
	ks_base_default: '',
	ks_scenes: 'studio,desk',
	ks_hosts: '*',
	ks_fulfiller: '',
	ks_label: 'Acrylic block 5 x 7',
	ks_render_width: '1200' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `ks_model` | Plate on a stand, or a solid block | `plate`, `block` | `plate` | **With care**: must match the real product |
| `ks_page` | Design page rendered onto the keepsake. Required | a page name in the design | none | **With care**: must name a real page |
| `ks_plate_w` | Width of that page. Required | a length | none | **With care**: must match the page size (checked against the render) |
| `ks_plate_h` | Height of that page. Required | a length | none | **With care**: must match the page size |
| `ks_thickness` | Acrylic thickness | a length | `0.125in` | **Yes**: match the real product |
| `ks_photo_option` | The photo upload option on the design | option code | `image-upload` | **No**: must match the option code on your design |
| `ks_base_option` | The base (stand) option on the design. Plate only | option code | `base` | **No**: must match the option code on your design |
| `ks_bases` | Stand model per base value. Plate only | `value:model` pairs; models `bent`, `stand`, `wood`, `none` | the value itself when it is a model name, else guessed | **With care**: values must match your base option |
| `ks_base_default` | Stand when the design has no base option. Plate only | `bent`, `stand`, `wood`, `none` | `bent` | **Yes** |
| `ks_hosts` | Site host names where the preview runs | comma list of host names, or `*` / blank for everywhere | everywhere | **Yes** |
| `ks_fulfiller` | Maker name, shown only in the browser console | text | blank | **Yes** |
| `ks_label` | Product name for screen readers | text | `Acrylic keepsake` | **Yes** |
| `ks_scenes` | Views offered. Labels are fixed: Close-up, On the shelf, On the desk. A block uses `studio,desk` | `studio`, `shelf`, `desk` (comma list) | `studio,shelf` | **Yes** |
| `ks_render_width` | Render width in px | number, platform caps at 1200 | `900` | **Yes** |
| `ks_sample_page` | Page rendered before an upload. Block only | a page name, or blank for `ks_page` with the sample photo | blank | **With care**: a name that does not exist (for example `none`) falls back to `ks_page` with the sample photo, so it is a safe way to get the sample on a design with no sample page. *Corrected 2026-11-09: previously said it must name a real page. Verified by query, 2026-10-31.* |
| `ks_trim` | Cut from each side of the render. Block only | a length | none | **With care**: must match the template bleed |
| `ks_front` | The choice that prints a PDF layer on the front of a block. Block only | `variant:<code>,on:<value>,layer:<layer name>`, or `none` | everything on the back | **With care**: must match the variant code, value and the PDF layer name in the XML template |

Notes on two settings
- **`ks_photo_option` is often a design option, and its code can differ between templates of one product line.** The template page can say "There are no options for this template" while each design carries its own photo option (see `16_PRODUCT_HIERARCHY.md`); the admin options list of the template (`admin/templates/<id>/options`) shows those design-level options too. On one lab's acrylic block range the code was `upload-image` on some templates and `photo` on others, so read it per template before writing each mount. Platform-level fact, site-specific values. *Verified by query, 2026-11-03.*
- **The empty photo area is found by its grey, which differs per site.** The placeholder image is per site (one site renders 215,215,215, the platform default 210,208,208), so the tool does not rely on one hard-coded grey: from 0.7.2 it picks the most common flat light grey when the default is absent. See `61_PIXFIZZ_API.md` § 9. Tool-level. *Verified by query, 2026-10-31.*
- **`ks_front` reads only variant choices** (`variants[...]`), not template options (`template_options[...]`). A site whose layered-text choice is a template option needs an engine change before `ks_front` can follow it. Tool-level. *Verified by reading source (0.7.4), 2026-11-03.*

Things that are not set here
- The plate outline (arched, round, square). It is read from the transparent edge of the design render.
- Production. The print file is still the design's own print page.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
