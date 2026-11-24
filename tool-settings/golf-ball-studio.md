# Golf Ball Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
1.0.0, built 23 Nov 2026. Being installed on experience.pixfizz.com as the demo; not yet on a customer site. Snippet: `product/golf-ball-studio`. Mount option: `gb_mount`. Prefix: `gb_`.

The shopper designs the print on a golf ball (a face, a photo, a logo, initials, a name or a ready design), sees it on a 3D ball, picks how many balls, and adds it to the cart. The tool writes a print-ready circle PNG and, for a name per ball, a sheet of circles.

Example mount:
```liquid
{% snippet 'product/golf-ball-studio',
	gb_version: '1.0.0',
	gb_ball_in: '1.68in',
	gb_print_in: '0.875in',
	gb_dpi: '600',
	gb_min_dpi: '100',
	gb_unit_balls: '3',
	gb_ladder: '3,6,12,24,48,96',
	gb_styles: 'face,photo,logo,initials,name,design',
	gb_ai: 'on',
	gb_enhance: 'on',
	gb_branded: 'none',
	gb_ready: 'Ready in about a week',
	gb_money: '$1,234.50' %}
```

| Setting | What it controls | Accepted values | Default if missing | Can you change it? |
|---|---|---|---|---|
| `gb_ball_in` | Ball diameter, for the 3D ball | length with unit, for example `1.68in` | none, required | **No**, unless your balls differ |
| `gb_print_in` | Diameter of the print circle on the ball | length with unit, for example `0.875in` | none, required | **With care**: must match your printer's print area and the XML template |
| `gb_dpi` | Resolution of the print file | number, for example `600` | none, required | **With care**: must match your production spec |
| `gb_min_dpi` | Below this the photo gets a low resolution warning | number | `100` (logged as missing) | **Yes** |
| `gb_unit_balls` | How many balls one cart quantity is | `3` when the product price is per pack of 3, `1` when per ball | none, required | **With care**: must match how the product is priced |
| `gb_ladder` | Ball counts in the How many list | whole numbers of units, comma separated | `3,6,12,24,48,96` (logged) | **Yes**: each must be a multiple of `gb_unit_balls` |
| `gb_styles` | Which styles are offered, in this order | any of `face`, `photo`, `logo`, `initials`, `name`, `design`, comma separated | all six (logged) | **Yes** |
| `gb_ai` | AI looks (cut out, cartoon) and the logo background remover | `on`, `off` | `on` (logged) | **Yes**. AI needs AI tokens on your site; without them the tool hides the AI looks |
| `gb_enhance` | Automatic color correction on the Photo style and the Original look, for signed-in shoppers | `on`, `off` | `on` (logged) | **Yes**. Uses one AI token per photo |
| `gb_branded` | Ball values that carry a maker logo on the other side (only "Maker logo only" is offered there) | `gb_ball` value codes, comma separated, or `none` | `none` (logged) | **With care**: codes must match your ball variant values |
| `gb_ready` | Turnaround line in the header | text, or `none` | `none` (logged) | **Yes** |
| `gb_money` | Money format sample | for example `$1,234.50` | `$1,234.50` (logged) | **Yes**: match your store currency |
| `gb_version` | The mount version, for your reference | text | blank | **Yes**, informational |

What the tool writes (do not rename these on your template): variants `gb_ball`, `gb_side2`, `gb_pack`, `gb_art_style`, `gb_proof` (those your product has) and template options `px_print_file`, `px_print_back`, `px_preview`, `px_spec`, `gb_photo`, `gb_artwork`, `gb_look`. Prices come from your product and variant prices.

Things that are not set here
- Prices, ball models, packaging and proof choices: variants on the product.
- Print circle position and page: the XML template.

*Verified by reading `custom-design-tools/golf-ball-studio/src/product__golf-ball-studio.liquid` and `config/MOUNT-golf-ball.liquid`, 24 Nov 2026. Not yet verified on a live page.*

## Changelog

- 2026-11-24: Page created.
