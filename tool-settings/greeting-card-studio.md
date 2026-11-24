# Greeting Card Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.7.3 on the Shopper parent; 0.7.9 on My Picture Perfect; 0.7.11 and 0.7.12 built, not live (24 Nov 2026).

As recorded on 3 Nov 2026: Status: live, version 0.6.13 (as of the CHANGELOG). Snippet: `product/greeting-card-studio`. Mount: not a template option. The studio sits on its own Pages page, whose page content is only the snippet tag. Prefix: `gcs_`.

## Changes since 3 November 2026

- `gcs_editor` (from 0.7.0): `true` shows "Finish in the advanced design tool", `false` hides every route into the design tool. **Yes.** The parent snippet passes it (verified in the 0.7.11 snippet, `data-editor`). The default when it is left out is not confirmed: pass it.
- `gcs_quantity_min`, `gcs_quantity_max` (from 0.6.33): turn on an "Another amount" typed quantity row (for example minimum 4). `gcs_proof_variant` (from 0.6.31): code of a free printed proof variant. **Ask Pixfizz.** The studio script reads all three, but the standard parent snippet does not pass them (checked in 0.7.11), so they only work on a site whose own snippet passes them (Hite Photo).
- Everyday designs (baby, Valentine's Day, thank you) are a separate studio page with its own collections; the settings are the same.

*Checked against docs/releases/0.7.11-unreleased on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper picks a 5 x 7 holiday card design (flat one-sided, flat two-sided or folded, portrait or landscape), adds photos, edits the words, picks paper, corners, envelopes and quantity, approves a proof and adds the cards to the cart.

Example mount (page content of the studio page):
```liquid
{% snippet 'product/greeting-card-studio',
	gcs_collection: 'holiday-cards',
	gcs_products: 'flat1-p:00000,flat1-l:00000,flat2-p:00000,flat2-l:00000,fold-p:00000,fold-l:00000,photo-p:00000,photo-l:00000',
	gcs_paper_variant: 'gcs_paper',
	gcs_finish_variant: 'gcs_finish',
	gcs_corners_variant: 'gcs_corners',
	gcs_envelope_variant: 'gcs_envelope',
	gcs_quantities: '10,25,50,100,150,300,500',
	gcs_default_quantity: '50',
	gcs_default_format: 'double',
	gcs_login_url: '/site/login',
	gcs_editor_on_mobile: 'true',
	gcs_diy: 'true',
	gcs_env: 'full',
	gcs_env_logo: 'either',
	gcs_env_logo_ink: 'color',
	gcs_env_colors: 'white,kraft',
	gcs_env_flap: 'true',
	gcs_env_country: 'US',
	gcs_env_max: '1000',
	gcs_fonts: 'studio',
	gcs_env_style: 'standard' %}
```
Replace each `00000` with the product id on your site.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `gcs_collection` | Design collection(s) the studio loads designs from. Designs are found by code `gcs-<design>-<flat1/flat2/fold/photo>-<p/l>` | Collection name(s), comma separated | none: the studio does not load | **With care**: must name collections on your site that hold the studio designs |
| `gcs_products` | Which card product sits behind each card type and orientation | `role:product id` pairs. Roles `flat1-p`, `flat1-l`, `flat2-p`, `flat2-l`, `fold-p`, `fold-l`, `photo-p`, `photo-l`. Leave out roles you do not sell | none: the studio does not load | **With care**: ids must be your real card products. Removing a role hides that card type |
| `gcs_paper_variant` | Variant type code of the paper on flat and folded cards | A variant type code, or `none` | `gcs_paper` | **No**: must match the variant type code on the products |
| `gcs_finish_variant` | Variant type code of the photo finish on photo cards | A variant type code, or `none` | `gcs_finish` | **No**: must match the variant type code |
| `gcs_corners_variant` | Variant type code of the corners on flat and photo cards | A variant type code, or `none` | `gcs_corners` | **No**: must match the variant type code |
| `gcs_envelope_variant` | Variant type code of the envelope on every card product | A variant type code, or `none` | `gcs_envelope` | **No**: must match the variant type code |
| `gcs_quantities` | Card quantities offered | Numbers, comma separated, smallest first | `10,25,50,75,100,150,200` | **Yes**: your pricing must give a price at every quantity |
| `gcs_default_quantity` | Quantity selected when the studio opens | A number from `gcs_quantities` | `50` | **Yes** |
| `gcs_default_format` | Card type a design opens on | `single`, `double`, `folded` | `double` | **Yes** |
| `gcs_login_url` | Sign-in page used by the upload window | A site path | `/site/login` | **Yes** |
| `gcs_editor_on_mobile` | Show Finish in the design tool on screens under 768 px | `true`, `false` | `true` | **Yes** |
| `gcs_diy` | Offer Your own design | `true`, `false` | `false` | **With care**: needs the `gcs-own` designs in the collection |
| `gcs_env` | Envelope printing | `off`, `return`, `full` | `off` | **With care**: needs the `gcs_env_print` variant values on the products and a lab that prints envelopes |
| `gcs_env_logo` | Logo on the envelope: next to the return address, or next to or instead of it | `off`, `with`, `either` | `off` | **With care**: only if your lab prints envelope logos |
| `gcs_env_logo_ink` | How the lab prints the logo | `color`, `black` | `color` | **With care**: must match your printer |
| `gcs_env_colors` | Envelope colors you can print on | Envelope value codes, comma separated (for example `white`, `natural`, `kraft`, `red`, `java`) | `white` | **With care**: must be values of your envelope variant |
| `gcs_env_flap` | Offer the return address on the back flap | `true`, `false` | `false` | **With care**: also needs the `gcs_env_flap` variant value `flap` |
| `gcs_env_country` | Home country. Home addresses get no country line | Two-letter country code | `US` | **Yes** |
| `gcs_env_max` | Most addresses per order | A number | `1000` | **Yes** |
| `gcs_fonts` | Studio fonts, or your site body and heading fonts | `studio`, `site` | `studio` | **Yes** |
| `gcs_env_style` | Envelope text style. `mpp` adds six envelope fonts, ink colors, sizes and alignment | `standard`, `mpp` | `standard` | **Yes** |
| `gcs_msp_collection` | MSP subscribers only: collection(s) holding MSP Hub card designs | Collection name(s), or leave out | none: no MSP designs | **With care**: MSP subscribers only |
| `gcs_msp_products` | MSP subscribers only: the MSP card products | `role:product id` pairs, same roles as `gcs_products` | none | **With care**: must be your MSP card products |
| `gcs_msp_files` | MSP library index file | A URL, or leave out | built-in Pixfizz index | **No**: leave it out unless Pixfizz gives you a URL |
| `gcs_store_collection` | Your own card designs, shown in the studio | Collection name(s), or leave out | none | **With care**: needs the next two settings too |
| `gcs_store_products` | Products behind your own designs | `role:product id` pairs | none | **With care**: must be your real products |
| `gcs_store_files` | Index file of your own design library | URL of `gcs-store-<tag>-files.json` in your site assets | none | **No**: set by whoever builds your library |
| `gcs_store_paper_variant` | Paper variant code on your own products, if different | A variant type code, or `none` | same as `gcs_paper_variant` | **No**: must match the variant type code |
| `gcs_store_envelope_variant` | Envelope variant code on your own products, if different | A variant type code, or `none` | same as `gcs_envelope_variant` | **No**: must match the variant type code |

Every setting should be in the mount. One that is left out is listed in `data-config-missing` on `#gcs-root` and in a console error, and the default is used. The `gcs_msp_` and `gcs_store_` settings and `gcs_fonts` are optional. Never put an apostrophe in a value.

From 0.6.41, a mount that has the envelope settings but not `gcs_diy` and `gcs_env_logo_ink` reports `the mount is missing: gcs_diy, gcs_env_logo_ink`. When you add a feature to an older mount, write the full current setting list shown above, including features that are off (`gcs_diy: 'false'`, `gcs_env_logo_ink: 'color'`). Tool-level. *Verified live on a client site, 2026-11-19.*

Things that are not set here:
- Paper, finish, corner and envelope names, descriptions and prices: the product variants.
- Card prices: the products and their pricing (the studio asks the platform for every price).
- The designs themselves and their artwork: the design collection.
- Card size: the card products and their templates, not the mount.

Not yet confirmed: Defaults come from the tool documentation and are not confirmed against the tool script.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
