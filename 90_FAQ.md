# 90 — Frequently Asked Questions

**Authority Scope:** Customer-facing Q&A grounded in platform truth (files 10–32, 60). Covers Full Pixfizz / Shopper and Shopify + Pixfizz deployments. Not a developer reference — answers are written for store owners and operators.

_Last updated: 2026-07-31_

---

## Scope tags used in this file

- **All** — applies to every Pixfizz deployment
- **Full Pixfizz** — applies to sites using the Pixfizz storefront and checkout
- **Shopper** — applies specifically to Shopper-template sites (subset of Full Pixfizz)
- **Shopify** — applies to Shopify + Pixfizz deployments only
- **Film Labs** — applies to film processing / photo lab businesses specifically

---

## Section 1 — Getting Started & Account Basics

**Q: What is Pixfizz and how does it work?**
_Applies to: All_

Pixfizz is a personalized commerce platform for print and photo product businesses. It handles product personalization (photo books, prints, canvas, calendars, etc.) and the full journey from customization through to production and fulfillment. Depending on your setup, Pixfizz may also run your storefront and checkout, or it may work alongside an existing platform like Shopify.

---

**Q: What's the difference between Full Pixfizz and Shopify + Pixfizz?**
_Applies to: All_

In a **Full Pixfizz** deployment, Pixfizz runs everything — your storefront, checkout, product catalog, payments, and production workflow. In a **Shopify + Pixfizz** deployment, Shopify handles the storefront, catalog, checkout, and payments, while Pixfizz handles product personalization and production. The right setup depends on whether you already have a Shopify store or are starting fresh.

---

**Q: What is a guest user vs a registered user?**
_Applies to: All_

A **registered user** has created an account with login credentials and can access saved projects and order history. A **guest user** completes checkout without registering — they can receive order confirmation emails and their order is tracked, but they cannot log in or access saved projects later. Admins can merge a guest user into a registered account via the admin, which transfers their order history and projects.

---

**Q: Can a customer use the site without being logged in?**
_Applies to: All_

Yes. Pixfizz creates a temporary anonymous identity for every visitor automatically. Customers can browse, personalize products, and add to cart without logging in. If they then register or log in, their cart and any saved projects transfer to their account automatically.

---

**Q: How do I raise a support ticket, and where do I track it?**
_Applies to: All_

Support runs through the **myPixfizz portal** (`my.pixfizz.com`). Raise a ticket there, track its status, and see the full conversation thread in one place. The portal is also where you find training videos, what's new, the roadmap, and your onboarding tasks.

The previous third-party helpdesk is being retired — from **1 September 2026** the myPixfizz portal is the support channel. Tickets raised through the old system before that date are being carried across; if you have an open ticket, it will continue to be worked. Email to the support address still reaches the team either way.

---

**Q: Is there regular training on new platform features?**
_Applies to: All_

Yes. Pixfizz runs a **quarterly review webinar** covering platform updates and new capabilities — the next one is **1 September 2026**. Recordings and walkthrough videos are published in the myPixfizz portal afterwards, so missing the live session is not a problem.

---

## Section 2 — Products, Templates & Collections

**Q: What's the difference between a Product Attribute, a Template, and a Design?**
_Applies to: All_

These three objects work together to define a product:

- A **Template** is the production blueprint — it defines dimensions, DPI, and how artwork is generated for manufacturing.
- A **Design** sits under a Template and is the customer's starting point when they open the editor — a pre-built layout they can personalize.
- A **Product Attribute** is the commercial side — pricing, variants, name, and description — linked to a Template.

Multiple Product Attributes can share the same Template (e.g. metal prints and acrylic prints with identical production specs but different prices).

---

**Q: What's the difference between a design product and a static product?**
_Applies to: All_

A **design product** is a Product Attribute linked to a Template. It requires personalization — customers go through the editor before adding to cart. A **static product** has no Template link — it's a standard item like a frame, gift voucher, or accessory that goes straight to the cart without any customization step.

---

**Q: What is a Collection and how does it relate to products?**
_Applies to: Full Pixfizz_

A Collection is how products are published to your storefront. A product does not appear in your store until it's added to at least one Collection. Collections also determine the URL structure — customers browse by collection path (e.g. `/shop/photo-books`). A product can appear in more than one Collection.

---

