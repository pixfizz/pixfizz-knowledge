# 02 — Retrieval Map (Use this to pick the right source file fast)

**Authority Scope:** Navigation + retrieval guidance only. This file does not introduce new platform or template rules.

_Last updated: 2026-10-22_

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

### "OrderHub how-to / OrderHub help article / how do I set up X in OrderHub (POS, tills, film developing, OrderHub Desktop, Ship Desk, accounts, staff, Conversations)?"
→ **orderhub/README.md** (index), then the article under `orderhub/<category>/`. These are the OrderHub help articles, mirrored nightly from OrderHub. For platform rules that only live in calls and Slack, also check **45_ORDERHUB.md**.

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

### "Can I read or write snippets, pages, layouts or price variables through the API?"
→ **61_PIXFIZZ_API.md** § 13f (Experimental Admin API — staging only as of 2026-09-16)

### "My script calls /admin/custom_types or /admin/assets — will it keep working?"
→ **61_PIXFIZZ_API.md** § 13c (the `/admin` → `/v1/admin` retirement)

### "Where do I export or import all custom field or custom type definitions at once?"
→ **18_ADMIN_NAVIGATION.md** § Custom Fields, Schema Order and Bulk Export/Import

### "Can customers log in from the upload dialog?"
→ **17_DESIGN_TOOL.md** § Login inside the upload dialog

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

---

## Added 2026-09-09

### "Server-side purchase / GA4 Measurement Protocol / why is revenue double counted / why do purchases land as Direct?"
→ **85_GA4_SERVER_SIDE_PURCHASE.md**

### "Analytics is connected but I see nothing / GTM container but no revenue?"
→ **81_SEO_AND_GEO_REFERENCE.md** § The half-a-chain trap, and **40_PLAYBOOK_UPDATED.md**

### "Do I set the GTM container or the GA4 tag id / both / Google Ads conversions?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § The technical standard: GTM only, and **80_ONBOARDING.md** § Google Account Ownership and Analytics Setup

### "Who owns the customer's Google accounts / can we hold their login?"
→ **80_ONBOARDING.md** § The ownership standard

### "Add to Cart does nothing / dead button / no network request?"
→ **40_PLAYBOOK_UPDATED.md**, then **22_OPTION_VARIANT_RENDERING.md** § A Required File-Upload Option Behind a Trigger Silently Kills Add to Cart

### "Does `disable_required_form` unblock Add to Cart?"
→ **22_OPTION_VARIANT_RENDERING.md** § `disable_required_form` does NOT fix this

### "The editor says Not Found on one variant and works on another?"
→ **60_SHOPIFY_INTEGRATION.md** § Editor "Not Found" on one variant, and **40_PLAYBOOK_UPDATED.md**

### "A Shopify variant does not reach the editor / cart shows two different values for the same option?"
→ **60_SHOPIFY_INTEGRATION.md** § 10b

### "Custom Type instances list as 1, 10, 11, 2, 3?"
→ **16_PRODUCT_HIERARCHY.md** § Custom Type Instances Sort by the Custom Field's Declared Type

### "A product is in no collection but people are still ordering it?"
→ **16_PRODUCT_HIERARCHY.md** § Unpublishing Does Not Make a Product Unreachable

### "Error saving: Price isn't valid / what can a variant price formula contain?"
→ **30_PRICING_ENGINE.md** § Variant Price Formula Validator — Narrower Than Ruby

### "Can a variant formula read `value` and `quantity` together / tier on pages x copies?"
→ **30_PRICING_ENGINE.md** § `value` and `quantity` Are Both In Scope in a Variant Price Formula

### "One more unit costs less / the price ladder goes backwards at a boundary?"
→ **30_PRICING_ENGINE.md** § Whole-order stepped ladders are not monotonic

### "Can an Extra Fee read a cart custom field / how is an Extra Fee formula written / is a fee taxable?"
→ **30_PRICING_ENGINE.md** § Extra Fees, and **21_SHOPPER_CHECKOUT_POLICY.md** § Extra fees and the VAT tax base

### "A 100%-off promo code still leaves a fee on the order?"
→ **30_PRICING_ENGINE.md** § A minimum-order fee survives a 100%-off promo code

### "My page-count pricing formula prices the wrong number of pages on an upload product?"
→ **30_PRICING_ENGINE.md** § `pages` on a product whose artwork is uploaded, not designed

### "Catalogue and category pages show $0 / how do I show a From price?"
→ **30_PRICING_ENGINE.md** § Catalogue and Category Pages Show $0 for Variant-Driven Prices

### "The custom design tool does not appear on this site?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 5 (install order) and § 4 (the mount), **51_CUSTOM_FIELDS_REFERENCE.md** (`custom_script` is a template-option custom field), and **90_FAQ.md** § Section 11 — Silent Failures (symptom first)

### "Which object does a custom field definition archive land on?"
→ **51_CUSTOM_FIELDS_REFERENCE.md**, and **18_ADMIN_NAVIGATION.md** § A custom field definition archive has no object type in it

### "Can I re-import an archive to update a product / why did I get a duplicate with a -1 suffix?"
→ **16_PRODUCT_HIERARCHY.md** § Import Behavior, **51_CUSTOM_FIELDS_REFERENCE.md**, and **01_CODE_GOVERNANCE_UPDATED.md** § Never Re-Import to Update

### "Reusing a template-options export from another site?"
→ **51_CUSTOM_FIELDS_REFERENCE.md** § The Standalone `__template_options.yml` Archive

### "How do I add a feature to a parent snippet without changing every site?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 21. Parent-Safe Changes to Shopper 24, and **41_IMPLEMENTATION_PATTERNS_UPDATED.md**

### "How do I create a new checklist snippet on a child site?"
→ **52_SNIPPET_INVENTORY.md** § Creating a New Snippet — Parent First, Always, and **01_CODE_GOVERNANCE_UPDATED.md** § Parent first, stated at the point of use

### "collection_filters fields / five-field syntax / snippet args / asset_images?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 21.1, and **22_OPTION_VARIANT_RENDERING.md** § `collection_filters` Has Two Syntaxes

### "Nav links work from the homepage and fail between sub-categories?"
→ **50_LIQUID_REFERENCE.md** § 5. Navigation hrefs must be root-relative

### "Why does `{% if x | strip == 'TRUE' %}` break the page?"
→ **50_LIQUID_REFERENCE.md** § 1. Filters are not allowed inside an `{% if %}` condition

### "Free option values render as +$0.00?"
→ **22_OPTION_VARIANT_RENDERING.md** § `value.price` Exports Blank, Not Zero

### "Changing a size filter opens the tool at the old size until I refresh?"
→ **17_DESIGN_TOOL.md** § A dialog reparented to `<body>` survives an AJAX partial re-render

### "The editor opens in the wrong language even though the locale is passed?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § Editor locale needs editor-namespace translations imported, and **18_ADMIN_NAVIGATION.md**

### "Which kiosk checklist keys does a kiosk storefront need?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § Kiosk mode is not touchscreen mode, and **80_ONBOARDING.md** § Kiosk Storefront Prerequisites

### "An outsourced print-on-demand line invoiced at zero?"
→ **32_ORDER_LIFECYCLE.md** § Print-on-Demand Parent/Child Routing, and **45_ORDERHUB.md**

### "How long do files stay on the FTP drop?"
→ **45_ORDERHUB.md** § Files Left on the Pixfizz FTP Drop Are Auto-Deleted After a Week

### "Failed generating files: unexpected token at ']' / invalid _additional_files.json?"
→ **31_FULFILLMENT_ENGINE.md** § The trailing-comma failure

### "Which files must _additional_files.json carry for a tool-driven product?"
→ **31_FULFILLMENT_ENGINE.md** § The four source buckets

### "My regex edit to an exported template definition matched nothing?"
→ **19_XML_TEMPLATE_REFERENCE.md** § Editing an Exported Template Definition — Attribute Order Is Alphabetical

