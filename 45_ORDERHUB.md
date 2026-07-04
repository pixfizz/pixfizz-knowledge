# 45 — OrderHub Reference

**Authority Scope:** OrderHub operational configuration, Jobs, Production Board, Processes, Locations, integrations, and notifications. For the core Pixfizz order lifecycle see `32_ORDER_LIFECYCLE.md`.

_Last updated: 2026-06-30_

---

## What is OrderHub?

OrderHub is the workflow management layer of Pixfizz. It operates as a separate web application at `orderhub.pixfizz.com` and sits downstream of the Pixfizz CMS. It handles everything after an order is placed: routing jobs to production, managing job statuses, printing tickets, coordinating fulfillment, and notifying customers.

OrderHub accepts orders from multiple sources:
- Pixfizz storefronts (via webhook)
- Shopify
- Square and Lightspeed POS systems
- Manual creation in the OrderHub UI

---

## Jobs

Jobs are the individual line items within an order. Each job represents one product/service to be produced.

### Job Statuses

Standard status flow: **New → In Production → Completed**

- **New** — job received, not yet started
- **In Production** — work underway
- **Completed** — production finished

### Custom Statuses

Each Process can define up to **2 custom statuses**. Custom statuses act as sub-states of **New** — they slot into the workflow between New and In Production. They are useful for multi-step pre-production stages (e.g. "Awaiting Materials", "Sorted").

### Automated Order Status Cascade

When all jobs in an order reach **Completed**, OrderHub automatically updates the parent order status. This cascade eliminates manual order-level status management.

### Job Detail Fields

Each job record includes:
- Product name and thumbnail (photo print jobs only — see Job Thumbnails below)
- Assigned Process
- Notes
- Due date
- Assigned person
- Quantity
- Customer-selected options
- Custom fields

### Job Thumbnails

Thumbnails are generated via a Pixfizz Webhook and appear in the Jobs interface and on PDF tickets.

- **Photo print products** — thumbnail generated automatically
- **Static products** — no thumbnail generated

---

## Production Board

Kanban-style interface for visualising and managing all active jobs.

### Views

**Timeline view** — columns represent due dates. Drag a job card horizontally to change its due date.

**Status view** — columns represent job statuses (New, In Production, Completed, plus any custom statuses). Drag a job card horizontally to change its status.

Both views support filtering by location and date range.

### Drag-and-Drop Behaviour

Dragging a card updates the corresponding field instantly — no confirmation dialog. Changes sync to the order record in real time.

---

## Processes

Processes define production workflows. Each job is assigned to a Process when it arrives in OrderHub.

### Process Configuration

Each Process has:
- **Name** and **colour** (for Production Board display)
- **Default print size** and **copies**
- **OHD flag** — whether jobs in this Process are pushed to the OrderHub Downloader desktop app
- **Location overrides** — per-location settings that differ from the Process default

### Category–Process Linking

Each category (product type) is linked to a Process. The link includes:
- **Lead time** (days before production starts)
- **Production days** (days to complete production)

Processes are configured in **Settings** within the OrderHub app.

---

### Variant Value Routing (OrderHub Desktop)

OrderHub Desktop maps an orderline to the correct output by reading the **variant/finish value** (for example `lustre`, `glossy`) together with the size — not a lab- or printer-specific numeric finish code.

- Numeric finish codes (e.g. `222`, `202`) are specific to a given lab and printer. The same finish on a different printer or site can carry a different number, so numeric codes do not map cleanly in Desktop and make routing harder to maintain.
- For products fulfilled via OrderHub Desktop, prefer human-readable finish/variant codes (`lustre`, `glossy`, etc.) used consistently across all sizes. Desktop's own routing setup then translates those readable values to the correct printer/queue.

## Locations

Locations represent physical sites or branches of the organisation.

### Location Configuration

