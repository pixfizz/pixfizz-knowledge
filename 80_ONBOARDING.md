# 80 — Onboarding

**Authority Scope:** Onboarding process, deployment path phase sequences, pre-onboarding preparation, launch readiness, and email notification templates.

_Last updated: 2026-05-27_

---

## How to Use This Guide

This document covers the full onboarding process for getting a Pixfizz site live. It applies to all deployment paths.

- **If you're a new Pixfizz customer:** Start with "Preparing for Onboarding" — it tells you exactly what to gather before work begins.
- **If you're a Pixfizz team member:** Use the phase sequences for your deployment path to track progress and identify blockers.
- **If you're a prospect evaluating Pixfizz:** The deployment path overview and preparation checklist will give you a realistic picture of what's involved.

---

## Deployment Paths

Before anything else, confirm which deployment path applies. This determines which phases are in scope and what the customer owns vs what Pixfizz owns.

| Path | Storefront | Cart & Checkout | Personalization | Production |
|------|-----------|-----------------|-----------------|------------|
| Full Pixfizz (Shopper) | Pixfizz (Shopper template) | Pixfizz | Pixfizz | Pixfizz |
| Full Pixfizz (Custom) | Pixfizz (custom template) | Pixfizz | Pixfizz | Pixfizz |
| Shopify + Pixfizz | Shopify | Shopify | Pixfizz | Pixfizz |
| Custom API Integration | Customer's platform | Customer's platform | Pixfizz | Pixfizz |
| Marketplace (Etsy, etc.) | Marketplace | Marketplace | Pixfizz | Pixfizz |

**Full Pixfizz (Shopper)** is the most common path. The Shopper template provides a fully managed storefront with built-in checkout, cart, and navigation. Most photo labs and print businesses use this path.

**Full Pixfizz (Custom)** is for customers who need a completely bespoke storefront within the Pixfizz CMS — different layout, navigation, or UX than Shopper provides. This path inherits the Pixfizz CMS and eCommerce engine but builds the frontend from scratch. Higher cost, longer timeline.

**Shopify + Pixfizz** is for customers who already have or want a Shopify store. Shopify handles the storefront and checkout; Pixfizz handles personalization and production.

**Custom API Integration** is for customers who have an existing eCommerce platform (not Shopify) and want to add Pixfizz personalization and fulfillment. Requires developer resources on the customer's side.

**Marketplace** is for customers who sell through Etsy or similar marketplaces. Pixfizz handles personalization and production; the marketplace handles the storefront and payment.

---

## Preparing for Onboarding — What You Can Do Before We Start

This section is for customers who have committed to Pixfizz and want to get a head start. Gathering these items before the kickoff call significantly reduces onboarding time.

### Required for all deployment paths

**Brand assets:**
- Logo in SVG or high-resolution PNG (minimum 1000px wide)
- Brand colours as hex codes (primary, secondary, accent)
- Brand fonts — if you use custom fonts, provide the font files (OTF or TTF) or the Google Fonts name
- Brand guidelines document if you have one (not required, but helpful)

**Product information:**
- Complete product list — every product you want to sell, with names, descriptions, and categories
- Pricing for each product, including any variant-based pricing (e.g. different sizes at different prices)
- Variant structure for each product: sizes, finishes, paper types, quantity tiers — whatever options customers choose from
- Product images — at least one hero image per product. Lifestyle or in-context photography is ideal but not required at launch

**Production specifications:**
- File format requirements from your lab or print supplier (PDF, JPEG, TIFF, colour profile)
- Bleed and trim specifications per product (your print supplier will know these)
- Any special production requirements (e.g. spine width calculations for books, packaging inserts)

### Required for Full Pixfizz (Shopper or Custom)

Everything above, plus:
- **Domain name** — the domain your store will live on (e.g. shop.yourbusiness.com). You'll need DNS access to point it to Pixfizz.
- **Payment gateway account** — Stripe is the most common. Have your Stripe account created and verified before kickoff. Pixfizz will connect it during onboarding.
- **Shipping rate structure** — flat rate, weight-based, zone-based, or free shipping. Include any free shipping thresholds. If you offer in-store collection, note the pickup location(s).
- **Tax registration** — VAT/sales tax registration details if applicable.
- **Navigation structure** — a rough outline of how you want your site organized. Categories, subcategories, service pages. A simple list or sketch is fine.
- **Content** — About Us text, contact information, FAQs, terms and conditions, privacy policy. These can be drafted during onboarding but having them ready saves time.

