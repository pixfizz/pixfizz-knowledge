# 18 — Admin Navigation

**Authority Scope:** Pixfizz Core admin interface sections and settings only.

_Last updated: 2026-09-24_

---

## Admin Overview

The Pixfizz Core admin is the central interface for managing your store, products, orders, and settings. Since 2026-09-23 it lives at `https://admin.pixfizz.com/site/<slug>/admin/`. The old `{your-domain}/admin` address still works in a browser: it answers 301 to the same path on the admin host.

> **The admin moved host on 2026-09-23** (a security update, confirmed intentional and permanent
> by the core developer). Paths in this file are written relative to `/admin`; in a browser they
> resolve under `admin.pixfizz.com/site/<slug>/admin/`. The same deploy converted parts of the
> admin to htmx, and tools and bookmarks with hard-coded admin URLs broke (stated by the core
> developer, 2026-09-23). Two consequences:
>
> - Some admin forms and field values are now rendered client-side and are **not in the fetched
>   HTML** (for example template option custom fields and the Ace-backed `custom_script`
>   textarea). Read them from a rendered page, never from a `fetch` of the page.
> - **The API did not move.** Integrations keep calling `https://<slug>.pixfizz.com/v1/admin/...`,
>   never the admin host. See `61_PIXFIZZ_API.md` § 2.
>
> Re-check any path recorded here against the live admin before sending it to a customer.
> *Host change verified live, 2026-09-23 and 2026-09-28.*

---

## Admin Sidebar Sections

### Dashboard
Key business metrics: Gross Revenue, Orders Fulfilled, Average Order Value, Sessions, Conversion Rate. Quick links to OrderHub, community, help center.

The dashboard also surfaces active (in-progress) carts, giving visibility into carts in flight for account management and follow-up. Exact location and any configuration to be confirmed.

### Orders
- **Orders** — view/manage with status filters, CSV export, barcode search
    - **Design custom fields in the orderline CSV:** to output a design-level custom field as a column in the orderline CSV export, reference it with the nested path `custom:print_book:print_theme:<field-name>` (for example `custom:print_book:print_theme:primary_collection_path`). The standard orderline CSV cannot filter on these fields, but it can output them using this path.
    - **Money log:** the order detail page shows a payment **summary** only. The individual money log entries for that order live in Super Admin, reached with the **Show details** link on the summary. Go via Show details for per-transaction history, partial captures, or when a payment total on the order page looks wrong.
- **Abandoned Carts** — incomplete checkouts
- **Production Files** — production book files with project, page count, status
- **Projects** — end users' saved personalization projects

### Galleries
Public galleries where customers share and view photo projects.

### Users
Customer accounts and access management.
- **API Keys** (since 2026-09-23): a section on each user's page, below Change Password and Custom Fields, with **Add API Key** at its top right. A key belongs to that user on that site; there is no site-level keys page. The full key is shown once. See `61_PIXFIZZ_API.md` § 2 for how to use it.
- The user page sections, in order: user details (User Status, Email, names, Telephone, Category, the Admin checkbox), AI Tokens, Change Password, Custom Fields, API Keys, Promocodes, Orders, Projects, Galleries, Calendars, Addresses. *Verified from admin screenshots, 2026-09-28.*

### Shipping
- **Shipping Services** — carrier and method configuration
- **Packaging** — package type definitions
- **Taxes** — tax rules
- **Addresses** — address management
- **Extra Fees** — additional fee configuration. Full path: **Main Admin → Shipping → Extra Fees**. Each fee carries three fields above the formula: **Name**, **Code**, and a per-fee **Taxable** checkbox. Whether a fee is taxed is configuration on the fee — per fee, per site — not a template change and not a site-wide policy, so the same fee code can be taxable in one jurisdiction and not another. Verified by reading source (admin screenshot, 2026-09-09). For the formula itself see `30_PRICING_ENGINE.md`.