Each Location has:
- Name and address
- **Payment terminals** — one or more terminals per location, supporting Stripe, Helcim, and Gravity
- **Printer mappings** — logical Named Printer roles mapped to physical PrintNode-connected printers
- **Website link** — associates the location with a Pixfizz website (used for branding in notifications)
- **Opening hours** — shown to customers when they choose a pickup location at checkout
- **Google Maps link** — a map / directions link surfaced alongside the pickup address at checkout

Locations are managed in **Settings → Locations** within OrderHub.

---

## PDF Layout Studio

Visual drag-and-drop editor for designing printed production documents.

### Supported Document Types

- **Production Job tickets** — per-job work orders for the production floor
- **Order Summaries** — full order rundowns
- **Shipping Labels** — address labels for outbound shipments
- **Packing Slips** — pick-and-pack documents
- **QC Checklists** — quality control forms

### Key Features

- Dynamic variables pulled from order and job data (e.g. order number, customer name, product options)
- AI layout editor for rapid template generation
- Auto-print via PrintNode on trigger events
- S3 archiving of generated PDFs

Accessed via **Settings → PDF Designer** in OrderHub.

---

## PrintNode Integration

PrintNode is the bridge between OrderHub's PDF Layout Studio and physical printers on the lab floor.

### Architecture

Two-layer system:
1. **Named Printers** — logical roles defined in OrderHub (e.g. "Front Desk Printer", "Production Printer"). These are stable identifiers used in PDF Layout Studio auto-print rules.
2. **PrintNode connection** — each Named Printer is mapped to a physical printer connected via the PrintNode desktop agent on a lab computer.

### Setup

1. Install the PrintNode desktop agent on each production computer
2. In OrderHub **Settings → Locations**, assign a computer and map Named Printer roles to physical PrintNode printers
3. PDF Layout Studio auto-print rules reference Named Printers, not physical printer names

### Cost

PrintNode is **free for Pixfizz customers** — no separate PrintNode subscription required.

### EasyPost Shipping Label Auto-Print

When EasyPost is configured, purchased shipping labels can also auto-print via PrintNode. Configured per location.

### Troubleshooting: invoice/document not auto-printing
Invoice and document auto-print depends on both sides being aligned: the OrderHub Named Printer role must be mapped to a live PrintNode printer for that location, and the PDF Layout Studio auto-print rule must reference that Named Printer. If invoices are not printing, check for gaps between the OrderHub-side mapping and the PrintNode-side printer/agent configuration — a missing or mismatched mapping silently prevents printing. Source: Fireflies (2026-07-02).

---

## Film Scans Module

Dedicated module for managing scanned negatives workflow.

### Storage

All scan files are stored in **S3** (cloud object storage). The module does not manage physical film — only the digital scan assets.

### Status Tabs

Jobs flow through the following tabs:

| Status | Meaning |
|---|---|
| Needs Processing | Scans received but not yet processed |
| Gallery Pending | Processing started; gallery not yet created |
| Gallery Created | Gallery ready for customer |
| Emailed | Customer has been notified with gallery link |
| Archived | Complete; moved to long-term storage |

### Twin Check Number

Each film scan job has a **Twin Check Number** — a unique identifier used to match physical film rolls to their digital scan files throughout the workflow.

### S3 Auto-Sync

The module supports **S3 Auto-Sync**: when new scan files are deposited to a configured S3 path, they are automatically ingested into the Film Scans queue without manual upload.

---

## OrderHub Downloader (OHD)

Desktop application for photo lab operators to receive, prepare, and route print jobs to local production equipment.

### What it does

OHD runs on a lab's local machine and continuously polls OrderHub for new jobs flagged for local production. When jobs arrive, it:
- Downloads job details and associated print files from OrderHub via API
- Organises files into structured folders with human-readable naming
- Applies channel and product routing logic to match jobs to the correct print workflow
  - **Gotcha:** when updating variants, do not copy old/stale channel IDs from a previous variant into a new one. Carrying over an outdated channel ID causes jobs to route to the wrong workflow (or fail to route) in OrderHub Desktop. Set the channel explicitly per variant, and prefer readable finish/variant values over numeric codes (see the 2026-06-30 variant-value routing note below). Source: Fireflies (2026-07-03).
