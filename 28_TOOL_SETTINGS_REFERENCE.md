# 28 — Tool Settings Reference (the mount)

**Authority Scope:** customer-facing reference for the settings of every Pixfizz studio,
custom design tool, order builder, product preview and platform extension: where the
settings live, what each one does, which values it accepts, and whether a lab can safely
change it. Template-level (Shopper 24) unless a row says otherwise.

**Not in scope:** how a tool is built, versioned or installed (`26_CUSTOM_DESIGN_TOOLS.md`),
the platform Design Tool's own feature toggles (`17_DESIGN_TOOL.md`), Live Finish and 3D
preview internals (`27_LIVE_FINISH_AND_3D_PREVIEWS.md`), pricing formulas
(`30_PRICING_ENGINE.md`).

_Last updated: 2026-11-03_

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

Each tool below lists its status and version as of the last update of this file, an example mount with generic values, and every setting the tool reads. Your installed mount may be on an older version; the console start-up line shows which.

## 6.1 Studios and design tools

### Business Cards
Status: live, version 3.6.0 (latest built, per the CHANGELOG; 3.x is the current tool). Snippet: `product/business-cards`. Mount option: `px_mount`. Prefix: `bc_`.
The shopper uploads print-ready card artwork or starts from one of the designs, types their details, and sees a trimmed proof (front and back) before ordering. The tool builds the production print file.

Example mount:
```liquid
{% snippet 'product/business-cards',
	bc_units: 'in',
	bc_trim_w: '3.5in',
	bc_trim_h: '2in',
	bc_bleed: '0.125in',
	bc_safe: '0.125in',
	bc_xml_w: template.width,
	bc_xml_h: template.height,
	bc_xml_min_dpi: template.minimum_dpi,
	bc_min_dpi: '300',
	bc_max_mb: '100',
	bc_sides_variant: 'bc_sides',
	bc_stock_variant: 'bc_stock',
	bc_corners_variant: 'bc_corners',
	bc_corner_radius: '0.125in',
	bc_designs: 'true',
	bc_title: 'Business cards',
	bc_intro: 'Upload your print-ready file, or start from one of our designs and type your details.',
	bc_trust: 'Bleed added for you|Proof before you order'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `bc_units` (required) | Units shown to the shopper, and the unit of any length typed without one | `in` or `mm` | `in` | **Yes** |
| `bc_trim_w`, `bc_trim_h` (required) | Finished card width and height | A length, e.g. `3.5in` or `89mm` | none (required) | **With care**: must match the XML template page size and your cutter. Ignored when `bc_xml_w` / `bc_xml_h` are set |
| `bc_bleed` (required) | Bleed added around the trim | A length, e.g. `0.125in` or `3mm` | none (required) | **With care**: must match your production bleed |
| `bc_safe` (required) | Safe margin inside the trim (the dashed line) | A length | none (required) | **With care**: match your production spec |
| `bc_xml_w`, `bc_xml_h`, `bc_xml_min_dpi` | Reads size and minimum dpi straight from the template XML; when set they win over the trim and dpi settings | Leave as `template.width`, `template.height`, `template.minimum_dpi` | blank | **No**: leave as shown so the tool always follows the XML |
| `bc_min_dpi` (required) | Minimum resolution for an uploaded print file | A number, e.g. `300` | none (required) | **With care**: your print quality bar |
| `bc_low_dpi` | Below this, a photo or logo placed in a design gets a "may look soft" warning | A number | `100` | **Yes** |
| `bc_max_mb` (required) | Largest file a shopper can upload | A number in MB | none (required) | **Yes** |
| `bc_sides_variant`, `bc_stock_variant`, `bc_corners_variant` (required) | Which product variant codes carry printed sides, paper and corners | Variant codes | none (required) | **No**: must match your variant codes exactly. Change only if you named those variants differently |
| `bc_corner_radius` (required) | Corner radius used for a corners value that does not state one (e.g. `rounded`) | A length | none (required) | **With care**: match your rounding die |
| `bc_designs` (required) | Shows the "start from one of our designs" library | `true` or `false` | none (required) | **Yes** |
| `bc_designs_file` | Which designs file to load | An asset name | `business-cards-designs.json` (square cards use `business-cards-designs-25x25.json` or `business-cards-designs-2x2.json`) | **With care**: the file must match the card size. A designs file is per trim size: from 3.2.0 the tool refuses a file whose width x height does not match the card trim, and a finished file is never scaled to another size (the bleed extension is applied per size, so each size is generated from the source design). *Verified by reading source, 2026-11-02.* |
| `bc_designs_more_file` | Your own designs file, listed before the standard designs | An asset name on your site | blank | **Yes** |
| `bc_designs_more_assets` | The image assets your own designs file uses | Asset names separated by `\|` | blank | **Yes** (must list every asset that file uses) |
| `bc_size_links` | Shows a Card size switch linking sibling products | `3.5x2=<url>\|2.5x2.5=<url>` (path or product url_path) | blank (no switch) | **Yes** |
| `bc_qtys` | Quantity choices, only when quantity is not already a list on the product | Numbers separated by commas or spaces | blank | **With care**: must match your pricing |
| `bc_title` | Heading at the top of the tool | Text, no apostrophes | `Business cards` | **Yes** |
| `bc_intro` | Intro line under the heading | Text, no apostrophes | A standard intro | **Yes** |
| `bc_trust` | Short reassurance lines in the header | Lines separated by `\|` | `Bleed added for you\|Proof before you order` | **Yes** |
| `bc_look` | Extra style classes on the tool | Class names, e.g. `pxt-flyer` | blank | **Yes** |
| `bc_owns_page` | Whether the tool takes over the product page layout | `true` or `false` | `true` | **Yes** |
| `bc_print_code`, `bc_preview_code`, `bc_original_code`, `bc_spec_code` | Template options the print file, preview, original upload and job spec are written to | Option codes | `px_print_file`, `px_preview`, `px_original`, `px_spec` | **No**: your fulfillment reads these. Change only if your template uses other codes |
| `bc_add_to_cart_selector` | Which button counts as Add to Cart, on a heavily customized page | A CSS selector | blank (standard button) | **No**, unless Pixfizz support asks you to |

Lengths accept inches and millimeters (`3.5in`, `89mm`); a bare number uses `bc_units`.

Legacy Business Cards 1.x (`product/business-card`, singular, mounted on the `bc_artwork_front` option) is a separate upload-only tool still on some sites and being retired. Its settings are `bc_sides`, `bc_units`, `bc_trim_w`, `bc_trim_h`, `bc_bleed`, `bc_safe`, `bc_min_dpi`, `bc_max_mb`, `bc_output` (`NORMALISED` or `PASSTHROUGH`), `bc_placement` (`AUTO`, `FILL`, `FIT`, `TRIM`, `NONE`), `bc_show_bleed`, `bc_preview`, `bc_reposition`, `bc_price_footer` (`off`, `total`, `each`), `bc_requires_design`, `bc_block_on_error`, `bc_designer_link`, `bc_human_check`, `bc_display` (`modal` or `inline`), plus the code settings `bc_sides_code`, `bc_shape_code`, `bc_corners_code`, `bc_stock_code`, `bc_print_code`. If your site still runs 1.x, ask Pixfizz about moving to 3.x rather than tuning it.

Things that are not set here
- Paper, sides, corners and quantity are variants on the product: they carry the shopper's choice and the price.
- Prices live in the product's pricing, not the mount.
- Page size and minimum dpi live in the template XML (the mount reads them through `bc_xml_*`).

### Greeting Card Studio
Status: live, version 0.6.13 (as of the CHANGELOG). Snippet: `product/greeting-card-studio`. Mount: not a template option. The studio sits on its own Pages page, whose page content is only the snippet tag. Prefix: `gcs_`.
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

Things that are not set here:
- Paper, finish, corner and envelope names, descriptions and prices: the product variants.
- Card prices: the products and their pricing (the studio asks the platform for every price).
- The designs themselves and their artwork: the design collection.
- Card size: the card products and their templates, not the mount.

Not yet confirmed: Defaults come from the tool documentation and are not confirmed against the tool script.

### Calendar Studio
Status: live, version 0.2.2 on the platform; 0.3.0 built and not yet released (as of the CHANGELOG). The settings are the same in both. Snippet: `product/calendar-studio`. Mount: not a template option. The studio sits on its own Pages page, whose page content is only the snippet tag. Prefix: `cs_`.
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

### Collage Studio
Status: beta, version 1.1.0 built 3 Nov 2026 (as of the release notes; the repo marks every release folder unreleased, so live status is not verified). Snippet: `product/collage-studio`. Mount: not a template option. The studio sits on its own Pages page, whose page content is only the snippet tag. Prefix: `col_`.
One collage poster product instead of one per size and photo count. The shopper adds photos first, the studio generates a layout for them, and the shopper picks size, material, background and title before adding it to the cart.

Example mount (page content of the studio page):
```liquid
{% snippet 'product/collage-studio',
	col_units: 'in',
	col_materials: 'none',
	col_products: '8x10:00000,11x14:00000,16x20:00000,20x30:00000',
	col_collection: 'collage-studio',
	col_design: 'col-blank',
	col_finish_variant: 'finish',
	col_mount_variant: 'none',
	col_min_cell: '2.5in',
	col_min_dpi: '150',
	col_max_photos: '60',
	col_layouts: 'fit,rows,grid,hero,instant,film,heart',
	col_backgrounds: 'white,ivory,kraft,sage,navy,black,custom',
	col_room: 'sofa',
	col_fonts: 'serif:Fraunces Regular,script:Caveat Medium,sans:DM Sans SemiBold,line:DM Sans Medium',
	col_login_url: '/site/login',
	col_exit_url: '/',
	col_exit_label: 'Shop' %}
```
Replace each `00000` with the product id of that size on your site.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `col_units` | How sizes and lengths are shown, and the unit of a size written without one | `in`, `mm` | `in` | **Yes** |
| `col_materials` | Materials offered, in order; the first is the default. With more than one, the shopper gets a Material step | `none`, or `<code>:<label>[:<kind>[ <depth>]]`, comma separated. Kind `print`, `canvas`, `metal` or `acrylic` (for example `poster:Poster,cv20:Canvas 2 cm:canvas 20mm`) | `none` | **With care**: every material needs its own size products in `col_products` |
| `col_products` | The size products the collage can be ordered on | `[material/]size:product id` pairs, comma separated. A size is `<w>x<h>` with an optional unit (`in`, `mm`, `cm`) or a name such as `A3`, sized by its template. With materials, every entry starts with its material code | none: no sizes | **With care**: ids must be your real products. The template definition always wins over the size you write |
| `col_collection` | Design collection(s) holding the blank collage designs | Collection name(s), comma separated | `collage-studio` | **With care**: must name collections on your site holding those designs |
| `col_design` | Design code suffix. The studio uses `<template code>-<col_design>` on each product | A code suffix | `col-blank` | **No**: must match your blank design codes |
| `col_finish_variant` | Variant type code(s) of the paper or finish on the size products | A code, several codes comma separated (each product uses the first it has), or `none` | `none` | **No**: must match your variant type codes |
| `col_mount_variant` | Variant type code(s) of the mounting | A code, several codes, or `none` | `none` | **No**: must match your variant type codes |
| `col_min_cell` | Smallest a photo may be printed, on its short side | A length, `in` or `mm` (also `cm`), for example `2.5in` or `60mm` | `2.5in` | **Yes** |
| `col_min_dpi` | Below this, a photo is flagged as low resolution | A number, minimum 50 | `150` | **Yes** |
| `col_max_photos` | Most photos in one collage | A number from 2 to 120 | `60` | **Yes** |
| `col_layouts` | Layouts offered, in this order | Any of `fit`, `rows`, `grid`, `hero`, `instant`, `film`, `heart`, comma separated | all seven | **Yes** |
| `col_backgrounds` | Background swatches offered | Any of `white`, `ivory`, `kraft`, `sage`, `navy`, `black`, `custom`, comma separated | all seven | **Yes** |
| `col_room` | Room scene for the On the wall preview | `sofa`, or `none` | `sofa` | **Yes** |
| `col_fonts` | Title fonts, by site font name | `serif:<name>,script:<name>,sans:<name>,line:<name>` | `serif:Fraunces Regular,script:Caveat Medium,sans:DM Sans SemiBold,line:DM Sans Medium` | **With care**: each name must be a font available on your site, or the print will not match |
| `col_login_url` | Sign-in page used by the upload window | A site path | `/site/login` | **Yes** |
| `col_exit_url` | Where the breadcrumb goes | A site path | `/` | **Yes** |
| `col_exit_label` | Breadcrumb text | Text | `Shop` | **Yes** |

Every setting should be in the mount. One that is left out is listed in `data-config-missing` on `#col-root` and in a console error, and the default is used. A value the studio does not recognize is reported in the console and left out. Never put an apostrophe in a value. Lengths accept in and mm.

Things that are not set here:
- Print size, bleed and unit: the template XML definition of each size product.
- Finish and mounting names and prices, and collage prices: the product variants and pricing.
- `?material=<code>` on the page link opens the studio on that material; that is a link, not a mount setting.

Known issues
- **The studio can be wider than header and footer on some themes.** The studio frame uses a fixed 1400 px maximum width with 16 px padding at every width. On a theme whose `.container` follows the Bootstrap breakpoints (540 / 720 / 960 / 1400 px), the studio is up to 24 px wider than the header and footer between 576 and 1199 px. Site fix used, in the studio's Pages page: `@media (min-width:576px){body .col-root .wrap{max-width:540px}}`, and the same with 720, 960 and 1400 px at 768, 992 and 1200 px. A tool fix (size the frame from the site's `.container`) is wanted. Tool-level. *Verified by query on a lab site, 2026-11-04.*
- **Hide the studio's design collection as soon as you create it.** A new collection (for example the one holding the studio's blank designs) shows at once as a card in collection grids. Set the collection's `unpublished` custom field right after creating it; the studio still reads the collection through `/v1/theme_categories`. Template-level (Shopper). *Verified by query on a lab site, 2026-11-04.*

### Puzzle Studio
Status: live, version 0.2.0 on the platform; 0.4.0 built and not yet released (as of the CHANGELOG). Snippet: `product/puzzle-studio`. Mount option: `px_mount` (hidden Text template option on every puzzle template of the range). Prefix: `pz_`.
The shopper picks a puzzle size and shape, adds one photo or a collage of up to eight, chooses pieces and material, sees the finished puzzle in Live Finish, and adds it to the cart.

The same mount also calls `product/live-finish`, before the Puzzle Studio call. Those `lf_` settings are documented under Live Finish.

Example mount (Puzzle Studio part):
```liquid
{% snippet 'product/puzzle-studio',
	pz_items: '5x7=00000 8x10=00000 11x14=00000 16x20=00000 circle=00000 hexagon=00000',
	pz_collection: 'puzzles',
	pz_max_photos: '8',
	pz_min_dpi: '100',
	pz_material_notes: 'on',
	pz_pickup: 'Ready for pickup in store.',
	pz_range: 'Photo Puzzle',
	pz_units: 'in' %}
```
Replace each `00000` with the design id (or design code) of that puzzle on your site.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `pz_items` | The puzzle range: which sizes and shapes the studio offers, and the design behind each | `key=design id or design code`, space separated. The key names the size (`5x7`, `8x10`, `circle`, `hexagon`; with `pz_units: 'cm'` a paper name such as `A4`) | required: none. Missing is listed in `data-config-missing` | **With care**: ids must be your real puzzle designs. Removing an entry removes that size |
| `pz_collection` | Collection holding the range | A collection name | the collection each design is published in | **Yes**: must exist on your site |
| `pz_max_photos` | Most photos in one collage | `1` to `8` (circle and hexagon take at most 4) | `8` | **Yes** |
| `pz_min_dpi` | Low resolution warning below this | A number | `100` | **Yes** |
| `pz_material_notes` | One-line notes under each material | `on`, `off` | `on` | **Yes** |
| `pz_pickup` | One sentence under About, for example where to pick the order up | Text, no apostrophe | empty | **Yes** |
| `pz_range` | Word after the size in the title | Text | `Photo Puzzle` | **Yes** |
| `pz_units` | Size display unit. With `cm`, a paper size key (`A4`, `A3`) is used as the size name (0.4.0) | `in`, `cm` | `in` | **Yes**: match how your sizes are named |

Only `pz_items` is required; the other settings have defaults set in the snippet. Never put an apostrophe in a value or in the mount.

Things that are not set here:
- Product, trim, bleed, resolution, prices and production time: read live from each puzzle design and product.
- Pieces and materials: the template options `pieces` (values `<n>pieces`) and `puzzlematerial` on each size product.
- The finished-puzzle preview (looks, light, room): the `product/live-finish` call in the same mount.