### Marketing
- **Automatic Discounts** — cart-level discounts that apply with no promo code, each driven by a Liquid formula that returns an amount. This is where they live, not under Promotions. See `30_PRICING_ENGINE.md` § Automatic Discounts for the formula patterns and the `cart.promocode_code` trap.
- **Promocodes** — promotional code management
- **Gift Vouchers** — gift voucher / gift card issuing and tracking
    - **A voucher's value can be changed after the voucher has been created.** It is not fixed at issue. Updating the value is supported through the API, which makes it possible to top up or reset an existing voucher rather than issuing a replacement code. Useful for re-use-credit promotions where the customer already holds the code. TO CONFIRM: whether the value is also editable directly in the admin UI, and the exact API endpoint and payload.
    - Gift voucher emails are sent through the `email-shopper/templates/gift-voucher` template (see `52_SNIPPET_INVENTORY.md`).
    - Voucher codes can be **embedded in fulfillment output** — printed into the production/packaging files for an order — so an online-issued voucher can be tracked when it is redeemed in store. Implemented through the fulfillment template, the same mechanism as any other per-order dynamic value (see `31_FULFILLMENT_ENGINE.md`). TO CONFIRM: the exact Liquid accessor for the voucher code on the orderline.

### Products
- **Published Products** — published products list (everything in a Collection). Renamed from "All Products" on 2026-03-30 to make it clear the listing is scoped to published items only.
  - The button that adds a product here is labelled **Publish Product** since the 2026-09-28 deploy (previously **Add Product**). *Stated by the core developer, 2026-09-28.*
- **Product Attributes** — commercial product definitions. Prices are now **editable inline** directly from the product attributes list — click on any price field, type the new value (simple price or formula), press Enter for simple prices or click OK for multi-line formulas, Esc to cancel. No need to open each product individually.
- **Templates** — production specifications. Includes a bulk **Text Upgrade** action (shipped 2026-03-05) that applies text-box vertical alignment fixes across all templates in one step — use when migrating older templates that pre-date the current text rendering.
  - **Template option `custom_script`** is an Ace editor over a hidden textarea, `template_option_type[custom][custom_script]`. Save it with the **Custom Fields** Save button on the option page, not the top form's Save. The value is stored and exported with CRLF line ends (a browser form submit sends CRLF), while the textarea in the page shows LF; compare mounts after folding CRLF to LF. *Verified live, 2026-09-25 and 2026-09-26.*
- **Collections** — product groupings for storefront
- **Fonts** / **Font Palettes** / **Color Palettes** — typography and color management
  - **Gotcha:** the default font palette tooltip implies fonts are auto-assigned, but fonts must be **manually assigned** to a palette. If a design shows fallback typography, check the palette assignment rather than assuming the admin auto-populated it.
- **Variant Types** — the commercial option types a product offers. **Variant price formulas are edited on the variant type, not on the product attribute**, at `/site/<site>/admin/variant_types/<id>/edit`. That field runs its own validator, which is narrower than Ruby — see `30_PRICING_ENGINE.md`. Verified by reading source (live admin test, 2026-09-08).
- **Element Substitutions** — swap rules
- **Calendars** — calendar product configuration
- **Inventory tracking** — see the dedicated Inventory Management section below.

### Website (CMS)
Manages storefront content. What's visible depends on Shopper vs standalone CMS:

- **Pages** — CMS pages forming storefront URL structure (standalone CMS only — Shopper manages these automatically)
    - **Admin-only visibility:** individual pages and blog posts can be set to admin-only, so they are visible to logged-in admins but hidden from the public. Use this to stage and review content before its public release, then switch it on to publish. This is a publish gate, distinct from `d-none`, which only hides an element visually while leaving its links crawlable.
- **Layouts** — wrapper templates that pages render inside (standalone CMS only)
- **Snippets** — HTML/Liquid template fragments (building blocks of pages)
- **Custom Types** — dynamic content types for flexible page content
- **Assets** — file manager for images, fonts, media
- **Crawler** — website crawl management for sitemaps and product feeds. Accessed at `{domain}/admin/website_crawls`. Each crawl run lists every URL the crawler followed, with response status (200, 404, etc.) and the page that contained the link (shown in the rightmost column). Use this to track down broken internal links — the linking page column tells you exactly which snippet or page to fix.
    - **Sitemap:** generated at `{domain}/sitemap.xml`. Product feed at `{domain}/product-feed.json`. Both are platform-level features, not Shopper-specific.
    - **Gotcha:** the sitemap is **not** at `/site/sitemap`. Do not invent this URL. Confirm against the live admin.
    - **Gotcha (2026-02-02):** product XML feed URLs have shipped with an explicit `:80` port in the URL, making them unreachable for some consumers (Google Merchant Center rejected ~915 URLs on one site). If you see products missing from a feed with no obvious cause, inspect the raw XML for `:80` in the URLs before looking elsewhere.

