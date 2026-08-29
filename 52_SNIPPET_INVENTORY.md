# 52 — Shopper Snippet Inventory

**Authority Scope:** Complete inventory of every snippet on the Shopper parent template (`shopper24.pixfizz.com`). Use this to identify snippet names, understand what renders what, and determine which snippet a child site needs to override.

_Source: CMS backup 2026-05-27. Total: 926 snippets across 23 namespaces._

_Last updated: 2026-05-27_

---

## How to Use This File

- To find a snippet by what it does: scan the namespace that matches (e.g. collection page issue → `collection/`, cart issue → `checkout/`).
- To identify which snippet renders a specific piece of UI: check the rendering chain notes at the top of each namespace section.
- To override a snippet on a child site: the snippet must exist on the parent (this list). Child sites can only override existing parent snippets — they cannot create new ones.
- Snippet paths use `/` notation. In the CMS filesystem, `/` is stored as `__` (e.g. `collection/banner` → `collection__banner`).

---

## Critical Rendering Chains

These are the most commonly needed "what renders what" chains. Start here before scanning individual snippets.

### Collection page → product cards

```
Page: shop/:collection-level-1
  → Routes to one of:
    - collection/collection-filters        (standard — most common)
    - collection/collection-load-more      (if collection.custom.load_more)
    - product/details-filter-dual-mode     (if collection.custom.pdp_layout)
    - product/product-details-prints       (if collection.custom.print_ux)
```

**Product cards on collection grids are rendered INLINE within `collection/collection-filters` and `collection/collection-load-more` — NOT as separate snippets.** The `product/cards/*` snippets exist but are not called from the main collection grid. They are standalone card components available for use in custom sections or dynamic blocks.

### "As low as" pricing on collection cards

Already built into the parent `collection/collection-filters` snippet. The pricing block checks `product.custom.from_pricing` — if populated, renders "As low as {price}" automatically. No override needed; just populate the `from_pricing` custom field on the product.

Pricing display priority on design product cards:
1. `product.custom.from_pricing` → "As low as {price}"
2. `product.custom.regular_pricing` → strikethrough + SALE badge + current price
3. `product.custom.custom_pricing` → free-text pricing string
4. Fallback → `product.price | currency`

### Navigation

```
Layout: index
  → admin/checklist/header-logo-position value determines nav style:
    - "LEFT"   → navigation/style1
    - "CENTER" → navigation/style3  (default)
```

### Product detail page

```
Page: product/:collection/:url-path
  → product/product-details              (standard)
  → product/product-details-filter       (with sidebar filters)
  → product/product-details-prints       (photo prints UX)
  → product/details-filter-dual-mode     (dual design+static)
```

---

## Root-Level Snippets (5)

These sit outside any namespace.

| Snippet | Description |
|---|---|
| `back-to-top` | Back-to-top scroll button |
| `calendar-date-form` | Calendar product date selection form |
| `footer` | Full site footer — newsletter signup, logo, social links, 4 columns |
| `gdpr-banner` | GDPR/cookie consent banner overlay |
| `html.head` | Global `<head>` content — analytics scripts, meta defaults, favicon, fonts, CSS includes |

---

## `account/` — Customer Account Pages (32 snippets)

Components for the customer account area (`/site/account-*` pages).

| Snippet | Description |
|---|---|
| `account/address-form` | Address form with US & Canadian state dropdowns |
| `account/custom-order-details` | Custom order details display |
| `account/custom-registration` | Custom registration form fields |
| `account/extra-group-project-code` | Extra code injection for group projects |
| `account/extra-order-code` | Extra code injection in order context |
| `account/extra-orderline-code` | Extra code injection per orderline |
| `account/galleries-index-v1` | Gallery listing page (v1) |
| `account/galleries-index-v2` | Gallery listing page (v2) |
| `account/gallery-item-modal` | Gallery item detail modal |
| `account/gallery-preview` | Gallery preview card (v1) |
| `account/gallery-preview-v2` | Gallery preview card (v2) |
| `account/gallery-shop-preview` | Gallery shop preview card |
| `account/gallery-show-v1` | Gallery detail page (v1) |
| `account/gallery-show-v2` | Gallery detail page (v2) |
| `account/home-login-form` | Home page login gate form |
| `account/login` | Login page content |
| `account/login-form` | Reusable login form component |
| `account/mailer_label` | Film mailer label for print |
| `account/navigation` | Account sidebar navigation |
| `account/password-reset-sent` | Password reset confirmation message |
| `account/personal-info` | Personal info edit form |
| `account/v2/addresses` | Account v2 — saved addresses |
| `account/v2/carts` | Account v2 — saved carts |
| `account/v2/dashboard` | Account v2 — dashboard overview |
| `account/v2/dates` | Account v2 — personal dates |
| `account/v2/empty-state` | Account v2 — empty state placeholder |
| `account/v2/galleries` | Account v2 — galleries listing |
| `account/v2/info` | Account v2 — personal info |
| `account/v2/order-details` | Account v2 — order detail view |
| `account/v2/orders` | Account v2 — order history |
| `account/v2/projects` | Account v2 — saved projects |
| `account/v2/sidebar` | Account v2 — sidebar navigation |

