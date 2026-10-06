# 71 — MyPixfizz Features & Routes

**Authority Scope:** Feature inventory and route map for my.pixfizz.com.

_Last updated: 2026-10-06_

---

## How This Map Was Verified

Route list, navigation and feature state rewritten 2026-10-06 from the app source. *Verified by
reading source (Lovable, commit d6290ee): `src/App.tsx`, `src/config/adminNav.ts`,
`src/components/customer/CustomerSidebar.tsx`, `src/config/tools.ts`,
`src/components/brand-page/BrandPage.tsx` and the pages and functions named below.*

- The source includes builds that were not yet published when this was written. Lovable publishes
  the whole project at once, so a screen that is in the source may not be on my.pixfizz.com yet
  (`70_MYPIXFIZZ_OVERVIEW.md` § Operational Rules for Changing myPixfizz). The live site was not
  checked for this rewrite.
- Several routes in the 2026-03-26 version no longer exist or moved (Corrected 2026-10-06):

| Old route | Now |
|---|---|
| `/portal/*` (customer portal) | Customer routes sit at the root: `/dashboard`, `/support`, `/brand`, `/tools` and so on. There is no `/portal` prefix. |
| `/dashboard` (admin dashboard) | `/dashboard` is the **customer** dashboard. Staff land on Support Overview (`/` and `/admin/support`). |
| `/leads` | `/lead-inbox` |
| `/opportunities/:id` | `/pipeline/:id` |
| `/ideas`, `/roadmap`, `/product-intelligence` (admin) | `/product/ideas`, `/product/roadmap`, `/product/intelligence` ("Insights"). `/roadmap` is now the customer roadmap. |
| `/onboarding` | `/admin/onboarding` |
| `/kickoff/:orgId` | `/welcome` (customer kickoff wizard) |
| `/tasks` (admin global task list) | No admin route. `/tasks` is a customer route with no sidebar item. |

---

## Routing Basics

- **`/`** redirects by role. Staff go to Support Overview (on a phone, to the mobile support app at
  `/m/support`). A customer whose organization is at `pre_kickoff`, `ready_for_kickoff` or
  `kickoff_scheduled` goes to `/welcome`; any other customer with an organization goes to
  `/dashboard`. A signed-in user with no organization sees a no-access screen (staff create the
  organization and send the invite).
- **Public routes:** `/auth` (Password, Email code or Magic link), `/register`,
  `/share/leads/:token` (shared lead list view).
- **Admin portal preview:** every customer route is mirrored under
  `/admin/portal-preview/:orgId/...`, so staff see exactly what that organization sees. Customer
  pages must take the organization from `useCustomerOrg()` and build every internal link with
  `usePortalBasePath()`, or preview breaks (`70_MYPIXFIZZ_OVERVIEW.md` § Recurring Defect Patterns).
- **Mobile support app:** `/m/support`, `/m/support/case/:caseId`,
  `/m/support/unmatched/:unmatchedId`. On a phone, `/admin/support` and `/admin/support-inbox` hand
  over to it, carrying an open case; "Desktop view" in its menu turns the hand-over off for the tab.

---

## Admin Portal Navigation

The admin sidebar and the command palette (Cmd/Ctrl+K) are built from one config,
`src/config/adminNav.ts`. A test enforces its rules: at most 4 pinned items, 7 groups and 7 items
per group; settings-type pages only in Settings; every icon unique; and every admin route in
`App.tsx` is either in the nav or in the hidden-routes list.

| Place | Items (route) |
|---|---|
| Pinned | Support Overview (`/admin/support`), Support Inbox (`/admin/support-inbox`), Executive (`/executive`), Assistant (`/assistant`, internal AI assistant over Pixfizz data) |
| Customers | Organizations (`/organizations`), Contacts (`/contacts`), Brands (`/brands`), Brand Performance (`/performance`), Onboarding (`/admin/onboarding`), Projects (`/projects`), Call Log (`/call-log`) |
| Sales | Pipeline (`/pipeline`), Lead Inbox (`/lead-inbox`), Targets (`/targets`, prospecting targets and outreach) |
| Product | Ideas (`/product/ideas`), Roadmap (`/product/roadmap`), Insights (`/product/intelligence`) |
| Content & Marketing | Marketing (`/admin/marketing`, Pixfizz's own channel telemetry), GA4 Events (`/admin/ga4-events`), Webinars & Events (`/admin/events`), Announcements (`/announcements`, What's New posts), Videos (`/admin/videos`, customer video library) |
| Storefront Tools | Catalog Manager (`/admin/catalog-manager`), POD Catalog (`/admin/pod-catalog`), Tools (`/admin/tools`), Google Reviews (`/admin/reviews`) |
| Finance | Billing (`/billing`), Invoices (`/invoices`), Suppliers (`/suppliers`), Costs & Salaries (`/private/costs`, owner only) |
| Settings | Users (`/users`), Portal Access (`/admin/portal-access`), Support Contacts (`/admin/settings/support-contacts`), Infrastructure (`/infrastructure`) |

