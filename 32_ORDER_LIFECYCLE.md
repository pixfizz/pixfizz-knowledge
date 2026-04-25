# 32 — Order Lifecycle & Production Pipeline

**Authority Scope:** OrderHub order lifecycle, production pipeline, fulfillment destinations, and OrderHub Desktop.

_Last updated: 2026-04-23_

---

## What is OrderHub?

OrderHub is the Workflow Layer of Pixfizz. It manages everything after a customer places an order — routing orders to production, generating artwork, delivering production assets, and tracking fulfillment.

Handles orders regardless of origin: online storefront, Shopify, kiosk, in-store counter, or external platform via API.

---

## Order Lifecycle Statuses

Every order moves through defined statuses:

1. **Pending** — order received, awaiting confirmation
2. **Draft** — order saved but not yet finalized
3. **Confirmed** — order confirmed, ready for production
4. **Downloaded** — production assets downloaded
5. **Manufactured** — production complete
6. **Shipped** — order dispatched to customer
7. **Fulfilled** — order delivered and complete

Exception statuses: **Payment Failed**, **Error**, **Canceled**, **Refunded**.

Each status transition can trigger an email notification (configured in admin: Settings > Email Notifications).

---

## Order Origination

Orders can enter Pixfizz through multiple paths:
- **Pixfizz storefront** — orders placed on a Full Pixfizz site
- **Shopify** — ingested via webhooks
- **External platforms** — submitted via Pixfizz API
- **Marketplaces** — from connected marketplaces (e.g. Etsy)
- **Kiosk / In-store** — orders at physical retail locations

All orders follow the same lifecycle and production pipeline regardless of origin.

---

## Production Pipeline

Once an order is confirmed, it enters the production pipeline:

1. **Order confirmation** — order finalized, payment captured
2. **Artwork generation** — Pixfizz renders personalized design into production-ready artwork (based on Template specs: DPI, dimensions, format)
3. **Fulfillment transformation** — transformations defined on Template or Design are applied (color profile conversion, bleed adjustments, etc.)
4. **Asset delivery** — production files delivered to configured fulfillment destination via FTP or HTTP
5. **Job creation** — OrderHub job created with all production details
6. **Production download** — production team downloads via OrderHub Desktop
7. **Status updates** — order progresses: Downloaded → Manufactured → Shipped → Fulfilled

---

## Fulfillment Destinations

Define where production assets are delivered. Configured in Super Admin.

### FTP Delivery
- FTP Host, User, Password
- FTP Directory
- Filename Template (using Liquid variables from order/orderline context)
- Directory Template (folder structure on FTP server)

### HTTP Delivery
- WebService URL
- Content Type (e.g. application/json)
- Parameter Name and Fixed Parameters

### Common Output Settings (both delivery types)
- Format (PDF, JPEG, etc.)
- Color Profile (sRGB, CMYK, etc.)
- Single Page Output — each page as separate file
- Multiple Cut Print Copies — generate copies for cut-print workflows

### Filename Template
Uses Liquid-style variables to dynamically name production files. Example:
`{{ orderline.product.code }}.{{ order.code }}.{{ orderline.barcode }}_{{ page_output_name }}.{{ format }}`

---

## Order Management in Admin

### Orders List
- Order Code (PREFIX-ALPHANUMERIC pattern)
- Customer email, date, total, orderline count, status
- Filter by status, search by code/barcode, CSV export

### Order Detail
- Order ID, Code, Status, Date, Total, Paid status
- Shipping Service
- User Notes and internal Notes
- Priority flag
- Customer info (email, name, telephone, address)
- Job Ticket link
- Order History (status change log)
- Force-Refulfill option

### Orderlines
Each order has one or more orderlines (individual products). Each shows:
- Generated Files (downloadable production assets)
- Product options selected by customer
- Custom Fields
- Fulfillment Code (determines which destination receives assets)

### Other Admin Order Sections
- **Abandoned Carts** — incomplete checkouts
- **Production Files** — production book files with project, page count, status
- **Projects** — saved personalization projects
- **Cross-Website Order Management** — Super Admin aggregates orders from all websites

---

## OrderHub Desktop (OHD)

Desktop application for production teams to download, view, and manage production jobs.

### Key Features
- Download production jobs with all associated assets
- View job details including production instructions
- Track job status locally
- Manage production queue
- Barcode-based job lookup

### OHD Workflow
1. OrderHub creates a production job (on order confirmation + artwork generation)
2. Assets delivered to fulfillment destination (FTP/HTTP)
3. Job appears in OHD queue
4. Team downloads job and production assets
5. After production, team updates status (Downloaded → Manufactured)
6. Status change flows back to OrderHub, triggers next lifecycle step

> OHD is a companion tool to OrderHub — not standalone. Requires active OrderHub connection.

---

## Changelog
- 2026-03-30: Created from master platform documentation export.

---

## Pending Order Email — Scope to Manual Payment Orders

When sending reminder emails for orders stuck in `pending` status, the trigger must be scoped to orders where the `manual_payment` custom field equals `true`.

Reason: without this condition, the email would also fire for failed credit card attempts — where the customer has likely already given up or re-ordered. A "please pay" email arriving a week after a failed card attempt creates a confusing experience.

Before sending:
1. Check `order_custom_manual_payment == true`
2. Check the customer has not placed a newer confirmed order — if they have, skip the reminder

The `manual_payment` field is a custom field set at order creation. It returns `true` for orders placed using manual payment methods (e.g. bank transfer, pay-in-store). Confirmed by Rapid Studio implementation (April 2026).

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-23: Added pending order email scoping rule for manual_payment field.