**Version note:** Account v2 snippets are gated by the `admin/checklist/account-v2` flag. Both v1 and v2 coexist on the parent.

---

## `admin/` — Admin Components (246 snippets)

### `admin/checklist/` — Feature Flags & Config (198 snippets)

Fully documented in `50_SHOPPER_TEMPLATE_REFERENCE.md` Section 5. Not repeated here. Key groups: account settings, cart/checkout, delivery, galleries, icons, kiosk mode, payment gateways, photo print sizes, product display, SEO, content pages.

### Other `admin/` Snippets (48 snippets)

| Snippet | Description |
|---|---|
| `admin/checklist-confirm-start-date` | Confirm start date form component |
| `admin/contribution-label` | Contribution/donation label text |
| `admin/forms/asset` | Admin form — asset upload field |
| `admin/forms/checkbox` | Admin form — checkbox field |
| `admin/forms/radio-button` | Admin form — single radio button |
| `admin/forms/radio-buttons` | Admin form — radio button group |
| `admin/forms/snippet` | Admin form — snippet editor field |
| `admin/guest-or-create-account-label` | Guest vs create account label text |
| `admin/header` | Admin page header bar |
| `admin/modals/favicon` | Favicon upload modal |
| `admin/modals/logo` | Logo upload modal |
| `admin/modals/ogimage` | OG image upload modal |
| `admin/order-filters` | Order management filter controls |
| `admin/orders/custom-filters` | Custom order filter definitions |
| `admin/orders/date-format` | Order date display format |
| `admin/orders/table-column4` | Custom 4th column in order table |
| `admin/orders/table-header` | Order table header row |
| `admin/pagination` | Admin list pagination |
| `admin/sidenav` | Admin sidebar navigation (full) |
| `admin/sidenav-clean` | Admin sidebar navigation (minimal) |
| `admin/starting-at-label` | "Starting at" label text |
| `admin/urlroot` | Admin URL root path |
| `admin/vat-rate` | VAT rate value |

---

## `checkout/` — Checkout Components (47 snippets)

| Snippet | Description |
|---|---|
| `checkout/authorizedotnet-checkout` | Authorize.net on-page credit card form |
| `checkout/available-payment-methods` | Payment method selector |
| `checkout/braintree-checkout` | Braintree on-page credit card form |
| `checkout/bridgepay-checkout` | BridgePay on-page credit card form |
| `checkout/cart-extra-product-details` | Additional content below product name in cart |
| `checkout/cart-note` | Cart notes text area |
| `checkout/cashondelivery` | Cash on delivery / pay-in-store option |
| `checkout/checkout-list-addresses` | List of user's saved addresses |
| `checkout/checkout-standard-addressform` | Single address form for checkout |
| `checkout/confirm_order_invoice` | Order confirmation with invoice |
| `checkout/cross-sell-collection` | Cross-sell product suggestions at checkout |
| `checkout/custom-cart-option` | Custom cart option field |
| `checkout/disclaimer` | Checkout disclaimer (forced acknowledgment) |
| `checkout/extra-cart-code` | Extra code injection in cart |
| `checkout/extra-cart-orderline-code` | Extra code injection per cart orderline |
| `checkout/extra-checkout-code` | Extra code injection at checkout |
| `checkout/extra-orderline-code` | Extra code injection per orderline |
| `checkout/film-processing-note` | Film processing drop-off/mail-in note |
| `checkout/guest-phone` | Guest checkout phone number field |
| `checkout/login-form` | Checkout login form |
| `checkout/min-charge-note` | Minimum charge notice |
| `checkout/no-rush-button` | Standard processing button (non-rush) |
| `checkout/only-group-redirect-message` | Group order redirect message |
| `checkout/payment-failed` | Payment failed page content |
| `checkout/payment-success` | Payment success page content |
| `checkout/pickup-contact-details` | Pickup contact details form |
| `checkout/prepay-only-message` | Prepay required message |
| `checkout/proceed-to-checkout` | Proceed to checkout button/section |
| `checkout/public-list-addresses` | Public address list (guest checkout) |
| `checkout/rush-button` | Rush processing button (primary) |
| `checkout/rush-button2` | Rush processing button (secondary/special) |
| `checkout/rush-disclaimer` | Rush processing disclaimer text |
| `checkout/rush-label` | Rush processing label |
| `checkout/rush-title` | Rush processing section title |
| `checkout/shipping-details` | Shipping address review section |
| `checkout/shipping-film-order-note` | Film order shipping note |
| `checkout/shipping-options` | Shipping method options |
| `checkout/shipping-service-extra-note` | Additional shipping service note |
| `checkout/shipping-unavailable-text` | Shipping unavailable message |
| `checkout/square-checkout` | Square on-page credit card form |
| `checkout/states-canada` | Canadian province dropdown options |
| `checkout/states-india` | Indian state dropdown options |
| `checkout/states-usa` | US state dropdown options |
| `checkout/states-usa-custom` | Custom US state dropdown |
| `checkout/stripe-checkout` | Stripe on-page credit card form |
| `checkout/trust-badge` | Checkout trust badge |
| `checkout/utm-cart-code` | UTM parameter capture for cart |

