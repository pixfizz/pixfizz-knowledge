# 16 — Product Hierarchy

**Authority Scope:** Product object relationships and catalog structure only.

_Last updated: 2026-09-09_

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
**Template options and design options are one object type.** The same option, with the same fields (value type, custom fields, order, hide flags), attached either to the template (applies to every design on it) or to one design (that design only). An option **code must be unique across both levels**: adding a design option with a code the template already uses fails with "code already used". To move an option between levels, export it, delete it, then create or import it at the new level. *Stated by Alex and verified by reading an export, 2026-09-28.*

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

**A template import is not atomic.** The full import creates the template and its options
first, then the designs and products. When a later record fails, everything created before it
stays on the site: a design that failed left the template, its options and the earlier
designs behind; a product that failed validation answered 500 and left the template with no
products and no designs. **Delete the partial template before importing again**, or every
code in the second import gets a `-1` suffix. The 500 page gives a reference; Admin → Error
Query with that reference shows the exception. *Verified by query, 2026-10-05 and 2026-10-06.*

**Design names must be unique per site.** When a second template carries designs whose names
match designs already on the site, the importer appends `-1` to the **name** only; codes are
untouched. Rename them afterwards on the design page; the platform accepts the duplicate name
on edit. The storefront URL slug keeps the `-1` (`theme=<id>-name-badge-1`), which is harmless.
*Verified by query, 2026-10-05.*

**Fonts and images in the archive are added to the site under new ids**, and `font=` in the
page XML is remapped to them. *Verified by query, 2026-10-05.*

The admin import routes, including design-only and product-only imports, are in
`18_ADMIN_NAVIGATION.md`.

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