Admin routes with no nav item: `/organizations/:id`, `/pipeline/:id`, `/projects/:id`,
`/projects/:id/architect` (Onboarding Architect), `/brands/new`, `/brands/:brandId`,
`/brands/:brandId/performance` (redirects to the brand page Performance tab), `/brands/merge`,
`/admin/onboarding/:id`, `/admin/pod-catalog/:productId`, `/admin/catalog-manager/:brandId`,
`/admin/tools/merge-users`, `/admin/tools/upgrades`, `/admin/support/known-errors`,
`/admin/support/notifications`, `/admin/support/projects/:id`, `/templates`, `/pod-catalog`,
`/costs`.

---

## Admin Portal — Feature Map

### Sales & CRM

| Route | Feature | Notes |
|---|---|---|
| `/pipeline` | Pipeline Kanban | Drag-and-drop deal stages, MRR aggregates in GBP, days-in-stage badges, momentum arrows, stale deal detection (14+ days) |
| `/pipeline/:id` | Deal Detail | Tabbed: Overview, Activity, Notes, Emails, Meetings, Files |
| `/lead-inbox` | Lead Inbox | Inbound leads, AI analysis, manual lead creation. Sidebar badge counts new leads. |
| `/organizations` | Organizations | Master company records. Lifecycle: Lead, Onboarding, Active, On Hold. Add-ons per organization (§ Organization Add-ons in `70_MYPIXFIZZ_OVERVIEW.md`). |
| `/contacts` | Contacts | CRM directory. Bidirectional user and contact conversion. |
| `/projects` | Projects | Internal project hub: notes, meetings, links, Loom videos, documents, per-item client visibility. |
| `/call-log` | Call Log | Fireflies-synced transcripts with AI summaries, action items and org/project matching. |

### Brands (storefronts)

A **brand** is one storefront; the words are interchangeable in the app. One page per brand,
shared by staff (`/brands/:brandId`) and customers (`/brand?storefront=<id>`). Customers and staff
edit the same record on the same page. It replaced the old brand-edit dialog.

| Route | Feature | Notes |
|---|---|---|
| `/brands` | Brands list | One row per storefront. Storefronts with no type get a banner asking for their type. |
| `/brands/:brandId` | Brand page | Tabs below. Staff also see a "Pixfizz setup" card and a **Beta features** card on the Storefront tab. |
| `/brands/merge` | Merge brands | Four-step merge of a duplicate brand into the one kept. Moves the duplicate's linked rows, copies only fields the kept brand has empty, reports conflicts, archives the duplicate. Brands are archived, never hard-deleted. |
| `/performance` | Brand Performance roll-up | Order performance across all brands (archived brands excluded). |

**Storefront type** (`brands.storefront_type`) decides which tabs show. The four types are
Shopper, Shopify + Pixfizz (`shopify_standard`), Custom setup (own website or app on the Pixfizz
API) and Custom CMS (a Pixfizz CMS site not on the Shopper template). Once set, only staff can change
it.

**Brand page tabs:**

| Tab | Shown for | What it holds |
|---|---|---|
| Launch plan | Staff always; customers only when a plan exists | See § Launch Plan. An active plan is the first tab and the default; a launched plan moves to the end as the record. |
| Performance | All | See § Brand Performance. |
| Storefront | All | Name, organization, subdomain, storefront URL, type, currency, website code (order prefix) and billing code. Only staff can change the organization, subdomain, website code or billing code. |
| Connections | All | One card per connection (live orders webhook, GA4, Google Reviews, Pixfizz API access and others). See § Brand Connections and Catalog. |
| Upgrades | Shopper only | Shopper Upgrades for this storefront (§ Tools). |
| Configure | Shopper and Shopify + Pixfizz | Shopper: § Shopper Configure. Shopify + Pixfizz: § Shopify Style. |

The header shows status chips (Live orders, GA4 purchases, Google Reviews; "Shopify + Pixfizz" on
Shopify storefronts) that jump to the matching connection. A save bar appears only while something
is unsaved.

### Onboarding

| Route | Feature | Notes |
|---|---|---|
| `/admin/onboarding` | Onboarding Admin | Plans across customers. |
| `/admin/onboarding/:id` | Plan detail | |
| `/projects/:id/architect` | Onboarding Architect | AI-generated project scaffolding from kickoff data. |
| `/welcome` | Kickoff Wizard (customer) | Multi-section kickoff form, shown before kickoff. |

