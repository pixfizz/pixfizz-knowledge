# 16 — Product Hierarchy

**Authority Scope:** Product object relationships and catalog structure only.

_Last updated: 2026-10-22_

---

## The Four Core Objects

Pixfizz Core organizes products through a relationship model of four objects. These are not a strict top-down hierarchy — they form a flexible relationship model where objects can be shared.

### Product Attributes
The commercial side of a product — how it appears in the store, how it's priced, and what options are available.
- Name, Code, Description
- Pricing (base price, currency)
- Variants (size, finish, material — customer-facing options that affect pricing)
- Packaging (physical packaging specs)
- Template Link — connects to its production specification
- Preview Images
- Custom Fields

A Product Attribute linked to a Template = **design product** (requires personalization).
A Product Attribute without a Template link = **static product** (standard eCommerce item like frames, gift vouchers, accessories).

**A custom design tool product is always a design product**, even when the platform renders no production file from it: the tool's own template can have every set `fulfillment="false"` and still be a design product. Rule of thumb: if the tool produces a production file, or needs a template per trim size, it is a design product. Reasons: a static product cannot sit in a mixed collection with design products; `custom_script` on a static product's variant is invisible in admin; and orderlines store design choices in `chosen_template_options` and static choices in `chosen_variants`, so anything reading an orderline must handle both. *Decided by Alex, 2026-09-20.*

### Templates
The production specification — the technical blueprint for how personalized artwork is generated.
- Name, Code, Category
- XML Definition — layer structure defining editable and fixed elements
- Page Range — min/max pages (for multi-page products)
- DPI — resolution for production output
- Units (mm, inches, pixels)
- Product/Print Dimensions
- Design Tool Configuration — which design tool profile to use
- Cut Print settings
- Fulfillment Transformations — rules for processing artwork for production
- Options — production-level choices (not customer-facing pricing options)
- Preview Images and Preview Sections
- Mapped Previews — advanced preview configuration

Multiple Product Attributes can share the same Template (e.g. "metal print" and "acrylic print" with identical production specs but different pricing).

### Designs
What the end customer starts from when personalizing. Each Design sits under a Template.
- **Pages** — what the customer sees and edits. Each page has layers: preview-clean, preview, print.
- **Layouts** — switchable content arrangements per page (e.g. one photo centered, two side-by-side, three in grid). Can be linked across designs and organized into folders using tags.
- **Backgrounds** — background images/colors customers can apply
- **Clipart** — decorative elements customers can add
- **Masks** — shape masks controlling image crop/framing
- **Design Options** — variant-like choices at the design level
- **Fulfillment Transformations** — design-specific overrides to Template fulfillment rules
- Preview Images, Custom Fields

A Template can have one Design (e.g. single full-bleed metal print) or hundreds (e.g. greeting card with designs for every occasion).

### Collections
Grouping mechanism for publishing products to the storefront.
- Name, URL path
- Description, asset image
- Preview Images, Custom Fields

Publishing a product = adding it to a Collection. Storefront URL structure: `/shop/:collection/:product/:design`.

**A top-level collection has no verified unpublish flag.** The `unpublished` collection custom field works on sub-collections only. An empty top-level collection renders a blank shop page at `/site/shop/<path>` with status 200, so "unpublish the empty collections" is not a launch step; remove or repoint the links to them instead (`80_ONBOARDING.md` § Launch Check: Empty Collections and Unpriced Products). *Verified by query, 2026-10-04.*

---

## Variants vs Options

Same mechanism, different objects:
- **Variants** live on Product Attributes — commercial choices (size, finish) that affect pricing
- **Options** live on Templates or Designs — production-level choices

This split avoids redundancy when multiple Product Attributes share a Template. Common production options live on the Template; product-specific pricing/commercial choices live on each Product Attribute.
**Template options and design options are one object type.** The same option, with the same fields (value type, custom fields, order, hide flags), attached either to the template (applies to every design on it) or to one design (that design only). An option **code must be unique across both levels**: adding a design option with a code the template already uses fails with "code already used". To move an option between levels, export it, delete it, then create or import it at the new level. *Stated by Alex and verified by reading an export, 2026-09-28.* **A template page that says "There are no options for this template" can still have options on its designs** (design page, section **Design Options**), and the product page then renders them (upload controls included). Before concluding a product has no upload options, open each design at `/site/<site>/admin/print_theme/theme/<id>`; their edit URLs still sit under `/templates/<template id>/options/<option id>/edit`. *Verified by query, 2026-10-16.*

