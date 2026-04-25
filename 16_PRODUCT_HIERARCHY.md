# 16 — Product Hierarchy

**Authority Scope:** Product object relationships and catalog structure only.

_Last updated: 2026-03-30_

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

---

## Variants vs Options

Same mechanism, different objects:
- **Variants** live on Product Attributes — commercial choices (size, finish) that affect pricing
- **Options** live on Templates or Designs — production-level choices

This split avoids redundancy when multiple Product Attributes share a Template. Common production options live on the Template; product-specific pricing/commercial choices live on each Product Attribute.

---

## Typical Flow for a Design Product

1. **Create a Template** — define production specs (dimensions, DPI, XML definition)
2. **Add Designs to the Template** — each design is a customer starting point
3. **Create a Product Attribute** — set pricing, variants, packaging; link to Template
4. **Add to a Collection** — publish on storefront
5. **Customer personalizes and purchases** — browse collection → select design → personalize → add to cart

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