### Required for Shopify + Pixfizz

Everything in the "all deployment paths" section, plus:
- **Shopify store** — your Shopify store must be on at least the Basic Shopify plan
- **Admin access** — Pixfizz needs collaborator access to your Shopify admin
- **Existing products** — if you already have products in Shopify, provide a mapping of which ones should have personalization enabled

### Required for Custom API Integration

Everything in the "all deployment paths" section, plus:
- **Developer availability** — this path requires a developer on your side who can integrate with the Pixfizz API. They should be available for the full onboarding period.
- **Platform details** — what eCommerce platform you're running, what language/framework, and how orders currently flow
- **User identity approach** — how your users log in and how you'll pass their identity to Pixfizz (see `61_PIXFIZZ_API.md` § 7 for the user handoff mechanism)
- **Domain setup** — Pixfizz needs a subdomain on your domain (e.g. `design.yoursite.com`) pointing to Pixfizz hosting. This is required for the design tool to work within your site's domain.

### Required for Marketplace (Etsy, etc.)

Everything in the "all deployment paths" section, plus:
- **Active marketplace store** — your Etsy (or other marketplace) store must be set up and active
- **Listing structure** — how your personalized products are listed on the marketplace
- **Order notification setup** — how you'll receive orders from the marketplace (email, API, CSV export)

### Things you do NOT need to have ready

- You don't need final product photography — we can use placeholder images during setup and replace them before launch
- You don't need every product priced to the penny — pricing can be refined during Phase 2, but the structure (flat, tiered, volume-based) should be decided
- You don't need your domain pointed to Pixfizz yet — that happens near the end of onboarding
- You don't need to know how Pixfizz works — that's what onboarding is for

---

## Pre-Onboarding (Pixfizz Internal — Before Kickoff Call)

Before the customer kickoff call, Pixfizz handles these internally:
- Organisation created in myPixfizz (contacts, billing, subscription)
- Pixfizz environment provisioned
- Asset storage and SFTP access configured (if needed)
- Base product templates prepared based on customer's product list
- Production pipeline validated end-to-end

Do not schedule the kickoff call until these are confirmed complete.

---

## Full Pixfizz (Shopper) — Onboarding Phases

A typical Shopper onboarding runs 25–40 tasks across 5 phases, managed in myPixfizz.

### Phase 1: Store Foundations

_Pixfizz completes internally; customer provides brand assets._

- Select and configure Shopper theme
- Upload logo and configure brand colours in style snippets
- Set up custom domain and SSL
- Build primary navigation structure
- Configure homepage (sections, hero, featured products)
- Set announcement bar, footer content, contact information

**Blockers:** Logo, colours, domain/DNS access, and a rough navigation structure from customer.

#### Custom domain and SSL sequence

Domain setup is a sequence with waiting built into it, not a single step to be
left until launch week. Start it early in Phase 1.

1. Customer creates a CNAME for the storefront domain or subdomain pointing at
   `hosting.pixfizz.com`.
2. Register the domain in Pixfizz admin under **Settings → General → Domain
   Hosting**.
3. Wait for DNS propagation. Allow up to **48 hours**. Tell the customer not to
   change DNS records again while propagation is in flight, since further edits
   restart the wait and make the problem much harder to diagnose.
4. Once the domain resolves to Pixfizz, **request the SSL certificate**. SSL is
   not provisioned automatically when DNS changes, it is requested after DNS is
   confirmed correct.
5. Certificate issuance takes roughly **40 minutes** from the request.

If propagation is still failing well past 48 hours, the problem is normally at
the registrar or the previous host rather than at Pixfizz. Have the customer
raise it with whoever manages the domain.

The same sequence applies to every additional domain on a site, including kiosk
domains and the personalization subdomain on a Shopify plus Pixfizz build.

### Phase 2: Product Setup

_Longest phase. Can run in parallel with Phase 1 if assets are ready._

- Define product categories and collection structure
- Create product templates (XML) for each product type
- Create design templates and configure personalization options
- Set up product variants (size, finish, quantity tiers, etc.)
- Configure pricing rules per product (Ruby pricing formulas)
- Upload product imagery and preview designs
- For stores with large static product catalogs (standard print sizes, fixed products without personalization), use the bulk static product CSV importer at **Custom Admin → manage/tools/product-importer**. Download the CSV template directly from that page.

**Blockers:** Complete product catalog, pricing structure, and product images from customer. Pricing must be fully confirmed before launch — changes after go-live cause friction.

### Phase 3: Checkout Configuration