### "Where is a photobook's page count set?"
→ **19_XML_TEMPLATE_REFERENCE.md** § Page Count and `minimum-dpi` Are Definition-Level Decisions

### "Can an AI-generated scene go in a preview page?"
→ **83_AI_IMAGERY_PRODUCTION.md** § A5, and **19_XML_TEMPLATE_REFERENCE.md** § Preview Sets

### "A next-number trigger collides only for customers / duplicate key on submit?"
→ **70_MYPIXFIZZ_OVERVIEW.md** § RLS and Aggregates in Triggers

### "Can I read or write Price Variables through the API / is there a template import endpoint?"
→ **61_PIXFIZZ_API.md** § 13e. What Is Not Possible Today

### "Where do support tickets go now?"
→ **70_MYPIXFIZZ_OVERVIEW.md** § Support Intake, **80_ONBOARDING.md**, and **90_FAQ.md** § Section 1

### "It fails silently and there is nothing in the console — what class of fault is this?"
→ **90_FAQ.md** § Section 11 — Silent Failures (symptom first)


## Routes added 2026-09-19 — custom design tools

### "Which custom design tools exist / what is the current version of one / is X ready to sell?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 2 — the registry. A tool absent from that table does not exist.

### "What does the Sticker Designer / Gang Up / Business Cards / Cover Studio / Publication Upload / Document Uploader / Booklet Uploader / Fan Face / Flyer Fold / Design Brief / Film Builder do, and what does it write to the orderline?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 7 — per-tool reference

### "Where does a custom design tool read its settings from / can I put this in a product custom field?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 4 — the mount argument list, and nothing else. Supersedes the settings cascade in every spec written before 11 Sep 2026.

### "How do I install a custom design tool on a new site?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 5. Two steps, not four — `41_IMPLEMENTATION_PATTERNS_UPDATED.md` and `51_CUSTOM_FIELDS_REFERENCE.md` carry the same order.

### "Should this product be a custom design tool or a template with options?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 1 — three tests, all of which must be true

### "The tool's preview does not show in the cart / saved projects / the order email?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 6 — the six surfaces, the two variable spellings, and the two live defects

### "What has to be true before a custom design tool is finished?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 8 — definition of done

### "Why can't the pricing formula see the paper / colour / binding the customer chose?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 1, and **30_PRICING_ENGINE.md**. The tool resolves the rate card in the browser and writes one number to a variant whose formula is `value`.

---

## Routes added 2026-09-24

### "I'm writing install / setup steps for a Shopper site — can I tell them to create a snippet?"
→ **01_CODE_GOVERNANCE_UPDATED.md** § Install-Step Gate. Run it on every set of install instructions. Only shopper24 can create snippets; for a customer the answer is "contact Pixfizz support".

### "Why can't I find this snippet in the child site's admin?"
→ **13_TEMPLATE_BOUNDARIES.md** § A Child Site's Snippet List Shows Only Its Overrides

### "How do PDF layers work / how do I hide something from the customer but print it / split foil or white ink into its own file?"
→ **19_XML_TEMPLATE_REFERENCE.md** § PDF Layers in Practice

### "How do I show something in the editor that must never print?" / "Why did part of the design not print?"
→ **17_DESIGN_TOOL.md** § Show in the Editor, Never Print — the Uneditable Placeholder

### "I can't click an element because another one is on top"
→ **17_DESIGN_TOOL.md** § Grouping Elements to Hide Them While Editing — View Settings

### "Does a template export carry its layouts?"
→ **19_XML_TEMPLATE_REFERENCE.md** § Layouts Travel With a Template Export (corrected 2026-09-24)

### "Layout swap stacks two placeholders" / "Duplicate filename when fulfilling project on JPEG output"
→ **19_XML_TEMPLATE_REFERENCE.md** § Layout Content Requires `layout="true"`; § Parameter Reference (`output-name`)

### "Are variant types shared between products?" / "Should this choice be a variant or a template option?"
→ **22_OPTION_VARIANT_RENDERING.md** § Variant Type Exports (corrected) and § Pricing and POS-Relevant Choices Belong on Variants

### "Kiosk customer's session carried over / order never placed / recover a guest's photos"
→ **21_SHOPPER_CHECKOUT_POLICY.md** § Kiosk Sessions

### "Is staging safe for testing a data change?"
→ **80_ONBOARDING.md** § Staging and Production Share One Database. No: one database.

### "Admin API calls return 401 since 23 Sep" / "Products API fields, page size, inventory writes"
→ **61_PIXFIZZ_API.md** § 13c (retirement is live), § 1 (page size), § 13g (Products API)

### "Child CSS does nothing" / "where does CSS go on the parent?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § CSS delivery and § 18.1

### "Static Product Importer hung / products in no collection" / "filter a static collection by category"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 15 importer notes and § 23

### "Product save silently does nothing" / "text custom field truncated"
→ **51_CUSTOM_FIELDS_REFERENCE.md** (1,024-character description limit; 256-character text fields)

## Routes added 2026-09-29

### "Roll a template option, a mount or a variant set across many live templates" / "bulk update templates"
→ **18_ADMIN_NAVIGATION.md** § Bulk Update Tools: Rolling a Change Across Live Templates. Never hand-edit each template, never delete and re-import. Pointers in 22 and 26.

### "API key" / "Basic auth" / "how should an integration authenticate?"
→ **61_PIXFIZZ_API.md** § API keys (recommended for every Basic-auth call), then § Admin UI host vs API host

### "Admin login / 2FA / TOTP / passkey / super admin URL / impersonation timeout?"
→ **18_ADMIN_NAVIGATION.md** § Admin Login: Two-Factor and Passkeys

### "Which order notification email fires when? / BCC on confirmation / Fulfilled vs Shipped email?"
→ **32_ORDER_LIFECYCLE.md** § Which notification email fires when

### "OrderHub site stock / inventory overwritten / where to edit stock?"
→ **45_ORDERHUB.md** § Stock on OrderHub Sites: OrderHub Owns the Count

### "Changed a template but existing projects still have the old size / refulfill uses old geometry?"
→ **19_XML_TEMPLATE_REFERENCE.md** § Template Changes Do Not Reach Existing Projects

### "Rotate an element in page XML / landscape layout on a portrait page?"
→ **19_XML_TEMPLATE_REFERENCE.md** § Element Rotation in Page XML: `rotate`

### "A PDP swap left my custom tool showing the previous product's settings?"
→ **26_CUSTOM_DESIGN_TOOLS.md** (PDP swap keeps the `custom_script` root and rewrites its attributes)

### "myPixfizz Google Reviews / brand connections / Catalog Manager?"
→ **71_MYPIXFIZZ_FEATURES_ROUTES.md** § Brand Connections and Catalog

## Routes added 2026-10-06

### "Live Finish / lf_mount / metal or acrylic finish preview / tilt / room view?"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 2 The Live Finish standard. New file: product previews are not custom design tools.

### "Live Finish shows blank, 0 px, or not at all" / "which page is the print page (lf_page)?"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 2.6 Gating and § 2.4 Geometry comes from the template; dark sites § 2.7

### "3D preview / p3d_mount / mug preview / View in 3D / Blender / GLB model?"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 3 The 3D Preview standard (§ 3.3 models built in Blender)

### "Which product families have Live Finish or 3D today / known preview bugs / install a preview on a site?"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 5 Status by product family, § 6 Known defects, § 7 Installing on a site

### "Which custom design tools exist now / Uploader 2.0 / NCR forms / Wall Designer / Board Engraver / Gang Up trim and cut path?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 2 registry and § 7 per-tool reference (updated 2026-10-06)

### "Which option codes does a tool write for fulfillment / px_print_file / px_preview / legacy codes / FTP artwork paths?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 3 The fulfillment contract, then **31_FULFILLMENT_ENGINE.md** § Custom Tool Fulfillment Standard

