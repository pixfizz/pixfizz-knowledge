# kbsync 2026-11-13: what changed

Sources: 70 internal working notes written 9 to 13 Nov (about 55 applied in full or in part, the rest skipped as tooling, internal process, site-specific or already covered), plus the Phase 1 scan of 13 Nov (Slack, 20 calls, 44 support tickets, Notion Dashboard, help.pixfizz.com). An independent review checked 62 claims; 3 mismatches were fixed before delivery.

Upload every changed .md file in this folder to github.com/pixfizz/pixfizz-knowledge, main branch, replacing the existing ones. 18 files, this one included. Nothing in `orderhub/` is touched.

## Corrections (marked "Corrected 2026-11-13" in the files)
- 18 Import and publish routes: the GET `add_products` link now answers Not Found. Static products are added through the collection page's Add Static Product picker, or by POST `add_products` with `products[]`. The older pointer in 50 was updated to match.
- 50 § 25 Photo prints flow: sub-collection groups sort by display name, not by path.

## New rules and sections, by file
- 18: More Admin Screens and Form Routes (email notification URLs and the To field, Pages entry edit URL, design admin page, hidden template options as JSON, "-1" on imported design names, publish adds without removing, export from the provider, collection export with child collections, import timing and layout_id, Bulk Updates template options copy, variant forms, product copy, the product form, collection Ace fields, page XML save, asset names, admin origin and CSP).
- 50: § 25 additions (bare /site/prints, size labels, size panel overflow, empty variant heading); new § 26 Collection Pages, Product Pages and the Page Frame.
- 50_LIQUID: redirects must never catch platform routes; Two Silent Liquid Failures (filters in a for tag, asset_url on a missing asset).
- 30: pack prices per unit; shipping formulas break the cart page; uploader price cards (engine order, Flat Prints).
- 19: definition unit from the theme JSON; Output DPI, Bleed and the First page (incl. orientation twins by transpose).
- 17: the editor close button and exit_target; px-design-preview notes.
- 22: variant export and rendering behavior.
- 26: cross-tool rules (WebGL context loss, upload project timing, project-edit size, product schema sku, priced sizes only, Collage Studio, envelope looks).
- 27: Live Finish findings (variant codes, ornament ribbons, English-only labels, standoff keys, Live Finish on a Pages page, name on a tag, clean scenes).
- 31: test the FTP login of every destination.
- 60: Cart handling (addon lines): the double-billing fix and master page precedence.
- 61: MCP connector setup note; § 13i snippets, Pages entries, placeholder images and host-signed asset URLs.
- 80: New Shopper Children: Check Before Writing.
- 81: Part H (empty llms.txt, schema sku). 83: start from the real product at the real scale. 40: five symptom pointers.
- 02: 40 new routes.

## Pending confirmation
- Sub-collection group order: the latest live check says display name; an earlier note the same day said path. A source read or retest settles it.
