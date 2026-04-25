# 14 — Glossary

**Authority Scope:** Definitions only.

_Last updated: 2026-03-30_

---

# Glossary (Pixfizz)

## Platform vs template
- **Pixfizz CMS (platform):** objects, pricing engine, identity model, admin + custom fields.
- **Shopper (template):** an opinionated storefront implementation on top of Pixfizz CMS.
- **Custom storefront:** a site using Pixfizz CMS but not the Shopper theme behavior.

## User states
- **Anonymous user:** session-based identity stored in browser; owns cart/projects while browsing.
- **Guest user:** persisted user record flagged as guest; cannot login/reset password; owns orders/projects; mergeable into registered.
- **Registered user:** persisted user record with credentials.
- **Logged-in:** a registered user with an authenticated session.
- **Admin / Super Admin:** permission roles on a user.

## Print products
- **Photo Prints UI:** special workflow where per-photo quantities are set outside cart.
- **cut_print_quantity:** derived count used for pricing for photo print orderlines.

## Delivery
- **Digital-only:** static product variant `version` == `digital-only` triggers checkout-only behavior (ignore shipping, hidden public address).

## Pricing
- **Pricing formula:** Ruby expression returning a numeric price.
- **Price Variable:** admin-defined numeric constant available by name (e.g., `hardcover_base_a4`, `whitelabel`).


## Fulfillment
- **Fulfillment Template:** A template that outputs a job ticket payload (JSON/XML/TXT) and references generated artwork files for an external lab/system.
- **Job ticket:** The payload sent to a provider describing what to produce; often accompanies artwork files.
- **Generated files:** Output files produced for an orderline (often per page/layer), accessible via `orderline.generated_files`.
- **Filename Template:** Admin-configured pattern used to name (and optionally folder-route) generated files; commonly uses `order.code`, `orderline.barcode`, `page_output_name`, `layer_output_name`, `idx`, and `format`.
- **Fulfillment Destination:** Where production assets are delivered. Configured in Super Admin. Supports FTP/SFTP and HTTP delivery.
- **Fulfillment Transformation:** Rules controlling how artwork is processed for production (color profile, format, bleed). Exist at Template level and Design level (Design overrides Template).
- **Split by orderline:** Setting that fans out one order into multiple fulfillment requests (one per orderline).
- **Hotfolder:** A watched folder/shared drive where jobs are delivered by dropping files/manifests.
- **Adapter:** A mapping from the Pixfizz Default Fulfillment JSON to a vendor-specific payload.

## Platform architecture
- **Personalization Layer:** The core Pixfizz layer — product configuration, design tools, image workflows, XML templates. Used in every deployment.
- **Commerce Layer:** Storefront, checkout, catalog, payments, shipping. Only active in Full Pixfizz deployments.
- **Workflow Layer:** Order management, artwork generation, asset delivery, OrderHub. Active in all deployments.
- **Website Layer:** Storefront building tools (Shopper, custom frontend, theming, domain/SSL). Only in Full Pixfizz.

## Products
- **Pixfizz Core:** The personalization engine — product configuration, templates, designs, design tool, and admin interface.
- **Product Attribute:** The commercial side of a product (pricing, variants, packaging). Links to a Template (design product) or stands alone (static product).
- **Template:** Production specification for a product (dimensions, DPI, XML definition, page range, fulfillment rules).
- **Design:** Customer starting point under a Template (pages, layouts, backgrounds, clipart, masks).
- **Collection:** Grouping mechanism for publishing products to the storefront. URL: `/shop/:collection/:product/:design`.
- **Design product:** A Product Attribute linked to a Template. Customer personalizes before purchase.
- **Static product:** A Product Attribute without a Template link. Standard eCommerce item (frames, gift vouchers, etc.).
- **Variant:** Customer-facing option on a Product Attribute (size, finish, material) that affects pricing.
- **Option:** Production-level choice on a Template or Design (not customer-facing pricing).
- **Design Tool Configuration:** Controls design tool appearance and features for a specific context. Templates reference a configuration.

## Order lifecycle
- **OrderHub:** The Workflow Layer of Pixfizz. Manages order routing, artwork generation, asset delivery, production jobs, fulfillment tracking.
- **OrderHub Desktop (OHD):** Desktop application for production teams to download, view, and manage production jobs.
- **Order statuses:** Pending → Draft → Confirmed → Downloaded → Manufactured → Shipped → Fulfilled. Exceptions: Payment Failed, Error, Canceled, Refunded.
- **Orderline:** An individual product within an order. Each has generated files, options, fulfillment code.

## Account management
- **myPixfizz:** Account management hub — subscriptions, onboarding projects, team access, billing.
- **Pixfizz Select:** Marketplace for discovering Pixfizz-powered services and partners.
- **Pixfizz API:** Integration layer for connecting personalization and fulfillment into external platforms.
- **Super Admin:** Cross-website management layer for organizations managing multiple Pixfizz websites.