### "Tool price ladder / price_forecast / cent rounding / tool on a store that prices with template options?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 1 Price ladders read the platform price, § 4 Stores that price with template options; **30_PRICING_ENGINE.md** § Platform Price per Quantity and Cart Rounding

### "Tool looks wrong on a dark site / pxt tokens / inline tool page layout / inches or mm?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 3 The shared look, § 3 Page layout, § 4 Lengths accept inches and millimeters

### "Every customer choice on a variant / pos_hidden / recall plan?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 4 Every customer choice on a variant; **22_OPTION_VARIANT_RENDERING.md** § Pricing and POS-Relevant Choices Belong on Variants

### "Tool's cart or checkout image is wrong?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 6 The tool chooses its own cart image

### "Open the upload dialog from a tool / photo prints intercept hooks (goToCropOptions)?"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 3 Open the platform upload dialog standalone; **41_IMPLEMENTATION_PATTERNS_UPDATED.md** § PhotoPrintsComponent hooks

### "myPixfizz routes / sidebar / which page does X / portal preview?"
→ **71_MYPIXFIZZ_FEATURES_ROUTES.md** (rewritten 2026-10-06 from the app source): § Routing Basics, § Admin Portal Navigation

### "myPixfizz Marketing / Campaign Manager / Shopper Configure / kiosk staff / Shopify Style / Launch Plan / Blog / Shopper tools and upgrades?"
→ **71_MYPIXFIZZ_FEATURES_ROUTES.md** § Marketing, § Shopper Configure, § Shopify Style, § Launch Plan, § Tools; tables in **72_MYPIXFIZZ_DATA_MODEL.md** § Tables Added September to October 2026

### "myPixfizz support: urgent ticket / order error form / awaiting customer auto-resolve?"
→ **71_MYPIXFIZZ_FEATURES_ROUTES.md** § Support

### "Safe changes to myPixfizz / migrations hit live / publishing ships the whole queue / organization add-ons?"
→ **70_MYPIXFIZZ_OVERVIEW.md** § Operational Rules for Changing myPixfizz, § Organization Add-ons

### "Spine insertion into a cover / design import (__print_theme.yml) / linked layouts / edit page XML in place?"
→ **19_XML_TEMPLATE_REFERENCE.md** § Growing Spine, § Design Import, § Linked Layouts drive the layout picker, § Editing Design Pages and Layouts in Place

### "shrink / valign / fontsize unit / crop=\"false\" / edit=\"false\" / lock a shape?"
→ **19_XML_TEMPLATE_REFERENCE.md** § Text Elements, § Image elements, § Element Permission Flags; **17_DESIGN_TOOL.md** § Locking an Element

### "Logo cropped / image_crop_flag / substitution tags / target element 255 characters / color option?"
→ **17_DESIGN_TOOL.md** § Element Substitution Types and § Image crop flag; **22_OPTION_VARIANT_RENDERING.md** § Template Option Substitutions

### "Editor Custom CSS field / editor.css not loading / editor CSS variables / unedited placeholders gate?"
→ **17_DESIGN_TOOL.md** § Editor CSS Customization (corrected 2026-10-06)

### "Drive the editor from a script / autofill / iframe editor / images missing from the editor tray?"
→ **17_DESIGN_TOOL.md** § Driving the Editor From a Script; **61_PIXFIZZ_API.md** § 6 (`/upload/image?gallery_id=`)

### "Upload image endpoint / project gallery / user by external id / delete a gallery?"
→ **61_PIXFIZZ_API.md** § 4, § 6 Add an image (corrected), § 7 Look up a user by external ID

### "Custom type instance PUT or DELETE / asset replace or delete / child snippet override by API / cms_snippets write?"
→ **61_PIXFIZZ_API.md** § 13c Custom types, § 13c Assets, § 13f CMS snippets

### "Admin form writes / collection image, order or custom fields / variant create / add_themes / design custom fields?"
→ **61_PIXFIZZ_API.md** § 13h Admin Form Writes From a Browser Session

### "Template import routes / export_all / template provider and consumers / snippet edit page DELETE trap?"
→ **18_ADMIN_NAVIGATION.md** § Bulk Update Tools (Import and publish routes) and § Template Provider and Template Consumers

### "Partial template import / -1 after a failed import / Error Query / font remap / Price isn't valid at quantity 1?"
→ **16_PRODUCT_HIERARCHY.md** § Import Behavior; **40_PLAYBOOK_UPDATED.md** § Error saving: Price isn't valid

### "Imported option only shows in kiosk mode / string false from another site's export?"
→ **22_OPTION_VARIANT_RENDERING.md** § 3.1 (amended 2026-10-06)

### "Variant type export flags (hidden / read_only / hide_from_cart) / child variant types / hide_value_labels?"
→ **22_OPTION_VARIANT_RENDERING.md** § Variant Type Exports (corrected 2026-10-06) and § 4.9

### "Custom fields on variants / owner_type / false custom values stored as strings / multitext by API?"
→ **51_CUSTOM_FIELDS_REFERENCE.md** Key Notes (amended 2026-10-06)

### "Cut prints: variant value price per copy?"
→ **30_PRICING_ENGINE.md** § Cut prints: a variant value price is not per copy unless the formula says so

### "manage/* radio values / checklist token vs label / wrong value stored?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 5; **01_CODE_GOVERNANCE_UPDATED.md** § A checklist value is the exact token, never the label

### "Dark child site cart and checkout / form-control colors / ship.svg?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 4 and § 18.2. Checkout is high risk: double-check every change.

### "show_prices on size tiles / size tile order / homepage delivery rules / services/ 404 / Liquid inside page_content?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 21.1, § 13, § 14; tile order follows collection order (**61_PIXFIZZ_API.md** § 13h)

### "Klaviyo signup consent / free shipping bar / promotions fly-out / blog section?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 20.1, § 12; **20_SHOPPER_CART_RULES.md**; **81_SEO_AND_GEO_REFERENCE.md** Part G

### "Liquid json filter / escape_json / website.* list cut off at 20 / snippet variable scope?"
→ **50_LIQUID_REFERENCE.md** (quirks, § Paginate)

### "Kiosk associate tip / Pay in Store limit / carts per domain?"
→ **21_SHOPPER_CHECKOUT_POLICY.md** (Kiosk associate tip; Pay in Store limit)

### "Cart custom field missing on the order / Order custom field definition / add_print_product cart links?"
→ **20_SHOPPER_CART_RULES.md**; **80_ONBOARDING.md** § Create Order custom field definitions for every cart field

### "Email templates not inherited / email kit / cart reminders / SendGrid pxemail domain?"
→ **32_ORDER_LIFECYCLE.md**; **80_ONBOARDING.md** § Email Notification Templates and § Phase 3

### "Launch check: empty collections, zero-price products orderable / migrating a catalog from another platform / what not to ask a lab?"
→ **80_ONBOARDING.md** § Launch Check: Empty Collections and Unpriced Products, § Phase 2, § Questions to put to the customer

### "Shopify My Projects click does nothing / gallery create error?"
→ **60_SHOPIFY_INTEGRATION.md** § 15 Troubleshooting My Projects and My Galleries

### "Asset or snippet YAML import fails / folded long scalars / line_width?"
→ **01_CODE_GOVERNANCE_UPDATED.md** § CMS Backup Tar Packaging Rule (Asset and snippet YAML must parse; Long plain scalars fold)

### "Number variant price formula / multiplied by quantity / decimal step blocks the form / area tiers?"
→ **30_PRICING_ENGINE.md** § Number Variant Price Formulas; import default: **22_OPTION_VARIANT_RENDERING.md** § Variant Type Exports

### "Write variant prices by API / unit_intervals or min and max units by API?"
→ **61_PIXFIZZ_API.md** § 13g Variant writes and Quantity limits