- Connect payment gateway (Stripe typical)
- Configure shipping rules and rates
- Set up tax handling (VAT, sales tax)
- Configure order confirmation and notification emails (14 templates available — see Email Notifications section below)
- Set minimum order amounts if required

**Email delivery:** Shopper email notifications are sent via SendGrid. Deliverability problems are almost always caused by missing or misconfigured SPF/DKIM DNS records on the customer's sending domain. Verify email authentication DNS records during this phase and send test notifications before launch.

**Blockers:** Payment gateway credentials, shipping rate structure, tax registration info.

### Phase 4: Production Setup

- Verify artwork generation for each product template
- Test production file output and confirm format matches lab/production requirements
- Confirm OrderHub job creation per product type
- Validate production file downloads
- Configure fulfillment template (job ticket) if sending to external lab

**Blockers:** Production specs from the lab or print supplier. Must know file format, colour profile, bleed/trim requirements per product before this phase can complete.

### Phase 5: Launch Readiness

- Place internal test orders across all product types
- Validate full order lifecycle end-to-end (order → payment → production → fulfillment)
- Confirm payment capture and production routing
- Verify all email notifications fire correctly
- Content completeness check — all products and pages have descriptions (see Content Completeness section below)
- SEO setup — redirects configured if migrating from an existing site (see SEO Migration section below)
- Soft launch (restricted access or internal only)
- Fix any issues found in soft launch
- Full launch

**Do not skip internal test orders.** Issues found here are far cheaper than issues found by live customers.

---

## Full Pixfizz (Custom) — Onboarding Phases

Custom template builds follow the same phase structure as Shopper but with additional time and scope in Phase 1.

### Phase 1: Store Design & Build

_Significantly longer than Shopper Phase 1. Requires design approval cycles._

- Design review and approval (wireframes, mockups, or reference sites)
- Custom template development (layouts, snippets, CSS, navigation)
- Homepage and key page builds
- Responsive testing across devices
- Custom admin configuration (if applicable)

**Blockers:** Design approval from customer. This phase cannot proceed without signed-off designs. Budget 2–4 weeks for design iteration on top of the build time.

### Phases 2–5

Same as Shopper (Product Setup, Checkout Configuration, Production Setup, Launch Readiness).

---

## Shopify + Pixfizz — Onboarding Phases

### Phase 1: Shopify Store Access

- Customer grants Pixfizz collaborator admin access
- Confirm Shopify plan and theme compatibility
- Review existing product catalog structure
- Identify which products will have personalization

### Phase 2: Pixfizz Integration Setup

- Install Pixfizz Shopify integration code (snippets, scripts, metafields)
- Generate API credentials and connect systems
- Configure webhooks for order sync
- Set up the Pixfizz subdomain on the customer's domain

### Phase 3: Product Synchronization

- Map Shopify products to Pixfizz templates via product metafields
- Configure personalization options per product
- Map variants between Shopify and Pixfizz
- Validate pricing alignment between platforms
- Test the "Personalize" button and design tool launch flow

### Phase 4: Order Flow Validation

- Validate cart integration and personalization data transfer
- Verify orders flow correctly from Shopify to Pixfizz on payment
- Confirm order line item data is complete for production routing
- Test the full checkout flow end-to-end

### Phase 5: Production Verification

- Place test orders through Shopify
- Confirm artwork generation and OrderHub job creation
- Validate production routing and file output for each product type

### Phase 6: Launch

- Validate full personalization and checkout UX on the live theme
- Confirm production output quality
- Enable products for public purchase

---

## Custom API Integration — Onboarding Phases

This path requires developer resources on the customer's side. Pixfizz provides the API, documentation, and support; the customer's developer builds the integration.

### Phase 1: Environment Setup

- Pixfizz environment provisioned
- Customer's subdomain configured (e.g. `design.customersite.com` → `hosting.pixfizz.com`)
- External host authorized for CORS in Pixfizz admin
- API credentials generated and shared securely
- Customer's developer reviews `61_PIXFIZZ_API.md`

### Phase 2: User Handoff Integration

- Implement the user handoff endpoint (POST `/v1/users/_uid/...`) — see `61_PIXFIZZ_API.md` § 7
- Generate the security hash server-side (never expose the shared secret to the browser)
- Call user handoff on every page load where the user is logged in, and before any Pixfizz API interaction
- Test: verify Pixfizz user is created and logged in when handoff is called