**A template option appears on every product that uses the template.** One template often serves several Product Attributes (metal, acrylic, wood and canvas on one wall-decor template), so a template option, including a `custom_script` mount, runs on all of them. Before adding one to a live range, list which product lines share each template (`/v1/themes/<id>` gives the print product for each theme). *Verified by query, 2026-09-27.*

---

## Typical Flow for a Design Product

1. **Create a Template** — define production specs (dimensions, DPI, XML definition)
2. **Add Designs to the Template** — each design is a customer starting point
3. **Create a Product Attribute** — set pricing, variants, packaging; link to Template
4. **Add to a Collection** — publish on storefront
5. **Customer personalizes and purchases** — browse collection → select design → personalize → add to cart

---

## Import Behavior: Ids in an Import Tar Are Not Honored

**Platform-level rule. Verified live, 1 September 2026.**

Ids written into an import tar are ignored. On import the admin assigns a new id, and
where it finds an object that already exists it auto-appends `-1`, `-2` and so on to
**both the code and the name** of a Template, Product Attribute or Design.

Two consequences to design around:

- **A re-import of a corrected tar does not update the existing object** — it creates a
  suffixed duplicate. A correction means deleting the old object first, not re-importing
  over it.
- **The suffix lands on the code as well as the name**, so a silent duplicate import
  breaks any storefront Liquid or collection path that references that code. Check for
  `-1` suffixes after a batch import.

Fixed ids can therefore be carried unchanged across a generated set of tars without
collision, because they are discarded either way.
**Asset ids are the exception: the importer matches them.** When a tar's `__asset_map` id already
exists on the site, the imported records point at that existing asset. An id the site does not
have creates one new asset with a new site id, and later tars that carry that site id reuse it
rather than copying it. A 90-template range imported with 25 assets, not 90 copies. So the
importer does create assets from a populated `assets/` directory. For a generated range, import
the first tar, read the new asset id from an export, and write that id into the rest.
*Verified by query, 2026-09-26.*

**A template export carries its product and design only when asked.** The export link on a
template page opens a modal; the underlying URL is
`/print_theme/export_print_product/<template>?print_theme_ids[]=<design>&product_ids[]=<product>`.
Without those two parameters the archive has `products: []` and `print_themes: []`. Design ids
are the `print_theme/theme/<id>` links on the template's page. *Verified by query, 2026-09-26.*
With `print_theme_ids[]` and no `product_ids[]` the export carries only the chosen designs and no
products: the quick way to pull a few designs out of a very large template. Such a template-level
tar (template header plus `print_themes`, `products: []`) imports as a new template with its own
category and all its designs, and no product attribute. *Verified by import, 2026-10-20.*

**A template import is not atomic.** The full import creates the template and its options
first, then the designs and products. When a later record fails, everything created before it
stays on the site: a design that failed left the template, its options and the earlier
designs behind; a product that failed validation answered 500 and left the template with no
products and no designs. **Delete the partial template before importing again**, or every
code in the second import gets a `-1` suffix. The 500 page gives a reference; Admin → Error
Query with that reference shows the exception. *Verified by query, 2026-10-05 and 2026-10-06.*
Two more failures that leave a partial template, both verified on the demo site:

- **A variant type without `element_substitutions: []`, `translations: []` and `children: []`**
  (on the variant type itself, after `variant_values`) makes `import_print_product` answer 500.
  The template and its product attribute are created, the designs are not. The same file with
  the three empty lists added imports with 200 and every design. When writing a variant type
  into an import file by hand, copy every key a real export carries. *Verified by query,
  2026-10-20.*
