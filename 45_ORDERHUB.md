# 45 — OrderHub Reference

**Authority Scope:** OrderHub operational configuration, Jobs, Production Board, Processes, Locations, integrations, and notifications. For the core Pixfizz order lifecycle see `32_ORDER_LIFECYCLE.md`.

_Last updated: 2026-10-09_

**The OrderHub help articles are mirrored in `orderhub/`** (one file per article, index in `orderhub/README.md`), regenerated every night from OrderHub. OrderHub is the source of truth for them: never edit `orderhub/` by hand. This file is the curated overview plus the rules that only come from calls, Slack and tickets. Sections that only repeated an article were cut to a pointer on 2026-10-09. Where this file and an article disagree, both are kept and the point is listed for the OrderHub owner to confirm.

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

Jobs are the line items of an order, one per product or service to produce. The standard status flow (New, In Production, Completed) and the order status cascade when every job completes: `orderhub/jobs/jobs-overview.md`, `orderhub/orders/order-status-s.md`.

### Custom Statuses

Each Process can define up to **2 custom statuses**. Custom statuses act as sub-states of **New** — they slot into the workflow between New and In Production. They are useful for multi-step pre-production stages (e.g. "Awaiting Materials", "Sorted").

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

Kanban-style interface for all active jobs: `orderhub/general/production-board.md`.

### Views

**Timeline view** — columns represent due dates. Drag a job card horizontally to change its due date.

**Status view** — columns represent job statuses (New, In Production, Completed, plus any custom statuses). Drag a job card horizontally to change its status.

Both views support filtering by location and date range.

### Drag-and-Drop Behaviour

Dragging a card updates the corresponding field instantly — no confirmation dialog. Changes sync to the order record in real time.

---

## Processes

Processes define production workflows; each job is assigned one when it arrives. Process settings (name, colour, default print size and copies, the OrderHub Desktop toggle, location overrides): `orderhub/general/processes.md`.

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

Locations represent physical sites or branches of the organisation. Full settings: `orderhub/general/locations.md`.

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

Visual drag-and-drop editor for printed production documents (job tickets, order summaries, shipping labels, packing slips, QC checklists): `orderhub/general/pdf-designer-overview.md`.

### Key Features

- Dynamic variables pulled from order and job data (e.g. order number, customer name, product options)
- AI layout editor for rapid template generation
- Auto-print via PrintNode on trigger events
- S3 archiving of generated PDFs

Accessed via **Settings → PDF Designer** in OrderHub.

---

## PrintNode Integration

PrintNode connects PDF Layout Studio to the printers on the lab floor through Named Printers mapped per location. It is free for Pixfizz customers, and purchased EasyPost labels can auto-print through it: `orderhub/general/printing-pdf-tickets.md`, `orderhub/general/printnode-automated-printing-of-pdf-tickets.md`, `orderhub/general/locations.md`.

### Setup

1. Install the PrintNode desktop agent on each production computer
2. In OrderHub **Settings → Locations**, assign a computer and map Named Printer roles to physical PrintNode printers
3. PDF Layout Studio auto-print rules reference Named Printers, not physical printer names

### Troubleshooting: invoice/document not auto-printing
Invoice and document auto-print depends on both sides being aligned: the OrderHub Named Printer role must be mapped to a live PrintNode printer for that location, and the PDF Layout Studio auto-print rule must reference that Named Printer. If invoices are not printing, check for gaps between the OrderHub-side mapping and the PrintNode-side printer/agent configuration — a missing or mismatched mapping silently prevents printing. Source: Fireflies (2026-07-02).

---

## Film Scans Module

Dedicated module for managing scanned negatives workflow. The current articles describe the newer Film Development workflow (`orderhub/film-developing-workflow/`), which hides this legacy Film Scans page; the subsections below describe the legacy module and no article backs them.

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

### Automated Print Job Creation from Rolls

Film scan jobs can automatically generate matching print jobs. When a roll is processed, OrderHub creates print jobs whose quantities match the roll quantities on the order, and uploads the scanned artwork directly to the operator desktop for further processing. This removes the manual step of re-keying a develop-and-print order as a separate print job, and it links into the wider darkroom services workflow.

