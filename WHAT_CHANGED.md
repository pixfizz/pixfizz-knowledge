# kbsync 2026-10-17: what changed

Sources: 145 Vault kb-patches dated 10 to 17 Oct (plus late additions to earlier ones), Slack #development 12 to 17 Oct, Fireflies summaries 9 to 17 Oct, the Notion Dashboard (no new entries since 9 Oct) and help.pixfizz.com (no articles changed since 1 Oct). Base: `main` at 050e8eb, checked again before delivery (unchanged).

Upload every .md file in this folder to github.com/pixfizz/pixfizz-knowledge, main branch, replacing the existing ones. Nothing in `orderhub/` is touched.

## Corrections (marked "Corrected 2026-10-17" in the files)
- 50_SHOPPER § 20, 52, 85 § 7: the GA4 server-side bridge is live since 2026-10-12 (was "not deployed"). `view-item` is now a dataLayer push.
- 61 § 13h, 18: `add_themes.json` answers 404 and adds nothing (12 to 14 Oct). The Publish Products wizard route (`manage_products/publish`) replaces it, by hand or by URL, including creating the product on a consumer site.
- 26: the inline tool page layout is reversed: the product description and delivery box stay in the tool's right column. The mount option code is `px_mount` for every tool.
- 23: `weekday` on `<defdate>` and `<dateshift>` jumps to that weekday inside the Monday to Sunday week, not the nearest occurrence.
- 32: kiosk tracking. Before 2026-10-12 nothing on the parent wrote `kiosk_id`; now it is written and needs a definition.
- 22: `file_upload` accept reads `option.accept`.
- 50_SHOPPER § 5 and § 9: `confirm-with-invoice` checklist meaning; child favicon takeover.
- 01: the `{% dynamic %}` scoping note now says not to rely on a dynamic block reading snippet params or earlier variables (observed live; a nested snippet call was a second cause).
- 71: the "no variant write" reason for read-only Catalog Manager prices is out of date against 61 § 13g.

## New rules and sections, by file
- 50_SHOPPER (19 add, 15 amend): page content uses the header's frame, and widen the header rather than the page; no `#modalLogin`; megamenu hover needs `data-toggle="dropdown"`; new checklist rows (`checkout-rush-style`, `pickup-phone-not-required`, `autofill-pickup-phone`); `page_schema` printed raw; `admin_only` is not access control; Services v2; splitting on a captured newline; collection filters and line endings (parent fixed 14 Oct, child overrides keep the old split), no empty state, driving filters from script with `Px.CMS.ThemeFilters`; `url_path` redirects and the `theme` parameter.
- 50_LIQUID, 52: a literal `{` before a tag; redirect regexes match the query string; `asset_url` resolves the child asset first; storefront snippet scope and a nested `{% snippet %}` clearing the caller's params; partial `user_update` posts; snippet inventory annotations.
- 20, 21: text variants never show on the cart line; `hide_edit_in_cart`; static products with no image; pickup contact phone; rush as dropdown, not radio; Invoice only for `invoice` users.
- 81, 82: Website Crawls; custom-domain sites still served on the pixfizz host; `/feed/products.xml` gaps; product x theme URLs are self-canonical duplicates; icons as inline SVG.
- 61 (20 add, 11 amend): admin users paging and addresses; calendar project create and the `POST /v1/calendars.json` observations; projects cannot be deleted; custom field definitions by admin form and the size limit; asset upload 413 on large files, `/fz/` signed URLs; an API key cannot add products to a collection; webhook payload facts; collections read and signed-out `price_forecast`; design product create; admin CSP; Extra Fee create.
- 60: master snippet fixes miss sites that override `px-option`; hiding the variant picker on photo-prints; Add to Cart hanging at 0% on a sku mismatch.
- 70, 71, 72, 45: self-chaining edge functions; the POD Catalog sync owns a synced supplier's rows; `ga4_event_outbox.raw_payload`; S3 CORS origin with a trailing slash breaks uploads.
- 18 (13 add, 6 amend): Template Versions page; the DELETE-form trap on more admin pages; renaming a design page does not rename its XML `name`; design and product-only import routes; bulk template option copy ignored design options before 13 Oct; 12 Oct admin UI changes; 2FA enforced for all admins, super admin reset; template provider and consumer behavior (publish, custom fields, preview images, provider price variables, categories).
- 16: design options on templates with no options; the `-N` name suffix; unpublished designs skipped on import; a missing price variable fails an import; the fulfillment code travels; `publish_theme` to unpublish from one collection.
- 19 (12 add, 2 amend): print PDFs downsampled to the template `dpi`; RGB only, no spot colors; a separate-file layer inside an `<ipage>` renders empty, with the two-ipage workaround (stated by the core developer, not test-rendered); spine in closed-book previews; top-fold cards and shaped windows; layflat counts spreads; image `rotate` / `grayscale` / `radius`; `kerning` in mm; text render validation; layout names over 32 characters fail import.
- 23: new section Calendar Element Tags, Fills and Transformations.
- 17, 51, 32, 40, 80, 83: editor page list includes `editor="false"` sets; `src = ''` fires `onerror`; `collection_footer` full width; orderline script can stamp an uploaded file URL; auto-submit firing twice; production file without an order; placeholder terms and shipping pages on children; blank 200 from a subdomain is not a live store; handoff checklist additions; image widening and reveal loops.
- 26 (11 add, 4 amend): `px_rates` rate card and the `pricing.js` contract with OrderHub (not verified); tool asset naming; upload dialog classes, events and the 250 px preview; the platform crop tool standalone; dark mode tokens and wording standards; filter-switched siblings share one mount; project edit page has no price; Gang Up canvas ceiling.
- 27: ornament-only bases; 4:5 room plates; supplier spec before modelling; wrong `p3d_page` shows the fallback button; hit testing on `preserve-3d`; brushed metal; a copied mount carries the source site's backdrop and filter ids.
- 22: product custom keys stored as `"false"` hide design options; textarea whitespace; tile labels truncate; `px-file-upload` internals; project-edit rendering; a non-additive finish x size matrix is one variant with combined codes.
- 30: price variables in option formulas; rounding to .95; `price_forecast` traps; Extra Fee loop confirmed; shipping rate by cart content; hidden number variants.
- 31: DPI and RGB; `product[fulfillment]` and Website default; moving a site off a shared destination; global (formerly public) fulfillment codes are internal-only (stated by the core developer); map every template's option codes in a job ticket; stale print files after project edit.
- 02: 84 new routes; every target section was written in this sync.

## Not added
- Claude and session tooling (33 patches or parts): Chrome uploads, harness habits, git, paste blocks, prototype tests, webfetch.
- Commercial: sales policy, client prices.
- Client-specific configuration (12), work in progress (9): Live Finish versions not installed, Pet Studio, Calendar and Greeting Card Studio internals, kiosk `store_id` format.
- 46 candidates were already covered.

## Pending confirmation (not written, or written as not verified)
Listed for the core developer in the delivery folder: calendars create by role, asset size limit, `{% dynamic %}` scoping, `add_themes.json` retired or bug, projects never deletable, pricing API rate, foil workaround test render, `kerning` unit, render queue and bad calendar pages, import timeout after 504, kiosk `store_id`, `sold_out` on scripted add to cart, global fulfillment code visibility, NCR forms pricing, `px-file-dropzone` and `orderline-pieces` on the parent, GA4 site-apply flow, `confirm-order-only` live.

## Size
Repo grows by roughly 150 KB.