### "Add to cart from a script / editor=add-to-cart / one tool action, several cart lines / book[pages] on create?"
→ **61_PIXFIZZ_API.md** § 4 Create a project and Add to the cart from a script

### "Project preview of a given page / unsaved options in a project preview / px-project-preview page-number?"
→ **61_PIXFIZZ_API.md** § 9 Project previews

### "SiteFlow status callback / siteflow_update / print-on-demand callbacks across accounts?"
→ **61_PIXFIZZ_API.md** § 13 SiteFlow status callbacks

### "Snippet content will not save from a script / override a parent snippet from the admin form / admin login but storefront says not admin?"
→ **61_PIXFIZZ_API.md** § 13h Snippet content and § 2 Admin UI host vs API host; **50_SHOPPER_TEMPLATE_REFERENCE.md** § 17

### "Editor crop zoom, left, top / text labels on the editor toolbar / tag-only substitutions?"
→ **17_DESIGN_TOOL.md** § Image crop state, § Editor Buttons and Per-Site CSS, § Substitutions bind by element name

### "Custom tool label layout broken / options written into the URL by a tool / tool vanishes after closing a modal / pxt-step collapses?"
→ **17_DESIGN_TOOL.md** § A tool mounted by `custom_script` lives inside the variant selector; **26_CUSTOM_DESIGN_TOOLS.md** § The shared look

### "Replace /site/photo-prints on a child / photo-prints-landing?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 14 Creating Pages on Child Sites

### "Reorder of a custom tool line has no artwork / tool line shows the catalog photo in emails / cut print sideways in the cart?"
→ **20_SHOPPER_CART_RULES.md** § Reorder, order details and emails for a custom tool line, § Cut prints with autorotate

### "One cart line, many supplier items in an HTTP fulfillment?"
→ **31_FULFILLMENT_ENGINE.md** § Custom Tool Fulfillment Standard (HTTP push payload)

### "Copy a design to another site / design export loses linked layouts / copy design options between sites?"
→ **18_ADMIN_NAVIGATION.md** § Import and publish routes; **22_OPTION_VARIANT_RENDERING.md** § Template Option Substitutions

### "Design preview cards blank in a collection grid / Live Finish wall view too big / MSP template mount reaches other sites?"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 6 Known defects, § 7 Installing on a site

### "myPixfizz retention rule deleted data / procedure cannot commit / is this table unused?"
→ **70_MYPIXFIZZ_OVERVIEW.md** § Operational Rules for Changing myPixfizz

## Routes added 2026-10-17

### "GA4 bridge, gtag-only sites, ga4-server-side-purchase, pf_ga4_debug"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 20 GA4 bridge for gtag-only sites

### "modalLogin, modal target ids, sign-in link does nothing"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 1 The sign-in modal on every page

### "page width, header frame, alignment check, wrapper max-width"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 14 Page content uses the header's frame

### "widen header, catalog homepage nav, navbar container override"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 2 When a child page needs a wider frame

### "megamenu hover not opening, data-toggle dropdown"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 3 How megamenus work

### "Services v2, services hub, svc-page, services-hub-v2"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 14 Services v2

### "admin_only Pages, park a page, hidden page still renders"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 14 Constraints

### "favicon inherited default, sitewide fonts, head tags on child, custom-links"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 9 HTML Head

### "payment heading no methods, CRLF split, captured newline, production time 1 day wrong"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 17

### "collection_filters empty sidebar, LF line endings, only first filter parses"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 21.1 `collection_filters` and line endings

### "Px.CMS.ThemeFilters, change two filters, pushFilterChange"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 21.1 Driving filters from script

### "url_path redirect, clean URL 404, theme parameter required"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 24

### "nested snippet clears params, snippet scope collection"
→ **50_LIQUID_REFERENCE.md** § snippet

### "redirect query string, fbclid 404, redirect regex"
→ **50_LIQUID_REFERENCE.md** § Website Redirects (config shape); 80_ONBOARDING.md § SEO Migration: Sitemap and 301 Redirects

### "asset_url child asset override, replace parent logo"
→ **50_LIQUID_REFERENCE.md** § asset_url

### "pickup phone required, autofill pickup phone"
→ **21_SHOPPER_CHECKOUT_POLICY.md** § Pickup contact phone

### "rush radio loses choice, checkout-rush-style"
→ **21_SHOPPER_CHECKOUT_POLICY.md** § Pay in Store maximum cart total (rush notes)

### "static product no cart thumbnail"
→ **20_SHOPPER_CART_RULES.md** § A static product with no image shows no thumbnail anywhere

### "text variant not in cart, hide_edit_in_cart"
→ **20_SHOPPER_CART_RULES.md** § Hiding options and variants from the cart line

### "Google product feed, feed/products.xml, Merchant Center gaps"
→ **81_SEO_AND_GEO_REFERENCE.md** § Trust and local

### "duplicate titles theme URLs, canonical utm"
→ **81_SEO_AND_GEO_REFERENCE.md** § Part G

### "icon glyph off center iPhone"
→ **82_MOBILE_UX_REFERENCE.md** § 4a

### "template versions page diff, current template XML"
→ **18_ADMIN_NAVIGATION.md** § Products

### "admin DELETE form same action URL, bulk template option copy design options ignored"
→ **18_ADMIN_NAVIGATION.md** § Bulk Update Tools: Rolling a Change Across Live Templates

### "publish designs to collection, add_themes.json 404, manage_products wizard route"
→ **18_ADMIN_NAVIGATION.md** § Import and publish routes (admin tab, logged in); 61_PIXFIZZ_API.md § 13h Collections

### "consumer publishes provider design, provider preview image on consumer, provider price variables, template categories"
→ **18_ADMIN_NAVIGATION.md** § Template Provider and Template Consumers

### "2FA enforced, super admin reset 2FA"
→ **18_ADMIN_NAVIGATION.md** § Admin Login: Two-Factor and Passkeys

### "fulfillment template route super admin"
→ **18_ADMIN_NAVIGATION.md** § Customer Super User Access

### "unpublished design skipped on import, Price isn't valid import, fulfillment_setting_id, option inside template export"
→ **16_PRODUCT_HIERARCHY.md** § Import Behavior: Ids in an Import Tar Are Not Honored

### "template says no options but designs have options"
→ **16_PRODUCT_HIERARCHY.md** § Variants vs Options

### "unpublish design from one collection, publish_theme"
→ **16_PRODUCT_HIERARCHY.md** § Unpublishing Does Not Make a Product Unreachable

### "foil layer empty, ipage separate-file, pdf_layer, spot color, PMS, white ink, RGB only"
→ **19_XML_TEMPLATE_REFERENCE.md** § PDF Layers in Practice

### "print PDF downsampling dpi"
→ **19_XML_TEMPLATE_REFERENCE.md** § `<definition>` Attributes; 31_FULFILLMENT_ENGINE.md § 1) Asset generation (files)

### "kerning unit, Unknown type of color, unsupported characters"
→ **19_XML_TEMPLATE_REFERENCE.md** § Text Elements: `shrink`, `valign` and Font Size

### "layflat page count spreads"
→ **19_XML_TEMPLATE_REFERENCE.md** § Multi-Page Product Page-Count Rules

### "layout name length 32 import 500"
→ **19_XML_TEMPLATE_REFERENCE.md** § What an import does

### "folded top-fold card, shaped photo window"
→ **19_XML_TEMPLATE_REFERENCE.md** § Greeting Card

### "closed book preview spine"
→ **19_XML_TEMPLATE_REFERENCE.md** § How the spine is inserted into a cover at render

### "calendar element tags idx fill, calendar_transformations, website-calendar captions"
→ **23_XML_CALENDAR_REFERENCE.md** § Calendar Element Tags, Fills and Transformations

### "weekday defdate dateshift grid start"
→ **23_XML_CALENDAR_REFERENCE.md** § 4. Dates Vocabulary (`<defdate>`)

