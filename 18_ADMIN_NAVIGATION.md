# 18 — Admin Navigation

**Authority Scope:** Pixfizz Core admin interface sections and settings only.

_Last updated: 2026-06-01_

---

## Admin Overview

The Pixfizz Core admin is the central interface for managing your store, products, orders, and settings. Accessed at `{your-domain}/admin`.

---

## Admin Sidebar Sections

### Dashboard
Key business metrics: Gross Revenue, Orders Fulfilled, Average Order Value, Sessions, Conversion Rate. Quick links to OrderHub, community, help center.

The dashboard also surfaces active (in-progress) carts, giving visibility into carts in flight for account management and follow-up. Exact location and any configuration to be confirmed.

### Orders
- **Orders** — view/manage with status filters, CSV export, barcode search
    - **Design custom fields in the orderline CSV:** to output a design-level custom field as a column in the orderline CSV export, reference it with the nested path `custom:print_book:print_theme:<field-name>` (for example `custom:print_book:print_theme:primary_collection_path`). The standard orderline CSV cannot filter on these fields, but it can output them using this path.
- **Abandoned Carts** — incomplete checkouts
- **Production Files** — production book files with project, page count, status
- **Projects** — end users' saved personalization projects

### Galleries
Public galleries where customers share and view photo projects.

### Users
Customer accounts and access management.

### Shipping
- **Shipping Services** — carrier and method configuration
- **Packaging** — package type definitions
- **Taxes** — tax rules
- **Addresses** — address management
- **Extra Fees** — additional fee configuration

### Marketing
- **Promocodes** — promotional code management

### Products
- **Published Products** — published products list (everything in a Collection). Renamed from "All Products" on 2026-03-30 to make it clear the listing is scoped to published items only.
- **Product Attributes** — commercial product definitions. Prices are now **editable inline** directly from the product attributes list — click on any price field, type the new value (simple price or formula), press Enter for simple prices or click OK for multi-line formulas, Esc to cancel. No need to open each product individually.
- **Templates** — production specifications. Includes a bulk **Text Upgrade** action (shipped 2026-03-05) that applies text-box vertical alignment fixes across all templates in one step — use when migrating older templates that pre-date the current text rendering.
- **Collections** — product groupings for storefront
- **Fonts** / **Font Palettes** / **Color Palettes** — typography and color management
  - **Gotcha:** the default font palette tooltip implies fonts are auto-assigned, but fonts must be **manually assigned** to a palette. If a design shows fallback typography, check the palette assignment rather than assuming the admin auto-populated it.
- **Element Substitutions** — swap rules
- **Calendars** — calendar product configuration
- **Inventory tracking** — see the dedicated Inventory Management section below.

### Website (CMS)
Manages storefront content. What's visible depends on Shopper vs standalone CMS:

- **Pages** — CMS pages forming storefront URL structure (standalone CMS only — Shopper manages these automatically)
    - **Admin-only visibility:** individual pages and blog posts can be set to admin-only, so they are visible to logged-in admins but hidden from the public. Use this to stage and review content before its public release, then switch it on to publish. This is a publish gate, distinct from `d-none`, which only hides an element visually while leaving its links crawlable.
- **Layouts** — wrapper templates that pages render inside (standalone CMS only)
- **Snippets** — HTML/Liquid template fragments (building blocks of pages)
- **Custom Types** — dynamic content types for flexible page content
- **Assets** — file manager for images, fonts, media
- **Crawler** — website crawl management for sitemaps and product feeds. Accessed at `{domain}/admin/website_crawls`. Each crawl run lists every URL the crawler followed, with response status (200, 404, etc.) and the page that contained the link (shown in the rightmost column). Use this to track down broken internal links — the linking page column tells you exactly which snippet or page to fix.
    - **Sitemap:** generated at `{domain}/sitemap.xml`. Product feed at `{domain}/product-feed.json`. Both are platform-level features, not Shopper-specific.
    - **Gotcha:** the sitemap is **not** at `/site/sitemap`. Do not invent this URL. Confirm against the live admin.
    - **Gotcha (2026-02-02):** product XML feed URLs have shipped with an explicit `:80` port in the URL, making them unreachable for some consumers (Google Merchant Center rejected ~915 URLs on one site). If you see products missing from a feed with no obvious cause, inspect the raw XML for `:80` in the URLs before looking elsewhere.

> On Shopper sites, Pages and Layouts are pre-configured. You primarily work with Snippets, Custom Types, and Assets.

### Settings
- **General** — website title, timezone, language, currency, domain hosting, currency formatting
- **Email Notifications** — email templates for order lifecycle events (14 templates)
- **Design Tool** — Design Tool Configurations (branding, features, defaults)
- **Translations** — see the dedicated Translation Support section below.
- **Webhooks** — webhook configuration for external integrations (e.g. order.created, order.status_changed)
  - **Analytics pattern — server-side GA4 via webhook (Nicolas Restrepo, 2026-03-26):** client-side GA4 typically captures only ~10% of conversion events due to ad blockers, cookie consent drop-off, and tracking prevention. Piping `order.created` / `order.confirmed` through a Pixfizz webhook to a server-side GA4 endpoint (via a middleware like GTM Server, Stape, or a simple Cloud Function) lifts capture to ~50%+. Recommend this for any site where attribution accuracy matters for paid media decisions.

---

## Inventory Management

Pixfizz supports per-product inventory tracking, allowing you to monitor available stock and automatically reduce quantities when orders are placed.

