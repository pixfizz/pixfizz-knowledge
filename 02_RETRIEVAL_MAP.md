# 02 — Retrieval Map (Use this to pick the right source file fast)

**Authority Scope:** Navigation + retrieval guidance only. This file does not introduce new platform or template rules.

_Last updated: 2026-05-21_

---

## Quick decision tree

### "Who is the user / login / guest / anonymous / merge?"
→ **11_USER_IDENTITY_MODEL.md**

### "What owns what (cart, orderlines, projects, galleries)?"
→ **12_OBJECTS_AND_OWNERSHIP.md**

### "Is this v1 or v2 / which Shopper version does this apply to / version scope convention?"
→ **03_VERSION_SCOPE.md**

### "Is this platform vs Shopper vs custom — where should we implement?"
→ **13_TEMPLATE_BOUNDARIES.md**

### "Cart behavior (editability, option visibility, photo prints quantity, cart UX rules)?"
→ **20_SHOPPER_CART_RULES.md**

### "Checkout behavior (kiosk mode, shipping/pickup unavailable, film, digital-only checkout)?"
→ **21_SHOPPER_CHECKOUT_POLICY.md**

### "How options/variants render (type/selector/multi-upload/triggers)?"
→ **22_OPTION_VARIANT_RENDERING.md**

### "Pricing formulas / price variables / tiering / cut_print_quantity rule?"
→ **30_PRICING_ENGINE.md**

### "Fulfillment templates / job tickets / generated_files / filename templates?"
→ **31_FULFILLMENT_ENGINE.md**

### "High-level mental model of the whole system?"
→ **10_CORE_MENTAL_MODEL.md**

### "Four platform layers / deployment models / Shopify vs Full Pixfizz vs API vs Marketplace?"
→ **15_PLATFORM_ARCHITECTURE.md**

### "Product Attributes / Templates / Designs / Collections / product hierarchy / design vs static product?"
→ **16_PRODUCT_HIERARCHY.md**

### "Can I have the same option code on design and template? / Design option template option conflict / option same-code error?"
→ **16_PRODUCT_HIERARCHY.md** § Variants vs Options

### "Design tool features / feature toggles / design tool configuration / branding / Custom JS/CSS?"
→ **17_DESIGN_TOOL.md**

### "Blend modes / Multiply / Screen / Overlay / element blend mode / group blending context?"
→ **17_DESIGN_TOOL.md** § Blend Modes

### "How do mapped previews work? / Can I get a mapped preview via API? / Mapped preview coordinates / mapped preview page indexing?"
→ **17_DESIGN_TOOL.md** § Mapped Preview Behavior

### "Are page masks included in production output? / Page mask fulfillment / mask presentation only?"
→ **17_DESIGN_TOOL.md** § Page Masks

### "Admin sections / admin sidebar / where is X in admin / settings / super admin?"
→ **18_ADMIN_NAVIGATION.md**

### "XML template syntax / page parameters / bleed / safe area / margin / snap points / hinge / gutter / growing spine / layflat spread / output-name?"
→ **19_XML_TEMPLATE_REFERENCE.md**

### "Can I translate captions? / Caption lang attribute / Multilingual captions / Layout tag translation / Background tag translation?"
→ **19_XML_TEMPLATE_REFERENCE.md** § Caption Translation Limitations

### "Calendar XML / planner XML / date sequences / foreachdate / dategen / dateshift / defdate?"
→ **23_XML_CALENDAR_REFERENCE.md**

### "Order lifecycle / order statuses / production pipeline / OrderHub / OHD / fulfillment destinations?"
→ **32_ORDER_LIFECYCLE.md**

### "OrderHub Jobs / job statuses / custom job statuses / job cascade to order status?"
→ **45_ORDERHUB.md** § Jobs

### "Production Board / Kanban / Timeline view / Status view / drag-and-drop jobs?"
→ **45_ORDERHUB.md** § Production Board

### "Processes / production workflow config / process colour / process categories / lead time?"
→ **45_ORDERHUB.md** § Processes

### "Locations / payment terminals / Stripe Helcim Gravity in OrderHub / printer mappings per location?"
→ **45_ORDERHUB.md** § Locations

### "PDF Layout Studio / PDF ticket designer / production tickets / packing slips / shipping labels / QC checklists / AI layout editor?"
→ **45_ORDERHUB.md** § PDF Layout Studio