The old automatic stall and overdue checks were switched off on 2026-09-30 (their cron jobs were
unscheduled by migration).

### Tasks

There is no admin task list route and no admin dashboard route any more (Corrected 2026-10-06). The tasks data and its background jobs remain; the rules below are from the 2026-03-26 version (not re-verified 2026-10-06, except that the `task-mercy-reset` function still exists in source):
- 27 standardized categories, synced between DB and UI. Do not add categories in only one place.
- Recurring patterns: daily, weekdays, weekly, biweekly, monthly, custom. Auto-clone on completion for recurring tasks.
- Midnight Mercy Rule: hourly reset of incomplete focus tasks (`task-mercy-reset`).
- Task Detail Drawer: inline-editable title, delegation context, comments, recurrence, project/org/brand/contact/user linking.
- Daily Planning Flow (Must Win Today, Weekly Wins, Smart Nudges) and Today Focus panel: the components are still in the source, but the admin dashboard that hosted them has no route; where they show now is not re-verified.

### Product & Roadmap

| Route | Feature | Notes |
|---|---|---|
| `/product/ideas` | Ideas | Customer and internal submissions. Fireflies auto-creation with mandatory human review. Voting weighted by org revenue tier. |
| `/product/roadmap` | Roadmap Kanban | Priority scoring: revenue impact, votes, effort, complexity, strategic fit, sales blocker bonus. Public visibility toggle. |
| `/product/intelligence` | Insights | Blocked revenue and product demand signals. |

### Finance & Billing

| Route | Feature | Notes |
|---|---|---|
| `/billing` | Billing | Billing runs, CSVs and customer charges. Monthly billing runs: CSV processing, loyalty discounts, VAT, multi-currency (EUR/USD to GBP conversion) (not re-verified 2026-10-06). |
| `/invoices` | Invoices | QuickBooks-synced invoices, payments, FX. |
| `/suppliers` | Suppliers | Supplier records. |
| (inline) | FX Tracker | Exchange rate monitoring with historical charts (component present in source; where it is shown not re-verified 2026-10-06). |
| `/private/costs` | Costs & Salaries | Owner-only cost and cash planner. Hidden from every other staff account. |

### Infrastructure & Tools

| Route | Feature | Notes |
|---|---|---|
| `/infrastructure` | Infra Monitoring | Service health dashboard; a status strip shows in both portals. |
| `/admin/ga4-events` | GA4 Event Log | Debug and monitor the GA4 purchase pipeline. |
| `/admin/tools` | Admin Tools | Staff tools: Merge users (`/admin/tools/merge-users`) and the Shopper Upgrades catalog editor (`/admin/tools/upgrades`). |
| `/admin/pod-catalog` | POD Catalog | Print-on-demand product catalog and product edit. |
| (global) | Command Palette | Cmd/Ctrl+K on every admin page. |

---

## Support

### Customer side

| Route | Feature |
|---|---|
| `/support` | Case list and support hub. Cases waiting for the customer are listed first with an amber "Your reply needed". |
| `/support/new` | Raise a ticket. `?type=` opens a form directly (`bug`/`issue`, `question`/`ticket`, `order`, `feature`); `?subject=` and `?storefront=<brand id>` prefill. |
| `/support/:caseId` | Case thread. A reply on a resolved case reopens it; closed cases stay locked. |
| `/support/projects`, `/support/projects/new`, `/support/projects/:id` | Project requests: the customer describes paid project work (setup, feature or training), staff reply with a quote, and the customer approves or declines it in the portal before work starts. |
| `/support/subscription`, `/support/agreement` | Subscription and agreement pages (content is commercial and not covered here). |

**Ticket types** (cards on `/support/new`): Report an issue, Ask a question, Request a feature,
and **An order is in error**. Customer-facing wording is "Report an issue", never "bug".

**"How urgent is this?"** replaces the old severity picker on every type except feature requests
(always minor):

| Answer | Severity |
|---|---|
| Not urgent | minor (default for questions) |
| Something is wrong | major (default for issues and order errors) |
| Urgent: customers cannot order | blocking |
| Urgent: we cannot produce | blocking |

Choosing an urgent answer opens a confirm panel: a required "What exactly is blocked right now?"
(200 characters at most) and a required checkbox; submit stays disabled until both are filled. Old
answers on historic cases show with the new labels (`production` as "Urgent: we cannot produce",
`website` as "Urgent: customers cannot order", `one_order` as "Something is wrong").
*Verified by reading source (`_shared/support-urgency.ts`, `SupportCaseNew.tsx`).*

