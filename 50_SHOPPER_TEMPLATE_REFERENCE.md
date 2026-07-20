# 50 — Shopper Template Reference

**Authority Scope:** Structural anatomy of the Shopper parent template — layouts, navigation, snippets, theming, CSS delivery, and admin checklist system. Derived from a full CMS backup scan (2026-03-12).

_Last updated: 2026-06-01_

---

## What this file covers

This reference documents the **actual Shopper template structure** as found in the parent site backup. It covers:
- Layout files and when each is used
- Navigation system (styles, megamenu, checklist control)
- Snippet namespace conventions
- Theming system (style snippets + CSS delivery)
- Admin checklist system (how it works + full key inventory)
- JavaScript library stack
- Key page inventory

This is **structural knowledge** — not behavioral rules. For behavior and logic, consult files 20–22.

---

## 1. Layouts

Five layouts exist. The `index` layout is the default for all storefront pages.

| Layout | Used for |
|---|---|
| `index` | Default storefront layout (all public-facing pages) |
| `admin` | Custom admin pages (`/site/admin/*`) — admin-only, has side nav |
| `shopper-admin` | Shopper v2 custom admin (`/site/manage/*`) — admin-only, sidebar nav, requires `cms.js` + `cms.css` for form submission |
| `order-management` | Custom admin without side nav (standalone admin views) |
| `quickstart` | Pixfizz setup/onboarding wizard — admin-only, has full side nav |
| `iframe` | Modal/iframe content only |
| `feed_xml` | XML feed pages (product feeds) |

### `index` layout structure

The `index` layout assembles the storefront page in this order:

1. GTM noscript (if GTM enabled)
2. Global modals (always present — do not remove):
   - `modals/password-reset`
   - `modals/shopping-cart`
   - `modals/cart-notification`
   - `modals/search`
   - `modals/upload`
   - `modals/login`
   - `modals/warning`
   - `modals/promotions`
   - `modals/proof`
   - `modals/skip-proof`
3. Optional home-page login gate (`admin/checklist/home-page-login-form`)
4. Optional top promotion bar (`admin/checklist/top-promotion-bar`)
5. Navigation (determined by `admin/checklist/header-logo-position`)
6. `{{ page.content }}`
7. Optional back-to-top (`admin/checklist/back-to-top`)
8. Optional GDPR banner
9. Footer (`snippets/footer`)
10. Third-party scripts (ShareMe, chatbot, cookie consent, Klaviyo, Constant Contact)
11. JavaScript library stack

### JavaScript library stack (index layout)

All loaded via `asset_url` at the bottom of `<body>`:
- `jquery.min.js`
- `jquery.fancybox.min.js`
- `bootstrap.bundle.min.js`
- `flickity.pkgd.min.js`
- `highlight.pack.min.js`
- `jarallax.min.js`
- `list.min.js`
- `simplebar.js`
- `smooth-scroll.min.js`
- `flickity-fade.js`
- `theme.min.js`
- Then: `integrations/custom-body-scripts`

---

## 2. Navigation System

### Navigation styles

The layout selects a navigation snippet based on `admin/checklist/header-logo-position`:

| Checklist value | Snippet used | Layout |
|---|---|---|
| `LEFT` | `navigation/style1` | Logo left, center menu, icons right — single row |
| `CENTER` (default) | `navigation/style3` | Two rows: row 1 = logo center + social icons + account/cart; row 2 = main nav menu |
| `CUSTOM` | `navigation/style1` or `navigation/logo-left` | Conditional on `user.category == 'beta'` |

Additional styles exist (`style2`, `style4`, `style5`) but are not currently wired to the checklist — developmental/alternative variants.

### style1 — Logo Left (single row)

- Logo on the left
- Nav links centered (`mx-auto`)
- Account icon + cart icon on the right
- Suppresses nav when `admin/checklist/clean-checkout == 'TRUE'`
- Account dropdown shows: Custom Admin (admins only), Saved Projects, Orders, Galleries, Personal Info, Logout

### style3 — Two-Row Center Logo (default)

- Row 1: Social media icons (left, `d-none d-lg-flex`) + center logo (absolute positioned) + account/cart (right)
- Row 2 (`nav.main-menu`): Product navigation links centered
- Row 2 suppressed on `page.url == 'checkout-single-page'`
- Mobile: logo left + hamburger toggler; nav links collapse into mobile accordion
- Suppresses nav when `admin/checklist/clean-checkout == 'TRUE'`

### Navigation links (parent defaults)

Both style1 and style3 define the same default nav link set in a `{% capture navigation_links %}` block:

- Prints → `navigation/megamenu/prints`
- Wall Art → `navigation/megamenu/wall-art`
- Stationery → `navigation/megamenu/stationery`
- Photo Books → `navigation/megamenu/photo-books`
- Gifts → `navigation/megamenu/gifts`
- Lab Services → `navigation/megamenu/services`
- Business → `navigation/megamenu/business` (hidden: `d-none`)

**Client sites override this by editing the nav style snippet directly** — the `{% capture navigation_links %}` block at the top of the snippet is the right place to make those edits.

### `clean-checkout` flag

When `admin/checklist/clean-checkout == 'TRUE'`, both nav styles suppress the main navigation links, leaving only the logo and cart icon visible. Used for a distraction-free checkout experience.

---

## 3. Megamenu System

### How megamenus work

Each nav item is a Bootstrap dropdown with `position-static`. The dropdown content is a full-width panel (`dropdown-menu w-100`) containing a card with a container/row grid.