**Important:** Users created via the `_uid` handoff endpoint are classified as **external users** — they log in via the customer's platform, not directly into Pixfizz. Do not use the `_uid` endpoint to create users who need direct Pixfizz login access (e.g. admin users or OrderHub operators). Those users must be created via the standard `/v1/users` endpoint.

**Blocker:** This must work before any other integration step. If user handoff fails, nothing else will work.

### Phase 3: Product & Template Setup

- Create product templates (XML) and design templates in Pixfizz
- Configure personalization options and pricing
- Map customer's product IDs to Pixfizz product/template codes
- Test project creation via the API (POST `/v1/books.json`)

### Phase 4: Design Tool Integration

- Implement the design tool launch flow: create project → redirect to editor → handle cart callback
- Configure the `cart_target` URL to receive the completed project ID
- Test the full flow: launch editor → personalize → add to cart on customer's platform
- Test re-editing: reopen a saved project in the editor

### Phase 5: Order & Fulfillment Integration

- Implement order creation via the API (POST `/v1/admin/orders/_external/...`) or
  configure webhook-based order sync
- Test: place an order on the customer's platform → verify it appears in Pixfizz → verify artwork generation → verify production file output
- Configure fulfillment template if needed

### Phase 6: Launch

- End-to-end test of the full flow: browse → personalize → cart → checkout → order → production
- Validate production output quality
- Go live

**Note:** Custom API integrations typically take longer than Shopper or Shopify paths because of the development work on the customer's side. Budget 4–8 weeks depending on the customer's developer availability and platform complexity.

---

## Marketplace (Etsy, etc.) — Onboarding Phases

Marketplace integrations are the simplest from a storefront perspective — the marketplace handles the customer-facing side. Pixfizz handles personalization and production.

### Phase 1: Workflow Design

- Define how customers will submit personalization requests (e.g. upload photos via Etsy message, use a linked Pixfizz design tool page, provide details in order notes)
- Decide whether the design tool will be linked from marketplace listings or managed via a separate Pixfizz page
- Set up the Pixfizz environment and product templates

### Phase 2: Product & Template Setup

- Create product templates matching the marketplace listings
- Configure personalization options
- Set up pricing (the marketplace handles payment; Pixfizz pricing is for internal cost tracking and production routing)

### Phase 3: Order Workflow Setup

- Define how marketplace orders enter Pixfizz:
  - **Manual entry:** operator creates orders in Pixfizz admin from marketplace notifications
  - **CSV import:** batch import orders from marketplace export files
  - **API automation:** use the external order API to push marketplace orders into Pixfizz programmatically (requires development)
- Configure production routing and fulfillment

### Phase 4: Production Verification

- Place test orders through the marketplace workflow
- Verify artwork generation and production file output
- Test the fulfillment and shipping notification flow

### Phase 5: Launch

- Validate the full workflow end-to-end
- Update marketplace listings with personalization instructions or design tool links
- Go live

**Note:** Marketplace integrations vary significantly depending on the marketplace's API capabilities. Etsy's API supports order retrieval but not real-time webhooks for all events, so some manual steps may be required.

---

## Common Blockers by Phase

These are the most common reasons a phase stalls. Flagging them early saves weeks.

| Phase | Typical blocker |
|-------|-----------------|
| Phase 1 | Customer slow to provide logo / brand assets |
| Phase 1 | Domain/DNS access not available (IT department delays) |
| Phase 2 | Product catalog incomplete or pricing not finalized |
| Phase 2 | Customer hasn't decided on variant structure |
| Phase 3 | Payment gateway credentials missing or gateway not yet set up |
| Phase 3 | Shipping rate structure not defined |
| Phase 4 | Production specs not confirmed with lab/supplier |
| Phase 4 | Lab requires a specific file format Pixfizz doesn't output by default (confirm early) |
| Phase 5 | Customer unavailable to approve test orders |
| Phase 5 | Missing content — product descriptions, about page, terms and conditions |

---

## Vertical-Specific Notes

### Photo Labs