**Order error form.** Order code chips (several codes allowed; pasting an admin order URL works).
The code prefix is the brand's website code, so the form picks the site from it and warns in amber
when a code matches none of the organization's sites. Optional Shopify order number, the error text
from the order, "What have you tried?" checkboxes, and "Anything else?". When the error text matches
a published entry in the known-errors library, a **Known fix** panel appears: "That fixed it" logs a
deflection and sends no ticket. The subject is generated ("Order XX in error") and the first message
is built from the fields.

**Awaiting customer.** A case in Awaiting customer gets a reminder email after 7 days with no
customer reply and is resolved automatically 7 days after the reminder (14 days in all). A further
external staff reply restarts the clock; internal notes do not. Auto-resolved cases get their own
email and no satisfaction survey, and a customer reply reopens them. The customer dashboard shows a
"Waiting for your reply" card and the Support nav item carries a count. *Verified by reading source
(migration `support_awaiting_customer_sweep`, hourly at :07 UTC).*

### Staff side

| Route | Feature | Notes |
|---|---|---|
| `/admin/support` | Support Overview | Support health, volumes and response times. Staff landing page. |
| `/admin/support-inbox` | Support Inbox | Case list and conversation panel. Open urgent cases are pinned at the top with a red "Urgent" label until a staff external reply, a downgrade or closing. Awaiting cases show "Resolves in X days" or "Reminder sent". |
| `/admin/support/known-errors` | Known order errors | The library behind the Known fix panel, with "fixed it" counts. |
| `/admin/support/projects/:id` | Project request (staff) | Scope, quote versions, messages. |
| `/admin/support/notifications` | Support notifications | |
| `/admin/settings/support-contacts` | Support Contacts | Staff directory shown to customers; also the recipients of urgent alerts. Drag-and-drop ordering, per-org visibility, avatar uploads (not re-verified 2026-10-06). |
| (background) | SLA checking | Automated SLA monitoring by the `sla-check` edge function (function present in source). |

- **Urgent alert.** A blocking case emails every active Pixfizz support contact ("URGENT: ...") and
  posts in Slack, mentioning staff who have a Slack user id; priority is set to high. One re-alert
  ("STILL URGENT") goes out after 30 minutes when there is no external staff reply and no owner.
  It runs inside the per-minute support sender, not as its own cron job.
- **"Not urgent? Change to Major"** (staff only) sets severity to major, adds a system line
  "Changed from Urgent to Major by {name}" and emails the customer.
- **Orders block** in the case sidebar: an admin link and copy button per order code, impact, what
  was tried, the error text and the matched fix. It also scans the subject and messages of every
  case, emailed ones included, for `<website code>-XXXX` tokens and links them.
- Admin order links use the order **code**:
  `https://admin.pixfizz.com/site/<subdomain>/admin/orders/<ORDER_CODE>`.
- Customer case emails start with the `[#n]` case tag; inbound replies match on the first `#number`
  in the subject.

*Verified by reading source (Lovable, commit d6290ee). The alert email and Slack post layouts were
not seen live.*

---

## Brand Performance

The Performance tab on the brand page (customer `/brand?tab=performance`, staff
`/brands/:brandId?tab=performance`). *Verified by reading source (RPCs `brand_monthly_sales`,
`brand_benchmarks`, `brand_top_designs`, `msp_card_ranking`; tables `brand_targets`,
`brand_long_goals`).*

- **Monthly sales** come from the billing product data, keyed by the brand's billing code (falls
  back to the subdomain). This covers nearly every billing account, not only brands with the live
  order webhook. Live order data still comes from the order webhook.
- **Targets.** A recommended or custom target per brand (`brand_targets`), editable by the customer.
- **Benchmarks.** Percentile bands (growth, holiday share, photo prints, wall art, gifts) against the
  customer's cohort (`organizations.type`). A cohort needs at least 20 accounts; smaller cohorts fall
  back to "All Pixfizz stores", named on screen. Film processing shows how many accounts sell it and
  the brand's own share. Never a rank or another customer's name.
- **Top designs.** Which designs sell most on multi-design products, from the billing data.
- **MSP card ranking.** Cross-lab ranking of holiday and graduation card designs, returned only for
  organizations that are both IPI members and MSP subscribers; a design must sell at 3 or more labs.
- **Long-term goal.** One goal per brand (a growth multiplier between two dates). Staff can set a
  "special" goal; customers can add and edit only a custom one.

---

## Launch Plan

Per-brand launch plan on the brand page's Launch plan tab, replacing the old onboarding task plan.
*Verified by reading source (migration of 2026-09-30, RPCs `launch_plan_for_brand`,
`launch_plan_create`, `launch_plan_summaries`).*

- One template per deployment path: `shopper`, `shopify`, `custom_api`. The storefront type picks
  it (Custom setup uses `custom_api`; Custom CMS uses `shopper`).