> On Shopper sites, Pages and Layouts are pre-configured. You primarily work with Snippets, Custom Types, and Assets.

### Settings
- **General** — website title, timezone, language, currency, domain hosting, currency formatting
- **Email Notifications** — email templates for order lifecycle events (14 templates)
- **Design Tool** — Design Tool Configurations (branding, features, defaults)
- **Translations** — see the dedicated Translation Support section below.
- **Webhooks** — webhook configuration for external integrations (e.g. order.created, order.status_changed)
  - **Analytics pattern — server-side GA4 via webhook (Nicolas Restrepo, 2026-03-26):** client-side GA4 typically captures only ~10% of conversion events due to ad blockers, cookie consent drop-off, and tracking prevention. Piping `order.created` / `order.confirmed` through a Pixfizz webhook to a server-side GA4 endpoint (via a middleware like GTM Server, Stape, or a simple Cloud Function) lifts capture to ~50%+. Recommend this for any site where attribution accuracy matters for paid media decisions.

---

## Inventory Management

Pixfizz supports per-product inventory tracking, allowing you to monitor available stock and automatically reduce quantities when orders are placed.

### Enabling inventory tracking

Inventory is managed at the **product attribute level** — it is not active by default. To enable:
1. Open the Product Attribute in admin.
2. Enable the inventory tracking toggle.
3. Set the current inventory count (how many units are currently available).

### How stock is reduced

When an order includes a product that tracks inventory, the system automatically subtracts the purchased quantity **the first time** the order enters one of these statuses:
- **Confirmed (C)**
- **Draft (W)**

Stock is only reduced once — the first time the order hits either status. Subsequent status changes do not re-deduct.

### Negative stock and out-of-stock behavior

Inventory tracking **does not automatically block purchases**. If stock reaches 0, customers can still place an order — the inventory count will go negative. This gives operators flexibility but means you may want to add storefront controls.

### Liquid properties for storefront logic

Two Liquid properties are available on products for CMS-level stock management:

- `product.tracks_inventory` — boolean, true if inventory tracking is enabled for this product
- `product.current_inventory` — integer, the current stock count (can be negative)

Use these to:
- Display stock levels ("Only 3 left")
- Show "Out of stock" messaging
- Disable or hide the Add to Cart button when inventory reaches 0

Example logic pattern:
```
{% if product.tracks_inventory and product.current_inventory <= 0 %}
  <!-- show out of stock message, hide add-to-cart -->
{% endif %}
```

### Dynamic stock messaging

Pair inventory tracking with configurable product display names via Liquid (`50_LIQUID_REFERENCE.md`) to show threshold-based messages like "Only 3 left" directly in the product title or product card. The ability to block orders below a configured stock level is also available.

### Who is this for

Inventory management is designed for store owners who track physical stock or limited-quantity products — limited edition items, seasonal products, pre-produced goods, or physical items with fixed inventory.

---

## Built-in Translation Support

Pixfizz includes built-in translation support for core platform objects. This allows multi-language stores to manage translations directly in the admin and display the correct content automatically based on the user's language.

### Enabling multi-language support

1. Go to the **Super Admin**.
2. Enable **Multi-language support**.
3. Select the languages you want to support — to select multiple, hold **Command (Mac)** or **Ctrl (Windows)** while clicking.
4. Save.

Once enabled, translation options become available across supported objects.

### What can be translated

The following core objects support translation:

- **Products** — names and descriptions
- **Designs** — names, tags (layouts, backgrounds, clipart)
- **Collections** — names and descriptions
- **Templates** — names, page captions
- **Variant types and values** — the commercial options customers see on the storefront
- **Template option types and values** — production-level options

This covers both the storefront (CMS) and the design tool — customers see localized content end to end.

