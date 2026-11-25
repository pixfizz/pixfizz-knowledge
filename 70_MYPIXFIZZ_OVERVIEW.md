# 70 — myPixfizz for Customers

## What myPixfizz Is

myPixfizz (`my.pixfizz.com`) is the Pixfizz customer portal. Photo labs and print businesses that use Pixfizz sign in there to get support, follow their launch, see how their stores are doing, use self-service tools and keep up with what is new.

It is separate from the Pixfizz commerce platform itself: storefronts, products, the design tool, orders and fulfillment run in the Pixfizz admin (`admin.pixfizz.com`), not in myPixfizz. Some myPixfizz tools write to a storefront through the Pixfizz admin API, using the storefront's own API credential.

Everything a customer sees is limited to their own organization and its storefronts.

---

## Portal Map

| Group | Item | Route | What it is for |
|---|---|---|---|
| Work | Dashboard | `/dashboard` | Cases waiting for your reply, what needs attention, your sales, your storefronts, an active launch plan, quotes to approve, the next event and What's New. |
| Work | Marketing | `/marketing` | Seasonal campaign planning per store (beta, switched on per storefront). |
| Work | Events | `/events` | Upcoming and past webinars and events, with calendar export. |
| Support | Support | `/support` | Raise and follow support cases and project requests. |
| Support | Call log | `/call-log-portal` | Your calls with Pixfizz. |
| Learn | What's new | `/whats-new` | Platform announcements. |
| Learn | Videos | `/videos` | Video library. |
| Learn | Roadmap | `/roadmap` | Public roadmap with reactions and feedback. |
| Brand | Storefronts | `/brand` | Your storefront pages, with tabs such as Performance, Launch plan, Connections and Configure. |
| Brand | Assets | `/assets` | Shared documents and links. |
| Brand | Tools | `/tools` | Self-service tools (below). |

Links that open in a new tab: the Claude setup guide (`aisetup.pixfizz.com`; the same guide in reference form is `75_AI_ASSISTANT_SETUP.md`), Documentation (`help.pixfizz.com`) and OrderHub. Before kickoff, a new customer lands on `/welcome`.

---

## Support

All Pixfizz support runs through myPixfizz. Support email also lands there as a case. Older help articles that point to a previous helpdesk address or portal are out of date.

- `/support` lists your cases. Cases waiting for your reply come first, marked "Your reply needed".
- `/support/new` raises a case. Types: **Report an issue**, **Ask a question**, **Request a feature** and **An order is in error**. `?type=` (`issue`, `question`, `order`, `feature`), `?subject=` and `?storefront=<brand id>` prefill the form.
- **How urgent is this?** Not urgent, Something is wrong, Urgent: customers cannot order, Urgent: we cannot produce. An urgent answer asks what exactly is blocked (200 characters at most) and a confirmation before it can be sent.
- `/support/:caseId` is the case thread. Replying to a resolved case reopens it; closed cases stay locked.
- **Project requests** (`/support/projects`): describe paid project work (setup, a feature or training), Pixfizz replies with a quote, and you approve or decline it in the portal before work starts.

---

## Storefront Pages

### Performance
Monthly sales per storefront, an editable sales target, benchmark bands against similar Pixfizz stores (growth, holiday share, photo prints, wall art, gifts), top-selling designs and a long-term goal. Benchmarks never show another customer's name or a rank; a group smaller than 20 stores falls back to all Pixfizz stores.

### Launch plan
New storefronts get a launch plan with a kickoff date and target go-live. Every item has an owner (You, Pixfizz or Together) and a due date; you tick your own items done and upload files to upload items, and Pixfizz is notified.

### Connections
One card per connection: live orders, Google Analytics, Google Reviews, the Pixfizz admin API credential and others. Tools that write to your store need the API credential, and the user behind it must be an admin.

**Google Reviews:** connect on the Connections tab, then **Sync now**. Reviews above the auto-approve threshold arrive ready to publish but still need **Publish**; nothing goes live on its own. The product-page rating widget is switched on separately through the storefront's Integrations snippet (`52_SNIPPET_INVENTORY.md`).