- Staff create a plan from a template with a kickoff date and a target go-live; item due dates are
  the kickoff date plus each template item's day offset. One active plan per brand.
- Every item has an owner (You, Pixfizz or Together), a due date and an optional action (an upload
  or a how-to link). Items can be staff-only.
- Customers can only tick their own items done and upload files to their upload items. Ticking an
  item notifies the plan's Pixfizz contact (or every admin when no contact is set).
- **Call suggestions.** Action items found in calls land in a staff review queue
  (`launch_plan_suggestions`) with the source quote; a staff click adds them to the plan with the
  call as their source. A call is checked once per plan.
- Customers with an active plan see a launch plan card on their dashboard.

---

## Marketing (Campaign Manager)

Customer route `/marketing`, campaign page `/marketing/campaigns/:campaignId`; both mirrored in
portal preview. Beta. *Verified by reading source (Lovable, commit d6290ee).*

- **Hidden behind a per-brand switch.** Staff turn Marketing on per storefront in the Beta features
  card on the brand page's Storefront tab (`brand_features`, `feature_key = 'marketing'`). It is off
  for every storefront by default and only offered for Shopper stores. A customer sees the sidebar
  item, the routes and the dashboard card only when at least one of their brands has it on; a
  direct URL otherwise redirects to the dashboard.
- **Sidebar:** "Marketing" under Work, below Dashboard, with a BETA tag and a count of open
  suggestions. It is the one approved exception to the no-new-nav-items rule.
- **Per store.** Everything is stored per brand. The page header always carries the store picker and
  names the store.
- **Tabs:** Campaigns, Website, Email, Social, Connections. A Send feedback link opens a support
  question.
