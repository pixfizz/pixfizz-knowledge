# 60 — Shopify Integration

**Authority Scope:** Shopify + Pixfizz integration architecture, snippets, metafields, cart page, and order sync.

_Last updated: 2026-04-10_

---

## 1. Architecture Overview

In the Shopify deployment path:

- **Shopify** handles the storefront, cart, checkout, and payment.
- **Pixfizz** handles personalization (editor, options, photo prints), project storage, and production/order orchestration.
- Pixfizz runs on a **subdomain of the Shopify store's domain** (e.g. `create.myprintshop.com`). It must be a custom hostname — not the default `*.pixfizz.com` subdomain.
- A JavaScript API (`api.js`) is loaded from the Pixfizz host into every Shopify page. It provides the `Pixfizz.Shopify.*` methods used by all integration snippets.
- Customer identity is passed from Shopify to Pixfizz via an MD5 hash of the customer ID and email, signed with a shared secret.
- When a Shopify order is paid, a webhook fires to Pixfizz, confirming the order. Pixfizz changes the order status to "Confirmed".
- When an order is fulfilled in Shopify, a second webhook can fire to mark the Pixfizz order as "Shipped".

---

## 2. Shopify Metafields

### Product Metafields (`pixfizz.*` namespace)

| Name | Namespace + Key | Type | Notes |
|---|---|---|---|
| Pixfizz Integration Type | `pixfizz.integration_type` | Single line text | One of: `editor`, `options-to-editor`, `options-to-cart`, `photo-prints`. Defaults to `editor` if blank. |
| Pixfizz Product SKU | `pixfizz.product_sku` | Single line text | `theme-code:product-code` from Pixfizz. Used for products without Shopify variations. |
| Pixfizz Extra Page Addon | `pixfizz.page_addon_product` | Product Variant | Links to the variant used to charge for extra pages (photo books). Price is per single page. |
| Pixfizz Option Addons | `pixfizz.option_addon_products` | Product | Products added to cart when a linked Pixfizz option is selected. |
| Pixfizz Option Type Code | `pixfizz.option_type_code` | Single line text | On addon products — links the addon to a Pixfizz option type code. |
| Pixfizz Adjust Cart QTY | `pixfizz.adjust_cart_qty` | True or False | Set to `false` to prevent customers changing cart quantity. Used for photo prints. |
| Pixfizz Photo Prints Collection Path | `pixfizz.photo_prints_collection_path` | Single line text | Collection path used when launching the photo prints flow. |
| SEO Hidden | `seo.hidden` | Integer | Set to `1` to hide addon products from search/Google. |

### Variant Metafields (`pixfizz.*` namespace)

| Name | Namespace + Key | Type | Notes |
|---|---|---|---|
| Pixfizz Product SKU | `pixfizz.product_sku` | Single line text | Used for products WITH Shopify variations. Takes precedence over product-level SKU. |
| Pixfizz Extra Page Addon | `pixfizz.page_addon_product` | Product Variant | Variant-level override for the extra page addon. |
| Pixfizz Option Value Code | `pixfizz.option_value_code` | Single line text | Links this Shopify variant to a specific Pixfizz option value code. |
| Pixfizz Option Addons | `pixfizz.option_addon_products` | Product | Variant-level option addon override. |

### SKU Precedence Rule

For products WITH Shopify variations: the SKU **must** be set on the variant metafield. The product-level SKU is the fallback only.

---

## 3. Integration Types

Set via `pixfizz.integration_type` product metafield.

| Value | Behaviour |
|---|---|
| `editor` | Launches the Pixfizz editor directly. Default. Used for photo books and products where full customization is expected. |
| `options-to-editor` | Customer selects options first (e.g. calendar start month), then enters the editor. |
| `options-to-cart` | Launches a live preview modal. Customer selects options and sees a preview before adding to cart. No full editor. Used for notebooks and simpler products. |
| `photo-prints` | Launches the photo prints flow. Quantity is set inside Pixfizz, not in the Shopify cart. |

---

## 4. Cart Item Properties (Pixfizz-managed)