### "PrintNode / auto-print / Named Printers / PrintNode setup / printing PDF tickets automatically?"
→ **45_ORDERHUB.md** § PrintNode Integration

### "Film Scans / film scan workflow / scan status / Twin Check Number / S3 auto-sync / gallery pending?"
→ **45_ORDERHUB.md** § Film Scans Module

### "OHD details / OrderHub Downloader / DPOF / AI upscaling / first-come-first-served job polling / OHD multi-instance?"
→ **45_ORDERHUB.md** § OrderHub Downloader

### "EasyPost / shipping labels / OrderHub shipping integration?"
→ **45_ORDERHUB.md** § EasyPost Shipping Integration

### "POS category filter / Lightspeed Square category import / which categories get imported from POS?"
→ **45_ORDERHUB.md** § POS Integration

### "Unassigned categories alert / assigning Pixfizz categories to processes / amber banner on Orders page?"
→ **45_ORDERHUB.md** § Assigning Pixfizz Categories to Production Processes

### "OrderHub email notifications / SMS notifications / RCS / Twilio / customer notify on ship or complete / notification suppression / sendNotifications false / two-way SMS / keyword auto-reply?"
→ **45_ORDERHUB.md** § Email & SMS/RCS Notifications

### "Job thumbnails / thumbnail in Jobs interface / photo print thumbnail / static product no thumbnail?"
→ **45_ORDERHUB.md** § Jobs § Job Thumbnails

### "Onboarding phases / setup process / email notification templates?"
→ **80_ONBOARDING.md**

### "Customer FAQ / store owner question / how do I do X on my Pixfizz site / common operator questions?"
→ **90_FAQ.md**

### "Definitions of terms?"
→ **14_GLOSSARY.md**

### "Troubleshooting mindset + common pitfalls?"
→ **40_PLAYBOOK_UPDATED.md**

### "How do I regenerate a production file? / Delete before regeneration / production file already exists?"
→ **40_PLAYBOOK_UPDATED.md** § Production File Regeneration

### "Font rendering issues / Tofu rectangles in editor / embedded font error in PDF / Transfonter?"
→ **40_PLAYBOOK_UPDATED.md** § Font Stability

### "Code formatting / boundaries / full-block replacement / conventions?"
→ **01_CODE_GOVERNANCE_UPDATED.md**

### "Liquid object properties / filters / tags (what properties exist on cart, orderline, product, user, etc.)?"
→ **50_LIQUID_REFERENCE.md**

### "What custom fields exist on product / collection / design / post / option / cart / user / order (or any other CMS object)?"
→ **51_CUSTOM_FIELDS_REFERENCE.md**

### "What does product.custom.X / collection.custom.X / design.custom.X do?"
→ **51_CUSTOM_FIELDS_REFERENCE.md**

### "Which files use a specific custom field?"
→ **51_CUSTOM_FIELDS_REFERENCE.md**

### "Navigation styles / megamenu structure / nav checklist key?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md**

### "Checklist key name / what value does it take?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md**

### "Style snippet token names / CSS delivery / theming system?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md**

### "Layout structure / page inventory / snippet namespaces?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md**

### "How do I create a new page on a child site of Shopper?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 14 (Custom Type wildcard pattern)
---
### "AI search / GEO / answer engine optimization / how to get cited by AI / AI Overviews / how AI search works?"
→ **81_SEO_AND_GEO_REFERENCE.md**

### "Pixfizz structured data / product schema / LocalBusiness schema / review schema / sitemap / product feed for AI visibility?"
→ **81_SEO_AND_GEO_REFERENCE.md** § Part D, then **50_SHOPPER_TEMPLATE_REFERENCE.md** § SEO & Metadata

---
### "Mobile UX principles / mobile conversion gap / mobile homepage patterns / mobile checkout patterns / mobile audit checklist?"
→ **82_MOBILE_UX_REFERENCE.md**

### "Which mobile is this — storefront, design tool, or kiosk?"
→ **82_MOBILE_UX_REFERENCE.md** § 1. They are three different surfaces and a design does not carry between them.

### "Mobile editor page navigation / show page thumbnails on mobile / px-mobile-page-list / the editor looks wrong until I rotate the phone?"
→ **17_DESIGN_TOOL.md** § Mobile Editor CSS