- **Campaigns:** suggested seasonal campaigns from a seeded calendar of occasions
  (`marketing_occasions`: holiday cards and calendars, Small Business Saturday, gift bundles,
  e-gift vouchers after the last shipping date, year-end photo books, Valentine's Day, Mother's Day,
  graduation, Father's Day, back-to-school), "Not now" dismissals per season, and a campaign plan
  wizard (goal and dates, offer, website, email, social, review and approve).
- **Approve** needs three owner confirmations (discount level, order-by dates, stacking); the
  database refuses an approval without all three. Customers can create and edit only drafts.
- **Nothing is written outside myPixfizz in the current build.** Approving saves the campaign as
  approved. Klaviyo is read only (scheduled emails, flows and lists through the `marketing-read`
  function); no promo code, snippet or Klaviyo draft is created yet.
- **Klaviyo connection:** the brand's Klaviyo private key is kept in the vault
  (`brand_klaviyo_credentials`, with only a short hint stored), set through a backend function, and
  never returned to the browser. Klaviyo calls go through edge functions.

---

## Shopper Configure

The Configure tab on a Shopper storefront's brand page. It replaces the Shopper `setup/*` and
`manage/*` custom admin pages with settings that write the storefront's own snippet overrides
through the Pixfizz admin API. *Verified by reading source (`ShopperConfigureTool`,
`_shared/shopper-settings-schema.ts` schema v3, `shopper-configure` function).*

- **Sections in the current schema:** Store details, Brand, Header menu and footer, Emails,
  In-store kiosk, Pages (home page rows only). Other sections (payments, accounts, prints, marketing
  and search) are not in the schema yet.
- **Level:** Pixfizz CMS snippets on a Shopper child (template level). Settings are the existing
  `admin/checklist/*`, `style/*`, `website/*` and similar snippet keys; no template change.
- **Save** validates every change server-side, writes it (update or create), reads it back and
  compares, then records history (`shopper_configure_history`) and the last saved values.
  **Reset to standard** deletes the store's own snippet so the shopper24 parent value applies again;
  only an id from the child's own snippet list is ever deleted.
- **Team-only rows** (`team_only`) and rows that need a template change are hidden from customers
  and refused by the function. Code fields can only be changed by staff. Asset rows (logo, icons,
  sharing image) are read-only and point to Admin > Website > Assets.
- Needs the brand's Pixfizz API credential; Shopper storefronts only.

**In-store kiosk section** (custom layout):

- **Status card** with four checks before the Kiosk switch unlocks: the kiosk address points to
  Pixfizz (CNAME to `hosting.pixfizz.com`, or the same A records, checked over DNS over HTTPS); the
  security certificate is issued (HTTPS answers); the kiosk address is saved on the site
  (`kiosk-mode-domain`); and the robot check is off on the kiosk (`kiosk-remove-captcha`). Check
  results are never stored. Turning the kiosk off is always allowed.
- Cards when the kiosk is on: Kiosk screen, At checkout, Staff and tips, Photo picker, More settings.
  The "New kiosk interface" switch is team-only while it is in development.
- **Staff list** is the `kiosk_staff` Custom Type on the store, managed through the Pixfizz API
  (list, add, edit, hide, delete, reorder, photo). Staff changes save at once, outside the save bar.
  If the type is missing the card says so: its six fields (`staff_name`, `staff_photo`,
  `staff_sort`, `staff_inactive`, `staff_code`, `staff_location_id`) cannot be created through the
  API.
- Storefront behavior it relies on (template level, `kiosk/associate-tip`): `staff_sort` may sort as
  text, so it is written zero-padded in steps of 10 (`0010`, `0020`); `staff_inactive` values
  `true`, `1` or `yes` hide a person; a record with an empty `staff_name` is skipped; the photo field
  holds the asset **name**, rendered with `asset_url`.
- Turning the photo picker on also sets `kiosk-picker-allow-removable` to TRUE, because the parent
  ships it FALSE and the picker alone then shows nothing.
- Tips also need, per site, the Order fields `associate_name`, `tip_percent`, `tip_amount`,
  `tip_other` and the Extra Fee `kiosk-tip`; there is no API to check them.

---

## Shopify Style

The Configure tab on a Shopify + Pixfizz storefront. It styles the Pixfizz screens shown inside a
Shopify store (options modal, edit project, photo prints, draft order page) and writes the result to
the site's `shopify/custom-styles` snippet through the Pixfizz admin API. Settings are kept per
storefront in `shopify_style_settings` with a history row per save (`shopify_style_history`).
The design tool itself is out of scope. *Verified by reading source (tables and `shopify-style`
function present); the screens were not checked.*

---

## Tools

### Customer Tools page

`/tools` lists the tools as cards in three groups, with a store picker. Every card and tool page
carries a Beta tag while the tool is tested in production. *Verified by reading source
(`src/config/tools.ts`).*

| Group | Tool | Route | For |
|---|---|---|---|
| Set up your store | Shopper Setup | `/tools/shopper-setup` | Shopper |
| Set up your store | Pixfizz Kiosk (Windows) | `/tools/kiosk` | Shopper |
| Set up your store | Live Finish setup | `/tools/live-finish` | Shopper |
| Set up your store | Template resizer | `/tools/template-resizer` | Shopper |
| Products and pricing | Catalog Manager | `/tools/catalog-manager` (`/:brandId`) | Any storefront |
| Products and pricing | Static Product Uploader | `/tools/static-product-uploader` | Any storefront |
| Products and pricing | Pricing formula builder | `/tools/pricing-formula-builder` | Any storefront |
| Customers and marketing | Merge users | `/tools/merge-users` | Any storefront |
| Customers and marketing | Store health check | `/tools/store-health` | Any storefront |
| Customers and marketing | Promo codes | `/tools/promo-codes` | Any storefront |
| Customers and marketing | Customer export | `/tools/customer-export` | Any storefront |
| Customers and marketing | Blog | `/tools/blog`, `/tools/blog/new`, `/tools/blog/:postId` | Shopper |
| Shopper tools | Shopper tools catalog | `/tools/shopper`, `/tools/shopper/flyer-fold` | Shopper |

Coming soon cards: Site Import, Static Pages, Collections. "Start a new Shopper brand" was removed
on 2026-09-30.

Other customer tool routes: `/tools/upgrades`, `/tools/upgrades/:slug`,
`/tools/upgrades/lf-ornaments/setup` (Shopper Upgrades, also shown as the Upgrades tab on a Shopper
brand page).

### What each tool does

- **Shopper Setup:** two downloads for a new Shopper site, one archive with every custom field
  definition and one with the custom types (Kiosk Staff and Google Reviews pieces included). The
  archives ship inside the app and are checked against their SHA-256 before download; "What's in
  this file" lists the objects and types. Custom field definitions do not inherit from parent to
  child, so they are imported per site (`51_CUSTOM_FIELDS_REFERENCE.md`).
- **Pixfizz Kiosk (Windows):** download and setup of the Windows helper app that locks a lab PC to
  a full-screen storefront with USB-only file access.
- **Live Finish setup:** upload a template export; the page reads its pages and choices in the
  browser, asks only what the file cannot answer, shows the live preview in a sandboxed frame, and
  produces either a template option import file (new install) or the settings text to paste into
  an existing Live Finish option. Product types: canvas, card, folded card, print, metal, ornament
  (ornaments preset-based). A page whose name starts with "preview" is never used as the print page.
  See `27_LIVE_FINISH_AND_3D_PREVIEWS.md`.
- **Template resizer:** mode "Move a design to another size". Drop a source template export and a
  destination template export of the same shape (aspect ratios within 0.5%); it scales everything
  inside the trim, keeps the bleed absolute, keeps the calendar date grid as designed, shows proof
  panels, and outputs one design export per selected design for Import design on the destination
  template. Runs entirely in the browser; files never leave the computer.
- **Catalog Manager (customer):** the customer version of the staff tool (below). Customers can
  change prices, stock and pricing on their own store; writes are verified by reading back.
- **Static Product Uploader:** bulk-create ready-made (static) products from a spreadsheet, not
  design templates, with an optional collection step. Each row ends as created, skipped, failed or
  left out. The product create API always creates and never updates (`61_PIXFIZZ_API.md`).
- **Pricing formula builder:** answers to a few questions become a pricing formula
  (`30_PRICING_ENGINE.md`).
- **Merge users:** merge a duplicate storefront account into the main one through the Pixfizz merge
  call; the duplicate is deleted. Customer self-service for their own storefronts (staff can do any).
  Merges are logged (`user_merge_log`).
- **Store health check:** checks a storefront for what stops customers buying or finding it
  (groups: buying, look, Google), with fix, look and good findings. Keeps the last 10 runs per brand;
  an optional weekly run can be switched on.
- **Promo codes:** one shared code, or a batch of single-use codes (up to 5,000), percentage or
  amount, with start and end dates.
- **Customer export:** download the store's customers as a spreadsheet; every export is logged.
- **Blog:** see § Blog.
- **Shopper tools catalog:** one card per Shopper custom design tool. The first tool page is
  Flyer / Brochure (`/tools/shopper/flyer-fold`): sizes, the lab's own variants, the look and the
  install files. Setups save per brand and tool (`shopper_tool_setups`). Per the build report, it
  does not yet read what is already installed on a store, and Shopify storefronts get a card saying
  the tool is made for Shopper (not verified on screen).
- **Shopper Upgrades:** a catalog of upgrades (live previews, design tools, store extras) with
  their state on the lab's site, an upgrade page, a guided setup and an "On your site" list
  (`shopper_upgrades`, `upgrade_installs`). The ornaments setup scans the site's products, then
  hands over one template option import per template plus a guide, and "Check my site" confirms the
  install. Publishing straight to the site waits for a template option write API. Staff edit the
  catalog at `/admin/tools/upgrades`.

### Blog

Customer route `/tools/blog` (Plan and Settings, new post outline at `/tools/blog/new`, editor at
`/tools/blog/:postId`). Shopper storefronts only. *Verified by reading source (tables `blog_*`,
functions `pixfizz-blog`, `blog-ai`, `blog-scheduler`).*

- Posts can be written entirely by hand (visual editor or HTML) or with AI help; AI can be switched
  off per storefront.
- Posts publish to the store's `blog_post` Custom Type through the Pixfizz admin API, then are read
  back and compared; every attempt is logged. Only the publishing service can mark a post published
  or failed.
- **Scheduled posts are written to the store only when they are due** (hourly scheduler), because
  the Shopper 24 post page does not check `blog_unpublished` or future dates (template level).
- Image fields `blog_image` and `blog_thumbnail` are asset fields: the tool writes the asset
  **name**, not the URL. Text fields are capped at 256 characters.
- AI help: ideas, outlines, drafts, rewrites, images, suggested alt text and meta, with monthly
  usage limits per storefront that only staff can change.
- A store without the `blog_post` Custom Type needs it set up first (a ticket can be raised from the
  tool).

---

## Brand Connections and Catalog

| Route | Feature | Notes |
|---|---|---|
| `/brands/:id?tab=connections` | Brand Connections | One card per connection: live orders (order webhook), GA4, Google Reviews, the Pixfizz admin API credential and others. The admin page, the customer portal and Portal Preview share the same brand page. |
| `/admin/catalog-manager/:brandId` | Catalog Manager (staff) | Bulk view and edit of a storefront's products over the Pixfizz admin API. Reached from the storefront's Catalog link, which shows only when the brand has an API credential. |
| `/tools/catalog-manager/:brandId` | Catalog Manager (customer) | The same tool for customers on their own storefronts (Corrected 2026-10-06: it is no longer staff only). |
| `/admin/reviews` | Google Reviews (staff) | Google Reviews sync for storefronts. |

**Google Reviews**
- Connect it on the brand's Connections tab. It needs the brand's Pixfizz admin API credential.
- Sync is manual (**Sync now**).
- Reviews above the auto-approve star threshold (above 4 stars on current setups) arrive ready to publish, but still need **Publish**. Nothing goes live on its own.
- Reviews publish as instances of a Shopper custom type. The review-ID custom field must exist in that type's schema, or publishing fails.
- The product-page widget is switched on separately on the storefront, through its Integrations snippet (`52_SNIPPET_INVENTORY.md`, `integrations/google/product-page-rating-widget`). The widgets emit no review structured data (`81_SEO_AND_GEO_REFERENCE.md`).

**Catalog Manager**
- Needs an admin API credential on the brand. **Test connection** fails when the user behind it is not an admin.
- Adjusts product prices in bulk by a percentage or a fixed amount, then publishes through the API with a read-back check of every row.
- A push can also change stock, stock tracking, maximum units and price variables (value, create, delete). *Verified by reading source (`catalog_push_rows` field list).*
- Variant prices are visible but read-only, because the API has no variant write (`61_PIXFIZZ_API.md` § 13g).
- Stock editing is locked when the organization's stock is managed in OrderHub (`45_ORDERHUB.md` § Stock on OrderHub Sites).
- After a verified write, the grid shows the value the site read back, not the value typed. The platform derives `price_formula` from the shape of the value written, so the two can differ.

*Stated on calls, 2026-09-22, 2026-09-24 and 2026-09-29. Routes and the OrderHub stock lock verified by reading source, 2026-09-28 and 2026-09-29.*

---

## Customer Portal — Feature Map

Routes at the root, all data scoped to the user's organization (RLS). *Sidebar verified by reading
source (`CustomerSidebar.tsx`).*

| Sidebar group | Item | Route | Notes |
|---|---|---|---|
| Work | Dashboard | `/dashboard` | Waiting for your reply (only when cases wait), needs attention, your sales, storefronts table, launch plans (only while one is active), quote approval, Marketing card, next event, your tickets, What's New. |
| Work | Marketing | `/marketing` | Only when a brand has Marketing on (§ Marketing). BETA. |
| Work | Events | `/events` | Upcoming (live indicators, Zoom links, .ics export) and past events. |
| Support | Support | `/support` | § Support. Count of cases waiting for the customer's reply. |
| Support | Iris | `/iris`, `/iris/:conversationId` | Iris assistant. Flagged admin-only in the sidebar and not available in portal preview. |
| Support | Call log | `/call-log-portal` | |
| Learn | What's new | `/whats-new` | Announcements feed. |
| Learn | Videos | `/videos` | Video library. |
| Learn | Roadmap | `/roadmap` | Public roadmap with reactions and feedback. |
| Brand | Storefronts | `/brand` | One storefront opens its brand page; several show a list. `?storefront=<id>&tab=` opens a tab. |
| Brand | Performance | `/brand?tab=performance` | § Brand Performance. |
| Brand | Assets | `/assets` | Documents and links; staff control delete and visibility. |
| Brand | Tools | `/tools` | § Tools. |

- **Open in a new tab:** Claude setup (aisetup.pixfizz.com), Documentation (help.pixfizz.com),
  OrderHub. "Pixfizz Select" shows as Coming soon.
- `/tasks` (external-visible tasks) still exists but has no sidebar item.
- Staff browsing the customer layout get a "Back to admin" link in the header.

---

## Prompt-Writing Notes (for Lovable)

When asking Lovable to modify or build features, always specify:
- Which portal (admin or customer) the change is in
- The route or component name if known
- Whether it involves an edge function, RLS policy, or DB change
- If it touches a Fireflies/QuickBooks/GA4 integration, name the integration explicitly
- For task-related changes: note whether it affects recurring logic, the daily focus system, or the 27-category enum, because these are tightly coupled

---

## Changelog
- 2026-03-26: Initial version. Compiled from Lovable project summary.
- 2026-09-29: Added Brand Connections and Catalog: routes, Google Reviews connector behavior, Catalog Manager behavior. Source: fireflies-call.
- 2026-10-06: Rewrote the route map to the current source (commit d6290ee): corrected the old routes table (no `/portal` prefix, `/dashboard` is the customer dashboard, `/lead-inbox`, `/pipeline/:id`, `/product/*`, `/admin/onboarding`, `/welcome`), routing basics (role redirect, portal preview mirror, mobile support app), admin navigation from `adminNav.ts`, brand page tabs and storefront types, Support (ticket types, urgency, urgent alerts, order error form and known errors, awaiting-customer reminder and auto-resolve, project requests), Brand Performance v2, Launch Plan, Marketing (Campaign Manager), Shopper Configure with the In-store kiosk section, Shopify Style, the Tools catalog (Shopper Setup, Kiosk, Live Finish, Template resizer, customer Catalog Manager, Static Product Uploader, Pricing formula builder, Merge users, Store health, Promo codes, Customer export, Blog, Shopper tools catalog, Shopper Upgrades), and the customer sidebar. Catalog Manager corrected: it now also has a customer route. Restored the task rules, billing run, FX tracker, Support Contacts and SLA checking details. Source: claude-chat.