Pixfizz sets these line item properties on Shopify cart items. They drive all cart-page behaviour.

| Property | Meaning |
|---|---|
| `_pixfizz_project_id` | ID of the associated Pixfizz project. Present on all Pixfizz personalized items. Private (underscore prefix = hidden from customer display). |
| `_pixfizz_addon` | Present on addon orderlines. Value equals the `_pixfizz_project_id` of the parent orderline. Used to link addons to their parent and to suppress their independent display. |
| `_pixfizz_unit_quantity` | The per-unit quantity of an addon. Used by the quantity handler to scale addon quantities correctly when the parent quantity changes. |

Properties with underscore prefix are not displayed to customers in Shopify's default rendering. The cart template already filters these out via `property_first_char != '_'`.

---

## 5. Core Shopify Snippets

Five snippets must be created in the Shopify theme. All are referenced by `{% render %}` calls in the product template and cart page.

---

### `pixfizz-setup.liquid`

**Purpose:** Loads `api.js` from the Pixfizz host and initialises the integration with customer identity.

**Rendered in:** `theme.liquid` `<head>`, via `{% render 'pixfizz-setup' %}`.

**Key configuration:**
- `pixfizz_host`: the Pixfizz custom subdomain (e.g. `create.myprintshop.com`)
- `shared_secret`: from Pixfizz superadmin API Settings

**How identity works:** If a Shopify customer is logged in, their ID and email are MD5-hashed twice (with the shared secret) and passed to `Pixfizz.Shopify.setup()`. If no customer, `null` is passed and Pixfizz uses an anonymous session.

---

### `pixfizz-launch-product-handler.liquid`

**Purpose:** Returns an inline JS handler (as escaped string) used in the product page "Personalize" button `onclick`.

**Usage in product template:**
```liquid
<button onclick="{% render 'pixfizz-launch-product-handler' %}">Personalize</button>
```

**What it does:**
- Builds a map of Shopify variant IDs → Pixfizz SKUs
- Builds an addon map (page addons + option addons) per variant
- Reads the selected variant from the product form
- Calls `Pixfizz.Shopify.launchProduct(sku, integration_type, product_id, variant_id, parameters)`

**Note:** This snippet may need adjustment for themes other than Dawn, specifically the `form = document.querySelector('form[action*="/cart/add"]')` selector.

---

### `pixfizz-orderline-preview-handler.liquid`

**Purpose:** Replaces the static product image in the cart with a live Pixfizz project preview, using a `<style onload>` pattern so it fires on DOM injection.

**Usage in cart template** (placed immediately after the `<img>` element inside `cart-item__image-container`):
```liquid
{% render 'pixfizz-orderline-preview-handler', item: item, width: 300 %}
```

**Parameters:**
- `item`: the cart line item object
- `width` or `height`: desired preview dimension (use one or both)

**How it works:** Uses `<style onload>` so the preview replacement fires every time the element is injected into the DOM (including after AJAX cart updates). Only acts if `item.properties._pixfizz_project_id` is present — safe to include unconditionally for all items.

---

### `pixfizz-edit-orderline-handler.liquid`

**Purpose:** Returns an inline JS handler used on the cart item image link `onclick`, allowing the customer to re-enter the Pixfizz editor from the cart.

**Usage in cart template** (on the `<a>` element wrapping the cart item image):
```liquid
onclick="{% render 'pixfizz-edit-orderline-handler', item: item %}"
```

**Only rendered when** `item.properties._pixfizz_project_id` is present.

**What it does:**
- For `photo-prints` integration type: calls `Pixfizz.Shopify.launchPhotoPrints(parameters)`
- For all other types: calls `Pixfizz.Shopify.launchProduct(...)` — re-opens the same editor flow with the existing project

---

### `pixfizz-orderline-quantity-handler.liquid`

**Purpose:** Intercepts quantity changes and item deletions in the cart, ensuring addon orderlines are updated or removed in sync with their parent.

**Usage in cart template:**

On the quantity `<input>`:
```liquid
onchange="{% render 'pixfizz-orderline-quantity-handler', item: item, cart: cart %}"
onkeydown="if(event.keyCode===13)event.preventDefault()"
```