**Q: Why can't my customers see a product I've just set up?**
_Applies to: Full Pixfizz_

The most common reason is that the product hasn't been added to a Collection yet. Check that it's published to at least one active Collection. If it is, also check that the Collection itself is active and accessible from your storefront navigation. For Shopify deployments, check that the product metafields are correctly configured — see the Shopify section below.

---

**Q: What are Variants and Options, and what's the difference?**
_Applies to: All_

**Variants** live on Product Attributes. They're the commercial choices customers make (size, finish, material) and affect pricing. **Options** live on Templates or Designs. They're production-level choices. This split means multiple products sharing the same Template don't have to duplicate their production options.

---

**Q: Can I track inventory on my products?**
_Applies to: All_

Yes. Inventory tracking can be enabled per product at the product attribute level. Once enabled, set the current stock count in admin. Pixfizz automatically reduces stock the first time an order reaches Confirmed or Draft status. Stock is only deducted once per order. Note that out-of-stock does not automatically block purchases — the count can go negative. You can add storefront logic to show "Out of stock" messaging or hide the Add to Cart button using the `product.tracks_inventory` and `product.current_inventory` Liquid properties. See `18_ADMIN_NAVIGATION.md` for the full setup guide.

---

## Section 3 — The Design Tool & Personalisation

**Q: How do customers start personalizing a product?**
_Applies to: All_

From the storefront, customers browse to a product, select a Design (their starting point), and enter the design editor. Inside the editor they can upload images, add text, apply layouts, and preview their product. When satisfied, they add it to cart and proceed to checkout. On Shopify, this flow is triggered by a Customize or Personalize button on the product page.

---

**Q: Can I control which features appear in the editor?**
_Applies to: All_

Yes. Design Tool Configurations (found in admin under Settings > Design Tool) let you toggle 30+ individual features on or off — image rotation, crop, filters, autofill, text tools, shape buttons, and more. Different product types can use different configurations. Some configuration settings are managed by Pixfizz staff at the platform level.

---

**Q: I only want image upload available on some templates, not all of them — how do I do that?**
_Applies to: All_

A single global `editor.css` rule (or Custom CSS field) that hides image upload applies to every template using that configuration, so it can't give you per-template control on its own. Because a Design Tool Configuration is assigned per Template or per Design (not per product or category), the way to get per-template control is to create a separate configuration for the templates that should hide image upload, with its own CSS, and assign only those templates to it — leaving the rest on a configuration where upload stays visible. See "Can I control which features appear in the editor?" above and 17_DESIGN_TOOL.md § Design Tool Configurations / Editor CSS Customization. Source: support ticket #18337.

---

**Q: Can customers save their designs and come back to them later?**
_Applies to: All_

Yes. Personalization projects are saved to the customer's account (or to their browser session if not logged in). Registered users can access saved projects by logging in. Admins can view and manage all projects in the admin under Orders > Projects.

---

**Q: What happens if a customer uploads a low-resolution image?**
_Applies to: All_

The design tool can display a resolution warning when an uploaded image is below the minimum DPI required for quality output. Whether this warning appears depends on your Design Tool Configuration. Customers can still proceed with a low-resolution image, so it's worth communicating minimum image requirements in your product descriptions or FAQs.

Relatedly, uploading **CMYK JPEGs** (for example to business-card products) can cause print-quality problems, because the design tool and previews are built around RGB/sRGB. Advise customers to upload **sRGB** images and reserve any CMYK conversion for the fulfillment transformation step. Source: Fireflies (2026-07-03).

---

## Section 4 — Pricing & Options

**Q: How is product pricing set up?**
_Applies to: Full Pixfizz_

Pricing is set on Product Attributes using Ruby formula expressions. Formulas can be simple fixed prices, linear (per item), tiered (volume price breaks), or page-based for multi-page products like photo books. You can also use Price Variables — named numeric values defined once in admin and referenced across multiple formulas — to make bulk pricing updates easier.

---

**Q: What are Price Variables?**
_Applies to: Full Pixfizz_

Price Variables are named numeric values you define once in admin and reference by name in pricing formulas. For example, you could define a variable called `base_rate` and use it across multiple product formulas. When you need to change the rate, you update it in one place and all products using that variable reflect the change automatically.

---

**Q: My pricing isn't displaying correctly — what should I check?**
_Applies to: Full Pixfizz_

