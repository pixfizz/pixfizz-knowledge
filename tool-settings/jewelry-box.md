# Jewelry Box: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-27_

## Current version
0.1.3, built and tested on baseline 18 Nov 2026; demo install on experience; not yet on a customer site. Snippet: `product/jewelry-box`. Mount option: `px_mount`. Prefix: `jb_`. Needs Live Finish Plates 0.6.0 or later on the parent.

The shopper personalizes a travel jewelry box lid: box color, a design, a name or monogram and lettering, shown on photo-real box views.

Example mount:
```liquid
{% snippet 'product/jewelry-box',
	jb_color_variant: 'boxcolor',
	jb_colors: 'black:black,blue:blue,brown:brown,gray:gray,natural:natural,pink:pink,red:red,teal:teal',
	jb_fonts: 'Great Vibes,Allura,Caveat SemiBold:0.86,Playfair Display:serif,Cormorant Garamond Medium:caps:0.7,Josefin Sans SemiBold:caps',
	jb_font_serif: 'Playfair Display',
	jb_font_caps: 'Montserrat SemiBold',
	jb_designs: 'all',
	jb_sets: 'off',
	jb_max_name: '16',
	jb_min_dpi: '150',
	jb_title: 'Personalized Jewelry Box',
	jb_lede: 'Pick the color, a design and the name.' %}
```

| Setting | What it controls | Accepted values | Default if missing | Can you change it? |
|---|---|---|---|---|
| `jb_color_variant` | Code of the variant that holds the box color | variant code | none, required | **No**, unless you renamed it |
| `jb_colors` | Which box color shows which photo views | `value code:plate color` pairs, comma separated. Plate colors: black, blue, brown, gray, natural, pink, red, teal | none, required | **With care**: values must match your color variant |
| `jb_fonts` | Six lettering fonts: three scripts, a serif, two capitals | font names from your site's font list, comma separated, with optional `:serif`, `:caps` and a size factor | none, required | **With care**: each must be on your site's font list |
| `jb_font_serif` | Font for monogram letters and big initials | font name | none, required | **With care** |
| `jb_font_caps` | Font for the small capital lines | font name | none, required | **With care** |
| `jb_designs` | Which designs are offered | `all`, or design ids, comma separated | none, required | **Yes** |
| `jb_sets` | "A name on each box": several names, one cart line each | `on`, `off` | none, required | **Yes** |
| `jb_max_name` | Most letters in a name | number | none, required | **Yes** |
| `jb_min_dpi` | Below this a photo gets the low resolution warning | number | none, required | **Yes** |
| `jb_title` | Heading | text; blank uses the product name | blank | **Yes** |
| `jb_lede` | One line under the heading | text; blank shows none | blank | **Yes** |

The tool writes the color variant and the template options `px_print_file`, `px_preview`, `px_spec`, `px_state`.

- **Print file:** PNG with a transparent background and no bleed; only the ink (and the photo, when the design has one) is opaque. The size comes from the template XML definition (for example 4 x 4 in at 300 dpi = 1200 x 1200 px), and the dpi is written into the PNG so it opens at its real size. The "Full photo" design covers the whole lid, so that file has no transparent area. It is stored on the order line as `px_print_file`; `px_preview` is a 600 px JPEG of the lid on the box for the cart; `px_spec` carries color, size and dpi as text. *Verified by reading source and a local test, 2026-11-24. Not yet verified: a real order's print file arriving through fulfillment.*
- **Sets on (`jb_sets: 'on'`):** each name becomes its own cart line at the normal box price, with the same color and design; no set discount. *Verified with three names, 2026-11-18.*
- **Saving `jb_sets`:** the template option edit page has two Save buttons. The `custom_script` field saves only with the lower Save (the custom fields form), not the top one.
- **Install:** template import carries the options, the box color variant and the mount; then publish to a collection and link it in the menu. Edit from cart stays hidden (`hide_edit_in_cart`) until restoring a design from the cart is tested live.

*Verified by reading `custom-design-tools/jewelry-box/build/src/product__jewelry-box.liquid` and `config/MOUNT-experience.liquid`, 24 Nov 2026.*

## Changelog

- 2026-11-24: Page created.
- 2026-11-27: Print file, sets, saving `jb_sets` (lower Save) and install notes.
