# 71 — MyPixfizz Features & Routes

**Authority Scope:** Feature inventory and route map for my.pixfizz.com.

_Last updated: 2026-03-26_

---

## Admin Portal — Feature Map

### Dashboard & Executive

| Route | Feature | Notes |
|---|---|---|
| `/dashboard` | Admin Dashboard | Daily planning, active projects, task focus, smart nudges |
| `/executive` | Executive Dashboard | YTD revenue, YoY comparisons, profitability, IPI market penetration, LTV tracking, cost/revenue charts, marketing health telemetry |

### Sales & Pipeline

| Route | Feature | Notes |
|---|---|---|
| `/pipeline` | Pipeline Kanban | Drag-and-drop deal stages, MRR aggregates in GBP, days-in-stage badges, momentum arrows, stale deal detection (14+ days) |
| `/leads` | Lead Inbox | Inbound leads, AI analysis, manual lead creation, auto-response target within 15 min |
| `/opportunities/:id` | Deal Detail | Tabbed: Overview, Activity, Notes, Emails, Meetings, Files |

### CRM

| Route | Feature | Notes |
|---|---|---|
| `/organizations` | Organizations | Master company records. Lifecycle: Lead → Onboarding → Active → On Hold. Tabs: Users, Tasks, Brands, Meetings, Documents, Links, Activity, Kickoff |
| `/contacts` | Contacts | CRM directory. Bidirectional user↔contact conversion. Auto-sync from platform users. |
| `/brands` | Brands | Per-org brand management. Logos, brand guides, social assets, GA4 config, billing currency, revenue sparklines. |
| `/projects` | Projects | Internal project hub. Notes, meetings, links, Loom videos, documents. Per-item client visibility toggles. |
| `/call-log` | Call Log | Fireflies-synced transcripts with AI summaries, action items, keyword extraction, org/project matching. |

### Task System

| Route | Feature | Notes |
|---|---|---|
| `/tasks` | Global Task List | Unified tasks across sales, onboarding, internal dev |
| (Dashboard panel) | Daily Planning Flow | Multi-step triage: Must Win Today, Weekly Wins, Smart Nudges |
| (Dashboard panel) | Today Focus Panel | Strict daily focus: overdue / due-today / must-win sections |

**Task rules:**
- 27 standardized categories (synced between DB and UI — do not add categories in only one place)
- Recurring patterns: daily, weekdays, weekly, biweekly, monthly, custom
- Auto-clone on completion for recurring tasks
- Midnight Mercy Rule: hourly reset of incomplete focus tasks
- Task Detail Drawer: inline-editable title, delegation context, comments, recurrence, project/org/brand/contact/user linking

### Onboarding

| Route | Feature | Notes |
|---|---|---|
| `/onboarding` | Onboarding Admin | Phase-based accordion, task tracking, stall detection, overdue checks |
| `/kickoff/:orgId` | Kickoff Wizard | Multi-section form: Vision, Branding, Products, Fulfillment, Homepage, Tax/Payment |
| (from Kickoff) | Onboarding Architect | AI-powered project scaffolding generated from kickoff form data |

### Product & Roadmap

| Route | Feature | Notes |
|---|---|---|
| `/ideas` | Ideas | Customer + internal submissions. Fireflies auto-creation with mandatory human review. Voting weighted by org revenue tier. Bulk actions, pagination. |
| `/roadmap` | Roadmap Kanban | Feature cards with priority scoring: revenue impact + votes + effort + complexity + strategic fit + sales blocker bonus. Target windows. Public visibility toggle. |
| `/product-intelligence` | Product Intelligence | IPI penetration tracking, ROI analysis |

### Finance & Billing

| Route | Feature | Notes |
|---|---|---|
| `/invoices` | Invoices | QuickBooks-synced. Line items, payment tracking, FX rates. |
| (from invoices) | Billing Runs | Monthly automated billing. CSV processing, loyalty discounts, VAT, multi-currency (EUR/USD → GBP conversion). |
| `/costs` | Costs | Company expense tracking. Suppliers, categories, receipt uploads. |
| `/suppliers` | Suppliers | Vendor management linked to costs. |
| (inline) | FX Tracker | Real-time exchange rate monitoring with historical charts. |

### Support (Admin)