- Generates **DPOF files** for compatible print controllers (Epson, Noritsu, etc.)
- Provides job review tools: colour correction, quantity management
- Offers **AI-powered upscaling** for low-resolution images
- Updates job status back in OrderHub once downloaded and processed

### Polling Behaviour

OHD polls for **New** jobs not yet received. If multiple OHD instances are running (e.g. across different workstations), job delivery is **first-come-first-served** — a job is only sent to one instance. Instances can be filtered by location.

### Auto-Update

OHD auto-update notifications are delivered via OrderHub. Labs always run the latest version without manual update steps.

### API Integration

OHD uses the OrderHub API to report job status changes back to the platform:

```
POST /functions/v1/update-job-status
Headers: X-API-Key: <api_key>
```

This keeps the OrderHub web UI in sync with local production progress.

> OHD is a companion tool to OrderHub — not standalone. Requires an active OrderHub connection and API key.

---

## EasyPost Shipping Integration

EasyPost provides shipping label generation within OrderHub.

### Setup

Connected via **Manage My Organisation → Shipping** tab.

Toggle between **Production** (live) and **Test** environments during setup and testing.

### Auto-Print

Purchased shipping labels can be automatically printed via PrintNode. Configured per location in Location settings.

---

## POS Integration — Category Filter

When orders arrive from Square or Lightspeed POS systems, OrderHub imports products based on their category.

### Category Filter Behaviour

- The filter is a **case-insensitive exact match** against the category name
- Products whose categories do not match the filter are **silently ignored** — no error is raised
- Lightspeed: categories come from items on the order
- Square: categories come from Line Items

Configure the allowed categories in **Settings → Point of Sale** within OrderHub.

---

## Assigning Pixfizz Categories to Production Processes

When Pixfizz website orders arrive in OrderHub, the platform auto-creates **categories** based on the product types in those orders. Each category must be manually linked to a **Production Process** for correct routing, Production Board display, and job tracking.

### Unassigned Categories Alert

If any categories are unassigned, an **amber alert banner** appears on the Orders page:

> "X categories need process assignment"

Click **View Details** to see the dialog showing each unassigned category, its source website, and creation date.

### Assignment Path

1. Click **View Details** in the amber banner
2. Click **Configure in Organisations**
3. Go to **Organisations page → organisation settings → Pixfizz Websites section**
4. Select the relevant website (e.g. MYLAB)
5. In the **Categories list**, use the dropdown to select a Process for each unassigned category

---

## Order Status Sync (OrderHub → Core)

Marking an order as **shipped** in OrderHub also updates it as **shipped** in Pixfizz Core, provided the integration's API user is enabled. This keeps the Core order status in sync without a separate manual update. If a shipped status set in OrderHub is not appearing in Core, confirm the API user is active. Source: #development, Richard (2026-07-01).

---

## Email & SMS/RCS Notifications

OrderHub can automatically notify customers when an order is shipped or completed. Both channels are configured in the **Notify tab** of Organisation settings.

### Notification Triggers

| Trigger | When it fires |
|---|---|
| Shipped | After a shipping label is purchased via EasyPost, or when an order is manually moved to "shipped" status |
| Completed | When an order is manually moved to "completed" status |

Each trigger has an independent **Send on Shipped** / **Send on Completed** toggle.

### Pixfizz Notification Suppression

When OrderHub notifications are enabled, OrderHub passes `sendNotifications: false` back to the Pixfizz API to prevent duplicate messages. If OrderHub email is disabled, Pixfizz sends its own notifications as normal.

