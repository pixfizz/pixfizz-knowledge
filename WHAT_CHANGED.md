# kbsync 2026-10-22: what changed

Sources: 114 Vault kb-patches written 17 to 22 Oct, Slack #development 17 to 22 Oct (#kb-sync was empty), 14 Fireflies calls 18 to 22 Oct, the Notion Dashboard (one new feature entry: AI upscaling at fulfillment) and help.pixfizz.com (no articles changed since 1 Oct). Base: `main` at fc9c24b, checked again before delivery (unchanged). Every changed line was reviewed independently before delivery (34 claims spot-checked against their sources; all mismatches fixed).

Upload every .md file in this folder to github.com/pixfizz/pixfizz-knowledge, main branch, replacing the existing ones. 34 files. Nothing in `orderhub/` is touched.

## Corrections (marked "Corrected 2026-10-22" in the files)
- 19 Element Permission Flags: text elements DO have `efontsize` and `ecolor` (only shapes lack `ecolor`).
- 18 and 61 § 13h: the publish POST needs `print_themes` positions (0-based ranges or dot lists); `print_theme_ids[]` publishes nothing (seen from 2026-10-20). The template option edit page carries three forms.
- 61 § 9: theme preview with no `width` returns 150 x 150 (was "default 100").
- 85 and 70: GA4 server purchases are browser first, server backfill only between 48 and 72 hours when GA4 has no purchase for the order (was: the server sends every confirmed order). Kiosk orders do fire a browser purchase, unreliably.
- 26 § 3: the platform upload dialog is mandatory for every tool that takes a customer file (was "should").
- 26 § 2 and § 7: Custom Sized Prints is priced by `px_rates` mode `area`, not per-stock Number variants.
- 82 § 4b: the iPad QR code issue is fixed; shapes, flip, filters and image controls are still open.
- 51 `spreads_as_pages`: label = (value - 1) x 2; the submitted value and `price_forecast` `pages` are system pages.
- 22, 26, 51: a product flag with no boolean definition (or typed as text) arrives as the string `"false"`; a real boolean definition is safe.

## New rules and sections, by file
- 16, 18, 13, 11: Template Export and Import Format (top-level maps, Ruby YAML tags and anchors, partial imports that fail on a variant type without its three empty lists or on `image: null`); Publishing Another Site's Template: what the lab product gets (no variants, no custom fields, fulfillment locked or pre-filled); POD tab sources and template consumers; the Publish Product link per template, Create and Tick, publish to an existing product, re-sorting and hiding rows; product attribute page is the edit page; Templates list for Import Template; admin access is all-or-nothing.
- 31, 32: template-locked fulfillment on the product page; Perfectly Clear upscaling at fulfillment (settings and per-template attributes, billing usage-based); spaces in filename templates stall transfers; `auto_confirm_orders` means no Pending window; Shopify orders start in `P`.
- 61, 30: saved vs ordered books, book detail fields, preview.png RGBA, page renders with `fulfillment=true`, `/v1/theme_categories.json` tree, `/v1/themes/<id>.json`, public `/v1/products.json` exposes formulas and custom fields, product create in one call, units on update only, content-addressed upload URLs, webhook auto-deactivation, merging users with `imsure`, draft orderlines; The Three Ways a Product Is Priced, `px_rates` area mode, flat per-line fee as amount / quantity, `price_forecast` prices only what you send, weight-based shipping formula mechanism.
- 17, 19, 22, 23: driving the editor (text, font, image, grayscale calls), tracking hooks, QR button, image border geometry, drawing order and groups in design exports, `show_on_preview` spelling, color wrap as a design option, `image_upload` target element names, variant type export and import; calendar layouts per page, week start in the date rule, render order of operations.
- 26, 27, 82, 83: upload dialog sources, file types, print wording, button classes read once at connect; `template` is nil in a mount like `product`; child-site install check for Template Option definitions; stepped number inputs block Add to Cart; retired options must be unpublished; CSS-only takeover with no flash; quiet drags; panel scroll anchoring; resolution shown only under 100 dpi; prototypes start from the live tool; component CSS defending against site `custom.css`; Generated template art; photo plates, room scenes, cylinders, banners, plate weight; GLB pipeline (Assets reject `.glb`, ship as JSON); CDN images as textures (both observations, safe rule); p3d page must name a template page; Live Finish flat printed mats; Nano Banana product plates and scenes.
- 50_SHOPPER, 52, 20, 21: cart remove confirm (new parent snippets), collection filters and CRLF (reconciled), CSS radio filters jumping, design products need `preview_img`, `px-design-preview` filters on the parent, `request.path` in custom.css, kit order emails, checkout gate for approved users, multi-store pickup posts the first public address, guest checkout fields browser-side only, Stripe Payment Element on dark sites, photo prints selection reset.
- 50_LIQUID, 51, 40, 41, 01: missing `asset_url` renders empty, parent asset fallback on children, `capture` into `website` redefines it, typographic apostrophe in strings, Pages `page_content` Liquid; custom field definition checks; scripted admin writes (select the form by a field, check `_method`); playbook entries for dark Stripe, a layout with no `page.content`, site CSS leaking into components.
- 60, 70, 71, 72, 85, 45, 80, 81: Shopify duplicate-user and empty-cart troubleshooting; myPixfizz SSO from the admin; binaries into myPixfizz through GitHub; POD Catalog category rules; GA4 `ga4_server_purchases`; OrderHub repricing from generic `px_` values, NAS hot folder delay, EasyPost tracking window; Etsy enabled per site; sitemap and feed cached about a minute.
- 02: 116 new routes, every target section checked.

## Not added
- Claude and session tooling, delivery conventions for Alex, GitHub Desktop and plugin version log (governance, not KB).
- Commercial: pack pricing basis, client prices, AI token costs, carrier rate figures.
- Client-specific configuration and work in progress (Pet Studio internals, Calendar Studio coil and pin, film app barcode stage, Lab Works XML pending confirmation).

## Pending confirmation
About 60 questions, listed for the core developer in QUESTIONS_FOR_MATJAZ_22OCT.md in the delivery folder, plus 3 for OrderHub support and 8 suggested OrderHub articles.