- **`image: null` on a product (`products[].image`) or a design (`print_themes[].preview_img`)**
  fails with `Mysql2::Error` "Column 'image' cannot be null"; the template and its template
  options were created, the product and design were not. Write `''` instead
  (`19_XML_TEMPLATE_REFERENCE.md` has the matching `nil` versus `''` table for products).
  *Verified by query, 2026-10-22.*

**Design names must be unique per site.** When a second template carries designs whose names
match designs already on the site, the importer appends `-1` to the **name** only; codes are
untouched. Rename them afterwards on the design page; the platform accepts the duplicate name
on edit. The storefront URL slug keeps the `-1` (`theme=<id>-name-badge-1`), which is harmless.
*Verified by query, 2026-10-05.* The suffix counts per name across the **whole site**, not per
template: the sixth design with one name gets `-5`. Renaming through the design form (all fields
re-posted, only `theme[name]` changed) keeps code, description and editor configuration.
*Verified by query, 2026-10-13 and 2026-10-16.* The count holds even after the older designs were
renamed (*verified by query, 2026-10-18*). Product attribute names count the same way: on a site
that already held the template and product from an earlier import, a second import gave the
template code `-1` and the product name `-2`. Rename the product before publishing it.
*Stated by Alex, 2026-10-22.*

**Check the target site for an older copy before importing.** A site that already holds a copy of
the template with the same codes would get a full set of `-1` codes from a second import (the
suffix rule above; this second import was not run). When only an option is new (a custom tool
mount, for example), import the source template's options export onto the existing template
instead (template page, **Import Options**; `18_ADMIN_NAVIGATION.md` § Bulk Update Tools).
*Verified live, 2026-10-18.*

**A design import carries the design custom field `url_path`.** Importing a new version of a design
next to the old one gives both the same `url_path`, and the clean product URL then resolves to
whichever the platform picks (`url_path` must be unique on the site, `50_SHOPPER_TEMPLATE_REFERENCE.md`
§ 24). Before or right after importing a replacement set, move the old designs' `url_path` aside
(for example `<code>-v10`) through the design custom fields form
(`POST /site/<site>/admin/print_theme/update/<id>`, all fields re-posted). A design with `url_path`
set redirects to its clean URL, which 404s while the design is in no collection; to check an
imported design that is not yet published, open it in the admin editor,
`/v1/editor?sitename=<site>&theme_templates=<design id>`, and close it without saving.
*Verified by query, 2026-10-18.*

**Fonts and images in the archive are added to the site under new ids**, and `font=` in the
page XML is remapped to them. *Verified by query, 2026-10-05.*

**An unpublished design is skipped on import, silently.** A design exported with `publish: false`
is not created by `import_print_product`: the template and product attribute are created, the
design is not, and no error is shown. Publish designs before exporting, or set `publish: true` in
a generated export. *Verified by query, 2026-10-14.*

**A price variable the target site does not define fails the import, part way.** A product price
formula that uses a site-specific price variable missing on the destination fails with
`ActiveRecord::RecordInvalid: Validation failed: Price isn't valid` after the template and its
options are already created (no design, no product). Replace or create site-specific price
variables before importing a template export onto another site, and delete the partial template
before retrying. *Verified by query, 2026-10-14.*

**Fulfillment in a template moved between sites.** The template's fulfillment code travels with a
full template import, even when the import flash says "No fulfillment transformations have been
configured"; the target site still needs that destination in its dropdown. The products'
`fulfillment_setting_id` and `design_fulfillment_setting_id` are source-site ids: blank them in
`__print_product.yml` before importing elsewhere, and the products use the site default.
*Verified by query, 2026-10-10 and 2026-10-13.*

**A template option can ride inside a template export.** Add the option entry (the same block an
options archive carries, `id` blank) under the top-level `template_options:` of
`__print_product.yml`, re-tar and import. The new template arrives with the option and its
`custom_script`, with no second options import. *Verified by query, 2026-10-13.*

The admin import routes, including design-only and product-only imports, are in
`18_ADMIN_NAVIGATION.md`.

---

## Template Export and Import Format

Platform-level (Pixfizz CMS export format). How to read and edit a template export
(`__print_product.yml`) or a design export (`__print_theme.yml`) without breaking the import.
Generator-side emission rules are in `01_CODE_GOVERNANCE_UPDATED.md` § Archive Emission.

