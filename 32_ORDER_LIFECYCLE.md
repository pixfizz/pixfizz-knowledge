# 32 — Order Lifecycle & Production Pipeline

**Authority Scope:** OrderHub order lifecycle, production pipeline, fulfillment destinations, and OrderHub Desktop.

_Last updated: 2026-05-19_

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

### Pending orders do not auto-route to production

An order sitting in **Pending** does not flow to production on its own. Nothing downstream fires — no artwork generation, no fulfillment delivery, no OrderHub job — until the order is moved to **Confirmed**.

Where an order lands as Pending (manual payment methods, gateway not configured, payment awaiting capture), someone must **manually confirm it in admin** before production sees it. This is the most common cause of "the order is in Pixfizz but nothing reached the lab".

Check the Pending queue as part of daily order review on any site that accepts manual or offline payment.

### Gateway callback webhooks must be configured on the gateway side

Configuring a payment gateway in Pixfizz admin is only half the setup. Most
gateways will not tell Pixfizz the outcome of a payment unless a callback
webhook is registered in the gateway's own dashboard, pointing back at the
Pixfizz callback endpoint for that site.

Where that webhook is missing, the customer pays successfully and is debited,
but Pixfizz never receives the result. The order stays in **Pending** and never
routes to production. Nothing errors on the storefront, so this presents as a
Pixfizz fault when it is a gateway configuration gap.

**Diagnosing.** Search the server logs for requests to the site's gateway
callback endpoint. No requests at all means the webhook is either not set up,
or set up against the wrong URL. Test payments that debit correctly but leave
the order Pending are the same signal.

**PayU (confirmed).** In the PayU admin under **Developers > Webhooks**, create
two webhooks, one for **Payments > Successful** and one for **Payments >
Failed**. Both point at the same URL:

```
https://<site-domain>/cart/payu_money_callback
```

Treat "is the gateway-side webhook registered" as the first check on any
gateway where payments succeed but orders stay Pending, before looking at
Pixfizz configuration.

### Status codes in Liquid

In Liquid templates, `order.status` returns a **single-letter code**, not the display label. Confirmed codes:
- `P` = Pending
- `F` = Payment Failed

Write conditionals against the letter, e.g. `{% if order.status == 'P' %}`, and fetch the newest pending order with `user.orders | where: 'status', 'P' | sort: 'created_at' | reverse | first`.

**Pay Now / payment retry.** Orders in Pending (`P`) or Payment Failed (`F`) can be re-paid using the `order_payment` form: `{% form 'order_payment', order: order %}`. This is used to surface a "Pay Now" action on the account orders page and on the empty-cart page for back-from-gateway recovery. (Form tag confirmed in use on a live site; verify the tag name against the current template before reusing.)

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

## Custom Order & Orderline Fields via Liquid Scripts

Pixfizz supports configurable Liquid scripts (set in the Super Admin) that automatically populate custom fields on orders and orderlines at the moment an order is created from a cart. This removes the need for manual data entry on orders.

### What scripts can do

- Set **custom fields on the order** (order-level script)
- Set **custom fields on individual orderlines** (orderline-level script)

### Script execution order

When an order is created, scripts run in this fixed sequence:

1. Auto-confirm script
2. Priority script
3. **Order-level custom field script**
4. **Orderline-level custom field script**

Custom field scripts run **after core processing**, so all standard order data (pricing, user, address, etc.) is available to the script at execution time.

### Script return format

Each script must return a **JSON dictionary**:

```json
{
  "field_name": "value",
  "another_field": "value"
}
```

Each key is a custom field name. Each value is what gets saved.

### Merge behavior

- The values generated by the script are **merged** with any existing custom fields on the order or orderline.
- Existing fields are preserved unless the script explicitly overwrites them with a new value for the same key.

### Use cases

- Automatically tagging orders with metadata (e.g. source channel, campaign)
- Storing calculated values at order time (e.g. margin, weight estimate)
- Passing data to external systems or fulfillment workflows
- Adding per-item attributes to orderlines (e.g. processing instructions, routing codes)

### Who configures this

This is an **advanced / developer feature**. Scripts are configured in the **Super Admin** — not in the per-site admin. It requires familiarity with Liquid and JSON structure. Contact Pixfizz support if unsure.

---

## Order Cancellation and Transaction Fees

When cancelling orders, marking an order as "Canceled" in the admin may not automatically prevent Pixfizz transaction fees from being applied.

### Cancellation process