Standard megamenu structure:
```liquid
<div class="dropdown-menu w-100">
  <div class="card card-lg">
    <div class="card-body">
      <div class="tab-content">
        <div class="tab-pane fade show active" id="navTab">
          <div class="container">
            <div class="row justify-content-center">
              <!-- columns here -->
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

### Megamenu column patterns

**Text link column** (standard):
```liquid
<div class="col-12 col-md-2">
  <div class="mb-4 font-weight-bold"><b>Section Heading</b></div>
  <ul class="list-styled mb-6 mb-md-0 font-size-sm">
    <li class="list-styled-item">
      <a class="list-styled-link pl-3" href="/site/path">Link Text</a>
    </li>
  </ul>
</div>
```

**Image tile column** (desktop only, `d-none d-lg-block`):
```liquid
<div class="col-2 d-none d-lg-block">
  <div class="card">
    <a href="/site/path">
      <img class="card-img" src="{{ 'image.webp' | asset_url }}" alt="...">
    </a>
    <b style="font-size:0.9rem">Tile Heading</b>
    <span style="font-size:0.8rem">Tile subtext</span>
  </div>
</div>
```

### Available megamenu snippets (parent library)

These exist in the parent and can be used or adapted for client sites:

`navigation/megamenu/all-products`, `archiving`, `art-services`, `bound-products`, `business`, `calendars`, `cards`, `cards-calendars`, `create`, `custom`, `digitize-media` (blank — stub), `digitizing`, `education`, `film`, `film-cameras`, `gifts`, `leaflets`, `photo-books`, `press-printing`, `print-services`, `prints`, `services`, `sports-events`, `stationery`, `studio`, `wall-art`, `wall-decor`, `wide-format-simple`

### Simple dropdown (non-megamenu)

For single-column dropdowns, use `navigation/dropdown` — standard Bootstrap dropdown without the full-width card panel.

---

## 4. Theming System

### How theming works

Shopper uses a two-layer theming system:

1. **`style/*` snippets** — small snippets each containing a single value (color, size, weight, etc.). These are the source of truth for all theme values.
2. **`pages/custom.css`** — the CSS delivery page that outputs all theme-aware CSS. It includes `{% snippet 'style/custom.css' %}` plus a large block of CSS that references the style snippets inline via `{% snippet 'style/...' %}`.

### CSS delivery

- The CSS page at URL `/site/custom.css` is loaded in `html.head` via: `<link rel="stylesheet" href="/site/custom.css">`
- The snippet `style/custom.css` is **blank in the parent** — it is the stub where per-site custom CSS goes.
- **All CSS customisations go in `style/custom.css`** — this is what gets injected into the CSS page.
- Do NOT write CSS inline in Liquid/HTML snippets.

### Style snippet inventory

**Colors:**
- `color-primary` — primary brand color (buttons, link hover) — default: `#faa21b`
- `color-secondary` — secondary / dark button color — default: `#2d2d2d`
- `color-secondary-50` — secondary at reduced opacity
- `color-button-bkg` — button background — default: `#32c5ff`
- `color-button-bkg-hover` — button background on hover
- `color-button-label` — button text — default: `#ffffff`
- `color-button-label-hover` — button text on hover
- `color-button-outline` — button border color
- `color-button-outline-hover` — button border on hover
- `color-font` — body text color
- `color-font-secondary` — secondary body text color
- `color-font-hover` — body text hover
- `color-footer` — footer background (`.bg-dark`) — default: `#2d2d2d`
- `color-highlight` — highlight/accent (used in photo prints UI)
- `color-highlight-light` — lighter version of highlight
- `color-background` — page background color
- `color-bg-v-light` — very light background for sections (`.bg-very-light`)
- `color-announcement-bar` — top bar background (`.bg-light`)
- `color-promotion-bar` — promotion bar background
- `color-promotion-text` — promotion bar text
- `color-pill-btn-outline` — outline color for pill/variant buttons
- `color-bullet-point` — bullet point color in lists
- `link-color` — default link color
- `link-hover-color` — link hover color
- `nav-hover-bkg-color` — nav link hover background (animated underline effect)
- `nav-hover-text-color` — nav link hover text
- `navbar-hover-text-color` — active nav item text color
- `badge-bkg-color` — badge background
- `badge-text-color` — badge text

**Fonts:**
- `fonts` — custom font-face declarations (if any)
- `custom-body-font` — font-family string when `font-body == 'custom'`
- `body-font-size` — default: `1rem`
- `body-font-weight` — default: `400`
- `nav-font-size` — default: `0.9rem`
- `nav-font-weight` — default: `400`

**Borders & radius:**
- `btn-border-radius` — standard button border radius — default: `6px`
- `btn-pill-border-radius` — pill/variant button radius — default: `6px`
- `btn-pill-bkg-color` — pill button background
- `btn-pill-bkg-color-hover` — pill button hover background
- `btn-pill-text-color` — pill button text
- `btn-pill-text-color-hover` — pill button hover text
- `content-border-radius` — cards/content border radius — default: `6px`
- `text-input-border-radius` — form input border radius — default: `6px`
- `var-img-border-radius` — variant image border radius
- `color-swatch-border-radius` — color swatch border radius
- `color-swatch-width` — color swatch width
- `pill-img-outline-width` — outline width on selected variant image

**Logo & footer:**
- `header/logo` — rendered logo element
- `header/logo-height-desktop` — logo height on desktop
- `header/logo-height-mobile` — logo height on mobile
- `footer/logo` — rendered footer logo
- `footer-color-background` — footer section background
- `footer-color-font` — footer text color
- `scroll-to-top-color-background` — scroll-to-top button background

---

## 5. Admin Checklist System

### How it works

Each checklist key is a snippet at `admin/checklist/<key-name>`. The snippet contains a plain text value (e.g. `TRUE`, `FALSE`, a color, a domain name, or blank). The layout and other snippets `{% capture %}` these snippets and branch logic based on the value.

**Convention:**
- Boolean flags: `TRUE` or blank (blank = false/disabled)
- `FALSE` is also used explicitly in some cases
- Non-boolean settings contain the actual value (color hex, domain, font name, etc.)

### Full checklist key inventory

#### Navigation & Header
| Key | Values / Notes |
|---|---|
| `header-logo-position` | `LEFT` = style1; `CENTER` (default) = style3; `CUSTOM` = conditional |
| `top-promotion-bar` | `TRUE` = show promotion bar above nav |
| `bottom-promotion-bar` | `TRUE` = show promotion bar below nav |
| `back-to-top` | `TRUE` = show scroll-to-top button |
| `search` | `TRUE` = show search icon in nav |
| `cart-icon` | Controls cart icon style (see icon variants below) |
| `user-icon` | Controls user icon style (see icon variants below) |

**Cart icon style options:**
`bag-outline`, `bag-outline-thin`, `bag-solid`, `basket-outline`, `basket-outline-thin`, `basket-solid`, `cart-outline`, `cart-outline-thin`, `cart-sharp-outline`, `cart-sharp-outline-thin`, `cart-sharp-solid`, `cart-solid`

**User icon style options:**
`user-circle-outline`, `user-circle-outline-thin`, `user-circle-solid`, `user-outline`, `user-outline-thin`, `user-person-outline`, `user-person-outline-thin`, `user-person-solid`, `user-sharp-outline`, `user-sharp-outline-thin`, `user-sharp-solid`, `user-solid`

#### Cart Behavior
| Key | Values / Notes |
|---|---|
| `cart-editable-options` | `TRUE` = options editable directly in cart |
| `cart-note` | `TRUE` = show order note field in cart |
| `cart-show-text-options` | `TRUE` = show text options in cart display |
| `cart-top-continue-shopping-link` | `TRUE` = show "continue shopping" link |
| `hide-coupon-form-cart` | `TRUE` = hide promo code input in cart |
| `hide_pricing_cart` | `TRUE` = hide pricing in cart |
| `activate-cart-cross-sell-section` | `TRUE` = enable cross-sell section in cart |
| `activate-cart-product-url-path` | Controls product URL path in cart |
| `hide-template-options-from-cart` | `TRUE` = hide template options from cart display |

#### Checkout Policy
| Key | Values / Notes |
|---|---|
| `clean-checkout` | `TRUE` = suppress nav links (logo + cart only) |
| `guest-checkout` | `TRUE` = allow guest checkout (default: `TRUE`) |
| `disable-delivery` | `TRUE` = disable delivery option |
| `default-delivery-option` | Sets default delivery method |
| `display-shipping-options` | `TRUE` = show shipping options |
| `disable-user-registration` | `TRUE` = prevent new registrations |
| `checkout-disclaimer` | `TRUE` = show checkout disclaimer text |
| `checkout-rush` | `TRUE` = enable rush delivery option |
| `checkout-rush-special` | `TRUE` = enable special rush option |
| `checkout-column-positions` | Controls checkout layout column order |
| `cash-on-delivery` | `TRUE` = enable cash on delivery |
| `pay-in-store` | `TRUE` = enable pay in store |
| `pickup-in-store` | `TRUE` = enable pickup in store |
| `dont-require-pickup-contact-details` | `TRUE` = skip contact details for pickup |
| `note-contact-details-required` | `TRUE` = mark contact details as required |
| `require-billing-address` | `TRUE` = require billing address |
| `require-billing-address-company` | `TRUE` = require company name in billing |
| `require-last-name` | `TRUE` = require last name at checkout |
| `require-telephone` | `TRUE` = require telephone at checkout |
| `require-telephone-registration` | `TRUE` = require telephone at registration |
| `input-public-address` | Public/system address for digital-only orders |
| `billing-address-state-global` | `TRUE` = show state field globally |
| `minimum-charge` | Minimum order charge amount |
| `max-cart-total-pay-in-store` | Maximum total for pay in store |
| `confirm-start-date` | `TRUE` = require start date confirmation |
| `confirm_start_date_label` | Label for start date field |
| `confirm-with-invoice` | `TRUE` = confirm order with invoice |
| `payment-link` | `TRUE` = enable payment link option |
| `payment-gateway` | Active gateway: `stripe`, `braintree`, `square`, `authorize.net`, `paypal` — default: `stripe` |
| `digital-only-delivery` | `TRUE` = enable digital-only delivery mode |
| `proof-order-checkout` | `TRUE` = enable proof before checkout |
| `promocode-checkout` | `TRUE` = show promo code at checkout |
| `hide-film-order-checkout` | `TRUE` = hide film processing at checkout |
| `film-order-drop-off-id` | Film drop-off location ID |
| `film-order-drop-off-id-label` | Label for film drop-off field |
| `enable-film-mailer-label` | `TRUE` = enable film mailer label printing |
| `custom_checkout_condition` | Custom checkout condition logic |
| `custom_checkout_logic` | Custom checkout logic |

#### Kiosk Mode
| Key | Values / Notes |
|---|---|
| `kiosk-mode-enabled` | `TRUE` = enable kiosk mode |
| `kiosk-mode-domain` | Alternate domain for kiosk detection |
| `kiosk-pay-in-store-only` | `TRUE` = restrict pay-in-store to kiosk only |
| `kiosk-remove-captcha` | `TRUE` = remove CAPTCHA in kiosk mode |

**Kiosk captcha is per-subdomain.** Kiosk mode usually runs on its own subdomain (`kiosk-mode-domain`). CAPTCHA configuration does not carry across from the main storefront to the kiosk subdomain — captcha must be removed/configured on the kiosk subdomain specifically (e.g. `kiosk-remove-captcha` set on the kiosk site). Symptom if missed: customers hit a CAPTCHA on the kiosk that the main storefront does not show.

#### Product Display
| Key | Values / Notes |
|---|---|
| `pricing-display` | Controls pricing display style |
| `pricing-tab-position` | Position of pricing tab |
| `description-position` | Position of product description |
| `collection-description-position` | Position of collection description |
| `align-collection-card` | Card alignment in collection |
| `align-collection-card-center` | Center-align collection cards |
| `align-collection-card-left` | Left-align collection cards |
| `align-collection-title` | Title alignment in collection |
| `collection-card-shadow` | `TRUE` = add shadow to collection cards |
| `variant_columns` | Number of variant columns (desktop) |
| `variant_columns_mobile` | Number of variant columns (mobile) |
| `upload-btn-style` | Upload button style variant |
| `display-color-palette` | `TRUE` = show color palette option |
| `hide-color-label` | `TRUE` = hide color label |
| `bullet-point-style` | List bullet style (disc, circle, etc.) |
| `pill-btn-outline` | `TRUE` = use pill/outline button style |
| `display-print-original-filename` | `TRUE` = show original filename for prints |

#### Photo Prints
| Key | Values / Notes |
|---|---|
| `prints-autoselect` | `TRUE` = auto-select first print size |
| `prints-thumbnails` | `TRUE` = show print thumbnails |
| `prints-thumbnails-crop` | `TRUE` = crop thumbnails |
| `prints-thumbnails-photo` | `TRUE` = show photo thumbnails |
| `enable-image-filters-image-upload` | `TRUE` = enable image filters for upload options |
| `enable-image-color-image-upload` | `TRUE` = enable color adjustment for upload options |
| `enable-image-filters-photo-prints` | `TRUE` = enable filters for photo prints |
| `enable-image-color-photo-prints` | `TRUE` = enable color adjustment for photo prints |

#### Print Product Sizes
Standard prints: `product-print-3x5`, `4x5`, `4x6`, `4x8`, `5x5`, `5x7`, `6x8`, `6x9`, `8x8`, `8x10`, `10x10`, `10x13`, `10x15`, `11x14`, `12x12`, `product-print-no-bleed`

Enlargements/large format: `product-enlargements-bleed-1-8`, `product-enlargements-no-bleed`, `product-large-16x16`, `16x20`, `16x24`, `18x24`, `20x20`, `20x30`, `24x24`, `24x30`, `24x36`, `30x40`, `40x60`

#### Fonts
| Key | Values / Notes |
|---|---|
| `font-body` | `lato`, `open-sans`, `avenir`, `custom` — default: `avenir` |
| `font-lato` | Lato font activation flag |
| `font-open-sans` | Open Sans font activation flag |

#### Account & User
| Key | Values / Notes |
|---|---|
| `home-page-login-form` | `TRUE` = show login gate on home page |
| `account_nav_version` | Account navigation version |
| `account_saved_projects_version` | Saved projects version |
| `account_saved_projects_view` | Default view for saved projects |
| `hide-saved-projects` | `TRUE` = hide saved projects from account |
| `hide-galleries` | `TRUE` = hide galleries from account |
| `hide-personal-dates` | `TRUE` = hide personal dates |
| `gallery_version` | Gallery version (v1 or v2) |
| `gallery_tile_layout` | Gallery tile layout style |
| `gallery-thumb-position` | Gallery thumbnail position |
| `gallery-download` | `TRUE` = enable gallery download |
| `gallery-image-download` | `TRUE` = enable individual image download |
| `gallery-image-filename` | `TRUE` = show image filename in gallery |
| `gallery-order-prints` | `TRUE` = enable order prints from gallery |
| `custom-registration` | `TRUE` = use custom registration form |
| `multiple-personal-calendars` | `TRUE` = allow multiple personal calendars |

#### Integrations
| Key | Values / Notes |
|---|---|
| `setup-google-tag-manager` | GTM container ID |
| `activate-klaviyo` | `TRUE` = enable Klaviyo |
| `activate-constant-contact` | `TRUE` = enable Constant Contact |
| `activate-stamped` | `TRUE` = enable Stamped.io reviews |
| `activate-shareme` | `TRUE` = enable ShareMe chat |
| `activate-google-reviews-ai-widget-domain` | `TRUE` = enable Google AI reviews widget |
| `google-reviews-ai-widget-domain` | Domain for Google reviews widget |
| `activate-contributions` | `TRUE` = enable contributions feature |
| `activate-group-projects` | `TRUE` = enable group projects |
| `activate-free-shipping-progress-bar` | `TRUE` = enable free shipping progress bar |
| `pixfizz-ai-chatbot` | `TRUE` = enable Pixfizz AI chatbot |
| `pixfizz-ai-chatbot-admin-only` | `TRUE` = show chatbot to admins only |
| `activate-vat-rate` | `TRUE` = enable VAT rate display |

#### SEO & Metadata
| Key | Values / Notes |
|---|---|
| `no-index` | `TRUE` = add noindex to entire site |
| `seo-tdks` | SEO title/description/keywords settings |
| `update-website-title` | Website title |
| `update-website-description` | Website description |
| `update-website-domain` | Website domain |
| `update-website-logo` | Website logo asset |
| `update-branding-design-tool` | Branding in design tool |
| `schema_loop_all_products` | `TRUE` = include all products in schema |

#### Content & Pages
| Key | Values / Notes |
|---|---|
| `blog` | `TRUE` = enable blog section |
| `custom-blog-page` | `TRUE` = use custom blog page |
| `custom-blog-post` | `TRUE` = use custom blog post template |
| `custom-home-page` | `TRUE` = use custom home page |
| `custom-contact-page` | `TRUE` = use custom contact page |
| `custom-faq-page` | `TRUE` = use custom FAQ page |
| `custom-terms-page` | `TRUE` = use custom terms page |
| `gdpr-banner` | `TRUE` = show GDPR cookie banner |
| `hide-contact-business-page` | `TRUE` = hide contact on business pages |
| `hide-contact-service-page` | `TRUE` = hide contact on service pages |
| `date-format` | Date display format |
| `country-filter` | Country filter for shipping |
| `filters-sticky` | `TRUE` = sticky collection filters |

---

## 6. Snippet Namespace Conventions

Shopper uses a consistent `namespace/name` path convention. In the file system, `/` is stored as `__`.

| Namespace | Purpose |
|---|---|
| `admin/checklist/` | Feature flags and config values |
| `admin/forms/` | Admin form components |
| `checkout/` | Checkout-specific components |
| `collection/` | Collection/shop page components |
| `email-notifications/` | Legacy email template components |
| `email-shopper/` | Current Shopper email templates |
| `footer/` | Footer sub-components |
| `header/` | Header sub-components (logo, promotion bars) |
| `helpers/` | Utility snippets (e.g. `helpers/is-kiosk-mode`) |
| `icons/` | SVG icon snippets |
| `integrations/` | Third-party integration scripts |
| `modals/` | Modal dialogs |
| `navigation/` | Nav components |
| `navigation/megamenu/` | Individual megamenu panels |
| `product/` | Product page components |
| `product/cards/` | Product card variants |
| `product/details/` | Product detail components |
| `sections/custom/` | One-off custom sections |
| `sections/dynamic/` | Dynamic/AJAX sections |
| `sections/static/` | Reusable static sections |
| `services/` | Services page components |
| `services/cards/` | Service card variants |
| `social-media/` | Social media URLs and OG tags |
| `style/` | Theme variable snippets |
| `website/` | Site-level data (contact info, title, etc.) |
| `website/contact/` | Contact detail snippets |

---

## 7. Website Contact & Data Snippets

These snippets store site-specific content that varies per client:

- `website/title` — site name
- `website/description` — meta description
- `website/brand` — brand name (used in footer copyright)
- `website/contact/support-telephone`
- `website/contact/support-telephone-label`
- `website/contact/support-email-address`
- `website/contact/support-hours`
- `website/contact/support-text`
- `website/contact/address`
- `website/contact/city`, `city-state`, `state`, `zip-code`
- `website/contact/location`
- `website/contact/title`, `meta_title`, `meta_description`
- `website/contact/faq_path`
- `website/contact/geo-location`, `geo-map`
- `website/contact/services-telephone`, `services-telephone-label`
- `website/contact/sla-note`
- `website/google-review-link`
- `website/gtag` — Google Analytics 4 tag ID
- `website/meta-pixel` — Facebook Pixel ID
- **Google Ads conversion tracking:** there is no dedicated built-in preset or snippet for a Google Ads conversion tag in Shopper (only GTM, GA4, and Meta Pixel exist). Deploy the Google Ads site tag (`AW-...`) and the purchase conversion event through GTM using the existing `setup-google-tag-manager` key (conversion linker + conversion action tag + thank-you-page event). A hardcoded gtag conversion snippet, if used instead, is site-specific code with no checklist key reserved for it.
- `website/px-subdomain` — Pixfizz subdomain
- `website/film-delivery-address` — for film mail-in orders
- `website/trust-badges` — trust badge images
- `website/current-promotions` — promotional content
- `website/sitewide-promotion` — sitewide promo text

---

## 8. Footer Structure

The footer (`snippets/footer`) has two sections:

**Top section** (`py-6 py-md-12 border-bottom border-gray-700`):
- Newsletter signup (hidden by default — uses Klaviyo)
- 4-column grid: logo + social links | support (phone/email/hours) | resources (Contact, FAQs, Shipping, Order Status) | company (Our Story, Blog if enabled)

**Bottom bar** (`py-3 bg-dark`):
- Copyright line using `website/brand`
- "Powered by Pixfizz" logo (desktop only)
- Terms & Privacy + Sitemap links
- Payment logos: Mastercard, Visa, AMEX

Footer background: `style/color-footer` (`.bg-dark`) and `style/footer-color-background`. Social icons only render if their `social-media/<platform>` snippet is non-blank.

---

## 9. HTML Head (`html.head`)

Key elements in order:

1. GTM script (if `integrations/google/tag-manager` set)
2. GA4 gtag (if `website/gtag` set)
3. Facebook Pixel (if `website/meta-pixel` set)
4. Pixfizz CMS JS (`cms.js`) + CSS (`cms.css`) via `pixfizz_asset_url`
5. Prefetch for editor assets
6. Stamped.io script (if API key set)
7. Theme CSS: `flickity-fade.css`, `jquery.fancybox.min.css`, `flickity.min.css`, `vs2015.css`, `simplebar.min.css`, `theme.min.css`, `px-shopper.css`, `feather.css`
8. Custom CSS: `<link rel="stylesheet" href="/site/custom.css">`
9. Title, meta description
10. Open Graph tags (`social-media/open-graph`)
11. Canonical URL
12. Favicon
13. No-index if `admin/checklist/no-index == 'TRUE'`
14. Ahrefs script, custom links
15. Fonts (Google Fonts for Lato/Open Sans; custom via `style/fonts`)

---

## 10. Key Page Inventory

All pages live at `/site/<url>`.

**Account:** `account`, `account-address`, `account-address-edit`, `account-address-new`, `account-carts`, `account-galleries`, `account-galleries/<gallery>`, `account-orders`, `account-orders/details`, `account-personal-calendars`, `account-personal-dates`, `account-personal-info`, `account-saved-projects`

**Commerce:** `cart`, `checkout`, `checkout2`, `checkout-order`, `checkout-print`, `confirm`, `thank-you`, `payment_success`, `payment_failed`, `payment-link`, `draft-order/<order_id>`, `group_order_success`, `add-to-cart`

**Shop/Products:** `shop`, `shop/<collection>` (1–3 levels), `product/<path>` (1–3 collection levels + url-path), `photo-prints`, `prints`, `productview`, `project-edit`, `promotions`

**Auth:** `login`, `login-checkout`, `login/email-sent`, `password-reset`, `reset`

**Content:** `__home` (home page), `blog`, `blog/<post>`, `contact-us`, `contact-thankyou`, `faq`, `shipping-and-returns`, `terms-conditions-privacy`, `our-work`, `sections-gallery`, `services`, `services/<path>`, `business`, `business/<path>`, `gallery-shop`, `gallery-shop/<gallery>`, `sitemap`, `404`

**Special:** `custom.css` (CSS delivery), `editor-scripts.js`, `editor.css`, `robots.txt`, `feed/products.xml`, `feed/products-custom.xml`, `search/index.json`, `search/worker.js`, `order-management`, `setup/*` (admin setup wizard)

**Generic catch-alls:** `-page-path-1`, `-page-path-1/-page-path-2`, `-page-path-1/-page-path-2/-page-path-3`

---

## 11. Email Templates

Two parallel systems exist. The current system is `email-shopper/`.

**Templates:**
- `email-shopper/templates/abandoned-cart`
- `email-shopper/templates/order-confirmed`
- `email-shopper/templates/order-draft`
- `email-shopper/templates/order-shipped`
- `email-shopper/templates/password-reset`
- `email-shopper/templates/user-signup`

**Style sub-snippets:** `email-shopper/style/background-light`, `brand-color`, `muted-color`, `text-color`

**Layout components:** `email-shopper/layout`, `header`, `logo`, `banner`, `trust-block`, `social-links`

> Email templates run outside the storefront session. Project previews in email require `share: orderline.project.share_code`. See `40_PLAYBOOK.md`.

---

## 12. Section Library

**Static sections (`sections/static/`):**
`2-column-cards`, `2-column-collection-spotlight` (+ `-2`), `2-photo-feature` (+ `-2`), `3-column-cards` (+ `-1`, `-2`, `-2nd-row`), `3-features`, `3-steps`, `4-block-best-sellers`, `4-company-values`, `6-across`, `6-categories`, `7-across`, `8-feature-spotlights`, `8-photo-feature`, `about`, `banner-alt`, `carousel`, `carousel-features`, `carousel-products`, `checklist-feature`, `coming-soon`, `countdown-promo-fullwidth`, `custom`, `features`, `hero-3-columns-fullwidth` (+ `-2`), `hero-4-features`, `hero-carousel`, `home-page__3-columns-fullwidth`, `home-page__digitize-services-cards`, `home-page__hero-banner-fullwidth`, `home-page__trusted-brands`, `image-slider-comparison`, `newsletter`, `our-blog`, `parallax-banner`, `parallax-text-block`, `project-gallery`, `projects`, `promo-banner-color`, `promo-countdown`, `reviews`, `reviews-1-across`, `services-contact-footer`, `shop-by-brand`, `top-item-feature`, `top-picks`

**Dynamic sections (`sections/dynamic/`):**
`blog`, `carousel-products`, `collection-block`, `free_shipping_progress_bar`, `product-carousel`, `product-description-tabs`, `services`

Dynamic sections re-inject into the DOM on AJAX updates. Use the `style onload` pattern for any JS that must survive re-injection (see `01_CODE_GOVERNANCE.md`).

### Shop All page

A Shopper page that displays an image for every top-level collection, giving
shoppers a single visual entry point to all categories. Addresses the visual
navigation gap when a store has many top-level collections. Live as of mid 2026.
The exact page or snippet name should be confirmed against the live deployment or
with Matjaz.

---

## 13. Practical Notes for Development

- **Editing nav links:** Always check which nav style is active before editing. `admin/checklist/header-logo-position` value `LEFT` renders `navigation/style1`; `CENTER` (default) renders `navigation/style3`. Edit the `{% capture navigation_links %}` block at the top of the **active** snippet only — editing the wrong one has no effect. Do not edit the HTML structure below the capture block.- **Adding a megamenu panel:** Create or edit `navigation/megamenu/<name>`. Use the patterns in section 3 above. Register the nav item in the `navigation_links` capture block.
- **CSS changes:** Always in `style/custom.css` only. Never inline in Liquid/HTML.
- **Theme color changes:** Edit the relevant `style/<token>` snippet. The CSS page picks them up automatically.
- **Checklist changes:** Edit the `admin/checklist/<key>` snippet value. Boolean flags expect exactly `TRUE` or blank.
- **New sections:** Use an existing section snippet from `sections/static/` or create a new one. Include in `pages/__home` or the relevant page.
- **Email project previews:** Always include `share: orderline.project.share_code` in the preview URL.
- **Font changes:** Set `admin/checklist/font-body` to `lato`, `open-sans`, `avenir`, or `custom`. For `custom`, populate `style/custom-body-font` with the font-family string.
- **Shared snippets:** If a snippet is used across multiple client sites, follow the Shared Snippet Contract Rule in `01_CODE_GOVERNANCE.md` — do not remove existing variables, IDs, or JS hooks.
- **Custom home page content:** Place home page content in the snippet `website/homepage`. This snippet is only rendered when the **"Custom snippet (website/homepage) for home page"** checkbox is ticked in Custom Admin → Storefront Settings. Both the snippet and the checkbox are required — neither works without the other.

## 14. Creating Pages on Child Sites

Child sites of Shopper cannot create real CMS pages. New pages are
created by adding instances of the `pages` Custom Type instead.

**Navigation path:** Main Admin → Website tab → Custom Types → Pages

Create a new instance with these field values:
- `page_path` — must match the URL segment exactly (e.g. `graduation` → `/site/graduation`)
- `page_title` — page title
- `page_description` — snippet field, optional
- `page_content` — snippet field — main page content goes here
- `page_schema` — leave blank unless you need structured data / ld+json

No redirect needed. The page resolves automatically once the instance is saved.

How it works:
- Shopper has catch-all pages at `/:page-path-1` (and level 2/3 variants)
- The catch-all page looks up a `pages` Custom Type instance where
  `custom.page_path` matches the URL segment
- If no match → 404
- Page content renders from `custom_page.custom.page_content`
- Full CMS context is available inside the content snippet

Constraints:
- Real CMS pages always take priority over Custom Type instances on the
  same path — always use a path that has no real page equivalent
- The path is a single segment only (level 1) — use level 2/3 catch-all
  pages for nested paths if needed

## 15. Custom Admin — Shopper v2 (`/site/manage/`)

The Shopper v2 custom admin is a replacement for the legacy `setup/` wizard pages. It uses the `shopper-admin` layout and provides a sidebar navigation to all configuration pages.

### Access control

The `shopper-admin` layout includes a `{% if user.is_admin %}` gate. Only admin users can access `/site/manage/*` pages. Non-admin users see nothing.

### Critical dependency

The `shopper-admin` layout must include `cms.js` and `cms.css` (loaded via `pixfizz_asset_url`). Without `cms.js`, all `{% form %}` tags with `async: true` / `autosubmit: true` render as static HTML — checkboxes and snippet-saving forms will not submit.

### Page inventory

| Path | Content |
|---|---|
| `manage/dashboard` | Overview / home page |
| `manage/branding` | Colors, fonts, logo, brand identity |
| `manage/store` | Storefront settings, collection layout, product page options |
| `manage/homepage` | Homepage content configuration |
| `manage/navigation` | Nav links, megamenu, footer links |
| `manage/collections` | Collection display options |
| `manage/products` | Product page configuration |
| `manage/gallery` | Gallery feature settings |
| `manage/cart` | Cart page options |
| `manage/checkout` | Checkout flow, rush fees, shipping display |
| `manage/payments` | Payment gateway settings |
| `manage/account` | Customer account area configuration |
| `manage/seo` | SEO settings, meta defaults, llms.txt management |
| `manage/emails` | Email template configuration |
| `manage/integrations` | Third-party integrations (GTM, Klaviyo, etc.) |
| `manage/advanced` | Advanced settings |

### Tools pages

| Path | Content |
|---|---|
| `manage/tools/product-importer` | CSV-based static product importer |
| `manage/tools/download-images` | Live preview image downloader (per-collection ZIP download) |

#### Static Product Importer — CSV format

The Static Product Importer (`manage/tools/product-importer`) is a Shopper template-level
tool, not a Pixfizz Core feature. It is gated by the `shopper-admin` layout's admin check,
so only admin users can reach it. On upload, it creates products via the Pixfizz API and
assigns them to a selected collection. A blank CSV template can be downloaded from the tool
page itself. Recommended for stores with large static catalogues (standard print sizes,
fixed products without personalization).

CSV column order (the tool reads columns positionally):

`name, code, price, description, category, asset_image_name, fulfillment_code, track_inventory, current_inventory, tax_exempt, min_quantity, max_quantity`

| Column | Type | Notes |
|---|---|---|
| `name` | string | **Required** |
| `code` | string | **Required** — the product code/SKU |
| `price` | number/formula | **Required** |
| `description` | string | Optional |
| `category` | string | Optional |
| `asset_image_name` | string | Optional — filename of an asset already uploaded under Website > Assets |
| `fulfillment_code` | string | Optional |
| `track_inventory` | boolean | `"true"` / `"false"` |
| `current_inventory` | integer | Whole number |
| `tax_exempt` | boolean | `"true"` / `"false"` |
| `min_quantity` | integer | Whole number |
| `max_quantity` | integer | Whole number |

- A header row is optional and is skipped if present.
- This tool drives the same product-creation path as the Pixfizz API; it does not create
  personalization templates or designs, only static products.

### Sidebar navigation

The sidebar is defined in a shared snippet. When adding new pages, update the sidebar snippet with the new nav item. The sidebar uses the `shopper-admin` design system CSS classes (`s-card`, `s-field`, `s-field-label`, etc.).

---

## 16. Kiosk Touchscreen Mode

**Status:** Partially implemented. Login gate, idle screen, and cart/checkout overlays not yet built. Parked as of May 2026.

Kiosk mode is a checklist-gated feature designed for in-store photo lab kiosks. When enabled, it transforms the Shopper storefront into a touch-friendly, simplified UI for two primary use cases: ordering photo prints and submitting film processing orders.

### Architecture

- **Gate:** The `admin/checklist/kiosk-touchscreen-mode` snippet controls activation. When set to `TRUE`, the `index` layout adds the class `kiosk-touchscreen` to the `<body>` tag.
- **9 checklist snippets** created on the parent template:
  - `admin/checklist/kiosk-touchscreen-mode` — master toggle
  - `admin/checklist/kiosk-prints-collection` — collection path for the prints workflow
  - `admin/checklist/kiosk-film-collection` — collection path for film processing
  - `admin/checklist/kiosk-other-collection` — collection path for secondary products
  - `admin/checklist/kiosk-prints-image` — hero tile image for prints
  - `admin/checklist/kiosk-film-image` — hero tile image for film
  - `admin/checklist/kiosk-other-image` — hero tile image for other products
  - `admin/checklist/kiosk-prints-label` — tile label for prints
  - `admin/checklist/kiosk-film-label` — tile label for film

- **Content snippets:** `kiosk/top-rail` (simplified header bar with logo + Start Over button) and `kiosk/home` (tile-based landing page).

### CSS scoping

All kiosk CSS is scoped under `.kiosk-touchscreen` so it has zero impact when the mode is off. CSS lives in the child site's `style/custom.css`.

### UX constraints

- No navigation bar (hidden via CSS)
- Large buttons, large tiles — designed for touch
- Minimal scrolling
- Login gate required (not yet implemented)
- Idle timeout with attractor screen (not yet implemented)

### Remaining work

1. Login gate + login page styling
2. Idle screen / attractor
3. Cart/checkout CSS overlays
4. PDP CSS overlay
5. Start Over + idle timer JS
6. Custom admin section for kiosk settings

---

## 17. Known Gotchas

These are recurring issues worth warning yourself about. Not fix recipes — the fix
is in the code or the commit history. These are "things to watch for when you are
debugging a symptom that matches one of these patterns".

### Image slider not refreshing after gallery updates (2026-02-23)
**Symptom:** Customer uploads / changes images in a gallery, the gallery data
updates, but the image slider on the product or cart page does not reflect the
change until the user does a hard page reload.
**Cause:** Slider initialization happens once on page load and does not observe
gallery data changes.
**Workaround:** Trigger a slider re-init hook after any gallery mutation, or
force a reload as a last resort on pages where the issue is visible.

### Date input failure in some Chrome / older Safari (2026-03-16)
**Symptom:** Customer cannot select a date on a form date-picker input in certain
Chrome versions and older Safari builds.
**Cause:** Browser-level `<input type="date">` implementation bug — not a Pixfizz
bug, but it affects Pixfizz forms.
**Workaround:** Where date selection is critical, use a JS-based date picker
rather than relying on the native `<input type="date">`. Document the affected
browsers in customer-facing support docs so they know to update their browser.

### Worker JS impacting site speed / SEO (2026-03-31)
**Status:** Under investigation as of 2026-03-31. No fix documented yet.
**Symptom:** Worker JS loading is impacting Core Web Vitals and Lighthouse SEO
scores on at least one site.
**Action:** Track separately — do not assume a fix is available when scoping SEO
work on a site that depends on Worker JS. Confirm current status before
committing to a performance target.

### CSV export filter excludes anonymous projects (Rapid)
**Status:** Known bug, 2026-03-16. To be fixed.
**Symptom:** CSV export of projects, when filtered, does not include anonymous
projects on the Rapid site.
**Workaround:** Export without filters and filter in a spreadsheet, or wait for
the platform fix.

### Logged-out app error on custom-admin pages (2026-06-08)
**Status:** Platform rendering-order behaviour. Confirmed, with a working fix.
**Symptom:** Visiting a `shopper-admin` custom-admin page (e.g. `manage/dashboard`) while
logged out throws an app error.
**Cause:** The page body document renders **before** the `shopper-admin` layout's
`{% if user.is_admin %}` gate is evaluated. Admin-only snippet calls placed in the page
body (for example `{% snippet 'admin/forms/checkbox' %}`) therefore execute for
unauthenticated visitors and throw, because the layout gate has not run yet.
**Fix:** Wrap the entire page-body content in its own `{% if user.is_admin %} ... {% endif %}`
guard rather than relying on the layout gate. Alternatively, pass `fallback_content` to any
admin-only snippet call so a missing/blocked snippet degrades gracefully instead of erroring.
**Related:** Bootstrap modal CSS/JS is **not** active on the `shopper-admin` instance, so a
Bootstrap `.modal` renders as unstyled inline content. For modals inside custom admin, use a
pure-CSS `:target` toggle (or another no-JS pattern) scoped under a `pf-` prefix to avoid
clashing with the `s-` design-system classes.

## Changelog
- 2026-03-14: Added website/homepage snippet pattern and Custom Admin checkbox requirement to Section 13.
- 2026-03-19: Added how to create pages with Custom Types to Section 14.
- 2026-04-08: Updated how to create pages with Custom Types to Section 14.
- 2026-04-10: Added Section 15 — Known Gotchas (image slider refresh, date input browser bug, Worker JS SEO, CSV export anonymous projects).
- 2026-04-20: Section 13 — clarified nav link editing must target the active nav style snippet based on header-logo-position checklist value.
- 2026-05-19: Added `shopper-admin` layout to layouts table. Added Section 15 — Custom Admin manage/ page inventory including tools pages. Added Section 16 — Kiosk Touchscreen Mode architecture and status. Renumbered Known Gotchas to Section 17. Source: Claude chats (admin v2 work, kiosk mode design).
- 2026-06-01: Added Shop All page feature note. Source: fireflies-call.
- 2026-06-15: Added Static Product Importer CSV column spec under Tools pages. Added Google Ads conversion-tracking note (no built-in preset; deploy via GTM). Added Known Gotcha: logged-out app error on custom-admin pages (page body renders before the layout user.is_admin gate; Bootstrap modal CSS inactive in custom admin). Source: claude-chat.
- 2026-07-20: Added kiosk captcha per-subdomain note — captcha config does not carry from the main storefront to the kiosk subdomain and must be set on the kiosk site. Source: support-ticket.