- **It is Ruby Psych output.** Long strings are folded over several lines (a `custom_script` is
  a double-quoted scalar spread over ten or more lines), and `created_at` / `updated_at` are
  multi-line `!ruby/object:ActiveSupport::TimeWithZone` blocks (`utc`, `zone`, `time`).
  Line-by-line edits break the file: replace the whole scalar or block, then parse the file back
  before packing. A plain `yaml.safe_load` rejects the `!ruby/` tags; load with a multi-constructor
  for `!` (or Ruby `Psych.unsafe_load` with the classes stubbed). *Verified by query and test,
  2026-10-19 and 2026-10-20.*
- **`__image_map`, `__font_map` and `__asset_map` sit at the top level** of a template export,
  not per design. *Verified by reading an export, 2026-10-19.*
- **Repeated Ruby objects are written once with a YAML anchor.** The first design carries, for
  example, `zone: &1 !ruby/object:ActiveSupport::TimeZone` under a design option's timestamps,
  and later designs only `zone: *1`. Cut the second or third design out on its own and the design
  import fails with Error Query `Psych::BadAlias Unknown alias: 1` (*verified 2026-10-22*). Remove
  a `template_options` block that holds the anchor and the later aliases break the template
  import silently, with no "successfully imported" message (*verified by query, 2026-10-20*).
  Any tool that cuts a design or removes a block must put the anchored node back at the first
  remaining `*N`, and check the output with a full parse.
- **Ids are ignored on import**, variant type and value ids included: a template imported with
  another site's variant ids got new ones. Null them anyway in a file meant for another site.
  *Verified by query, 2026-10-20.* Asset ids are the exception (§ Import Behavior).
- **Design gallery images travel in a template export** taken with `print_theme_ids[]`: each
  design's `print_themes[n].linked_assets` (`thing_type` `PrintTheme`, `order`, `asset` = file
  name), the files under `assets/<asset id>`, and `__asset_map` (`'<asset id>': {:name, :description}`).
  A generated tar can therefore bring product photos onto a site with no asset upload: write
  `linked_assets`, add the bytes as `assets/<id>`, and map the ids (they are only keys inside the
  tar). A seed export brings the seed's own photos; drop those entries when the seed is only a
  structure. To get another site's asset bytes from an admin tab, fetch the file from the asset's
  admin edit page (`/site/<site>/admin/assets/<id>/edit`, same origin; find the id with
  `/site/<site>/admin/assets?search=<name>`): storefront `/fz/` URLs are signed per host
  (`61_PIXFIZZ_API.md` § Assets). *Verified by query (export read, bytes fetched, tar packed and
  read back; import not yet run), 2026-10-19.*
- The keys a hand-written variant type needs, and `image: ''` rather than `null`, are in
  § Import Behavior.

---

## Size Naming Convention

**Standing rule, stated by the platform owner 2 September 2026. Applies to every sized
range, not to one build.**

The size code and name stay in the catalog's own notation regardless of orientation. A
landscape 16x20 is still `16x20`, never `20x16`. Orientation is distinguished by
`custom.orientation` alone.

Identical codes across a portrait and a landscape template collide on import and are
auto-suffixed (above), so the **code** needs an orientation token — for example
`MetalPrint-16x20-L` — while `name` and `custom.size` stay the catalog string `16x20`.

---

## Inventory Is Tracked Per Product, Not Per Variant

**Verified by reading source (`18_ADMIN_NAVIGATION.md` § Inventory Management).**

Tracking is enabled at the **Product Attribute** level, with a single current-inventory
count per product. There is no variant-level stock.

Any attribute the lab holds separate stock counts for therefore has to be a separate
product. A stocked frame sold in four colors, with different quantities held per color,
is four products — not one product with a color variant.

Stock is decremented the **first time** an order enters Confirmed (C) or Draft (W), once
only.

---

## One Design Cannot Render Several Product Variations

**Verified live on a product detail page, 8 September 2026 (a photo lab client).**

