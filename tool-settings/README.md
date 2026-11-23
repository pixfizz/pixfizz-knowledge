# Tool settings: one page per tool

**Authority Scope:** the settings (the mount) of every Pixfizz studio, custom design tool, order builder and product preview, one page each. How the mount works, the Yes / With care / No rating and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a page says otherwise.

_Last updated: 2026-11-24_

Each page gives the current version, what changed since 3 Nov 2026, an example mount with generic values, and every setting with its accepted values, default and whether you can change it yourself. Settings marked **No** or **Ask Pixfizz** need a Pixfizz staff member.

## Studios and design tools

| Tool | Page | Status |
|---|---|---|
| Business Cards | `business-cards.md` | 3.7.2 is the newest release (Playpoint name cards, Nov 2026). Which version each site runs is not checked here: the console start-up line shows it. |
| Greeting Card Studio | `greeting-card-studio.md` | 0.7.3 on the Shopper parent; 0.7.9 on My Picture Perfect; 0.7.11 and 0.7.12 built, not live (24 Nov 2026). |
| Calendar Studio | `calendar-studio.md` | 0.5.2 live on the Shopper parent since 20 Nov 2026. |
| Collage Studio | `collage-studio.md` | beta, version 1.1.0 built 3 Nov 2026 (as of the release notes; the repo marks every release folder unreleased, so live status is not verified). Snippet: `product/collage-st |
| Puzzle Studio | `puzzle-studio.md` | 0.6.0 live on the Shopper parent since 5 Nov 2026. |
| Stationery Studio | `stationery-studio.md` | 0.4.0 live on the Shopper parent since 19 Nov 2026. |
| Creative Studio (Make your own notebook cover) | `creative-studio.md` | 0.1.5 live on the Shopper parent (5 Nov 2026); 0.1.8 built. |
| Pet Studio | `pet-studio.md` | live, version 0.4.1 (as of the CHANGELOG; snippet 0.4.1, asset 0.4.0). Snippet: `product/pet-studio`. Mount option: `px_mount` (hidden Text template option, one per templat |
| Photo Book Studio | `photo-book-studio.md` | 4.0.5 live on the Shopper parent since 19 Nov 2026; 4.0.6 (Serbian wording) on one site only. |
| Cover Studio | `cover-studio.md` | in test, version 1.8.3 (asset; style updates numbered 1.9.1, as of the CHANGELOG). Snippet: `product/cover-studio`. Mount: `{% snippet 'product/cover-studio' %}` with no ar |
| Custom Sized Prints | `custom-sized-prints.md` | live, version 0.4.1 on the shopper24 parent; 0.5.0 (metric) built, not yet released. Snippet: `product/custom-print`. Mount option: `px_mount` (hidden Text template option, |
| TilePix Wall Studio (Wall Designer) | `tilepix-wall-studio.md` | 0.6.12 live on the Shopper parent since 9 Nov 2026. Asset-only releases: mounts unchanged. |
| Wall Design Studio (metal and acrylic walls) | `wall-design-studio.md` | 0.8.9 live on the Shopper parent since 9 Nov 2026. |
| Fan Face (face fan designer) | `fan-face.md` | 1.3.1 live on Hite Photo only (23 Nov 2026). Older versions on other installs. 2.0.0 built and tested, not released. |
| Custom Framing | `custom-framing.md` | built, version 0.4.0; not yet moved to the mount standard. Snippet: `product/framing`. |

## Print and cut tools

| Tool | Page | Status |
|---|---|---|
| Sticker Studio | `sticker-studio.md` | 6.3.0 live on the Shopper parent since 3 Nov 2026; 6.4.0 built, not released. |
| Sticker Designer (legacy) | `sticker-designer-legacy.md` | 5.31 (4 Nov 2026). Being retired: sites move to Sticker Studio. |
| Gang Up (DTF gang sheet) | `gang-up.md` | 2.2.0 live; 2.3.0 built, not released (24 Nov 2026). |

## Upload tools

| Tool | Page | Status |
|---|---|---|
| Uploader 2.0 (documents, booklets, forms, flat prints) | `uploader-2.md` | live, version 2.6.0 (per the release notes; this folder has no CHANGELOG). Replaces Document Uploader and Booklet Uploader on new products. Snippet: `product/uploader`. Mou |
| Document Uploader | `document-uploader.md` | live, version 1.3.3 (per the CHANGELOG). To be replaced by Uploader 2.0. Snippet: `product/document-uploader`. Mount option: `du_mount`. Prefix: `du_`. |
| Booklet Uploader | `booklet-uploader.md` | live, version 1.5.2 (per the CHANGELOG). To be merged into Uploader 2.0. Snippet: `product/booklet-uploader`. Mount option: `bu_mount`. Prefix: `bu_`. |
| Publication Upload | `publication-upload.md` | 1.16.0 released 4 Nov 2026 (upload standard). |
| Flyer / Brochure | `flyer-brochure.md` | 1.0.4 live (10 Nov 2026); 1.1.0 built, not released. |

## Order builders

| Tool | Page | Status |
|---|---|---|
| Design Service (Design Brief) | `design-service.md` | beta, version 2.3.1 (live on the shopper24 parent; first client installs October 2026). Snippet: `product/design-brief`. Mount option: `px_mount` on new installs (older ins |
| Film Order Builder | `film-order-builder.md` | in progress, version 0.2.0, no client installs. Snippet: `product/film-builder`, called from a page (not a template option mount). Prefix: `pxfb_`. |
| Film Roll Builder | `film-roll-builder.md` | phase 1, version 0.1.1, live on the demo site. Snippet: `film/roll-builder`, mounted on a Pages page whose content is only `{% snippet 'film/roll-builder' %}`. Prefix: `frb |