Start with the pricing formula on the Product Attribute. Check that it uses the correct variable for the product type — `cut_print_quantity` is only valid for photo prints; all other products should use `units` or `quantity`. Check that any Price Variables referenced actually exist in admin with the right values. If tiered pricing is set up, make sure all quantity ranges are correctly defined and cover the full expected range.

---

**Q: Can I edit product prices without opening each product individually?**
_Applies to: Full Pixfizz_

Yes. Prices are editable inline directly from the product attributes list page. Click on any price field to switch to edit mode, type the new price or formula, and press Enter for simple prices or click OK for multi-line formulas. Press Esc to cancel without saving.

---

## Section 5 — Cart & Checkout (Full Pixfizz / Shopper)

**Q: Can customers edit their personalized product from the cart?**
_Applies to: Full Pixfizz / Shopper_

Yes, depending on your site's cart settings. If the cart edit setting is enabled, customers can re-enter the editor directly from the cart. If not, they would need to return to the product and start a new project. Photo print quantities are not editable in the cart — they are managed inside the Photo Prints UI.

---

**Q: Why is a delivery option missing at checkout?**
_Applies to: Full Pixfizz / Shopper_

If a product in the cart has shipping or pickup marked as unavailable via its custom fields (`shipping_unavailable` or `pickup_unavailable`), those delivery methods are hidden at checkout for that cart. Check the product's custom fields in admin. Film processing products also affect checkout delivery options if the film processing flag is set.

---

**Q: How does digital-only delivery work?**
_Applies to: Full Pixfizz / Shopper_

For digital products — those with the variant version set to `digital-only` — Pixfizz bypasses shipping entirely at checkout. A system address is applied in the background and the customer sees a digital delivery state rather than shipping options. The `digital-only-delivery` admin checklist toggle must be enabled for this to activate.

---

## Section 6 — Shopify + Pixfizz

**Q: How does the Pixfizz integration work on my Shopify store?**
_Applies to: Shopify_

Pixfizz runs on a custom subdomain of your store (e.g. `create.yourstore.com`). A Pixfizz JavaScript API loads into your Shopify theme. When a customer clicks Personalize on a product page, they're taken into the Pixfizz editor. On completion, their project is linked to a Shopify cart item. When the order is paid, Shopify sends a webhook to Pixfizz, which confirms the order and triggers the production pipeline.

---

**Q: My Shopify product isn't launching the Pixfizz editor — what should I check?**
_Applies to: Shopify_

The most common causes are:
1. The `pixfizz.product_sku` metafield isn't set on the product (or on the variant if the product has Shopify variants — the variant-level SKU takes precedence).
2. The `pixfizz.integration_type` metafield is missing or set to an unrecognised value.
3. The Pixfizz JS snippet isn't included in the Shopify theme on that product page.

Check all three before raising a support ticket.

---

**Q: What integration types are available for Shopify products?**
_Applies to: Shopify_

There are four, set via the `pixfizz.integration_type` metafield:

- `editor` — launches the full Pixfizz editor. Default. Used for photo books and products requiring full customization.
- `options-to-editor` — customer selects options first (e.g. calendar start month), then enters the editor.
- `options-to-cart` — shows a live preview modal. Customer selects options and previews before adding to cart, with no full editor. Used for simpler products like notebooks.
- `photo-prints` — launches the photo prints flow. Quantity is managed inside Pixfizz, not in the Shopify cart.

---

**Q: Why isn't my Shopify order being confirmed in Pixfizz?**
_Applies to: Shopify_

This is almost always a webhook configuration issue. Shopify must be set up to send order payment webhooks to Pixfizz. Check that the webhook is active in Shopify (Settings > Notifications > Webhooks) and that the endpoint URL is pointing to your Pixfizz instance. Also verify that the shared secret configured in both systems matches.

---

**Q: Why are extra pages showing as a separate line item in my Shopify cart?**
_Applies to: Shopify_

This is expected for photo book products. Pixfizz adds extra page charges as a linked addon line item, using the product variant configured in the `pixfizz.page_addon_product` metafield. These addons are automatically linked to the parent item and scale with quantity changes. They should not display independently to the customer if the cart template is correctly filtering private line item properties (those prefixed with an underscore).

---

**Q: Can Pixfizz send fulfillment updates back to Shopify?**
_Applies to: Shopify_

