# Calendar Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-27_

## Current version
0.5.2 live on the Shopper parent since 20 Nov 2026.

As recorded on 3 Nov 2026: Status: live, version 0.2.2 on the platform; 0.3.0 built and not yet released (as of the CHANGELOG). The settings are the same in both. Snippet: `product/calendar-studio`. Mount: not a template option. The studio sits on its own Pages page, whose page content is only the snippet tag. Prefix: `cs_`.

## Changes since 3 November 2026

- Design tool link: from 0.5.0 the studio offers no route into the design tool unless the studio page config `#cs-config` holds `"editor":"true"`. Default `false` (verified in the 0.5.2 script). **Yes.**
- `cs_crop` default is `true` from 0.5.0 (Crop buttons shown, crops sent to the editor). **Yes.**
- `cs_stands` decides which stands the desk calendar offers and names them in the spec line; with one stand the format tile does not crossfade (0.3.7).
- **Own templates mode** (0.6.0 and later): `"mode":"own"` in the page's `#cs-config` block lets a lab sell its own calendar designs. Self-serve for labs (decided by Pixfizz, 2026-11-25). Keys (0.6.2, verified by reading source 2026-11-25): `cs_products` takes any role name, and role `X-cv` is `X` with a cover; in `#cs-config`, `themes` = `role:design id` pairs (comma separated), `families` = `family:wall` or `family:desk` (vertical bar, landing order), `names`, `lines` and `sizes` = `family:text` (vertical bar), `variant` = one variant type code, `shipNote`. Other keys seen: `lang`, `coil`, `editor`, `images`. Own mode is detected from the block text `"mode":"own"`. Which version is live on the Shopper parent: to confirm.
- `printLang` in `#cs-config` (0.6.2): the studio writes month names and labels in that language into the saved pages at hand-off (works because saved calendar pages keep their text, `23_XML_CALENDAR_REFERENCE.md`). *Verified on a production PDF, 2026-11-25.*
- Design codes: the `-cs-<word>` suffix rule is in `26_CUSTOM_DESIGN_TOOLS.md` (Cross-tool rules).

*Checked against docs/releases/0.5.2 and src on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

The shopper builds a 12-month photo calendar (wall or desk): picks a design, adds photos per month, adds personal dates and holidays, reviews every page and adds it to the cart, or fine-tunes it in the design tool.

Example mount (page content of the studio page):
```liquid
{% snippet 'product/calendar-studio',
	cs_collection: 'calendar-studio',
	cs_products: 'wall:00000,wall-mon:00000,desk:00000,desk-mon:00000',
	cs_default_kind: 'wall',
	cs_holidays: 'none',
	cs_stand_variant: 'stand',
	cs_stands: 'block:block,clip:clip,refill:refill',
	cs_skin: 'atelier',
	cs_landing: 'true',
	cs_login_url: '/site/login',
	cs_exit_url: '/',
	cs_editor_on_mobile: 'true',
	cs_crop: 'true',
	cs_paper: '100lb matte cover' %}
```
Replace each `00000` with the product id on your site.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `cs_collection` | Design collection(s) holding the studio designs. Designs are found by code `<template code>-cs-<design>` | Collection name(s), comma separated | none: the studio does not load | **With care**: must name collections on your site that hold the studio designs |
| `cs_products` | Which calendar product sits behind each kind | `role:product id` pairs. Roles `wall`, `wall-mon`, `desk`, `desk-mon` (`wall` and `desk` start the week on Sunday, `-mon` on Monday). Leave out roles you do not sell | none: the studio does not load | **With care**: ids must be your real calendar products |
| `cs_default_kind` | What the studio opens on | `wall`, `desk` | `wall` | **Yes** |
| `cs_holidays` | Holiday calendar(s) printed on the wall calendar. From 0.3.0 the studio preview shows these same holidays | Site calendar id(s), comma separated, or `none` | `none` | **Yes**: ids must be calendars that exist on your site |
| `cs_stand_variant` | Variant type code of the desk stand on the desk products | A variant type code | empty: no stand sent | **No**: must match the variant type code on the desk products |
| `cs_stands` | Stand choices shown, in this order | `stand:variant value code` pairs. Stands `block` (wood block), `clip` (clipboard), `refill`. `none` for no stand choice | none | **With care**: value codes must match your stand variant values |
| `cs_skin` | Look of the studio | `atelier`, `noir`, `fresh` | `atelier` | **Yes** |
| `cs_landing` | Show the landing (hero, wall and desk cards) before the design gallery | `true`, `false` | `true` | **Yes** |
| `cs_login_url` | Sign-in page. Personal dates need a signed-in shopper | A site path | `/site/login` | **Yes** |
| `cs_exit_url` | Where Exit and the Calendars breadcrumb go when there is no landing | A site path | `/` | **Yes** |
| `cs_editor_on_mobile` | Show Fine-tune in the design tool on screens under 768 px | `true`, `false` | `true` | **Yes** |
| `cs_crop` | Older crop setting. From 0.2.0 photos always use the standard Shopper crop and crops always reach the print; this setting only governs old-style crops | `true`, `false` | `false` | **Yes**: no visible effect from 0.2.0 |
| `cs_paper` | Paper line shown in the studio | Text (for example `100lb matte cover`), or `none` | `none` | **Yes**: should describe the paper you actually print on |

Every setting should be in the mount. One that is left out is listed in `data-config-missing` on `#cs-root` and in a console error, and the default is used. Never put an apostrophe in a value.

Things that are not set here:
- Calendar size, bleed and units: the template XML definition.
- Week start: which product you list as `wall` or `wall-mon` (and `desk` or `desk-mon`).
- Design colors and photo layouts: the design template options and theme layouts.
- Stand names and prices, and calendar prices: the product variants and pricing.

Not yet confirmed: Defaults come from the tool documentation and are not confirmed against the tool script.

## Known issues

- **Clearing a design's Published tick does not take it out of the studio.** The studio reads every design in `cs_collection`, published or not. To hide a design on one site, remove its row from the collection; put it back with the Publish Products wizard. *Verified by reading source and live, 2026-11-25.* (all studios: `26_CUSTOM_DESIGN_TOOLS.md` § Cross-tool rules added 2026-11)
- **Keep template codes to 30 characters or fewer.** Design codes are `<template code>-<design key>`, must stay under 50 characters, and the longest design key is 18 characters. *Verified by query, 2026-11-25.*
- **With the site skin, the studio's primary button loses the site's font weight.** `.cs-root .btn.primary { font-weight: 500 }` in `style/calendar-studio` has the same specificity as the site-skin button rule and comes later. Until the studio CSS is fixed, add to the page content: `#cs-root[data-skin="site"] .btn{font-weight:var(--cs-btn-fw)}`. Template-level (Shopper 24 parent). *Verified live, 2026-11-22.*

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
- 2026-11-27: Own templates mode (0.6.x, self-serve) and `printLang`. New Known issues: Published tick ignored, template code length, site skin button weight.