### Editor translations are a separate namespace, and enabling a language does not create them

**Enabling a language under Settings → Translations does not translate the editor.** Neither
does passing the locale into the theme `setup()` call. Editor strings live in their own
`editor` namespace, and if that namespace has no entries on the site, the editor opens in
English however the storefront is configured.

- Admin path: `/admin/translations?namespace=editor&locales[]=<iso>`.
- The editor-namespace translations must **exist on the site**. They are obtained by
  **exporting them from a site that already has them** and importing them into the new site;
  they are not generated by enabling the language.

Verified by reading source (theme setup call and admin translations path, 2026-09-09).
This corrects the reasonable assumption, implied by the enable flow above, that switching a
language on covers every surface — it covers the storefront objects listed above, not the
editor.

### Where to find translations in admin

When multi-language support is active, a **"Translate"** link appears in the top-left corner of supported objects. Clicking it opens the translation page where you can manage content for each enabled language.

### How translations are applied

Translations are automatically resolved based on the user's current language. In Liquid templates, standard object properties return the translated value automatically. For example:

```
{{ design.name }}
```

This displays the design name in the current language — no conditional logic needed.

### Bulk translation management

A major upgrade shipped 2026-03-30 added:
- Built-in translation support for **products and templates** (previously only pages and snippets were translatable).
- **Automatic translation key detection** during import.
- **Per-template and per-product-variant** translation handling.
- **Export / import** flow for bulk translation editing — export all translation keys to a file, translate externally, and re-import.

### What this is useful for

- Offer a fully localized experience across storefront and editor
- Manage translations in one place (Pixfizz admin) rather than external systems
- Ensure consistency across all touchpoints (product pages, design tool, options, captions)

---

## Custom Fields, Schema Order and Bulk Export/Import

### Settings → Custom Fields — one page for every definition

A dedicated admin page at **Settings → Custom Fields** lists every custom field definition on
the site, across object types. From it you can **export and import definitions for all
supported object types, or a chosen subset**, in one pass instead of visiting each object's
own custom fields screen.

- It moves **definitions**, not values.
- **Resolved 2026-09-21: the site-wide export carries the object type.** Every definition has an `owner_type` key (the per-object archive described below still does not). Admin label → class: Product Attributes → `Product`, Collections → `ThemeCategory`, Designs → `PrintTheme`, Template Options → `TemplateOptionType`, Template Option Values → `TemplateOptionValue`, Orders and carts → `Order`, Order Lines → `Orderline`, Users → `User`, Addresses (pickup locations) → `Location`, Pages → `Page`, Projects → `PrintBook`. Not yet seen: Galleries, Templates. *Verified by reading source.*
- **Re-importing the same site-wide file changes nothing.** It is refused with *No custom fields were imported. They may already exist on this site.* Existing definitions are skipped, never duplicated and never updated: a changed type, description or Public flag does not propagate by re-import. *Verified live, 2026-09-21.*

### Custom type definition export/import

The **Custom Types** index page can now export and import **all, or a subset of, custom type
definitions**. It exports the **type definitions only, not the instances** — content has to be
moved separately (for example with the `/v1/admin/custom_types/<id>/custom_type_instances`
endpoint in `61_PIXFIZZ_API.md` § 13c).

Both features were announced on the Notion Dashboard, week of 2026-09-21, alongside the
staging release. Confirm they are present on the site before sending a customer to them. Not
verified live.

### Reordering product custom fields — Edit Schema

Beside the custom fields on a product there is an **Edit Schema** link. It reorders which
custom fields appear at the top of the product editor.

- It is a **view preference only** — it changes presentation, never data, and never which
  fields exist.
- The order is **per admin user**, and it applies **across all products**, not to the one
  product it was opened from.
- It is freely changeable, so there is no cost to getting it wrong.
- **It saves automatically. There is no Save button** — the absence of one is not a bug and
  is the thing people get stuck on.

Stated on a client call and reproduced in admin in the same session; **verified live**, not
verified by reading source.

### A custom field definition archive has no object type in it

A custom field definition export contains one member,
`./__custom_field_definitions.yml`, and that file is a flat list of definitions with
**no object-type key anywhere in it**. The object the definitions land on is decided
entirely by **where in the admin the import is run**.