Yes. When an order is fulfilled in Shopify, a second webhook can be configured to fire and mark the corresponding Pixfizz order as Shipped. This keeps order status in sync between both platforms.

---

## Section 7 — Orders & Production

**Q: What do the different order statuses mean?**
_Applies to: All_

| Status | Meaning |
|---|---|
| Pending | Order received, awaiting confirmation |
| Confirmed | Order confirmed, ready for production |
| Downloaded | Production assets downloaded by production team |
| Manufactured | Production complete |
| Shipped | Order dispatched to customer |
| Fulfilled | Order delivered and complete |
| Payment Failed | Payment was not successfully captured |
| Error | An error occurred — check the order history log |
| Canceled | Order canceled |
| Refunded | Order refunded |

---

**Q: Why hasn't my order moved to Confirmed?**
_Applies to: All_

For Full Pixfizz orders, confirmation happens after payment is successfully captured. For Shopify orders, confirmation is triggered by the Shopify order paid webhook — if it isn't firing or is misconfigured, the order stays in Pending indefinitely. Check your webhook configuration and review the order's history log in admin for any error messages.

---

**Q: Where do I find production files for an order?**
_Applies to: All_

Open the order in admin and look at the individual orderlines. Each orderline has a Generated Files section with downloadable production assets. You can also use the OrderHub Desktop (OHD) application to manage and download production jobs from your queue.

---

**Q: How do I update an order's status to Shipped?**
_Applies to: Full Pixfizz_

In the order detail view in admin, you can manually update the order status. Moving it to Shipped will trigger the shipping notification email to the customer if email notifications are configured for that status transition (Settings > Email Notifications).

---

**Q: A customer says they didn't receive a confirmation email — what should I check?**
_Applies to: All_

Check that email notifications are configured and active for the relevant status (Settings > Email Notifications). Verify the customer's email address is correctly recorded on the order. Also check that your email sending domain is correctly configured and not being flagged as spam — your Pixfizz account needs a verified sender domain for reliable delivery.

---

**Q: How do I cancel an order and avoid being charged transaction fees?**
_Applies to: All_

Marking an order as "Canceled" in the admin is **not sufficient** on its own to prevent Pixfizz transaction fees. You must also email the order codes to **finance@pixfizz.com** so the finance team can process the cancellation correctly. After sending, follow up to confirm the cancellations have been processed and no charges will apply. Also check that cancelled orders are not duplicated (e.g. from a customer re-ordering after a failed attempt) — duplicate charges can occur if both remain active.

---

## Section 8 — Storefront & Site Management (Full Pixfizz / Shopper)

**Q: Where do I make changes to my homepage?**
_Applies to: Shopper_

Custom homepage content lives in the `website/homepage` snippet in the CMS. This snippet only renders when the "Custom snippet (website/homepage) for home page" checkbox is enabled in Admin > Storefront Settings. Both the snippet content and the checkbox must be active at the same time for your changes to appear.

---

**Q: How do I add a new page to my site?**
_Applies to: Shopper_

On Shopper-based sites, new pages are created by adding instances of the `pages` Custom Type in your CMS admin — not by creating standard CMS pages. Each instance needs a page path, title, and content snippet. The page path must be a URL segment that doesn't conflict with any existing CMS page.

---

**Q: How do I change the logo or site fonts?**
_Applies to: Shopper_

Logo, fonts, and most branding settings are controlled through the admin checklist system in Storefront Settings. The relevant checklist keys are `update-website-logo` for the logo and `font-body` for the body font. Set the value in the checklist and save.

---

**Q: How do I add a promotion bar to the top of my site?**
_Applies to: Shopper_

Enable the `top-promotion-bar` checklist key in Storefront Settings. Once enabled, you can configure the bar content in the associated snippet or setting.

---

**Q: Can I run my store in multiple languages?**
_Applies to: All_

Yes. Pixfizz has built-in translation support for core objects including products, designs, collections, templates, variant types/values, and template option types/values. Enable multi-language support in the Super Admin, select your languages, and then use the "Translate" link that appears on supported objects in admin. Translations are applied automatically across the storefront and design tool — Liquid properties like `{{ design.name }}` resolve to the correct language automatically. Bulk translation management is also available via export/import. See `18_ADMIN_NAVIGATION.md` for the full setup guide.

---