### "img src empty fires onerror"
→ **17_DESIGN_TOOL.md** § Custom Design Tools — Browser-Side Rules

### "collection_footer product page full width"
→ **51_CUSTOM_FIELDS_REFERENCE.md** § Collection

### "kiosk_id order_source kiosk-N, location_id, uploaded file URL on orderline"
→ **32_ORDER_LIFECYCLE.md** § Kiosk Terminal Tracking; § What scripts can do

### "production file without an order, design page stale after attribute save, auto-submit fires twice"
→ **40_PLAYBOOK_UPDATED.md** § Production File Regeneration; § The Editor Returns "Not Found" on One Variant and Works on Another; § `style onload` Firing Once Then Stopping

### "lorem ipsum terms page, placeholder phone, blank 200 subdomain, static product image check"
→ **80_ONBOARDING.md** § Content Completeness Before Launch; § Launch Check: Empty Collections and Unpriced Products

### "widen image outpaint, ping-pong reveal loop"
→ **83_AI_IMAGERY_PRODUCTION.md** § B8; § B11

### "px_rates, price card, pricing.js, PxPricing, OrderHub pricing engine"
→ **26_CUSTOM_DESIGN_TOOLS.md** § When the tool computes the price

### "tool asset naming, asset overwrite"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 3. Anatomy of a tool

### "px-upload-dialog button classes, unstyled dialog, preview 250 px"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Open the platform upload dialog standalone

### "px-image-adjust-tool, standalone crop"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Using the platform crop tool standalone

### "tool dark mode tokens, :where reset, design tool wording"
→ **26_CUSTOM_DESIGN_TOOLS.md** § The shared look: `style/px-tool-theme`

### "px_mount, filter-switched sibling products, one template one mount"
→ **26_CUSTOM_DESIGN_TOOLS.md** § The mount gets its own hidden option; § Rules for a mount block

### "project edit page no price, tool reopen saved state"
→ **26_CUSTOM_DESIGN_TOOLS.md** § The project edit page has no price

### "canvas size limit, long gang sheet dpi loss"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Gang Up: live

### "Live Finish ornament-only bases, room plate 4:5, supplier spec before modelling"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 2.3; § 2.4

### "View in 3D fallback button, wrong p3d_page, preserve-3d hit testing, brushed metal"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 3.2; § Engineering lessons

### "copied mount site ids, lf_backdrop, lf_material_filter after import"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 7. Installing on a site

### "hide_design_options "false", no design options render"
→ **22_OPTION_VARIANT_RENDERING.md** § 3.1 Kiosk-only options

### "file_upload accept, px-file-upload required, project-edit form"
→ **22_OPTION_VARIANT_RENDERING.md** § 5.2 `file_upload`; § 5.2c

### "finish x size price matrix, combined variant codes"
→ **22_OPTION_VARIANT_RENDERING.md** § A Non-Additive Finish x Size Price Matrix Is One Variant With Combined Codes

### "price_forecast pages parameter, wrong option code, round(2), round total to .95"
→ **30_PRICING_ENGINE.md** § Platform Price per Quantity and Cart Rounding; § Canonical patterns

### "shipping rate by product category"
→ **30_PRICING_ENGINE.md** § Shipping Rate by Cart Content Inside One Shipping Service

### "global fulfillment code, website default fulfillment, barcode counter"
→ **31_FULFILLMENT_ENGINE.md** § Fulfillment Code Resolution and Precedence

### "job ticket option codes per template, stale print file after project edit"
→ **31_FULFILLMENT_ENGINE.md** § `options[]`; § Custom Tool Print Files

### "admin users list paging, user addresses, telephone-only PUT, users per page"
→ **61_PIXFIZZ_API.md** § 3 List all users (admin)

### "user calendar create, calendar dates recurring, book[calendars][], user-calendar captions"
→ **61_PIXFIZZ_API.md** § 11 Calendars

### "projects cannot be deleted, test projects"
→ **61_PIXFIZZ_API.md** § 4 Update a project

### "custom field size limit, "are too large" 422, page_content too large"
→ **61_PIXFIZZ_API.md** § 13c Custom types

### "custom field definition create by script, custom_field_definitions admin form"
→ **61_PIXFIZZ_API.md** § 13c Custom types

### "asset upload 413, large PDF asset, ?sitename ignored on storefront origin, /fz/ signed URL host"
→ **61_PIXFIZZ_API.md** § 13c Assets

### "API key add product to collection, add_products with API key"
→ **61_PIXFIZZ_API.md** § 13c Updating custom fields on existing objects

### "file upload option db: id, uploaded file URL, webhook production files, order_source, created_at time zone"
→ **61_PIXFIZZ_API.md** § 13d Order Webhook

### "theme_categories read, sub_categories, find designs from storefront, subcollection name, signed-out price_forecast"
→ **61_PIXFIZZ_API.md** § 13g Public catalog reads (no login)

### "create design product by API, print_product_id create"
→ **61_PIXFIZZ_API.md** § 13g Create

### "admin CSP connect-src, admin page fetch blocked"
→ **61_PIXFIZZ_API.md** § 13h General rules

### "extra fee create by script"
→ **61_PIXFIZZ_API.md** § 13h Extra Fees

### "photo-prints add to cart hangs 0%, getShopifyVariantId, sku map mismatch"
→ **60_SHOPIFY_INTEGRATION.md** § 11 Troubleshooting Guide

### "photo-prints hide variant picker, alternate product template ?view="
→ **60_SHOPIFY_INTEGRATION.md** § 6 Product Page Template

### "master snippet fix not on override, px-option required variant"
→ **60_SHOPIFY_INTEGRATION.md** § 1 Master Shopify CMS

### "self-chaining edge function, EdgeRuntime.waitUntil, chained sync stops silently"
→ **70_MYPIXFIZZ_OVERVIEW.md** § Operational Rules for Changing myPixfizz

### "POD catalog sync, site_subdomain, hand-loaded POD rows disappear"
→ **71_MYPIXFIZZ_FEATURES_ROUTES.md** § Brand Connections and Catalog

### "ga4_event_outbox raw_payload, live sales reporting from webhook"
→ **72_MYPIXFIZZ_DATA_MODEL.md** § ga4_event_outbox

### "own S3 bucket CORS, uploads fail, trailing slash origin"
→ **45_ORDERHUB.md** § Own Amazon S3 Storage: No Trailing Slash in a CORS Origin

### "dynamic block variables empty, snippet params inside dynamic"
→ **01_CODE_GOVERNANCE_UPDATED.md** § Scoping Rule — Keep `{% dynamic %}` Blocks Narrow

## Routes added 2026-10-22

### "Publish Products wizard, print_themes positions, 0-based, dot list, publish publishes nothing, print_theme_ids ignored"
→ **18_ADMIN_NAVIGATION.md** § Import and publish routes (admin tab, logged in)

### "publish link after template import, product to collection, Collections step, Create and Tick, create_theme_category, new collection in wizard"
→ **18_ADMIN_NAVIGATION.md** § Import and publish routes (admin tab, logged in)

### "Price isn't valid, publish wizard price prefill, consumer price variable missing"
→ **18_ADMIN_NAVIGATION.md** § Import and publish routes (admin tab, logged in)

### "Import Template button, import_print_product Application error, template soft delete, This template has been deleted"
→ **18_ADMIN_NAVIGATION.md** § Products / Templates

### "product edit 500, products/<id>/edit, Import Variants, Export All variants"
→ **18_ADMIN_NAVIGATION.md** § Products / Product Attributes

### "create collection, no create button, hide sub-collection, unpublished collection, which collections hold a product"
→ **18_ADMIN_NAVIGATION.md** § Products / Collections

### "template option edit forms, custom_script save, unpublish option, options JSON, options import token"
→ **18_ADMIN_NAVIGATION.md** § Bulk Update Tools: Rolling a Change Across Live Templates

