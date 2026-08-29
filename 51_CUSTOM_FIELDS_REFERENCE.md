# Pixfizz Shopper v2 Custom Fields Master Reference

**Last Updated:** 2026-06-30

---

## Table of Contents

1. [Summary & Statistics](#summary--statistics)
2. [Key Notes](#key-notes)
3. [Object Type Reference](#object-type-reference)
   - [Product (81 total)](#product-81-total)
   - [Collection (53 fields)](#collection-53-fields)
   - [Design (25 fields)](#design-25-fields)
   - [Post (36 fields across groups)](#post-36-fields-across-groups)
   - [Option (13 fields)](#option-13-fields)
   - [Cart (11 fields)](#cart-11-fields)
   - [User (7 fields)](#user-7-fields)
   - [Order (4 fields)](#order-4-fields)
   - [Address (3 fields)](#address-3-fields)
   - [Subcollection (6 fields)](#subcollection-6-fields)
   - [Promotion (6 fields)](#promotion-6-fields)
   - [Custom_Page (5 fields)](#custom_page-5-fields)
   - [Blog_Post Detail (8 fields)](#blog_post-detail-8-fields)
   - [Service Detail (9 fields)](#service-detail-9-fields)
   - [Business Detail (2 fields)](#business-detail-2-fields)
   - [Value (3 fields)](#value-3-fields)
   - [Variant (3 fields)](#variant-3-fields)
   - [Template_Option (2 fields)](#template_option-2-fields)
   - [Static_Product (2 fields)](#static_product-2-fields)
   - [Promo (2 fields)](#promo-2-fields)
   - [Google_Summary (2 fields)](#google_summary-2-fields)
   - [Top_Collection (1 field)](#top_collection-1-field)
   - [Project (1 field)](#project-1-field)
   - [Page (1 field)](#page-1-field)
   - [School (1 field)](#school-1-field)
   - [Club (1 field)](#club-1-field)
   - [Charity (1 field)](#charity-1-field)
   - [First_Static_Product (1 field)](#first_static_product-1-field)
   - [Next_Option (1 field)](#next_option-1-field)
   - [Previous_Option (1 field)](#previous_option-1-field)
4. [Next Steps](#next-steps)

---

## Summary & Statistics

This reference documents **30 object access patterns** mapping to approximately **15 actual CMS object types** discovered during comprehensive v2 codebase analysis.

| Object Type | Field Count | Notes |
|-------------|-------------|-------|
| Product | 81 (67 in code + 14 export-only) | Largest object type |
| Collection | 53 | Second largest |
| Post (all groups) | 36 | Shared across 7 post group types |
| Design | 25 | Theme/design configuration |
| Option | 13 | Product option handling |
| Cart | 11 | Checkout form inputs |
| Blog_Post | 8 | Detail fields |
| Service | 9 | Detail fields |
| User | 7 | Checkout form inputs |
| Subcollection | 6 | Collection child object |
| Promotion | 6 | Post group type |
| Custom_Page | 5 | Custom page fields |
| Order | 4 | Order data fields |
| Value | 3 | Option value representation |
| Address | 3 | Shipping/billing address |
| Variant | 3 | Cart item variant (via option.variant) |
| Template_Option | 2 | Cart item template option (via option.template_option) |
| Static_Product | 2 | Accessed in specific contexts |
| Promo | 2 | Promotion context variant |
| Google_Summary | 2 | Google review summary |
| Business | 2 | Detail fields |
| Single-field objects | 11 | Top_Collection, Project, Page, School, Club, Charity, First_Static_Product, Next_Option, Previous_Option |

**Total: 30 access patterns across ~15 actual CMS object types**

---

## Key Notes

### Object Type Relationships

- **Post Object Family**: `post`, `blog_post`, `service`, and `business` are all **the same CMS Post object** accessed in different page contexts (blog listing, service listing, business directory, or promotions modal). Custom field definitions are shared across all post groups.

- **Post Groups**: The Post object stores data using different field naming conventions depending on context:
  - Blog posts: `blog_*` fields
  - Service posts: `service_*` fields
  - Business posts: `business_*` fields
  - FAQ posts: `faq_*` fields
  - Promotion posts: `promo_*` fields
  - Our Work posts: `work_*` fields
  - Google Review posts: `google_*` fields

- **Promo Variants**: `promo` and `promotion` are different variable names for **Post objects** in the promotions modal context. `promo` accesses minimal fields; `promotion` accesses full post context.

- **Option Nesting**: `template_option` and `variant` are accessed via `option.template_option` and `option.variant` respectively in cart item loops.

- **Collection Nesting**: `subcollection` and `top_collection` are Collection objects accessed as children and parents of a parent collection.

- **Product Context**: `first_static_product` is a Product object accessed as the first static product in a collection.

- **Checkout Form Input**: Cart and User custom fields are **set via HTML form inputs** during the checkout process, not via CMS admin interface. These fields store shopper-provided data.

- **There is no `html` field type.** Confirmed twice (custom type schema editor, and the Product Attributes → Custom Field Schema → New Field dropdown). The type list is exactly **text · multitext · boolean · number · asset · snippet**. Entries in this file previously typed as `html` have been corrected to `snippet`.

- **What each non-string type returns in Liquid**: `snippet` returns the **rendered snippet**; `asset` returns an **`Asset` object** (pass it through the `asset_url` filter); everything else returns a plain string, or nil. A filename held in an `asset`-type field and the same filename held in a `text`-type field both work through `asset_url`, but they are not the same value — one is an object, one is a string. Pick one type per field and keep every product consistent, or fallback logic such as `{% if x != blank %}` behaves differently product to product.

- **The `Public` flag controls non-admin EDIT rights, not storefront visibility.** `product.custom.<field>` renders from Liquid whether or not Public is ticked. Leave it unticked for any field that exists purely to feed a template, which is nearly all of them. Tick it only where a non-admin user is meant to change the value themselves. The name reads like a visibility switch, and the cautious default of ticking it "so the storefront can see it" silently hands edit rights on template-critical fields to non-admin users.

- **Large content: snippet-type only.** A snippet-type custom field holds 20,000+ characters, returned byte-clean via `item.custom.<field>` with no entity encoding and round-tripping through `parse_json` intact. A **text-type field does not** — the practical cap is small, on the order of ~1KB. (Exact text-type limit still to be pinned; treat anything over ~1KB as snippet-type.) Snippet-type fields are conventionally used to *reference* a snippet but can also hold content directly. This is a separate limit from the ~2KB cap on **template options**.

- **Field type can differ by object for the same field name**: the product tab fields `details`, `features` and `production` render as markup at the Collection level but not at the Product level. Tab content authored as HTML belongs on the Collection, not on the individual product export.

- **New products start with blank custom field values**: field *definitions* exist on the site, but values default to blank (and boolean fields to false) on every newly created product. An export showing empty custom fields is expected behaviour, not a failed export.

- **`manage/custom-fields` is the in-CMS field authority**: the CMS carries a maintained reference page listing every custom field with its object type, field type and description, including fields that are missing from import tars. Where a field's type or purpose is ambiguous, that page outranks any export file.

- **Export-Only Fields**: Product has 14 fields defined in the CMS export but **not referenced in any template**. These are reserved for platform-level features including:
  - Gift Finder (budget_tier, gift_occasion, gift_type)
  - Lab production routing (lab_printer, lab_size, oversize)
  - Variant conversion (quantity_from_variants)
  - Future filtering and production workflows

---
- **Snippet-type custom field rendering requires a non-empty product Description**
  Custom fields of type `snippet` (features, details, options, pricing, production, additional tabs)
  are silently suppressed on the storefront if the product `description` field is empty — even if
  the snippet content is fully populated and saved in admin. Simple-type fields (production_time,
  promotion_badge, shipping flags) are unaffected. The Public toggle has no bearing on this.

  **Fix:** Add any text to the product Description before filling snippet custom field content.
  If the Description must appear blank to shoppers, use a non-breaking space or invisible character.

  RATIONALE: Silent failure with no admin warning. Repeat signal — #kb-sync message + Fireflies call (Xenia, Aug 18).
  SOURCE TYPE: slack-message + fireflies-call

  ---
## Object Type Reference

### Product (81 total)

**67 fields in code + 14 export-only fields**

#### Template-Referenced Product Fields (67)

| Field | Type | Description |
|-------|------|-------------|
| add_to_cart | boolean | Legacy flag for add-to-cart flow in saved projects |
| hide_from_search | boolean | Exclude this product from the storefront search flyout. Only affects the storefront search flyout — the product stays active elsewhere, including in POS. Per-design hiding is also supported via design.custom.hide_from_search |
| additional | snippet | 'Additional Info' tab content. Falls back to collection if not set |
| btn_add_to_cart | boolean | Show 'Add to Cart' button, skip Design Tool |
| btn_buy_now_design_later | boolean | Show 'Buy Now, Design Later' button |
| btn_design_tool | boolean | Show 'Design More' button (displayed with btn_add_to_cart) |
| cart_product_name_format | text | Format for cart line item name: 'design', 'both', or blank (default) |
| class | text | Display class for static collection filters and grouping |
| custom_pricing | text | Custom pricing text display, e.g. 'From 5 for $10' |
| details | snippet | 'Details' tab content on product page |
| disable_required_form | boolean | Bypass HTML5 form validation on design-now form |
| display_each_pricing | boolean | Show per-unit pricing on quantity selector |
| do_not_save | boolean | Add to cart only, do not create saved project |
| dynamic_production_time | boolean | Enable dynamic production time calculation |
| envelope_imprinting | text | Envelope design code (must be in 'envelopes-with-cards' collection) |
| extra_charge | number | Extra shipping or handling fee in cents |
| features | snippet | 'Features' tab content on product page |
| film_processing | boolean | Show film drop-in vs mail-in option selector |
| from_pricing | number | Minimum price for tiered pricing display |
| gallery_stacked | boolean | Stack preview images vertically instead of grid |
| google_category | text | Google Product Taxonomy code for feed |
| gtin | text | Product GTIN (Global Trade Item Number) |
| hide_design_options | boolean | Hide design options selector panel |
| hide_edit_in_cart | boolean | Hide 'Edit' button in cart for this product |
| hide_select_month | boolean | Year-only selection on calendar products, hide month selector |
| hide_tooltip | boolean | Disable option tooltips on design-now form |
| meta_description | text | SEO meta description |
| meta_title | text | SEO meta title |
| mpn | text | Manufacturer product number |
| name_alt | text | Superscript alternate name on Shop page listings |
| options | snippet | 'Options' tab content on product page |
| orientation | text | Product orientation filter value (e.g., 'landscape', 'portrait') |
| pickup_unavailable | boolean | Disable local pickup option |
| prepay_only | boolean | Force prepay checkout, disable pay-on-delivery |
| preview_alt_tag | text | Alt tag for live preview images |
| pricing | snippet | 'Pricing' tab content on product page |
| product_filters | snippet | Wildcard filters applied on product page |
| product_footer | snippet | Content footer at bottom of product page |
| product_type_custom | text | Custom product type for feed generation |
| production | snippet | 'Production & Shipping' tab content on product page |
| production_time | number | Days for production (0 = same day production) |
| promotion | snippet | Campaign promotional message displayed on product |
| promotion_badge | text | Badge text: 'On Sale', 'Best Seller', etc. |
| promotion_message | text | Custom promotional message |
| regular_pricing | text | Regular price displayed with strikethrough + SALE badge |
| remove_live_preview_img_schema | boolean | Exclude live preview image from schema.org markup |
| select_date | boolean | Show month/year selection on product page |
| select_page_count | boolean | Page count dropdown (photo books) |
| select_page_count_label | text | Custom page count label on selector |
| select_quantity | boolean | Quantity input field on product page |
| shipping | text | Legacy: 'false' disables shipping for this product |
| shipping_unavailable | boolean | Disable shipping option |
| size | text | Product size filter value (e.g., '4x6', '8x10') |
| skip_cart_redirect | boolean | Stay on product page after add-to-cart instead of redirect |
| sold_out | boolean | Disable purchase buttons, mark as sold out |
| special_promo | text | Promo message displayed below cart line item |
| spreads_as_pages | boolean | TRUE doubles the number shown in the page-count selector (display only). The submitted `book[pages]` value, pricing, and fulfillment are unchanged. |
| starting_at | number | Starting price for tiered pricing |
| strikethrough | number | Original price displayed as strikethrough |
| to_pricing | number | Maximum price for price range comparison |
| upsell_img | asset | Cart upsell banner image |
| upsell_label | text | Cart upsell call-to-action label |
| upsell_link | text | Cart upsell URL |
| url | text | Custom URL for structured data |
| url_parameter | text | URL parameter appended to product URL |
| url_path | text | Custom path for static product URL slug |
| video | asset | Product video (webm format) for gallery display |
| requires_design | boolean | Platform-level gate: holds the Add to Cart button until the orderline carries a design. Correct and safe to leave TRUE on products driven by a custom design tool — see the note below |

### `requires_design` on custom-design-tool products

`requires_design: true` makes the platform disable the Add to Cart button. That is
correct and is usually what you want, including on products driven by a custom design
tool.

A tool that submits the product form itself must **release the button before clicking
it** — a disabled button ignores `.click()` silently. If the tool's button resolver is
wrong, the release never happens and the order dies at the last step with no error.

**The symptom is: tool completes, Add to Cart does nothing, customer stays on the product
page, nothing in the console.** That symptom has been misattributed to `requires_design`
itself at least three times. It is not the flag. Verified 10 Aug 2026 by exporting two
products on the same site — one shipping orders, one failing — and diffing them field by
field: **zero differing fields, both `requires_design: true`.** The difference was
entirely in the tool's JavaScript.

Do not set `requires_design: false` as a workaround. Fix the resolver.

#### Export-Only Product Fields (14 — not referenced in templates)

Reserved for platform-level features, production routing, and future functionality.

| Field | Type | Description |
|-------|------|-------------|
| budget_tier | text | Gift Finder budget tier classification |
| category | multitext | Product categories for filtering and organization |
| gift_occasion | multitext | Gift Finder occasion tags (e.g., 'Birthday', 'Wedding') |
| gift_type | text | Gift Finder gift type classification |
| invoice_preview_page | number | Invoice preview page number for product |
| lab_printer | boolean | Lab printer workflow flag for production routing |
| lab_size | text | Lab size code for production routing decisions |
| orientation_img | asset | Custom orientation illustration image |
| oversize | text | Oversize shipping extra fee code |
| product_name | text | Override product name on collection listings |
| quantity_from_variants | boolean | Convert variants to quantity inputs for checkout |
| ratio | text | Product image aspect ratio (e.g., '4x6', '1x1') |
| shop_by_price | text | Price tier for shop price-based filtering |
| style | multitext | Left-hand side filter category values |

---

### Collection (53 fields)

| Field | Type | Description |
|-------|------|-------------|
| additional | snippet | Additional collapsible details section on product page |
| banner | asset | Banner image displayed at top of collection |
| banner_html | snippet | HTML content displayed below banner image |
| breadcrumb | string | Breadcrumb format: 'product' or design name |
| btn_add_to_cart | boolean | Show 'Add to Cart' on product detail page |
| btn_buy_now_design_later | boolean | Show 'Buy Now & Design Later' button on PDP |
| btn_design_tool | boolean | Show 'Design Tool' button on PDP |
| collection_filters | text | Filter configuration, one per line: "Label \| URL \| Attribute" |
| collection_footer | snippet | Footer content on collection and product pages |
| collection_pdp_filters | text | Product page-specific filter configuration |
| combine_design_static | boolean | Combine design and static products in same listing |
| contact_label | string | Label for contact section on design-now page |
| custom_filters | snippet | Custom filter UI rendered alongside standard filters |
| design_now_disable_required | boolean | Skip required field validation on design-now |
| details | snippet | 'Details' tab content on product page |
| disable_live_preview_on_shop | boolean | Hide live preview on collection listing page |
| disable_required_form | boolean | Skip HTML5 required field validation |
| dynamic_production_time | boolean | Calculate production time dynamically |
| features | snippet | 'Features' section content on product page |
| gallery_stacked | boolean | Display gallery images vertically instead of grid |
| google_category | string | Google Product Taxonomy category for schema.org |
| help_video | snippet | Embedded help video on product detail page |
| hide_collection_filters | boolean | Hide sidebar filter panel entirely |
| hide_delivery_options | boolean | Hide delivery/shipping option section on PDP |
| hide_design_options | boolean | Hide design options selector panel |
| hide_pricing | boolean | Hide pricing information on collection listings |
| hide_product_name | boolean | Hide product name on collection listings |
| load_more | boolean | Enable infinite scroll pagination |
| meta_description | string | SEO meta description for collection page |
| meta_title | string | SEO meta title for collection page |
| options | snippet | 'Options' tab content on product page |
| pdp_layout | boolean | Dual-mode product detail page layout |
| pricing | snippet | 'Pricing' tab content on product page |
| print_ux | boolean | Special photo prints product detail layout |
| production | snippet | 'Production & Shipping' tab on product page |
| production_time | number | Production time in days (0 = same-day) |
| production_time_custom | string | Custom production time text display |
| promotion | snippet | Promotional content displayed on cart page |
| promotion_message | string | Limited-time offer message |
| quickview | boolean | Enable quickview modal on collection listings |
| remove_live_preview_img_schema | boolean | Exclude preview images from schema.org markup |
| reverse | boolean | Reverse product sort order in collection |
| select_quantity | boolean | Enable quantity selector on checkout form |
| show_sub_collections | boolean | Display subcollections on collection page |
| starting_at | number | Starting price display |
| sticker_1 | asset | Badge/sticker image 1 |
| sticker_2 | asset | Badge/sticker image 2 |
| sub_collections | boolean | Enable subcollections with detail view |
| sub_collections_position | text | Subcollection render order relative to products on a collection page: ABOVE = before products, blank or BELOW = after products (default). Only applies when show_sub_collections is also enabled. Created on the parent, overridden per child site/collection. |
| subtitle | string | Subtitle displayed under collection heading |
| title_format | string | Title display format: 'design', 'both', 'collection', or blank |
| url_path_parameter | string | URL parameter added to product links |
| wildcard | boolean | Enable wildcard carousel display |
| wildcard_filter | boolean | Enable wildcard filtering on product page |

---

### Design (26 fields)

| Field | Type | Description |
|-------|------|-------------|
| badge | string | Badge text on product cards (e.g., 'Limited Edition') |
| cart_edit_url | string | Cart edit link target: 'editor' or 'project-edit' |
| disable_live_preview_on_shop | boolean | Disable preview on collection listing page |
| hide_from_search | boolean | Exclude this specific design from the storefront search flyout (per-design override) |
| disable_required_form | boolean | Bypass form field validation |
| display_name | string | Custom name overriding design code name |
| envelope_imprinting | string | Envelope design code reference for cart display |
| extra_info | string | Extra metadata for prints JSON output |
| features | snippet | Features content (cascade: design > product > collection) |
| holiday_dates | boolean | Enable holiday date picker on design-now |
| img_alt | string | Alt text for shop gallery images |
| meta_description | string | SEO meta description for design page |
| meta_title | string | SEO meta title for design page |
| msp_code | string | Marketplace SKU code for search/discovery |
| only_add_to_cart | boolean | Show only 'Add to Cart', hide 'Design Now' option |
| only_design_now | boolean | Show only 'Design Now', hide 'Add to Cart' option |
| personal_dates | boolean | Enable personal date picker on design-now |
| preview_alt_tag | string | Alt text for gallery preview images |
| promotion | snippet | Promotional content on cart page |
| promotion_badge | string | Badge on product cards and PDP (e.g., 'On Sale') |
| reverse_live_preview_on_shop | boolean | Swap front/back preview orientation on shop |
| shop_label | string | Custom label on product cards in shop |
| style | string | Style category for sidebar filter matching |
| url_parameter | string | Additional URL parameters for tracking |
| url_path | string | Primary custom URL slug |
| url_slug | string | Fallback URL slug when url_path not set |

---

### Post (36 fields across groups)

Post is a single CMS object type with context-dependent field naming. Fields are organized by post group type.

#### Blog Posts (blog_*)

| Field | Type | Description |
|-------|------|-------------|
| blog_date | text | Publication date |
| blog_description | text | Excerpt for blog listing |
| blog_path | text | URL slug for blog post |
| blog_thumbnail | asset | Thumbnail image for blog listing |
| blog_thumbnail_alt | text | Alt text for thumbnail image |
| blog_title | text | Blog post title |
| blog_unpublished | text | 'true' to hide from blog listing |

#### Service Posts (service_*)

| Field | Type | Description |
|-------|------|-------------|
| service_meta_description | text | SEO meta description / listing excerpt |
| service_path | text | URL slug for service page |
| service_thumbnail | asset | Thumbnail image for service listing |
| service_thumbnail_alt | text | Alt text for service thumbnail |
| service_title | text | Service name/title |
| service_unpublish | boolean | Hide service from listing |

#### FAQ Posts (faq_*)

| Field | Type | Description |
|-------|------|-------------|
| faq_content | snippet | Answer/content for FAQ item |
| faq_header | text | Question heading |
| faq_order | number | Sort order in FAQ listing |

#### Business Posts (business_*)

| Field | Type | Description |
|-------|------|-------------|
| business_meta_description | text | Business description for listing |
| business_path | text | URL slug for business directory listing |
| business_thumbnail | asset | Business photo/logo thumbnail |
| business_thumbnail_alt | text | Alt text for thumbnail |
| business_title | text | Business name |

#### Promotion Posts (promo_*)

| Field | Type | Description |
|-------|------|-------------|
| promo_end_date | text | Promotion end date |
| promo_img | asset | Promotion card background image |
| promo_message | text | Description text |
| promo_name | text | Promotion title |

#### Our Work Posts (work_*)

| Field | Type | Description |
|-------|------|-------------|
| work_description | text | Project description |
| work_image | asset | Portfolio gallery image |
| work_image_alt | text | Alt text for image |
| work_title | text | Project title |

#### Google Review Posts (google_*)

| Field | Type | Description |
|-------|------|-------------|
| google_name | text | Reviewer name |
| google_publish_time | text | Review publication date |
| google_rating | number | Star rating (1-5) |
| google_text | text | Review content/text |

#### Generic/Top Picks Posts

| Field | Type | Description |
|-------|------|-------------|
| image | asset | Post image |
| path | text | Link path/URL |
| title | text | Title |

---

### Option (13 fields)

| Field | Type | Description |
|-------|------|-------------|
| description | text | Option description shown to shopper |
| display | boolean | Show option on product page |
| snippet | snippet | Custom HTML for option rendering |
| img | asset | Option preview image |
| img_alt | text | Alt text for preview image |
| label | text | Display label for option |
| name | text | Internal option name |
| order | number | Sort order in option list |
| required | boolean | Mark option as required |
| selected | boolean | Default selected state |
| template_option | object | Template option metadata (accessed via option.template_option) |
| type | text | Option type (select, checkbox, radio, etc.) |
| variant | object | Variant data (accessed via option.variant) |

---

### Cart (16 fields)

**Note:** Cart custom fields are set via HTML form inputs during checkout, not CMS admin.

| Field | Type | Description |
|-------|------|-------------|
| expedited_shipping | text | Expedited shipping selection from checkout form |
| gift_message | text | Gift message entered by shopper |
| gift_wrap | boolean | Gift wrap option selected |
| locale | text | Shopper's locale preference |
| note | text | Order note from checkout form |
| option1 | boolean | Generic white-labelled order flag, wording configured per client. Reaches OrderHub once whitelisted. Rolling out from 2026-08. |
| option2 | boolean | Generic white-labelled order flag. Rolling out from 2026-08. |
| option3 | boolean | Generic white-labelled order flag. Rolling out from 2026-08. |
| phone | text | Shopper phone number from form |
| preferred_delivery | text | Preferred delivery method |
| rush | boolean | Standard rush delivery tier. Mutually exclusive with `urgent`. Rolling out from 2026-08. |
| rush_production | boolean | Rush production option selected (older single-purpose flag; superseded by `rush` on sites moved to the delivery-speed radio group) |
| scheduled_delivery_date | text | Scheduled delivery date if applicable |
| special_instructions | text | Special handling instructions |
| timezone | text | Shopper's timezone for scheduling |
| urgent | boolean | Faster-than-rush delivery tier (typically same day). Mutually exclusive with `rush`. Rolling out from 2026-08. |

**Naming.** `option1`–`option3` carry no underscore. All five flags must be
lowercase and whitelisted in OrderHub before they route — see
`45_ORDERHUB.md` § The five order-level boolean slots.

**Value convention.** Write the explicit strings `true` and `false`, never blank.
Blank is not confirmed to clear a cart custom field, and a flag that sticks on
`true` keeps charging the customer after they switch back. Because `false` is a
non-empty string, every Liquid test must be `== 'true'`; a blank value means the
field was never set, which reads as off.

---

### User (7 fields)

**Note:** User custom fields are set via HTML form inputs during checkout, not CMS admin.

| Field | Type | Description |
|-------|------|-------------|
| company | text | Shopper company name from checkout |
| email | text | Shopper email address |
| first_name | text | Shopper first name |
| last_name | text | Shopper last name |
| phone | text | Shopper phone number |
| preferences | text | Shopper communication preferences |
| title | text | Shopper job title |

---

### Order (5 fields)

| Field | Type | Description |
|-------|------|-------------|
| gift_message | text | Order-level gift message |
| order_notes | text | Internal order notes |
| special_handling | text | Special handling requirements |
| store_id | text | Per-store order tracking identifier (set at order time; used for multi-location fulfillment routing). Added 2026-02-27. |
| tracking_update_preference | text | Shopper's tracking notification preference |

---

### Address (4 fields)

| Field | Type | Description |
|-------|------|-------------|
| apartment | text | Apartment/unit/suite number |
| hide_address | boolean | Hide the address lines from the customer-facing UI while keeping the address data in the backend. Used for named pickup locations where the shopper should not see the physical address, but the address is still needed for order routing/fulfillment. |
| instructions | text | Delivery instructions |
| phone | text | Address-specific phone number |

---

### Subcollection (6 fields)

**Subcollection is a Collection object accessed as a child of a parent collection.**

| Field | Type | Description |
|-------|------|-------------|
| description | text | Subcollection description |
| image | asset | Subcollection thumbnail image |
| meta_description | string | SEO meta description |
| meta_title | string | SEO meta title |
| title | text | Subcollection display name |
| url_path | text | URL slug for subcollection |

---

### Promotion (6 fields)

**Promotion is a Post object accessed in promotions modal context.**

| Field | Type | Description |
|-------|------|-------------|
| active | boolean | Promotion is active/displayed |
| discount_code | text | Discount code if applicable |
| end_date | text | Promotion end date |
| promotion_message | text | Promotion description |
| promotion_name | text | Promotion title |
| start_date | text | Promotion start date |

---

### Custom_Page (5 fields)

| Field | Type | Description |
|-------|------|-------------|
| content | snippet | Page content |
| meta_description | text | SEO meta description |
| meta_title | text | SEO meta title |
| title | text | Page title |
| url_path | text | Custom page URL slug |

---

### Blog_Post Detail (8 fields)

**Blog_Post accessed as a detail/item object (vs. the Post listing context).**

| Field | Type | Description |
|-------|------|-------------|
| author | text | Blog post author name |
| blog_date | text | Publication date |
| blog_description | text | Post excerpt |
| blog_path | text | URL slug |
| blog_thumbnail | asset | Feature image |
| blog_thumbnail_alt | text | Image alt text |
| blog_title | text | Post title |
| content | snippet | Full post content |

---

### Service Detail (9 fields)

**Service accessed as a detail/item object (vs. the Post listing context).**

| Field | Type | Description |
|-------|------|-------------|
| content | snippet | Service page full content |
| service_description | text | Service excerpt |
| service_meta_description | text | SEO meta description |
| service_path | text | URL slug |
| service_thumbnail | asset | Service image/icon |
| service_thumbnail_alt | text | Image alt text |
| service_title | text | Service name |
| contact_info | text | Service contact information |
| related_services | text | Links to related services |

---

### Business Detail (2 fields)

**Business accessed as a detail/item object (vs. the Post listing context).**

| Field | Type | Description |
|-------|------|-------------|
| business_title | text | Business name |
| business_description | text | Business description/info |

---

### Value (3 fields)

**Value is typically accessed within option value iteration.**

| Field | Type | Description |
|-------|------|-------------|
| label | text | Display label for value |
| name | text | Internal value name |
| price | number | Price adjustment if applicable |

---

### Variant (3 fields)

**Variant is accessed via `option.variant` in cart loops.**

| Field | Type | Description |
|-------|------|-------------|
| id | text | Variant identifier |
| label | text | Display label for variant |
| price_adjustment | number | Price modification from base product |

---

### Template_Option (2 fields)

**Template_Option is accessed via `option.template_option` in cart loops.**

| Field | Type | Description |
|-------|------|-------------|
| label | text | Template option display name |
| value | text | Selected template option value |

---

### Static_Product (2 fields)

| Field | Type | Description |
|-------|------|-------------|
| name | text | Static product name |
| price | number | Static product price |

---

### Promo (2 fields)

**Promo is a Post object in minimal promotions modal context (vs. full Promotion).**

| Field | Type | Description |
|-------|------|-------------|
| promo_name | text | Promotion title |
| promo_message | text | Promotion description |

---

### Google_Summary (2 fields)

**Google_Summary for Google reviews/ratings display.**

| Field | Type | Description |
|-------|------|-------------|
| rating | number | Average star rating |
| review_count | number | Number of reviews |

---

### Top_Collection (1 field)

**Top_Collection is a Collection object accessed as a parent collection.**

| Field | Type | Description |
|-------|------|-------------|
| name | text | Parent collection name |

---

### Project (1 field)

| Field | Type | Description |
|-------|------|-------------|
| name | text | Project name/title |

---

### Page (1 field)

| Field | Type | Description |
|-------|------|-------------|
| title | text | Page title |

---

### School (1 field)

| Field | Type | Description |
|-------|------|-------------|
| name | text | School name |

---

### Club (1 field)

| Field | Type | Description |
|-------|------|-------------|
| name | text | Club name |

---

### Charity (1 field)

| Field | Type | Description |
|-------|------|-------------|
| name | text | Charity organization name |

---

### First_Static_Product (1 field)

**First_Static_Product is a Product object accessed as the first static product in a collection.**

| Field | Type | Description |
|-------|------|-------------|
| name | text | First static product name |

---

### Next_Option (1 field)

| Field | Type | Description |
|-------|------|-------------|
| label | text | Next option/button label |

---

### Previous_Option (1 field)

| Field | Type | Description |
|-------|------|-------------|
| label | text | Previous option/button label |

---

## Next Steps

### Phase 1: Export Current Definitions

Export custom field definitions from CMS admin for each object type:

1. **Collection** — Export current custom field definitions
   - Target file: `__custom_field_definitions_collection.yml`
   
2. **Design** — Export current custom field definitions
   - Target file: `__custom_field_definitions_design.yml`
   
3. **Post (all groups)** — Export current custom field definitions
   - Target file: `__custom_field_definitions_post.yml`
   - Includes: blog_*, service_*, business_*, faq_*, promo_*, work_*, google_*
   
4. **Option** — Export current custom field definitions
   - Target file: `__custom_field_definitions_option.yml`
   
5. **User** — Export current custom field definitions
   - Target file: `__custom_field_definitions_user.yml`
   
6. **Order** — Export current custom field definitions
   - Target file: `__custom_field_definitions_order.yml`
   
7. **Address** — Export current custom field definitions
   - Target file: `__custom_field_definitions_address.yml`

### Phase 2: Diff & Identify Gaps

1. Compare exported definitions against this codebase reference
2. Identify missing field definitions
3. Document missing descriptions for fields lacking proper labeling
4. Flag fields with incorrect type classifications

### Phase 3: Add Missing Definitions

1. For each missing field, create definition with:
   - Proper field name (lowercase, snake_case)
   - Type (text, multitext, boolean, number, asset, snippet — there is no `html` type)
   - Description from this reference
   - Any relevant UI hints or validation rules

2. Prioritize by impact:
   - Product (65 in code)
   - Collection (53)
   - Post groups (37)
   - Design (25)
   - Option (13)

### Phase 4: Generate Import Packages

1. Create `__custom_field_definitions.yml` files for each object type
2. Package each as individual tar.gz:
   - `custom-field-definitions-collection.tar.gz`
   - `custom-field-definitions-design.tar.gz`
   - `custom-field-definitions-post.tar.gz`
   - `custom-field-definitions-option.tar.gz`
   - `custom-field-definitions-user.tar.gz`
   - `custom-field-definitions-order.tar.gz`
   - `custom-field-definitions-address.tar.gz`

3. Root-level tar structure (one file per archive):
   ```
   __custom_field_definitions_[object_type].yml
   ```

4. Import via CMS admin: Settings > Custom Fields > Import

---

**Reference Document Version:** 1.1  
**Generated From:** Comprehensive Pixfizz Shopper v2 codebase analysis  
**Maintenance:** Update this document when new custom fields are added or existing fields are modified.

---

## Site-Specific Custom Field Patterns

These are custom field patterns seen on individual client sites. They are **not
platform-level** and should not be added to the master Shopper v2 baseline — but
they are worth documenting as real-world examples for similar use cases.

### Wholesale / Retail / Margin Pattern (2026-02-10)

A wholesale/retail client site uses Product-level custom fields to store pricing inputs that
feed into the pricing formulas:

| Field | Type | Purpose |
|-------|------|---------|
| wholesale_price | number | Base wholesale cost (source of truth for margin calcs) |
| retail_price | number | Retail price for the product |
| margin | number | Margin % or fixed-margin adjustment used by the pricing formula |

These fields are **exposed in the template-import flow** so they can be set in bulk
when importing product templates, and they are **referenced directly from the Ruby
pricing formula** on the product (e.g. `retail_price` or `wholesale_price * (1 + margin/100.0)`).

This is a useful reference pattern for any wholesale/retail dual-pricing scenario
where the same product is sold under both models — the pricing variable lives on
the product as a custom field rather than as a global Price Variable, keeping
per-SKU variation local to the product.

## The per-product export archive is also an import format

Manage Products → Product Attributes → *product* → **Export** produces an
archive that **imports** back through Product Attributes → **Import**, one
product per archive. Unlike the Static Product Importer CSV — which creates flat
products with a single price and cannot create variants — this format carries
`variant_types`, each with a full `variant_values` list. A size ladder is
therefore expressible, which makes a large catalogue a generated artifact rather
than a five-minute-per-product hand build.

**Archive shape** (gzipped `.tar.gz`, same five-empty-media-directory convention
as the Custom Type instance archive):

```
./assets/  ./fonts/  ./glb_files/  ./images/  ./pdfs/     (all empty)
./__product.yml
```

`__product.yml` holds the product row, its `custom:` hash, `linked_assets`, and
`variant_types`, then the four `__*_map: {}` keys.

**Verified behaviour:**

- **A blank `id:` is accepted** at product, variant-type and variant-value
  level; the platform assigns its own. No id reservation, no collision handling.
- **An asset-type custom field is set from a plain filename string**, with
  `__asset_map: {}` left empty. The asset must already exist in Website →
  Assets, so import the CMS backup (which carries `asset_files/`) **before** the
  products.
- **Unset custom fields simply do not appear.** The `custom:` hash of a fresh
  product contains only the boolean schema fields at `false`. Absence is not an
  error and is not the same as an empty string.

**Untested — flag before relying on:** whether re-importing an archive whose
`code` already exists updates in place or creates a duplicate; whether
`linked_assets` drives the Preview Images panel; whether `image:` accepts a bare
filename the way an asset-type field does.

**If generating the YAML, reproduce Ruby Psych's whitespace exactly.** Psych
writes a nil as `key: ` (key, colon, one **trailing space**) and a mapping or
sequence key as `key:` with no trailing space. Python's `yaml.dump` gets both
wrong. Parse a real export, re-emit it, assert byte-identical, and refuse to run
otherwise — and keep that reference export next to the generator, or the
assertion has nothing to assert against.

**Naming rule that pairs with this:** put print dimensions in the variant name
(`8x10`, `16 x 20 in`, `50 x 70 cm`) so size-aware storefront features can read
them off the platform's own rendered controls rather than needing injected data
attributes.

---

## Fields Seen Live But Absent From This Reference

Recorded so the next person does not conclude they do not exist. **Unconfirmed —
each needs a one-object test before being relied on.**

| Object | Field | Seen | Status |
|---|---|---|---|
| Collection | `unpublished` | 2026-08-26, in a live collection's `custom` hash | **Unconfirmed** whether the Shopper `/site/shop` index respects it. Not the same as `blog_unpublished` or `service_unpublish`, both of which are listed above. |

This one matters because Shopper generates its shop index from **all** collections,
so a private or corporate collection is publicly listed by default unless something
hides it. Test on one collection before promising it to a client.

## Changelog
- 2026-06-01: Added Collection field sub_collections_position (subcollection render order). Source: chat/slack/call.
- 2026-06-30: Added hide_from_search boolean (Product + Design) — excludes a product/design from the storefront search flyout. Deployed platform-wide on Shopper. Source: claude-chat, slack-message (#development).
- 2026-07-04: Clarified hide_from_search scope — affects the storefront search flyout only; the product remains available in POS. Source: Fireflies (2026-07-01).
- 2026-07-11: Added Address field hide_address (boolean) — suppresses address display in the customer-facing UI for pickup locations while keeping the backend address for order routing (Address count 3 → 4). Source: slack-message (#development, 2026-07-10).
- 2026-07-25: Added Product field spreads_as_pages (boolean, display-only page-count doubling on the page selector). Removed blog_meta_description from both the Blog Posts group and Blog_Post Detail tables — the field does not exist; Shopper SEO Settings exposes blog_post.custom.description and blog_post.custom.blog_description instead. Normalised Product counts to 81 total / 67 template-referenced (summary stats row had drifted to 79/65), Post to 36, Blog_Post Detail to 8. Source: claude-chat, fireflies-call.
- 2026-07-28: Added Key Notes entries — product tab fields (`details`, `features`, `production`) are snippet-type at Product level and html-type at Collection level; new products start with blank custom field values by design; `manage/custom-fields` is the in-CMS authority for field types and descriptions. Source: claude-chat.
- 2026-08-14: Added the checkout delivery-speed and generic order flags to the Cart table (`rush`, `urgent`, `option1`, `option2`, `option3`; count 11 → 16), with the no-underscore naming rule, the OrderHub lowercase/whitelist dependency, and the explicit `true`/`false` value convention. Noted `rush_production` as the older single-purpose flag it supersedes. Source: fireflies-call (2026-08-13), claude-chat.
- 2026-08-11: Corrected the field type list — there is no `html` type; all 18 table rows typed `html` changed to `snippet`, and the Phase 3 type list corrected to text/multitext/boolean/number/asset/snippet. Added Key Notes for what each non-string type returns in Liquid, the `Public` flag controlling non-admin edit rights rather than storefront visibility, and large-content capacity being snippet-type only (text-type caps around 1KB). Added Section — the per-product export archive as a bulk-creation format carrying variant types and values. Source: claude-chat (Shopper v2 verification kit, art-archive build).
- 2026-08-21: Added snippet-type custom field rendering gotcha (requires non-empty Description). Source: slack-message + fireflies-call.
- 2026-08-29: Added Fields Seen Live But Absent From This Reference — `unpublished` observed in a live Collection `custom` hash, distinct from `blog_unpublished` and `service_unpublish`, with the open question of whether the Shopper shop index respects it (Shopper lists all collections by default). Source: claude-chat.