| Route | Feature | Notes |
|---|---|---|
| `/admin/support` | Support Inbox | Case management. Metrics, assignee filtering, severity/type filters, conversation panel. |
| (inline) | SLA Checking | Automated SLA monitoring via edge function. |
| (settings) | Support Contacts | Managed staff directory. Drag-and-drop ordering, per-org visibility, avatar uploads. |

### Events

| Route | Feature | Notes |
|---|---|---|
| `/admin/events` | Admin Events | CRUD for webinars and workshops. |

### Infrastructure & Tools

| Route | Feature | Notes |
|---|---|---|
| `/infrastructure` | Infra Monitoring | Service health dashboard. |
| (inline) | GA4 Event Log | Debug and monitor the GA4 analytics pipeline. |
| (global) | Command Palette | Keyboard-driven navigation. |
| (global) | Keyboard Shortcuts | Task management hotkeys. |
| `/admin/portal-preview/:orgId` | Portal Preview | Admin view of a specific customer's portal. |

### Brand Connections and Catalog

| Route | Feature | Notes |
|---|---|---|
| `/brands/:id?tab=connections` | Brand Connections | One card per connection: live orders (order webhook), GA4, Google Reviews, the Pixfizz admin API credential and others. The admin page, the customer portal and Portal Preview share the same brand page. |
| `/admin/catalog-manager/:brandId` | Catalog Manager | Bulk view and edit of a storefront's products over the Pixfizz admin API. Staff only; reached from the storefront's Catalog link, which shows only when the brand has an API credential. |

**Google Reviews**
- Connect it on the brand's Connections tab. It needs the brand's Pixfizz admin API credential, which is moving from an admin login to an API key on an admin user.
- Sync is manual (**Sync now**).
- Reviews above the auto-approve star threshold (above 4 stars on current setups) arrive ready to publish, but still need **Publish**. Nothing goes live on its own.
- Reviews publish as instances of a Shopper custom type. The review-ID custom field must exist in that type's schema, or publishing fails.
- The product-page widget is switched on separately on the storefront, through its Integrations snippet (`52_SNIPPET_INVENTORY.md`, `integrations/google/product-page-rating-widget`). The widgets emit no review structured data (`81_SEO_AND_GEO_REFERENCE.md`).

**Catalog Manager**
- Needs an admin API credential on the brand. **Test connection** fails when the user behind it is not an admin.
- Adjusts product prices in bulk by a percentage or a fixed amount, then publishes through the API with a read-back check of every row.
- Variant prices are visible but read-only, because the API has no variant write (`61_PIXFIZZ_API.md` § 13g).
- Stock editing is locked when the organization's stock is managed in OrderHub (`45_ORDERHUB.md` § Stock on OrderHub Sites).
- After a verified write, the grid shows the value the site read back, not the value typed. The platform derives `price_formula` from the shape of the value written, so the two can differ.

*Stated on calls, 2026-09-22, 2026-09-24 and 2026-09-29. Routes and the OrderHub stock lock verified by reading source, 2026-09-28 and 2026-09-29.*

---

## Customer Portal — Feature Map

All routes prefixed `/portal/*`. All data scoped to the user's organization.

| Feature | Notes |
|---|---|
| Customer Dashboard | Brands snapshot, active projects, upcoming events, What's New feed, support hub |
| Brands | Subscription/loyalty details with bidirectional sync to admin side |
| Projects | View projects with client-visible items only (visibility toggle controlled by admin) |
| Assets | Document uploads + link management. Admin controls delete. Client-visible toggles. |
| Roadmap View | Public roadmap features. Customer reactions and feedback on items. |
| Tasks | External-visible tasks only. Customer can update via RPC. |
| Events | Upcoming events (live indicators, Zoom links, .ics export) and past events (YouTube embeds + AI summaries) |
| Support Hub | Submit and track support cases. View support contacts with avatars and Calendly links. |
| What's New | Announcements feed |
| Profile | Account management |

---

## Prompt-Writing Notes (for Lovable)

When asking Lovable to modify or build features, always specify:
- Which portal (admin or customer) the change is in
- The route or component name if known
- Whether it involves an edge function, RLS policy, or DB change
- If it touches a Fireflies/QuickBooks/GA4 integration, name the integration explicitly
- For task-related changes: note whether it affects recurring logic, the daily focus system, or the 27-category enum — these are tightly coupled

---

## Changelog
- 2026-03-26: Initial version. Compiled from Lovable project summary.
- 2026-09-29: Added Brand Connections and Catalog: routes, Google Reviews connector behavior, Catalog Manager behavior. Source: fireflies-call.