### "add template consumer, Print on Demand tab, POD tab sources, pxtemplates, Pixfizz-built templates, source_website_id"
→ **18_ADMIN_NAVIGATION.md** § Template Provider and Template Consumers

### "New User form fields, set user category, Log in as user, admin roles, restricted admin, projects only admin"
→ **11_USER_IDENTITY_MODEL.md** § Admin / Super Admin

### "POD publish no variants, POD product no custom fields, supplier code -2, OLD- code rename, swap local copies"
→ **16_PRODUCT_HIERARCHY.md** § Publishing Another Site's Template: What the Lab Product Gets

### "Psych BadAlias, Unknown alias, zone anchor, TimeWithZone, folded YAML, edit template export, linked_assets in export"
→ **16_PRODUCT_HIERARCHY.md** § Template Export and Import Format

### "import 500 variant type keys, element_substitutions translations children, image null, preview_img null, duplicate url_path after import"
→ **16_PRODUCT_HIERARCHY.md** § Import Behavior: Ids in an Import Tar Are Not Honored

### "fulfillment code disabled on product, POD fulfillment prefill, QA destination demo site"
→ **31_FULFILLMENT_ENGINE.md** § Fulfillment Code Resolution and Precedence

### "auto_confirm_orders, test order reached print partner, no Pending window"
→ **32_ORDER_LIFECYCLE.md** § Order Lifecycle Statuses / `auto_confirm_orders`: no Pending window, so check it before any test order

### "Generate new file refused, cannot generate until existing deleted, re-render project"
→ **32_ORDER_LIFECYCLE.md** § Other Admin Order Sections

### "does the child override this snippet, edit_by_name"
→ **13_TEMPLATE_BOUNDARIES.md** § A Child Site's Snippet List Shows Only Its Overrides

### "saved project ordered false, did the design convert, source_book_id, saved vs ordered book"
→ **61_PIXFIZZ_API.md** § 4. Projects (Books) / A saved project and its ordered project are two books

### "list projects across users, unfinished designs, abandoned designs, admin books list"
→ **61_PIXFIZZ_API.md** § 4. Projects (Books) / List user's projects

### "preview request changes updated_at, preview bumps activity date"
→ **61_PIXFIZZ_API.md** § 4. Projects (Books) / Preview a project (JPEG, free)

### "upload done never fires, same file re-uploaded hangs, file-url unchanged, content addressed upload"
→ **61_PIXFIZZ_API.md** § 6. Galleries and Images / File uploads are content addressed

### "hide_placeholders, placeholder text missing in preview, preview.webp 150 px, blank design tile, empty placeholder grey"
→ **61_PIXFIZZ_API.md** § 9. Dynamic Design Previews / Query parameters

### "layouts.json, design layouts list, auto-fill needs layouts, photo count tags"
→ **61_PIXFIZZ_API.md** § 9. Dynamic Design Previews / A design's layouts

### "print render 500, stale mask id, fulfillment render missing placeholder text, check every page without an order"
→ **61_PIXFIZZ_API.md** § 9. Dynamic Design Previews / Preview resolution and production-quality output

### "glb has incorrect filetype, json asset, import js asset from cdn, C2PA chunk, asset file size differs"
→ **61_PIXFIZZ_API.md** § 13c. Admin Content API / Assets

### "products.json exposes custom fields, formula readable publicly, secrets in product custom fields"
→ **61_PIXFIZZ_API.md** § 13g. Admin Products API / Public catalog reads (no login)

### "admin page cannot fetch site host, eval blocked admin, strict-dynamic nonce, inject script admin tab"
→ **61_PIXFIZZ_API.md** § 13h. Admin Form Writes From a Browser Session / General rules; § Admin UI host vs API host: never interchangeable

### "publish wizard publishes nothing, print_themes positions, print_theme_ids ignored, Create and Tick collection, hide sub-collection unpublished"
→ **61_PIXFIZZ_API.md** § 13h. Admin Form Writes From a Browser Session / Collections

### "three ways a product is priced, px_rates mode area, px_width_in px_price codes, OrderHub cannot evaluate Ruby"
→ **30_PRICING_ENGINE.md** § The Three Ways a Product Is Priced

### "price_forecast missing variant price, default variant not included, forecast lower than cart, spreads_as_pages pages parameter"
→ **30_PRICING_ENGINE.md** § Platform Price per Quantity and Cart Rounding

### "flat setup fee on a variant charged per piece, rounded corners per order, amount divided by quantity"
→ **30_PRICING_ENGINE.md** § Per-Order Charges Must Never Sit on a Variant / A flat charge per order line on a variant value is `amount / quantity`

### "weight based shipping formula, dimensional weight, packing class by product code"
→ **30_PRICING_ENGINE.md** § Shipping Rate by Cart Content Inside One Shipping Service / A per-cart weight estimate by product code

### "text font size locked, efontsize, ecolor, customer changes text size, hide size control"
→ **19_XML_TEMPLATE_REFERENCE.md** § Element Permission Flags; **17_DESIGN_TOOL.md** § Locking an Element

### "QR code button, add-qrcode, hideAddQrCodeBoxButton, hide QR button in editor"
→ **17_DESIGN_TOOL.md** § Editor Buttons and Per-Site CSS

### "editor script text font, pointsize, fontsize MobX error, fillPlaceholder, setLayout, removeElement"
→ **17_DESIGN_TOOL.md** § Text, image and layout calls

### "editor add_to_cart tracking, px-cart-button, editor GTM, dataLayer empty in editor"
→ **17_DESIGN_TOOL.md** § Tracking hooks; § Integrations

### "autofill throws, image factory error, project unsaveable, db: id"
→ **17_DESIGN_TOOL.md** § Autofill in a chosen order

### "image border substitution width, border half inside, image_border_width"
→ **17_DESIGN_TOOL.md** § Image border substitutions

### "negative z, element covers photo, g group, score element, show_on_preview, cmyk color in XML"
→ **19_XML_TEMPLATE_REFERENCE.md** § Drawing Order, Groups and Other Elements Seen in Design Exports

### "display text prints lower than prototype, Oswald offset, Playfair offset"
→ **19_XML_TEMPLATE_REFERENCE.md** § Text Elements: `shrink`, `valign` and Font Size

### "calendar layout prints Month placeholder, cal tags in layout not resolved, per-month photo layouts"
→ **23_XML_CALENDAR_REFERENCE.md** § Per-Page Photo Layouts on Calendar Templates

### "calendar week start Monday Sunday switch, twin template"
→ **23_XML_CALENDAR_REFERENCE.md** § Week start is in the date rule, not a setting

### "calendar Colors option, set-color overrides color option, render order"
→ **23_XML_CALENDAR_REFERENCE.md** § Calendar Element Tags, Fills and Transformations / Order of operations at render

### "uploaded photo not in preview, image upload option shows empty design, target element name"
→ **22_OPTION_VARIANT_RENDERING.md** § Template Option Substitutions: Target Elements, Types and Limits

### "copy variants to another site, POD product has no variants, variant_types export_all import"
→ **22_OPTION_VARIANT_RENDERING.md** § Variant Type Exports

### "upload dialog must, tool file picker, px-upload-dialog sources, only This Device shows, a tool opens calm"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Open the platform upload dialog standalone

### "upload dialog grey buttons, unstyled This Device button, button-class after appendChild, local-source-button-class"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Open the platform upload dialog standalone

### "SUPPORTED_FILE_TYPES, upload dialog accept, PDF in upload dialog, Upload Photos wording on a print tool"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Open the platform upload dialog standalone

### "file-url unchanged, upload hangs on same file, froze in review, content addressed upload, edit from cart restore"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Attaching a file to an option: `file-url` is content addressed

### "site custom.css breaks tool, white text on white button, img hover zoom, generic class leak, contrast check"
→ **26_CUSTOM_DESIGN_TOOLS.md** § The shared look: `style/px-tool-theme`