On the remove `<a>` button:
```liquid
onclick="{% render 'pixfizz-orderline-quantity-handler', item: item, cart: cart, delete_item: true %}"
```

**What it does:**
- When quantity changes: finds all addon items whose `_pixfizz_addon` matches this item's `_pixfizz_project_id`, and updates their quantities using `_pixfizz_unit_quantity * new_quantity`
- Uses `event.stopImmediatePropagation()` + `__pixfizz_addons_handled` flag to prevent double-firing
- When deleting: updates all quantities to 0 then reloads the page (native handler not reliable for deletions)
- Uses `fetch('/cart/update.js')` to apply all quantity changes in a single request

**Critical:** This handler must be present on every part of the UI that can modify or delete orderlines — not just the main cart page. In the drawer cart, on quick-buy, etc., the same handlers need to be applied.

---

## 6. Product Page Template

For Pixfizz products, the standard Shopify buy button block is replaced with a "Personalize" button.

**Recommended approach (Shopify 2.0 themes):**
1. Create a new product template called "Pixfizz Product" based on the default product template
2. Remove the buy button block
3. Add a Custom Liquid block with the personalize button

**Button code:**
```liquid
<button onclick="{% render 'pixfizz-launch-product-handler' %}" class="button button--full-width">
  Personalize
</button>
```

The button text can be changed freely. The `onclick` handler is what matters.

---

## 7. Cart Page — Working Version (Dawn, inline_asset_content)

This is the production-tested cart page. It is based on Dawn theme using the `inline_asset_content` filter for SVG icons (a pattern introduced in later Dawn versions). It differs from the v13/v11 examples in the docs.

### Key differences from earlier documented Dawn versions

- SVG icons use `'icon-*.svg' | inline_asset_content` instead of `{% render 'icon-*' %}`
- Quantity lock uses `item.product.metafields.pixfizz.adjust_cart_qty.value == false` (metafield boolean check) instead of product type string comparison
- Edit handler does not have the `options-to-cart` exclusion present in v11 docs — simpler: any item with `_pixfizz_project_id` gets the handler
- Info button uses `small-hide medium-hide` classes

### Pixfizz modifications summary (what was added and where)

**1. Skip addon items at the top of the loop**
```liquid
{%- if item.properties._pixfizz_addon -%}
  {% continue %}
{%- endif %}
```
Addon items are rendered separately as sub-rows under their parent. They must not get a full independent row.

**2. Edit handler on the image link**
```liquid
style="z-index:1;"
{%- if item.properties._pixfizz_project_id -%}
  onclick="{% render 'pixfizz-edit-orderline-handler', item: item %}"
{%- endif -%}
```
`z-index:1` is required because the image container sits above the link in stacking order; without it the click doesn't reach the `<a>`.

**3. Project preview replacement**
```liquid
{% render 'pixfizz-orderline-preview-handler', item: item, width: 300 %}
```
Placed immediately after the `<img>` tag, inside `cart-item__image-container`.

**4. Quantity lock via metafield**
```liquid
{%- if item.product.metafields.pixfizz.adjust_cart_qty.value == false -%}
  <span>{{ item.quantity }}</span>
{%- else -%}
  <quantity-input ...>...</quantity-input>
{%- endif -%}
```
When `adjust_cart_qty` is `false`, renders a plain quantity display instead of the input controls. Used for photo prints.

**5. Quantity change handler on input**
```liquid
onchange="{% render 'pixfizz-orderline-quantity-handler', item: item, cart: cart %}"
onkeydown="if(event.keyCode===13)event.preventDefault()"
```

**6. Delete handler on remove button**
```liquid
onclick="{% render 'pixfizz-orderline-quantity-handler', item: item, cart: cart, delete_item: true %}"
```