### Enabling inventory tracking

Inventory is managed at the **product attribute level** — it is not active by default. To enable:
1. Open the Product Attribute in admin.
2. Enable the inventory tracking toggle.
3. Set the current inventory count (how many units are currently available).

### How stock is reduced

When an order includes a product that tracks inventory, the system automatically subtracts the purchased quantity **the first time** the order enters one of these statuses:
- **Confirmed (C)**
- **Draft (W)**

Stock is only reduced once — the first time the order hits either status. Subsequent status changes do not re-deduct.

### Negative stock and out-of-stock behavior

Inventory tracking **does not automatically block purchases**. If stock reaches 0, customers can still place an order — the inventory count will go negative. This gives operators flexibility but means you may want to add storefront controls.

### Liquid properties for storefront logic

Two Liquid properties are available on products for CMS-level stock management:

- `product.tracks_inventory` — boolean, true if inventory tracking is enabled for this product
- `product.current_inventory` — integer, the current stock count (can be negative)

Use these to:
- Display stock levels ("Only 3 left")
- Show "Out of stock" messaging
- Disable or hide the Add to Cart button when inventory reaches 0

Example logic pattern:
```
{% if product.tracks_inventory and product.current_inventory <= 0 %}
  <!-- show out of stock message, hide add-to-cart -->
{% endif %}
```

### Dynamic stock messaging

Pair inventory tracking with configurable product display names via Liquid (`50_LIQUID_REFERENCE.md`) to show threshold-based messages like "Only 3 left" directly in the product title or product card. The ability to block orders below a configured stock level is also available.

### Who is this for

Inventory management is designed for store owners who track physical stock or limited-quantity products — limited edition items, seasonal products, pre-produced goods, or physical items with fixed inventory.

---

## Built-in Translation Support

Pixfizz includes built-in translation support for core platform objects. This allows multi-language stores to manage translations directly in the admin and display the correct content automatically based on the user's language.

### Enabling multi-language support

1. Go to the **Super Admin**.
2. Enable **Multi-language support**.
3. Select the languages you want to support — to select multiple, hold **Command (Mac)** or **Ctrl (Windows)** while clicking.
4. Save.

Once enabled, translation options become available across supported objects.

### What can be translated

The following core objects support translation:

- **Products** — names and descriptions
- **Designs** — names, tags (layouts, backgrounds, clipart)
- **Collections** — names and descriptions
- **Templates** — names, page captions
- **Variant types and values** — the commercial options customers see on the storefront
- **Template option types and values** — production-level options

This covers both the storefront (CMS) and the design tool — customers see localized content end to end.

### Where to find translations in admin

When multi-language support is active, a **"Translate"** link appears in the top-left corner of supported objects. Clicking it opens the translation page where you can manage content for each enabled language.

### How translations are applied

Translations are automatically resolved based on the user's current language. In Liquid templates, standard object properties return the translated value automatically. For example:

```
{{ design.name }}
```

This displays the design name in the current language — no conditional logic needed.

### Bulk translation management

A major upgrade shipped 2026-03-30 added:
- Built-in translation support for **products and templates** (previously only pages and snippets were translatable).
- **Automatic translation key detection** during import.
- **Per-template and per-product-variant** translation handling.
- **Export / import** flow for bulk translation editing — export all translation keys to a file, translate externally, and re-import.

### What this is useful for

- Offer a fully localized experience across storefront and editor
- Manage translations in one place (Pixfizz admin) rather than external systems
- Ensure consistency across all touchpoints (product pages, design tool, options, captions)

---

## Super Admin

Cross-website management layer (for organizations managing multiple Pixfizz websites).

### Customer Super User Access
- **Websites** — manage all Pixfizz websites, configuration, feature flags, integrations
- **Super Users** — admin access across organization
- **Orders** — cross-website order aggregation and search
- **Fulfillment** — configure fulfillment destinations (FTP and HTTP endpoints)

### Website Management (Super Admin)
Per-website configuration includes:
- Core settings (name, subdomain, theme, language, currency)
- Feature flags enabling/disabling platform capabilities
- External integrations (Shopify, Etsy, Google OAuth, Dropbox, HappyAR, ReCAPTCHA)
- Website inheritance — share Design, Products, Tax, Email configurations across websites

> Some Super Admin features are managed by Pixfizz staff. The customer-facing view is deliberately focused.

---

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-10: Added Published Products rename, Text Upgrade bulk action, inventory tracking with dynamic stock messaging, translation export/import upgrade, font palette tooltip gotcha, server-side GA4 via webhook pattern.
- 2026-04-22: Expanded Crawler entry with admin path, 404 reporting behaviour, and sitemap URL gotcha.
- 2026-05-19: Expanded Inventory Management into dedicated section with enable flow, stock reduction rules, negative stock behavior, Liquid properties (product.tracks_inventory, product.current_inventory), and out-of-stock CMS pattern. Expanded Translation Support into dedicated section with Super Admin enable flow, translatable objects list, Liquid auto-resolution, translate link location, and bulk export/import. Added inline price editing to Product Attributes. Source: Notion KB articles.
- 2026-06-01: Added active carts on the dashboard. Source: fireflies-call.
- 2026-06-15: Documented admin-only visibility for Pages/blog (pre-publish staging gate) and the design custom-field column path for the orderline CSV export (custom:print_book:print_theme:<field>). Source: slack-kb-sync (Matjaz, #development; design-field reporting work).