The consequence, and it is the one that bites: **one archive imported at two different
object types creates the fields on both.** Import at the wrong screen and the fields exist,
but not on the object the code reads. Verified by reading source (archive contents,
2026-09-09). See `51_CUSTOM_FIELDS_REFERENCE.md` for the per-object field inventories.

### Bulk export and import

- **Price variables** can be bulk exported and imported from admin — a full export of every
  price variable on the site, edited externally and re-imported. Stated, not independently
  verified; a help article dated 2026-08-05 is the canonical reference.
- **Full variant exports can be exported and edited, but re-importing one never updates in
  place.** A code collision appends `-1` to every code in the set (settled 2026-09-20, see
  `22_OPTION_VARIANT_RENDERING.md` § Variant Type Exports), so an export, edit, re-import round
  trip is not a bulk price-editing route. To copy a variant set from one template to others, use
  Bulk Update Tools (below). Per-product prices are edited per product: there is no write API
  for variant prices (`61_PIXFIZZ_API.md` § 13g).

---

## Bulk Update Tools: Rolling a Change Across Live Templates

**Admin → Advanced → Bulk Update Tools** (super admin only) copies **Product Variants**,
**Template Options** and **Design Options** from one template to a chosen set of other templates
in one pass. Platform-level (Pixfizz CMS).

- **This is the route for rolling a template option or a variant set across a range that is
  already live**, for example adding a custom design tool's `custom_script` mount option to every
  template in a range. Do not delete and re-import the range (imports never update in place, see
  `01_CODE_GOVERNANCE_UPDATED.md` § Never Re-Import to Update), and do not hand-edit each template.
- It copies those three things only. **Per-product values**, such as a different price per size,
  still need per-product edits after the copy.
- **XML definitions and design pages are not covered.** They need per-template edits, which can
  be scripted from a logged-in admin tab through the admin's own forms (paths relative to
  `admin.pixfizz.com/site/<slug>/admin`):
  - XML definition: the template form, `PATCH /print_theme/product/<id>` (fields
    `print_product[name|code|description|category|editor_configuration_id|fulfillment|layout]`,
    `save=Save`). The current layout is not in the fetched textarea; read it from the page's Ace
    mount script.
  - Add a design page: the page row's Copy form, `POST /print_theme/copy_page/<design>?page=<page id>`,
    creates a copy with the same name. Find it by its new id, rename it with
    `PATCH /print_pages/<id>` (`page[name]`) and set its XML with `PATCH /print_pages/<id>` (`page[data]`).
  - Preview flag on a design page: `PUT /print_theme/set_page_as_preview/<design>` with
    `page=<page id>` and `preview=1` (what the Preview checkbox does).
  - Template option custom fields (for example `custom_script`): `PATCH /templates/<t>/options/<option>`
    with **every** `template_option_type[custom][...]` field (checkboxes as a hidden 0 plus 1 when
    on). The form is rendered client-side, so build the field list from a rendered edit page, not
    from a fetch, or the fields left out are blanked.
  - The template option edit page's only server-rendered form is the **DELETE** form
    (`_method=delete`). Never script-submit a form found on that page.

*Bulk Update Tools scope stated by Alex, 2026-09-26 and 2026-09-29. The per-template routes
verified live on a 90-template range, 2026-09-27.*

---

## Website Settings Worth Knowing

- **Website → Crawler.** A daily automatic crawl keeps the sitemap and product feed current; a manual crawl is super-admin only. There is a per-site field for the sitemap URL. *Stated on a client call, 2026-09-22.*
- **Website → robots.txt** is editable in admin, as a second way to block crawling alongside page-level meta tags. *Stated on a client call, 2026-09-22.*
- **Advanced → Redirects** holds 301 redirects as JSON (an outer array of `[from, to]` pairs; a bare single pair silently fails). It is tucked away deliberately because a bad redirect can break the site. *Stated on a client call, 2026-09-22.*
- **Website → Snippets** has a **Search CMS** box that full-text searches snippet content. On a child site the list shows only that site's overrides (`13_TEMPLATE_BOUNDARIES.md`).
- **Super-admin password reset** from the regular admin currently throws an application error; it is a known bug. Reset it from the super admin panel until fixed. *Confirmed as a bug by the core developer, 2026-09-23.*