The gallery renders `px-design-preview`, which renders the **design**. Where several
products share one design per template, the preview cannot follow the product: selecting
a different product switches the title, description, price, stock and URL parameter
correctly, and leaves the preview showing the template's own artwork.

That forces a choice, and neither half is right:

| | Live preview on | Live preview off (`disable_live_preview_on_shop`) plus per-product preview images |
|---|---|---|
| Customer sees their own photo | yes | no |
| Preview matches the selected product | no — always the template's | yes |

**The correct architecture is one template per variation.** The variation then becomes a
template switch, so it drives a live preview of the customer's own photo, and inventory
stays on the product where it has to be. The product count is unchanged. This is a
rebuild, not a field edit.

---

## Paper Is a Variant, Not a Template Per Paper

When a lab sells the same size on several papers, build **one template per size** and put paper on the product as a priced multiple-choice **variant**. Never generate a template per paper per size (one lab's range would have meant 329 imports). Fine-art prints are wall art with their own product page: one template per size, paper as a variant, base price set to the cheapest paper at that size and the others carrying a **positive** delta, because negative variant prices are not supported (`30_PRICING_ENGINE.md`). *Stated by Alex, 2026-09-22.*

## Cloning a Product

**Verified by reading source.**

The per-product export archive (Manage Products → Product Attributes → *product* →
Export) carries `variant_types` with a full `variant_values` list and imports back
through Product Attributes → Import. The **Static Product Importer CSV cannot create
variants**.

Clone a product by exporting a working one, not by CSV. The archive shape, its verified
behavior and the create-only constraint are in `51_CUSTOM_FIELDS_REFERENCE.md`.

---

## Unpublishing Does Not Make a Product Unreachable

**Verified live — client report reproduced, September 2026.**

Removing a product from every collection does not take it off the storefront. A
previously published and crawled product URL stays live and orderable: no automatic 404,
no redirect, no unavailable state. The product does drop out of the site's product feed,
so the feed self-heals and the page does not.

Suppression that exists today:

- enable inventory tracking and let stock reach zero — the preferred route; or
- set the product-level `sold_out` custom field.

A flag or redirect behavior for this is **not built**. Do not document it as existing.

**Unpublish a design from one collection without removing it.** Each collection row has a
Published tick: `PUT /site/<site>/admin/theme_categories/<collection id>/publish_theme?product_theme_id=<row id>`
with `product_theme[published]=0` and the row form's `authenticity_token`. The design leaves the
collection grid, its clean `url_path` URL returns 404, and the next website crawl drops it from
`sitemap.xml`; the old `/site/product/c/<collection>?product=<id>&theme=<id>` URL still renders by
direct link. Prefer this to the DELETE `remove_theme` call when cleaning a demo or launch site.
*Verified by query, 2026-10-13 and 2026-10-14.* The same form with `product_theme[published]=1`
publishes an unpublished row again. *Verified by query, 2026-10-20.*

**A product that was never added to any collection has no storefront URL**
(`/site/product/<id>` returns 404). So unpriced products that were never published need no
action before launch; just never add them to a collection until they are priced. The
reachability problem above applies only to a product that was published and crawled first.
*Verified by query, 2026-10-04.*

---

## Semi-Inheritance: Publishing a Parent Lab's Templates

**Verified by call, September 2026.**

For a child site to publish a parent lab's templates, the parent's Super Admin must add
that site to the template's site list. Once added, selecting the template on Publish
Products auto-populates the product code from the template code.

**That auto-populated code is editable by the child admin.** This is how print-on-demand
price lookups drift, because the parent lab prices an outsourced line by product code and
variant code against its own site rather than from the child's order. See
`32_ORDER_LIFECYCLE.md`.
Templates shared from a hub site are listed in the hub's admin template list, not on the site that sells them. *Verified by query, 2026-09-27.*

---

## Publishing Another Site's Template: What the Lab Product Gets

Platform-level (Pixfizz CMS). Applies when a lab publishes a supplier's template from Publish
Products → **Print on Demand**, and to any template consumer publishing a provider template
(`18_ADMIN_NAVIGATION.md` § Template Provider and Template Consumers). The wizard routes are in
`18_ADMIN_NAVIGATION.md` § Import and publish routes.

**The template stays the supplier's.** The new product's `print_product_id` is the supplier's
template id, so the supplier template's options, `custom_script` mounts such as Live Finish
included, apply on the lab site with no setup. *Verified by query, 2026-10-18.*

**The product is created with name, code, category and price only.** Variants and product custom
fields are not copied: a product published from a supplier template with variants came out with
`variants: []` and `custom: {}`, and a product with Size variants on the supplier published with
no variants. *Verified by query, 2026-10-19 and 2026-10-20.* Whether a POD publish should copy the
provider's variants has been raised with the core developer; until it does, add them on the lab
product:

- **Copy the variant set across sites:** `GET /site/<provider>/admin/products/<id>/variant_types/export_all`
  (a gzipped tar: `__variant_types.yml` plus value images under `assets/`), then
  `POST /site/<lab>/admin/products/<new id>/variant_types/import` with the tar as `exported_file`
  (no authenticity token in that form). **The import is additive:** run it only on a product
  with no variants, or the set is added twice. What survives the copy and how to reprice it are
  in `22_OPTION_VARIANT_RENDERING.md` § Variant Type Exports. *Verified by query, 2026-10-19.*
- **Or add them by hand** (Add Variant, then the values), with **codes that match the supplier's**,
  so fulfillment receives the right SKU. *Verified by query, 2026-10-20.*

**Product custom fields have to be copied too.** Shopper collection pages that use the size picker
read `product.custom.orientation` and `product.custom.size`; with only POD-published products in a
collection the page rendered "Select Orientation" and no tiles. Copy the old local product's
non-default custom fields with `PUT /v1/admin/products/<id>.json?sitename=<site>`
(`product[custom][key]=value`, a merge write), but **never copy `sold_out`** from a retired copy.
*Verified by query, 2026-10-18.* (Template-level effect: Shopper size picker.)

**Price.** The Product Attributes step pre-fills the price with the provider product's price
formula; a formula that uses a price variable the lab does not define answers "Price isn't valid"
(`18_ADMIN_NAVIGATION.md` § Import and publish routes). The lab changes only the price, never the
supplier's product code.

**Fulfillment.** What the lab product gets depends on the supplier's template. *Verified by query,
2026-10-19 and 2026-10-20.*

- **The supplier template carries a fulfillment code:** the wizard's Fulfillment Code row is a
  disabled text field showing the template's value. The lab product cannot change it and orders
  route to the supplier, so such a product can never be pointed at a QA destination on a demo site.
- **The supplier template has none:** the row is the editable select, pre-filled with the
  **supplier product's** fulfillment. Products published this way carried the supplier's code;
  one whose supplier product and template both had none came out with fulfillment `null`. So set
  fulfillment on the supplier **product** (not only the template) before offering it for POD, and
  on a demo site change the row to the supplier's QA destination before publishing.
- Not verified: whether a published product follows later changes to the supplier product. Treat
  it as a copy taken at publish time: check and fix per product.

The precedence rules are in `31_FULFILLMENT_ENGINE.md` § Fulfillment Code Resolution and
Precedence.

**Product code clashes.** When the lab already holds a product with the supplier's code (an older
local copy), the published product gets the code plus `-2`. A suffixed code (`-1`, `-2`) does not
work for supplier fulfillment (*stated by Alex, 2026-10-19*). Give the plain code to the product
that is live in a collection: rename the retired copy (for example `OLD-<code>`) with
`PUT /v1/admin/products/<id>.json` (`product[code]`), then set the new product to the plain code,
and check that no code is held by two products. Only the product code carries the suffix, not
variants or fulfillment. *Verified by query, 2026-10-19.*