---

## `collection/` — Collection Page Components (10 snippets)

| Snippet | Description |
|---|---|
| `collection/banner` | Collection banner image display |
| `collection/collection-filters` | **Main collection grid with sidebar filters — contains inline product card rendering** |
| `collection/collection-filters-collapse` | Collection filters with collapsible panels |
| `collection/collection-filters-static` | Static (non-interactive) filter display |
| `collection/collection-filters-top` | Horizontal filter bar above grid |
| `collection/collection-load-more` | Collection grid with "load more" pagination (contains inline card rendering) |
| `collection/description` | Collection description display |
| `collection/shop-all` | "Shop all" collection page variant |
| `collection/sub-collection` | Subcollection card display |
| `collection/top-banner` | Full-width banner above collection grid |

**Key pattern:** `collection/collection-filters` is the default collection page renderer. It's called from `shop/:collection-level-1` (and level 2/3 variants). Product cards are rendered inline — not as separate snippet calls.

---

## `cookie-consent/` — Cookie Consent (2 snippets)

| Snippet | Description |
|---|---|
| `cookie-consent/banner` | Cookie consent banner content |
| `cookie-consent/show-hide` | Set to TRUE to enable cookie banner |

---

## `editor/` — Design Tool (1 snippet)

| Snippet | Description |
|---|---|
| `editor/scripts.js` | Custom JS injected into the design tool editor iframe |

---

## `email-notifications/` — Legacy Email System (13 snippets)

| Snippet | Description |
|---|---|
| `email-notifications/button` | Email CTA button component |
| `email-notifications/content/abandoned-cart` | Abandoned cart email content |
| `email-notifications/content/order-confirmed` | Order confirmation email content |
| `email-notifications/content/order-pending` | Order pending email content |
| `email-notifications/content/order-shipped` | Order shipped email content |
| `email-notifications/content/orderline-fulfilled` | Orderline fulfilled (gift card) email content |
| `email-notifications/content/password-reset` | Password reset email content |
| `email-notifications/content/user-signup` | New user signup email content |
| `email-notifications/header` | Email header component |
| `email-notifications/layout` | Email layout wrapper (legacy) |
| `email-notifications/layout-new` | Email layout wrapper (updated) |
| `email-notifications/logo` | Email logo component |
| `email-notifications/social-links` | Email social links footer |

---

## `email-shopper/` — Current Email System (16 snippets)

| Snippet | Description |
|---|---|
| `email-shopper/banner` | Email banner image |
| `email-shopper/header` | Email header component |
| `email-shopper/layout` | Email layout wrapper |
| `email-shopper/logo` | Email logo component |
| `email-shopper/style/background-light` | Email background color token |
| `email-shopper/style/brand-color` | Email brand color token |
| `email-shopper/style/muted-color` | Email muted text color token |
| `email-shopper/style/text-color` | Email body text color token |
| `email-shopper/templates/abandoned-cart` | Abandoned cart email template |
| `email-shopper/templates/gift-voucher` | Gift voucher email template |
| `email-shopper/templates/order-confirmed` | Order confirmation email template |
| `email-shopper/templates/order-draft` | Draft order email template |
| `email-shopper/templates/order-shipped` | Order shipped email template |
| `email-shopper/templates/password-reset` | Password reset email template |
| `email-shopper/templates/user-signup` | User signup email template |
| `email-shopper/trust-block` | Email trust/credibility block |

