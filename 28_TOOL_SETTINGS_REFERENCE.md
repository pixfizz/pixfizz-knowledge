# 28 — Tool Settings Reference (the mount)

**Authority Scope:** customer-facing reference for the settings of every Pixfizz studio,
custom design tool, order builder, product preview and platform extension: where the
settings live, what each one does, which values it accepts, and whether a lab can safely
change it. Template-level (Shopper 24) unless a row says otherwise.

**Not in scope:** how a tool is built, versioned or installed (`26_CUSTOM_DESIGN_TOOLS.md`),
the platform Design Tool's own feature toggles (`17_DESIGN_TOOL.md`), Live Finish and 3D
preview internals (`27_LIVE_FINISH_AND_3D_PREVIEWS.md`), pricing formulas
(`30_PRICING_ENGINE.md`).

_Last updated: 2026-11-24_

---

## 1. Where a tool's settings live

Every studio and custom design tool on a Shopper site reads its settings from one place:
**the mount**. The mount is a short block of Liquid in the `custom_script` field of one
hidden template option on the product's template. It calls the tool and passes it its
settings, one per line:

```liquid
{% snippet 'product/gang-up',
	gu_roll_width: '22in',
	gu_min_dpi: '300',
	gu_cut: 'true',
	gu_cut_spot: 'CutContour',
	gu_title: 'Custom DTF Gang Sheet'
%}
```

- The first line names the tool (`product/<tool>`). Never change it.
- Each following line is one setting: `name: 'value',`. The last setting has no comma,
  and the block closes with `%}`. (This example is shortened; a real mount passes every
  setting the tool lists.)
- The option holding the mount is hidden and hidden from the cart. On tools built from
  20 September 2026 its code is `px_mount` and its name is "<Tool> Mount". Older installs
  use `<prefix>_mount` (for example `bc_mount`, `du_mount`) or, on the oldest, the tool's
  artwork option (for example `bc_artwork_front`). Section 6 lists the option per tool.

The tool code itself (the snippet and the script it loads) lives on the shared Shopper
parent and is maintained by Pixfizz. You change only the values in your mount.

**Why the mount and not a product setting.** A template export carries its template
options with it, so the mount travels with the template to any site it is imported on.
Product custom fields, design custom fields and site checklists do not travel, and a tool
cannot read the product from inside its mount. So a setting you add as a product custom
field is ignored. Platform-level.

### Settings are per template, not per product

A mount sits on a template, so it applies to **every product that uses that template**.
To give two products different settings (for example two trim sizes), they need two
templates, each with its own mount. Before changing a mount, check which products use the
template.

### Settings are not shopper choices

| | What it is | Where it lives |
|---|---|---|
| **Settings** | how the tool behaves for this product: trim size, bleed, minimum dpi, output mode, which controls are on, how it is displayed | the mount |
| **Shopper choices** | everything the shopper picks and everything that carries a price: sides, paper, finish, size, binding, quantity | **variants** on the Product Attribute |
| **Price** | what the shopper pays | the product price, variant prices, pricing formula, or the `px_rates` rate card |
| **Page size and print area** | the production file geometry of a design product | the XML template definition |

So a new paper or a new price is never a mount change. Where a tool has a setting ending
in `_code`, that setting only tells the tool which variant or option **code** to read, for a
site that named it differently from the standard.

## 2. Can I change it? The three kinds of setting

Every setting in section 6 is marked with one of these:

- **Yes**: display and behavior. Safe to change at any time: how the tool opens (modal or
  inline), whether a preview shows, labels, whether a step is shown. The worst result of a
  wrong value is a tool that looks or behaves differently from what you wanted.
- **With care**: affects the print file or the price. The value must match something
  outside the mount: the XML template (page size, bleed), your production spec (dpi, output
  mode, cut path), or your pricing. Change it together with that other thing, and place a
  test order before going live. A wrong value here produces a wrong print file, which may
  not show on screen.
- **No**: a code that must match a platform object (a variant, option or print code), or
  an internal value. Leave it as installed. The only reason to change one is that you
  renamed that object on your site, and then the value must be the new code exactly.