### "Logo too big on mobile / variant options wrong width on a phone?"
→ **82_MOBILE_UX_REFERENCE.md** § 2 (`header/logo-height-mobile`, `variant_columns_mobile`)

_Note: `83_MOBILE_UX_AUDIT.md` was routed here from before 2026-05-21 and never existed. The audit checklist lives in **82_MOBILE_UX_REFERENCE.md** § 6; 83 is AI imagery production._

### "Cross-cutting implementation patterns / style onload re-injection / once-per-load guard / canvas and form patterns / skip cart redirect / collection filters / measured platform behaviour?"
→ **41_IMPLEMENTATION_PATTERNS_UPDATED.md**

### "Which snippet renders this / what is the snippet called / does the parent ship a snippet for X?"
→ **52_SNIPPET_INVENTORY.md**

---

## Pixfizz API

Use **61_PIXFIZZ_API.md** for all questions about the Pixfizz REST API, JS API, and custom eCommerce integration.

### "What REST endpoints exist / what objects can the API manage?"
→ **61_PIXFIZZ_API.md** § 1 (Overview)

### "How do I authenticate with the API?"
→ **61_PIXFIZZ_API.md** § 2 (Authentication: basic auth, cookie, OAuth)

### "How do I create a project / book via the API?"
→ **61_PIXFIZZ_API.md** § 4 (Projects)

### "How do I trigger PDF generation for a project?"
→ **61_PIXFIZZ_API.md** § 5 (Book Files)

### "How do I push an external order into Pixfizz to trigger fulfillment?"
→ **61_PIXFIZZ_API.md** § 8 (Order Fulfillment)

### "How do I integrate Pixfizz into a non-Shopify external storefront?"
→ **61_PIXFIZZ_API.md** § 7 (Custom eCommerce Integration)

### "How do I pass a user from my site into Pixfizz (user handoff)?"
→ **61_PIXFIZZ_API.md** § 7 (User Handoff)

### "How do I show a preview of a design without creating a project?"
→ **61_PIXFIZZ_API.md** § 9 (Dynamic Design Previews)

### "What JS API functions are available / how do I use Pixfizz.setup / createProject / openProject?"
→ **61_PIXFIZZ_API.md** § 10 (JS API)

### "What does the Shopify JS API (Pixfizz.Shopify.*) do?"
→ **60_SHOPIFY_INTEGRATION.md** (not the general JS API)

### "How does Pixfizz receive status callbacks from a fulfillment partner (Advertek, Navitor, etc.)?"
→ **61_PIXFIZZ_API.md** § 13 (Fulfillment Partner Callbacks)

---

## Shopify Integration

Use **60_SHOPIFY_INTEGRATION.md** for all questions relating to the Shopify + Pixfizz deployment path.

### "What Shopify metafields do I need to set up?"
→ **60_SHOPIFY_INTEGRATION.md** § 2

### "Which integration type should I use for this product?"
→ **60_SHOPIFY_INTEGRATION.md** § 3

### "What do _pixfizz_project_id / _pixfizz_addon / _pixfizz_unit_quantity do?"
→ **60_SHOPIFY_INTEGRATION.md** § 4

### "What does each Pixfizz Shopify snippet do / how do I use it?"
→ **60_SHOPIFY_INTEGRATION.md** § 5

### "How do I set up the Shopify cart page for Pixfizz?"
→ **60_SHOPIFY_INTEGRATION.md** § 7

### "How do addon products / extra pages work in Shopify?"
→ **60_SHOPIFY_INTEGRATION.md** § 8

### "How do I set up the order sync webhook?"
→ **60_SHOPIFY_INTEGRATION.md** § 9

### "How do I link a product in Shopify to Pixfizz?"
→ **60_SHOPIFY_INTEGRATION.md** § 10

### "Why isn't the preview showing / addon not updating / quantity not locking / order not confirming?"
→ **60_SHOPIFY_INTEGRATION.md** § 11

### "What is Buy Now, Personalize Later / how does the draft order flow work?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md**

### "How does the draft button work / what does draft: true do in the launch handler?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 2–3

### "How does the customer approve their draft order / what is orderline_commit?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 7

### "How does the magic link email work for draft orders / what is signin_token?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 6

### "When does fulfillment trigger for a draft order / what is order status W?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 5 + 7

### "Which integration types support Buy Now, Personalize Later?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 1