---

## `footer/` — Footer (1 snippet beyond root `footer`)

| Snippet | Description |
|---|---|
| `footer/logo` | Footer logo (usually inverted/white for dark background) |

---

## `header/` — Header Components (7 snippets)

| Snippet | Description |
|---|---|
| `header/bottom-promotion` | Bottom promotion content (below nav) |
| `header/bottom-promotion-bar` | Bottom promotion bar HTML |
| `header/logo` | Site logo image and alt tag |
| `header/logo-height-desktop` | Logo height for desktop |
| `header/logo-height-mobile` | Logo height for mobile |
| `header/promotion` | Top promotion bar message text |
| `header/promotion-bar` | Top promotion bar HTML wrapper |

---

## `helpers/` — Utility Snippets (1 snippet)

| Snippet | Description |
|---|---|
| `helpers/is-kiosk-mode` | Renders `TRUE` if the current session is in kiosk mode |

---

## `icons/` — SVG Icon Library (182 snippets)

Full icon library. Each snippet contains a single SVG icon. Called via `{% snippet 'icons/name.svg' %}` with optional `class` parameter.

Icons include: account, arrow, bag, basket, calendar, camera, cart, check, chevron, close, edit, eye, social media (facebook, instagram, linkedin, pinterest, tiktok, twitter, youtube), file types, navigation, e-commerce (credit-card, gift, ship, store, tag, truck), and miscellaneous (award, badge-check, brush, building-shield, crop, download, flag-usa, frame, graduation, golf, etc.).

Not listed individually — 182 SVG icons. Browse the namespace or search by icon name.

---

## `integrations/` — Third-Party Integrations (27 snippets)

| Snippet | Description |
|---|---|
| `integrations/ahrefs/script` | Ahrefs analytics script |
| `integrations/chatbot/script` | AI chatbot script injection |
| `integrations/constant-contact/universal-code` | Constant Contact embed code |
| `integrations/custom-body-scripts` | Custom scripts injected at end of `<body>` |
| `integrations/custom-links` | Custom `<link>` tags in `<head>` |
| `integrations/google/ai-widget` | Google AI widget |
| `integrations/google/event/add-to-cart` | GA4 add-to-cart event |
| `integrations/google/event/begin-checkout` | GA4 begin-checkout event |
| `integrations/google/event/purchase` | GA4 purchase event |
| `integrations/google/event/view-item` | GA4 view-item event |
| `integrations/google/gtag` | Google Analytics tag (measurement ID) |
| `integrations/google/rating-widget` | Google rating widget (small) |
| `integrations/google/reviews-widget` | Google reviews widget (full) |
| `integrations/google/tag-manager` | GTM container ID |
| `integrations/google/utm-capture` | UTM parameter capture script |
| `integrations/klaviyo/added-to-cart` | Klaviyo add-to-cart event |
| `integrations/klaviyo/api-key` | Klaviyo public API key |
| `integrations/klaviyo/viewed-product` | Klaviyo viewed-product event |
| `integrations/la_cameras_dealercode` | LA Cameras dealer code |
| `integrations/sharemechat/script` | ShareMe.Chat widget script |
| `integrations/stamped.io/api-key` | Stamped.io public API key |
| `integrations/stamped.io/collection-review-stars` | Stamped review stars on collection cards |
| `integrations/stamped.io/collection-review-widget` | Stamped review widget on collection pages |
| `integrations/stamped.io/product-review-stars` | Stamped review stars on product pages |
| `integrations/stamped.io/product-review-widget` | Stamped review widget on product pages |
| `integrations/stamped.io/script` | Stamped.io base script |
| `integrations/stripe/apple-pay-verification` | Apple Pay domain verification file content |

---

## `kiosk/` — Kiosk Mode Components (3 snippets)

| Snippet | Description |
|---|---|
| `kiosk/home` | Kiosk home screen (product tiles) |
| `kiosk/idle-screen` | Kiosk idle/attractor screen |
| `kiosk/top-rail` | Kiosk top navigation rail |

---

## `modals/` — Modal Dialogs (18 snippets)