## Product previews

| Tool | Page | Status |
|---|---|---|
| Live Finish | `live-finish.md` | 0.29.0 live on the Shopper parent since 17 Nov 2026; 0.29.1 (wording in the store's language) built for one site. |
| Live Finish Plates | `live-finish-plates.md` | 0.5.0 live; 0.6.0 built for the Jewelry Box, not on the parent yet. |
| 3D Preview | `preview-3d.md` | 0.7.4 live on the Shopper parent since 10 Nov 2026 (includes 0.7.3); 0.7.5 built, not released. |
| Keepsake Studio | `keepsake-studio.md` | live, version 0.6.3 (as of the CHANGELOG; snippet 0.6.0). Snippet: `product/keepsake-studio`. Mount option: `ks_mount` (hidden Text template option, custom_script field). P |
| Bookmark Studio | `bookmark-studio.md` | live, version 0.2.1 (as of the CHANGELOG). Snippet: `product/bookmark-studio`. Mount option: `bs_mount` (hidden Text template option, custom_script field). Prefix: `bs_`. |

## New studios (built, not yet on customer sites)

| Tool | Page | Status |
|---|---|---|
| Golf Ball Studio | `golf-ball-studio.md` | 1.0.0, built 23 Nov 2026. Being installed on experience.pixfizz.com as the demo; not yet on a customer site. Snippet: `product/golf-ball-studio`. Mount option: `gb_mount`. Prefix:  |
| Mug Studio | `mug-studio.md` | 0.2.4, built 19 Nov 2026, not yet installed on a site. Snippet: `product/mug-studio`. Mount option: `ms_mount`. Prefix: `ms_`. |
| T-shirt Studio | `tshirt-studio.md` | 0.2.3, built 23 Nov 2026, not yet installed on a customer site. Snippet: `product/tshirt-studio` (product page) and `product/tshirt-studio-page` (a studio page). Mount option: `ts_ |
| Jewelry Box | `jewelry-box.md` | 0.1.3, built and tested on baseline 18 Nov 2026; demo install on experience; not yet on a customer site. Snippet: `product/jewelry-box`. Mount option: `px_mount`. Prefix: `jb_`. Ne |
| Starring You | `starring-you.md` | 0.1.x, prototype and demo on experience.pixfizz.com (Nov 2026), not on customer sites. Snippet: `product/starring-you`. Mount option: `sy_mount`. |
| AI switches in the studios | `ai-switches.md` | Several studios can use the platform AI image tools: cut a person out of their background, turn a photo into a cartoon, color-correct a photo. **Every AI feature is a switch in the |

## Changelog

- 2026-11-24: Folder created from `28_TOOL_SETTINGS_REFERENCE.md` section 6. Source: claude-chat, tool source.