### "What CMS pages make up the Shopify CMS site (the non-Shopper Pixfizz CMS backing the Shopify integration)?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 9

### "How do I add a Saved Projects or My Galleries page to a Shopify store?"
→ **60_SHOPIFY_INTEGRATION.md** § 13

### "How does the Pixfizz session work on a Shopify page / can I call the Pixfizz API from Shopify JS?"
→ **60_SHOPIFY_INTEGRATION.md** § 13

### "What methods does Pixfizz.Shopify expose / what is Pixfizz.Shopify._user / pixfizz_origin?"
→ **60_SHOPIFY_INTEGRATION.md** § 13

### "How do I get a gallery thumbnail at a higher resolution / thumbnail URL size parameter?"
→ **60_SHOPIFY_INTEGRATION.md** § 13

### "How do I configure local pickup so Shopify orders route to the right address in Pixfizz?"
→ **60_SHOPIFY_INTEGRATION.md** § 12

### "Shopify pickup order has no address in Pixfizz / webhook has no shipping address?"
→ **60_SHOPIFY_INTEGRATION.md** § 12

---

## Scope warning addition

Under "If a site is using the Shopify deployment path" add:

- 62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md (draft order / Buy Now, Personalize Later flow)

---

## MyPixfizz (my.pixfizz.com)

MyPixfizz is a separate product from the Pixfizz CMS platform — it is the internal ERP/CRM and customer self-service portal for Pixfizz. Use the 70-series files for all MyPixfizz questions.

### "What is MyPixfizz / what does it do / who uses it?"
→ **70_MYPIXFIZZ_OVERVIEW.md**

### "What routes/pages exist / what does a specific feature do?"
→ **71_MYPIXFIZZ_FEATURES_ROUTES.md**

### "What database tables exist / what columns does a table have / how are tables related?"
→ **72_MYPIXFIZZ_DATA_MODEL.md**

### "Help me write a Lovable prompt for MyPixfizz"
→ **71_MYPIXFIZZ_FEATURES_ROUTES.md** § Prompt-Writing Notes, then **72_MYPIXFIZZ_DATA_MODEL.md** for table/column accuracy

> **Scope note:** MyPixfizz files (70–72) are entirely separate from Pixfizz CMS files (10–61). Do not apply Shopper, Liquid, or pricing rules to MyPixfizz questions.

---

## Retrieval preference rules (for best answers)

- Prefer **the narrowest file** that matches the question (prevents concept bleed).
- If a question spans layers (e.g., cart + pricing), answer in this order:
	1) Template behavior (Shopper) or integration layer (Shopify / custom)
	2) Platform truth
	3) Pricing/Fulfillment (commercial engine)
- When uncertain whether a behavior is template-level or platform-level:
	→ consult **13_TEMPLATE_BOUNDARIES.md** first.

---

## Scope warning

If a site is not using the Shopper template, treat Shopper files as **non-authoritative** and fall back to:
- 10_CORE_MENTAL_MODEL.md
- 11_USER_IDENTITY_MODEL.md
- 12_OBJECTS_AND_OWNERSHIP.md
- 30_PRICING_ENGINE.md
- 31_FULFILLMENT_ENGINE.md
- 50_SHOPPER_TEMPLATE_REFERENCE.md (Shopper structural anatomy — layouts, nav, theming, checklist keys)

If a site is using the **Shopify deployment path**, Shopper files are non-authoritative for cart/checkout. Use:
- 60_SHOPIFY_INTEGRATION.md (Shopify-specific cart, snippets, metafields, order sync)
- 10/11/12/30/31 (platform truth — still applies)

### "Automatic discounts / cart-level discounts without promo code / auto discount formula?"
→ **30_PRICING_ENGINE.md** § Automatic Discounts

### "Static product importer / CSV product import / bulk product upload?"
→ **18_ADMIN_NAVIGATION.md** § Products (Static Product Importer)

### "Inventory tracking / stock levels / out of stock / product availability?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § Inventory Tracking
→ **18_ADMIN_NAVIGATION.md** § Inventory Management

### "Login modal / sign in from editor / design tool login?"
→ **17_DESIGN_TOOL.md** § Login Modal

### "myPixfizz / customer portal / hub / Meltingspot replacement?"
→ **15_PLATFORM_ARCHITECTURE.md** § myPixfizz Hub

