# Photo Book Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
4.0.5 live on the Shopper parent since 19 Nov 2026; 4.0.6 (Serbian wording) on one site only.

As recorded on 3 Nov 2026: Status: live, version 3.4.4 (as of the CHANGELOG). Snippet: `product/photo-book-studio`. Mount: not a template option. The studio sits on its own Pages page, whose page content is the snippet tag, optionally followed by a `pbs-config` block. Prefix: `pbs_`.

*Notes up to 21 Nov carried over; not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

A full-page photo book wizard for Marathon Press books: the shopper picks a book type, size and cover design, adds photos, arranges the pages, reviews the book and orders it, or finishes in the design tool.

Example mount (page content of the studio page):
```liquid
{% snippet 'product/photo-book-studio',
	pbs_collection: 'photo-book-studio',
	pbs_products: '00000,00000,00000',
	pbs_types: 'hardcover,softcover,layflat',
	pbs_default_product: '00000',
	pbs_default_style: 'gallery',
	pbs_default_softcover: '00000',
	pbs_default_layflat: '00000',
	pbs_skin: 'atelier',
	pbs_look: 'paper',
	pbs_editor_on_mobile: 'true',
	pbs_crop: 'true',
	pbs_title_standins: 'true',
	pbs_landing: 'true',
	pbs_login_url: '/site/login',
	pbs_exit_url: '/',
	pbs_loader: 'cover' %}
```
Replace each `00000` with a book product id on your site.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `pbs_collection` | Design collection(s) holding the studio designs (codes `<w>x<h>-design-<style>-<hc/sc/lf>`) | Collection name(s), comma separated (for book types kept in separate collections) | none | **With care**: must name collections on your site that hold the studio designs |
| `pbs_products` | Book products shown, in display order | Product ids, comma separated | none | **With care**: ids must be your real book products |
| `pbs_types` | Book types shown, in order | Any of `hardcover`, `softcover`, `layflat`, comma separated | `hardcover` | **Yes**: only types you have products and designs for |
| `pbs_default_product` | Product the studio opens on | A product id from `pbs_products` | first product | **Yes** |
| `pbs_default_style` | Design the studio opens on | A design style key, or `gallery` to open on the design gallery | `gallery` | **Yes** |
| `pbs_default_hardcover` | Hardcover product picked when the shopper switches type | A product id | none | **Yes** |
| `pbs_default_softcover` | Softcover product picked when the shopper switches type (when softcover is listed) | A product id | none | **Yes** |
| `pbs_default_layflat` | Layflat product picked when the shopper switches type (when layflat is listed) | A product id | none | **Yes** |
| `pbs_skin` | Look of the studio | `atelier`, `noir`, `fresh` | `atelier` | **Yes** |
| `pbs_look` | Card look in the gallery | `paper` | `paper` | **No**: only one value in this version |
| `pbs_editor_on_mobile` | Offer finishing in the design tool on phones | `true`, `false` | `true` | **Yes** |
| `pbs_crop` | Photo crop in the studio | `true`, `false` | `true` | **Yes** |
| `pbs_title_standins` | Extra title layout suggestions when a design has few. They have no layout id, so the design tool does not offer them | `true`, `false` | `true` | **Yes** |
| `pbs_landing` | Show the landing before the gallery | `true`, `false` | `true` | **Yes** |
| `pbs_login_url` | Sign-in help link | A site path | `/site/login` | **Yes** |
| `pbs_exit_url` | Where Exit goes | A site path | `/` | **Yes** |
| `pbs_loader` | Loading animation | `cover` (the cover is pressed), `pages` (pages turning) | `cover` | **Yes** |

Settings left out are listed in `data-config-missing` on `#pbs-root` and in a console error. Never put an apostrophe in a value. Add `?pbs_debug=1` to the page link to see the studio log; that is a link, not a setting.

Your own books (advanced). A store that sells its own book lines and designs instead of the standard Marathon Press set switches this on with a JSON block on the same page, after the snippet tag: `<script type="application/json" id="pbs-config">{"collections":{"source":"collections", ...}}</script>`. Inside `collections`, the studio reads `ranges` (required: `<range>:<product ids>` separated by `;`), `rangeNames` and `rangeLines` (separated by a vertical bar), `exclude`, `blankFrom`, `unit`, `moneyDecimals` (`true` or `false`) and a few layout keys. These are set up by Pixfizz for each store; treat them as **No** unless Pixfizz tells you otherwise. From 4.0.4, a store whose own books are page books (`cover`, `page01`, `page02` ...) rather than lay-flat spreads also has `"layflat":"false"` in the `collections` block; without it the page stays on "Setting up your studio" (default: lay-flat). Each own design also needs its Linked Layouts, or the studio has no layouts. *Verified live on a client site, 2026-11-19.* The same block can switch on the 3D landing hero (`hero3d`), also set up by Pixfizz. *Corrected 2026-11-09: from 4.0.1 the 3D hero needs no page block; see "Landing hero" below.*

**From 3.4.0, collections settings work only in `#pbs-config`.** The parent snippet no longer passes collections-mode params (`pbs_source`, `pbs_ranges`, `pbs_range_names`, ...) to `#pbs-root`; params given in the snippet call are silently ignored. Symptom: the page shows "The studio could not load. Please refresh the page." and the console says the mount is missing `pbs_collection, pbs_products, pbs_types, pbs_default_style` and "pbs_collection and pbs_products are required", because the source never became `collections` and the standard book layer ran. Fix on the site, with no parent change: put the settings in the `#pbs-config` block in the Pages page content. Keys are the camelCase dataset names (`source`, `ranges`, `rangeNames`, `rangeLines`, `exclude`, `blankFrom`, `defaultProduct`, `skin`, `look`, `editorOnMobile`, `crop`, `titleStandins`, `landing`, `loginUrl`, `exitUrl`, `moneyDecimals`, `sampleTitles`, `demoTitle`, `demoCaptions`, and others); the 3.4.0 prelude copies each one into `#pbs-root` only where the snippet left it empty. Check: the console ready line reads `source collections` and lists your types. Tool-level. *Verified by query on a live client site with the block placed before the snippet tag, 2026-11-03 (hand-off to the design tool and add to cart not tested).* Whether the block also works after the snippet tag, as written above, is pending confirmation.

**Landing hero: 3D only.** The 2.x CSS hero animation is retired on every site; the 3D landing hero is the only hero. Since 4.0.1 its images and three.js come from the parent mount itself (`pbs-assets`: `three`, `heroPlate`, `heroPlateM`, `heroPoster`, `heroPosterM`), so no per-site `#pbs-config` block is needed to switch it on; the block may still override the images, and `heroPlay()` returns at once. With no WebGL or a renderer failure the page shows the 3D hero's own still (table and closed book) or the hero photo, never the CSS animation. Install check: the live page console logs `[photo-book-studio] hero {mode: 3d}`; a site still showing the old animation is running an older build or a page set up before 4.0.1. A console line `no 3D hero images (heroPlate)` means the shared parent mount snippet is older than the studio asset (seen 18 Nov 2026); ask Pixfizz to bring it level. Tool-level. *Decided by Pixfizz, 2026-11-03; seen on a pack install the same day.*

Things that are not set here:
- Page limits, book sizes and book types: the templates and design codes.
- Cover and paper options and their prices: the product variants and pricing.
- Designs and layouts: the design collection and its themes.

Not yet confirmed: The default of `pbs_default_product` and the exact meaning of `pbs_look` are not confirmed.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