### Stationery Studio
Status: live, version 0.3.3 (as of the CHANGELOG; the snippet header reads 0.3.0 and loads the 0.3.3 asset). Snippet: `product/stationery-studio`. Mount option: `ss_mount` (hidden Text template option, one per stationery template). Prefix: `ss_`.
Notepads, notebooks and postcards. The shopper picks a design, adds photos or a logo, changes the words and cover color, picks binding, pages and quantity, approves a proof and adds it to the cart, or customizes further in the design tool. On notebooks, Make your own opens Creative Studio inside the same page.

Example mount (one per template; this one is an 8.5 x 11 notebook):
```liquid
{% snippet 'product/stationery-studio',
	ss_family: 'notebook',
	ss_size: '8.5x11',
	ss_dpi: '300',
	ss_min_dpi: '150',
	ss_collection: '',
	ss_editor: 'true',
	ss_step: '1',
	ss_money: '$1,234.50',
	ss_creative: 'true',
	ss_bleed: '0.125' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `ss_family` | Kind of stationery this template is | `notepad`, `notebook`, `postcard` | required: none | **No**: must match the template and its design codes |
| `ss_size` | Size of this template, as written in the design codes | `4x5`, `5x5`, `4x10`, `5.5x8.5`, `8.5x11`, `4x6-2s` | required: none | **No**: must match the template and its design codes |
| `ss_dpi` | Resolution of the template XML definition | A number, the dpi in the XML definition | required: none | **With care**: must equal the dpi in the template XML definition |
| `ss_min_dpi` | Photo low resolution warning below this | A number | `150` | **Yes** |
| `ss_collection` | Collection path holding the designs | A collection path, or blank to use the collection in the page URL | blank | **Yes**: must exist on your site |
| `ss_editor` | Offer Customize more in the design tool | `true`, `false` | `true` | **Yes** |
| `ss_step` | Quantity step, for example 4 for postcards sold in fours. The minimum is the product minimum | A number | `1` | **With care**: must match how your pricing sells the product |
| `ss_money` | Money format sample for prices shown in the studio | A formatted amount, for example `$1,234.50` | `$1,234.50` | **Yes**: match your store currency format |
| `ss_creative` | Notebooks only: add the Make your own card (Creative Studio) | `true`, `false` | `false` | **With care**: the collection must hold the design `pxt-notebook-in-<size>-creative` |
| `ss_bleed` | Bleed of the XML definition in inches, for the Make your own print file | A number in inches | `0.125` | **With care**: must equal the bleed in the template XML definition |

`ss_family`, `ss_size`, `ss_dpi`, `ss_min_dpi`, `ss_editor` and `ss_step` must be in every mount; one that is left out is listed in `data-config-missing` on `#ss-root`. Never put an apostrophe in a value or in the mount.

Things that are not set here:
- Designs: found in the collection by code `pxt-<family>-in-<size>-<design>`.
- Binding, pages, paper and their prices: the product variants and pricing.
- Cover colors: built into the designs.
- Print size and bleed: the template XML definition. `ss_dpi` and `ss_bleed` only repeat it for the studio.

### Creative Studio (Make your own notebook cover)
Status: live, version 0.1.2 (as of the Stationery Studio CHANGELOG). Snippet: `product/creative-studio-assets`. Mount: none of its own. Prefix: `crs-` (CSS only).
Runs inside Stationery Studio on notebook pages: the shopper builds a notebook cover from drawings, a layout, a name and colors.

There are no settings for Creative Studio itself. It is switched on per notebook template in the Stationery Studio mount:
```liquid
	ss_creative: 'true',
	ss_bleed: '0.125'
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `ss_creative` (Stationery Studio mount) | Show the Make your own card on this notebook | `true`, `false` | `false` | **With care**: the collection must hold `pxt-notebook-in-<size>-creative` |
| `ss_bleed` (Stationery Studio mount) | Bleed of the Make your own print file, in inches | A number in inches | `0.125` | **With care**: must equal the bleed in the template XML definition |

Things that are not set here:
- Size, binding, pages, quantity and the cart: Stationery Studio and the product variants.
- The drawing library: built into the tool.

### Pet Studio
Status: live, version 0.4.1 (as of the CHANGELOG; snippet 0.4.1, asset 0.4.0). Snippet: `product/pet-studio`. Mount option: `px_mount` (hidden Text template option, one per template). Prefix: `ps_`.
Personalized pet products fulfilled by a print-on-demand supplier: bandanas (four sizes), dog and cat bowls, collars and tags. The shopper picks a design and breed, adds a photo and name, sees a 3D preview and adds it to the cart.

Example mount (a medium bandana template; design codes and ids are placeholders):
```liquid
{%- assign ps_xml_dpi = '' -%}{%- if template.dpi -%}{%- assign ps_xml_dpi = template.dpi -%}{%- endif -%}
{% snippet 'product/pet-studio',
	ps_product: 'bandana',
	ps_size: 'M',
	ps_items: 'S=BANDANA-S#00000 M=BANDANA-M#00000 L=BANDANA-L#00000 XL=BANDANA-XL#00000 dogbowl=dog-bowl#00000 catbowl=cat-bowl#00000 collar=dog-collar#00000 tag=circle-pet-tag#00000',
	ps_size_s: 'pet-bandana-small:7.25x4.125:0.39:7.8-11.8:0.339',
	ps_size_m: 'pet-bandana-medium:9.25x5.5:0.59:10.2-17.3:0.311',
	ps_size_l: 'pet-bandana-large:12.5x6.63:0.79:12.6-19.7:0.303',
	ps_size_xl: 'pet-bandana-xl:15x7.75:0.79::0.291',
	ps_spec_code: 'px_spec',
	ps_dpi: ps_xml_dpi,
	ps_trim: '0.25in',
	ps_safe: '0.5in',
	ps_min_dpi: '150',
	ps_title: 'Pet Bandana with Collar',
	ps_trust: 'bandana:Printed edge to edge on soft white fabric|tag:Glossy aluminum, printed on both sides' %}
```
The first line reads the dpi from the template XML definition, so `ps_dpi` always matches it.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `ps_product` | Which product this template is | `bandana`, `dogbowl`, `catbowl`, `collar`, `tag` | `bandana` | **No**: must match the template |
| `ps_items` | The whole range: the design behind every product and bandana size, so the shopper can switch between them | `key=design code#design id`, space separated. Keys `S`, `M`, `L`, `XL`, `dogbowl`, `catbowl`, `collar`, `tag` | blank: bandana sizes found by `url_path` | **With care**: codes and ids must be the designs on your site. Removing a key removes that product from the switcher |
| `ps_size` | Bandana size of this template | `S`, `M`, `L`, `XL` | required on bandana templates | **No**: must match the template |
| `ps_size_s`, `ps_size_m`, `ps_size_l`, `ps_size_xl` | Each bandana size: page url_path, print size, collar width, fit and band. Put all four on every template of the range | `url_path:WxH:collar:fit:band`, or blank when that size is not offered | blank: size not offered | **With care**: sizes must match the supplier spec and the templates |
| `ps_dpi` | Resolution of the template XML definition | A number. Pass `template.dpi` as in the example | required | **No**: leave it reading the template |
| `ps_trim` | Production buffer inside the page edge | A length, for example `0.25in` | required | **With care**: must match the supplier production spec |
| `ps_safe` | Safe line from the page edge | A length, for example `0.5in` | required | **With care**: must match the supplier production spec |
| `ps_collection` | Collection path holding the other sizes | A collection path | the collection in the page URL | **Yes**: must exist on your site |
| `ps_min_dpi` | Photo low resolution warning below this | A number | `150` | **Yes** |
| `ps_spec_code` | Template option that receives the job record for fulfillment | A template option code | `px_spec` | **No**: must match the option on the template |
| `ps_title` | Heading shown above the studio | Text | the page heading | **Yes** |
| `ps_maker` | Made by line in the product details | Text | empty: line hidden | **Yes** |
| `ps_trust` | Short reassurance lines. A line can name its product (`bandana:...`); unnamed lines show on every product | Lines separated by a vertical bar | empty | **Yes** |
| `ps_money` | Money format sample for prices shown in the studio | A formatted amount, for example `$1,234.50` | `$1,234.50` | **Yes**: match your store currency format |
| `ps_fulfiller` | Supplier name in the strip under the header | Text, or `none` to hide the strip | `Navitor` | **Yes** |
| `ps_fulfiller_logo` | Supplier logo in that strip | Asset name of an image in your site assets, or `none` to show the name as text | `pod-logo-navitor.webp` | **Yes** |
| `ps_range` | Chip next to Pet Studio in the strip | Text | `Bandana` | **Yes** |
| `ps_display` | Display mode | `inline` | `inline` | **No**: read but has no other mode in this version |

`ps_size` (bandana templates), `ps_dpi`, `ps_trim` and `ps_safe` must be in every mount; one that is left out is listed in `data-config-missing` on `#ps-root`. Never put an apostrophe in a value or in the mount. Lengths are written in inches.

Things that are not set here:
- Print size and bleed of each product: the template XML definition (supplier spec).
- Prices: the products and their pricing.
- Designs and breeds: the design collection and the tool itself.

### Photo Book Studio
Status: live, version 3.4.4 (as of the CHANGELOG). Snippet: `product/photo-book-studio`. Mount: not a template option. The studio sits on its own Pages page, whose page content is the snippet tag, optionally followed by a `pbs-config` block. Prefix: `pbs_`.
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

Your own books (advanced). A store that sells its own book lines and designs instead of the standard Marathon Press set switches this on with a JSON block on the same page, after the snippet tag: `<script type="application/json" id="pbs-config">{"collections":{"source":"collections", ...}}</script>`. Inside `collections`, the studio reads `ranges` (required: `<range>:<product ids>` separated by `;`), `rangeNames` and `rangeLines` (separated by a vertical bar), `exclude`, `blankFrom`, `unit`, `moneyDecimals` (`true` or `false`) and a few layout keys. These are set up by Pixfizz for each store; treat them as **No** unless Pixfizz tells you otherwise. The same block can switch on the 3D landing hero (`hero3d`), also set up by Pixfizz. *Corrected 2026-11-09: from 4.0.1 the 3D hero needs no page block; see "Landing hero" below.*

**From 3.4.0, collections settings work only in `#pbs-config`.** The parent snippet no longer passes collections-mode params (`pbs_source`, `pbs_ranges`, `pbs_range_names`, ...) to `#pbs-root`; params given in the snippet call are silently ignored. Symptom: the page shows "The studio could not load. Please refresh the page." and the console says the mount is missing `pbs_collection, pbs_products, pbs_types, pbs_default_style` and "pbs_collection and pbs_products are required", because the source never became `collections` and the standard book layer ran. Fix on the site, with no parent change: put the settings in the `#pbs-config` block in the Pages page content. Keys are the camelCase dataset names (`source`, `ranges`, `rangeNames`, `rangeLines`, `exclude`, `blankFrom`, `defaultProduct`, `skin`, `look`, `editorOnMobile`, `crop`, `titleStandins`, `landing`, `loginUrl`, `exitUrl`, `moneyDecimals`, `sampleTitles`, `demoTitle`, `demoCaptions`, and others); the 3.4.0 prelude copies each one into `#pbs-root` only where the snippet left it empty. Check: the console ready line reads `source collections` and lists your types. Tool-level. *Verified by query on a live client site with the block placed before the snippet tag, 2026-11-03 (hand-off to the design tool and add to cart not tested).* Whether the block also works after the snippet tag, as written above, is pending confirmation.

**Landing hero: 3D only.** The 2.x CSS hero animation is retired on every site; the 3D landing hero is the only hero. Since 4.0.1 its images and three.js come from the parent mount itself (`pbs-assets`: `three`, `heroPlate`, `heroPlateM`, `heroPoster`, `heroPosterM`), so no per-site `#pbs-config` block is needed to switch it on; the block may still override the images, and `heroPlay()` returns at once. With no WebGL or a renderer failure the page shows the 3D hero's own still (table and closed book) or the hero photo, never the CSS animation. Install check: the live page console logs `[photo-book-studio] hero {mode: 3d}`; a site still showing the old animation is running an older build or a page set up before 4.0.1. Tool-level. *Decided by Pixfizz, 2026-11-03; seen on a pack install the same day.*

Things that are not set here:
- Page limits, book sizes and book types: the templates and design codes.
- Cover and paper options and their prices: the product variants and pricing.
- Designs and layouts: the design collection and its themes.

Not yet confirmed: The default of `pbs_default_product` and the exact meaning of `pbs_look` are not confirmed.

### Cover Studio
Status: in test, version 1.8.3 (asset; style updates numbered 1.9.1, as of the CHANGELOG). Snippet: `product/cover-studio`. Mount: `{% snippet 'product/cover-studio' %}` with no arguments, in the `custom_script` field of an imprint template option on each album template (there is no `cvs_mount` option yet). Prefix: `cvs_`.
A foil imprint configurator for album and book covers: the shopper picks position, lines of text, foil and font for the front, back and spine, and sees the cover before ordering. It writes option values only; there is no print file because the foil is struck from a physical die.

Cover Studio does not use mount settings yet. It still reads the older settings sources below. A move to the standard mount is planned; until then, the mount line itself has nothing to change.

Example mount:
```liquid
{% snippet 'product/cover-studio' %}
```

Where its settings live today:

| Setting | Where it is set | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|---|
| `cvs-rate-per-line` | Site checklist `admin/checklist/cvs-rate-per-line` (Custom Admin, Cover Studio settings page) | Price per line of cover imprint | A number with no currency sign | `0` | **With care**: must match the price your lab charges |
| `cvs-rate-spine` | Site checklist `admin/checklist/cvs-rate-spine` | Price for a spine imprint | A number with no currency sign | `0` | **With care**: must match the price your lab charges |
| `cvs-design-btn-selector` | Site checklist `admin/checklist/cvs-design-btn-selector` | Which page button Cover Studio holds back until the cover is set | A CSS selector, or empty | empty: finds Design now, Start design or Personalize | **With care**: only if your theme renamed that button |
| `cvs-config` | Site checklist `admin/checklist/cvs-config`, or the product custom field `tool_config` (the product field wins) | Positions, characters per line, allowed characters, fonts, foils, preview spine and board size, fallback colors, hiding the platform imprint fields, and the option codes the tool reads | JSON with the keys `positions`, `optionCodes`, `maxCharsPerLine`, `charPattern`, `fonts`, `foils`, `previewSpineIn`, `previewBoardIn`, `fallbackColors`, `hideNativeImprint` | built-in: positions `center`, `bottom-right`; 30 characters per line; foils `gold`, `silver`, `copper`, `black`, `white`, `red`, `emboss`; font Times New Roman; spine preview 1.0 in; `optionCodes` family `leather-option`, color `leather-color` | **With care**: invalid JSON falls back to the built-in defaults. `optionCodes` must match your template option and variant codes. Ask Pixfizz before editing |
| `cover_dimensions` | Product custom field | Size of one cover board (the book trim size), with its unit | For example `10x10in`, `254x254mm`, `25.4x25.4cm` | read from the size variant, then the product heading | **With care**: must match the real book trim |
| `cvs_family` | Product custom field | Cover family the preview uses | Text | `leather` | **With care** |

Never put an apostrophe in any of these values.

Things that are not set here:
- The shopper's choices (position, text, foil, font): template options on the album template, sent with the order.
- Cover material and color: the product variants.
- Album prices other than the imprint rates: the product pricing.

Not yet confirmed: The behavior of some `cvs-config` keys is not confirmed; ask Pixfizz before editing that JSON.

### Custom Sized Prints
Status: live, version 0.4.1 on the shopper24 parent; 0.5.0 (metric) built, not yet released. Snippet: `product/custom-print`. Mount option: `px_mount` (hidden Text template option, one per template). Prefix: `cp_`.
The shopper types any width and height inside your limits, uploads a photo or PDF, sees it on a wall or a scene, and gets an area-based price. One tool covers paper prints, metal, acrylic, canvas and banners; `cp_kind` picks which.

Example mount (paper prints, inches with a cm option):
```liquid
{% snippet 'product/custom-print',
	cp_display: 'inline',
	cp_kind: 'paper',
	cp_units: 'in,cm',
	cp_unit_default: 'in',
	cp_fractions: 'true',
	cp_step: '0.25in',
	cp_step_cm: '0.5',
	cp_min_w: '4in',
	cp_min_h: '4in',
	cp_max_w: '60in',
	cp_max_h: '60in',
	cp_roll_width: '44in',
	cp_max_long: '60in',
	cp_bleed: '0in',
	cp_safe: '0.125in',
	cp_good_dpi: '150',
	cp_min_dpi: '100',
	cp_out_dpi: '300',
	cp_max_mb: '200',
	cp_pdf: 'true',
	cp_stock_label: 'Paper',
	cp_notes: 'lustre:Soft sheen|glossy:Deep blacks, bright color|matte:No glare, soft finish' %}
```
A canvas mount adds `cp_kind: 'canvas'`, `cp_wrap: '1.5:2in|0.75:1in'` and the depth and edge variant codes. A banner mount uses `cp_kind: 'banner'`, `cp_units: 'ft,in'` and `cp_grommets_variant: 'px_grommets'`.