### "Gift vouchers / voucher code tracking / gift card?"
→ **18_ADMIN_NAVIGATION.md** § Marketing (Gift Vouchers)

### "Infrastructure / Rails version / Ruby version / Prawn / PDF library?"
→ **15_PLATFORM_ARCHITECTURE.md** § Infrastructure Versions

### "AI generated images / AI video / Higgsfield / marketing imagery / lifestyle shots / can I use AI product photos?"
→ **83_AI_IMAGERY_PRODUCTION.md**

### "Can AI generate my product images? / AI product photography / is AI imagery allowed on product pages?"
→ **83_AI_IMAGERY_PRODUCTION.md** § Part A (answer is no for the product itself)

### "Customer training / AI setup / webinar schedule / support channels?"
→ **90_FAQ.md** § Section 1 (support channels and training)
→ *Note: `80_ONBOARDING.md` has no "Customer Training and Support Resources" section. This route was recorded in the 2026-05-21 sync against content that was never added. Corrected 2026-08-14.*

### "Communication preferences / email opt-in / SMS opt-in / marketing consent?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § Account Page Redesign (Communication Preferences)
→ **51_CUSTOM_FIELDS_REFERENCE.md** § User fields

If a site is using a **custom eCommerce integration** (external storefront, not Shopify):
- 61_PIXFIZZ_API.md § 7 (user handoff, project workflow, fulfillment)
- 10/11/12/30/31 (platform truth — still applies)
- Shopper and Shopify files are non-authoritative

## Routes added 2026-08-29

### "Style clipart folders / a different image per clipart tag / px-gallery-item / data-gallery-id / the folder icon in the editor?"
→ **17_DESIGN_TOOL.md** § Editor Gallery Folders — Per-Tag Theming

### "My custom colour is in style/custom.css but the live page ignores it / nav links are the wrong colour / why does !important not work?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 18 Generated CSS Is Appended After `style/custom.css`

### "Theming the account area / acv2 / the account pages ignore my CSS / the active sidebar item has no highlight?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 19 Account v2 (`acv2`) Theming

### "GA4 numbers look doubled / purchase fires twice / which Google snippet do I put the measurement id in?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 20 Analytics — GA4 Tagging Defects on the Parent

### "The tar imported but the homepage / promotion bar / logo position is still the template's?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 17, *A trailing newline in a value snippet silently breaks every flag*, and **01_CODE_GOVERNANCE_UPDATED.md** § Value Snippets in a CMS Tar Must Be Byte-Exact

### "How do I preselect in-store pickup or delivery on checkout?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 17, *`default-delivery-option` values are `public` / `private`*

### "Setting pixfizz.product_sku on hundreds of Shopify variants / bulk metafield import?"
→ **60_SHOPIFY_INTEGRATION.md** § 2 Setting variant metafields in bulk

### "414 Request-URI Too Large when clicking Personalize on a photo prints product?"
→ **60_SHOPIFY_INTEGRATION.md** § 11

### "Canvas wrap depth / borderwrap / gallery vs mirror vs colour wrap / what is ipage zoom?"
→ **19_XML_TEMPLATE_REFERENCE.md** § Canvas Wrap Geometry

### "Template import failed with Price can't be blank?"
→ **19_XML_TEMPLATE_REFERENCE.md** § Template Import — `products[].price` Validates Presence

### "Translations / the t filter / my translation file imported and nothing changed?"
→ **50_LIQUID_REFERENCE.md** § Translation Keys

### "Currency switcher on the storefront / showing prices in another currency?"
→ **50_LIQUID_REFERENCE.md** § Display Currency Switching on Shopper 24

### "A variant type with one value renders as a big dark button?"
→ **22_OPTION_VARIANT_RENDERING.md** § A Single-Value Variant Type Renders as a Selected Button

### "Where does postage or a flat fee go / my postage multiplied by the item count?"
→ **30_PRICING_ENGINE.md** § Per-Order Charges Must Never Sit on a Variant

### "Collections export format / what a collections export can and cannot tell me about inheritance?"
→ **01_CODE_GOVERNANCE_UPDATED.md** § The Collections Export Format

### "Spin GIFs are huge / optimising 360 product animations?"
→ **40_PLAYBOOK_UPDATED.md** § Optimising 360-Degree Product Spin GIFs