### "plain product page flashes before tool, owns-page flash, :has takeover"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Page layout: an inline tool owns its page

### "tool stuck on Adding to cart, step number input blocks submit, trying to be set with invalid value, retired option"
→ **26_CUSTOM_DESIGN_TOOLS.md** § The product form refuses silently

### "design picker tiles blank, preview.webp blank, layouts.json, template nil in mount, template.width empty"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Tools that show a design gallery

### "SVG canvas wrong font, fillText document.fonts, preview from page XML, cmyk colors in design XML, pdf.js worker"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Drawing a design in the browser

### "prototype of an existing tool, standalone mock rejected, preview corner standard"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Design first: a prototype of an existing tool starts from the live tool

### "tool harness passes but live misaligned, Bootstrap label margin, test page host CSS"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Testing a tool before the live check

### "artwork_top seam, artwork_base, card art layers, motif keying, pattern scale landscape, card back rules"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Generated template art

### "custom_script prints as raw text, TemplateOptionType definitions, tool kit on older child"
→ **26_CUSTOM_DESIGN_TOOLS.md** § 5. Install order on a new site

### "spreads_as_pages pricing, printed pages vs system pages, price_forecast pages"
→ **26_CUSTOM_DESIGN_TOOLS.md** § Price ladders read the platform price

### "px_rates mode area, priced by size, px_width_in, counter-only product OrderHub"
→ **26_CUSTOM_DESIGN_TOOLS.md** § When the tool computes the price

### "tool config on a Pages page, page_content JSON config, parent asset in child page_content"
→ **26_CUSTOM_DESIGN_TOOLS.md** § A tool mounted on a Pages page

### "Live Finish Plates, green box plate, one plate many sizes, object on the print, cylinder mapping, plate weight"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 2.10 Photo plates (Live Finish Plates)

### "room scene scale wrong, wall plane calibration, piece too big in room, keep-out, outpaint taller wall"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 2.11 Room scenes at true scale

### "lf_mats @flat, printed border, lf_wrap_color_variant, color wrap design option"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 2.2 Mount keys

### "GLB preview in a tool, gltfpack, bpy headless, never replace approved preview"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 3.3 Models are built in Blender

### "CDN image WebGL texture CORS, tainted canvas, TextureLoader fails, import asset_url ES module"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 4. Platform facts both engines rely on

### "Nano Banana product plate, print design on product mockup, paper stack too thick, bevel edges, exact design gibberish"
→ **83_AI_IMAGERY_PRODUCTION.md** § B12. Product plates and preview scenes with the Nano Banana family

### "cart remove confirm, px-rm-line, remove-line-confirm, child cart override instant remove"
→ **20_SHOPPER_CART_RULES.md** § Removing a cart line asks for confirmation (parent, 2026-10-22)

### "approved users only checkout, trade catalog site, disable-user-registration does not block, user.category gate"
→ **21_SHOPPER_CHECKOUT_POLICY.md** § Restricting checkout to approved users on a child site

### "multi-store pickup wrong store, first public address, cart[location_id], checkout auto-click race"
→ **21_SHOPPER_CHECKOUT_POLICY.md** § Multi-store pickup: the checkout posts the first public address

### "guest checkout no email, guest phone not required, film order no guest checkout"
→ **21_SHOPPER_CHECKOUT_POLICY.md** § Guest checkout: configurable fields

### "Stripe payment form light on dark site, Stripe appearance night, dark labels card field"
→ **21_SHOPPER_CHECKOUT_POLICY.md** § Payment Gateway Notes

### "grayscale filter on product preview does nothing, px-design-preview display contents, request.path in custom.css"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 17. Known Gotchas / CSS filters on the product preview must target the parent of `px-design-preview`

### "label pill ignores padding, label display contents, page content label styling"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 17. Known Gotchas / A theme stylesheet gives `<label>` `display: contents` in page content

### "CSS radio filter jumps to top, page scrolls on label click, hidden radio focus scroll"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 17. Known Gotchas / CSS-only filters built from radios jump the page to the top

### "order email prints every option, empty print file rows, email kit filters options"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 11. Email Templates

### "read Pages instance back, custom_type_instances export, page_content hash before edit"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 14. Creating Pages on Child Sites

### "design product no image, imported designs no preview, preview_img and Use as preview"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 24. Product URLs, Titles, Shop Tiles and Card Prices for Design Products

### "studio product URL 404, url_path blank, unpublished design redirect 404"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 24. Product URLs, Titles, Shop Tiles and Card Prices for Design Products

### "collection_filters only first line after parent fix, LF config, CRLF re-post"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 21.1 `collection_filters` has two syntaxes / `collection_filters` and line endings

### "editor-scripts.js fbq is not defined, design tool add_to_cart tracking, ga4-server-side-purchase GTM site double purchase"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 20. Analytics

### "asset_url parent fallback, parent asset on child site, image missing src empty, asset not uploaded"
→ **50_LIQUID_REFERENCE.md** § asset_url

### "apostrophe in Liquid string, Season's Greetings, typographic apostrophe"
→ **50_LIQUID_REFERENCE.md** § String Quoting Rules

### "custom_script prints raw Liquid, tool mount shows code as text, TemplateOptionType definitions"
→ **51_CUSTOM_FIELDS_REFERENCE.md** § Key Notes

### "products.json custom fields public, secrets in custom fields, pricing formula visible"
→ **51_CUSTOM_FIELDS_REFERENCE.md** § Key Notes

### "Stripe form dark site, card form unreadable, Payment Element appearance"
→ **40_PLAYBOOK_UPDATED.md** § Stripe: Payment Form Unreadable on a Dark Site

### "page edits do nothing, layout without page.content, stale page copy"
→ **40_PLAYBOOK_UPDATED.md** § Edits to a Page Do Nothing: The Layout Has No `page.content`

### "custom.css leaks into component, invisible button text, zooms on hover, img:hover transform"
→ **40_PLAYBOOK_UPDATED.md** § A Component Looks Wrong on One Site Only: The Site's Own CSS Is Leaking In

### "test order on live site, auto_confirm_orders, order sent to printer immediately"
→ **40_PLAYBOOK_UPDATED.md** § Before Any Test Order on a Live Site, Check Whether Orders Auto-Confirm

### "scripted admin write deleted record, select admin form, FormData delete form"
→ **41_IMPLEMENTATION_PATTERNS_UPDATED.md** § Scripted Admin Writes: Pick the Form by a Field, Never by Its Action URL

### "GA4 duplicate purchase, server backfill 48 72 hours, ga4_server_purchases no_browser, blank hostName"
→ **85_GA4_SERVER_SIDE_PURCHASE.md** § 3. How the Webhook Function Decides / Browser first, server backfill

### "GA4 order source, transactionId sessionSourceMedium, GA4 revenue lower than Pixfizz"
→ **85_GA4_SERVER_SIDE_PURCHASE.md** § 10. Order Source and Revenue: Joining Pixfizz Orders to GA4

### "POD catalog wrong category, SKIP_COLLECTION, category edit does not stick"
→ **71_MYPIXFIZZ_FEATURES_ROUTES.md** § Brand Connections and Catalog

### "myPixfizz images, binary files in public, og:image share preview"
→ **70_MYPIXFIZZ_OVERVIEW.md** § Images and other binary files

### "OrderHub reprice px_ variants, counter-only product px_rates"
→ **45_ORDERHUB.md** § Repricing Custom Tool Lines from Generic `px_` Variant Values

### "AI upscaling at fulfillment, Perfectly Clear upscale, low resolution image enlarged, output-perfectly-clear-upscale"
→ **31_FULFILLMENT_ENGINE.md** § AI Upscaling at Fulfillment (Perfectly Clear)

### "output-perfectly-clear-upscale-dpi, upscale per template"
→ **19_XML_TEMPLATE_REFERENCE.md** § `<definition>` Attributes