## 3. How to change a setting safely

1. In the admin, open **Templates** (`https://admin.pixfizz.com/site/<site>/admin/print_theme/index`),
   open the template, and open its mount option (see section 6 for the code).
2. In **Custom Fields**, edit only the value between the quotes on the line you are
   changing. Keep the quotes, the commas and the closing ` %}`.
3. Save with the **Custom Fields Save** button on the option page, not the Save button at
   the top of the page.
4. Open the product on your storefront and check the change. For a "With care" setting,
   place a test order and check the print file.
5. To roll the same change across many templates, change and test one first, then ask
   Pixfizz to copy the option to the rest. Never delete and re-import live products to
   change a setting.

Rules for mount values (platform-level):

- **Pass every setting, every time**, even when the value is the default. A tool lists any
  setting the mount did not pass and reports it as an error in the browser console. The tool
  still works on its built-in default, but a missing setting is treated as a mistake, not as
  a choice.
- **Never use an apostrophe** inside a value. Pixfizz Liquid cannot escape it, and the whole
  mount stops working. Write "Your logo" rather than "You're ready".
- **Lengths carry their unit**: `'3.5in'`, `'89mm'`. Tools accept inches and millimeters.
  Write the value as your production spec gives it.
- **Numbers and true/false go in quotes too**: `'300'`, `'true'`.
- **Accepted values are case sensitive** where section 6 shows them in capitals
  (`NORMALISED`, `AUTO`).
- **Blank on purpose.** A few settings may be passed as `''` when the mount cannot know the
  value (for example Business Cards sides, which differs per product on one template). The
  tool then reads it from the shopper's variant choice. Section 6 says where this is allowed.

### Products from the Pixfizz template library

If a product came to your site from the Pixfizz template library (Products → Publish
Products), its template, and so its mount, lives on the Pixfizz library site, not on yours.
You cannot edit it on your site; the change would apply to every store using that template.
Ask Pixfizz support for a site-specific copy if you need different settings.

### When a tool misbehaves after a change

Open the product page, open the browser console, and look for the tool's start-up line. It
lists the version, every setting the tool resolved, where each value came from, and any
setting the mount did not pass. Send that line to Pixfizz support with the product URL; it
is the first thing support will ask for.

## 4. Shared setting names

Settings that mean the same thing use the same name in every tool, after the tool's
prefix (`bc_trim_w`, `gu_min_dpi`):

| Name | Meaning |
|---|---|
| `trim_w`, `trim_h` | finished (trimmed) width and height, with unit |
| `bleed` | extra image beyond the trim on each side, with unit |
| `safe` | margin inside the trim that important content should stay within, with unit |
| `min_dpi` | lowest resolution the tool accepts or warns below |
| `max_mb` | largest upload size in MB |
| `output` | how the print file is produced (for example `NORMALISED` rebuilds the file to spec, `PASSTHROUGH` sends the customer file as uploaded) |
| `placement` | how artwork is fitted to the trim (`AUTO`, `FILL`, `FIT`, ...) |
| `units` | units shown to the shopper (`in` or `mm`); display only |
| `display` | how the tool opens on the page (`modal` or `inline`) |

## 5. Asking Claude about a tool's settings

With this knowledge base connected, ask in plain words and paste your mount:

- "Here is the mount on my business card template. Which of these can I change myself, and
  what would changing `bc_output` to PASSTHROUGH do?"
- "I want the Greeting Card Studio to open inline instead of in a pop-up. Which setting is
  that and what value?"
- "My sticker tool says a setting is missing in the console. Here is the line. What do I
  add to the mount?"

Claude answers from section 6. It will not invent a setting that is not listed there; if a
behavior you want has no setting, it is a change Pixfizz makes in the tool, so raise it in
myPixfizz.

---

## 6. Per-tool settings

From 24 Nov 2026 each tool has its own page in the folder `tool-settings/`, with its current version, what changed since 3 Nov 2026 and the full settings table. This section lists them. Sections 1 to 5 above apply to every page. Your installed mount may be on an older version; the console start-up line shows which.

## 6.1 Studios and design tools