## Pixfizz Kiosk App — Idle Timeout

The Pixfizz Kiosk desktop app (myPixfizz → Tools → Pixfizz Kiosk) locks the computer to the site's kiosk domain, allows USB image import, and has its own idle timeout that **defaults to 4 minutes** and can be set to 0 (off). This timeout is separate from the storefront's post-order logout (`21_SHOPPER_CHECKOUT_POLICY.md` § Kiosk Sessions). A site that turns it off to stop customers losing carts mid-order also loses the only logout that does not depend on reaching the thank-you page. *Stated on client calls, 2026-09-22 and 2026-09-24.*
## Admin Login: Two-Factor and Passkeys

Platform-level (Pixfizz CMS). Rolling out from 2026-09-28: super users first, then site admins. *Stated by the core developer, 2026-09-28.*

- Admin login runs through `login.pixfizz.com` and adds a **TOTP challenge** (an authenticator app).
- A user can also register a **passkey**. Once a passkey is registered, login requires it, unless an authenticator app is registered too, in which case the app is offered as the alternative.
- **Passkeys are bound to a device; an authenticator app works on any device.** An admin who signs in from more than one computer should register an authenticator app as well as a passkey, or they are tied to the device that holds the passkey.
- **Impersonated sessions** (logging in as a user from admin) expire after **30 minutes of inactivity**.
- Super admin location: see § Super Admin below.

## Super Admin

Cross-website management layer (for organizations managing multiple Pixfizz websites).
Super admin now lives on the admin host: `admin.pixfizz.com/superadmin`, with deep links of the form `admin.pixfizz.com/site/-/superadmin/...`. Old `login.pixfizz.com/superadmin/...` links return not found. Signing in needs the second factor (§ Admin Login). *Verified live, 2026-09-25; entry URL stated by the core developer, 2026-09-28.*

### Customer Super User Access
- **Websites** — manage all Pixfizz websites, configuration, feature flags, integrations
- **Super Users** — admin access across organization
- **Orders** — cross-website order aggregation and search
- **Fulfillment** — configure fulfillment destinations (FTP and HTTP endpoints)

### Website Management (Super Admin)
Per-website configuration includes:
- Core settings (name, subdomain, theme, language, currency)
- Feature flags enabling/disabling platform capabilities
- External integrations (Shopify, Etsy, Google OAuth, Dropbox, HappyAR, ReCAPTCHA)
- Website inheritance — share Design, Products, Tax, Email configurations across websites

> Some Super Admin features are managed by Pixfizz staff. The customer-facing view is deliberately focused.

---

### Semi-Inheritance — Granting a Parent Lab's Templates to a Child Site

For a child site to publish a parent lab's templates, the **parent's Super Admin** adds that
child site to the template's site list. Nothing on the child can grant this.

Once the site has been added:

- The template becomes selectable on the child's **Publish Products** screen.
- Selecting it **auto-populates the product code from the template code**.
- **That auto-populated code stays editable by the child admin**, and editing it is exactly
  how print-on-demand price lookups silently fall to zero — see `45_ORDERHUB.md`.

Stated on a client call, not independently verified against the Super Admin screen.

### AI Tokens (Super Admin)
 
AI token access is **off by default on every website**. It is a Super Admin
feature flag, activated per website by a Pixfizz staff member. A site owner
cannot turn it on themselves, and it is **not** a fulfillment template setting.
 
- Setting: **Enable AI Tokens**, in the website's Super Admin configuration.
- Formerly labelled **Enable Perfectly Clear**. Renamed 2026-07-31, when billing
  moved from a Perfectly Clear-specific model to a generic AI token model that
  also covers OpenAI and Gemini.
- Until Pixfizz activates it on that website, any AI feature that consumes
  tokens is unavailable to the site regardless of other configuration.
- Enabling it is a request to Pixfizz, not self-service.
Token allowances, daily limits and per-lab billing behaviour are separate from
this flag — see `17_DESIGN_TOOL.md` for AI restyle limits. TO CONFIRM: the exact
Super Admin screen and field position, and whether the flag is per website or
per organization.
 