| Snippet | Description |
|---|---|
| `modals/cart-notification` | Cart add notification toast |
| `modals/editor-help-guide` | Design tool help guide modal |
| `modals/login` | Login modal |
| `modals/modal-add-gallery` | Add gallery modal (v1) |
| `modals/modal-add-gallery-v2` | Add gallery modal (v2) |
| `modals/modal-edit-gallery` | Edit gallery modal (v1) |
| `modals/modal-edit-gallery-v2` | Edit gallery modal (v2) |
| `modals/orderline` | Orderline detail modal |
| `modals/password-reset` | Password reset modal |
| `modals/product` | Product quickview modal (TBC) |
| `modals/promotions` | Promotions fly-out sidebar |
| `modals/proof` | Proof approval modal |
| `modals/search` | Site search modal |
| `modals/ship-order` | Ship order admin modal |
| `modals/shopping-cart` | Shopping cart fly-out sidebar |
| `modals/skip-proof` | Skip proof confirmation modal |
| `modals/upload` | File upload modal |
| `modals/warning` | Generic warning modal |

**Note:** Modals listed in the `index` layout (password-reset, shopping-cart, cart-notification, search, upload, login, warning, promotions, proof, skip-proof) are rendered globally on every page. Do not remove them.

---

## `navigation/` — Navigation Components (43 snippets)

### Core Navigation

| Snippet | Description |
|---|---|
| `navigation/back-to-top` | Back-to-top button |
| `navigation/beta` | Beta navigation variant |
| `navigation/blog-breadcrumbs` | Blog-specific breadcrumbs |
| `navigation/breadcrumbs` | Non-product page breadcrumbs |
| `navigation/cart-icon` | Cart icon with item count badge |
| `navigation/dropdown` | Standard single-column dropdown menu |
| `navigation/logo-left` | Logo-left navigation variant |
| `navigation/megamenu` | Megamenu wrapper/shell |
| `navigation/pagination` | Page pagination controls |
| `navigation/style1` | Nav style 1: logo left, center menu, icons right |
| `navigation/style2` | Nav style 2: menu left, center logo, icons right |
| `navigation/style3` | Nav style 3 (default): 2 rows, center logo, main nav bottom row |
| `navigation/style4` | Nav style 4: 2 rows, left logo, main nav bottom row |
| `navigation/style5` | Nav style 5 |
| `navigation/user-icon` | User account icon |

### Megamenu Panels (28 snippets)

Each panel is a full-width dropdown for a specific product category. Registered in the `navigation_links` capture block of the active nav style snippet.

`navigation/megamenu/all-products`, `archiving`, `art-services`, `bound-products`, `business`, `calendars`, `cards`, `cards-calendars`, `create`, `custom`, `digitize-media`, `digitizing`, `education`, `film`, `film-cameras`, `gifts`, `leaflets`, `photo-books`, `press-printing`, `print-services`, `prints`, `services`, `sports-events`, `stationery`, `studio`, `wall-art`, `wall-decor`, `wide-format-simple`

---

## `product/` — Product Page Components (65 snippets)

### Product Cards (8 snippets)

Standalone card components. **Not used by the main collection grid** (which renders cards inline). Available for custom sections, dynamic blocks, or future use.

| Snippet | Description |
|---|---|
| `product/cards/double-image` | Two-image card with hover swap |
| `product/cards/double-image-hover-actions` | Two-image card with hover action icons |
| `product/cards/double-image-hover-button` | Two-image card with hover button |
| `product/cards/dynamic-collection` | Dynamic collection card (for carousel/block sections) |
| `product/cards/dynamic-design-product` | Dynamic design product card (for carousel/block sections) |
| `product/cards/single-image` | Single-image card |
| `product/cards/single-image-hover-actions` | Single-image card with quickview icon on hover |
| `product/cards/single-image-hover-button` | Single-image card with hover button |

### Product Detail Pages (5 snippets)

| Snippet | Description |
|---|---|
| `product/product-details` | Master product detail page (standard PDP) |
| `product/product-details-filter` | Product page with sidebar filter controls |
| `product/product-details-prints` | Product page for photo prints UX |
| `product/details-filter-dual-mode` | Dual-mode PDP supporting both design & static products |
| `product/product-quickview` | Product quickview modal content |

### Product Sub-Components