### Business Cards
3.7.2 is the newest release (name cards, Nov 2026). Which version each site runs is not checked here: the console start-up line shows it.
Settings: `tool-settings/business-cards.md`.

### Greeting Card Studio
0.7.3 on the Shopper parent; 0.7.9 on My Picture Perfect; 0.7.11 and 0.7.12 built, not live (24 Nov 2026).
Settings: `tool-settings/greeting-card-studio.md`.

### Calendar Studio
0.5.2 live on the Shopper parent since 20 Nov 2026.
Settings: `tool-settings/calendar-studio.md`.

### Collage Studio
Status: beta, version 1.1.0 built 3 Nov 2026 (as of the release notes; the repo marks every release folder unreleased, so live status is not verified). Snippet: `product/collage-studio`. Mount: not a template option. The studio sits on its own Pages page, whose page content is only the snippet tag. Prefix: `col_`.
Settings: `tool-settings/collage-studio.md`.

### Puzzle Studio
0.6.0 live on the Shopper parent since 5 Nov 2026.
Settings: `tool-settings/puzzle-studio.md`.

### Stationery Studio
0.4.0 live on the Shopper parent since 19 Nov 2026.
Settings: `tool-settings/stationery-studio.md`.

### Creative Studio (Make your own notebook cover)
0.1.5 live on the Shopper parent (5 Nov 2026); 0.1.8 built.
Settings: `tool-settings/creative-studio.md`.

### Pet Studio
Status: live, version 0.4.1 (as of the CHANGELOG; snippet 0.4.1, asset 0.4.0). Snippet: `product/pet-studio`. Mount option: `px_mount` (hidden Text template option, one per template). Prefix: `ps_`.
Settings: `tool-settings/pet-studio.md`.

### Photo Book Studio
4.0.5 live on the Shopper parent since 19 Nov 2026; 4.0.6 (Serbian wording) on one site only.
Settings: `tool-settings/photo-book-studio.md`.

### Cover Studio
Status: in test, version 1.8.3 (asset; style updates numbered 1.9.1, as of the CHANGELOG). Snippet: `product/cover-studio`. Mount: `{% snippet 'product/cover-studio' %}` with no arguments, in the `custom_script` field of an imprint template option on each album template (there is no `cvs_mount` option yet). Prefix: `cvs_`.
Settings: `tool-settings/cover-studio.md`.

### Custom Sized Prints
Status: live, version 0.4.1 on the shopper24 parent; 0.5.0 (metric) built, not yet released. Snippet: `product/custom-print`. Mount option: `px_mount` (hidden Text template option, one per template). Prefix: `cp_`.
Settings: `tool-settings/custom-sized-prints.md`.

### TilePix Wall Studio (Wall Designer)
0.6.12 live on the Shopper parent since 9 Nov 2026. Asset-only releases: mounts unchanged.
Settings: `tool-settings/tilepix-wall-studio.md`.

### Wall Design Studio (metal and acrylic walls)
0.8.9 live on the Shopper parent since 9 Nov 2026.
Settings: `tool-settings/wall-design-studio.md`.

### Fan Face (face fan designer)
1.3.1 live on one pilot site (23 Nov 2026). Older versions on other installs. 2.0.0 built and tested, not released.
Settings: `tool-settings/fan-face.md`.

### Custom Framing
Status: built, version 0.4.0; not yet moved to the mount standard. Snippet: `product/framing`.
Settings: `tool-settings/custom-framing.md`.

## 6.2 Print and cut tools

### Sticker Studio
6.3.0 live on the Shopper parent since 3 Nov 2026; 6.4.0 built, not released.
Settings: `tool-settings/sticker-studio.md`.

### Sticker Designer (legacy)
5.31 (4 Nov 2026). Being retired: sites move to Sticker Studio.
Settings: `tool-settings/sticker-designer-legacy.md`.

### Gang Up (DTF gang sheet)
2.2.0 live; 2.3.0 built, not released (24 Nov 2026).
Settings: `tool-settings/gang-up.md`.

## 6.3 Upload tools