Lengths always carry their unit: `in`, `ft`, `cm` or `mm` (for example `0.25in`, `5ft`, `1100mm`). A bare number is read as inches.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `cp_display` | Layout of the tool | `inline` (the only layout built) | none, required | No |
| `cp_kind` | Product family: drives the preview scene, canvas wrap, grommets and wording | `paper`, `metal`, `acrylic`, `canvas`, `banner` | none, required | With care: must match the product |
| `cp_units` | Units the shopper can switch between | comma list, for example `in,cm`, `ft,in`, `cm` | none, required | Yes |
| `cp_unit_default` | Unit shown first | one of the units in `cp_units`, for example `in`, `cm`, `ft` | none, required | Yes |
| `cp_fractions` | Show sizes as fractions (17 1/4) or decimals (17.25) | `true`, `false` | none, required | Yes |
| `cp_step` | Size step when working in inches or feet | length with unit, for example `0.25in`, `1in` | none, required | Yes |
| `cp_step_cm` | Size step when working in cm (a plain number) | number, for example `0.5` | none; required when `cp_units` includes `cm` | Yes |
| `cp_min_w`, `cp_min_h` | Smallest width and height | length with unit | none, required | With care: must be a size your lab can produce and your pricing card covers |
| `cp_max_w`, `cp_max_h` | Largest width and height | length with unit | none, required | With care: as above |
| `cp_roll_width` | The longest the short side can be (roll or sheet width) | length with unit | none, required | With care: must match your printer or media |
| `cp_max_long` | The longest the long side can be | length with unit | none, required | With care: must match your printer or media |
| `cp_bleed` | Print past the cut on every side | length with unit, `0in` for prints | none, required | With care: must match your production spec |
| `cp_safe` | Keep-clear margin inside the cut, shown on the layout | length with unit | none, required | Yes |
| `cp_wrap` | Canvas only: print bleed per frame depth | `depth:bleed` pairs split by `|`, for example `1.5:2in|0.75:1in` | none; required when `cp_kind` is `canvas` | With care: depth values must match your depth variant values |
| `cp_good_dpi` | At or above this the photo is called sharp | number, for example `150` | none, required | Yes |
| `cp_min_dpi` | Below this the photo is called soft (a warning only) | number, for example `100` | none, required | Yes |
| `cp_out_dpi` | Resolution of the print file | number | none, required | With care: must match the dpi in the template XML |
| `cp_max_mb` | Largest upload accepted | number, MB | none, required | Yes |
| `cp_pdf` | Accept a PDF as well as photos | `true`, `false` | none, required | Yes |
| `cp_size_unit` | 0.5.0 and later. Unit written to the size variants, job spec and file name | `in`, `mm` | `in` | With care: `mm` needs the `px_width_mm` / `px_height_mm` variants and a metric pricing card |
| `cp_title` | Heading above the tool | text | the product name | Yes |
| `cp_intro` | Short intro line | text | blank | Yes |
| `cp_stock_label` | Label for the material choice | text, for example `Paper`, `Finish`, `Canvas`, `Vinyl` | blank | Yes |
| `cp_notes` | One line under a choice | `value:text` pairs split by `|`; prefix `depth.`, `edge.` or `grommets.` for those choices | blank | Yes |
| `cp_templates` | Artwork template downloads offered to the shopper | `label=url` pairs split by `|` | blank | Yes |
| `cp_grommets_when` | Banner stocks that take grommets; on any other stock the grommet choice is hidden | comma list of stock values | blank (every stock) | Yes |
| `cp_width_variant`, `cp_height_variant` | Variant codes the size is written to | variant code | `px_width_in` / `px_height_in` (`px_width_mm` / `px_height_mm` with `cp_size_unit: 'mm'`) | No, unless you renamed those variants |
| `cp_stock_variant` | Variant code for the material | variant code | `px_stock` | No, unless renamed |
| `cp_price_variant` | Hidden Number variant that carries the unit price | variant code | `px_price` | No, unless renamed |
| `cp_hanging_variant`, `cp_depth_variant`, `cp_edge_variant`, `cp_edgecolor_variant`, `cp_grommets_variant` | Variant codes for hanging, canvas depth, edge, edge color and grommets | variant code | blank (choice not offered) | With care: set only if the product has that variant, and use its exact code |
| `cp_print_code`, `cp_preview_code`, `cp_original_code`, `cp_place_code`, `cp_spec_code` | Template options the print file, preview, original upload, placement and job spec are written to | option code | `px_print_file`, `px_preview`, `px_original`, `px_place`, `px_spec` | No, unless renamed |

Things that are not set here
- Prices: they come from the `px_rates` card on the product (area pricing), not the mount.
- The materials, depths, edges and grommet choices themselves: those are variant values on the product.

Not yet confirmed: A bare number is read as inches for some length settings; not confirmed for every one, so always write the unit.

### TilePix Wall Studio (Wall Designer)
Status: live, version 0.6.10 asset (snippet 0.6.6; any 0.6.x mount runs on the current asset). Snippet: `product/wall-designer`. Mount option: `wd_mount`. Prefix: `wd_`.
The shopper builds a wall of up to 16 pieces (TilePix classic tiles, glass prints, skateboard decks or a mix), picks a layout and spacing, adds photos (or one photo across the whole wall) and sees it in a room before ordering. The tool writes one print file per piece plus a hang guide.

Example mount:
```liquid
{% snippet 'product/wall-designer',
	wd_version: '0.6.3',
	wd_display: 'inline',
	wd_max_pieces: '16',
	wd_families: 'classic,glass,skate,mix',
	wd_gaps: '1,2,3',
	wd_default_gap: '2',
	wd_min_dpi: '100',
	wd_good_dpi: '150',
	wd_output: 'jpeg',
	wd_ppi: '300',
	wd_glass_pad: '12-pad',
	wd_code_classic_black: 'TP_8x8_BLK',
	wd_code_classic_white: 'TP_8x8_WHT',
	wd_code_classic_gray: '',
	wd_code_g825: 'TP_GLASS_8x8',
	wd_code_g1114: 'GLASS_11x14',
	wd_code_g1616: 'GLASS_16x16',
	wd_code_g1624: 'GLASS_16x24',
	wd_code_skate: 'DTC_SBD_Skateboard',
	wd_family_code: 'wd_family',
	wd_layout_code: 'wd_layout',
	wd_frame_code: 'wd_frame',
	wd_gap_code: 'wd_gap',
	wd_qty_prefix: 'wd_qty_',
	wd_print_code: 'px_print_file',
	wd_print_prefix: 'px_print_p',
	wd_preview_code: 'px_preview',
	wd_spec_code: 'px_spec',
	wd_hang_code: 'wd_hang_guide',
	wd_src_code: 'wd_src_photo',
	wd_requires_design: 'true',
	wd_title: 'Wall Studio',
	wd_intro: 'Peel, stick, hang. No nails, no holes.' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `wd_version` | Mount version, compared with the tool version | version number, for example `0.6.3` | none, required | No |
| `wd_display` | Layout | `inline` (the tool owns the product page) | none, required | No |
| `wd_max_pieces` | Most pieces on one wall | number up to `16` (one print file option per piece) | none, required | With care: the template needs a print file option for every piece |
| `wd_families` | Which product families the shopper can pick | any of `classic`, `glass`, `skate`, `mix`, comma separated | none, required | Yes, within what your template's variants support |
| `wd_gaps` | Spacing choices between pieces, inches | any of `1`, `2`, `3`, comma separated | none, required | Yes |
| `wd_default_gap` | Spacing on the first view, inches | one of the `wd_gaps` values | none, required | Yes |
| `wd_min_dpi` | Below this a piece is flagged as printing soft | number | none, required | Yes |
| `wd_good_dpi` | At or above this a piece is called sharp | number | none, required | Yes |
| `wd_output` | Print file format | `jpeg` | none, required | No |
| `wd_ppi` | Print file resolution | number, for example `300` | none, required | With care: must match the supplier's file spec |
| `wd_glass_pad` | Glass mounting pad written to the job record | supplier pad code, for example `12-pad` | none, required | With care: must match the supplier spec |
| `wd_code_classic_black`, `wd_code_classic_white`, `wd_code_classic_gray` | Supplier item code for each classic tile frame color; blank means not offered | supplier item code, or blank | blank (not offered) | With care: must be the supplier's exact item code; blank switches that frame off |
| `wd_code_g825`, `wd_code_g1114`, `wd_code_g1616`, `wd_code_g1624` | Supplier item code for each glass size; blank means not offered | supplier item code, or blank | blank (not offered) | With care: as above |
| `wd_code_skate` | Supplier item code for skateboard decks; blank means not offered | supplier item code, or blank | blank (not offered) | With care: as above |
| `wd_family_code`, `wd_layout_code`, `wd_frame_code`, `wd_gap_code` | Variant codes for family, layout, frame and spacing | variant code | none, required (standard: `wd_family`, `wd_layout`, `wd_frame`, `wd_gap`) | No, unless you renamed those variants |
| `wd_qty_prefix` | Prefix of the per-family count variants | text, standard `wd_qty_` | none, required | No, unless renamed |
| `wd_print_code` | First print file option | option code, standard `px_print_file` | none, required | No, unless renamed |
| `wd_print_prefix` | Prefix of print file options 2 to 16 | text, standard `px_print_p` (gives `px_print_p02` and so on) | none, required | No, unless renamed |
| `wd_preview_code` | Cart preview option | option code, standard `px_preview` | none, required | No, unless renamed |
| `wd_spec_code` | Job record option | option code, standard `px_spec` | none, required | No, unless renamed |
| `wd_hang_code` | Hang guide PDF option | option code, standard `wd_hang_guide` | `wd_hang_guide` (warns if missing) | No, unless renamed |
| `wd_src_code` | Original photo option when one photo spans the wall | option code, standard `wd_src_photo` | `wd_src_photo` (warns if missing) | No, unless renamed |
| `wd_requires_design` | Add to cart only through the tool | `true`, `false` | none, required | With care: `false` lets an empty wall reach the cart |
| `wd_title` | Heading above the tool | text | blank | Yes |
| `wd_intro` | Short intro line | text | blank | Yes |
| `wd_set` | Scene and sample set | `home`, `business` | `home` (blank) | Yes |
| `wd_font_heading`, `wd_font_body` | Font stacks for a studio look | CSS font stack | blank (your site theme fonts) | Yes; leave out to keep your theme fonts |
| `wd_font_css` | Stylesheet that loads those fonts | `https://fonts.googleapis.com/...` URL only | blank | Yes |
| `wd_studio`, `wd_studio_tag` | Studio bar label and small tag above the tool | text | blank (no bar) | Yes |
| `wd_by_logo`, `wd_by_name` | "Fulfilled by" supplier logo and name | image URL; text | blank (text only) | Yes |
| `wd_price_label` | Label for the page price on catalog or demo sites where the page price is what the lab pays | text, for example `Lab price` | blank (no lab pricing lines) | Yes; leave out on a retail site |
| `wd_retail_classic`, `wd_retail_g825`, `wd_retail_g1114`, `wd_retail_g1616`, `wd_retail_g1624`, `wd_retail_skate` | Suggested retail per piece shown with the lab markup (only with `wd_price_label`) | number, for example `19.99` | blank | Yes |

Things that are not set here
- Prices: they come from the product's variants and pricing, not the mount.
- Which families, layouts and frames exist as choices: those are variant values on the product; the mount only switches them on or off.

### Wall Design Studio (metal and acrylic walls)
Status: live, version 0.8.7 (snippet 0.8.5). Snippet: `product/wall-design-studio`. Mount: a Pages page (usually `/site/wall-design-studio`) whose content calls the snippet; it can also sit in the `custom_script` of a template option. Prefix: `wds_`.
The shopper picks one of 12 ready-made wall layouts (2 to 9 panels), a material and size, drags photos into the panels and sees the wall in a room. The studio adds one cart line per panel on your own single metal or acrylic products, plus a hang guide. Not the same tool as TilePix Wall Studio (`product/wall-designer`).

How values are written: plain text values as written; structured values (lists, prices, product maps, layouts, backings) as JSON with double quotes. Images are asset names on your site.