1. **Collect order codes** — gather the order codes for all orders that need to be cancelled.
2. **Email finance@pixfizz.com** — send the list of order codes to the Pixfizz finance team. This ensures the cancellations are processed correctly and fees are waived.
3. **Mark as canceled in admin** — update the order status in your admin. This is for your own record-keeping, but alone it does not guarantee fee removal.
4. **Follow up** — confirm with Pixfizz that the cancellations have been processed and no charges will apply.
5. **Check for duplicates** — verify that cancelled orders are not also duplicated (e.g. from a customer re-ordering after a failed attempt). Duplicate charges can occur if both the original and replacement order remain active.

> Simply marking an order as cancelled in admin is **not sufficient** to prevent transaction fees. You must also notify finance@pixfizz.com.

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

Desktop application for photo lab operators. Runs locally, polls OrderHub for new jobs, downloads print files, and feeds them to local print controllers.

### Key Capabilities
- Continuous polling for new jobs flagged for local production
- Downloads job details and print files from OrderHub via API
- Organises files into structured folders with human-readable naming
- Generates **DPOF files** for compatible print controllers (Epson, Noritsu, etc.)
- **AI-powered upscaling** for low-resolution images
- Channel and product routing to match jobs to the correct print workflow
- Job review tools: colour correction, quantity management
- Updates job status back in OrderHub once processed

### OHD Workflow
1. OrderHub creates a production job (on order confirmation + artwork generation)
2. Assets delivered to fulfillment destination (FTP/HTTP)
3. Job appears in OHD queue
4. OHD polls, downloads job and production assets to local machine
5. Team processes job; OHD updates status (Downloaded → Manufactured)
6. Status change flows back to OrderHub, triggers next lifecycle step

### Multi-instance Behaviour
If multiple OHD instances run across workstations, job delivery is **first-come-first-served** — a job goes to only one instance. Instances can be filtered by location.

### API Status Update Endpoint
```
POST /functions/v1/update-job-status
Headers: X-API-Key: <api_key>
```

> OHD is a companion tool to OrderHub — not standalone. Requires active OrderHub connection. For full operational detail see `45_ORDERHUB.md`.

---

## Automatic Discounts

Pixfizz supports **automatic discounts** applied as negative line values on orders. Key behavior:

- Discounts appear as **negative values on order lines** (not as a separate discount field)
- Automatic discounts **stack with promo codes** — both can be applied to the same order
- Applied at the checkout page
- Configured in admin — no customer action required to apply them

This differs from promocode-based discounts, which require the customer to enter a code. Automatic discounts trigger without any input.

---

## Pending Order Email — Scope to Manual Payment Orders

When sending reminder emails for orders stuck in `pending` status, the trigger must be scoped to orders where the `manual_payment` custom field equals `true`.

Reason: without this condition, the email would also fire for failed credit card attempts — where the customer has likely already given up or re-ordered. A "please pay" email arriving a week after a failed card attempt creates a confusing experience.

Before sending:
1. Check `order_custom_manual_payment == true`
2. Check the customer has not placed a newer confirmed order — if they have, skip the reminder

The `manual_payment` field is a custom field set at order creation. It returns `true` for orders placed using manual payment methods (e.g. bank transfer, pay-in-store). Confirmed in a production implementation (April 2026).

---

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-23: Added pending order email scoping rule for manual_payment field.
- 2026-05-19: Added Custom Order & Orderline Fields via Liquid Scripts section (from Notion KB). Added Order Cancellation and Transaction Fees process (from Notion KB).
- 2026-05-21: Added Automatic Discounts (negative order line values, stack with promo codes, applied at checkout). Source: Fireflies.
- 2026-05-21: Expanded OrderHub Desktop (OHD) section with DPOF generation, AI upscaling, multi-instance behaviour, and API endpoint. Added cross-reference to 45_ORDERHUB.md. Source: OrderHub help modal.
- 2026-06-26: Documented order.status single-letter Liquid codes (P=Pending, F=Payment Failed) and the order_payment form for Pay Now / payment retry. Source: claude-chat.
- 2026-07-25: Clarified that Pending orders do not automatically route to production and must be manually confirmed in admin before artwork generation, fulfillment delivery, or OrderHub job creation occurs. Source: fireflies-call.
- 2026-08-05: Documented gateway callback webhooks as a required gateway-side configuration step, covering the failure mode where payment succeeds but the order stays Pending. Added the confirmed PayU setup (two webhooks, Successful and Failed, both to `/cart/payu_money_callback`) and the log-based diagnosis. Source: slack-message (#development), fireflies-call.