### Uploader 2.0 (documents, booklets, forms, flat prints)
Status: live, version 2.6.0 (per the release notes; this folder has no CHANGELOG). Replaces Document Uploader and Booklet Uploader on new products. Snippet: `product/uploader`. Mount option: `px_mount`. Prefix: `up_`.
Settings: `tool-settings/uploader-2.md`.

### Document Uploader
Status: live, version 1.3.3 (per the CHANGELOG). To be replaced by Uploader 2.0. Snippet: `product/document-uploader`. Mount option: `du_mount`. Prefix: `du_`.
Settings: `tool-settings/document-uploader.md`.

### Booklet Uploader
Status: live, version 1.5.2 (per the CHANGELOG). To be merged into Uploader 2.0. Snippet: `product/booklet-uploader`. Mount option: `bu_mount`. Prefix: `bu_`.
Settings: `tool-settings/booklet-uploader.md`.

### Publication Upload
1.16.0 released 4 Nov 2026 (upload standard).
Settings: `tool-settings/publication-upload.md`.

### Flyer / Brochure
1.0.4 live (10 Nov 2026); 1.1.0 built, not released.
Settings: `tool-settings/flyer-brochure.md`.

## 6.4 Order builders

### Design Service (Design Brief)
Status: beta, version 2.3.1 (live on the shopper24 parent; first client installs October 2026). Snippet: `product/design-brief`. Mount option: `px_mount` on new installs (older installs: `dsn_mount`). Prefix: `dsn_`.
Settings: `tool-settings/design-service.md`.

### Film Order Builder
Status: in progress, version 0.2.0, no client installs. Snippet: `product/film-builder`, called from a page (not a template option mount). Prefix: `pxfb_`.
Settings: `tool-settings/film-order-builder.md`.

### Film Roll Builder
Status: phase 1, version 0.1.1, live on the demo site. Snippet: `film/roll-builder`, mounted on a Pages page whose content is only `{% snippet 'film/roll-builder' %}`. Prefix: `frb_`.
Settings: `tool-settings/film-roll-builder.md`.

## 6.5 Product previews