## Section 9 — Common Troubleshooting

**Q: A customer says their project has disappeared — what happened?**
_Applies to: All_

Projects are stored against user accounts. If the customer was browsing anonymously and didn't log in, their project is tied to their browser session — it will be lost if they clear cookies, switch browsers, or use a different device. If they were logged in, the project should still be accessible under their account. Admins can search for projects in admin under Orders > Projects.

---

**Q: Customers are getting a confirmation email but the order still shows as Pending — is that normal?**
_Applies to: All_

It's possible — email notifications can be configured to fire at the Pending stage, so a confirmation-style email can go out before the order is fully confirmed. Check Settings > Email Notifications to see which status triggers which email. If an order stays in Pending for an extended period, investigate the payment or webhook flow.

---

**Q: Sorting on my site looks wrong — items aren't appearing in the order I set.**
_Applies to: Shopper_

Sort order only works correctly when every item in a set has a value in the sort field. If some items are missing a position or order value, the result is unpredictable — those items appear in a random order. Go into admin and make sure every record in that Custom Type has a position value assigned, using a simple sequential numbering like 1, 2, 3. This is a common issue with services, FAQ items, and other Custom Type records.

---

**Q: A customer is getting an error when trying to add a product to cart — what could cause it?**
_Applies to: All_

Common causes include: the product isn't correctly published to a Collection (Full Pixfizz), the product's pricing formula has an error and can't return a valid price, required metafields are missing or incorrectly configured (Shopify), or the customer's session has a conflict with an existing project. Check the admin order logs and, if the issue persists, contact Pixfizz support with the order or error details.

---

## Section 10 — Film Lab Workflows

**Q: How does the Batch Film Uploader work?**
_Applies to: Film Labs_

The Batch Film Uploader automates the process of uploading scanned film to customer galleries via FTP. The full workflow is:

1. **Upload scans to FTP** — copy your daily scanned film folders to the designated Pixfizz FTP folder (typically `Pixfizz/Film Scans`). Pixfizz sets this up for you.
2. **Automatic sync** — Syncovery automatically moves the scanned folders every 5 minutes. When the source folder is empty, your scans have uploaded successfully.
3. **Link folders to orders** — open the order in admin (scan the QR code from the 6x4 label or the barcode on the order sheet). Find the `film_scans` custom field at the bottom of the order page. Enter the film scan folder IDs (must match FTP folder names exactly) and click Add, then Save.
4. **Mark order as Shipped** — after uploading all folders to FTP and linking them, mark the order as Shipped (or Ready for Collection). You can complete this step and step 3 at the same time.
5. **Automatic gallery creation** — after step 4, the system matches the film scan folder IDs with the FTP folders and creates Image Galleries in the customer's account. Each roll of film creates a separate gallery, named "Order code - Folder ID". There is a default 1-hour processing delay to ensure all scans finish uploading before matching begins.
6. **Customer notification** — the customer receives an email notification when their scans are available in their gallery.

Configurable options include:
- Custom notification email content (can include product recommendations)
- BCC email address for record-keeping
- Folder numbering prefix accommodation (e.g. auto-adding "0000" prefix)
- Reply-to address separate from the sending address
- Processing delay adjustment (default 1 hour)

---

## Changelog
- 2026-08-14: Added Section 1 entries for the support channel (myPixfizz portal; third-party helpdesk retired 1 September 2026) and the quarterly review webinar. Source: fireflies-call (2026-08-11/12/13, 3x repeat signal).
- 2026-04-06: Initial version. 35 Q&As covering Getting Started, Products, Design Tool, Pricing, Cart/Checkout, Shopify, Orders, Storefront, and Troubleshooting.
- 2026-05-19: Added inventory tracking Q&A (Section 2), inline price editing Q&A (Section 4), order cancellation and transaction fees Q&A (Section 7), multi-language support Q&A (Section 8), and Batch Film Uploader workflow (new Section 10 — Film Lab Workflows). Source: Notion KB articles.
- 2026-07-04: Added CMYK-JPEG upload caution to the image-upload Q&A (Section 3) — upload sRGB; reserve CMYK for fulfillment transformation. Source: Fireflies (2026-07-03).
- 2026-07-31: Added per-template image upload visibility Q&A (Section 3) — use a separate Design Tool Configuration per template rather than a single global editor.css rule. Source: support ticket.