**7. Addon sub-rows after each main item row**
After the closing `</tr>` of the main item, a second loop renders addon items as separate `<tr>` rows:
```liquid
{%- if item.properties._pixfizz_project_id -%}
  {%- for addon_item in cart.items -%}
    {%- if addon_item.properties._pixfizz_addon == item.properties._pixfizz_project_id -%}
      <tr class="cart-item">
        <td class="cart-item__media"></td>
        <td class="cart-item__details">
          {{ addon_item.product.title | escape }}
          {{ addon_item.final_price | money }}
          ...variants...
        </td>
        <td>{{ addon_item.final_line_price | money }}</td>
        <td>{{ addon_item.quantity }}</td>
        <td>{{ addon_item.final_line_price | money }}</td>
      </tr>
    {%- endif -%}
  {%- endfor -%}
{%- endif -%}
```
Addon rows: no media cell, no quantity controls, no remove button — display only.

---

## 8. Addon Products — Setup and Behaviour

### Extra Pages (photo books)

- Create a separate Shopify product representing "extra page" — price is per single page
- Link it to the main book product via `pixfizz.page_addon_product` metafield (Product Variant type)
- Pixfizz adds this product to the cart automatically when the user adds pages in the editor
- Pages are always added in pairs (or multiples) as set by the template; Shopify price is per page
- Hide addon product from search using `seo.hidden = 1`
- Recommended: disable pricing display in the Pixfizz editor for these products to avoid confusion

### Option Addons (variant-linked pricing)

- Create an addon product in Shopify. Set `pixfizz.option_type_code` on it to the code of the Pixfizz option type
- Create variants on the addon product. Set `pixfizz.option_value_code` on each variant to the corresponding Pixfizz option value code
- Link the addon product to the base product via `pixfizz.option_addon_products`
- Pixfizz adds the correct addon variant to the cart when the customer selects the linked option

---

## 9. Order Sync — Webhooks

### Payment webhook (required)
- **Event:** Order payment
- **Format:** JSON
- **URL:** `https://<pixfizz-host>/webhooks/shopify/orders`
- **Effect:** Pixfizz marks the order as "Confirmed"

### Fulfillment webhook (optional)
- **Event:** Fulfillment Creation
- **Format:** JSON
- **URL:** Same endpoint pattern
- **Effect:** Pixfizz marks the order as "Shipped"

### Webhook signing secret
After creating the webhook in Shopify (Settings → Notifications), copy the signing secret shown under the callback URL and paste it into the Pixfizz superadmin panel under Website → API Settings → Shopify Signing Secret.

---

## 10. Product Linking Rules

### Products WITHOUT Shopify variations
Required metafields on the product:
- `pixfizz.integration_type`
- `pixfizz.product_sku` (on the product, not variant)

### Products WITH Shopify variations
Required metafields:
- `pixfizz.integration_type` (on the product)
- `pixfizz.product_sku` (on **each variant**, not the product)

### General rules
- Shopify product must have "Track QTY" disabled
- For photo prints: set `pixfizz.adjust_cart_qty` to `false` on the product
- SKU format: `theme-code:product-code` (no spaces, no typos)

---

## 11. Troubleshooting Guide

### Project preview not showing in cart
- Check that `pixfizz-orderline-preview-handler` is placed immediately after the `<img>` element inside `cart-item__image-container`
- Check that `item.properties._pixfizz_project_id` is actually present (inspect cart item properties)
- The `<style onload>` pattern fires on DOM injection — if the preview worked once but stopped after a cart AJAX update, check that the snippet is not inside a `<script>` tag

### Edit link not firing on image click
- The `<a>` element needs `style="z-index:1;"` — without it, the image container sits above the link in the stacking order and swallows the click
- Confirm `_pixfizz_project_id` is present on the item

### Addon quantities not updating when parent quantity changes
- `pixfizz-orderline-quantity-handler` must be on the quantity `<input>` onchange
- If the theme has a drawer cart or other quantity controls elsewhere, the handler must be added there too — not just on the main cart page
- Check `_pixfizz_unit_quantity` is set on the addon item (Pixfizz sets this when adding to cart)

### Quantity selector showing for photo prints when it should be hidden
- `pixfizz.adjust_cart_qty` metafield must be set to `false` (boolean False, not the string "false")
- The check in the template is `.value == false` — if the metafield is missing or is the string "false", it will not match