### Configure (Shopper storefronts)
Change Shopper settings from myPixfizz instead of the Shopper setup pages: store details, brand, header menu and footer, emails, in-store kiosk and home page rows. Each save is checked and recorded; **Reset to standard** returns a setting to the template default. Logo, icons and sharing image stay in Admin > Website > Assets.

### Style (Shopify + Pixfizz storefronts)
Style the Pixfizz screens shown inside a Shopify store (options window, edit project, photo prints, draft order page).

---

## Marketing (Beta)

Switched on per Shopper storefront. Tabs: Campaigns, Website, Email, Social, Connections. Suggested seasonal campaigns come from a calendar of occasions (holiday cards and calendars, Small Business Saturday, gift bundles, e-gift vouchers after the last shipping date, year-end photo books, Valentine's Day, Mother's Day, graduation, Father's Day, back-to-school). A campaign plan walks through goal and dates, offer, website, email and social; approving needs three confirmations (discount level, order-by dates, stacking). Approving saves the plan in myPixfizz; it does not change your store on its own yet.

---

## Tools

`/tools`, grouped, with a store picker. Tools are marked Beta while they are tested.

| Group | Tool | For |
|---|---|---|
| Set up your store | Shopper Setup | Shopper |
| Set up your store | Pixfizz Kiosk (Windows) | Shopper |
| Set up your store | Live Finish setup | Shopper |
| Set up your store | Template resizer | Shopper |
| Products and pricing | Catalog Manager | Any storefront |
| Products and pricing | Static Product Uploader | Any storefront |
| Products and pricing | Pricing formula builder | Any storefront |
| Customers and marketing | Merge users | Any storefront |
| Customers and marketing | Store health check | Any storefront |
| Customers and marketing | Promo codes | Any storefront |
| Customers and marketing | Customer export | Any storefront |
| Customers and marketing | Blog | Shopper |
| Shopper tools | Shopper tools catalog | Shopper |

- **Shopper Setup:** downloads the custom field definitions and custom types a new Shopper site needs. Custom field definitions do not inherit between sites, so they are imported per site (`51_CUSTOM_FIELDS_REFERENCE.md`).
- **Pixfizz Kiosk (Windows):** the helper app that locks a store PC to a full-screen storefront with USB-only file access.
- **Live Finish setup:** upload a template export, answer what the file cannot tell, preview it live, and get the import file or settings text to install a Live Finish preview (`27_LIVE_FINISH_AND_3D_PREVIEWS.md`).
- **Template resizer:** moves a design to another template of the same shape; runs in the browser and files never leave the computer.
- **Catalog Manager:** bulk-edit prices, stock and pricing on your store, checked by reading back. On stores whose stock is managed in OrderHub, stock is edited in OrderHub instead.
- **Static Product Uploader:** bulk-create ready-made products from a spreadsheet, with an optional collection step (`61_PIXFIZZ_API.md`).
- **Pricing formula builder:** a few questions become a pricing formula (`30_PRICING_ENGINE.md`).
- **Merge users:** merge a duplicate storefront account into the main one; the duplicate is deleted.
- **Store health check:** checks what stops customers buying or finding your store (buying, look, Google). Keeps the last 10 runs; a weekly run can be switched on.
- **Promo codes:** one shared code or a batch of single-use codes (up to 5,000), percentage or amount, with dates.
- **Customer export:** download your store's customers as a spreadsheet.
- **Shopper Upgrades:** a catalog of upgrades (live previews, design tools, store extras) with guided setup and a "Check my site" step.

---

## Sign-in

myPixfizz accounts are one per email address across Pixfizz. From the Pixfizz admin dashboard, the myPixfizz link signs you in with your Pixfizz admin account. Storefront user accounts are separate and per site (`11_USER_IDENTITY_MODEL.md`).

---

## Changelog
- 2026-11-25: Portal Map now points to `75_AI_ASSISTANT_SETUP.md` for the AI setup guide.
- 2026-11-24: Rewritten as a customer-facing overview. The internal system reference that was here, and files 71 and 72, are no longer part of this repository.