Because the print jobs are generated from roll quantities rather than from the orderline quantity, verify the generated job count against the order before releasing to production on the first few jobs after enabling this.

---

## OrderHub Downloader (OHD)

Windows application (now called OrderHub Desktop in the articles) for lab operators to receive, prepare and route print jobs to local production equipment: `orderhub/orderhub-desktop/what-is-orderhub-desktop.md`, setup in `orderhub/orderhub-desktop/setting-up-orderhub-desktop.md`.

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

### Polling, batching and the status API

Polling (first come, first served across several instances, filterable by location), batching prints to an Epson Order Controller, and the job status API OHD reports to: `orderhub/jobs/jobs-overview.md`, `orderhub/orderhub-desktop/setting-up-an-epson-surelab-print-controller.md`.

### Auto-Update

OHD auto-update notifications are delivered via OrderHub. Labs always run the latest version without manual update steps.

### Known issue: film scan folders stuck in the OHD watch folder

Film scan folders have been reported not moving out of the OHD watch folder (distinct from the Film Scans Module's S3 Auto-Sync). This is a repeat issue type across support tickets; root cause and fix are not yet confirmed. If a lab reports scans not progressing, check whether files are stalled in the local OHD watch folder before escalating. Source: support ticket #18341 (pending confirmation from dev).

---

## Files Left on the Pixfizz FTP Drop Are Auto-Deleted After a Week

Files placed on the Pixfizz FTP drop are **automatically deleted after a week**. Confirmed by
the core developer, 2026-09-09; stated, not independently verified by watching a file expire.

The consequence is a simpler multi-location workflow than the one labs usually build. A
process that has to reach several locations can **copy the files to all of them and leave
them there**, rather than retrieve-and-delete to keep the drop clean. Nothing has to sweep
the drop, and a location that collects late still finds its files inside the window.

Do not use the drop as storage: a week is the whole retention. Anything that must outlive
that belongs in the order record or in the lab's own storage.

---

## EasyPost Shipping Integration

Shipping labels inside OrderHub, with test and production modes and auto-print through PrintNode: `orderhub/general/setting-up-easypost-integration.md`, `orderhub/ship-desk/ship-desk-shipping-integrations.md`.

---

## POS Application Behaviour

### Loading a new build

The POS application does **not** pick up a new build by backgrounding and returning to it. To load the latest build, the operator must fully **close and reopen** the application.

The current build version is displayed at the **bottom of the login screen** when logged out. Use this to confirm which build a till is actually running before troubleshooting anything version-dependent.

### Screensaver

Burn-in protection, not a logout; 5 minutes by default and set per till: `orderhub/point-of-sale-pos/the-till-screensaver.md`.

### Receipt Printer Paper Size

Receipt paper size is a **software configuration, not a hardware property**. Loading a different paper roll does not change how receipts are formatted.

For the Epson TM-P20II the correct paper size is **58mm**, not the 80mm default assumed for larger countertop printers. The paper size must be set in two places:

1. The **Epson utility** for the printer itself
2. The **Mac print driver** for that printer

If receipts print with wrong margins, truncated lines, or excessive whitespace, check both of these before investigating the receipt template.

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

Pixfizz website orders create categories automatically, and each one must be linked to a Production Process; unlinked categories raise an alert. Steps: `orderhub/general/assigning-pixfizz-categories-to-production-processes.md`. The articles disagree with each other on where the alert shows (the Orders page or the Dashboard).

---

## Order Status Sync (OrderHub → Core)

Marking an order as **shipped** in OrderHub also updates it as **shipped** in Pixfizz Core, provided the integration's API user is enabled. This keeps the Core order status in sync without a separate manual update. If a shipped status set in OrderHub is not appearing in Core, confirm the API user is active. Source: #development, Richard (2026-07-01).

---

## Email & SMS/RCS Notifications

OrderHub notifies customers by email and SMS/RCS when an order is shipped or completed. Setup, triggers, the n8n and SendGrid email pipeline and its branding order, template placeholders, two-way SMS with its Twilio webhook, the notification log and test sends: `orderhub/general/email-and-sms-rcs-notifications.md`. The sections kept below are the parts not in that article.

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

### SMS/RCS Notifications

Sent directly via **Twilio REST API**. Each organisation uses its own Twilio credentials.

**Prerequisites:** Twilio account with active phone number; Account SID, Auth Token, and From Number entered in Notify settings.

**Twilio registration also requires three policy pages, specified in OrderHub, carrying
prescribed text.** Terms and conditions is one of the three. The lab specifies the three
pages in OrderHub, then runs a **validation step** that confirms the submission to the
Twilio API will be accepted before it is sent.

- **Not verified end to end** — the validation step itself is untested, and no lab has been
  taken through the whole sequence to a working number in a session we can cite.
- Expect to do the customer-side part with the lab rather than handing it over: labs
  generally cannot complete it unaided. Budget time for it in onboarding rather than
  treating it as a self-service step.

**RCS:** Toggle available to enable RCS messaging. Falls back to standard SMS automatically if the recipient's device doesn't support RCS.

### Website Chat (Twilio Conversations)

OrderHub can also run a website chat built on Twilio Conversations.

- **The storefront widget is a snippet override under Integrations** on the Shopper site (template-level). The exact snippet path is not recorded here.
- **Chats and staff replies live in OrderHub.**
- **A chat can be assigned to a task**, so a customer request becomes tracked work.

*Stated on calls, 2026-09-25 and 2026-09-28. Not verified by reading source.*

---

## Custom fields consumed by OrderHub must be lowercase

Any custom field that OrderHub is expected to read — order type flags, delivery
speed options, client-specific routing fields — must be named in **lowercase**.
Mixed-case or capitalised field names are not matched.

New fields must also be **whitelisted in OrderHub** before they route. Creating
the field on the Shopper side is not sufficient on its own; a field that exists
and holds a value but was never whitelisted simply does not reach OrderHub, with
no error on either side.

This came up while adding Rush and Urgent delivery classifications as boolean
fields, alongside generic white-labelled option fields for client-specific
order types.

### The five order-level boolean slots

OrderHub supports **five** order-level boolean custom fields, not four. Confirmed
2026-08-13 against the authoritative list circulated by the OrderHub owner:

| Field | Meaning |
|---|---|
| `rush` | Standard rush tier |
| `urgent` | Faster-than-rush tier (typically same day) |
| `option1` | Generic slot, white-labelled per client |
| `option2` | Generic slot, white-labelled per client |
| `option3` | Generic slot, white-labelled per client |

Naming rules that are easy to get wrong:

- **No underscore and no digit separator** — the names are `option1`, `option2`,
  `option3`, not `option_1`.
- `rush` and `urgent` are **mutually exclusive** — model them as one radio group,
  never as two independent checkboxes.
- `option1`–`option3` are independent of each other and of the rush tier. Reserve
  `rush` and `urgent` for genuine delivery-speed tiers; anything else (a
  slow-it-down discount, a VIP flag, a client-specific order type) belongs in a
  generic slot.

**Where the customer-facing label lives is not yet settled.** Two readings are on
record: the label is configured per client **in OrderHub** (so the storefront
sends only the boolean), or the label **travels from the storefront** alongside
the flag (which would require a companion `option1_label`-style text field,
lowercase and separately whitelisted). Confirm with the OrderHub owner before
building the label path — the two designs differ in how many fields need
whitelisting.

**Migration trap.** Where a site already applies a rush charge through an Extra
Fee rule keyed on an older single-string field (for example a `rush_option` field
holding `none` / `standard` / `sameday`), renaming the field **silently drops the
fee**: the customer selects the faster tier, pays nothing, and the order still
reports as urgent. Re-point the Extra Fee rule in the same deploy as the field
rename, never afterwards. Carts already open at cutover will read as no-rush
unless the old values are mapped across.

---

## Print-on-Demand Routing to a Parent Lab's OrderHub

When a child site outsources part of an order to a parent lab, the split is not what most
people assume.

- **The whole order goes to the child site's own fulfillment.** Only the **outsourced items**
  reach the parent lab's OrderHub.
- **Price is looked up, not passed.** The parent looks the line up **by product code and
  variant code against the parent lab's own site** and takes the parent's **wholesale**
  value. Nothing about the price travels from the child's order.
- **A code mismatch does not reject the order — it inserts a zero price.** That zero then
  flows straight into the parent's automatic wholesale invoicing, so the failure surfaces as
  an invoice that is short, not as an error anybody sees at order time.
- **The product feed carries nothing from the template**, so a template-level variant cannot
  be resolved by this route at all.

Stated on a client call and consistent across two labs; not independently verified by
reading source.

**Where the mismatch comes from.** The product code on a child site is auto-populated from
the template code when the template is selected on Publish Products, and it **remains
editable by the child admin** — see `18_ADMIN_NAVIGATION.md`. An edit there is invisible on
the child and fatal on the parent.

**Practical check:** before the first outsourced order, list the child's product and variant
codes for the outsourced lines and diff them against the parent lab's. After go-live, treat
any zero-value wholesale line as a code mismatch until proven otherwise. For the order
states either side of this see `32_ORDER_LIFECYCLE.md`.

A "POD SKU" custom property populated from the template code has been discussed as the fix.
**It is not built. Do not document it as existing.**

---

## No Template Import Endpoint, and Price Variables Are Not in the API

Two current limits worth knowing before designing a bulk workflow:

- **There is no template import endpoint.** Bulk-generated template tar files are imported
  **one at a time through admin**. Large-file handling and progress tracking are the named
  blockers on building one. Stated by the core developer, not independently verified.
- **Price variables: superseded.** This line said price variables were not reachable via the
  API. Read and update are now confirmed on production (`GET /v1/admin/price_variables.json`,
  `PUT /v1/admin/price_variables/<id>.json`); see `61_PIXFIZZ_API.md` § 13f. *Verified by
  query, 2026-09-23.*

Anything that needs either of these has to route through admin by hand. See
`61_PIXFIZZ_API.md` for what the API does cover.

---

## Stock on OrderHub Sites: OrderHub Owns the Count

- **Pixfizz Core has no locations.** It holds one `current_inventory` per product (`16_PRODUCT_HIERARCHY.md`); OrderHub holds stock per location.
- **On each sale OrderHub writes its figure back to Core, overwriting the Core count.** A stock edit made in Core in between (Pixfizz admin, the admin API, a bulk tool) is lost at the next sale.
- **OrderHub can expose either one primary location's stock or the sum of all locations** to Core.
- **On these sites, edit stock only in OrderHub.** The myPixfizz Catalog Manager locks stock editing when the organization's stock is managed in OrderHub (`71_MYPIXFIZZ_FEATURES_ROUTES.md`). The admin API inventory write is an absolute set with no compare-and-set (`61_PIXFIZZ_API.md` § 13g), so no tool can merge its change with a sale.

*Stated by Richard (OrderHub) on a call, 2026-09-25. The Catalog Manager lock is verified by reading source, 2026-09-29. The primary-location or sum choice is set under Settings → Inventory in OrderHub (`orderhub/products/inventory.md`).*

---

## Kiosk and Online Are Separate Catalogues

Confirmed pattern as of August 2026, across more than one lab.

- **Kiosk and online sales need separate products and separate pricing.** Do not
  try to serve one catalogue to both. Walk-in pricing, available sizes and the
  product set a customer can navigate on a touchscreen are all different from the
  web store's.
- **Location-specific order routing runs through OrderHub.** Orders placed at a
  given kiosk route to that location's queue.
- The **Pixfizz Kiosk** Windows app is distributed from myPixfizz (Tools → Pixfizz Kiosk);
  see `18_ADMIN_NAVIGATION.md` § Pixfizz Kiosk App. Kiosk storefront configuration does not
  depend on it: see `80_ONBOARDING.md` for the storefront-side prerequisites. *Stated on
  client calls, 2026-09-22 and 2026-09-24.*

Recorded from client calls; the routing behaviour is consistent with the Locations
model documented above, but the kiosk-to-location binding itself has **not been
verified by reading configuration**.

## Changelog
- 2026-05-21: Created. Content sourced from OrderHub help modal articles (orderhub.pixfizz.com). Covers: Jobs, custom statuses, Production Board, Processes, Locations, PDF Layout Studio, PrintNode, Film Scans, OHD, EasyPost, POS category filter, Pixfizz category assignment, Email/SMS/RCS notifications.
- 2026-06-15: Added pickup-location opening hours and Google Maps link fields (surfaced in the store pickup UI at checkout). Source: slack-kb-sync (client call).
- 2026-06-30: Documented OrderHub Desktop variant-value routing — Desktop maps on readable finish/variant value + size, not lab/printer-specific numeric codes; prefer readable finish codes. Source: slack-message (#development).
- 2026-07-04: Added Order Status Sync (OrderHub → Core, shipped requires API user enabled); channel-ID copy gotcha in OHD variant updates; email-consolidation limitation (separate emails cannot be merged); PrintNode invoice auto-print troubleshooting. Source: Fireflies, slack-message (#development).
- 2026-07-25: Added POS Application Behaviour section (close/reopen required to load a new build; build version shown at bottom of logged-out login screen; 5-minute screensaver is burn-in prevention, not a session timeout; receipt paper size is software config — Epson TM-P20II is 58mm, set in both the Epson utility and the Mac driver). Added automated print job creation from film roll quantities with artwork upload to operator desktop. Source: fireflies-call (3x repeat signal).
- 2026-07-31: Added known issue — film scan folders reported stuck in the OHD watch folder (repeat issue type, root cause/fix not yet confirmed). Source: support ticket #18341 (pending confirmation).
- 2026-08-11: Added the custom field naming rule — any new custom field that OrderHub must read has to be lowercase, and whitelisted in OrderHub before it will route. Source: fireflies-call (2026-08-07).
- 2026-08-14: Corrected the order-level boolean slot count from four to five (`rush`, `urgent`, `option1`, `option2`, `option3`) and documented the no-underscore naming rule, the rush/urgent mutual exclusivity, the unresolved label-ownership question, and the Extra Fee re-point trap when migrating off a single-string rush field. Source: fireflies-call (2026-08-13), slack-message (#development).
- 2026-09-09: Added that files left on the Pixfizz FTP drop are auto-deleted after a week, so a multi-location workflow can copy and leave rather than retrieve-and-delete. Added Print-on-Demand Routing to a Parent Lab's OrderHub — whole order to the child's own fulfillment, outsourced items only to the parent, price looked up by product and variant code against the parent's site at the parent's wholesale value, a mismatch inserting a zero price into automatic wholesale invoicing, and the product feed carrying nothing from the template. Added the Twilio prerequisite of three prescribed policy pages plus a validation step, marked not verified end to end. Added that there is no template import endpoint and that price variables are not reachable via the API. Noted that a Windows kiosk helper application exists and is not yet released. Source: fireflies-call, slack-message.
- 2026-08-29: Added Kiosk and Online Are Separate Catalogues — separate products and pricing for kiosk versus web, with location-specific order routing through OrderHub; the kiosk-to-location binding is not yet verified by reading configuration. Source: fireflies-call (2x repeat signal).
- 2026-09-29: Corrected the kiosk app line: distributed from myPixfizz Tools, not unreleased. Stock on OrderHub sites: OrderHub overwrites Core stock on each sale, primary location or sum, edit stock only in OrderHub. Superseded the stale 'price variables not in the API' line, pointing at 61 § 13f. Website chat on Twilio Conversations: snippet override under Integrations, chats and replies in OrderHub, assignable to a task. OHD batching to an Epson Order Controller (Settings → Routing, Maximum prints per job, Send Batches Automatically). Source: claude-chat, fireflies-call, slack-message.
- 2026-10-09: First pass under the orderhub/ rule. Added the pointer to the `orderhub/` mirror at the top. Cut to pointers the sections that only repeated an article: job statuses and the status cascade, Production Board intro, Process configuration, PDF Layout Studio document types, PrintNode architecture, cost and EasyPost auto-print, OHD polling, Epson batching (its not-verified note is resolved by the article) and the status API, EasyPost, the till screensaver, assigning categories to processes, and the notification triggers, email pipeline, placeholders, two-way SMS, log and testing. Resolved where the stock primary-or-sum choice is set. Kept unchanged, and listed for the OrderHub owner, the points where this file and an article disagree: custom statuses scope, Lead time and Production days meaning, the PDF Layout Studio path, PrintNode install scope, OHD updates, where OHD gets print files, the POS category filter path, the Production Board filters. Source: orderhub-article.
