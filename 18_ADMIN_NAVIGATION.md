# 18 — Admin Navigation

**Authority Scope:** Pixfizz Core admin interface sections and settings only.

_Last updated: 2026-04-10_

---

## Admin Overview

The Pixfizz Core admin is the central interface for managing your store, products, orders, and settings. Accessed at `{your-domain}/admin`.

---

## Admin Sidebar Sections

### Dashboard
Key business metrics: Gross Revenue, Orders Fulfilled, Average Order Value, Sessions, Conversion Rate. Quick links to OrderHub, community, help center.

### Orders
- **Orders** — view/manage with status filters, CSV export, barcode search
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
- **Product Attributes** — commercial product definitions
- **Templates** — production specifications. Includes a bulk **Text Upgrade** action (shipped 2026-03-05) that applies text-box vertical alignment fixes across all templates in one step — use when migrating older templates that pre-date the current text rendering.
- **Collections** — product groupings for storefront
- **Fonts** / **Font Palettes** / **Color Palettes** — typography and color management
  - **Gotcha:** the default font palette tooltip implies fonts are auto-assigned, but fonts must be **manually assigned** to a palette. If a design shows fallback typography, check the palette assignment rather than assuming the admin auto-populated it.
- **Element Substitutions** — swap rules
- **Calendars** — calendar product configuration
- **Inventory tracking** (available at the product level) — dynamic stock messaging based on thresholds, with the ability to block orders below a configured stock level. Pair with configurable product display names via Liquid (`50_LIQUID_REFERENCE.md`) to show messages like "Only 3 left" directly in the product title.

### Website (CMS)
Manages storefront content. What's visible depends on Shopper vs standalone CMS:

- **Pages** — CMS pages forming storefront URL structure (standalone CMS only — Shopper manages these automatically)
- **Layouts** — wrapper templates that pages render inside (standalone CMS only)
- **Snippets** — HTML/Liquid template fragments (building blocks of pages)
- **Custom Types** — dynamic content types for flexible page content
- **Assets** — file manager for images, fonts, media
- **Crawler** — website crawl management for sitemaps and product feeds. Accessed at `{domain}/admin/website_crawls`. Each crawl run lists every URL the crawler followed, with response status (200, 404, etc.) and the page that contained the link (shown in the rightmost column). Use this to track down broken internal links — the linking page column tells you exactly which snippet or page to fix.
    - **Sitemap:** generated at `{domain}/sitemap.xml`. Product feed at `{domain}/product-feed.json`. Both are platform-level features, not Shopper-specific.
    - **Gotcha:** the sitemap is **not** at `/site/sitemap`. Do not invent this URL. Confirm against the live admin.
    - **Gotcha (2026-02-02):** product XML feed URLs have shipped with an explicit `:80` port in the URL, making them unreachable for some consumers (Google Merchant Center rejected ~915 URLs on one site). If you see products missing from a feed with no obvious cause, inspect the raw XML for `:80` in the URLs before looking elsewhere.

RATIONALE: Admin path, 404 diagnostic behavior, and correct sitemap URL were all missing.
SOURCE: "Tracking down pixfizz crawler 404 errors" + "Creating a sitemap" chats, April 22

> On Shopper sites, Pages and Layouts are pre-configured. You primarily work with Snippets, Custom Types, and Assets.

### Settings
- **General** — website title, timezone, language, currency, domain hosting, currency formatting
- **Email Notifications** — email templates for order lifecycle events (14 templates)
- **Design Tool** — Design Tool Configurations (branding, features, defaults)
- **Translations** — language/translation management. Major upgrade shipped 2026-03-30:
  - Built-in translation support for **products and templates** (previously only pages and snippets).
  - **Automatic translation key detection** during import.
  - **Per-template and per-product-variant** translation handling.
  - **Export / import** flow for bulk translation editing.
- **Webhooks** — webhook configuration for external integrations (e.g. order.created, order.status_changed)
  - **Analytics pattern — server-side GA4 via webhook (Nicolas Restrepo, 2026-03-26):** client-side GA4 typically captures only ~10% of conversion events due to ad blockers, cookie consent drop-off, and tracking prevention. Piping `order.created` / `order.confirmed` through a Pixfizz webhook to a server-side GA4 endpoint (via a middleware like GTM Server, Stape, or a simple Cloud Function) lifts capture to ~50%+. Recommend this for any site where attribution accuracy matters for paid media decisions.

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