- **Kiosk mode:** If the lab needs an in-store kiosk experience, plan for this in Phase 1 — it affects storefront structure. Basic kiosk mode uses a dedicated alternate domain (separate CNAME pointing to `hosting.pixfizz.com` with SSL). The kiosk domain is registered under **Settings → General → Domain Hosting** in admin. Checklist keys to set: `kiosk-mode-enabled: TRUE`, `kiosk-mode-domain: <kiosk-domain>`, `kiosk-pay-in-store-only: TRUE` (restricts pay-in-store to kiosk sessions only). Pay-in-store auto-confirmation is configured separately in admin. If the kiosk offers products at different pricing than the web store, unpublish those collections from the public storefront or access-gate them.
- Establish whether the lab offers in-store collection (affects checkout/shipping config)
- **Film processing:** If offered, needs separate product template setup. 120 and 220 film formats have different frame counts and scan rates — set them up as separate products, not as variants of the same product.
- **Same-day collection:** Same-day or next-day collection messaging is a strong commercial differentiator — plan for it in Phase 1. A JavaScript-based time cutoff (typically after 1pm Mon–Fri) can be configured in Shopper to switch checkout messaging once the same-day window has passed. This is implemented at the snippet level.
- **OrderHub Desktop (OHD):** OHD is a single-location install only. Installing it at multiple workstations in the same location without proper single-instance setup causes order conflicts. Verify Mac compatibility against the current OHD release before recommending it.
- Photo labs often have existing customers who expect a specific workflow — discuss migration messaging

### School / Sports Photography

- Access code or class lookup flow must be planned before navigation is built (Phase 1)
- Package tiers (print bundles vs single prints) must be fully defined before Phase 2
- Seasonal campaigns mean launch timing is critical — delay past peak season has real revenue impact
- High-volume gallery management is a core requirement — plan gallery structure early

### Photo Gifting Brands

- Occasion-based navigation structure should be confirmed in Phase 1 before building nav
- Gift wrapping and delivery date options add complexity to Phase 3 — flag early
- Shopify + Pixfizz is the most common path for gifting brands

### Print-on-Demand / Trade

- Fulfillment routing is the core complexity — multiple production partners, each with different file format and delivery requirements
- Pricing formulas often need wholesale/retail logic — confirm pricing model in Phase 2
- Template auto-import may be relevant if the catalog is large

---

## Content Completeness Before Launch

Before going live, verify that all products and pages have descriptions populated. Missing descriptions are a common missed step during setup.

Missing descriptions cause:
- **Sitemap quality degradation** — blank description entries reduce the usefulness of the XML sitemap submitted to search engines
- **Product feed issues** — Google Shopping and Meta catalogue feeds require descriptions; missing ones cause feed errors or rejected products
- **Google Search Console warnings** — structured data validation flags missing or empty description fields
- **Crawler / feed generation errors** - when the built-in crawler runs (Admin > Website Crawls), products or pages with missing descriptions can throw JSON or crawl errors that interrupt `sitemap.xml` / `product-feed.json` generation. The fix is to populate the missing descriptions in admin. Seen across multiple client sites.

Add a description check to the final pre-launch validation step for all deployment paths. Setting the crawler to run automatically on a daily schedule catches newly added products with blank descriptions before they cause feed errors.

---

## SEO Migration: Sitemap and 301 Redirects

When migrating a customer from an existing website to Pixfizz, SEO continuity requires three things:

1. **URL mapping** — the customer provides a spreadsheet of their current live URLs, categorized by content type (products, pages, blog posts). Map each to the corresponding new Pixfizz URL structure.
2. **301 redirects** — configure redirects in Pixfizz using the JSON config format. The config is an **array of pairs** (array of arrays), even for a single redirect: `[["^/old-path/?$", "/new-path"]]`. Each inner pair is `[regex_pattern, destination]`. A bare single pair (`["^/old-path/?$", "/new-path"]`) **silently fails** with no error and no redirect, so always wrap pairs in the outer array. On sites that use the `/site/` prefix it is included in the pattern. This preserves Google rankings and prevents 404s for bookmarked or indexed URLs.
3. **Sitemap submission** — once the new site is live, submit the new sitemap to Google Search Console. Monitor indexing for the first 2–4 weeks to catch any missed redirects.

System paths (cart, checkout, account, order-confirmation) do not need redirects — they are handled by the platform.

Start the URL mapping exercise early in the onboarding process. It is the customer's responsibility to provide the existing URL list, but Pixfizz should prompt for it and explain why it matters.

---

## Pre-Launch Handoff Checklist

Use this checklist before declaring a site ready for launch.

- [ ] All product types have been test-ordered internally
- [ ] Payment capture confirmed (real test transaction, not test mode only)
- [ ] Production files generated and approved for each product type
- [ ] All email notification templates reviewed — active ones customized for customer brand
- [ ] Email delivery verified — SPF/DKIM DNS records confirmed for SendGrid deliverability
- [ ] Custom domain live with SSL confirmed
- [ ] Navigation and homepage signed off by customer
- [ ] Pricing confirmed and validated against customer's agreed rate card
- [ ] Shipping rules tested with real address(es)
- [ ] All products and pages have descriptions populated
- [ ] 301 redirects configured and tested (if migrating from existing site)
- [ ] Sitemap submitted to Google Search Console (if applicable)
- [ ] Kiosk mode tested on intended hardware (if applicable)
- [ ] Mobile experience reviewed on actual devices
- [ ] Legal pages present (terms, privacy policy, returns policy)