**Swapping old local copies for POD products in a collection.** Hide the old rows with the
per-row Published tick (`publish_theme`, § Unpublishing, reversible), then re-sort the collection
with `POST theme_categories/<id>/order` sending every row id (`61_PIXFIZZ_API.md` § 13h) so the new
rows take the old sequence: collection order drives the size tiles. *Verified by query,
2026-10-18.*

---

## Custom Type Instances Sort by the Custom Field's Declared Type

**Verified by query — admin instance list, September 2026.**

Sorting follows the field's declared **type**, not what the value looks like. A text field
sorts lexicographically: 1, 10, 11, 12, 13, 2, 3. Make any field intended as a numeric
sort key a **number** type. Admin ordering corrects immediately on the change; the
front-end page can lag behind it while the CMS cache is stale.

---

## The Photo-Prints Component Cannot Deliver a Pack of N Different Photos

**Verified by call, September 2026.**

The photo-prints ordering component tiers quantity against a **single** image, so a pack
of 18, 36 or 52 different photos cannot be expressed through it.

That product has to be built as a template with one photo per page plus autofill, which
routes the customer through the editor. Border versus borderless in that pattern is a
layout, triggered by the variant option.

---

## Supporting Product Features

Beyond the core hierarchy:
- **Fonts** — upload and manage fonts for the design tool
- **Font Palettes** — curated font sets
- **Color Palettes** — color sets for text, borders, backgrounds
- **Element Substitutions** — rules for swapping design elements based on conditions
- **Calendars** — configuration for calendar-type products

