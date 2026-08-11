# 50 — Pixfizz Liquid Reference

**Authority Scope:** Pixfizz Liquid objects, filters, and tags.
**Source:** Compiled from https://pixfizz.notion.site/Liquid-Documentation
**Last compiled:** 2026-04-10

---

## Overview

Pixfizz extends [stock Liquid](https://shopify.github.io/liquid/) with Pixfizz-specific objects, filters, and tags.
Liquid is supported in: CMS pages/layouts/snippets, email templates, SMS templates, and fulfillment templates.
Most objects, filters, and tags behave the same in all rendering contexts unless otherwise specified.

---

## String Quoting Rules

Pixfizz Liquid strings are delimited by single quotes (`'...'`). There is **no escape character** for single quotes inside Liquid strings. Backslash escaping (e.g. `\'`) is not supported and will cause a Liquid syntax error.

**Consequences:**
- **Apostrophes and contractions** (`we'll`, `don't`, `it's`) cannot appear inside single-quoted Liquid strings.
- If a string contains an apostrophe, rephrase to avoid it (e.g. `we will` instead of `we'll`).
- This applies to all Liquid contexts: `{{ '...' | t: ns: '...' }}`, `{% assign x = '...' %}`, `{% capture %}` string arguments, etc.
- **CMS import silently rejects** any snippet file containing this error — the file will not appear in the CMS after import, with no error message during the import process.

**Example — WRONG:**
```liquid
{{ 'Answer a few questions and we\'ll recommend something.' | t: ns: 'homepage' }}
```

**Example — CORRECT:**
```liquid
{{ 'Answer a few questions and we will recommend something.' | t: ns: 'homepage' }}
```

---

# OBJECTS

---

## Address

Represents a user registration address, an order delivery address, or a store public location.

**Filtering:** `id`, `custom.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

| Property | Description |
|---|---|
| `address.id` | Unique ID |
| `address.title` | Title |
| `address.email` | Email |
| `address.first_name` | First name |
| `address.last_name` | Last name |
| `address.company` | Company |
| `address.street` | Street |
| `address.street2` | Street line 2 |
| `address.city` | City |
| `address.region` | Region |
| `address.postcode` | Postcode |
| `address.telephone` | Telephone |
| `address.country` | Associated `Country` object |
| `address.is_public` | `true` if public/system address |
| `address.custom` | `CustomFields` object |

---

## Asset

Represents a website static asset (image or file uploaded to the CMS).

**Filtering:** `name`
**Sorting:** `name`

| Property | Description |
|---|---|
| `asset.name` | Asset name |
| `asset.url` | Asset URL path |
| `asset.description` | Asset description |

**Usage:** Access via `website.assets['filename.png']` or directly on objects that return an Asset (e.g. `product.image`, `collection.image`).
**Rendering:** Use the `asset_url` filter to generate a sized/formatted URL.

---

## Cart

Represents the current user's cart. Available globally as `cart` in the CMS.

**Filtering:** `id`, `custom.<field-name>`
**Sorting:** `created_at`, `custom.<field-name>`

| Property | Description |
|---|---|
| `cart.id` | Unique ID |
| `cart.promocode_code` | Text code of applied promocode, if any |
| `cart.total` | Total (orderlines + tax + shipping - discounts). Use `currency` filter. |
| `cart.orderlines_total` | Orderlines subtotal. Use `currency` filter. |
| `cart.orderlines_discount` | Amount discounted from orderlines. Use `currency` filter. |
| `cart.shipping_discount` | Amount discounted from shipping. Use `currency` filter. |
| `cart.discount` | Total discount (always = `orderlines_discount` + `shipping_discount`). Use `currency` filter. |
| `cart.shipping` | Shipping cost. Use `currency` filter. |
| `cart.extra_fees` | Array of active `ExtraFee` objects |
| `cart.tax` | Total tax amount. Use `currency` filter. |
| `cart.user_notes` | Customer notes (empty string if none) |
| `cart.telephone` | Buyer telephone |
| `cart.created_at` | Cart creation datetime. Use `date` filter. |
| `cart.updated_at` | Last updated datetime. Use `date` filter. |
| `cart.orderlines` | Paginated collection of `Orderline` objects (excludes child orderlines, default page size 50) |
| `cart.all_orderlines` | Paginated collection of all `Orderline` objects including children (default page size 50) |
| `cart.address` | Associated `Address` object, if any |
| `cart.shipping_option` | Associated `ShippingOption` object, if any |
| `cart.available_shipping_options` | List of all available `ShippingOption` objects |
| `cart.custom` | `CustomFields` object |

---

## ChosenOption

Represents a chosen value of a variant or template option on a specific `Orderline`.

| Property | Description |
|---|---|
| `chosen_option.key` | Key (typically the option type's code) |
| `chosen_option.value` | Value stored (option value code, or user-entered text for text options) |
| `chosen_option.variant` | `OptionType` object if this is a product variant, else `nil` |
| `chosen_option.variant_value` | `OptionValue` object if multiple choice variant, else `nil` |
| `chosen_option.template_option` | `OptionType` object if this is a template option, else `nil` |
| `chosen_option.template_option_value` | `OptionValue` object if multiple choice template option, else `nil` |
| `chosen_option.option_type` | Alias: `variant` or `template_option` depending on context |
| `chosen_option.option_value` | Alias: `variant_value` or `template_option_value` depending on context |
| `chosen_option.uploaded_image` | `Image` object if `image_upload` type, else `nil` |
| `chosen_option.uploaded_file` | `UploadedFile` object if `file_upload` type, else `nil` |

**Reading a `file_upload` option's asset.** Use `chosen_option.uploaded_file.url`
and `chosen_option.uploaded_file.filename`. The obvious alternatives do not work:
`chosen_option.value` returns the raw internal reference (`db:NNN`),
`chosen_option.asset.url` and `chosen_option.thumbnail_url` are not defined on a
chosen option, and the `preview_url` filter applies to projects, not uploads.
This matters most when rendering a cart thumbnail for a file attached by a
custom design tool.

---

## ChosenOptions

Proxy that allows access to `ChosenOption` objects on an `Orderline`. Returned by `orderline.chosen_variants` and `orderline.chosen_template_options`.

```liquid
{{ orderline.chosen_variants['foil-color'].variant_value.name }}

{% for option in orderline.chosen_template_options %}
	{{ option.template_option.name }}: {{ option.value }}
{% endfor %}
```

**Shopify IDs live in `chosen_variants`.** `shopify_product_id`, `shopify_variant_id`,
and `shopify_line_id` are accessed via `orderline.chosen_variants['<key>'].value`,
not `chosen_template_options`. See `31_FULFILLMENT_ENGINE.md` for the worked example.

| Property | Description |
|---|---|
| `chosen_options.size` | Number of chosen options |

---

## Collection

Represents a collection of design/product combinations as defined in "Manage Products → Collections". Auto-defined as `collection` on CMS URLs of form `/site/<pagename>/c/<collection-path>`.

**Filtering:** `id`, `path`, `custom.<field-name>`
**Sorting:** `path`, `custom.<field-name>`

| Property | Description |
|---|---|
| `collection.id` | Unique ID |
| `collection.path` | Collection path |
| `collection.name` | Display name |
| `collection.description` | Description |
| `collection.image` | `Asset` image |
| `collection.preview_images` | Paginated collection of `Asset` objects |
| `collection.collections` | Paginated collection of direct sub-`Collection` objects |
| `collection.design_products` | Paginated collection of published `DesignProduct` objects |
| `collection.static_products` | Paginated collection of published static `Product` objects |
| `collection.custom` | `CustomFields` object |

---

## Country

Represents a country. Exposed as `website.countries`.

**Filtering:** `id`, `code`, `code3`, `name`
**Sorting:** `code`, `code3`, `name`

| Property | Description |
|---|---|
| `country.id` | Unique ID |
| `country.name` | Country name |
| `country.code` | 2-char ISO code |
| `country.code3` | 3-char ISO code |
| `country.numeric_code` | Numeric ISO code |

---

## CustomFields

Proxy for accessing custom fields on various objects. Accessed via `.custom` on most objects.

- Custom field of type `snippet` → returns rendered snippet
- Custom field of type `asset` → returns `Asset` object
- All other types → returns string value (or `nil`)

```liquid
{{ collection.custom['product-types'] }}
{% if user.custom.is_vip == 'true' %}VIP{% endif %}
<img src="{{ page.custom.banner_image | asset_url }}" />
```

---

## CustomTypeInstance

Represents an instance of an admin-defined custom type.

**Filtering:** `id`, `custom.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

| Property | Description |
|---|---|
| `custom_type_instance.id` | Unique ID |
| `custom_type_instance.custom` | `CustomFields` object |

---

## CustomTypes

Proxy object for accessing custom type instances via `website.custom_types`.

```liquid
{% for post in website.custom_types['blog_posts'] %}
	{{ post.custom.title }}
{% endfor %}
```

Returns a paginated collection of `CustomTypeInstance` objects, or `nil` if the type doesn't exist.

---

## Design

Represents a design. Auto-defined as `design` when URL query string contains `?theme=<design_id>`.

**Filtering:** `id`, `code`, `name`, `custom.<field-name>`, `template.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

| Property | Description |
|---|---|
| `design.id` | Unique ID |
| `design.code` | Code |
| `design.name` | Name |
| `design.description` | Description |
| `design.image` | `Asset` image |
| `design.preview_images` | Paginated collection of `Asset` preview images |
| `design.template` | Associated `Template` object |
| `design.template_options` | Paginated collection of published top-level `OptionType` objects |
| `design.all_template_options` | Paginated collection of all `OptionType` objects (all levels, including unpublished) |
| `design.preview_page_names` | Array of page names marked as previews, in sort order |
| `design.custom` | `CustomFields` object |

---

## DesignProduct

Represents a Design + Product combination (what "Manage Products → Products" shows in admin). Note: equivalent to "Manage Products → Products" in the admin.

**Filtering:** `id`, `sku`, `design.<field-name>`, `product.<field-name>`
**Sorting:** `order`

| Property | Description |
|---|---|
| `design_product.id` | Unique ID |
| `design_product.sku` | Unique combination of design + product codes separated by `:` |
| `design_product.design` | Associated `Design` object |
| `design_product.product` | Associated `Product` object |

---

## ExtraFee

Represents an extra fee charged on an order or associated with a cart.

| Property | Description |
|---|---|
| `extra_fee.name` | Fee name |
| `extra_fee.code` | Fee code |
| `extra_fee.is_taxable` | `true` if taxable |
| `extra_fee.amount` | Fee amount. Use `currency` filter. |

---

## ExternalFile

> **DEPRECATED.** External files are deprecated. Use file_upload variant/template option instead.

| Property | Description |
|---|---|
| `external_file.url` | Download URL |
| `external_file.page_count` | Page count as provided by user (`nil` if not provided) |
| `external_file.description` | Description (`nil` if not provided) |

---

## Form

Available as `form` inside a `{% form %}` block.

| Property | Description |
|---|---|
| `form.errors` | `FormErrors` object from previous submission (`nil` if no prior submission) |
| `form.values` | `FormValues` object from previous submission (`nil` if no prior submission) |
| `form.captcha` | Renders default captcha (currently hCaptcha) |
| `form.hcaptcha` | Renders hCaptcha widget |
| `form.recaptcha` | Renders reCaptcha widget (requires superadmin setup) |

---

## FormValues

Accessible as `form.values` inside a form block. Proxy that lets you access submitted field values by parameter name.

```liquid
{% form 'address_create' %}
  <input name="address[street]" value="{{ form.values.street | escape }}" />
{% endform %}
```

---

## Gallery

Represents an image gallery.

**Filtering:** `id`, `code`, `custom.<field-name>`
**Sorting:** `id`, `created_at`, `custom.<field-name>`

| Property | Description |
|---|---|
| `gallery.id` | Unique ID |
| `gallery.name` | Name |
| `gallery.code` | Code |
| `gallery.created_at` | Creation datetime. Use `date` filter. |
| `gallery.images` | Paginated collection of `Image` objects |
| `gallery.custom` | `CustomFields` object |

---

## GeneratedFile

Represents a file generated during fulfillment. **Only available in fulfillment templates.**

| Property | Description |
|---|---|
| `generated_file.format` | `pdf` or `jpeg` |
| `generated_file.type` | `cover` or `pageN` (N = 1-based incrementing counter) |
| `generated_file.url` | Download URL |
| `generated_file.dirname` | Directory the file was fulfilled into (typically the order code) |
| `generated_file.filename` | Name of the generated file |
| `generated_file.quantity` | Always `1` except cut prints as JPGs/single-page PDFs, where it equals the user's chosen quantity |
| `generated_file.original_filename` | Original uploaded filename (cut print JPGs only) |
| `generated_file.autorotated` | `true` if image was auto-rotated to fit (cut print JPGs only) |

---

## Group

Represents a user group.

**Filtering:** `id`, `code`
**Sorting:** `id`, `created_at`, `updated_at`

| Property | Description |
|---|---|
| `group.id` | Unique ID |
| `group.name` | Name |
| `group.code` | Code |
| `group.category` | Category |
| `group.created_at` | Creation datetime. Use `date` filter. |
| `group.updated_at` | Last updated datetime. Use `date` filter. |
| `group.projects` | Paginated collection of shared `Project` objects |

---

## Image

Represents an image uploaded to Pixfizz (user-uploaded, gallery image, etc.).

**Filtering:** `id`
**Sorting:** `created_at`

| Property | Description |
|---|---|
| `image.id` | Unique ID |
| `image.filename` | Original uploaded filename |
| `image.width` | Width in pixels |
| `image.height` | Height in pixels |
| `image.url` | Full-size image URL |
| `image.created_at` | Creation datetime. Use `date` filter. |

> Use the `thumbnail_url` filter to get sized thumbnail URLs.

---

## Order

Represents a completed order. Auto-defined as `order` in email/SMS and fulfillment templates. Accessible via `user.orders` in the CMS.

**Filtering:** `id`, `code`, `status`
**Sorting:** `created_at`

**Status codes:**
- `P` = Pending, `W` = Draft, `C` = Confirmed, `F` = Payment Failed
- `D` = Downloaded, `M` = Manufactured, `S` = Shipped
- `X` = Cancelled, `E` = Error, `R` = Refunded

| Property | Description |
|---|---|
| `order.id` | Unique ID |
| `order.code` | Order code |
| `order.status` | One-letter status code |
| `order.promocode_code` | Associated promo code, if any |
| `order.total` | Order total. Use `currency` filter. |
| `order.orderlines_total` | Orderlines subtotal. Use `currency` filter. |
| `order.orderlines_discount` | Orderlines discount. Use `currency` filter. |
| `order.shipping_discount` | Shipping discount. Use `currency` filter. |
| `order.discount` | Total discount (= orderlines_discount + shipping_discount). Use `currency` filter. |
| `order.shipping` | Shipping cost. Use `currency` filter. |
| `order.extra_fees` | Array of `ExtraFee` objects |
| `order.tax` | Total tax. Use `currency` filter. |
| `order.email` | Buyer email |
| `order.telephone` | Buyer telephone |
| `order.first_name` | Buyer first name |
| `order.last_name` | Buyer last name |
| `order.payment_gateway` | Payment gateway name (`nil` if unpaid/manual) |
| `order.payment_reference` | Payment reference from gateway |
| `order.is_paid` | `true` if paid |
| `order.is_priority` | `true` if priority flag is set |
| `order.external_source` | External source name (`shopify`, `etsy`, etc.) if applicable |
| `order.external_reference` | External order reference if applicable |
| `order.created_at` | Cart session creation datetime. Use `date` filter. |
| `order.updated_at` | Last updated datetime. Use `date` filter. |
| `order.confirmed_at` | Confirmation datetime. Use `date` filter. |
| `order.manufactured_at` | Manufactured datetime. Use `date` filter. |
| `order.shipped_at` | Shipped datetime. Use `date` filter. |
| `order.shipping_method` | Shipping method name |
| `order.user_notes` | Customer notes (empty string if none) |
| `order.notes` | Admin notes (internal — do not display to customers) |
| `order.address` | `Address` object |
| `order.orderlines` | Paginated collection of `Orderline` objects (excludes children, default page size 50) |
| `order.all_orderlines` | Paginated collection of all `Orderline` objects including children (default page size 50) |
| `order.custom` | `CustomFields` object |
| `order.fulfillment_code` | Fulfillment code for the order |

---

## Orderline

Represents an orderline in a cart or order. Atomic commerce unit in Pixfizz.

**Filtering:** `id`, `is_draft`

| Property | Description |
|---|---|
| `orderline.id` | Unique ID |
| `orderline.quantity` | Quantity |
| `orderline.units` | Sum of quantities of all orderlines for the same product |
| `orderline.page_count` | Page count |
| `orderline.is_draft` | `true` if draft status |
| `orderline.is_cut_print` | `true` if cut print product |
| `orderline.cut_print_quantity` | Quantity of cut prints (for non-cut-print projects, equals page count — not `0`) |
| `orderline.unit_price` | Unit price. Use `currency` filter. |
| `orderline.price` | Orderline total price. Use `currency` filter. |
| `orderline.barcode` | Barcode (only available after fulfillment) |
| `orderline.product` | Associated `Product` object |
| `orderline.design` | Associated `Design` object, if any |
| `orderline.project` | Associated user `Project`, if any |
| `orderline.chosen_variants` | `ChosenOptions` object for product variants |
| `orderline.chosen_template_options` | `ChosenOptions` object for template options |
| `orderline.fulfillment_code` | Fulfillment setting code that would be used now |
| `orderline.external_files` | List of `ExternalFile` objects (deprecated) |
| `orderline.generated_files` | List of `GeneratedFile` objects (fulfillment templates only) |
| `orderline.children` | Paginated collection of child `Orderline` objects |

> `orderline.preview_url` is deprecated — use the `preview_url` filter instead.

---

## OptionType

Represents a product variant type or template option type.

**Filtering:** `id`, `code`, `name`, `type`, `custom.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

**Types:** `multiple_choice`, `text`, `number`, `color`, `image_upload`, `file_upload`

| Property | Description |
|---|---|
| `option_type.id` | Unique ID |
| `option_type.type` | Type string (see types above) |
| `option_type.name` | Name |
| `option_type.code` | Code |
| `option_type.description` | Description |
| `option_type.image` | `Asset` image, if any |
| `option_type.is_required` | `true` if required |
| `option_type.is_published` | `true` if published |
| `option_type.default_value` | Default value (number/color/font types only; `nil` otherwise) |
| `option_type.placeholder` | Placeholder text (text/number types; `nil` otherwise) |
| `option_type.min_length` | Min length (text only; `nil` otherwise) |
| `option_type.max_length` | Max length (text only; `nil` otherwise) |
| `option_type.pattern` | Pattern setting (text only; `nil` otherwise) |
| `option.min` | Min value (number only; `nil` otherwise) |
| `option.max` | Max value (number only; `nil` otherwise) |
| `option.step` | Step (number only; `nil` otherwise) |
| `option_type.accept` | Accept setting (file_upload only; `nil` otherwise) |
| `option_type.crop_aspect_ratio` | Crop aspect ratio (image_upload only; `nil` otherwise) |
| `option_type.color_palette` | Associated `ColorPalette`, if any (color type only) |
| `option_type.font_palette` | Associated `FontPalette`, if any (font type only) |
| `option_type.target_element_names` | Target element names as array (text/image_upload only; `nil` otherwise) |
| `option_type.has_pricing` | `true` if this option influences price |
| `option_type.has_element_substitutions` | `true` if this option or any children trigger element substitutions |
| `option_type.trigger_value` | `OptionValue` that triggers this child option (`nil` if not a child) |
| `option_type.children` | Paginated collection of published child `OptionType` objects |
| `option_type.values` | List of published `OptionValue` objects (multiple_choice only; `nil` otherwise) |
| `option_type.all_values` | List of all (including unpublished) `OptionValue` objects (multiple_choice only) |
| `option_type.custom` | `CustomFields` object |

---

## OptionValue

Represents an available value for a multiple choice variant or template option.

| Property | Description |
|---|---|
| `option_value.id` | Unique ID |
| `option_value.name` | Name |
| `option_value.code` | Code |
| `option_value.description` | Description |
| `option_value.is_default` | `true` if this is the default value |
| `option_value.is_published` | `true` if published |
| `option_value.price` | Price. Use `currency` filter. |
| `option_value.image` | `Asset` image, if any |
| `option_value.custom` | `CustomFields` object |

---

## Paginate

Represents a paginated collection. Returned by most collection properties (e.g. `user.orders`, `cart.orderlines`). Default page size is usually 20 (cart.orderlines defaults to 50).

Items on the current page can be iterated with `{% for %}`.

| Property | Description |
|---|---|
| `paginate.current_page` | Current page number (defaults to `1`) |
| `paginate.current_offset` | Offset of first item on current page |
| `paginate.total_items` | Total items in entire collection (same as `paginate.size`) |
| `paginate.total_pages` | Total available pages |
| `paginate.page_size` | Items per page |
| `paginate.first` | First item on current page |
| `paginate.last` | Last item on current page |
| `paginate.size` | Total items in collection |

**Compatible filters:** `first`, `last`, `size`, `map`, `sort`, `where`, `page`, `page_size`, `reverse`

```liquid
{% for order in user.orders %}
  {{ order.code }}
{% endfor %}

{% assign lines = cart.orderlines | page_size: 1000 %}
{% for line in lines %}
  {{ line.product.name }}
{% endfor %}
```

---

## Product

Represents a product (equivalent to "Manage Products → Product Attributes" in admin). Auto-defined as `product` when URL contains `?product=<product_id>`.

**Filtering:** `id`, `code`, `name`, `category`, `custom.<field-name>`, `template.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

| Property | Description |
|---|---|
| `product.id` | Unique ID |
| `product.code` | Product code |
| `product.name` | Name |
| `product.description` | Description |
| `product.category` | Category |
| `product.price` | Calculated price. Use `currency` filter. |
| `product.is_static` | `true` if static product |
| `product.is_deleted` | `true` if deleted from admin |
| `product.min_quantity` | Minimum quantity |
| `product.max_quantity` | Maximum quantity |
| `product.quantity_intervals` | Array of available quantity values, or `nil` |
| `product.tracks_inventory` | `true` if inventory tracking is enabled |
| `product.current_inventory` | Current inventory count |
| `product.image` | `Asset` image |
| `product.preview_images` | Paginated collection of `Asset` preview images |
| `product.template` | Associated `Template` object (`nil` for static products) |
| `product.variants` | Paginated collection of published top-level `OptionType` objects |
| `product.all_variants` | Paginated collection of all `OptionType` objects (all levels, including unpublished) |
| `product.custom` | `CustomFields` object |

---

## Project

Represents a user-created project (book, print set, etc.).

**Filtering:** `id`, `is_saved`, `is_ordered`, `share_code`
**Sorting:** `created_at`, `updated_at`

| Property | Description |
|---|---|
| `project.id` | Unique ID |
| `project.name` | Project name |
| `project.created_at` | Creation datetime. Use `date` filter. |
| `project.updated_at` | Last updated datetime. Use `date` filter. |
| `project.share_code` | Share code |
| `project.page_count` | Page count (excludes `count="false"` pages) |
| `project.uncounted_page_count` | Count of `count="false"` pages |
| `project.double_page_count` | Count of double (`position="left-right"`) pages |
| `project.uncounted_double_page_count` | Count of pages that are both double and uncounted |
| `project.cut_print_quantity` | Total cut prints in project (equals page count for non-cut-print projects) |
| `project.is_cut_print` | `true` if cut print project |
| `project.is_saved` | `true` if explicitly saved by user |
| `project.is_ordered` | `true` if this is a copy added to cart (does not guarantee actual order placed) |
| `project.weight` | Project weight (requires template setup; `0` if not configured) |
| `project.price` | Project price. Use `currency` filter. |
| `project.preview_page_numbers` | Array of 1-based page numbers marked with `preview="true"` |
| `project.editor_page_numbers` | Array of 1-based page numbers not marked `editor="false"` |
| `project.fulfillment_page_numbers` | Array of 1-based page numbers not marked `fulfillment="false"` |
| `project.product` | Associated `Product` object |
| `project.design` | Associated `Design` object |
| `project.chosen_variants` | `ChosenOptions` for product variants |
| `project.chosen_template_options` | `ChosenOptions` for template options |
| `project.images` | Array of `Image` objects for each editable image used |
| `project.custom` | `CustomFields` object |

**Preview in email (requires share code):**
```liquid
https://{{ website.hostname }}{{ orderline.project | preview_url: height: 300, share: orderline.project.share_code }}
```

---

## Request

Contains information about the current HTTP request. Available globally as `request` in the CMS only (not in email/SMS/fulfillment).

| Property | Description |
|---|---|
| `request.url` | Current URL |
| `request.base_url` | Scheme + host (with optional port) |
| `request.scheme` | `http` or `https` |
| `request.host` | Host of the request |
| `request.port` | Port |
| `request.path` | Path |
| `request.fullpath` | Path + query string |
| `request.query_string` | Query string |
| `request.params` | Proxy for query parameters. Access with `request.params.book_id` or `request.params['param-name']` |
| `request.locale` | Two-letter code of active language |

---

## ShippingOption

Contains information about a shipping option for the cart.

| Property | Description |
|---|---|
| `shipping_option.id` | Unique ID |
| `shipping_option.name` | Name |
| `shipping_option.code` | Code |
| `shipping_option.cost` | Cost. Use `currency` filter. |

---

## Template

Contains information about a product template.

**Filtering:** `id`, `code`, `name`, `category`, `custom.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

| Property | Description |
|---|---|
| `template.id` | Unique ID |
| `template.code` | Code |
| `template.name` | Name |
| `template.description` | Description |
| `template.category` | Category |
| `template.min_page_count` | Minimum pages |
| `template.max_page_count` | Maximum pages |
| `template.default_page_count` | Default page count for new projects |
| `template.page_increments` | Pages added/removed per increment |
| `template.dpi` | Output DPI |
| `template.minimum_dpi` | Minimum DPI (used for low-res warnings) |
| `template.width` | Width of first page (variable width = minimum width; binding not included) |
| `template.height` | Height of first page (variable height = minimum height) |
| `template.is_cut_print` | `true` if cut print template |
| `template.min_cut_print_quantity` | Min prints required before add-to-cart (cut print only; `nil` otherwise) |
| `template.max_cut_print_quantity` | Max prints allowed (`0` = no limit; cut print only; `nil` otherwise) |
| `template.is_deleted` | `true` if deleted from admin |
| `template.preview_images` | Paginated collection of `Asset` preview images |
| `template.mapped_previews` | Paginated collection of `MappedPreview` objects |
| `template.custom` | `CustomFields` object |

---

## UploadedFile

Represents a file uploaded to a Pixfizz "File Upload" variant or template option.

**Filtering:** `id`, `code`
**Sorting:** `created_at`

| Property | Description |
|---|---|
| `uploaded_file.id` | Unique ID |
| `uploaded_file.filename` | Original filename |
| `uploaded_file.url` | File URL |
| `uploaded_file.created_at` | Creation datetime. Use `date` filter. |

**Resized delivery.** A Pixfizz-hosted file URL accepts a `thumbnail/{n}` path
segment, where `n` is the rendered size in pixels — `.../thumbnail/250/...`. It
is a **path** parameter, not a query parameter; `?height=N` has no effect. Use it
anywhere a full-size customer upload is displayed at thumbnail scale (cart lines,
gallery strips, review modals). On customer artwork the saving is typically an
order of magnitude. Shopify gallery previews use the same segment — see
**60_SHOPIFY_INTEGRATION.md**.

---

## User

Contains information about the current website user. Available globally as `user` in CMS, email/SMS templates, and fulfillment templates.

| Property | Description |
|---|---|
| `user.id` | Unique ID |
| `user.is_anonymous` | `true` if anonymous user |
| `user.is_logged_in` | Reverse of `is_anonymous` (also `true` for guest users) |
| `user.is_guest` | `true` if guest user |
| `user.is_admin` | `true` if admin |
| `user.external_id` | External ID (`nil` if not set) |
| `user.email` | Email |
| `user.first_name` | First name |
| `user.last_name` | Last name |
| `user.telephone` | Telephone |
| `user.category` | User category |
| `user.projects` | Paginated collection of saved `Project` objects |
| `user.all_projects` | Paginated collection of all `Project` objects (including unsaved and cart copies) |
| `user.signup_address` | Signup `Address` object |
| `user.addresses` | Paginated collection of stored `Address` objects |
| `user.orders` | Paginated collection of `Order` objects |
| `user.carts` | Paginated collection of `Cart` objects |
| `user.calendars` | Paginated collection of `Calendar` objects |
| `user.groups` | Paginated collection of `Group` objects the user belongs to |
| `user.impersonating_admin_user` | Admin `User` currently impersonating this user (`nil` if not impersonated) |
| `user.custom` | `CustomFields` object |

---

## Website

Contains general website information. Available globally as `website` in all rendering contexts.

| Property | Description |
|---|---|
| `website.title` | Website title |
| `website.hostname` | Public hostname |
| `website.pixfizz_subdomain` | The `*.pixfizz.com` subdomain |
| `website.currency_code` | Currency code (`USD`, `EUR`, etc.) |
| `website.currency_sign` | Currency sign (`$`, `€`, etc.) |
| `website.currency_formatting_settings` | `CurrencyFormattingSettings` object (renders as JSON) |
| `website.locale` | Two-letter default language code |
| `website.assets` | Access assets by name: `website.assets['logo.png']` |
| `website.collections` | Paginated collection of top-level `Collection` objects |
| `website.all_collections` | Paginated collection of all `Collection` objects ordered alphabetically by path |
| `website.design_products` | Paginated collection of all `DesignProduct` objects |
| `website.static_products` | Paginated collection of all static `Product` objects |
| `website.public_addresses` | Paginated collection of top-level public `Address` objects |
| `website.galleries` | Paginated collection of published public `Gallery` objects |
| `website.calendars` | Paginated collection of admin-created `Calendar` objects |
| `website.countries` | Paginated collection of all `Country` objects (default page size 300) |
| `website.custom_types` | `CustomTypes` proxy object |
| `website.dropbox_app_key` | Dropbox app key |
| `website.google_oauth2_client_id` | Google OAuth 2.0 client ID |
| `website.stripe_publishable_key` | Stripe Publishable Key |
| `website.square_application_id` | Square application ID (production or sandbox) |
| `website.square_location_id` | Square location ID |
| `website.square_sandbox_mode` | `false` if live Square; `true` otherwise |
| `website.authorizedotnet_api_login_id` | Authorize.net API Login ID |
| `website.authorizedotnet_public_client_key` | Authorize.net Public Client Key |
| `website.authorizedotnet_sandbox_mode` | `true` if Authorize.net sandbox mode |

---

# FILTERS

---

## asset_url

Renders a URL to a website asset. Input: asset name string or `Asset` object.

```liquid
{{ 'logo.png' | asset_url }}
{{ website.assets['hero.jpg'] | asset_url: 800 }}
{{ 'logo.png' | asset_url: '300x150' }}
{{ collection.image | asset_url }}
{{ 'image.jpg' | asset_url: cdn: false }}
{{ 'image.jpg' | asset_url: 150, cdn: false }}
{{ 'mypic.png' | asset_url: size: 250 }}
{{ 'mypic.png' | asset_url: 500, format: 'webp' }}
```

**Parameters:**
- Size: single number or `'WxH'` string
- `format`: `"jpeg"`, `"png"`, or `"webp"` (images only)
- `cdn: false` — disable CDN (use for OG/metadata/ld+json URLs)

---

## barcode_datauri

Returns a PNG data-URI of an EAN13 barcode for the given input number.

```liquid
{{ '978020137962' | barcode_datauri }}
```

---

## cms_url

Returns a URL to a CMS page. Used to build URLs for collection and product pages.

```liquid
{{ 'contact' | cms_url }}
{{ my_collection | cms_url }}
{{ my_collection | cms_url: page: 'kolekzion' }}
{{ my_design_product | cms_url, collection: my_collection }}
{{ my_product | cms_url: page: 'produkt' }}
```

**Input types accepted:** page name string, `Page` object, `Collection` object, `DesignProduct` object, `Product` object.
**Parameters:** `page`, `collection`, `product`, `design`, `design_product`

---

## currency

Formats a number as a price using website currency settings (configured in Admin → Website → Config → Currency Formatting).

```liquid
{{ 1234567890.50 | currency }}
{{ 1234567890.50 | currency: format: '%u %n' }}
{{ 1234567891.50 | currency: precision: 0 }}
{{ 1234567890.50 | currency: unit: '€', separator: ',', delimiter: '' }}
```

**Parameters:** `format` (`%u` = unit, `%n` = number), `negative_format`, `unit`, plus all `number` filter parameters.

---

## design_tool_url

Returns the design tool (editor) URL path for a given `Project`.

```liquid
{{ project | design_tool_url }}
{{ project | design_tool_url: cart: 't' }}
```

---

## escape_json

Escapes a string for use inside JSON by replacing double quotes, newlines, etc. with escaped versions. Still requires wrapping in double quotes in JSON.

```liquid
var json = ["{{ my_string | escape_json }}"];
```

---

## page

Sets the current page of a `Paginate` object. Defaults to the `page` URL parameter, or `1` if absent.

```liquid
{% assign countries = website.countries | page: 1 %}
{% assign orders = user.orders | page: request.params['order-page'] %}
```

---

## page_size

Sets the page size of a `Paginate` object.

```liquid
{% assign lines = cart.orderlines | page_size: 1000 %}
```

---

## parse_json

Parses a JSON string into a Liquid object.

```liquid
{% assign json = json_string | parse_json %}
{{ json.a[1] }}
{{ json.b.c }}
```

---

## preview_url

Returns a preview URL path for a `Project`. Accepts optional keyword parameters added as query parameters.

```liquid
{{ project | preview_url }}
{{ project | preview_url: height: 250 }}
{{ project | preview_url: width: 100, ts: 123456 }}

{# For email use — requires share code: #}
{{ orderline.project | preview_url: height: 300, share: orderline.project.share_code }}
```

---

## push

Adds a single item to an array (similar to `concat` but for one item).

```liquid
{% assign colors = 'red green blue' | split: ' ' %}
{% assign colors = colors | push: 'pink' %}
```

---

## sort

Enhances Liquid's built-in `sort` to work with `Paginate` objects. Sorts efficiently in the database. Ascending by default; use `reverse` for descending.
```liquid
{% assign countries = website.countries | sort: 'code' %}
{% assign last_orders = user.orders | sort: 'created_at' %}
```

**Important limitation:** `sort` only works reliably when applied to a `Paginate` object (i.e. a collection returned directly from the platform, such as `website.custom_types[...]`, `user.orders`, etc.). It does **not** work reliably on plain arrays built via `push`. Nested dot-notation sort keys (e.g. `'custom.blog_date'`) fail silently on push-built arrays — the array is returned in unpredictable order with no error thrown.

**Rule:** Always apply `sort` at initial collection assignment. Never apply it to a push-built array.

---

## t

Translates a string using translations configured in the Pixfizz Admin.

```liquid
{{ 'Monday' | t }}
{{ 'Monday' | t: ns: 'phonetic' }}
{{ 'Monday' | t: locale: 'es' }}
```

---

## thumbnail_url

Returns a URL to a thumbnail version of an `Image` object. Defaults to 250px.

```liquid
<img src="{{ image | thumbnail_url | escape }}" />
<a href="{{ image | thumbnail_url: 1024 | escape }}">
```

---

## where

Enhances Liquid's built-in `where` to work with `Paginate` objects. Filters efficiently in the database.

```liquid
{% assign usa = website.countries | where: 'code', 'US' | first %}
{% assign confirmed_orders = user.orders | where: 'status', 'C' %}
```

Supports drill-down filtering: `'design.template.category', 'Canvases'`
Supports custom field filtering: `'custom.express_service', 'true'`

---

## Other standard filters (Pixfizz-extended)

| Filter | Description |
|---|---|
| `first` | Extended to work with `Paginate` objects |
| `last` | Extended to work with `Paginate` objects |
| `size` | Extended to work with `Paginate` objects |
| `map` | Extended to work with `Paginate` objects |
| `reverse` | Extended to work with `Paginate` objects (reverses sort order) |
| `convert_base` | Converts a number from one base to another |
| `hmac_sha1` | HMAC-SHA1 hash |
| `hmac_sha256` | HMAC-SHA256 hash |
| `md5` | MD5 hash |
| `sha1` | SHA1 hash |
| `sha256` | SHA256 hash |
| `number` | Formats a number with precision/separator/delimiter params |
| `pixfizz_asset_url` | Pixfizz internal asset URL variant |
| `json_parse` | Parses a JSON-formatted string into a Liquid object |

---

# TAGS

---

## snippet

Renders a CMS snippet by name. Additional keyword parameters become variables inside the snippet.

```liquid
{% snippet 'image-thumbnail', image_src: '/path/to/image.jpg', color: '#ffaabb' %}
{% snippet 'my-snippet', fallback_content: '' %}
```

**Note:** If the snippet is not found, an error is thrown unless `fallback_content` is provided.

**CMS import behaviour:** The CMS importer silently skips any snippet file that contains a Liquid syntax error. The snippet will simply not exist after import — no error is shown during the import process itself. This makes syntax errors in snippet files especially dangerous: they appear as mysterious "Snippet not found" errors at render time, even though the file was present in the tar.

---

## form

Wraps form submissions. The `form` object ([Form](/2f9228d5a61e4e9683f742facbcda109)) is available inside the block as `form`, giving access to `form.errors`, `form.values`, `form.captcha`, etc.

The tag renders the HTML `<form>` element with appropriate attributes. You are responsible for including control elements (`<input>`, `<textarea>`, `<select>`, ...) and buttons.

### Generic Parameters

All form types accept these optional parameters:

| Parameter | Description |
|---|---|
| `page` | CMS page name to redirect to after successful submission |
| `target` | Full URL path (with optional query string) to redirect to after success. Takes precedence over `page`. |
| `autosubmit` | When `true`, auto-submits on any contained element's `change` event |
| `async` | When `true`, submits asynchronously and updates HTML without page reload |
| `selectors` | CSS selector(s) for partial page update after async submission. Only use with `async: true`. |

Any other parameter is rendered as an HTML attribute on the generated `<form>` element.

### Form Types

**Contact & Auth**

| Form Type | Required Params | Description |
|---|---|---|
| `contact` | — | Contact form. Use `{{ form.captcha }}` inside. |
| `user_login` | — | User login form |
| `user_registration` | — | User registration form |
| `user_update` | — | User update form |
| `forgotten_password` | `reset_page:` or `reset_target:` | Forgotten password form. Points to the page containing the reset form. |
| `password_reset` | — | Password reset form |
| `locale_set` | — | Sets user session language. Include input with `name="locale"` and two-letter code value. |

**Cart**

| Form Type | Required Params | Description |
|---|---|---|
| `cart_add_product` | `product:` | Adds the given Product to the cart |
| `cart_set_shipping_option` | — | Sets the cart's shipping option |
| `cart_apply_promocode` | — | Applies a promocode to the cart |
| `cart_remove_promocode` | — | Removes the current promocode from the cart |
| `cart_update` | — | Updates the cart |
| `cart_payment` | *`gateway:`, *`onsuccess:`, *`oncancel:` | Cart payment form. Optional params are PayPal-only. |
| `cart_clear` | — | Removes all orderlines from the current cart. The cart stays assigned to the session; promocode, address and other cart properties are kept on the now-empty cart, and it remains in `user.carts`. |
| `cart_unset` | — | Removes the current cart from the session. The cart and its orderlines are NOT deleted and remain accessible under `user.carts`. |
| `cart_delete` | *`cart:` | Destroys a cart. With no `cart:` param it deletes the current session cart; pass an existing cart to delete that one instead. Deletion is asynchronous — the cart can take a moment to disappear from `user.carts`. |

**Cart teardown — clear vs unset vs delete**

These three forms are easy to confuse. Choose by what you want to keep:

- `cart_clear` — empties the cart (removes orderlines) but keeps the cart itself, including its promocode and address. Use when the shopper wants to start over but stay in the same cart.
- `cart_unset` — detaches the current cart from the session without deleting anything. A fresh cart is started on the next add-to-cart, and the old one is still listed in `user.carts`. Use for "save this cart for later / start a new one".
- `cart_delete` — permanently destroys a cart. Deletion runs asynchronously, so it may still appear in `user.carts` briefly after submission. On a "My carts" page, delete a specific saved cart by passing it in: `{% form 'cart_delete', cart: cart %}`.

**Writing cart custom fields**

`cart_update` is the form type that writes to `cart.custom`. Name the input
`cart[custom][<field_name>]`:

```liquid
{% form 'cart_update', async: true, autosubmit: true %}
	<input type="hidden" name="cart[custom][order_source]" value="{{ source }}">
{% endform %}
```

Three conditions apply:

- The field must already exist as a custom field on that site. Custom fields do
  not inherit parent to child.
- Where the value comes from a helper snippet capture, always `| strip` before
  comparing. Helper snippets render with a trailing newline; an unstripped
  comparison never matches, which turns a guarded auto-submit into a submit loop.
- Guard on cart state before firing. A hidden auto-submitting `cart_update`
  against an empty cart is wasted work at best; gate on
  `cart.orderlines.size > 0` and on the stored value being absent or stale.

**Projects**

| Form Type | Required Params | Description |
|---|---|---|
| `project_create` | `product:`, `design:`, *`parent_orderline:` | Creates a project. Pass `parent_orderline` for child orderlines. |
| `project_update` | `project:` | Updates a project's attributes |

**Addresses**

| Form Type | Required Params | Description |
|---|---|---|
| `address_create` | *`assign_to_user:`, *`assign_to_cart:` | Creates an address. By default the new address is set on the current cart **and** saved to the user's saved addresses. Pass `assign_to_user: false` and/or `assign_to_cart: false` to suppress either behaviour. |
| `address_update` | `address:` | Updates the given Address |
| `address_delete` | `address:` | Deletes the given Address |

**Orders & Orderlines**

| Form Type | Required Params | Description |
|---|---|---|
| `orderline_update` | `orderline:` | Updates the given Orderline |
| `orderline_commit` | `orderline:` | Commits a draft Orderline. If it's the last draft in the order and order is "Draft", order moves to "Confirmed". |
| `orderline_delete` | `orderline:` | Deletes the given Orderline |
| `order_payment` | `order:`, *`gateway:`, *`onsuccess:`, *`oncancel:` | Pays the given Order. Optional params are PayPal-only. |

**Galleries & Images**

| Form Type | Required Params | Description |
|---|---|---|
| `gallery_create` | — | Creates a new gallery |
| `gallery_update` | `gallery:` | Updates the given Gallery |
| `gallery_delete` | `gallery:` | Deletes the given Gallery |
| `image_create` | `gallery:` | Uploads an image to the given Gallery |
| `image_delete` | `image:` | Deletes the given Image |

**Calendars**

| Form Type | Required Params | Description |
|---|---|---|
| `calendar_create` | — | Creates a new calendar |
| `calendar_update` | `calendar:` | Updates the given Calendar |
| `calendar_delete` | `calendar:` | Deletes the given Calendar |
| `calendar_date_create` | `calendar:` | Creates a new date in the given Calendar |
| `calendar_date_update` | `calendar:`, `date:` | Updates a CalendarDate in the given Calendar |
| `calendar_date_delete` | `calendar:`, `date:` | Deletes a CalendarDate from the given Calendar |

**Other**

| Form Type | Required Params | Description |
|---|---|---|
| `signin_token_renew` | `signin_token:` | Renews a soft-expired SigninToken |

*Parameters marked with * are optional.*

---

## redirect_to

Redirects to another URL.

---

## return_404

Returns a 404 response.

---

## signin_token / promocode / braintree_client_token

Specialized tags for authentication tokens, promo codes, and payment setup.

---

# GLOBAL VARIABLES SUMMARY

| Variable | Available In | Description |
|---|---|---|
| `cart` | CMS | Current user's cart |
| `user` | CMS, email, SMS, fulfillment | Current user |
| `website` | All contexts | Website object |
| `request` | CMS only | Current HTTP request |
| `order` | Email, SMS, fulfillment | Order that triggered the template |
| `collection` | CMS (collection URLs) | Current collection |
| `product` | CMS (`?product=<id>`) | Current product |
| `design` | CMS (`?theme=<id>`) | Current design |
| `form` | Inside `{% form %}` blocks | Form errors and submitted values |
| `orderlines` | Fulfillment templates | Orderlines being fulfilled |

---

## Notes on Contexts

- **CMS** (pages, layouts, snippets): Full access to all objects and filters.
- **Email/SMS templates**: `order`, `user`, `website` available. No `request`, no `cart`.
- **Fulfillment templates**: `order`, `orderlines`, `user`, `website` available. `orderline.generated_files` only available here.
- Project previews in email **must include the share code**: `share: orderline.project.share_code`

---

# KNOWN CMS LIQUID QUIRKS

These behaviours differ from standard Liquid or Shopify Liquid. Confirmed through testing on the Pixfizz CMS.

| What doesn't work | Use instead |
|---|---|
| `!= blank` | `!= ''` or `.size > 0` |
| `== false` (explicit boolean comparison) | Use the truthy/falsy pattern: `{% unless var %}` |
| `nil == false` | `false` — nil and false are both falsy but not equal. Use `!= true` instead of `== false` |
| `== 'true'` on boolean Custom Type field | Use `{% if instance.custom.field_name %}` directly — boolean fields evaluate as truthy/falsy |
| `{% assign var = nil %}` | `{% assign var = '' %}` (then test with `!= ''`) |
| Checklist snippet capture without `strip` | Always pipe through `strip` after capture — snippet renders with trailing newline that breaks `== 'TRUE'` |
| `{% if product != blank %}` to detect a nil object | Test an attribute that always exists on a real one: `{% if product.id %}`. Nil yields nil, which is falsy in every engine. `!= blank` is not portable and can take the "it exists" branch for an object that is nil |
| `product.custom.x != blank` to test whether a field was set | `{% assign v = product.custom.x \| default: '' \| strip %}{% if v != '' %}`. Comparing against the empty string behaves identically everywhere; `!= blank` does not |

---

# RECENT PLATFORM ADDITIONS

These are Liquid-related capabilities added to the platform recently. They are not yet
documented on the master Notion reference and may still be evolving — confirm against
a live site before depending on specific syntax.

---

## Liquid scripts can set custom order fields

**Shipped, 2026-03-23.**

Liquid scripts can now write values into custom order fields. This allows a script to
compute something at order time (e.g. a production deadline, a routing key, a split
SKU) and persist it onto the order for downstream use — the order template, the
admin, exports, fulfillment transformations, etc.

**Use case seen in production:** a live client site — a Liquid script calculates the
production deadline from the product, shipping option, and current date, writes it
into a custom order field, and the value is then displayed on the order detail page
and used by the production workflow.

Treat this as a genuinely new Liquid capability: before this change, custom order
fields could only be set at add-to-cart time via form inputs or by the user at
checkout. They can now be written programmatically at any point a Liquid script
runs on an order.

Canonical snippet pending — capture from the reference site once stable.

---

## Product file-name Liquid templates

**Shipped, 2026-03-31.**

The product file-naming Liquid template (used when generating production file names
for a product) now supports:

- **Capitalization control** — force upper/lower/title case on substituted values.
- **Dynamic dash insertion** — insert separators between variable pieces without
  hard-coding them in the template literal.

Used on client sites for file naming that needs to match a specific
fulfillment partner's naming convention. Exact syntax to be captured from the TK
site and added here once confirmed.

---

## Configurable product display names via Liquid

**Shipped, 2026-03-03.**

Product display names can now be configured via Liquid, so the title shown in
product lists and on the product page can be generated dynamically rather than
stored as a static string.

**Primary use case:** dynamic inventory messaging. Combined with the new inventory
tracking feature, a product title can append `"Only 3 left"` or `"Out of stock"`
based on current stock level, without duplicating the product or maintaining
parallel copies.

Pair with the inventory tracking feature (see `18_ADMIN_NAVIGATION.md`) for the
complete pattern.

---

## Meta title inheritance — per-product disable checkbox

**Shipped, 2026-01-26.**

A new checkbox exists on the product admin to **disable design meta title inheritance**
for that product. Use this when a template's meta title is unintentionally overwriting
a product's SEO-facing title.

When hit: the symptom is a product whose `<title>` in the rendered head comes from
the associated design template rather than from the product's own SEO title. Tick
the checkbox to pin the product's own title.

---

## UTM capture snippet

**Shipped, 2026-03-24** (initial client deployment).

A reusable Liquid snippet that captures incoming UTM parameters (`utm_source`,
`utm_medium`, `utm_campaign`, etc.) into user state so they persist through the
session and can be pushed into analytics, order custom fields, or Meta Pixel /
GA4 events.

Use this whenever a site needs marketing source attribution that survives the
user navigating away from the landing URL. Final canonical version still
stabilizing — pull from the reference deployment when adding to a new site.

---

## Meta Pixel + GA4 — debug mode gotcha

**Recurring gotcha (2026-03-24).**

When setting up Meta Pixel alongside GA4 on a Shopper site, **remember to
disable GA4 debug mode** before going live. GA4 debug mode (enabled via the
GTM Preview or a `?gtm_debug=1` parameter) filters events out of the
production property, so launched sites can appear to be capturing zero
conversions even though everything else is wired correctly.

Checklist when adding analytics to a Shopper site:
- Meta Pixel base code injected via Custom JS or a head snippet.
- GA4 tag firing on page view, add-to-cart, purchase.
- `debug_mode: false` (or the debug parameter stripped) on production.
- Test a real transaction end-to-end and confirm events land in both the
  Meta Events Manager and the GA4 DebugView → then Realtime report.

For server-side GA4 tracking via webhook, see `18_ADMIN_NAVIGATION.md`
under Settings > Webhooks.

---

## Blog chronological sort — gotcha

Custom blog / listing pages built on top of a CMS collection **do not sort
chronologically by default**. This has been hit on multiple sites (most recently
a client site, 2026-03-12) and is an easy thing to miss because the admin list in
the CMS *is* sorted by date, so the issue only surfaces on the public page.

**Fix options:**
- Set an explicit sort order on the collection in the CMS admin, or
- Apply a Liquid `sort` / `reverse` filter on the listing loop (`sorted_by: 'created_at'`
  / reverse as appropriate).

Always verify the public page order after building a blog — do not trust that
"date-descending" is the default.

---

## Save-and-exit in add-to-cart for limited-option products

**Pattern, not a new feature.**

For products with a small number of customisable options (e.g. name-only cards,
single-photo prints), the "Save & Exit" action from the design tool can be wired
directly into the add-to-cart flow so the customer does not have to re-enter the
design tool to proceed. Used on a client site, 2026-03-12.

This is a Shopper UX pattern, not a platform capability — implemented at the
snippet level on the product page.

---

## Checkout column capture-block refactor

**Canonical deduplication pattern for Shopper checkout (2026-01-26).**

Matjaz's refactor of the checkout-page columns removed duplicated markup between
the left and right columns by wrapping the shared regions in `{% capture %}` blocks
and rendering the captured content in both places.

This is the recommended way to deduplicate repeated block regions in checkout
templates. See `50_SHOPPER_TEMPLATE_REFERENCE.md` for the detailed pattern and
example markup.

## Changelog
- 2026-06-01: Noted Shopify IDs live in chosen_variants. Source: claude-chat.
- 2026-06-15: Added json_parse filter to Pixfizz-extended filters. Added assign_to_user / assign_to_cart optional params to the address_create form. Source: notion-dashboard.
- 2026-07-07: Documented cart_clear, cart_unset and cart_delete forms in the Cart forms table, plus a clear/unset/delete comparison note. Source: notion-page, slack-message.
- 2026-07-28: Added file_upload accessor note on ChosenOption (`uploaded_file.url` / `.filename`; `value`, `asset.url`, `thumbnail_url` and the `preview_url` filter do not work), the `thumbnail/{n}` path segment on UploadedFile, and the `cart[custom][field]` write pattern for `cart_update`. Source: claude-chat.