---

## Email Notification Templates

Pixfizz includes 14 email templates mapped to the order lifecycle. Configured in Admin → Settings → Email Notifications. Each can be enabled or disabled individually.

**Order lifecycle emails:**
- Order Pending
- Order Draft
- Order Confirmed
- Order Downloaded
- Order Manufactured
- Order Shipped
- Order Fulfilled
- Order Error
- Order Canceled
- Orderline Fulfilled

**Other emails:**
- Cart Abandoned
- User Signup
- Password Reset
- Sign-in Token Renewed

Review and customize all active templates before launch — default content references generic Pixfizz copy that should be replaced with the customer's brand voice.

---

## Post-Import Checks a Tar Cannot Cover

A CMS tar imports snippets. It cannot set anything that lives outside the snippet
tree, and the failures are silent — the site looks imported and behaves like the
seed. Check these by hand after every import, on the live site:

1. **`admin/checklist/no-index` ships `TRUE` on the parent.** Any live storefront
   needs it set to `FALSE`. Nothing on the page shows it.
2. **The custom home page.** Load the site root and confirm the custom homepage
   renders rather than the seed one, by asserting the wrapper class the homepage
   emits is in the DOM. See 50_SHOPPER_TEMPLATE_REFERENCE.md § 14 and § 17.
3. **The navigation style.** Which `navigation/style*` renders is an admin setting
   a tar cannot read or set. Override every style the parent ships, then view-source
   the live header to confirm which one is actually rendering.
4. **`default-delivery-option`** and the other checkout preselects, if the site
   overrides them — the importer is wipe-and-replace, so a later bundle that does not
   carry the override silently reverts it to the parent value.
5. **Every value snippet's trailing whitespace**, if the bundle was generated rather
   than exported. See 01_CODE_GOVERNANCE_UPDATED.md.

**A local render verifies the file, not the site.** Rendering a snippet through a
Liquid engine and screenshotting it says nothing about a live storefront. Load the
live URL and assert the expected DOM before reporting an import as done.

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-23: Added content completeness (descriptions) pre-launch checklist item.
- 2026-04-27: Added SEO migration workflow (sitemap + 301 redirects for domain moves).
- 2026-06-26: Corrected 301 redirect config to outer-array format (bare single pair fails silently). Added crawler/feed JSON-error failure mode for missing product descriptions and daily auto-crawl note. Source: claude-chat/fireflies-call.
- 2026-05-21: Major rewrite. Added all deployment paths (Custom API, Marketplace/Etsy). Added "Preparing for Onboarding" customer preparation section. Added Full Pixfizz Custom path. Expanded phase sequences with blockers. Merged content from onboarding skill. Added pre-launch handoff checklist. Added vertical-specific notes.
- 2026-05-27: Photo Labs vertical notes: added kiosk mode setup procedure (CNAME, checklist keys, pay-in-store config), OHD single-location install rule, film 120/220 as separate products, same-day JS cutoff pattern. Phase 2: added static product CSV importer note (manage/tools/product-importer). Phase 3: added SendGrid deliverability and DNS authentication note. Pre-launch checklist: added email delivery DNS check. Custom API Phase 2: added external user warning (_uid creates non-login users; OrderHub operators must use /v1/users). Source: Fireflies calls, Slack #dev, support tickets.
- 2026-08-05: Added the custom domain and SSL sequence to Phase 1 (CNAME to hosting.pixfizz.com, register under Settings > General > Domain Hosting, up to 48 hours propagation, SSL requested manually after DNS confirms, roughly 40 minutes to issue). Confirms SSL is not auto-provisioned. Source: fireflies-call.
- 2026-08-29: Added Post-Import Checks a Tar Cannot Cover — `no-index` ships `TRUE` on the parent and must be set to `FALSE` on a live store; assert the custom homepage wrapper class on the live root; confirm which navigation style actually renders because a tar cannot read or set that admin value; re-check checkout preselects after any wipe-and-replace import; and verify value-snippet trailing whitespace on generated bundles. Restated that a local render verifies the file and not the site. Source: claude-chat.