Example mount (shape only: product ids, paths and codes come from your own catalog):
```liquid
{% snippet 'product/wall-design-studio',
	wds_photo_option: 'photo',
	wds_hang_option: 'px_print_hang',
	wds_products: '{"metal":{"12x18":101,"16x24":102},"acrylic":{"12x18":201}}',
	wds_paths: '{"metal":"/site/shop/metal-prints","acrylic":"/site/shop/acrylic-prints"}',
	wds_mount: '{"metal":{"code":"mounting","value":"float","label":"Float mount"}}',
	wds_discount: '{"label":"Wall discount","tiers":{"2":5,"3":10,"4":15,"5":20}}',
	wds_dpi: '{"min":100,"good":150}',
	wds_room_sofa: 'room-sofa.webp',
	wds_room_bedroom: 'room-bedroom.webp',
	wds_room_hallway: 'room-hallway.webp',
	wds_samples: 'Beach:sample-1.webp|Forest:sample-2.webp' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `wds_photo_option` | Image template option on each panel product that receives the photo | option code | none, required (the tool falls back to `photo`) | No, unless your panel products use another code |
| `wds_hang_option` | File option on panel 1 that receives the hang guide PDF | option code | `px_print_hang` | No, unless renamed |
| `wds_products` | Which of your products is each panel size, per material | JSON: material to size to product id | none, required unless `wds_demo` is `true` | With care: every id must be a live product of that material and size |
| `wds_paths` | Shop page (collection) per material where each size's add-to-cart form is found; can carry per-size query strings | JSON: material to collection path | none | With care: must resolve to the right product for every size |
| `wds_mount`, `wds_finish`, `wds_fixed` | Variant code and value the studio sets on every panel (mounting, finish, other fixed choices such as corners) | JSON per material, with `code` and `value` (and an optional `label`) | none | With care: codes and values must match your products' variant types exactly |
| `wds_discount` | Wall discount shown in the studio | JSON with `label` and `tiers` (panel count to percent) | none (no discount shown) | With care: tiers must equal your Automatic Discount, or the studio shows a price the cart does not charge |
| `wds_prices` | Panel prices used only when a size has no product id (demo sites) | JSON: material to size to price | none | Yes, on a demo site |
| `wds_materials` | Which materials are offered | JSON list, `metal` and/or `acrylic` | both | Yes |
| `wds_mat_text` | Replacement wording for a material | JSON per material | built-in wording | Yes |
| `wds_dpi` | Photo sharpness thresholds | JSON `{"min":..,"good":..}` | min 100, good 150 | Yes |
| `wds_hang` | Hanging point per material (inches down from the top edge) for the hang guide | JSON per material | none | With care: must match the hardware you ship |
| `wds_size_advice` | Size advice notes on the wall | `false` turns them off | on | Yes |
| `wds_ship_box` | Largest shipping box, short and long side in inches | JSON `[short, long]` | none | With care: must match your shipping |
| `wds_backings`, `wds_frame_colors`, `wds_hardware`, `wds_start_back` | Optional backing, frame color and hanging hardware choices (each can carry a per-panel price) and the backing selected first | JSON lists; `wds_start_back` is a backing id | none (not offered) | With care: prices must match your product pricing |
| `wds_layouts` | Restrict or reorder the built-in layouts | JSON list of layout ids | all 12 | Yes |
| `wds_auto` | Layout picked automatically for a given number of photos | JSON: photo count to layout id | built-in map | Yes |
| `wds_start_layout`, `wds_start_opt`, `wds_start_scene` | Layout, size option and room shown first | layout id; size option; `sofa`, `bedroom`, `hallway` | `tri`; first available; `sofa` | Yes |
| `wds_room_sofa`, `wds_room_bedroom`, `wds_room_hallway`, `wds_room_sofa_front` | Room images | asset names | `wds_room_sofa` required | Yes |
| `wds_samples` | Sample photos shown before the shopper adds their own | `Name:file.webp` pairs split by `|` | none | Yes |
| `wds_max_samples` | How many samples to use | number | `4` | Yes |
| `wds_title`, `wds_title_tool`, `wds_facts_line`, `wds_facts`, `wds_facts_tool`, `wds_product_name` | Headings and short fact lines | text | built-in wording ("Wall Design Studio") | Yes |
| `wds_gallery_id` | Customer gallery the upload window opens | gallery id | none | With care: must be a gallery on your site |
| `wds_upload_sources` | Upload sources offered | any of `local`, `galleries`, `qr`, space separated | `local galleries qr` | Yes |
| `wds_demo`, `wds_demo_note`, `wds_demo_done` | Demo mode with no real cart, and its messages | `true` / `false`; text | off | Yes, demo sites only |

Things that are not set here
- Panel prices: they are your own products' prices and variant surcharges. The wall discount is an Automatic Discount on your site.
- Brand colors and the wall texture: theme tokens (`--pxt-*`, `--wdx-gallery`) in your site CSS.
- Canvas and wood walls are not supported yet; metal and acrylic only.

Not yet confirmed: The inner field names of the JSON settings are not confirmed here; copy the shape from your installed mount.

### Fan Face (face fan designer)
Status: beta, version 1.2.0 (built, being rolled out to sites; older installs still use the pre-1.0 settings page, see below). Snippet: `product/face-fan-designer`. Mount option: `facefan_mount`. Prefix: `facefan_`.
The shopper uploads a face photo, positions it in an oval (or a hand-cut shape) on a big-head fan sheet, and approves a proof. The tool writes the print file with a cut path for the cutter.

Example mount:
```liquid
{%- assign facefan_xml_dpi = '' -%}
{%- if template.dpi -%}{%- assign facefan_xml_dpi = template.dpi -%}{%- endif -%}
{% snippet 'product/face-fan-designer',
	facefan_display: 'inline',
	facefan_sizes: '8.5x11:8.5x11in|12x18:12x18in',
	facefan_size_variant: 'size',
	facefan_cut_variant: 'cut',
	facefan_cut_modes: 'oval:oval|head:none',
	facefan_upscale_variant: 'upscale',
	facefan_bleed: '0.125in',
	facefan_min_dpi: '150',
	facefan_good_dpi: facefan_xml_dpi,
	facefan_max_mb: '40',
	facefan_zoom_max: '1000',
	facefan_cut_spot: 'CutContour',
	facefan_cut_file: 'embedded',
	facefan_block_low_dpi: 'false',
	facefan_title: 'Design your face fan',
	facefan_trust: 'Printed and assembled in our shop|Ready in 2 business days' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `facefan_display` | Layout | `inline`, `modal` | `inline` | Yes |
| `facefan_sizes` | Sheet size for each size variant value; the print file is the sheet, the oval is the sheet less the bleed | `value:WxHin` pairs split by `|`, for example `8.5x11:8.5x11in|12x18:12x18in` | none, required | With care: values must match your size variant values, sizes must match the XML template |
| `facefan_size_variant` | Code of the size variant the tool keeps selected | variant code | none, required | No, unless you renamed it |
| `facefan_bleed` | Photo past the cut on every side | length with unit, for example `0.125in` | none, required | With care: must match your production spec |
| `facefan_min_dpi` | Below this the photo is called blurry | number, for example `150` | none, required | Yes |
| `facefan_good_dpi` | Print resolution and the sharp threshold; passed from the XML template dpi | `template.dpi` as in the example, or a number | none, required (falls back to 300 with a console error) | With care: must match the dpi in the template XML; keep the `template.dpi` lines |
| `facefan_max_mb` | Largest photo accepted | number, MB | none, required | Yes |
| `facefan_zoom_max` | Highest zoom, percent (100 = the photo just fills the oval) | number | none, required | Yes |
| `facefan_cut_spot` | Spot color name of the cut path | text, for example `CutContour` | none, required | With care: must be the name your cutter or RIP expects |
| `facefan_cut_file` | Where the cut path goes | `embedded` (inside the print file), `separate` (its own file on `px_print_cut`) | none, required | With care: must match how your cutter takes files |
| `facefan_block_low_dpi` | Whether a blurry photo can still be approved | `true` (cannot approve), `false` (warning only) | none, required | Yes |
| `facefan_cut_variant` | Code of the cut variant (oval or hand cut) | variant code, or blank | blank (oval only) | No, unless renamed |
| `facefan_cut_modes` | What each cut value means | `value:oval` or `value:none` pairs split by `|`; `none` = no cut path, cut by hand | blank | With care: values must match your cut variant values |
| `facefan_upscale_variant` | Code of the upscale variant offered when the photo is under `facefan_min_dpi` | variant code, or blank | blank (not offered) | No, unless renamed |
| `facefan_upscale_yes`, `facefan_upscale_no` | Value codes of the upscale variant | value code | `yes`, `no` | No, unless renamed |
| `facefan_title`, `facefan_intro` | Heading and intro line | text | blank | Yes |
| `facefan_trust` | Short reassurance lines under the tool | lines split by `|` | blank | Yes |
| `facefan_print_code`, `facefan_cut_code`, `facefan_preview_code`, `facefan_original_code`, `facefan_place_code`, `facefan_spec_code` | Template options the files and job record are written to | option code | `px_print_file`, `px_print_cut`, `px_preview`, `facefan_original`, `facefan_place`, `px_spec` | No, unless renamed |
| `facefan_trim_w`, `facefan_trim_h` | Older single-size form (1.0.0 mounts) | length with unit | blank | No; use `facefan_sizes` on new mounts |

Things that are not set here
- Prices: product and variant pricing.
- Installs older than 1.0.0 do not use a mount: they read product custom fields `facefan_*` and a "Fan Face Settings" checklist under Custom Admin. Those settings stop being read once the site moves to 1.2.0.

### Custom Framing
Status: built, version 0.4.0; not yet moved to the mount standard. Snippet: `product/framing`.
No mount settings yet. Settings come from product custom fields `framing_*` first, then the "Framing Settings" checklist under Custom Admin > Tools (keys `framing-units`, `framing-size-mode`, `framing-min-in`, `framing-max-in`, `framing-max-united-in`, `framing-preset-sizes`, `framing-mat-modes`, `framing-mat-min-in`, `framing-mat-max-in`, `framing-mat-default-in`, `framing-mat-weighted-bottom`, `framing-min-dpi`, `framing-dpi-action`, `framing-max-file-mb`, `framing-show-price`, `framing-room-scenes`, `framing-lab-prints`, `framing-preview-disclaimer`, `framing-preview-disclaimer-detail`, `framing-fallback-url`, `framing-fallback-message`, `framing-pricing`, `framing-config`), then built-in defaults. Mouldings are Custom Type instances; mats, glazing and finishing are variant values; rates are the `tool_pricing` JSON. Ask Pixfizz before changing `framing-pricing`, `framing-config` or `tool_config`.

## 6.2 Print and cut tools

### Sticker Studio
Status: live, version 6.2.2 (per the CHANGELOG). Snippet: `product/sticker-studio`. Mount option: `px_mount`. Prefix: `stk_`.
The shopper uploads artwork, the tool traces the cut line around it (contour or a chosen shape), shows a proof, and builds the print file and cut file for a print-and-cut workflow.

Example mount:
```liquid
{% snippet 'product/sticker-studio',
	stk_size_variant: 'size',
	stk_finish_variant: 'finish',
	stk_border_min: '0.04',
	stk_border_max: '0.25',
	stk_border_default: '0.08',
	stk_max_mb: '50',
	stk_xml_min_dpi: template.minimum_dpi,
	stk_cut_spot: 'CutContour',
	stk_cut_output: 'combined',
	stk_title: 'Custom die-cut stickers',
	stk_trust: 'Cut line traced for you|Proof before you order'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `stk_size_variant` (required) | The variant that carries the sticker size | A variant code whose values are W x H in inches, e.g. `3x3` | `size` | **No**: must match your size variant code |
| `stk_cut_spot` (required) | Spot color name your RIP or cutter reads as the cut line | A name, letter for letter, for example `CutContour` or `CutPath` | none (required) | **With care**: must match the cut spot name in your RIP exactly, or the line prints instead of cutting |
| `stk_cut_output` (required) | Cut line inside the print file, or as a separate cut file | `combined` or `split` | `combined` | **With care**: match your production |
| `stk_max_mb` (required) | Largest file a shopper can upload | A number in MB | `50` if unreadable | **Yes** |
| `stk_border_min`, `stk_border_max`, `stk_border_default` (required) | Smallest, largest and starting white border around the artwork | Numbers in inches | `0.04`, `0.25`, `0.08` if unreadable | **With care**: within what your cutter can hold |
| `stk_xml_min_dpi` | Minimum dpi read from the template XML; wins over `stk_min_dpi` | Leave as `template.minimum_dpi` | blank | **No**: leave as shown |
| `stk_min_dpi` | Minimum dpi when the XML gives none | A number | `300` | **With care**: your print quality bar |
| `stk_low_dpi` | Below this the shopper is warned the print may look soft | A number | `100` | **Yes** |
| `stk_backing_variant` | Variant that carries kiss cut, die cut with border, or die cut without border (read from the value names). May list several codes as `a\|b`; the first one the product has is used | Variant code(s) | blank | **No**: must match your variant code(s) |
| `stk_backing_kind` | Cut style used when the product has no backing variant | `border`, `noborder` or `kiss` | `border` | **With care**: match the product you sell |
| `stk_finish_variant` | Variant that carries the finish (e.g. matte or gloss) | A variant code | blank | **No**: must match your variant code |
| `stk_cut_variant`, `stk_cut_value` | A fixed variant the tool sets for every order, e.g. a cut type of custom shape | A variant code and one of its value codes | blank | **No**: must match your variant and value codes |
| `stk_hide_variants` | Other variants to hide on the page because the tool handles them | Variant codes, space separated | blank | **Yes**, as long as the codes are right |
| `stk_kiss_margin` | Margin around the artwork on a kiss cut | A number in inches | `0.0625` | **With care** |
| `stk_no_border` | Cut distance on a die cut with no border | A number in inches | `0.02` | **With care**: match your cutter tolerance |
| `stk_cut_stroke` | Cut line stroke width | A number in points | `0.5` | **With care** |
| `stk_title`, `stk_intro` | Heading and intro line | Text, no apostrophes | `Custom stickers` and a standard intro | **Yes** |
| `stk_trust` | Short reassurance lines in the header | Lines separated by `\|` | `Cut line traced for you\|Proof before you order` | **Yes** |
| `stk_look` | Extra style classes on the tool | Class names | blank | **Yes** |
| `stk_owns_page` | Whether the tool takes over the product page layout | `true` or `false` | `true` | **Yes** |
| `stk_print_code`, `stk_cut_code`, `stk_preview_code`, `stk_original_code`, `stk_spec_code` | Template options the print file, cut file, preview, original and job spec are written to | Option codes | `px_print_file`, `px_print_cut`, `px_preview`, `px_original`, `px_spec` | **No**: your fulfillment reads these |
| `stk_add_to_cart_selector` | Which button counts as Add to Cart, on a heavily customized page | A CSS selector | blank | **No**, unless Pixfizz support asks you to |

Settings added in 6.1 and 6.2 (read by the tool; confirm with Pixfizz that your site has the 6.2 snippet before relying on them):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `stk_shape_variant` | Variant that carries the shape (contour, circle, square, oval, rectangle, heart, star, diamond) | A variant code | blank | **No**: must match your variant code |
| `stk_corner_radius` | Corner radius for Rounded square and rectangle shapes | A number in inches | `0.125` | **With care**: match your die or cutter |
| `stk_bleed` | How far a Fill artwork bleeds past the cut line | A number in inches | `0.0625` | **With care** |
| `stk_custom_size` | Typed sizes (width and height fields) when the product has a sheet rate card | `off` to turn off; on otherwise | on | **Yes** |
| `stk_width_variant`, `stk_height_variant`, `stk_price_variant` | Number variants the typed width, height and unit price are written to | Variant codes | `px_width_in`, `px_height_in`, `px_price` | **No**: must match your variant codes |

Border, margin and cut lengths are plain numbers in inches.

Things that are not set here
- Size, finish, backing and shape choices are variants on the product: they carry the shopper's choice and the price.
- Typed-size prices come from the product's sheet rate card (`px_rates`), not the mount.
- Sticker Designer 5.30 (`product/sticker-designer`) is a separate, older tool.

Not yet confirmed: The 6.1 and 6.2 settings are read by the tool script; whether every installed snippet passes them through is not confirmed. Ask Pixfizz before relying on them.

### Sticker Designer (legacy)
Status: live on older installs, version 5.30 (per the CHANGELOG). New installs use Sticker Studio. Snippet: `product/sticker-designer`. Mount option: the `custom_script` field on the `sticker_artwork` template option. Prefix: `stk_`.
The shopper uploads artwork in a pop-up (or inline) editor, picks shape and size, and the tool builds a print PDF with a contour cut line.

Example mount:
```liquid
{% snippet 'product/sticker-designer',
	stk_display: 'modal',
	stk_cut_spot: 'CutContour',
	stk_cut_stroke: '0.5',
	stk_cut_layer: 'true',
	stk_cut_overprint: 'true',
	stk_cut_split: 'false',
	stk_max_mb: '50',
	stk_target_dpi: '300',
	stk_price_footer: 'each',
	stk_quantity_tiers: '',
	stk_type_toggle: 'false',
	stk_artwork_check: 'true',
	stk_requires_design: 'true'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `stk_display` | Pop-up editor behind a button, or open on the page | `modal` or `inline` | `modal` | **Yes** |
| `stk_cut_spot` | Spot color name your RIP or cutter reads as the cut line | A name, letter for letter, for example `CutContour` or `CutPath` | `CutPath` | **With care**: must match the cut spot name in your RIP exactly, or the line prints instead of cutting |
| `stk_cut_stroke` | Cut line stroke width | A number in points | `0.5` | **With care** |
| `stk_cut_layer` | Puts the cut line on its own PDF layer | `true` or `false` | `true` | **With care**: match your RIP |
| `stk_cut_overprint` | Sets the cut line to overprint | `true` or `false` | `true` | **With care**: match your RIP |
| `stk_cut_split` | Cut line as a separate file instead of inside the print file | `true` or `false` | `false` | **With care**: match your production |
| `stk_max_mb` | Largest file a shopper can upload | A number in MB | `50` | **Yes** |
| `stk_target_dpi` | Resolution of the print file the tool renders | A number from 72 to 1200 | `300` | **With care**: your print quality bar |
| `stk_price_footer` | Price line in the tool footer | `off`, `total` or `each` | `off` | **Yes** |
| `stk_quantity_tiers` | Quantity buttons shown in the tool | Numbers separated by commas, or empty for none | empty (no buttons) | **With care**: should match the quantities your pricing expects |
| `stk_qty_min`, `stk_qty_max` | Limits for the quantity buttons and typed quantity | Numbers | the product min and max quantity | **Yes** |
| `stk_type_toggle` | Shows a switch between sticker formats (uses the collection filter) | `true` or `false` | `false` | **Yes** |
| `stk_artwork_check` | Runs the artwork check and records the result | `true` or `false` | `true` | **Yes** |
| `stk_artwork_check_code` | Template option the artwork check result is written to | An option code | `artwork_check` | **No**: must match the option on your template |
| `stk_requires_design` | Blocks Add to Cart until artwork is added | `true` or `false` | `true` | **Yes** |
| `stk_nominal_w`, `stk_nominal_h` | Starting size shown, in inches (the shopper still picks the real size) | Numbers | `3` and `3` | **Yes** |
| `stk_units` | Unit label only. Figures stay in inches, so `mm` shows the wrong label | `in` | `in` | **No**: leave at `in` |

Flags also accept `yes`/`no`, `on`/`off` and `1`/`0`. Sizes are inches only.

Things that are not set here
- Sticker size, shape and quantity prices come from the product's variants and pricing.
- Older installs may still read some of these from a product custom field or the Sticker Settings page in Custom Admin. The mount wins when it sets a value; move every setting into the mount.

### Gang Up (DTF gang sheet)
Status: live, version 2.2.0 (latest built, per the CHANGELOG). Snippet: `product/gang-up`. Mount option: `px_mount`. Prefix: `gu_`.
The shopper uploads their designs, sets size and copies for each, and the tool packs them onto a roll of film. The shopper pays for the length used, and the tool builds the print file (and an optional cut file).

Example mount:
```liquid
{% snippet 'product/gang-up',
	gu_roll_width: '22in',
	gu_margin: '0.25in',
	gu_gap: '0.125in',
	gu_min_length: '12in',
	gu_max_length: '120in',
	gu_length_variant: 'length',
	gu_min_dpi: '300',
	gu_max_mb: '50',
	gu_accept: 'png jpg webp svg',
	gu_trim: 'true',
	gu_units: 'in',
	gu_view: 'auto',
	gu_output: 'pdf',
	gu_cut: 'false',
	gu_cut_spot: 'CutContour',
	gu_title: 'Custom DTF Gang Sheet',
	gu_intro: 'Upload your designs and set the size and copies. You pay only for the length you use.',
	gu_trust: 'Full white underbase|Pay only for the length you use|File checked before you order'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `gu_roll_width` (required) | Printable width of the film roll. Its presence also tells the platform this is a 2.x mount | A length, e.g. `22in` or `559mm` | none (a mount without it runs the old 1.x tool) | **With care**: must match the roll on your printer |
| `gu_margin` (required) | Empty edge left on each side of the sheet | A length | `0.25in` if unreadable | **With care**: match your printer |
| `gu_gap` (required) | Space between designs | A length | `0.125in` if unreadable | **With care**: enough to cut or weed between designs |
| `gu_min_length`, `gu_max_length` (required) | Shortest and longest sheet a shopper can order | Lengths | `12in` and `120in` if unreadable | **With care**: must match the lengths your pricing covers |
| `gu_length_variant` (required) | The product variant whose value is the priced sheet length | A variant code | `length` | **No**: must match your length variant code exactly |
| `gu_min_dpi` (required) | Minimum resolution for an uploaded design | A number | `300` if unreadable | **With care**: your print quality bar |
| `gu_max_mb` (required) | Largest file a shopper can upload | A number in MB | `50` if unreadable | **Yes** |
| `gu_accept` (required) | File types the shopper may upload | Any of `png jpg webp svg`, space separated | `png` if unreadable | **Yes** (transparent PNG is the safe choice for DTF) |
| `gu_trim` (required) | Automatically trims empty transparent space around each design | `true` or `false` | `true` | **Yes** |
| `gu_units` (required) | Units shown to the shopper | `in` or `mm` | none (required) | **Yes** |
| `gu_view` (required) | Background the sheet is shown on | `auto`, `checker`, `white`, `grey`, `charcoal`, `black` | `auto` | **Yes** |
| `gu_view_options` | Which backgrounds the shopper can switch between | Any of `checker white grey charcoal black custom`, space separated | all six | **Yes** |
| `gu_output` (required) | Print file format | `pdf` or `png` | `pdf` | **With care**: must be what your RIP accepts |
| `gu_cut` (required) | Also builds a contour cut line around each design | `true` or `false` | `false` | **With care**: only for a print-and-cut workflow |
| `gu_cut_offset` | How far the cut line sits outside each design | A number in inches | `0.0625` (max `0.5`) | **With care**: match your cutter |
| `gu_cut_clearance` | Extra space between neighboring cut lines | A number in inches | `0.0625` (max `1`) | **With care** |
| `gu_cut_spot` | Spot color name your cutter reads as the cut line | A name, letter for letter, for example `CutContour` or `CutPath` | `CutContour` | **With care**: must match the cut spot name in your RIP exactly, or the line prints instead of cutting |
| `gu_cut_output` | Cut line in the print file, or in a separate file | `combined` or `split` | `combined` | **With care**: match your production |
| `gu_cut_stroke` | Cut line stroke width in points | A number, 0.1 to 5 | `0.5` | **With care** |
| `gu_cut_page_units` | How the cut file page size is written | `points` or `userunit` | `points` | **With care**: change only if your RIP reads the size wrong |
| `gu_samples` | Shows a sample design before the first upload | `true` or `false` | `true` | **Yes** |
| `gu_title`, `gu_intro` | Heading and intro line | Text, no apostrophes | standard text | **Yes** |
| `gu_trust` | Short reassurance lines in the header | Lines separated by `\|` | `Full white underbase\|Pay only for the length you use` | **Yes** |
| `gu_look` | Extra style classes on the tool | Class names | blank | **Yes** |
| `gu_owns_page` | Whether the tool takes over the product page layout | `true` or `false` | `true` | **Yes** |
| `gu_print_code`, `gu_cut_code`, `gu_preview_code`, `gu_original_code`, `gu_spec_code` | Template options the print file, cut file, preview, originals and job spec are written to | Option codes | `px_print_file`, `px_print_cut`, `px_preview`, `px_original`, `px_spec` | **No**: your fulfillment reads these |
| `gu_check_code` | A text option that receives the artwork check result | An option code, e.g. `artwork_check` | blank (off) | **With care**: the option must exist on the template |
| `gu_add_to_cart_selector` | Which button counts as Add to Cart, on a heavily customized page | A CSS selector | blank | **No**, unless Pixfizz support asks you to |

Lengths accept inches and millimeters (`22in`, `559mm`). Gap and margin are raised automatically when the cut line is on, so cut lines never overlap.

Things that are not set here
- The price per length is the product pricing on the length variant, not the mount.
- Older Gang Up 1.x mounts (on the `gangup_artwork` option, without `gu_roll_width`) still work but only read the cut and trim settings; ask Pixfizz about moving to 2.x.

Not yet confirmed: `gu_cut_offset` and `gu_cut_clearance` are read as plain numbers in inches; a unit suffix is not confirmed to work.

## 6.3 Upload tools

### Uploader 2.0 (documents, booklets, forms, flat prints)
Status: live, version 2.6.0 (per the release notes; this folder has no CHANGELOG). Replaces Document Uploader and Booklet Uploader on new products. Snippet: `product/uploader`. Mount option: `px_mount`. Prefix: `up_`.
The shopper uploads a PDF, chooses size, printing, paper, finishing (and binding, cover or parts, depending on the mode), sees the price and adds it to the cart. One asset with four modes.

Example mount (the rate card block sits above the snippet call in the same field):
```liquid
<script type="application/json" data-px-rates>{ ...your rate card, on one line... }</script>
{% snippet 'product/uploader',
	up_mode: 'booklet',
	up_size: '',
	up_bleed: '0.125',
	up_crop_marks: 'false',
	up_max_mb: '100',
	up_min_dpi: '150',
	up_full_width: 'true',
	up_single_cart: 'true',
	up_write_preview: 'true',
	up_max_qty: '5000' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `up_mode` | What the product is | `flat` (documents, copies, flyers), `booklet` (saddle-stitched booklets), `forms` (carbonless NCR forms), `prints` (flat prints imposed on a press sheet) | `flat` | **With care**: must match the rate card in the same mount; a card for another mode stops pricing |
| `up_size` | Flat mode only: the one size this product sells | A size code from your rate card, or blank | blank (offers every priced size as a picker) | **With care**: must match a size code in your rate card and the template |
| `up_bleed` | Bleed added on every page of the print file | A number in inches | `0.125` | **With care**: match your production |
| `up_crop_marks` | Adds crop marks to the print file | `true` or `false` | `false` | **With care** |
| `up_max_mb` | Largest file a shopper can upload | A number in MB | `100` | **Yes** |
| `up_min_dpi` | Minimum resolution for images inside the PDF | A number | `150` | **With care**: your print quality bar |
| `up_full_width` | Tool uses the full page width | `true` or `false` | `true` | **Yes** |
| `up_single_cart` | Hides the theme Add to Cart so only the tool button shows | `true` or `false` | `true` | **Yes** |
| `up_write_preview` | Saves a preview image for the cart and order | `true` or `false` | `true` | **Yes** |
| `up_max_qty` | Highest quantity a shopper can order | A number | `5000` | **Yes** |
| `up_print_file_code`, `up_print_cover_code`, `up_original_code`, `up_original_cover_code`, `up_preview_code`, `up_spec_code` | Template options for the print file, cover print file, original uploads, preview and job spec | Option codes | `px_print_file`, `px_print_cover`, `px_original`, `px_original_cover`, `px_preview`, `px_spec` | **No**: your fulfillment reads these. Pass only if your template renamed one |
| `up_price_code`, `up_pages_code`, `up_sheets_code`, `up_size_code`, `up_paper_code`, `up_color_code`, `up_sides_code`, `up_finishing_code`, `up_binding_code`, `up_cover_src_code`, `up_cover_print_code`, `up_cover_paper_code`, `up_blanks_code`, `up_fit_code`, `up_notes_code` | Variants (and the notes option) the shopper's choices and the price are written to | Variant or option codes | the `px_` standard: `px_price`, `px_pages`, `px_sheets`, `px_size`, `px_paper`, `px_color`, `px_sides`, `px_finishing`, `px_binding`, `px_cover_src`, `px_cover_print`, `px_cover_paper`, `px_blanks`, `px_fit`, `px_notes` | **No**: must match your product variants; your POS and OrderHub read them. Pass only if you renamed one |

All ten main settings should be passed every time; any left out is reported in the browser console. Bleed is a plain number in inches.

The rate card is a one-line JSON block, `<script type="application/json" data-px-rates>...</script>`, in the same `px_mount` field above the snippet call. It holds the sizes, papers, colors, finishing, bindings, covers or form parts, and every price. Change prices there, never in the snippet arguments. Starter cards for documents, letter booklets and NCR forms exist; ask Pixfizz for one. From 2.6.0 the tool reads the product custom field `px_rates` first and the mount card second, so a generic template can carry a zero-price card in its mount while each site prices on its own product. *Verified by reading source (2.6.0), 2026-11-02.*

Things that are not set here
- Every shopper choice and the price are product variants, so the order, POS and OrderHub can read them.
- Products still on Document Uploader (`du_`) or Booklet Uploader (`bu_`) keep their own settings until moved.

### Document Uploader
Status: live, version 1.3.3 (per the CHANGELOG). To be replaced by Uploader 2.0. Snippet: `product/document-uploader`. Mount option: `du_mount`. Prefix: `du_`.
The shopper uploads a document (PDF), confirms its page count, picks paper, color, sides and finishing, sees the price, and adds it to the cart. One product, one template, one trim size per mount.

Example mount:
```liquid
{% snippet 'product/document-uploader',
	du_size: '85x11',
	du_bleed: '0.125',
	du_safe: '0.125',
	du_crop_marks: 'false',
	du_max_mb: '100',
	du_min_dpi: '150',
	du_full_width: 'true',
	du_single_cart: 'true',
	du_omit_empty: 'true',
	du_merge_size_into_paper: 'true',
	du_thumbs_max: '24',
	du_brand: '',
	du_write_preview: 'true',
	du_confirm_pages: 'true',
	du_block_on_error: 'false',
	du_file_code: 'du_file',
	du_preview_code: 'du_preview',
	du_price_code: 'du_price',
	du_pages_code: 'du_pages',
	du_sheets_code: 'du_sheets',
	du_spec_code: 'du_spec',
	du_size_code: 'du_size',
	du_paper_code: 'du_paper',
	du_color_code: 'du_color',
	du_sides_code: 'du_sides',
	du_finishing_code: 'du_finishing',
	du_notes_code: 'du_notes' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `du_size` (required) | Which size from the rate card this product prints | The code of a size entry in your rate card, e.g. `85x11` | none (reported as missing) | **With care**: must match a size code in your rate card and the template trim |
| `du_bleed` | Bleed added on every side of the print file | A number in inches | `0.125` | **With care**: match your production |
| `du_safe` | Safe margin inside the trim | A number in inches | `0.125` | **With care** |
| `du_crop_marks` | Adds crop marks to the print file (only drawn when bleed is 0.25 or more) | `true` or `false` | `false` | **With care**: match your production |
| `du_max_mb` | Largest file a shopper can upload | A number in MB | `100` | **Yes** |
| `du_min_dpi` | Minimum resolution for images inside the document | A number | `150` | **With care**: your print quality bar |
| `du_full_width` | Hides the product image column so the tool uses the full page width | `true` or `false` | `true` | **Yes** |
| `du_single_cart` | Hides the theme Add to Cart so the shopper sees only the tool button | `true` or `false` | `true` | **Yes** |
| `du_omit_empty` | Leaves empty hidden options out of the cart, emails and order | `true` or `false` | `true` | **Yes** |
| `du_merge_size_into_paper` | Writes size and paper together on one order line, e.g. "8.5 x 11 in, Gloss" | `true` or `false` | `true` | **Yes** |
| `du_thumbs_max` | Most page thumbnails shown | A number | `24` | **Yes** |
| `du_brand` | Accent color override. Empty uses your store button color | A color, or empty | empty | **Yes** |
| `du_write_preview` | Saves a preview image for the cart and order | `true` or `false` | `true` | **Yes** |
| `du_confirm_pages` | Page-count confirmation step | `true` or `false` | `true` | **Yes** |
| `du_block_on_error` | Blocks the order when the file check finds an error | `true` or `false` | `false` | **Yes** |
| `du_file_code`, `du_preview_code`, `du_price_code`, `du_pages_code`, `du_sheets_code`, `du_spec_code`, `du_size_code`, `du_paper_code`, `du_color_code`, `du_sides_code`, `du_finishing_code`, `du_notes_code` | Template options the tool writes the file, preview, price, pages, sheets, job spec and each choice to | Option codes | the same name as the setting without `_code`, e.g. `du_file`, `du_price` | **No**: must match the options on your template; your fulfillment reads them |

Bleed and safe are plain numbers in inches.

Things that are not set here
- Rates, papers, finishing and the volume ladder live in the rate card snippet `product/document-uploader-rates`, which your site overrides with its own. Never in the mount.
- The rate card also sets how double-sided is priced (`duplex_mode`) and what the volume ladder counts (`tier_driver`).

Not yet confirmed: `du_confirm_pages` and `du_block_on_error`: the exact effect on the page is not confirmed.

### Booklet Uploader
Status: live, version 1.5.2 (per the CHANGELOG). To be merged into Uploader 2.0. Snippet: `product/booklet-uploader`. Mount option: `bu_mount`. Prefix: `bu_`.
The shopper uploads a booklet PDF, confirms the page count, chooses cover, paper, color and binding, sees the price and adds it to the cart. Always full width on the page, always double-sided. One product, one template, one trim size per mount.

Example mount (the rate card block sits above the snippet call in the same field; see below):
```liquid
{% snippet 'product/booklet-uploader',
	bu_size: '55x85',
	bu_bleed: '0.125',
	bu_safe: '0.125',
	bu_crop_marks: 'false',
	bu_max_mb: '100',
	bu_min_dpi: '150',
	bu_full_width: 'true',
	bu_single_cart: 'true',
	bu_omit_empty: 'true',
	bu_merge_size_into_paper: 'true',
	bu_thumbs_max: '24',
	bu_brand: '',
	bu_write_preview: 'true',
	bu_confirm_pages: 'true',
	bu_block_on_error: 'false',
	bu_file_code: 'bu_file',
	bu_preview_code: 'bu_preview',
	bu_price_code: 'bu_price',
	bu_pages_code: 'bu_pages',
	bu_sheets_code: 'bu_sheets',
	bu_spec_code: 'bu_spec',
	bu_size_code: 'bu_size',
	bu_paper_code: 'bu_paper',
	bu_color_code: 'bu_color',
	bu_binding_code: 'bu_binding',
	bu_finishing_code: 'bu_finishing',
	bu_notes_code: 'bu_notes' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `bu_size` (required) | Which size from the rate card this product prints | The code of a size entry in your rate card, e.g. `55x85` | none (reported as missing) | **With care**: must match a size code in your rate card and the template trim |
| `bu_bleed` | Bleed added on every page of the print file | A number in inches | `0.125` | **With care**: match your production |
| `bu_safe` | Safe margin inside the trim | A number in inches | `0.125` | **With care** |
| `bu_crop_marks` | Adds crop marks (only drawn when bleed is 0.25 or more) | `true` or `false` | `false` | **With care** |
| `bu_max_mb` | Largest file a shopper can upload | A number in MB | `100` | **Yes** |
| `bu_min_dpi` | Minimum resolution for images inside the document | A number | `150` | **With care**: your print quality bar |
| `bu_full_width` | Tool uses the full page width | `true` | `true` | **No**: keep `true`; the tool is built full width |
| `bu_single_cart` | Hides the theme Add to Cart so only the tool button shows | `true` or `false` | `true` | **Yes** |
| `bu_omit_empty` | Leaves empty hidden options out of the cart, emails and order | `true` or `false` | `true` | **Yes** |
| `bu_merge_size_into_paper` | Writes size and paper together on one order line | `true` or `false` | `true` | **Yes** |
| `bu_thumbs_max` | Most page thumbnails shown | A number | `24` | **Yes** |
| `bu_brand` | Accent color override. Empty uses your store button color | A color, or empty | empty | **Yes** |
| `bu_write_preview` | Saves a preview image for the cart and order | `true` or `false` | `true` | **Yes** |
| `bu_confirm_pages` | Page-count confirmation step | `true` or `false` | `true` | **Yes** |
| `bu_block_on_error` | Blocks the order when the file check finds an error | `true` or `false` | `false` | **Yes** |
| `bu_file_code`, `bu_preview_code`, `bu_price_code`, `bu_pages_code`, `bu_sheets_code`, `bu_spec_code`, `bu_size_code`, `bu_paper_code`, `bu_color_code`, `bu_binding_code`, `bu_finishing_code`, `bu_notes_code` | Template options the tool writes the file, preview, price, pages, sheets, job spec and each choice to | Option codes | the setting name without `_code`, e.g. `bu_file`, `bu_binding` | **No**: must match the options on your template; your fulfillment reads them |

There is no sides setting: a booklet is always double-sided.

The rate card (prices, papers, bindings, covers, volume ladder) is a JSON block in the same `bu_mount` field, written as `<script type="application/json" data-bu-rates>...</script>` above the snippet call. Edit it with the Booklet Settings editor in Custom Admin: paste the current mount, change the card, copy the whole field back. The editor stores nothing itself. If the JSON is not valid, pricing stops (it does not fall back). An older site may still read rates from the `product/booklet-uploader-rates` snippet; the tool logs that as legacy.

Things that are not set here
- Covers, papers and bindings offered, and their prices, are in the rate card, not the snippet arguments.

Not yet confirmed: `bu_full_width: 'false'`: the exact effect is not confirmed.

### Publication Upload
Status: live on publishing-program sites, version 1.15.1 (per the CHANGELOG). Snippet: `product/publication-upload`. Mount option: `pu_mount` from 1.15.0 (older installs: the `custom_script` on the `pu_text` option). Prefix: `pu_`.
For publishers who reprint their own titles: the shopper uploads the book block (and optionally a new cover) for a title, the tool checks page count, trim size, bleed, resolution and ISBN against that title, and holds the order on a hard fail.

Example mount (one template holds many titles, so the per-title values are read from design custom fields):
```liquid
{%- assign pu_asset = design.custom.pu_file -%}
{% snippet 'product/publication-upload',
	pu_isbn: design.custom.pu_isbn,
	pu_file: pu_asset | asset_url,
	pu_file_name: pu_asset,
	pu_pages: design.custom.pu_pages,
	pu_trim_width: design.custom.pu_trim_width,
	pu_trim_height: design.custom.pu_trim_height,
	pu_cover_mode: 'separate',
	pu_bleed: '3',
	pu_min_dpi: '300',
	pu_soft_dpi_floor: '150',
	pu_max_file_mb: '150',
	pu_block_on_fail: 'true',
	pu_partner: '',
	pu_generic: 'false',
	pu_upload_required: 'true'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `pu_pages` | Exact page count for the title. Blank turns the page count check off | A number, or blank | blank | **With care**: must match the title; this is the tool's only hard fail |
| `pu_trim_width`, `pu_trim_height` | Trim size of the title. Both blank turns the size check off | Numbers in mm, or blank | blank | **With care**: must match the title and your production |
| `pu_cover_mode` | Cover supplied as separate front and back, or as one spread | `separate` or `spread` | `separate` | **With care**: match how your production expects covers |
| `pu_bleed` | Expected bleed | A number in mm | `3` | **With care**: match how your artwork is built |
| `pu_min_dpi` | Images below this are flagged | A number | `300` | **With care** |
| `pu_soft_dpi_floor` | Images below this are always flagged | A number | `150` | **With care** |
| `pu_max_file_mb` | Largest file a shopper can upload | A number in MB, at most `150` (the platform limit) | `250` in 1.15.1 (being lowered to 150); set `150` | **Yes**, up to 150 |
| `pu_block_on_fail` | Holds Add to Order on a hard fail | `true` or `false` | `true` | **Yes** |
| `pu_upload_required` | `false` offers the uploader but never blocks the order | `true` or `false` | `true` | **Yes** |
| `pu_partner` | Production partner named in the shopper-facing text. Blank names nobody | Text, no apostrophes, or blank | blank | **Yes** |
| `pu_generic` | Marks the title as deliberately unspecified (reads differently from an unfinished setup) | `true` or `false` | `false` | **Yes** |
| `pu_isbn` | The title ISBN, shown in the header and matched against the uploaded file | An ISBN | blank (falls back to the ISBN in the product code) | **With care**: must be the title ISBN |
| `pu_sku` | Production SKU for the title | Text | the product code | **No**: leave unset unless your production SKU differs from the product code |
| `pu_file`, `pu_file_name` | The contents PDF already on file for the title, and the name shown for it. Blank means the shopper uploads | An asset URL (`{{ 'name.pdf' \| asset_url }}`) and a file name | blank | **With care**: must point at the right title file |
| `pu_cover_file`, `pu_cover_file_name` | The cover PDF already on file, and the name shown for it | An asset URL and a file name | blank (or the product's stored cover) | **With care** |
| `pu_cover_required` | Requires a cover file as well as the book block | `true` or `false` | `false` | **Yes** |
| `pu_cart_image` | Image shown in the cart, checkout and emails: page 1 of the upload, or the title's catalog image | `content` or `product` | `content` | **Yes** |
| `pu_support_email` | Address a shopper is pointed to when a file carries an ISBN no title has | An email address | the first email link on the page | **Yes** |
| `pu_text_code`, `pu_cover_code`, `pu_preview_code`, `pu_preflight_code`, `pu_state_code`, `pu_changes_code` | Template options the tool writes to | Option codes | the tool's standard `pu_` codes | **No**: pass only if your template renamed one |

Pass every setting every time; any left out is named in the browser console. Trim and bleed are in millimeters.

Things that are not set here
- Per-title values (ISBN, page count, trim, file) live in design custom fields and are passed in by the mount; those field definitions must exist on your site.
- A stored cover can also come from the product custom field `px_stored_cover`, which the edition roll-forward maintains.
- Older installs may still read some settings from the Publication Upload checklist keys; the mount wins.

Not yet confirmed: The built-in defaults of `pu_text_code` and the other code settings are not confirmed; keep the values your install came with.

### Flyer / Brochure
Status: live, version 1.0.3 (per the CHANGELOG; 1.0.3 changes the asset only, the snippet is the 1.0.1/1.0.2 one). Snippet: `product/flyer-brochure`. Mount option: `px_mount`. Prefix: `ff_`.
The shopper uploads a print-ready flyer, postcard or folded brochure, sees it trimmed (and folded in 3D for brochures) before ordering, and picks paper, color, sides, finish and quantity. The tool builds the print file.

Example mount (tri-fold, inches):
```liquid
{% snippet 'product/flyer-brochure',
	ff_fold: 'c',
	ff_units: 'in',
	ff_trim_w: '11in',
	ff_trim_h: '8.5in',
	ff_panels: '3.6250in,3.6875in,3.6875in',
	ff_bleed: '0.125in',
	ff_safe: '0.125in',
	ff_min_dpi: '300',
	ff_max_mb: '200',
	ff_paper: 'paper',
	ff_color: 'print_color',
	ff_qtys: '50,100,250,500,1000',
	ff_qty_default: '250',
	ff_word: 'brochures|brochure',
	ff_title: 'Tri-fold brochures',
	ff_intro: 'Upload your print-ready PDF: page 1 is the outside, page 2 the inside.',
	ff_trust: 'Checked before you order|Folded preview',
	ff_look: 'pxt-flyer'
%}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `ff_fold` (required) | The fold | `flat` (flyers, postcards), `h` (half fold), `c` (tri-fold, C-fold), `z` (Z-fold) | none (required) | **With care**: must match the product and your folding |
| `ff_units` (required) | Units shown to the shopper | `in` or `mm` | `in` | **Yes** |
| `ff_trim_w`, `ff_trim_h` (required) | Flat trimmed sheet size (unfolded) | Lengths, e.g. `11in` or `297mm` | none (required) | **With care**: must match the template and your cutter |
| `ff_panels` (required unless `flat`) | Panel widths of page 1, left to right; must add up to `ff_trim_w` | Lengths separated by commas | none (required) | **With care**: must match your fold positions |
| `ff_bleed`, `ff_safe` (required) | Bleed around the trim, and safe margin inside it | Lengths | none (required) | **With care**: match your production |
| `ff_min_dpi` (required) | Resolution of the print file | A number | none (required) | **With care**: your print quality bar |
| `ff_low_dpi` | Below this the shopper is warned the print may look soft | A number | `100` | **Yes** |
| `ff_max_mb` (required) | Largest file a shopper can upload | A number in MB | none (required) | **Yes** |
| `ff_paper` (required) | Variant (or template option) that carries the paper | A code | none (required) | **No**: must match your code exactly |
| `ff_color`, `ff_sides`, `ff_finish` | Variants for print color, printed sides (flat pieces) and lamination. Blank hides that step | Codes, or blank | blank | **No**: must match your codes. Blank is fine if you do not offer that choice |
| `ff_sides_fixed` | For a flat piece with no sides choice: always one side or both | `1` or `2`, or blank | blank (one side, unless `ff_sides` is set) | **With care**: must match what the product prints |
| `ff_hide` | Variant values the tool must not offer | `CODE:VALUE` pairs separated by spaces | blank | **Yes**, with exact codes |
| `ff_qtys` | Quantity choices, only when quantity is not already a list on the product | Numbers separated by commas | blank | **With care**: must match your pricing |
| `ff_qty_default` | Starting quantity | A number | blank | **Yes** |
| `ff_sizes` | Size switch linking sibling size pages | `WxH=url` or `WxH Name=url`, separated by `\|` | blank (no switch) | **Yes** |
| `ff_design_url` | Adds a "Design it online" link | A URL | blank | **Yes** |
| `ff_hard_proof` | Shows a "send me a printed proof" checkbox | `true` or `false` | `false` | **Yes** |
| `ff_hard_proof_code` | Text option that receives yes or no for the printed proof | An option code | blank | **No**: must match the option on your template |
| `ff_word` | What the product is called in the tool | `plural\|singular`, e.g. `brochures\|brochure` | `pieces\|piece` for flat, brochures for folded | **Yes** |
| `ff_title`, `ff_intro` | Heading and intro line | Text, no apostrophes | `Print-ready flyers and brochures` and a standard intro | **Yes** |
| `ff_trust` | Short reassurance lines in the header | Lines separated by `\|` | `Checked before you order\|Proof before you order` | **Yes** |
| `ff_look` | Extra style classes on the tool | Class names, e.g. `pxt-flyer` | blank | **Yes** |
| `ff_owns_page` | Whether the tool takes over the product page layout | `true` or `false` | `true` | **Yes** |
| `ff_file_prefix` | Start of the print file name | Text | `flyer` (flat) or `brochure` (folded) | **With care**: your production may sort by file name |
| `ff_print_code`, `ff_preview_code`, `ff_original_code`, `ff_spec_code` | Template options the print file, preview, original and job spec are written to | Option codes | `px_print_file`, `px_preview`, `px_original`, `px_spec` | **No**: your fulfillment reads these |
| `ff_add_to_cart_selector` | Which button counts as Add to Cart, on a heavily customized page | A CSS selector | blank | **No**, unless Pixfizz support asks you to |

Lengths accept inches and millimeters (`11in`, `297mm`). From 1.0.3, a mm mount shows mm everywhere.

Legacy flyer-fold 0.5.x (`product/flyer-fold`, mounted on `flyer_mount`, `flyer_` prefix) is a separate older tool being retired. Its settings are `flyer_fold` (`c`, `z` or `h`), `flyer_trim_w`, `flyer_trim_h`, `flyer_panels` (mm), `flyer_bleed`, `flyer_safe`, `flyer_units`, `flyer_variants`, plus optional `flyer_size`, `flyer_fold_name`, `flyer_title`, `flyer_intro`, `flyer_options_title`, `flyer_trust`, `flyer_crease_gsm`, `flyer_qty_default`, `flyer_qtys`, `flyer_select_url`, `flyer_select_label`, `flyer_file_code`, `flyer_preview_code`, `flyer_spec_code`. Ask Pixfizz about moving to 1.0.x rather than tuning it.

Limits of 1.0.x (tool-level)
- **No triggered child variant trees.** 1.0.x drives one code per step (`ff_paper`, `ff_color`, `ff_sides`, `ff_finish`). It cannot drive a parent value that shows a child variant (for example a paper type value that triggers a child variant holding the paper weights). 0.5.x could, through `flyer_variants` with `CODE.*` patterns. A site with a paper tree stays on 0.5.x until the tool gets a two-level paper step. *Verified by reading source (1.0.3 has no trigger or child handling) and a live lab page, 2026-11-02.*
- **File carriers are not hidden by the tool.** 1.0.x hides the store controls of its own steps, but not the file carriers. When a mount reuses a visible store file option as the print carrier (for example `ff_print_code: 'document_file_upload'`), add a style line after the snippet call in the same `custom_script` that hides that option and the preview and spec carriers. 0.5.x hid the file option in script, so a straight swap shows an extra File Upload box. *Verified by reading source; carrier swap tested in the 1.0.3 test harness, 2026-11-02.*
- **`ff_original_code` is optional.** With no such option on the template the tool skips the originals zip, so there is no edit-from-cart restore of the original file. *Verified by reading source, 2026-11-02.*

Things that are not set here
- Paper, color, sides, finish and quantity are variants on the product: they carry the shopper's choice and the price.

## 6.4 Order builders

### Design Service (Design Brief)
Status: beta, version 2.3.1 (live on the shopper24 parent; first client installs October 2026). Snippet: `product/design-brief`. Mount option: `px_mount` on new installs (older installs: `dsn_mount`). Prefix: `dsn_`.
A "we design your photo book for you" order builder. The shopper picks a book type, size and length, sends photos (or an album link), marks must-have photos, tells the story, and orders. Your team designs the book. The price is the Photo Book Studio book price plus your design fee.

Example mount (the kit value; fill in `dsn_books` with your own product ids):
```liquid
{% snippet 'product/design-brief',
	dsn_skin: 'atelier',
	dsn_types: 'hard,soft,layflat',
	dsn_books: 'hard=8.5x8.5:1001,10x10:1002|soft=8.5x8.5:1003',
	dsn_paper: '',
	dsn_pages: '',
	dsn_auto_pages: '60',
	dsn_units: 'in',
	dsn_guidance: 'auto=150|40=100:50 to 70|60=120:75 to 100|80=180:120 to 140',
	dsn_min_photos: '20',
	dsn_max_must: '20',
	dsn_draft_days: '5',
	dsn_rounds: '2',
	dsn_approve_days: '21',
	dsn_occasions: 'Family year|Travel|Wedding|New baby|Birthday|Memorial|Something else',
	dsn_alt_url: '',
	dsn_alt_label: '',
	dsn_exit_url: '/',
	dsn_max_mb: '50',
	dsn_brand: '' %}
```
No apostrophes and no curly brackets anywhere in the mount values.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `dsn_skin` | The look of the brief | `atelier`, `noir`, `fresh` | `atelier` | Yes |
| `dsn_types` | Book types offered, in order; the first is preselected. A type shows only if `dsn_books` gives it a product | any of `hard`, `soft`, `layflat`, comma separated | `hard,soft,layflat` | Yes |
| `dsn_books` | Sizes offered and the Photo Book Studio product that prices each one (sizes in inches, width x height) | `type=WxH:product id,...` per type, types split by `|` | none, required | With care: every id must be a live book product of that type and size |
| `dsn_paper` | Paper option code the book is priced and printed on | option code, or `type=code` per type split by `|`, or blank when priced by pages alone | blank | With care: must match the paper option code on your book products |
| `dsn_pages` | Page counts the books can be made at; lengths outside are not offered | `type=min-max` split by `|` | blank (no limit) | With care: must match what your book products allow |
| `dsn_auto_pages` | Page count "Let the designer decide" is priced at | whole number | `60` | With care: it sets the price for that choice |
| `dsn_units` | Units shown to the shopper (sizes in `dsn_books` stay in inches) | `in`, `cm` | `in` | Yes |
| `dsn_guidance` | Length choices: page count = most photos to send : typical photos used; `auto` lets the designer decide | `auto=most`, then `pages=most:text` entries split by `|` | see example | Yes; each page count is the length the shopper pays for |
| `dsn_min_photos` | Fewest photos before the shopper can continue (an album link also counts) | whole number | `20` | Yes |
| `dsn_max_must` | Most must-have photos | whole number | `20` | Yes |
| `dsn_draft_days` | Working days to the first draft, shown on review | whole number | `5` | Yes |
| `dsn_rounds` | Rounds of changes included | whole number | `2` | Yes |
| `dsn_approve_days` | Days to approve after the first draft | whole number | `21` | Yes |
| `dsn_occasions` | Occasion choices on the Story step | text split by `|` | see example | Yes |
| `dsn_alt_url`, `dsn_alt_label` | Optional link under Start for shoppers who prefer to make the book themselves; shown only when both are set | page URL; text | blank | Yes |
| `dsn_exit_url` | Where Exit takes the shopper | page on your site | `/` | Yes |
| `dsn_max_mb` | Largest photo accepted | whole number, MB | `50` | Yes |
| `dsn_brand` | Your store styling over the skin | `key=value` pairs split by `|`; keys `primary`, `on_primary`, `tint`, `ink`, `muted`, `faint`, `canvas`, `surface`, `stage`, `stage_hi`, `line`, `line_strong`, `accent`, `heading`, `body`, `mono` (Google font names), `heading_weight`, `radius` (px), `buttons` (`upper`) | blank | Yes |
| `dsn_variant_prefix` | Prefix of the brief's variants | text | `dsn_` | No, unless renamed |
| `dsn_spec_code`, `dsn_brief_code`, `dsn_preview_code` | Options the job record, brief JSON and cart preview are written to | option code | `px_spec`, `dsn_brief`, `px_preview` | No, unless renamed |
| `dsn_sizes` | Older size list, replaced by `dsn_books` | | blank | No; use `dsn_books` |

Things that are not set here
- Your design fee: the price of the "Photo Book Design Service" product attribute. Leave the hidden "Book price" variant (`dsn_book_price`) as it is.
- Book prices: they come from your Photo Book Studio products.

### Film Order Builder
Status: in progress, version 0.2.0, no client installs. Snippet: `product/film-builder`, called from a page (not a template option mount). Prefix: `pxfb_`.
No mount settings: it reads site checklist keys under Custom Admin, which are safe display settings: `film-skin` (default `contact`), `film-accent` (color), `film-chrome` (default `none`), `film-panel` (default `true`), `film-hero-image` (asset name, default `film-hero.jpg`), `film-hero-align` (default `right`), `film-lab-name` (default your site name). The rate card is inside the tool and is not a lab setting yet.

### Film Roll Builder
Status: phase 1, version 0.1.1, live on the demo site. Snippet: `film/roll-builder`, mounted on a Pages page whose content is only `{% snippet 'film/roll-builder' %}`. Prefix: `frb_`.
No mount settings: it reads the checklist key `film-roll-builder-products` (your film product code or codes, comma separated, for example `COLOR-FILM-DEV`; must match the product code exactly) and `account-v2-film-prefs` (`TRUE` prefills each roll from the shopper's saved film preferences). Options and prices come from the product itself; a text variant with code `roll_name` on the product turns on per-roll names.

## 6.5 Product previews

### Live Finish
Status: live, version 0.26.1 (as of the CHANGELOG, live on the shopper24 parent). Snippet: `product/live-finish`. Mount option: `lf_mount` (hidden Text template option, custom_script field). Prefix: `lf_`.
Shows the shopper their own design on the main product image as the finished product: the finish catching the light, the corners, the mounting, canvas sides, a frame or an ornament, and optionally on a wall in a room. It writes nothing to the order.

Example mount (a fine art paper print):
```liquid
{% snippet 'product/live-finish',
	lf_page: 'photo',
	lf_units: 'in',
	lf_sample: 'lf-sample-metal.webp',
	lf_finish_variant: 'paper-type',
	lf_looks: 'smooth:fineart-matte,textured:textured-matte,etching:etching-matte',
	lf_look: 'fineart-matte',
	lf_corner_variant: 'none',
	lf_corners: '',
	lf_radius: '0in',
	lf_mounting_variant: 'none',
	lf_mountings: '',
	lf_mounting: 'loose',
	lf_standoff_variant: 'none',
	lf_standoff_colors: '',
	lf_standoff_color: 'silver-satin',
	lf_standoff: 'diameter:0.75in,inset:1in,depth:0.5in',
	lf_render_variants: 'none',
	lf_room: 'none',
	lf_room_label: 'Change room',
	lf_view: 'closeup',
	lf_tilt: 'on',
	lf_compare: 'on',
	lf_start: 'live',
	lf_label: 'Live Finish',
	lf_off_label: 'Standard view',
	lf_light_label: 'Move the light',
	lf_sample_note: 'Sample photo. Upload yours to see it here.',
	lf_closeup_label: 'Close-up',
	lf_wall_label: 'On the wall',
	lf_compare_label: 'Compare papers',
	lf_tilt_label: 'Tilt to light',
	lf_render_width: '1200' %}
```

How values are written: most map settings are comma lists of `value:result` pairs, where `value` is the variant value exactly as it is on the product (matched exactly, then ignoring case). A value may contain spaces but never a comma, a colon or an apostrophe. Looks are `<base>-<finish>`. Bases: `white`, `clear`, `brushed`, `acrylic`, `paper`, `metallic`, `fineart`, `textured`, `etching`, `canvas`, `wood`, and for ornaments `ceramic`, `glass`, `clearacrylic`, `crystal`. Finishes: `gloss`, `semi`, `matte`, and `pearl` (canvas only). A finish alone means the white base; a base alone means gloss.

Core settings (every mount declares these; anything left out is listed in `data-config-missing` and logged as a console error):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lf_page` | The production page that is rendered and measured. The preview shape comes from this page size less its bleed | a page name in the template | none, the preview stays off | **With care**: must name a real page of the XML template |
| `lf_units` | Unit for bare numbers in `lf_corners` and `lf_radius` | `in`, `mm` | `in` | **Yes** |
| `lf_sample` | Sample photo shown before the shopper uploads | asset name, full URL, or `none` | none (the render with placeholders shows) | **Yes**: upload your own asset first |
| `lf_finish_variant` | Variant code the finish is read from. A comma list is allowed: the first listed variant on the page wins | variant code(s) or `none` | `none` | **With care**: must match the variant code on your products exactly |
| `lf_looks` | Which look each finish value shows | `value:look` pairs | none | **With care**: values must match your variant values; an unmapped value uses the first look |
| `lf_look` | Look used when no finish variant decides. `off` means no preview on a page without a listed finish variant | a look, or `off` | `white-gloss` | **Yes** |
| `lf_corner_variant` | Variant code the corners are read from | variant code or `none` | `none` | **With care**: must match the variant code |
| `lf_corners` | Corner radius per corner value | `value:radius` pairs, for example `rounded:0.125in,square:0in`; `value:scallop@0.375in` for scalloped | none | **With care**: values must match your variant |
| `lf_radius` | Corner radius when no corner variant decides | a length | `0` | **Yes**: match the real product |
| `lf_mounting_variant` | Variant code the mounting is read from | variant code or `none` | `none` | **With care**: must match the variant code |
| `lf_mountings` | Mounting per value, with optional depth and edge color | `value:mounting`, `value:mounting@depth`, `value:mounting#edge`. Mountings: `flat`, `float`, `standoffs`, `easel`, `block`, `stand`, `floatframe`, `curved`, `curvedout`, `loose`, `board`, `standout`, `card`, `folded`, `edgeframe`, `recessed`, `floater`, `picture`, `ornament`. Edge colors: `#white`, `#black`, `#silver`, `#pewter`, a hex; floater colors `#walnut`, `#ebony`, `#maple`, `#black`, `#white`, `#silver` | none | **With care**: values must match your variant |
| `lf_mounting` | Mounting when no mounting variant decides | one mounting, optional `@depth` and `#edge` | `flat` | **Yes**: match the real product |
| `lf_standoff_variant` | Variant code the standoff cap color is read from | variant code or `none` | `none` | **With care**: must match the variant code |
| `lf_standoff_colors` | Cap per value | `value:cap`. Caps: `silver-satin`, `chrome`, `gold-satin`, `gold`, `black` | none | **With care**: values must match your variant |
| `lf_standoff_color` | Cap when no variant decides | a cap name | `silver-satin` | **Yes** |
| `lf_standoff` | Standoff size: cap diameter, inset from the trim edge to the cap center, depth off the wall | `diameter:<len>,inset:<len>,depth:<len>` | `diameter:0.75in,inset:1in,depth:0.5in` | **Yes**: match the real hardware |
| `lf_render_variants` | Variant values forced on the preview render only, for templates whose production page draws the mounting into the artwork | `variant:value` pairs or `none` | `none` | **With care**: must match variant codes and values on the template |
| `lf_room` | Rooms for the wall view, first one shown first. `desk` and `shelf` show small prints standing on a surface | `sofa`, `bedroom`, `hallway`, `desk`, `shelf` (comma list) or `none` | `none` | **Yes**: each room needs its `lf-room-<room>.webp` asset on the site |
| `lf_view` | View shown first | `closeup`, `wall` | `closeup` | **Yes** |
| `lf_tilt` | The light follows a phone's tilt | `on`, `off` | `on` | **Yes** |
| `lf_compare` | The Compare button (shows only when the product offers two or more looks) | `on`, `off` | `on` | **Yes** |
| `lf_start` | What shows first | `live` (Live Finish), `standard` (platform preview) | `live` | **Yes** |
| `lf_render_width` | Render width in px. The platform caps it at 1200 | number | `1200` | **Yes**: lower only to save bandwidth |
| `lf_label` | Button that switches Live Finish on | text | `Live Finish` | **Yes** |
| `lf_off_label` | Button that switches back to the platform preview | text | `Standard view` | **Yes** |
| `lf_light_label` | Light slider label | text | `Move the light` | **Yes** |
| `lf_sample_note` | Note over the sample photo | text | `Sample photo. Upload yours to see it here.` | **Yes** |
| `lf_closeup_label` | Close-up view button | text | `Close-up` | **Yes** |
| `lf_wall_label` | Wall view button | text | `On the wall` | **Yes** |
| `lf_compare_label` | Compare button | text | `Compare` | **Yes** |
| `lf_tilt_label` | Tilt button (iPhone) | text | `Tilt to light` | **Yes** |
| `lf_room_label` | Button that cycles the rooms | text | `Change room` | **Yes** |

Feature settings (only needed when the mount uses the feature; left out means the feature is off or uses the default shown):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lf_trim_inset` | Length taken off each side of the page trim, when the template trim box is larger than the product | a length | `0` | **With care**: must match the XML template and the finished size |
| `lf_design_looks` | Look (and optional mounting) chosen by design code, for a template shared by products with no finish variant. First match wins | `pattern:look` or `pattern:look/mounting`; `*` matches any characters; look may be `off` | none | **With care**: patterns must match your design codes |
| `lf_design_code` | Hint for the current design code | pass `design.code` (a Liquid variable, no quotes) | empty | **No**: leave as `design.code` |
| `lf_holes_variant` | Variant code that decides drilled holes | variant code or `none` | off | **With care**: must match the variant code |
| `lf_holes` | Hole set per value | `value:none`, `value:top`, `value:corners` | none | **With care**: values must match your variant |
| `lf_hole` | Hole size | `diameter:<len>,inset:<len>` | `diameter:0.1875in,inset:0.5in` | **Yes**: match the real product |
| `lf_frame` | Frame dimensions for `floatframe`, `recessed`, `floater` and `picture` | `width`, `gap`, `depth`, `lip`, `recess` as `key:<len>` pairs | floatframe `width:0.75in,gap:0.25in,depth:1.25in`; picture `width:0.75in,lip:0.25in,depth:0.875in` | **Yes**: match the real frame |
| `lf_stand` | Clear acrylic stand size for the `stand` mounting | `width:<len>,height:<len>` | `width:4in,height:0.5in` | **Yes** |
| `lf_curve` | Curve of a curved print | `radius:<len>` or `arc:<angle>deg`, plus optional `shape:concave` / `shape:convex` or `variant:<code>` | `radius:12in` | **Yes**: match the real product |
| `lf_edge_variants` | For a mounting value, the variant that holds its frame | `mounting value:variant code` pairs | off | **With care**: must match variant codes and values |
| `lf_edges` | Frame depth and color per frame value | `value:<depth>#<color>` pairs | off | **With care**: values must match your variant |
| `lf_border` | A border printed round the photo from the product choices (not on canvas) | `variant:<choice>,on:<value>|<value>,width:<choice or length>,color:<choice or hex>,stroke:<choice>=<value>@<width>#<hex>` | no border | **With care**: must match your option codes and values |

Canvas settings (needed when a `canvas` look is used):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lf_bar` | Depth of the canvas sides drawn | a length, for example `0.75in`, `1.5in` | `1.5in`, reported | **Yes**: match your stretcher bars |
| `lf_bar_variant` | Variant code the bar depth is read from | variant code or `none` | reported | **With care**: must match the variant code |
| `lf_bars` | Bar depth per value | `value:<len>` pairs | none | **With care**: values must match your variant |
| `lf_wrap` | Wrap when no variant decides | `image`, `mirror`, `stretch`, `color`, `color@<hex>` | `image`, reported | **Yes** |
| `lf_wrap_variant` | Variant code the wrap is read from | variant code or `none` | reported | **With care**: must match the variant code |
| `lf_wraps` | Wrap per value | `value:wrap` pairs | none | **With care**: values must match your variant |
| `lf_wrap_color_variant` | Variant a bare `color` wrap takes its color from (a color picker or named colors) | variant code or `none` | reported when used | **With care**: must match the variant code |
| `lf_wrap_colors` | Named colors for that variant | `name:#hex` pairs, or `none` for a color picker | none | **Yes** |

Picture frame settings (needed with the `picture` mounting):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lf_frame_variant` | The frame color variant | variant code | first face, reported | **With care**: must match the variant code |
| `lf_frame_faces` | Frame face image and side color per frame value | `value:asset#side-hex` pairs; an entry with `/` is a path | plain frame, reported | **With care**: each image must be uploaded and drawn to the frame spec |
| `lf_frame_filter` | The page color filter whose swatches show a corner of each frame | filter name or `none` | reported | **With care**: must match the filter name |
| `lf_mat_variant` | The presentation (mat) choice, or a list | variant code(s) | no mat, reported | **With care**: must match the variant code |
| `lf_mats` | Mat window per value, as the print hangs portrait | `value:none` or `value:<w>x<h>[in]` | no mat | **With care**: must match the real mat |
| `lf_mat_color` | Mat color | hex | `#f7f5ef` | **Yes** |
| `lf_orient_variant` | The orientation choice | variant code | as the page, reported | **With care**: must match the variant code |
| `lf_orients` | How far the frame hangs turned from the page | `value:0`, `90`, `-90`, `180` | as the page | **With care**: values must match your variant |
| `lf_sample_wide` | Sample photo for a frame hanging wide | asset name or path | `lf_sample` | **Yes** |
| `lf_all_label` | Label of the all colors view | text | `All colors` | **Yes** |
| `lf_crops` | Crop shape set on the photo upload per mat value, so Adjust crops what prints | `value:<w>/<h>` pairs (`*` for any value), or `none` | nothing set | **With care**: must match the printed opening |

Ornament settings (needed with the `ornament` mounting):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lf_ornament` | Ornament thickness, ribbon width, optional disc and drilled hole | `thickness`, `ribbon`, `disc`, `hole`, `inset` as `key:<len>` pairs | `thickness:0.04in,ribbon:0.24in`, reported | **Yes**: match the real ornament |
| `lf_ornament_shape` | Where the outline and hole come from | `mask`, `drilled`, `crown` | `mask`, reported | **With care**: must match the template mask |
| `lf_ornament_sides` | One side, two sides on one page, or each side on its own page | `one`, `front,back`, `back,front`, `pages:<front>,<back>` (cards: `pages:<front>,<inside>,<back>`) | `one`, reported | **With care**: must match the template pages |
| `lf_sample_page` | Page shown with placeholders before an upload, or the sample image filling it | a page name, `none`, `fill`, `fill:all`, `frames` | `lf_page`, reported | **With care**: must name a real page |
| `lf_ribbon_variant` | Ribbon color variant | variant code or `none` | reported | **With care**: must match the variant code |
| `lf_ribbons` | Ribbon color per value | `value:#hex` or `value:#hex/#trim` pairs | `lf_ribbon` | **With care**: values must match your variant |
| `lf_ribbon` | Ribbon when no variant decides | hex | first `lf_ribbons` entry, else `#b0232c` | **Yes** |
| `lf_backdrop` | Background behind the ornament | `page:<page>`, `page:<page>@<theme id>/<product id>`, an asset name or path, or `none` | plain stage, reported | **With care**: pages and ids must exist |
| `lf_material_filter` | The materials view across materials | filter name, or `links:<Label>=<url>|<Label>=<url>` | reported | **With care**: must match the filter or real URLs |
| `lf_material_label` | Materials view label | text | `Materials` | **Yes** |
| `lf_sides_label` | Both sides view label, or side names for cards | text, or a comma list such as `Front,Back` | `Both sides` | **Yes** |
| `lf_flip_label` | Turn over button | text | `Turn over` | **Yes** |

Things that are not set here
- The preview size and shape. They are always read from the page of the XML template named in `lf_page`.
- Price. Live Finish shows the product; it never changes the price or the order.
- Room photos, frame faces and sample photos are site assets. The mount only names them.

Not yet confirmed: The full list of `lf_sample_page` values is not confirmed beyond those shown.

### Live Finish Plates
Status: live, version 0.3.0 on the shopper24 parent (0.4.x in the CHANGELOG is a prototype, not installed). Snippet: `product/live-finish-plate`. Mount option: `lfp_mount` (hidden Text template option, custom_script field). Prefix: `lfp_`.
A companion to Live Finish for products that need a real photo: it places the shopper's design onto product photos (plates) such as a mousepad on a desk or a T-shirt on a hanger, one view per plate. It writes nothing to the order.

Example mount:
```liquid
{% snippet 'product/live-finish-plate',
	lfp_page: 'wall-decor',
	lfp_plates: 'mousepad-studio,mousepad-desk',
	lfp_views: 'Studio,On the desk',
	lfp_sample: 'lf-plate-sample-dog.jpg',
	lfp_label: 'Live Finish',
	lfp_off_label: 'Standard view',
	lfp_light_label: 'Move the light',
	lfp_sample_note: 'Sample photo. Upload yours to see it here.',
	lfp_render_width: '1200' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lfp_page` | Production page rendered onto the plate | a page name in the template | none, preview off (unless `lfp_pages` is set) | **With care**: must name a real page of the XML template |
| `lfp_pages` | One page per design, for a plate showing several designs (a set of coasters). Replaces `lfp_page` | comma list of page names | `lfp_page` | **With care**: must name real pages |
| `lfp_plates` | The plates, one per view | comma list of plate names | none, preview off | **With care**: each plate is two Pixfizz-made assets (`lf-plate-<name>` photo and map); only use plate names that exist on the parent |
| `lfp_views` | Label of each view, same order as the plates | comma list of text | the plate names | **Yes** |
| `lfp_sample` | Sample photo shown before an upload; a comma list gives one per design | asset name(s) or path(s) | none | **Yes**: upload your own asset first |
| `lfp_label` | Button that switches the preview on | text | `Live Finish` | **Yes** |
| `lfp_off_label` | Button that switches back to the platform preview | text | `Standard view` | **Yes** |
| `lfp_light_label` | Close-up hint (not shown on fabric) | text | `Move the light` | **Yes** |
| `lfp_sample_note` | Note over the sample photo | text | `Sample photo. Upload yours to see it here.` | **Yes** |
| `lfp_render_width` | Render width in px | number, platform caps at 1200 | `1200` | **Yes** |
| `lfp_box` | Print size in inches, only for plates drawn at the template size (T-shirts, wall stickers) | `WxH`, for example `16x20` | the plate's own print box, with a console error | **With care**: must match the printed size of that template |

Things that are not set here
- Where the design sits on the photo. That is built into each plate's map image by Pixfizz.
- New plates. A plate is a photo plus a map made by Pixfizz; ask Pixfizz for one.

### 3D Preview
Status: live, version 0.7.1 (as of the CHANGELOG; 0.7.2 built, not live). Snippet: `product/preview-3d`. Mount option: `p3d_mount` (hidden Text template option, custom_script field). Prefix: `p3d_`.
Shows the shopper's design on a rotatable 3D product (mugs, pillows), either as the main product image or behind a View in 3D button. The artwork is the platform's own render of the production page. It writes nothing to the order.

Example mount (11 oz mug as the main image):
```liquid
{% snippet 'product/preview-3d',
	p3d_shape: 'mug',
	p3d_model: 'mug-11oz',
	p3d_page: 'front',
	p3d_units: 'in',
	p3d_trim_w: '8in',
	p3d_trim_h: '3.5in',
	p3d_color: '#f1f1ef',
	p3d_inner_color: '#f1f1ef',
	p3d_label: '11 oz ceramic mug',
	p3d_placement: 'inline',
	p3d_render_width: '1200' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `p3d_shape` | Which product shape is drawn | `mug`, `pillow` | `mug` | **With care**: must match the product |
| `p3d_model` | The 3D model drawn. Optional for a mug, required for a pillow | `mug-11oz`, `mug-15oz`, `mug-15oz-875`, `mug-20oz`, `pillow-linen-16` | none (mug built in code) | **With care**: must match the real product; each model is a Pixfizz asset on the parent |
| `p3d_page` | Template page rendered as the artwork | a page name in the template | `front` | **With care**: must name the page in the XML template |
| `p3d_units` | Unit for bare numbers | `in`, `mm` | `in` | **Yes** |
| `p3d_trim_w` | Print area width | a length | none, preview off | **With care**: must equal the page size in the XML template (checked at load) |
| `p3d_trim_h` | Print area height | a length | none, preview off | **With care**: must equal the page size in the XML template |
| `p3d_diameter` | Mug body diameter. Only without a model | a length | none | **With care**: must match the real mug |
| `p3d_height` | Mug body height. Only without a model | a length | none | **With care**: must match the real mug |
| `p3d_print_bottom` | Mug base to the bottom of the print area. Only without a model | a length | none | **With care**: must match the real mug |
| `p3d_color` | Product color (ceramic, or fabric for a pillow) | hex | `#ffffff` | **Yes** |
| `p3d_inner_color` | Inside of the mug | hex | same as `p3d_color` | **Yes** |
| `p3d_label` | Viewer title | text | blank | **Yes** |
| `p3d_placement` | Where the 3D preview appears | `inline` (replaces the main image), `image` (View in 3D button on the image), `below` (button under the gallery) | `image` | **Yes** |
| `p3d_render_width` | Artwork render width in px | number, platform caps at 1200 | `1200` | **Yes** |
| `p3d_accent_variant` | Variant whose value colors the handle and rim (accent mugs). Needs a mug model | variant code | off | **With care**: must match the variant code |
| `p3d_accent_colors` | Accent color per variant value; unlisted values stay plain | `value:#hex` pairs | none | **With care**: values must match your variant |
| `p3d_accent_parts` | Which parts take the accent | `handle`, `rim`, `handle,rim` | `handle,rim` | **Yes**: match the real mug |
| `p3d_rim_depth` | Glaze depth below the top edge | a length | `0.04in` | **Yes** |
| `p3d_handle_stub` | White stub where the handle joins the mug | a length | `0.15in` | **Yes** |
| `p3d_handle_field` | Form field (variant code or field name) that picks a sport handle. Needs a model with a sport handle | code or field name | off | **With care**: must match the field on the product |
| `p3d_handle_styles` | Handle style per field value | `value:style` pairs; styles `plain`, `football`, `baseball` | none | **With care**: values must match your field |

Things that are not set here
- The 3D models. They are made by Pixfizz and uploaded to the parent.
- The artwork. It is always the platform render of `p3d_page`, so the preview shows what prints.

Not yet confirmed: `mm` as a unit value is not confirmed; use inches.

### Keepsake Studio
Status: live, version 0.6.3 (as of the CHANGELOG; snippet 0.6.0). Snippet: `product/keepsake-studio`. Mount option: `ks_mount` (hidden Text template option, custom_script field). Prefix: `ks_`.
Replaces the main product image with a real-time 3D acrylic keepsake (a plate on a stand, or a thick acrylic block) that follows every choice the shopper makes. The artwork is the platform render of the design page, so it matches the print. It writes nothing to the order.

Example mount (acrylic plate on a stand):
```liquid
{% snippet 'product/keepsake-studio',
	ks_page: 'preview',
	ks_plate_w: '5.25in',
	ks_plate_h: '7.25in',
	ks_thickness: '0.125in',
	ks_photo_option: 'image-upload',
	ks_base_option: 'base',
	ks_bases: 'bent:bent,stand:stand,wood:wood',
	ks_hosts: '*',
	ks_fulfiller: '',
	ks_label: 'Acrylic keepsake 5 x 7' %}
```

Example mount (acrylic block):
```liquid
{% snippet 'product/keepsake-studio',
	ks_model: 'block',
	ks_page: 'print',
	ks_sample_page: 'sample',
	ks_plate_w: '5.13in',
	ks_plate_h: '7.13in',
	ks_trim: '0.065in',
	ks_thickness: '1in',
	ks_front: 'variant:printing-style,on:front,layer:Text',
	ks_photo_option: 'photo',
	ks_bases: '',
	ks_base_option: '',
	ks_base_default: '',
	ks_scenes: 'studio,desk',
	ks_hosts: '*',
	ks_fulfiller: '',
	ks_label: 'Acrylic block 5 x 7',
	ks_render_width: '1200' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `ks_model` | Plate on a stand, or a solid block | `plate`, `block` | `plate` | **With care**: must match the real product |
| `ks_page` | Design page rendered onto the keepsake. Required | a page name in the design | none | **With care**: must name a real page |
| `ks_plate_w` | Width of that page. Required | a length | none | **With care**: must match the page size (checked against the render) |
| `ks_plate_h` | Height of that page. Required | a length | none | **With care**: must match the page size |
| `ks_thickness` | Acrylic thickness | a length | `0.125in` | **Yes**: match the real product |
| `ks_photo_option` | The photo upload option on the design | option code | `image-upload` | **No**: must match the option code on your design |
| `ks_base_option` | The base (stand) option on the design. Plate only | option code | `base` | **No**: must match the option code on your design |
| `ks_bases` | Stand model per base value. Plate only | `value:model` pairs; models `bent`, `stand`, `wood`, `none` | the value itself when it is a model name, else guessed | **With care**: values must match your base option |
| `ks_base_default` | Stand when the design has no base option. Plate only | `bent`, `stand`, `wood`, `none` | `bent` | **Yes** |
| `ks_hosts` | Site host names where the preview runs | comma list of host names, or `*` / blank for everywhere | everywhere | **Yes** |
| `ks_fulfiller` | Maker name, shown only in the browser console | text | blank | **Yes** |
| `ks_label` | Product name for screen readers | text | `Acrylic keepsake` | **Yes** |
| `ks_scenes` | Views offered. Labels are fixed: Close-up, On the shelf, On the desk. A block uses `studio,desk` | `studio`, `shelf`, `desk` (comma list) | `studio,shelf` | **Yes** |
| `ks_render_width` | Render width in px | number, platform caps at 1200 | `900` | **Yes** |
| `ks_sample_page` | Page rendered before an upload. Block only | a page name, or blank for `ks_page` with the sample photo | blank | **With care**: a name that does not exist (for example `none`) falls back to `ks_page` with the sample photo, so it is a safe way to get the sample on a design with no sample page. *Corrected 2026-11-09: previously said it must name a real page. Verified by query, 2026-10-31.* |
| `ks_trim` | Cut from each side of the render. Block only | a length | none | **With care**: must match the template bleed |
| `ks_front` | The choice that prints a PDF layer on the front of a block. Block only | `variant:<code>,on:<value>,layer:<layer name>`, or `none` | everything on the back | **With care**: must match the variant code, value and the PDF layer name in the XML template |

Notes on two settings
- **`ks_photo_option` is often a design option, and its code can differ between templates of one product line.** The template page can say "There are no options for this template" while each design carries its own photo option (see `16_PRODUCT_HIERARCHY.md`); the admin options list of the template (`admin/templates/<id>/options`) shows those design-level options too. On one lab's acrylic block range the code was `upload-image` on some templates and `photo` on others, so read it per template before writing each mount. Platform-level fact, site-specific values. *Verified by query, 2026-11-03.*
- **The empty photo area is found by its grey, which differs per site.** The placeholder image is per site (one site renders 215,215,215, the platform default 210,208,208), so the tool does not rely on one hard-coded grey: from 0.7.2 it picks the most common flat light grey when the default is absent. See `61_PIXFIZZ_API.md` § 9. Tool-level. *Verified by query, 2026-10-31.*
- **`ks_front` reads only variant choices** (`variants[...]`), not template options (`template_options[...]`). A site whose layered-text choice is a template option needs an engine change before `ks_front` can follow it. Tool-level. *Verified by reading source (0.7.4), 2026-11-03.*

Things that are not set here
- The plate outline (arched, round, square). It is read from the transparent edge of the design render.
- Production. The print file is still the design's own print page.

### Bookmark Studio
Status: live, version 0.2.1 (as of the CHANGELOG). Snippet: `product/bookmark-studio`. Mount option: `bs_mount` (hidden Text template option, custom_script field). Prefix: `bs_`.
A live preview of a metal bookmark: close-up, in a book, at actual size, the back, and a hanging tassel that follows the tassel option. It writes nothing to the order.

Example mount:
```liquid
{% snippet 'product/bookmark-studio',
	bs_source: 'design',
	bs_place: 'gallery',
	bs_page: 'bookmark',
	bs_size: '1.5x6.63in',
	bs_product_size: '1.375x6.5x0.03in',
	bs_finish: 'aluminum-gloss-silver-back',
	bs_option_selector: 'auto',
	bs_tassel_options: 'tassel',
	bs_tassel_colors: 'no tassel=none|gold=gold|black=black|cream=cream|white=white',
	bs_tassel_assets: 'gold=bs-tassel-gold.webp|black=bs-tassel-black.webp|cream=bs-tassel-cream.webp|white=bs-tassel-white.webp',
	bs_scene: 'bs-scene-book.webp',
	bs_views: 'close,book,actual',
	bs_start: 'close',
	bs_labels: 'Close-up|In a book|Actual size|Drag to tilt and catch the shine|Tilt with your phone|Close to real size on most screens|See the back|See the front' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `bs_source` | What is previewed: the design on the product page, or a saved project | `design`, `project` | none, reported | **No**: `design` on a product page mount |
| `bs_place` | Where the preview shows | `gallery` (over the first gallery image), `inline` (where the snippet renders) | `gallery` for a design, `inline` for a project | **Yes** |
| `bs_page` | The page rendered: a page name for a design, a 1-based page number for a project | page name or number | none, reported | **With care**: must name the page in the XML template |
| `bs_size` | Page size including bleed | `WxHin`, for example `1.5x6.63in` | none, reported | **With care**: must match the XML template page |
| `bs_product_size` | Finished bookmark size and thickness | `WxHxDin`, for example `1.375x6.5x0.03in` | none, reported | **With care**: must match the real product |
| `bs_finish` | Surface and back. Words in the value set the look: `matte`, `satin` or `semi`, otherwise gloss; `metal` or `alumin` draws metal edges; `silver` or `white-back` sets the back | text such as `aluminum-gloss-silver-back` | `metal-gloss` | **Yes**: match the real product |
| `bs_option_selector` | How the tassel option is found on the page | `auto`, or an element id | none, reported | **Yes**: leave `auto` |
| `bs_tassel_options` | The tassel option code | option code | none, reported | **No**: must match the option code on the template |
| `bs_tassel_colors` | Tassel color per option value | `value=color` pairs separated by `|`; `none` for no tassel | none, reported | **With care**: values must match your option values |
| `bs_tassel_assets` | Tassel image per color | `color=asset` pairs separated by `|` | none, reported | **With care**: each image must be uploaded to the site |
| `bs_scene` | Background photo for the in a book view | asset name | none, reported | **Yes**: upload your own asset first |
| `bs_views` | Views offered, in order | `close`, `book`, `actual` (comma list) | `close,book,actual` | **Yes** |
| `bs_start` | View shown first | `close`, `book`, `actual` | `close` | **Yes** |
| `bs_labels` | The 8 interface labels, in this order: close-up, in a book, actual size, drag hint, phone tilt hint, actual size note, see the back, see the front | 8 texts separated by `|` | none, reported | **Yes**: keep all 8 in order |
| `bs_project_id`, `bs_design_id`, `bs_product_id`, `bs_timestamp` | Identify the saved project on an edit page. Project source only | Liquid values from the page | blank | **No**: set by the page, not by hand |

Things that are not set here
- Production. The platform renders and fulfills from the template as before.

Not yet confirmed: The meaning of each of the 8 labels is confirmed only by their order in the example.

## 6.6 Platform extensions

Kiosk Mode, Prints Review and the Size Picker have no template, so they have no mount. Their switches and settings are site checklist keys under Custom Admin, set up by Pixfizz when the extension is activated. A per-key reference for them is planned for a later update of this file; until then, ask Pixfizz support before changing one.

## 6.7 Not configurable yet

Specified or in prototype, with no settings to change: Acrylic Statue Designer, Collage Wall, Team Cards, Restoration Estimator.

---

## Changelog

- 2026-11-03: File created. Customer-facing settings reference for every studio, custom design tool, order builder and product preview: where the mount lives, how to change it safely, the Yes / With care / No rating, and the full settings list per tool. Settings, accepted values and defaults verified by reading the tool source in the design tools repo, 3 Nov 2026; items not confirmed are marked per tool. Source: claude-chat, tool source.
- 2026-11-09: Business Cards designs file per trim size. Collage Studio known issues (frame wider than header on Bootstrap-breakpoint themes, hide the new design collection). Photo Book Studio collections settings only in `#pbs-config` from 3.4.0, 3D-only landing hero from 4.0.1 (CORRECTED: `hero3d` block no longer needed). Uploader 2.0 reads product `px_rates` before the mount card. Flyer / Brochure 1.0.x limits (no child variant trees, carriers not hidden, `ff_original_code` optional). Keepsake notes on `ks_photo_option` as a design option and `ks_front` reading variants only. Round 2: CORRECTED Keepsake `ks_sample_page` (a missing page name falls back to `ks_page` with the sample photo); placeholder grey differs per site. Source: claude-chat.
