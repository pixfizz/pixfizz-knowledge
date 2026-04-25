# 15 — Platform Architecture

**Authority Scope:** Platform-level architecture layers and deployment models only.

_Last updated: 2026-03-30_

---

## Four Platform Layers

Pixfizz is organized around four capability layers that work together:

### 1. Personalization Layer
The core of every Pixfizz deployment. Provides the tools customers use to configure and customize products before purchase.
- Product configuration interfaces (size, format, options)
- Personalization tools (image upload, text placement, design editing)
- Image upload workflows with asset management
- Template-driven product definitions using XML

Every deployment uses this layer — whether the storefront runs on Shopify, the native Pixfizz CMS, or a custom frontend via API.

### 2. Commerce Layer
Handles storefront and checkout when Pixfizz is the full commerce platform.
- Storefront CMS with page management
- Product catalog and category management
- Checkout and payment processing
- Shipping and tax configuration
- Customer accounts and order history

> Only active in Full Pixfizz deployments. In Shopify + Pixfizz (or custom API), the external platform handles this layer.

### 3. Workflow Layer
Manages everything after order placement — the journey from order to production-ready output.
- Order ingestion (from Pixfizz checkout, Shopify webhooks, or external platforms via API)
- Artwork generation from personalized designs
- Asset delivery via FTP/SFTP or HTTP
- OrderHub job creation, production routing, and production download

### 4. Website Layer
Tools for building and managing storefront experiences on the Pixfizz platform.
- Shopper — a managed, customizable website template
- Custom frontend option for businesses with own dev resources
- Theme selection and brand identity configuration
- Domain and SSL configuration
- Site navigation and homepage layout management

> Only relevant in Full Pixfizz deployments. Shopify or external platforms manage their own storefronts.

---

## How the Layers Connect

**Full Pixfizz deployment:** All four layers active and tightly integrated. Customer browses storefront (Website) → personalizes product (Personalization) → completes checkout (Commerce) → order flows into production (Workflow).

**Integrated deployment (Shopify / custom API / marketplace):** External platform handles Commerce and Website layers. Pixfizz provides Personalization and Workflow layers.

---

## Deployment Models

### Shopify + Pixfizz
- Shopify manages: storefront, checkout, commerce
- Pixfizz manages: personalization, product configuration, order management
- Orders originate in Shopify → ingested via webhooks → production pipeline via OrderHub
- Best for: existing Shopify stores adding advanced personalization

### Full Pixfizz
- Pixfizz manages: storefront (Shopper or custom), checkout, personalization, order management, production workflows
- All four platform layers active
- Best for: new businesses, replacing aging platforms, tight storefront-to-production integration

### Custom API Integration
- External platform (WooCommerce, Magento, custom-built) handles storefront/checkout
- Pixfizz provides personalization and fulfillment via API
- Requires development resources to manage integration
- Best for: non-Shopify eCommerce platforms, custom frontends

### Marketplace Integration
- Marketplace (e.g. Etsy) handles storefront/checkout
- Pixfizz manages personalization and production
- Marketplace orders flow into same production pipeline as direct sales
- Best for: sellers on Etsy or similar marketplaces

### Side-by-Side Comparison

|  | Shopify + Pixfizz | Full Pixfizz | Custom API | Marketplace |
| --- | --- | --- | --- | --- |
| Storefront | Shopify | Pixfizz (Shopper or custom) | Your platform | Marketplace |
| Checkout | Shopify | Pixfizz | Your platform | Marketplace |
| Product catalog | Shopify | Pixfizz | Your platform | Marketplace |
| Personalization | Pixfizz | Pixfizz | Pixfizz | Pixfizz |
| Order management | Pixfizz | Pixfizz | Pixfizz | Pixfizz |
| Production workflows | Pixfizz | Pixfizz | Pixfizz | Pixfizz |

> All deployment options include full access to Personalization and Workflow layers. The choice comes down to storefront and checkout ownership.

---

## Key Platform Products

- **Pixfizz Core** — The personalization engine: product configuration, templates, designs, design tool, and admin.
- **OrderHub** — The Workflow Layer: order management, production routing, fulfillment. Includes OrderHub Desktop (OHD) for production teams.
- **Shopper** — Managed website template within the Pixfizz eCommerce CMS. Not a standalone product.
- **myPixfizz** — Account management hub: subscriptions, onboarding projects, team access, billing.
- **Pixfizz Select** — Marketplace for discovering Pixfizz-powered services and partners.
- **Pixfizz API** — Integration layer for connecting personalization and fulfillment into any external platform.

---

## Changelog
- 2026-03-30: Created from master platform documentation export.