| Snippet | Description |
|---|---|
| `product/approve-selection-checkbox` | Photo selection approval checkbox |
| `product/badges` | Product badge rendering |
| `product/breadcrumb-schema` | Breadcrumb structured data |
| `product/breadcrumbs` | Product page breadcrumbs |
| `product/canonical/collection-name` | Canonical collection name |
| `product/canonical/collection-path` | Canonical collection path |
| `product/canonical/image-url` | Canonical image URL |
| `product/canonical/review-id` | Canonical review ID |
| `product/canonical/review-image-url` | Canonical review image URL |
| `product/canonical/review-url` | Canonical review URL |
| `product/canonical/sku` | Canonical SKU |
| `product/canonical/url` | Canonical product URL |
| `product/custom-breadcrumb` | Custom breadcrumb override |
| `product/custom-prints-code` | Custom photo prints code injection |
| `product/custom-scripts` | Custom scripts on product page |
| `product/description` | Product description display |
| `product/design-now` | Design Now form (launches Design Tool) |
| `product/extra-prints-code` | Extra photo prints code injection |
| `product/features` | Product features tab content |
| `product/filter-controls` | Product filter dropdown/radio rendering |
| `product/gallery/standard` | Standard product gallery (preview images + live previews) |
| `product/gallery/testing` | Gallery testing variant |
| `product/header_instructions_photo_selection` | Photo selection page instructions |
| `product/inventory` | Product inventory display |
| `product/name` | Product name rendering |
| `product/photo-prints` | Photo prints UI |
| `product/photo-prints-design-product` | Photo prints for design products |
| `product/photo-prints-option` | Photo prints option selector |
| `product/photo-prints-path` | Photo prints URL path logic |
| `product/pricing` | Product pricing display |
| `product/pricing-prefix` | Pricing prefix text ("Starting at", "From") |
| `product/production-time` | Production time display |
| `product/production_time` | Production time display (alternate) |
| `product/px-option` | Option/variant renderer (dropdowns & radio buttons) |
| `product/px-option-cart` | Option/variant renderer for cart context |
| `product/px-option-selector` | Cart option selector (editable variants) |
| `product/px-option-selector-alt` | Cart option selector (specific options) |
| `product/px-options` | Master options renderer (loops all product options) |
| `product/px-options-shop` | Color swatch display on collection cards |
| `product/px-pdf-detection` | PDF upload detection logic |
| `product/schema` | Product structured data (JSON-LD) |
| `product/select-date` | Date selection for calendar products |
| `product/select-website-calendars` | Website calendar selection |
| `product/service-schema` | Service structured data |
| `product/shipping-available` | Shipping availability display |
| `product/subheader` | Product page subheader |

---

## `sections/` — Page Sections (73 snippets)

### Dynamic Sections (7 snippets)

Re-inject into DOM on AJAX updates. Use `style onload` pattern for JS.

| Snippet | Description |
|---|---|
| `sections/dynamic/blog` | Blog post listing (from Custom Types) |
| `sections/dynamic/carousel-products` | Product carousel (AJAX-safe) |
| `sections/dynamic/collection-block` | Collection product block (first 8 designs by default) |
| `sections/dynamic/free_shipping_progress_bar` | Free shipping progress bar |
| `sections/dynamic/product-carousel` | Product carousel variant |
| `sections/dynamic/product-description-tabs` | Tabbed product description section |
| `sections/dynamic/services` | Services listing (from Custom Types) |

### Static Sections (52 snippets)

Reusable homepage/content sections. Include in `pages/__home` or relevant pages.

`sections/static/2-column-cards`, `2-column-collection-spotlight` (+ `-2`), `2-photo-feature` (+ `-2`), `3-column-cards` (+ `-1`, `-2`, `-2nd-row`), `3-features`, `3-steps`, `4-block-best-sellers`, `4-company-values`, `6-across`, `6-categories`, `7-across`, `8-feature-spotlights`, `8-photo-feature`, `about`, `banner-alt`, `carousel`, `carousel-features`, `carousel-products`, `checklist-feature`, `coming-soon`, `countdown-promo-fullwidth`, `custom`, `discover-more-ways`, `features`, `hero-3-columns-fullwidth` (+ `-2`), `hero-4-features`, `hero-carousel`, `home-page/3-columns-fullwidth`, `home-page/digitize-services-cards`, `home-page/hero-banner-fullwidth`, `home-page/trusted-brands`, `image-slider-comparison`, `newsletter`, `our-blog`, `parallax-banner`, `parallax-text-block`, `project-gallery`, `projects`, `promo-banner-color`, `promo-countdown`, `reviews`, `reviews-1-across`, `services-contact-footer`, `services/cards/audio-transfers`, `services/cards/movie-film`, `services/cards/photo-scanning`, `services/cards/vhs-tapes`, `shop-by-brand`, `top-item-feature`, `top-picks`