### "myPixfizz SSO, login.pixfizz.com, sso/start site parameter, same email on two sites"
→ **70_MYPIXFIZZ_OVERVIEW.md** § Single sign-on from the Pixfizz admin

### "webhook deactivated, 20 consecutive failures, WebhookDeactivated email, webhook stopped firing"
→ **61_PIXFIZZ_API.md** § 13d. Order Webhook

### "merge users imsure, merge warning, merge 403, merge direction"
→ **61_PIXFIZZ_API.md** § User Handoff

### "orderline draft, is_draft, (Draft) label, draft boolean"
→ **61_PIXFIZZ_API.md** § Draft orderlines

### "Shopify order status P, order.created always P, listen to order.status_changed"
→ **60_SHOPIFY_INTEGRATION.md** § 9. Order Sync

### "Shopify order missing in Pixfizz, duplicate users, external_source, duplicate and confirm in Shopify"
→ **60_SHOPIFY_INTEGRATION.md** § Order missing in Pixfizz: duplicate users

### "Shopify add to cart empty cart, shopify_product_id missing, shopify_variant_id missing"
→ **60_SHOPIFY_INTEGRATION.md** § Add to Cart lands on an empty Shopify cart

### "fulfillment folder space, illegal characters in filename, transfer stalled, replace space with underscore"
→ **31_FULFILLMENT_ENGINE.md** § Filename Templates (default vs customized)

### "capture website, website.hostname empty in email, global variable redefined"
→ **50_LIQUID_REFERENCE.md** § KNOWN CMS LIQUID QUIRKS

### "collection filter matches nothing, portrait landscape vs vertical horizontal, filter values must match"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 21.1 `collection_filters` has two syntaxes

### "Authorize.net AVS, address verification rejects card"
→ **21_SHOPPER_CHECKOUT_POLICY.md** § Payment Gateway Notes

### "enable Etsy, Etsy integration per site"
→ **80_ONBOARDING.md** § Marketplace (Etsy, etc.)

### "sitemap stale, product feed not updated, sitemap cache one minute"
→ **81_SEO_AND_GEO_REFERENCE.md** § Crawlable

### "canvas format px-preview, experimental canvas renderer, preview JSON"
→ **27_LIVE_FINISH_AND_3D_PREVIEWS.md** § 4. Platform facts both engines rely on

### "photo prints image selection reset, quantity update, like Photo Finale"
→ **20_SHOPPER_CART_RULES.md** § Photo prints

### "manage products template dropdown, grouped by source website, show more categories, provider logo"
→ **18_ADMIN_NAVIGATION.md** § Template Provider and Template Consumers

### "order fulfillment page all codes, POD supplier callback any site"
→ **31_FULFILLMENT_ENGINE.md** § Fulfillment Code Resolution and Precedence

### "Darkroom hot folder on NAS, files not picked up, OHD output folder"
→ **45_ORDERHUB.md** § Print controller hot folder on a NAS: files picked up late

### "EasyPost tracking every six hours, 21 days tracking"
→ **45_ORDERHUB.md** § EasyPost Shipping Integration

### "spreads_as_pages label, printed pages vs system pages, page selector doubled"
→ **51_CUSTOM_FIELDS_REFERENCE.md** § Product (81 total)

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
- 2026-09-09: Added 85_GA4_SERVER_SIDE_PURCHASE.md and 40 routing entries for the 2026-09-09 sync — server-side GA4, the GTM-only standard, Google account ownership, the dead Add to Cart / stale page cache / lexicographic sort / formula validator diagnostics, the Extra Fee corrections, unpublished products staying orderable, archive re-import duplication, parent-safe snippet changes and the parent-first snippet rule, collection_filters, root-relative nav hrefs, the reparented modal, kiosk keys, print-on-demand zero-price routing, FTP retention, the _additional_files.json trailing comma, alphabetical export attribute order, the AI preview-scene narrowing, the RLS-in-triggers rule, and the silent-failure index. Every target section was written in the same sync. Source: kbsync.
- 2026-09-09: Route check run as the mandatory closing step now recorded in 01_CODE_GOVERNANCE_UPDATED.md. Every route resolves. Note for whoever runs the naive one-liner next: it also matches filenames quoted inside this file's own prose and changelog, so `01_CODE_GOVERNANCE.md`, `40_PLAYBOOK.md` and `83_MOBILE_UX_AUDIT.md` are reported as missing. All three are historical references, not live routes: the first two appear only in the 2026-08-29 changelog entries describing corrections already made, and the third only in the note above § 82 recording that it never existed. Excluding the changelog leaves one known false positive. Source: kbsync audit.
- 2026-09-16: Added routing for the experimental Admin API, the `/admin` endpoint retirement, site-wide custom field / custom type definition export, and upload dialog login. Source: notion-page (Dashboard).
- 2026-09-19: Added `26_CUSTOM_DESIGN_TOOLS.md` and 8 routing entries for it. Corrected the "tool does not appear" route, which pointed at a four-step install order that no longer exists. Source: kbsync (custom tool estate).
- 2026-09-24: Added 14 routes for the 2026-09-24 sync, led by the Install-Step Gate and PDF layers. Every target section was written in the same sync. Source: kbsync.
- 2026-09-24: Route check run as the mandatory closing step. Only the three known historical false positives are reported (01_CODE_GOVERNANCE.md, 40_PLAYBOOK.md, 83_MOBILE_UX_AUDIT.md). Source: kbsync audit.
- 2026-09-29: Added 9 routes for the 2026-09-29 sync (Bulk Update Tools, API keys, admin login, notification emails, OrderHub stock, template changes vs existing projects, element rotation, PDP swap and custom_script, myPixfizz connections). Every target section was written in the same sync. Source: kbsync.
- 2026-10-06: Added 40 routes for the 2026-10-06 sync, led by the new 27_LIVE_FINISH_AND_3D_PREVIEWS.md, the refreshed custom design tool registry, the rebuilt myPixfizz route map, and the Admin Form Writes API section. Source: kbsync.
- 2026-10-09: Added the route to `orderhub/`, the nightly mirror of the OrderHub help articles (72 at first sync), generated by `scripts/orderhub_kb_sync.py` from the OrderHub kb-export endpoint. Never edit files in `orderhub/` by hand: edit the article in OrderHub. Source: claude-chat.
- 2026-10-09: Added 14 routes for the 2026-10-09 sync (number variant formulas, experimental variant API writes, scripted add to cart, project previews, SiteFlow callbacks, snippet saves and admin sessions, editor crop and toolbar, tools inside the variant selector, photo-prints landing, tool lines on reorder and email, one line to many supplier items, design copy, preview defects and MSP mounts, myPixfizz retention). Every target section was written in the same sync. Source: kbsync.
- 2026-10-17: Added 84 routes for the 2026-10-17 sync (Shopper frame and header, GA4 bridge live, collection filters and line endings, redirects and query strings, publish routes and the add_themes.json 404, template provider and consumer, 2FA, XML foil and DPI and text attributes, calendar tags and transformations, px_rates and pricing.js, upload dialog and crop tool, fulfillment codes, Live Finish install ids, admin users and calendars API, POD catalog sync). Every target section was written in the same sync. Source: kbsync.
- 2026-10-22: Added 116 routes for the 2026-10-22 sync (publish wizard positions and Create and Tick, POD publish without variants, template export YAML and import failures, saved vs ordered books, content addressed uploads, preview parameters and layouts, the three ways a product is priced and `px_rates` area mode, editor text and QR controls, calendar layouts and week start, the upload dialog standard for tools, Live Finish plates and room scenes, cart remove confirm, approved-user checkout and multi-store pickup, GA4 browser first, Perfectly Clear upscaling at fulfillment, myPixfizz SSO, webhook auto-deactivation, Shopify order status and duplicate users, OrderHub hot folders on a NAS). Every target section name checked against the file; section names are shortened before an em dash. Source: kbsync.