---

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-09-09: Added platform import behavior — ids in an import tar are not honored and duplicate codes/names are auto-suffixed `-1`, `-2`, so a re-import duplicates rather than updates and can break code-referencing Liquid or collection paths. Added the size naming convention (catalog notation regardless of orientation, orientation carried by `custom.orientation`, orientation token in the code only). Added that inventory is tracked per product, not per variant, and is decremented once on first Confirmed or Draft. Added that one design cannot render several product variations, with the live-preview versus per-product-image trade-off and the one-template-per-variation architecture. Added product cloning via the per-product export archive rather than the Static Product Importer CSV. Added that unpublished and de-collectioned products stay purchasable via their old URL, with the two suppression routes that exist today. Added semi-inheritance of a parent lab's templates and the editable auto-populated product code. Added that Custom Type instances sort by the custom field's declared type. Added that the photo-prints component cannot deliver a pack of N different photos. Source: claude-chat, fireflies-call, slack-message.
- 2026-09-24: Added the rule that a custom design tool product is always a design product, and "Paper Is a Variant, Not a Template Per Paper". Source: claude-chat.
- 2026-09-29: Import: __asset_map ids that exist on the site are reused (importer creates assets from assets/); template export needs print_theme_ids[] and product_ids[] to carry design and product. Template and design options are one object; code unique across both; template options reach every product on the template. Hub-shared templates are listed on the hub. Source: claude-chat.
- 2026-10-06: Collections: a top-level collection has no verified unpublish flag and an empty one renders a blank 200 page. Import Behavior: template import is not atomic (delete the partial template first, Error Query reference), design names get `-1` on a name clash, fonts and images get new ids with `font=` remapped. Unpublishing: a never-collected product has no storefront URL. Source: claude-chat, vault-doc.
- 2026-10-17: Variants vs Options: a template with no options can still carry design options. Import Behavior: design-name suffix counts per name across the whole site, rename via the design form; unpublished designs are skipped silently; a missing site price variable fails the import part way; fulfillment code travels, blank `fulfillment_setting_id` ids; a template option can ride inside a template export. Unpublishing: per-collection Published tick (`publish_theme`). Source: claude-chat.
- 2026-10-22: Import Behavior: designs-only template export and template-level tars; two more partial-import failures (variant type without the three empty lists, `image: null` on a product or design `preview_img`); the name suffix also hits product names and survives renames; check the target for an older copy first; a design import carries `url_path` (move the old one aside, check unpublished designs in the admin editor). New Template Export and Import Format (Psych folding and TimeWithZone blocks, top-level maps, YAML anchors and `Psych::BadAlias`, ids ignored, design gallery images in the export). Unpublishing: `publish_theme` with `published=1` republishes. New Publishing Another Site's Template (supplier template id travels, no variants or custom fields copied with the export/import and by-hand routes, price prefill, fulfillment locked or pre-filled from the supplier product, `-2` code clashes and the rename rule, swapping old copies). Source: claude-chat.