### Product-Specific Sections (5 snippets)

| Snippet | Description |
|---|---|
| `sections/product/canvas/details` | Canvas prints detail section |
| `sections/product/canvas/wrap-description` | Canvas wrap description |
| `sections/product/canvas/wrap-options` | Canvas wrap options |
| `sections/product/photo-gifts/landing-page` | Photo gifts landing page section |
| `sections/product/prints/details` | Photo prints detail section |
| `sections/product/prints/image-slider-comparison` | Before/after image comparison for prints |

### Other Sections (2 snippets)

| Snippet | Description |
|---|---|
| `sections/holiday` | Holiday/seasonal promotional section |
| `sections/homepage` | Homepage section wrapper |
| `sections/custom/cattle-catalogs-fall-2024` | Client-specific section (cattle catalogs fall 2024) |
| `sections/custom/cattle-catalogs-fall-2025` | Client-specific section (cattle catalogs fall 2025) |

---

## `services/` — Services Page Components (23 snippets)

### Service Cards (16 snippets)

Individual service cards for the services listing page.

`services/cards/audio`, `commercial-archiving`, `design`, `image-recovery`, `movie-film`, `negative-scanning`, `passport-photos`, `photo-classes`, `photo-restorations`, `photo-scanning`, `pre-owned`, `rentals`, `repairs`, `slide-scanning`, `specialty-scanning`, `vhs-tapes`

### Service Page Components (7 snippets)

| Snippet | Description |
|---|---|
| `services/copyright` | Copyright/licensing notice |
| `services/film-services` | Film processing services section |
| `services/gather-box` | Gather box CTA section |
| `services/hero-banner` | Services hero banner (4 images with text overlay) |
| `services/list` | Full services listing |
| `services/portrait-studio` | Portrait studio section |
| `services/safe-local` | Local/safe messaging section |

---

## `social-media/` — Social Media Links (9 snippets)

Each contains the URL to the site's social profile. Used in footer and OG tags.

`social-media/etsy`, `facebook`, `instagram`, `linkedin`, `open-graph`, `pinterest`, `tiktok`, `twitter`, `youtube`

---

## `staging/` — Staging (1 snippet)

| Snippet | Description |
|---|---|
| `staging/homepage` | Staging homepage content |

---

## `style/` — Theme Tokens (53 snippets)

Fully documented in `50_SHOPPER_TEMPLATE_REFERENCE.md` Section 4. Each snippet contains a single CSS value. Referenced inline by `pages/custom.css`.

**Colors (28):** `badge-bkg-color`, `badge-text-color`, `color-announcement-bar`, `color-background`, `color-bg-v-light`, `color-bullet-point`, `color-button-bkg`, `color-button-bkg-hover`, `color-button-label`, `color-button-label-hover`, `color-button-outline`, `color-button-outline-hover`, `color-font`, `color-font-hover`, `color-font-secondary`, `color-footer`, `color-highlight`, `color-highlight-light`, `color-pill-btn-outline`, `color-primary`, `color-promotion-bar`, `color-promotion-text`, `color-secondary`, `color-secondary-50`, `link-color`, `link-hover-color`, `nav-hover-bkg-color`, `nav-hover-text-color`, `navbar-hover-text-color`

**Fonts (5):** `body-font-size`, `body-font-weight`, `custom-body-font`, `fonts`, `nav-font-size`, `nav-font-weight`

**Borders/Radius (8):** `btn-border-radius`, `btn-pill-bkg-color`, `btn-pill-bkg-color-hover`, `btn-pill-border-radius`, `btn-pill-text-color`, `btn-pill-text-color-hover`, `color-swatch-border-radius`, `color-swatch-width`, `content-border-radius`, `pill-img-outline-width`, `text-input-border-radius`, `var-img-border-radius`

**Layout (3):** `footer-color-background`, `footer-color-font`, `header-class`, `scroll-to-top-color-background`

**Custom CSS (2):** `custom.css` (per-site custom CSS — blank on parent), `editor.css` (custom CSS for design tool)

---

## `website/` — Site-Level Data (48 snippets)

### Contact Information (16 snippets)

`website/contact/address`, `city`, `city-state`, `faq_path`, `form`, `geo-location`, `geo-map`, `location`, `meta_description`, `meta_title`, `services-telephone`, `services-telephone-label`, `sla-note`, `state`, `support-email-address`, `support-hours`, `support-telephone`, `support-telephone-label`, `support-text`, `title`, `zip-code`