### Live Finish
0.29.0 live on the Shopper parent since 17 Nov 2026; 0.29.1 (wording in the store's language) built for one site.
Settings: `tool-settings/live-finish.md`.

### Live Finish Plates
0.5.0 live; 0.6.0 built for the Jewelry Box, not on the parent yet.
Settings: `tool-settings/live-finish-plates.md`.

### 3D Preview
0.7.4 live on the Shopper parent since 10 Nov 2026 (includes 0.7.3); 0.7.5 built, not released.
Settings: `tool-settings/preview-3d.md`.

### Keepsake Studio
Status: live, version 0.6.3 (as of the CHANGELOG; snippet 0.6.0). Snippet: `product/keepsake-studio`. Mount option: `ks_mount` (hidden Text template option, custom_script field). Prefix: `ks_`.
Settings: `tool-settings/keepsake-studio.md`.

### Bookmark Studio
Status: live, version 0.2.1 (as of the CHANGELOG). Snippet: `product/bookmark-studio`. Mount option: `bs_mount` (hidden Text template option, custom_script field). Prefix: `bs_`.
Settings: `tool-settings/bookmark-studio.md`.

## 6.6 Platform extensions

Kiosk Mode, Prints Review and the Size Picker have no template, so they have no mount. Their switches and settings are site checklist keys under Custom Admin, set up by Pixfizz when the extension is activated. A per-key reference for them is planned for a later update of this file; until then, ask Pixfizz support before changing one.

Upsell Studio is a platform extension too: its switch is the child snippet `admin/checklist/upsell-studio-enabled` (`TRUE` on, blank off) and its settings are the JSON in the child snippet `upsell-studio/rules`; the full reference is `26_CUSTOM_DESIGN_TOOLS.md` § 7 Upsell Studio. When you change the rules: read the snippet first, keep the JSON valid, change a percent or a category in the rules and in the matching Automatic Discount together, never rename a matched category or value, and keep the mount lines last in the hook snippets. *Verified by reading source and live test, 2026-11-18.*

## 6.7 Not configurable yet

Specified or in prototype, with no settings to change: Acrylic Statue Designer, Collage Wall, Team Cards, Restoration Estimator.

---

## 6.8 New studios (built, not yet on customer sites)

### Golf Ball Studio
1.0.0, built 23 Nov 2026. Being installed on experience.pixfizz.com as the demo; not yet on a customer site. Snippet: `product/golf-ball-studio`. Mount option: `gb_mount`. Prefix: `gb_`.
Settings: `tool-settings/golf-ball-studio.md`.

### Mug Studio
0.2.4, built 19 Nov 2026, not yet installed on a site. Snippet: `product/mug-studio`. Mount option: `ms_mount`. Prefix: `ms_`.
Settings: `tool-settings/mug-studio.md`.

### T-shirt Studio
0.2.3, built 23 Nov 2026, not yet installed on a customer site. Snippet: `product/tshirt-studio` (product page) and `product/tshirt-studio-page` (a studio page). Mount option: `ts_mount`. Prefix: `ts_`.
Settings: `tool-settings/tshirt-studio.md`.

### Jewelry Box
0.1.3, built and tested on baseline 18 Nov 2026; demo install on experience; not yet on a customer site. Snippet: `product/jewelry-box`. Mount option: `px_mount`. Prefix: `jb_`. Needs Live Finish Plates 0.6.0 or later on the parent.
Settings: `tool-settings/jewelry-box.md`.

### Starring You
0.1.x, prototype and demo on experience.pixfizz.com (Nov 2026), not on customer sites. Snippet: `product/starring-you`. Mount option: `sy_mount`.
Settings: `tool-settings/starring-you.md`.

### AI switches in the studios
Several studios can use the platform AI image tools: cut a person out of their background, turn a photo into a cartoon, color-correct a photo. **Every AI feature is a switch in the tool's mount, off by default** (or offered only when you list it). A mount without the switch behaves exactly as before and loads no AI files. Platform-level rule, from Nov 2026.
Settings: `tool-settings/ai-switches.md`.

## Changelog

- 2026-11-03: File created. Customer-facing settings reference for every studio, custom design tool, order builder and product preview: where the mount lives, how to change it safely, the Yes / With care / No rating, and the full settings list per tool. Settings, accepted values and defaults verified by reading the tool source in the design tools repo, 3 Nov 2026; items not confirmed are marked per tool.
- 2026-11-09: Business Cards designs file per trim size. Collage Studio known issues (frame wider than header on Bootstrap-breakpoint themes, hide the new design collection). Photo Book Studio collections settings only in `#pbs-config` from 3.4.0, 3D-only landing hero from 4.0.1 (CORRECTED: `hero3d` block no longer needed). Uploader 2.0 reads product `px_rates` before the mount card. Flyer / Brochure 1.0.x limits (no child variant trees, carriers not hidden, `ff_original_code` optional). Keepsake notes on `ks_photo_option` as a design option and `ks_front` reading variants only. Round 2: CORRECTED Keepsake `ks_sample_page` (a missing page name falls back to `ks_page` with the sample photo); placeholder grey differs per site.
- 2026-11-21: Greeting Card Studio full mount required from 0.6.41 (`gcs_diy`, `gcs_env_logo_ink`). Photo Book Studio `"layflat":"false"` for own page books (4.0.4), Linked Layouts, and the `no 3D hero images (heroPlate)` console line. Uploader 2.0 `?size=` link to a size (2.6.6). § 6.6 Upsell Studio switch and rules snippets.
- 2026-11-24: Section 6 split into one page per tool under `tool-settings/` (index: `tool-settings/README.md`). Each page adds the current version and the settings changed since 3 Nov 2026, checked against the tool source: Greeting Card Studio `gcs_editor` and quantity and proof settings, Calendar Studio design tool link off by default, Fan Face 1.3 and 2.0 (AI cut as a switch), Gang Up `gu_cut_shape`, Sticker Studio 6.4 bleed default, Live Finish framed ornament keys, 3D Preview inside color keys. New pages: Golf Ball Studio, Mug Studio, T-shirt Studio, Jewelry Box, Starring You, AI switches. Per-tool facts now go in the tool page, not here.