| Scenario | Who sends? |
|---|---|
| OrderHub email enabled + trigger enabled | OrderHub sends; Pixfizz suppressed |
| OrderHub email disabled | Pixfizz sends its own notifications |
| OrderHub SMS enabled + customer has phone | OrderHub sends SMS; Pixfizz doesn't send SMS |
| OrderHub SMS enabled + no phone on order | No SMS sent |
| Manual status change with "Notify" unchecked | Neither sends |

**Use OrderHub for order-confirmation and download emails; separate emails cannot be merged.** When OrderHub notifications are enabled, route order-confirmation and file-download emails through OrderHub. Combining multiple separate emails (e.g. confirmation + download) into a single message is not technically feasible — each remains its own message. Source: Fireflies (2026-06-29, 2026-07-02).

### Email Notifications

Emails are sent via an **n8n / SendGrid pipeline**.

**Prerequisites:**
- An n8n webhook URL configured in Organisation API secrets
- At least one active Pixfizz website linked to the organisation (for branding)

**Branding priority chain:**
1. The Pixfizz website linked directly to the order
2. The Pixfizz website linked to the order's pickup location
3. The organisation's first active Pixfizz website (fallback)

**Email settings:** From Name, Reply-To, BCC, subject line and body for each trigger.

### SMS/RCS Notifications

Sent directly via **Twilio REST API**. Each organisation uses its own Twilio credentials.

**Prerequisites:** Twilio account with active phone number; Account SID, Auth Token, and From Number entered in Notify settings.

**RCS:** Toggle available to enable RCS messaging. Falls back to standard SMS automatically if the recipient's device doesn't support RCS.

### Template Placeholders

These work in both email and SMS templates:

| Placeholder | Replaced with |
|---|---|
| `{{customer_name}}` | Customer name from order |
| `{{order_number}}` | Order number |
| `{{tracking_url}}` | Tracking link (shipped orders only) |
| `{{website_name}}` | Website/brand name |
| `{{customer_email}}` | Customer's email address |

### Two-Way SMS (Inbound)

OrderHub supports inbound SMS via a Twilio webhook, enabling two-way customer conversations.

**Twilio Messaging Webhook URL:**
```
https://nazkcvruighrhpgcarxg.supabase.co/functions/v1/twilio-inbound-webhook
```

Customer replies appear in the **SMS Conversation panel** on the order detail page.

**Automated keyword responses:**

| Keyword | Behaviour |
|---|---|
| `STATUS [order#]` | Looks up the order and replies with current status |
| `STOP` | Handled by Twilio (opt-out compliance) |
| Custom keywords | Configured via Auto-Replies in the Notify settings |

### Notification Log

Every notification attempt is logged on the order in the `notification_log` field, recording: channel (email/SMS), trigger, timestamp, success/failure status, and error details.

### Testing

Both the Email and SMS tabs include a **Send Test** button. Enter any email or phone number and choose a template (shipped or completed). Test notifications use placeholder values ("Order #TEST-001", "Test Customer") — no real order required.

---

## Changelog
- 2026-05-21: Created. Content sourced from OrderHub help modal articles (orderhub.pixfizz.com). Covers: Jobs, custom statuses, Production Board, Processes, Locations, PDF Layout Studio, PrintNode, Film Scans, OHD, EasyPost, POS category filter, Pixfizz category assignment, Email/SMS/RCS notifications.
- 2026-06-15: Added pickup-location opening hours and Google Maps link fields (surfaced in the store pickup UI at checkout). Source: slack-kb-sync (Wolf Camera call).
- 2026-06-30: Documented OrderHub Desktop variant-value routing — Desktop maps on readable finish/variant value + size, not lab/printer-specific numeric codes; prefer readable finish codes. Source: slack-message (#development).
- 2026-07-04: Added Order Status Sync (OrderHub → Core, shipped requires API user enabled); channel-ID copy gotcha in OHD variant updates; email-consolidation limitation (separate emails cannot be merged); PrintNode invoice auto-print troubleshooting. Source: Fireflies, slack-message (#development).