### Addon items appearing as independent rows
- The `{%- if item.properties._pixfizz_addon -%} {% continue %} {%- endif %}` block must be at the very top of the `for item in cart.items` loop, before the `<tr>` is opened
- If this block is missing or misplaced, addon items render as full rows

### Orders not confirming in Pixfizz after payment
- Check the Shopify webhook is set to "Order payment" event (not "Order creation")
- Verify the Shopify signing secret matches what's in Pixfizz superadmin
- Check that the webhook URL matches the Pixfizz host exactly

### "Personalize" button text overwritten with "Add to cart" after variant change
- Cause: Dawn's Section Rendering API re-renders the product section on variant change and overwrites the `innerHTML` of any element with class `product-form__add-button` — including the Pixfizz personalize button — replacing it with the "Add to cart" translation string.
- **Recommended fix**: change the button's class away from `product-form__add-button` to a custom class (e.g. `pixfizz-launch-btn`) so Dawn's variant update logic does not target it. Add a small script to hide the native submit button if needed.
- **Simpler fix**: wrap the button text in a structured inner `<span>` rather than a plain text node. Dawn targets plain text nodes specifically; a structured span may survive the replacement depending on the Dawn version.
- This affects any Dawn theme using the Section Rendering API — confirm which fix is appropriate by fetching a section render response and inspecting the returned button HTML.

### `Pixfizz is not defined` JS error
- `pixfizz-setup` snippet must be rendered in `theme.liquid` before `</head>`
- The Pixfizz host in `pixfizz-setup.liquid` must use the custom subdomain, not `*.pixfizz.com`

---

## 12. Retrieval Pointer

For questions about:
- Cart item properties and addon row rendering → sections 5, 7 of this file
- Metafield setup → section 2
- Integration types → section 3
- Product linking → section 10
- Webhook/order sync → section 9
- Troubleshooting → section 11
- Pixfizz platform objects (order, orderline) → `50_LIQUID_REFERENCE.md`
- Order external source/reference fields → `50_LIQUID_REFERENCE.md` (Order object)
- Local pickup address mapping → section 13
- Variable pages (saved projects, galleries, Pixfizz.Shopify methods) → section 15

---

## 13. Pickup Orders — Webhook Address Handling

When a Shopify order is placed using a local pickup option, Shopify sends no shipping address in the order webhook. Pixfizz handles this by attempting to match the **shipping method name** from the webhook to a **public address** in Pixfizz.

### Matching logic

- Pixfizz reads the shipping method name from the Shopify webhook
- It searches for a public address in Pixfizz where the **Company** field matches that name exactly (case-sensitive)
- If a match is found, that address is assigned to the order automatically
- If no match is found, the order arrives in Pixfizz with no address and must be assigned manually

### Setup

1. In Shopify admin → **Settings → Locations**, note the exact **Name** of the pickup location
2. In Pixfizz admin → **Settings → Addresses**, create or edit a public address and set the **Company** field to exactly that name
3. Test with a pickup order and confirm the address is assigned in Pixfizz

### Notes

- The match is case-sensitive — `CameraMall` and `cameramall` are treated as different values
- If you have multiple pickup locations, create a separate public address in Pixfizz for each one
- Cart-page snippets are unaffected — this is purely webhook ingestion behaviour

---

## 14. Multi-Site Product Inheritance (CameraMall Pattern)

**Pattern, 2026-03-05 (CameraMall Review).**

CameraMall operates multiple brand storefronts (CameraMall / Woodward / Arts
Cameras) that share a **single underlying product database** across all brands,
while each brand has its own Shopify storefront + Pixfizz personalization
configuration.

### How it works

- Products, templates, and fulfillment logic are defined once at the parent
  organization level in Pixfizz.
- Each brand's Shopify store references the same Pixfizz products via the
  usual `pixfizz.product_sku` metafield, but the Shopify-level surfaces
  (titles, prices, collection structure, theme) are independently managed
  per brand.
- Pixfizz Super Admin's **website inheritance** (see `18_ADMIN_NAVIGATION.md`)
  is how the shared product/design/tax/email configurations are exposed to
  each brand site.