---

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-10: Added Published Products rename, Text Upgrade bulk action, inventory tracking with dynamic stock messaging, translation export/import upgrade, font palette tooltip gotcha, server-side GA4 via webhook pattern.
- 2026-04-22: Expanded Crawler entry with admin path, 404 reporting behaviour, and sitemap URL gotcha.
- 2026-05-19: Expanded Inventory Management into dedicated section with enable flow, stock reduction rules, negative stock behavior, Liquid properties (product.tracks_inventory, product.current_inventory), and out-of-stock CMS pattern. Expanded Translation Support into dedicated section with Super Admin enable flow, translatable objects list, Liquid auto-resolution, translate link location, and bulk export/import. Added inline price editing to Product Attributes. Source: Notion KB articles.
- 2026-06-01: Added active carts on the dashboard. Source: fireflies-call.
- 2026-06-15: Documented admin-only visibility for Pages/blog (pre-publish staging gate) and the design custom-field column path for the orderline CSV export (custom:print_book:print_theme:<field>). Source: slack-kb-sync (Matjaz, #development; design-field reporting work).
- 2026-08-05: Noted that the order detail page now shows a payment summary only, with individual money log entries reached through the Show details link into Super Admin. Source: slack-message (#development).
- 2026-08-11: Documented the Enable AI Tokens Super Admin feature flag — off by default, activated per website by Pixfizz staff, formerly "Enable Perfectly Clear". Clarifies that it is a Super Admin setting and not a fulfillment template field. Source: internal correction.
- 2026-09-09: Added the full Extra Fees path with the per-fee Taxable checkbox, Name and Code; the variant price formula location on the variant type; the editor-namespace translation rule (enabling a language does not translate the editor); a new Custom Fields, Schema Order and Bulk Export/Import section covering Edit Schema autosave, the object-type-free definition archive, and price-variable / variant bulk export-import with the duplicate caveat; semi-inheritance of a parent lab's templates through the parent's Super Admin, with the editable auto-populated product code; and a warning that a staging-only change to admin hosting and paths will invalidate the admin URLs recorded here. Source: claude-chat, fireflies-call, slack-message.
- 2026-08-14: Added Gift Vouchers under Marketing — the section `02_RETRIEVAL_MAP.md` already routed to but which did not exist in this file. Documented that a voucher's value can be updated after creation via the API, and that voucher codes can be printed into fulfillment output for in-store redemption tracking. Removed a stray closing code fence at end of file. Source: slack-message (#development, 2026-08-14), fireflies-call (2026-08-10).
- 2026-09-16: Added the Settings → Custom Fields page (site-wide definition list with all-or-subset export/import) and custom type definition export/import from the Custom Types index (definitions only, not instances). Object-type handling in the new export pending confirmation. Source: notion-page (Dashboard).
- 2026-09-19: Added Automatic Discounts to the Marketing section, with the note that it is not under Promotions. Source: AdeB.
- 2026-09-24: Resolved the pending question on the Settings → Custom Fields site-wide export: it carries owner_type (mapping listed) and re-import is refused, never updating. Added Website Settings Worth Knowing (Crawler, robots.txt, Redirects JSON shape, Search CMS, super-admin password reset bug) and the Pixfizz Kiosk app idle timeout. Source: claude-chat, fireflies-call, slack-message.
- 2026-09-29: Admin now on admin.pixfizz.com/site/<slug>/admin (site /admin 301s there); htmx conversion; API unchanged. Replaced the 'not shipped' warning. Users: API Keys section on each user page, and the user page section order. Published Products: 'Add Product' button renamed 'Publish Product'. Templates: custom_script is an Ace field saved by the Custom Fields Save button; stored with CRLF. Corrected the variant bundle re-import bullet (never updates in place); added Bulk Update Tools (copy variants, template options, design options across live templates) and the per-template edit routes. Added Admin Login: TOTP via login.pixfizz.com, passkeys (device-bound, required once registered unless an authenticator app is also registered), 30-minute impersonation timeout. Super Admin moved to admin.pixfizz.com/superadmin; old login.pixfizz.com/superadmin links are dead. Source: claude-chat, slack-message.