### Content Pages (7 snippets)

| Snippet | Description |
|---|---|
| `website/404` | 404 page content |
| `website/blog_page` | Blog listing page content |
| `website/blog_post` | Individual blog post template |
| `website/contact_page` | Contact page content |
| `website/contact_thank_you` | Contact form thank you page |
| `website/faq` | FAQ page supplementary content |
| `website/faq_page` | FAQ page template |

### Site Settings (15 snippets)

| Snippet | Description |
|---|---|
| `website/brand` | Brand name/identity |
| `website/current-promotions` | Currently active promotions |
| `website/custom-css` | Legacy custom CSS (use `style/custom.css` instead) |
| `website/description` | Site meta description |
| `website/film-delivery-address` | Film mail-in delivery address |
| `website/gallery-preview` | Gallery preview configuration |
| `website/google-review-link` | Google review page link |
| `website/gtag` | Google Analytics measurement ID |
| `website/homepage` | Custom homepage snippet content |
| `website/llms.txt` | LLMs.txt content for AI crawlers |
| `website/meta-pixel` | Meta (Facebook) Pixel ID |
| `website/px-subdomain` | Pixfizz subdomain (required for AI chatbot) |
| `website/robots.txt` | Custom robots.txt content |
| `website/schema` | Homepage structured data (JSON-LD) |
| `website/shipping-returns-policy` | Shipping & returns policy content |
| `website/sitemap` | HTML sitemap content |
| `website/sitewide-promotion` | Sitewide promotion message |
| `website/terms_page` | Terms & conditions page content |
| `website/title` | Website title (suffix on all page titles) |
| `website/trust-badges` | Trust badges below Add to Cart |

---

## Annotations — Snippets Whose Row Does Not Tell the Whole Story

Added 2026-08-29 from live diagnosis. These correct or qualify rows above.

| Snippet | Annotation |
|---|---|
| `integrations/google/gtag` | **Orphaned — nothing includes it.** Its Description invites you to enter an Analytics account id, and editing it does nothing. The snippet that works is `website/gtag`. The same applies to `integrations/google/event/add-to-cart`, `.../begin-checkout` and `.../purchase`; keep only `.../view-item` as a no-GTM fallback. |
| `modals/shopping-cart` | This is the **cart fly-out**. Specs that name the fly-out as `shopper/cart-flyout` are wrong for this parent. It carries the custom-tool preview-code list, and as of 2026-08-27 that list was assigned as `flat_preview_codes` (underscore) and read as `flat-preview-codes` (hyphen), so it was never consulted — see 20_SHOPPER_CART_RULES.md. |
| `product/custom-prints-code` | Mount hook on the prints **collection landing page** (`/site/photo-prints` and collection-level prints pages, rendering `product/product-details-prints`). Not the flow itself. |
| `product/extra-prints-code` | Mount hook on `pages/prints` (`/site/prints?collection=...`), which renders `product/photo-prints` — the only page carrying `.px-photo-prints` and the `.px-btn-cart` button. Anything observing the prints component or its cart button belongs here. |
| `product/product-details-prints` | Does **not** render `product/design-now`, so photo-prints pages emit no `dataLayer` ecommerce event — see 50_SHOPPER_TEMPLATE_REFERENCE.md §20. |
| `kiosk/idle-screen` | Tests `has_logo != blank`, but the parent ships `update-website-logo` = `FALSE` and `'FALSE' != blank` is true, so the idle screen renders `header/logo` on sites that said they have none. Correct test is `has_logo == 'TRUE'`. |
| `admin/checklist/admin/checklist/kiosk-picker-idle-seconds` | A double-prefixed snippet path, created in error. Not a real setting. |

## Changelog

- 2026-05-27: Created from full CMS backup of `shopper24.pixfizz.com`. 926 snippets inventoried across 23 namespaces.
- 2026-08-29: Added an Annotations section correcting or qualifying seven rows — `integrations/google/gtag` and three sibling event snippets are orphaned (use `website/gtag`); `modals/shopping-cart` is the cart fly-out, not `shopper/cart-flyout`, and carried a never-consulted preview-code list; the two photo-prints mount hooks sit on different routes; `product/product-details-prints` emits no dataLayer event; the kiosk idle-screen logo test is inverted; and a double-prefixed checklist path exists in error. Source: claude-chat.