### When to reach for this pattern

- A customer operates multiple consumer-facing brands that sell the same
  underlying photo products with different branding and pricing.
- Each brand needs its own storefront identity and checkout, but the
  production pipeline is centralised.
- The alternative — copying products into each site separately — creates
  unmanageable drift over time.

### Gotchas

- Price differences between brands need to be handled at the Shopify layer
  (different prices per brand on the same underlying Pixfizz product), not
  in the Pixfizz pricing formula.
- Brand-specific production overrides (e.g. different fulfillment destinations
  per brand) need per-site fulfillment configuration, set at the Pixfizz
  website level rather than on the product itself.

---

## 15. Variable Pages — Saved Projects and Galleries

Shopify stores can host customer-facing pages that display Pixfizz data (saved projects, galleries) by calling the Pixfizz REST API directly from JavaScript. This works because `api.js` establishes a Pixfizz session for the customer on every page load, and `fetch()` calls with `credentials: 'include'` carry that session to the Pixfizz subdomain.

No additional authentication is required beyond the standard `pixfizz-setup.liquid` integration.

### Confirmed `Pixfizz.Shopify.*` method list (as of 2026-04-12)

```
pixfizz_origin         — the Pixfizz host URL (use this instead of hardcoding the subdomain)
locale
cart_target
skip_draft_redirect
product_data_loader    — loads and caches Shopify product data from the pixfizz-product-api page
_user                  — object: { uid, email } for the logged-in customer. null if not logged in.
_hash
_modal
_session_established
setup
establishSession
openModal
closeModal
launchProduct
launchPhotoPrints
launchProject
launchSavedProject
loadProjectPreview
replaceImageWithProjectPreview
getSavedProjects
```

### Key values for variable pages

- `Pixfizz.Shopify.pixfizz_origin` — use this as the base URL for all API calls. Never hardcode the subdomain.
- `Pixfizz.Shopify._user.uid` — the Pixfizz user ID for the logged-in customer. Required for user-scoped POST endpoints (e.g. create gallery).

### Gallery API endpoints (authenticated via session cookie)

```
GET  {origin}/v1/galleries/_mine.json                  — list customer's galleries
POST {origin}/v1/users/{uid}/galleries.json            — create a gallery (FormData: gallery[name])
GET  {origin}/v1/galleries/{id}/images.json            — list images in a gallery
POST {origin}/v1/galleries/{id}/images.json            — upload an image (FormData: data = file)
```

All calls require `credentials: 'include'` (fetch) or `xhr.withCredentials = true` (XHR).

### Image thumbnail URLs

Gallery image `preview` URLs contain a path segment `thumbnail/250` where `250` is the rendered size. This is a path parameter, not a query parameter — `?height=N` has no effect. To get different sizes, replace the number in the path:

```javascript
url.replace(/thumbnail\/\d+/, 'thumbnail/800')   // grid thumbnails
url.replace(/thumbnail\/\d+/, 'thumbnail/1600')  // lightbox
```

### Page template pattern

Variable pages are created as Shopify page templates (`page.{name}.liquid`) — not snippets, and not sections. The `{% schema %}` tag is not valid in page templates. Padding must be handled with plain CSS, not `section.settings`.

### Implementation reference

- Saved projects page: `pixfizz-saved-projects.liquid` snippet, also requires the `page.pixfizz-product-api.liquid` template and a corresponding Shopify page
- Gallery page: `page.pixfizz-galleries.liquid` — standalone template, no additional dependencies beyond `pixfizz-setup`

---

## Changelog
- 2026-03-13: Initial version. Compiled from public docs + working cart page (Dawn, inline_asset_content variant).
- 2026-03-21: Added Dawn button innerHTML overwrite troubleshooting entry (§11).
- 2026-04-10: Added pickup order webhook address handling (§13) and multi-site product inheritance pattern (§14).
- 2026-04-12: Updated §13 with specific pickup matching logic (Shopify location Name → Pixfizz address Company field). Added §15 Variable Pages — confirmed Pixfizz.Shopify method list, gallery API endpoints, thumbnail URL path pattern, page template conventions.