## Changelog
- 2026-03-13: Added Shopify Integration section and Shopify scope warning.
- 2026-03-26: Added MyPixfizz section pointing to 70/71/72 files.
- 2026-03-30: Added Pixfizz API section pointing to new 61_PIXFIZZ_API.md. Added custom eCommerce scope warning.
- 2026-04-03: Added 19_XML_TEMPLATE_REFERENCE.md entry for XML template page parameters.
- 2026-04-05: Added 51_CUSTOM_FIELDS_REFERENCE.md — master custom field audit across all v2 object types.
- 2026-04-06: Added 90_FAQ.md — customer-facing Q&A for store owners/operators.
- 2026-04-07: Added 03_VERSION_SCOPE.md — v1/v2 versioning convention and audit status.
- 2026-04-09: Added 62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md — full Buy Now, Personalize Later / draft order flow documentation.
- 2026-04-12: Added retrieval entries for §12 (local pickup) and §13 (variable pages / gallery API) in 60_SHOPIFY_INTEGRATION.md.
- 2026-04-18: Added retrieval entry for Design Tool Blend Modes (17_DESIGN_TOOL.md § Blend Modes).
- 2026-04-29: Added retrieval entries for Mapped Preview behavior (17_DESIGN_TOOL.md § Mapped Preview Behavior), Page Masks (17_DESIGN_TOOL.md § Page Masks), Design Option same-code conflict (16_PRODUCT_HIERARCHY.md § Variants vs Options), Caption translation limitations (19_XML_TEMPLATE_REFERENCE.md § Caption Translation Limitations), production file regeneration and font preprocessing (40_PLAYBOOK.md).
- 2026-05-21: Added 45_ORDERHUB.md — comprehensive OrderHub reference (Jobs, Production Board, Processes, Locations, PDF Layout Studio, PrintNode, Film Scans, OHD, EasyPost, POS, category assignment, Email/SMS/RCS notifications). Added retrieval entries for all OrderHub topics. Source: OrderHub help modal articles.
- 2026-05-21: Added routing entries for automatic discounts, static product importer, inventory tracking, login modal, myPixfizz hub, gift vouchers, infrastructure versions, customer training, communication preferences. Source: Q1 2026 webinar KB sync.
- 2026-07-26: Added 83_AI_IMAGERY_PRODUCTION.md — AI marketing imagery and video production with Higgsfield. Added routing entries for AI imagery and the product-representation rule.
- 2026-08-14: Audited the 2026-05-21 routing entries against the files they point at. Two pointed at sections that did not exist: Gift Vouchers (now written into 18_ADMIN_NAVIGATION.md § Marketing) and Customer Training and Support Resources (re-pointed to 90_FAQ.md § Section 1). Source: kbsync audit.
- 2026-08-29: Added 16 routing entries for the 2026-08-29 sync — editor gallery folder theming, generated-CSS specificity, acv2 theming, GA4 defects, the trailing-newline flag failure, `default-delivery-option`, Shopify bulk variant metafields, the photo-prints 414, canvas wrap geometry and ipage zoom, the template-import price validation, translations, the currency switcher, single-value variants, per-order charges on variants, the collections export format, and spin GIF optimisation. Every target section was written in the same sync. Source: kbsync.
- 2026-08-29: Removed a duplicated SEO/GEO block and three leaked kbsync scaffolding lines (RATIONALE / SOURCE / SOURCE TYPE) that had been pasted verbatim into the published file. Replaced the two dangling mobile UX routes with an honest statement of the gap: `82_MOBILE_UX_REFERENCE.md` and `83_MOBILE_UX_AUDIT.md` have never existed in this repo. Source: kbsync audit.
- 2026-08-29: `82_MOBILE_UX_REFERENCE.md` now exists, so the mobile routes point at a real file. Added routes for the three-mobile-surfaces distinction, the mobile editor CSS section in 17, and the two mobile-specific checklist keys. Recorded that the audit checklist lives in 82 § 6 rather than in an 83 file. Source: kbsync.
- 2026-08-29: Audited every filename this map points at against the repo. Corrected `40_PLAYBOOK.md` (3 routes) and `01_CODE_GOVERNANCE.md` (1 route) to their real `_UPDATED` filenames. Added the two files the map had never routed to at all, `41_IMPLEMENTATION_PATTERNS_UPDATED.md` and `52_SNIPPET_INVENTORY.md`. Every route now resolves. Source: kbsync audit.
