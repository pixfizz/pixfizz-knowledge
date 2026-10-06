# kbsync 2026-10-06: what changed

Sources: 57 project KB_PATCH docs (29 Sep to 5 Oct, plus older ones checked for coverage), 17 Vault kb-patches, Vault tools/ and clients/experience/ status docs, project specs for Live Finish, 3D previews, custom tools and myPixfizz, the myPixfizz app source in Lovable (commit d6290ee, read only), and the Phase 1 scan of 2 Oct.

Upload all 30 files in this folder (except this file) to github.com/pixfizz/pixfizz-knowledge, main branch, replacing the existing ones. 27 is a new file.

## New file
- 27_LIVE_FINISH_AND_3D_PREVIEWS.md: the KB had no coverage of Live Finish or 3D previews. Covers what a product preview is vs a custom design tool, the Live Finish standard (mount keys, looks, geometry from the template, gating, dark sites, reserved query params), the 3D Preview standard (p3d_mount, Blender GLB models, wrap rules, versioning), status by product family, known defects, install pattern, definition of done.

## Largest updates
- 26_CUSTOM_DESIGN_TOOLS.md: registry and per-tool reference brought to current state (Uploader 2.0 and NCR forms, Wall Designer, Board Engraver, Gang Up trim and cut path, flyer-fold, B2B tools on experience.pixfizz.com). New: fulfillment contract (px_ codes), shared px-tool-theme look, inline page layout, every choice on a variant (recall), price ladders, inches and mm, self-install from myPixfizz, tool cart image, upload dialog standalone.
- 31_FULFILLMENT_ENGINE.md: new Custom Tool Fulfillment Standard section. Client name scrubbed from the QR worked example.
- 71_MYPIXFIZZ_FEATURES_ROUTES.md: rebuilt from the app source. Every route verified by reading source; Marketing, Shopper Configure, Shopify Style, Launch Plan, Support (urgent tickets, order error form, auto-resolve), Tools (Blog, Upgrades, Template resizer, Live Finish setup, Merge users, Catalog Manager). Older details not re-checked are kept and labeled.
- 70 / 72 myPixfizz: operational rules (migrations hit live, publish ships the whole queue), organization add-ons, new tables Sep to Oct.
- 17 / 19 design tool and XML: spine insertion, design import, linked layouts, edit page XML in place, shrink / valign / crop / edit flags, editor CSS field (corrected), driving the editor from a script, size change cost on existing designs.
- 61_PIXFIZZ_API.md: /upload/image (corrected), external user lookup, custom type CRUD (corrected), asset PUT/DELETE (corrected), child snippet overrides by API, new § 13h Admin Form Writes.
- 50_SHOPPER / 50_LIQUID / 52: manage/* token values (corrected rows), dark child cart and checkout (high risk), show_prices, Klaviyo consent, services/ route, Liquid in page_content, no json filter, website.* lists cut at 20, shopper-admin file inputs.

## Corrections to existing rules (marked "Corrected 2026-10-06" in the files)
- 17: a design import creates a new design with remapped ids (it does not overwrite by id). Custom CSS field is a list of files, not CSS text.
- 20: requires_design does not break DTF / file-upload orders. Cart-to-order custom values need an Order custom field definition.
- 22: variant type exports DO carry hidden / read_only / hide_from_cart (verified 29 Sep).
- 30: Price Variables API works on production, not staging only.
- 32 / 90: meaning of the Fulfilled status; email templates are per site, not inherited.
- 52: film/roll-builder parent fix is still pending.
- 61: upload image route, custom type PUT/DELETE, asset replace, design custom fields via PUT save nothing.

## Not added (not for the public repo)
- Claude and Cowork tooling: device git index.lock, egress, zip in connected folder, Chrome file_upload limits, pbpaste, Recraft via Mac, render tests through file inputs, catalog scraping technique, design canvas without egress, Lovable prompting habits, one session per kit folder, GitBook relative links.
- Commercial: MSA support rates, no sandbox / no trials sales policy, any prices in specs.
- Client-specific configuration (product ids, formulas, account situations).
- Intuit production keys (billing internal).

## Already covered by the 24 and 29 Sep syncs (Phase 1 report)
API keys, passkey / 2FA, admin host move, notification email map, template dimension locking and refulfill, binding map error, google_category (the field is `google_category`; `google_product_category` does not exist in Shopper).

## Knowledge gaps flagged, not written
- Adding a new size to a live photobook range and setting additional-pages quantity and price (4 tickets from one client).
- Shopify cart blocked by a SKU or product mismatch.
- Moving a storefront from a legacy Pixfizz CMS URL to Shopify (blocking and redirecting the old URL).

## Open questions
Platform questions for Matjaz are on the Notion Dashboard, Weekly Tasks, October. Status questions for Alex are in the chat.

## Size
Repo grows from 1.34 MB to 1.67 MB. Project knowledge goes to roughly 1.72 MB of the 2 MB cap before the closing step deletes the applied KB_PATCH docs from the project.

## Addendum, later on 6 Oct: limits, product form and fulfillment hold

Source: the core developer's 2025 email advice to an agency building an AI story book site and an image-upload site, checked against the KB, plus Alex's and the core developer's answers on 6 Oct. Client names scrubbed.

- 51: CORRECTED the template option limit to 1,024 characters (was "~2KB"). Snippet type exists only for custom fields. New: 65,535-byte total for all custom field data on one object (projects, orders, products, users, custom type instances). Length limits summarized in one place.
- 22: new § 6.1 (`data-px-no-element-substitutions` keeps an input out of preview URLs, `skip_for_previews` override) and § 6.2 (options written into the page URL; long values crash the product page; ignore-list fix).
- 50 Liquid: writing project custom fields with `book[custom][<field>]` in `project_create`; the field must be Public.
- 17: Restoring Input Values After a Product Switch (`pageshow` and `px.fragmentsReloaded`). 41: qualifier on the same event.
- 40: Slow Previews (heavy PNGs) and Template Output Set to JPEG (no 1c black text).
- 32: Fulfillment Hold (Super Admin) is in minutes.
- 61: `/upload/image` `name` is optional.
