# 50 — Shopper Template Reference

**Authority Scope:** Structural anatomy of the Shopper parent template — layouts, navigation, snippets, theming, CSS delivery, and admin checklist system. Derived from a full CMS backup scan (2026-03-12).

_Last updated: 2026-09-09_

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

### Shopper 24 has no `layouts/main` and no px-tag layout

The parent's default layout is **`layouts/index`** (`default: true`, `renderer_type: 1`), built from `{% snippet %}` calls and opening with `{% snippet 'html.head', collection: collection, product: product, design: design, page: page %}`. There is **no `layouts/main`** on the Shopper 24 parent, and the count of `px:` macro tags across every parent layout is zero. *Verified by reading source, shopper24 backup 2026-09-18.*

| Site type | Layout | Shape |
|---|---|---|
| Shopper 24 child (Full Pixfizz storefront) | inherits the parent `index` | snippet-based, starts with `html.head` |
| Shopify integration host | its own `main`, `default: true` | the px-tag shell (`<px:setup>`, `<px:javascripts />`, `<px:stylesheets />`, `<px:content />`) |

A Shopper child that renders blank is diagnosed as "not inheriting `index`", which is a provisioning fix. **Never paste the px-tag shell into a Shopper child**: it strips the storefront and turns the symptom into a real breakage.

### The sign-in modal on every page

The `index` layout renders `#modalLoginCheckout` (Bootstrap `.modal fixed-right`) on every page. Its form posts to the current URL with `?login_user=t`, so the shopper returns to the same page after a full reload; a custom tool that opens it must save its own state first. **Never send a shopper who is mid-flow to `/site/login`**: it lands them in Saved Projects. Template-level (Shopper 24). *Observed on a live child site, 2026-10-04.* For sign-in inside the photo upload window without a reload, see `17_DESIGN_TOOL.md`.

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

### `#cart-link-icon` is used on more than one element (open defect)

`modals/cart-notification` binds the "added to cart" tooltip with `$('#cart-link-icon').tooltip(...)`, and `navigation/menubar` documents `#cart-link-icon` as the cart badge target. But the parent `navigation/style3` also wraps the **search** icon in `<span id="cart-link-icon">`, and the search `li` comes before the cart `li`, so on every Shopper 24 site with search on, the first `#cart-link-icon` in the DOM is the search icon and the cart tooltip targets it. Child overrides of `navigation/*` copy the pattern. What the shopper actually sees from the misplaced tooltip is not verified. *Verified by reading source and by query on shopper24.pixfizz.com, 2026-09-30.*

- **Parent fix (paste block, never a tar):** remove `id="cart-link-icon"` from the search icon span in `navigation/style3` and any other `navigation/*` style that wraps a non-cart icon. First list every child that overrides a `navigation/*` snippet, because those keep their own copy.
- **Audit check:** `document.querySelectorAll('[id="cart-link-icon"]').length` must be `1`, and that element must sit inside the `#modalShoppingCart` link.

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

**Checking the open state.** Shopper 24 megamenus open on hover through theme JS, which adds `.show` to the `li.dropdown`. In a hidden or background browser tab the opacity transition freezes and `getComputedStyle` keeps reporting `opacity: 0`, so test `li.classList.contains('show')`, not the opacity. *Verified by query, 2026-09-30.*

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
- **Child site: all CSS customisations go in the `style/custom.css` snippet override** — this is what gets injected into the CSS page.
- **Master parent (shopper24): CSS goes in the CMS page `custom.css`, never in the `style/custom.css` snippet**, which must stay empty on the parent because it is the stub children override.
- **The cascade runs the other way from what people expect.** The parent's `custom.css` page opens with `{% snippet 'style/custom.css' %}`, so a child's CSS is printed at the **top** of the served stylesheet and every parent rule comes after it. At equal specificity **the parent wins**. A child rule that restates a parent selector needs higher specificity (for example a leading `body`), `!important`, or a selector the parent does not use. New parent feature blocks belong at the end of the parent page. See § 18 and § 18.1. *Stated by Alex and verified by reading source, 2026-09-19.*
- Do NOT write CSS inline in Liquid/HTML snippets.

### Dark child sites: override the tokens, not just the background (2026-09-30)

Template-level (Shopper 24). *Stated as found during a dark child site build, 2026-09-30; color values read from the parent CSS. Not verified live.* The parent styles that tokens do not reach (checkout, modals, stepper, toggle, upload window, swatches, icons) are in § 18.2.

- A dark child site that overrides only `style/color-background` and `style/color-font` still ships the stock light-theme accents: cyan buttons (`#32c5ff`), black links (`#000`), red hovers (`#ff0000`), white option pills and orange pressed states (`#faa21b`). The parent's settings CSS prints after the child's `style/custom.css` with `!important`, so restating colors in CSS is not enough. Override the token snippets too: button, primary, link, nav hover, pill, badge, footer and the light background tokens (about 30 on one dark child site).
- **`.form-control { color: #111 }` is hard-coded in the parent `pages/custom.css`, not a token.** On any dark child, typed text in every form field (checkout included) is invisible until the child adds `html body .form-control { color: <light> !important }`. Candidate parent fix (not done): read it from `style/color-font`.
- Image-swatch option names carry an inline `style="color:#111111"` in the variant template, so a dark site needs `!important` to recolor them.
- Shared pxt tools (Live Finish and the custom design tools) take their ink from `--pxt-ink`; see `27_LIVE_FINISH_AND_3D_PREVIEWS.md` § 2.7.

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

### Value-bearing checklists — never assume boolean

Many checklist snippets hold a value the parent **interpolates directly**.
Overwriting one with `TRUE` does not disable a feature; it produces a
render-time failure. `cart-icon` set to `TRUE` resolves
`{% snippet 'icons/' + value + '.svg' %}` to `icons/TRUE.svg`, and because the
header renders on every page the whole site goes down with
`Liquid error: Snippet not found`.

Known value-bearing keys: `cart-icon`, `user-icon`, `font-body`
(`avenir` / `lato` / `open-sans` / `custom`), `header-logo-position`
(`LEFT` / `CENTER`), `gallery-thumb-position`, `align-collection-card`,
`align-collection-title`, `variant_columns` (`col-md-6`),
`variant_columns_mobile`, `country-filter` (`United States`),
`description-position`, `pricing-tab-position`, `upload-btn-style`.

**Rules when editing checklists on an existing site:**

1. **Read the seed value first.** Never assume a key is a boolean.
2. **Preserve the site's own vocabulary** — value type *and* token case. A site
   using `FALSE` (not blank) for off, and `LEFT` (not `left`), must keep both.
   Comparisons against `'TRUE'` are case-sensitive.
3. **Override only where the build genuinely requires it.** On one rebuild, 14
   of 26 changed checklists were reverted as unnecessary; every one had been an
   unforced risk.
4. **Capture cleanly.** A `{% capture %}` of a checklist snippet includes
   surrounding newlines and indentation. Always `| strip` (and `| upcase` where
   case is uncertain) before comparing, or every comparison silently falls
   through to the default.

**Token sets for radio-type keys.** The storefront compares exact tokens, and the `setup/*` pages write them. The `manage/*` pages wrote the human label instead for every key below, so a value such as `Version 2` or `mm/dd/yyyy (US)` found on a site is a manage/* write that the storefront does not recognize (see § 15). *Verified by reading source, shopper24 backup 2026-09-28; label values found on live sites by query, 2026-09-30.*

| Key | Tokens the code compares (setup/* writes) | Label manage/* wrote |
|---|---|---|
| `align-collection-card` | `LEFT`, `CENTER` | Left Align, Center Align |
| `align-collection-title` | `LEFT`, `CENTER` (code tests `CENTER`, or `TRUE`) | Left Align, Center Align |
| `checkout-column-positions` | `LEFT`, `RIGHT` | Order Summary on left/right |
| `collection-card-shadow` | `TRUE`, `FALSE` | Shadow, No Shadow |
| `date-format` | `MM`, `DD` | mm/dd/yyyy (US), dd/mm/yyyy |
| `description-position` | `ABOVE`, `BELOW` | Above Fold, Below Fold |
| `font-body` | `avenir`, `lato`, `open-sans`, `custom` | Avenir, Lato, Open Sans, Custom |
| `gallery-thumb-position` | `BOTTOM`, `LEFT` | Bottom, Left |
| `gallery_version` | `v1`, `v2` | Version 1, Version 2 |
| `account_saved_projects_version` | `v1`, `v2` | Version 1, Version 2 |
| `payment-gateway` | `stripe`, `square`, `authorizedotnet`, `braintree`, `bridgepay`, `paypal`, `other` | Stripe, Square, Authorize.net, ... |
| `pricing-display` | `PRODUCT`, `CART`, `BOTH` | Below Product Name Only, Add to Cart Button Only, Both Locations |
| `pricing-tab-position` | `Header`, `Footer` | Header Tabs, Footer Accordion Tabs |
| `prints-autoselect` | `true`, `false` (code tests `false`) | Select All, Do Not Select |
| `bullet-point-style` | `disc`, `circle`, `square` (a CSS value) | Disc, Circle, Square |
| `variant_columns` | `col-md-6`, `col-md-4`, `col-md-3`, `col-md-2` | 2 per row, 3 per row, ... |
| `variant_columns_mobile` | `col-6`, `col-4`, `col-3`, `col-2` | 2 per row, 3 per row, ... |

**Rule:** the value written to a checklist snippet must be the exact token the storefront code compares, matching case; a label is display only. Every admin control must write the snippet the storefront reads, so grep the template for the key before shipping a control.

**`custom-X-page` flags against an empty target snippet.** A key such as
`admin/checklist/custom-faq-page` set to `TRUE` with an empty `website/faq_page`
renders a blank page with no error. Worth checking on any site you touch — it is
frequently pre-existing rather than introduced.

### Full checklist key inventory

#### Navigation & Header
| Key | Values / Notes |
|---|---|
| `header-logo-position` | `LEFT` = style1; `CENTER` (default) = style3; `CUSTOM` = conditional |
| `top-promotion-bar` | `TRUE` = show promotion bar above nav; text in `header/promotion` |
| `bottom-promotion-bar` | Intended to show a promotion bar below the nav, but **no layout includes `header/bottom-promotion-bar`**, so the setting does nothing (Corrected 2026-10-06; verified by reading source, shopper24 backup 2026-09-28) |
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
| `default-delivery-option` | Preselects a delivery method on `/site/checkout`. Accepted values are **`public`** (in-store pickup), **`private`** (deliver to my address) and `none` (nothing preselected, the parent default) — see §17 |
| `display-shipping-options` | `TRUE` = show shipping options |
| `disable-user-registration` | `TRUE` = prevent new registrations |
| `checkout-disclaimer` | `TRUE` = show checkout disclaimer text |
| `checkout-rush` | `TRUE` = enable rush delivery option |
| `checkout-rush-special` | `TRUE` = enable special rush option |
| `checkout-column-positions` | Checkout column order: `LEFT` or `RIGHT` |
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
| `max-cart-total-pay-in-store` | Maximum cart total for pay in store. Number only, inclusive, blank = no limit. See `21_SHOPPER_CHECKOUT_POLICY.md` |
| `confirm-start-date` | `TRUE` = require start date confirmation |
| `confirm_start_date_label` | Label for start date field |
| `confirm-with-invoice` | `TRUE` = confirm order with invoice |
| `payment-link` | `TRUE` = enable payment link option |
| `payment-gateway` | Active gateway: `stripe`, `square`, `authorizedotnet`, `braintree`, `bridgepay`, `paypal`, `other`; default `stripe` (Corrected 2026-10-06: the token is `authorizedotnet`, not `authorize.net`) |
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
| `kiosk-tip-enabled` | `TRUE` = show the associate tip panel at kiosk checkout (`kiosk/associate-tip`, see `21_SHOPPER_CHECKOUT_POLICY.md`) |
| `kiosk-tip-fixed-below` | Cart subtotal below which the tip tiles are fixed amounts rather than percentages |

**Kiosk captcha is per-subdomain.** Kiosk mode usually runs on its own subdomain (`kiosk-mode-domain`). CAPTCHA configuration does not carry across from the main storefront to the kiosk subdomain — captcha must be removed/configured on the kiosk subdomain specifically (e.g. `kiosk-remove-captcha` set on the kiosk site). Symptom if missed: customers hit a CAPTCHA on the kiosk that the main storefront does not show.

#### Product Display
| Key | Values / Notes |
|---|---|
| `pricing-display` | Where the price shows: `PRODUCT`, `CART` or `BOTH` |
| `pricing-tab-position` | Position of pricing tab: `Header` or `Footer` |
| `description-position` | Position of product description: `ABOVE` or `BELOW` |
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
| `prints-autoselect` | `true` / `false`, lowercase; the code tests `false` (Corrected 2026-10-06: previously listed as `TRUE` = auto-select first print size) |
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
| `account_saved_projects_version` | Saved projects version: `v1` or `v2` |
| `account_saved_projects_view` | Default view for saved projects |
| `hide-saved-projects` | `TRUE` = hide saved projects from account |
| `hide-galleries` | `TRUE` = hide galleries from account |
| `hide-personal-dates` | `TRUE` = hide personal dates |
| `gallery_version` | Gallery version: `v1` or `v2` |
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
| `activate-klaviyo` | `TRUE` = load Klaviyo (see § 20.1) |
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
| `date-format` | Date display format: `MM` (month first) or `DD` |
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
- `website/google-review-link`: **not read by the reviews widget.** The widget reads `admin/checklist/google-review-link`; the manage/store "Google Review Link" field wrote this `website/` key (Corrected 2026-10-06; verified by reading source, 2026-09-30)
- `website/gtag` — Google Analytics 4 tag ID
- `website/meta-pixel` — Facebook Pixel ID
- **Google Ads conversion tracking:** there is no dedicated built-in preset or snippet for a Google Ads conversion tag in Shopper (only GTM, GA4, and Meta Pixel exist). Deploy the Google Ads site tag (`AW-...`) and the purchase conversion event through GTM using the existing `setup-google-tag-manager` key (conversion linker + conversion action tag + thank-you-page event). A hardcoded gtag conversion snippet, if used instead, is site-specific code with no checklist key reserved for it.
- `website/px-subdomain` — Pixfizz subdomain
- `website/film-delivery-address` — for film mail-in orders
- `website/trust-badges` — trust badge images
- `website/current-promotions`: promotional content. On the parent it holds hard-coded 2024 sale HTML: do not reuse it
- Store address: the address keys are `website/contact/address`, `city`, `state`, `zip-code`, and the info bar location reads `admin/checklist/menubar-location`. `website/contact/store-location` (written by manage/store "Store Location") is read by nothing
- `website/sitewide-promotion` — sitewide promo text

---

## 8. Footer Structure

The footer (`snippets/footer`) has two sections:

**Top section** (`py-6 py-md-12 border-bottom border-gray-700`):
- Newsletter signup (hidden by default). The footer newsletter form is a hidden placeholder and does not subscribe anyone to Klaviyo (see § 20.1)
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

**The favicon is the site asset named exactly `favicon.png`.** `html.head` emits one icon link, pointing at that asset served as a 96px WebP (`.../thumbnail/96/-/format/webp/~/favicon.png`). An icon uploaded under any other name (for example `brand-favicon-512.png` or `apple-touch-icon.png`) is referenced by nothing, and the site keeps showing the default icon. Every site build ships a 512x512 transparent PNG of the brand mark named `favicon.png` (Website > Assets), and the install steps say so. After upload, check that the icon link in the live head points at the new file. Not verified: whether a child-level `favicon.png` takes over from one inherited from the parent without anything being removed first. Template-level (Shopper 24). *Verified by query (live DOM on a child site), 2026-09-25.*

**The social sharing image is the asset named `og-preview-image.jpg`**, which `setup/seo` uploads. The manage/seo "Social sharing image" control uploaded an asset named `social-media/open-graph`, which the head never reads. Template-level. *Verified by reading source, shopper24 backup 2026-09-28.*

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

**Current state (2026-10-06).** The 14 notification templates are per site and not inherited, and from 2026-10-03 the supported setup is the **email kit** (`email-kit/*` on shopper24), which keeps each site's template bodies to one line calling parent snippets. The kit, its cart reminder settings and how templates behave in admin are in `32_ORDER_LIFECYCLE.md`; Liquid rules for email context are in `50_LIQUID_REFERENCE.md`. Emails use the asset `logo.png`; the `email-logo.png` uploaded by manage/branding and manage/emails is read by no email template. Inline SVG (`icons/*.svg`) does not render in Gmail or Outlook, so use PNG. *Verified by reading source and by query, 2026-09-30 and 2026-10-03.*

---

## 12. Section Library

**Static sections (`sections/static/`):**
`2-column-cards`, `2-column-collection-spotlight` (+ `-2`), `2-photo-feature` (+ `-2`), `3-column-cards` (+ `-1`, `-2`, `-2nd-row`), `3-features`, `3-steps`, `4-block-best-sellers`, `4-company-values`, `6-across`, `6-categories`, `7-across`, `8-feature-spotlights`, `8-photo-feature`, `about`, `banner-alt`, `carousel`, `carousel-features`, `carousel-products`, `checklist-feature`, `coming-soon`, `countdown-promo-fullwidth`, `custom`, `features`, `hero-3-columns-fullwidth` (+ `-2`), `hero-4-features`, `hero-carousel`, `home-page__3-columns-fullwidth`, `home-page__digitize-services-cards`, `home-page__hero-banner-fullwidth`, `home-page__trusted-brands`, `image-slider-comparison`, `newsletter`, `our-blog`, `parallax-banner`, `parallax-text-block`, `project-gallery`, `projects`, `promo-banner-color`, `promo-countdown`, `reviews`, `reviews-1-across`, `services-contact-footer`, `shop-by-brand`, `top-item-feature`, `top-picks`

**Dynamic sections (`sections/dynamic/`):**
`blog`, `carousel-products`, `collection-block`, `free_shipping_progress_bar`, `product-carousel`, `product-description-tabs`, `services`

Dynamic sections re-inject into the DOM on AJAX updates. Use the `style onload` pattern for any JS that must survive re-injection (see `01_CODE_GOVERNANCE.md`).

### Blog (`blog_post` Custom Type)

Template-level (Shopper 24). *Verified by reading source (shopper24 backup 2026-09-24) and live on a child site, 2026-09-30.*

- `blog_image` and `blog_thumbnail` on `blog_post` are **asset-type** fields rendered with `| asset_url`. Write the **asset name**, never a URL. An asset uploaded by API can be referenced by the name the upload returns (`61_PIXFIZZ_API.md`).
- **The post page renders unpublished posts.** `/site/blog/<blog_path>` returns 200 for a post whose `blog_unpublished` is true; only the listing hides it.
- **`sections/dynamic/blog` (the homepage blog section) applies no filter.** It lists every `blog_post` instance sorted by `custom.blog_title`, with no `blog_unpublished` or date filter, so unpublished and future-dated posts appear wherever the section is used.
- The post page BlogPosting JSON-LD prints `blog_title` and `blog_description` without `escape_json`, so a double quote in either breaks the JSON-LD.
- The `blog_post` type and its fields are per site and vary between sites (some lack `blog_title` or `blog_unpublished`). Check the site's fields before relying on any of the above.

### Promotions fly-out and bars

Template-level (Shopper 24). *Verified by reading source, shopper24 backup 2026-09-24.*

- **Fly-out.** `modals/promotions` reads the Custom Type `promotions` with fields `promo_name`, `promo_message`, `promo_code`, `promo_cta`, `promo_link`, `promo_img`, `promo_start_date` and `promo_end_date`. It shows only entries whose dates include today, so entries expire by themselves. The panel is always in the layout; a site still needs something that opens it.
- **Top bar.** `admin/checklist/top-promotion-bar`, with the text in `header/promotion`. The bottom (sub-nav) bar does nothing, see § 5.
- **Bundles engine.** `bundles/config`, `bundles/landing` and `bundles/landing-style` are on shopper24 (as of 2026-09-24).

### Shop All page

A Shopper page that displays an image for every top-level collection, giving
shoppers a single visual entry point to all categories. Addresses the visual
navigation gap when a store has many top-level collections. Live as of mid 2026.
The exact page or snippet name should be confirmed against the live deployment or
with Matjaz.

---

## 13. Practical Notes for Development

- **Editing nav links:** Always check which nav style is active before editing. `admin/checklist/header-logo-position` value `LEFT` renders `navigation/style1`; `CENTER` (default) renders `navigation/style3`. Edit the `{% capture navigation_links %}` block at the top of the **active** snippet only — editing the wrong one has no effect. Do not edit the HTML structure below the capture block.- **Adding a megamenu panel:** Create or edit `navigation/megamenu/<name>`. Use the patterns in section 3 above. Register the nav item in the `navigation_links` capture block.
- **CSS changes:** child site → `style/custom.css` snippet override; shopper24 parent → the `custom.css` page. Never inline in Liquid/HTML.
- **Theme color changes:** Edit the relevant `style/<token>` snippet. The CSS page picks them up automatically.
- **Checklist changes:** Edit the `admin/checklist/<key>` snippet value. Boolean flags expect exactly `TRUE` or blank.
- **New sections:** Use an existing section snippet from `sections/static/` or create a new one. Include in `pages/__home` or the relevant page.
- **Email project previews:** Always include `share: orderline.project.share_code` in the preview URL.
- **Font changes:** Set `admin/checklist/font-body` to `lato`, `open-sans`, `avenir`, or `custom`. For `custom`, populate `style/custom-body-font` with the font-family string **without a trailing semicolon** (`"Nunito Sans", sans-serif`). The parent `custom.css` page prints `{% snippet 'style/custom-body-font' %} !important;`, so a trailing semicolon ends the declaration early and the `!important` is dropped as an invalid declaration. The parent snippet's own Description says to include the semicolon; it is wrong (see `52_SNIPPET_INVENTORY.md`, Known Parent Defects). *Verified by reading source (shopper24 `pages/custom.css` and `style/px-tool-theme`), 2026-09-29.*
- **Shared snippets:** If a snippet is used across multiple client sites, follow the Shared Snippet Contract Rule in `01_CODE_GOVERNANCE.md` — do not remove existing variables, IDs, or JS hooks.
- **Custom home page content:** Place home page content in the snippet `website/homepage`. It is gated on the value snippet `admin/checklist/custom-home-page`, which `pages/__home` reads as:

  ```liquid
  {%- capture home-page-custom %}{% snippet 'admin/checklist/custom-home-page' %}{% endcapture -%}
  {% if home-page-custom == 'TRUE' %}
  ```

  **There is no `| strip` on that capture**, so the snippet body must be byte-exact: `'TRUE\n'` is not `'TRUE'` and the else branch silently serves the seed demo homepage. Every single-line value snippet in the shopper24 seed ends without a newline. See §17, *A trailing newline in a value snippet silently breaks every flag*.

  **PENDING CONFIRMATION — two records conflict.** A 2026-08-24 diagnosis recorded a Custom Admin → Storefront Settings checkbox as also required and not settable from a tar; a 2026-08-27 reading of the parent source found the checklist snippet to be the only gate, with the earlier symptom fully explained by the trailing newline. Until this is settled on a live site, ship the snippet byte-exact **and** check the Storefront Settings toggle after import.

  **Diagnosis.** Load the homepage and look for the wrapper class the custom homepage emits. Wrapper absent while `style/custom.css` tokens resolve and header and footer are branded = the tar imported and the gate is off. Wrapper absent and theme tokens unresolved = the tar did not import. Wrapper present with stale content = caching or a different snippet.

  **Delivery rules for a custom homepage with motion or widgets** (template-level, held on a child homepage build, 2026-10-04):
  - CSS goes in the child `style/custom.css` under **one wrapper class**, with two-class selectors (child CSS prints before the parent's, see § 18.1).
  - JS goes inline at the end of `website/homepage`, with **no Liquid inside the script**.
  - A `prefers-reduced-motion` block turns all motion off.
  - Any price shown on the homepage must first be readable on a live product page; a hard-coded copy must be re-checked after every price edit.
  - Check at 1440 px and 390 px wide for overflow and broken images before handing over.

## 14. Creating Pages on Child Sites

Child sites of Shopper cannot create real CMS pages. New pages are
created by adding instances of the `pages` Custom Type instead.

**Navigation path:** Main Admin → Website tab → Custom Types → Pages

Create a new instance with these field values:
- `page_path` — must match the URL path exactly. At level 1 this is the single
  segment (`graduation` → `/site/graduation`). At levels 2 and 3 it is the
  **full slash-joined path**, not just the final segment
  (`services/framing` → `/site/services/framing`)
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
- **A collection landing route can already hold the path.** `/site/photo-prints` answered 200 with the prints collection landing page (rendered by `product/product-details-prints`) on a child site that had no Pages instance on that path. So `/site/<path>` can be taken by a collection or built-in page, not only `/site/shop/<path>`. Before choosing a `page_path`, load `/site/<path>` and confirm it returns 404. **Settled for `photo-prints` (2026-10-08):** shopper24 has a real page `pages/photo-prints`, and a real page beats the Pages catch-all, so a Pages instance at `page_path: photo-prints` never renders (verified by reading source, shopper24 backup of 2026-10-07). For other paths, do not find out on a live site. Platform-level routing. *Verified live, 2026-09-25.*
- **Replacing `/site/photo-prints` on a child.** shopper24 `pages/photo-prints` captures `product/photo-prints-landing` (an empty parent stub, `fallback_content ''`), strips it, and renders it when it is not blank; otherwise it renders the standard `product/product-details-prints`. A child replaces the page by overriding `product/photo-prints-landing` only. Do not fork `product/product-details-prints` on a child: it also renders shop collections that set `collection.custom.print_ux`. Strip a capture before testing `!= blank`, because whitespace-only output is not blank. A paste meant for a child must never go on the parent stub: on 2026-10-08 a child landing pasted into the parent went live on every child for three minutes, so after any parent stub edit, check one other child. A child override edit can take about a minute to reach the cached page; a cache-busted fetch (`?nc=`) shows it first. Template-level (Shopper 24). *Verified by reading source and live, 2026-10-08.*
- **Nothing stops two instances having the same `page_path`.** Admin accepts a duplicate with no warning, and the storefront renders the **older** instance, so the new content never appears and it looks like caching. To replace a page, edit the existing instance; before pasting, list the instances for that path, and after pasting confirm a marker unique to the new content in the live DOM. *Verified live, 2026-09-22.*
- Levels 1, 2 and 3 each have their own catch-all page. Each one builds
  `page_path` by joining its own path params with `/`, so a level 3
  instance stores all three segments in that single field
- **Never use a Pages instance under `services/`.** shopper24 has real pages
  `/services` (In-Store Services), `/services/:service-path` and
  `/services/:service-path-1/:service-path-2`. `/services/:service-path` looks up
  `website.custom_types.service | where: 'custom.service_path', request.path_params['service-path'] | first`
  and runs `return_404` when nothing matches. Real pages beat the Pages catch-alls, so
  a Pages instance with `page_path` `services/<x>` always 404s, which looks like
  "slashes do not route" (they do; other two-segment paths work). A service page needs
  the **Service Custom Type** on the site (import it from shopper24, see
  `13_TEMPLATE_BOUNDARIES.md`) and one Service record per page; `service_no_layout`
  renders `service_content` raw, with breadcrumbs still above it. Template-level.
  *Verified by reading source and live, 2026-10-05.*

### Liquid inside `page_content`

- **Liquid output renders in `page_content`.** On a Shopper 24 child, `{{ 'x' | asset_url }}`
  and `{{ 1234.5 | currency }}` resolve inside a Pages instance's `page_content`. Meta title,
  description and noindex come from the instance fields, not the layout. A whole unlisted
  page can therefore ship as one instance plus one JS asset and one CSS asset, with its config
  in a `<script type="application/json">` block that Liquid fills (money format, asset URLs):
  no snippet, no tar. *Verified by query on a child site, 2026-10-04.* This answers the
  "does `page_content` render Liquid" question left open in `01_CODE_GOVERNANCE_UPDATED.md`.
- **Variables do not reach a snippet called from `page_content`.** On another child,
  `{% assign %}` and `{% capture %}` values set in `page_content` were empty inside a called
  snippet, and so were values passed as keyword arguments; an HTML comment written just before
  the call did not appear either. The `{% snippet %}` call itself ran, and the snippet's own
  assigns worked. *Verified live, 2026-09-17.* The two observations are not yet reconciled.
  Until they are, keep `page_content` to a single snippet call when a snippet needs per-page
  settings, and give the site its settings through an overridable parent stub instead:
  1. On the parent, a generic stub such as `bundles/config` containing `{}`.
  2. On the child, **Override Snippet** with the site's JSON.
  3. In the snippet: `{% capture j %}{% snippet 'bundles/config', fallback_content: '{}' %}{% endcapture %}` then `{% assign all = j | strip | parse_json %}`.
  4. Pick the entry for the current page with `request.path | split: '/' | last`.

  A captured snippet's output does not depend on variable scope. This pattern is verified in a
  local render only.

### Head-level dependencies must repeat the lookup

`html.head` renders **before** `{{ page.content }}`. A `custom_page` variable
assigned inside the catch-all page body therefore does not exist yet when the
head snippet runs. Anything in the head that depends on the Custom Type
instance (meta title, meta description, canonical, robots) has to repeat the
path build and the lookup inside `html.head` itself.

This is the usual reason meta title and description come out blank on Custom
Type pages while the visible page content renders correctly.

The head-side lookup rebuilds the path from the request params, appending
levels 2 and 3 only when they are present, then queries the same collection:

```liquid
{% assign cp_path = request.path_params['page-path-1'] %}
{% if request.path_params['page-path-2'] != blank %}
	{% assign cp_path = cp_path | append: '/' | append: request.path_params['page-path-2'] %}
{% endif %}
{% if request.path_params['page-path-3'] != blank %}
	{% assign cp_path = cp_path | append: '/' | append: request.path_params['page-path-3'] %}
{% endif %}
{% if cp_path != blank %}
	{% assign cp_page = website.custom_types.pages | where: 'custom.page_path', cp_path | first %}
{% endif %}
```

The `!= blank` guard matters. Without it the lookup runs on every page on the
site, not only on the catch-all pages.

**Suppressing indexing per page.** Add a boolean custom field to the `pages`
Custom Type and emit the robots tag from the head block above. `hide_from_index`
matches the naming already used for products and designs. Two platform rules
apply:

- Boolean custom fields are real booleans. Test with
  `{% if cp_page.custom.hide_from_index %}`, never against the strings
  `'true'` or `'false'`.
- Custom fields do not inherit from parent to child, so the field has to be
  created on every site that needs it.

The same head block is what fixes meta title and description on Custom Type
pages, so it is worth doing all of it in one pass rather than only the robots
tag.

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

- A header row is skipped **only when its first cell is exactly `name`**. Columns beyond those listed are ignored.
- `name` and the image filename are capped at **64 characters** on create; codes are unique **case-insensitively**; `description` is sent but **not stored** (an API defect, `61_PIXFIZZ_API.md` § 13g).
- **The importer can hang after creating the products, before assigning them to the collection.** The products exist but sit in no collection. Assign them from the admin host instead (`admin.pixfizz.com/site/<site>/admin/theme_categories/<id>/add_products?products[]=…`, about 10 ids at a time). **If a customer says the importer hung, check whether the products already exist before re-running the file**: a re-run duplicates every product. One target collection per upload run. *Verified by query, a 1,092-product import, 2026-09-23.*
- This tool drives the same product-creation path as the Pixfizz API; it does not create
  personalization templates or designs, only static products.

### Sidebar navigation

The sidebar is defined in a shared snippet. When adding new pages, update the sidebar snippet with the new nav item. The sidebar uses the `shopper-admin` design system CSS classes (`s-card`, `s-field`, `s-field-label`, etc.).

### File inputs and sample downloads on shopper-admin pages

- **The layout styles every file input itself: never override it.** `layouts/shopper-admin` runs `initFileInputs()` on `DOMContentLoaded` and on `px.fragmentsReloaded`. It walks `.s-main input[type="file"]` and injects, as siblings, a `.s-file-btn` button ("Choose file") and a `.s-file-label` span ("No file chosen"); `setup/css/shopper-admin.css` hides the native input. A page that forces the native input visible (an inline `position: static !important; opacity: 1 !important` block written because the input "looked missing") ends up with two Choose file controls side by side. Write the input plain, for example `<input type="file" id="js-csv-file" accept=".csv" required />`. `fileInput.files[0]` in page JS is unaffected, the injected label updates itself, and the injected handler re-enables `button[type="submit"]` or `button.btn-primary` in the enclosing form (safe outside a form). *Verified live on shopper24, 2026-09-18.*
- **Build sample-file downloads in the page, not as a site asset.** For a "download an example CSV" control on a parent Shopper page, build the file client-side from a `Blob` and a synthetic anchor rather than linking an uploaded asset with `asset_url`. The sample then travels with the page to every child, stays in step with the column order documented on the same page, and needs no per-site upload. Keep sample rows free of apostrophes (Pixfizz Liquid has no backslash escape) and use single-quoted JS strings so double quotes in quoted CSV fields need no escaping.

### Known defects in the manage/* pages (audit of 2026-09-30)

Template-level (Shopper 24). Read from the shopper24 backup of 2026-09-28; the manage/* pages were unchanged on the live parent (compared by content hash, 2026-09-30). Findings on live sites were checked by query (93 overrides across 17 sites). **Do not set any of the settings below through `/site/manage/*`**; use the matching `setup/*` page or set the snippet directly.

1. **Radio controls write the label, not the token**, for at least 17 radio-type checklist keys. The storefront then does nothing or falls back. The token table is in § 5. Effects seen live: `date-format` = `mm/dd/yyyy (US)` fell to the `DD` branch (day-first dates on a US store); `gallery_version` = `Version 2` rendered the v1 gallery inside account v2; `account_saved_projects_version` = `Version 2` was dormant because account v2 redirects the old saved projects page.
2. **Radio groups that write a snippet literally named `put`.** manage/account (Navigation Version), manage/gallery (Image Tiling, Gallery Download, Image Download, Order Prints Button), manage/homepage (Custom Homepage), manage/navigation (cart and user icon Style), manage/payments (Pay In Store) and manage/products (Upload Button Style) call `admin/forms/radio-button` with `snippet_name: 'put'`. Every one writes the same snippet, `put`, and none changes its setting. Nothing reads `put`; a `put` snippet found on a site can be deleted.
3. **Fields that write the wrong key:**
   - manage/store "Google Review Link" writes `website/google-review-link`; the reviews widget reads `admin/checklist/google-review-link`.
   - manage/seo "Hide from search engines" writes `admin/checklist/launch-no-index`; the site reads `admin/checklist/no-index` (see `81_SEO_AND_GEO_REFERENCE.md`).
   - manage/seo "Social sharing image" uploads an asset named `social-media/open-graph`; the head reads `og-preview-image.jpg` (§ 9).
   - manage/branding and manage/emails "Email Logo" upload `email-logo.png`; no email template reads it (emails use `logo.png`).
   - manage/integrations "Google Ads tag" writes `integrations/google/ads-id`, which nothing reads (§ 20).
   - manage/store "Store Location" writes `website/contact/store-location`, which nothing reads (§ 7).

**Lesson for audits.** Audit settings against the **live** parent, not only a backup: the live checkout changed between two backups two days apart. Compare by content hash through the CMS API with `?sitename=`. Before fixing a bad value on a live site, trace what it gates today: a "wrong" value can be dormant, or it can be what the site currently renders (day-first dates, the v1 gallery), in which case the fix is a visible change for the client.

---

## 16. Kiosk Touchscreen Mode

**Status:** Partially implemented. Login gate and cart/checkout overlays not yet built. **Amended 2026-08-29:** `kiosk/idle-screen` now exists on the parent (see the defect note in §17) — the "idle screen not yet implemented" line below is stale for the snippet itself, though the idle timer JS that triggers it is still outstanding. Verify against the parent before quoting this status.

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

### Kiosk mode is not touchscreen mode — the minimum key set (2026-09-09)

`kiosk-touchscreen-mode` is the **UI** switch documented above. `kiosk-mode-enabled` is the
**mode** switch, and a lab can run one without the other. The minimum a site needs for kiosk
mode itself:

| Checklist key | Value |
|---|---|
| `kiosk-mode-enabled` | `TRUE` |
| `kiosk-mode-domain` | the exact host the kiosk points at |
| `kiosk-remove-captcha` | `TRUE` — **must be set on the kiosk subdomain specifically, it does not carry over** (see §5) |
| `kiosk-pay-in-store-only` | `TRUE` only where pay-in-store should be kiosk-only |

`helpers/is-kiosk-mode` compares `request.host` against the **single** value in
`kiosk-mode-domain`. There is one kiosk domain; individual terminals are distinguished by
`?terminal=N` on the URL. For per-order terminal attribution add `kiosk-terminal-enabled` /
`kiosk-terminal-ids` and the `kiosk/terminal-capture` snippet.

**`is-kiosk-mode` fails silently on any host mismatch.** It renders nothing and every feature
gated on it goes dark, which presents as "the feature never appears". Check `kiosk-mode-domain`
against the actual host **before anything else** when a kiosk-gated feature is missing.

**Design tokens are defined on `.kiosk-touchscreen`, not on kiosk mode.** `--k-accent`,
`--k-radius-lg` and the rest of the kiosk token block are **declared inside `kiosk/style`
scoped to `.kiosk-touchscreen`**. A feature gated on **kiosk mode** rather than touchscreen
mode therefore resolves none of them, and ships an unstyled panel with dead custom properties
onto a live checkout. Any such feature must carry its own self-sufficient token block and, if
it wants the kiosk look where touchscreen mode is on, remap onto the kiosk tokens under
`.kiosk-touchscreen .<its-own-class>`.

*Verified by reading source — shopper24 CMS backup 2026-09-09.*

### Kiosk checkout styling (2026-10-02)

The parent `kiosk/style` now carries two blocks for touchscreen checkout, "Kiosk checkout store location cards" and "Kiosk opening hours button and modal", plus the associate tip panel (`21_SHOPPER_CHECKOUT_POLICY.md`). Two rules came out of building them:

- **Nested cards inherit the outer card reset.** On kiosk checkout the store address cards (`.card.card-outline-address`) sit inside an outer `.card.px-rounded`, and the existing rule `.kiosk-touchscreen .checkout-page .card.px-rounded .card { border: 0 !important; ... }` zeroes their borders, so a plain `.kiosk-touchscreen .card.card-outline-address` override loses on specificity. Prefix inner-card rules with `.kiosk-touchscreen .checkout-page .card.px-rounded`, and read the matched rules on the element before writing an override. Checked custom radios in kiosk now use `--k-accent`, which also restyles the Delivery radios.
- **A modal rendered inside a component inherits its text alignment and fonts.** The store hours modal (`#storeHours-<id>`, a Bootstrap modal) is rendered inside the card's `.text-right` column, so the whole modal was right-aligned and the title ran under the absolutely positioned close button. Always open modals and dropdowns when restyling a component, not only the closed state.

Template-level (Shopper 24). *Verified live by computed styles and screenshot on a client kiosk, 2026-10-02.*

---

## 17. Known Gotchas

These are recurring issues worth warning yourself about. Not fix recipes — the fix
is in the code or the commit history. These are "things to watch for when you are
debugging a symptom that matches one of these patterns".

### The live navbar is taller than a render harness shows

On live Shopper 24 pages `.navbar` carries 16 px vertical padding at phone width from a stylesheet that a local harness (theme.min.css plus px-shopper.css) does not load. A custom header that set `padding-top: 0; padding-bottom: 0` measured 63 px in the harness and 95 px live. Zero a custom navbar's vertical padding with `!important`, and measure the header on the live URL at 390 px before reporting it done. Template-level. *Verified by live DOM measurement, 2026-10-07.*

### An admin.pixfizz.com login does not open `user.is_admin` pages

The storefront needs its own admin sign-in before `manage/*`, `setup/*` or any other `user.is_admin` page shows anything (`61_PIXFIZZ_API.md` § Admin UI host vs API host). A site with no storefront login page needs one: `{% form 'user_login' %}` inside a narrow `{% dynamic %}` block, as in shopper24 `account/login-form`. *Verified by query, 2026-10-07.*

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

### Add to Cart button carries no `type` attribute (2026-08-10)
**Status:** Confirmed on live product pages.
**Symptom:** Custom JavaScript that resolves the cart button as
`form.querySelector('button[type="submit"], input[type="submit"]')` gets `null`, so any
programmatic enable or `.click()` does nothing. A `.click()` on a still-disabled button is
silently swallowed — no error, no navigation, and the user simply stays on the page.
**Cause:** The real control is
`<button class="btn btn-block btn-primary add-to-cart-button">ADD TO CART · $20.00</button>`.
A `<button>` inside a form submits by default, but an attribute selector matches only the
*literal* attribute, and `type` is absent.
**Fix:** Resolve by class and visible text — `.add-to-cart-button`, or
`/add[\s_-]*to[\s_-]*(cart|basket|bag)/i` — never by `button[type="submit"]` alone.

### The photo upload window is a native modal `<dialog>`: no z-index paints over it (2026-09-28)
**Status:** Confirmed live. Platform-level: the upload window is the same component on every site.
**Symptom:** A site-wide overlay element (a custom cursor, a toast, a banner) set to the maximum z-index disappears while the photo upload window is open. On a site that sets `cursor: none` for a custom cursor, the shopper sees no cursor at all.
**Cause:** The upload window is `<px-upload-dialog>` wrapping `<dialog class="px-upload-dialog">`, opened with `showModal()`. A modal dialog renders in the browser's **top layer**, which paints above every z-index on the page. Bootstrap `.modal` (z-index 1050) is not a top-layer element and is not affected.
**Fix:** Move the element inside the open modal dialog while one is open, and back to `<body>` when none is. Run the check on the event that drives the element (for a cursor, `mousemove`) and keep the node reference in JS, so it is re-attached to `<body>` if the dialog leaves the DOM:

```js
var top = null;
try { var open = document.querySelectorAll('dialog[open]');
  for (var i = 0; i < open.length; i++) { if (open[i].matches(':modal')) { top = open[i]; } } } catch (e) {}
var host = top || document.body;
if (el.parentNode !== host) { host.appendChild(el); }
```

`position: fixed` children of the dialog still position against the viewport only while the dialog has no `transform`, `filter` or `contain`. `.px-upload-dialog` has none (verified by computed style); check before reusing this on another dialog.
**Diagnosis in one call:** `document.querySelector('dialog:modal')`. Anything returned means the page is in top-layer territory and z-index is irrelevant.
**Related:** with `html { scroll-behavior: smooth }` in the site CSS, setting `scrollBehavior = 'auto'` and calling `scrollTo` in the same tick still scrolls smoothly, because style has not been recomputed yet. Read `getComputedStyle(document.documentElement).scrollBehavior` before `scrollTo` to force it.

### Site search results can link into test or hidden collections (2026-09-30)
**Status:** Open. Platform-level indexing behavior, mechanism unknown.
**Symptom:** On a child site, a search for a main product word returned nine results, all with URLs inside a `...-test` or `...-old` collection, while the same products also sat in the real, visible collection.
**Rule until the mechanism is known:** before turning on `admin/checklist/search` for a site, run three searches for its main product words and check that no result URL contains a test, old or hidden collection path. Clean those collections out first. How the index chooses a collection path for a product that sits in several collections is not verified.
*Verified by query on a live child site, 2026-09-30.*

## The Add to Cart control

Measured on a live Shopper product page, 10 Aug 2026.

```html
<form class="project_create">
	…
	<px-option code="…">…</px-option>
	…
	<button class="btn btn-block btn-primary add-to-cart-button">ADD TO CART · $20.00</button>
</form>
```

Three things matter to anything that scripts against it:

- **The form is `.project_create`**, not `.product-form`. The artwork/option elements are
  inside it, so `closest('form')` from a `px-option` reaches it reliably.
- **The button carries NO `type` attribute.** A `<button>` inside a form submits by
  default, but `form.querySelector('button[type="submit"]')` matches only the *literal*
  attribute and therefore **returns null**. This is a silent failure: the selector finds
  nothing, whatever depended on it quietly does not happen, and no error is raised.
- **Resolve it by text and class instead:**

```js
	/add[\s_-]*to[\s_-]*(cart|basket|bag)/i
```

  tested against the element's `textContent`, `value`, `aria-label`, `title`, `id` and
  `className`. That matches both the visible label and the `add-to-cart-button` class.

**A disabled button ignores `.click()` silently.** Anything that programmatically submits
the product form must re-enable the button first, and must therefore be able to find it.
The two failures compound: a resolver that returns null cannot re-enable anything, so the
click is swallowed and the customer stays on the product page with no error shown.

### Reading the product price from JavaScript (2026-08-10)
**Status:** Confirmed. Applies to any custom tool or snippet mirroring the live price.
- The price is rendered by a `px-product-price` web component, not by static markup.
  Read that element, and fall back to its `initial` attribute.
- **Do not scrape `.product-price` text.** On any product with
  `product.custom.regular_pricing` set, `product/product-details` renders the struck-through
  pre-discount price **first** inside the same block, so a non-global regex returns the wrong
  number. Verified: a product selling at $29.00 reported $45.00. If a text scrape is
  unavoidable, strip `<s>` and `<del>` content first.
- **A `MutationObserver` must sit on the `px-product-price` element itself.** It replaces its
  own contents, so watching a parent leaves the reader stale after a variant change.
- **Do not compute per-unit price by dividing total by quantity in JavaScript.** It disagrees
  with the platform on rounding, and on any ladder with a fixed component it is a different
  number. When `display_each_pricing: true` is set on the product, the page already renders
  the platform's own per-unit figure in a second `px-product-price` instance carrying
  `unit-price="true"` — read that.

### Hide, don't replace, a computed price (2026-09-09)

Extends the price-reading entry above. The four reading traps — read the `px-product-price`
custom element directly, fall back to its `initial` attribute then to a `.product-price`
scrape, strip `<s>` and `<del>` from any scrape because `product/product-details` renders the
struck regular price alongside the sale price when `regular_pricing` is set (verified: a $29.00
product reporting as $45.00), and take per-unit price from the second instance carrying
`unit-price="true"`, which needs `display_each_pricing: true` **on the product** — are all
recorded above and are unchanged. What follows is the writing side.

**Leave `px-product-price` in the DOM and merely hide it** with an inline style, once a figure
has been computed successfully, restoring it whenever the total quantity is zero. If the added
code ever throws, the platform's own figure is still on screen and the button looks untouched.
Select the **in-button instance specifically** —
`.add-to-cart-button px-product-price:not([unit-price])` — so a `display_each_pricing` unit-price
instance is left alone.

Compute from Liquid-supplied numbers (`data-value-price` on each input, `{{ product.price |
plus: 0 }}` as the base), never by parsing the component's rendered text.

**Any duplicated price arithmetic drifts, and the drift conditions belong next to the code.**
The figure becomes wrong the moment the product gains any of:

- a quantity break or tiered pricing ladder on the base price
- a second option carrying a price
- `regular_pricing` strike-through discounts
- a Ruby pricing formula doing anything other than a flat per-unit rate

For a flat base plus per-value adders it is exact. Beyond that, pull it. Same warning as the
"do not divide total by quantity" rule above, and it applies with equal force.
*Verified live on a child site (adult sizes billed at $17 against a $15 base while the button
read $15 — a display fault only, the cart was correct), 2026-08-20.*

### A price shown outside `px-product-price` must include default-variant surcharges (2026-09-27)

On a collection-driven PDP (`product/details-filter-dual-mode`) each size is a separate sibling product, and reading a sibling's price in Liquid gives its **base** price. The figure the shopper sees in `px-product-price` is the base **plus** the surcharge of every preselected default variant. Where a default variant carries a surcharge (for example a mounting option preselected on the smaller sizes), a size pill or "from" label built from sibling base prices disagrees with the headline price on the same page.

- Before shipping any price-in-pill or listing price, check whether any default variant carries a surcharge. Either add the default surcharges into the figure, or make the no-surcharge value the default (a commercial decision for the lab, not a template one).
- When auditing a product, read which radios are checked, not only the base price.
- **Quick check:** on the PDP, compare the `px-product-price` element's `initial` attribute with its shadow-root text. Different values mean a surcharged default.

Template-level (Shopper 24). *Verified by query on live child PDPs, 2026-09-24 and 2026-09-27.*

### Gallery arrows and driving the platform gallery (2026-09-09)

**Stock Shopper hides the gallery arrows until hover, which on touch means never.** That is why
a second image goes unnoticed on mobile. An always-visible override is correct, but **scope it**
— `:has(.px-item + .px-item)` — so single-image galleries keep the clean look.

**Do not reach into the gallery's IIFE.** `product/gallery/standard` ends in an IIFE that owns
scroll, snap, arrows and `data-selected-idx`, and exposes nothing. To move the gallery from
outside, **dispatch a click on its own thumbnail**: the gallery binds its click handler to the
gallery **element** rather than the navigation div, specifically so it survives fragment
reloads, so a bubbling click drives it through its own code path.

Keep external buttons in step with a `MutationObserver` on `data-selected-idx`, which also
covers the customer swiping or using the arrows.

**The auto-switch pattern (a value carrying a preview-face marker moving the gallery on
selection) is not verified live** — written and render-tested offline, nothing loaded on a live
Shopper URL.

### Editor locale needs editor-namespace translations imported (2026-09-09)

Passing the locale in the theme `setup()` call and enabling the language under
**Settings → Translations** does **not** translate the editor. Editor translations live in the
**`editor` namespace**, which is not populated by enabling a language: they must be **exported
from a site that already has them and imported here**.

Admin path: `/admin/translations?namespace=editor&locales[]=<iso>`

*Stated by the core developer, not independently verified.*

### A child's CMS backup can hold a stale copy of a parent snippet (2026-08-10)
**Status:** Confirmed on a live child site.
**Symptom:** A snippet present in the child's CMS backup is **not** what the site renders.
Observed on `collection/collection-filters-static`, where the backup's version built product
cards one way and the live render (inherited from the parent) built them another, with markup
and scripts in the live version that appear nowhere in the backup.
**Cause:** The child had no active override. The parent's newer snippet was rendering, and the
backup carried an inherited copy from whenever it was last synced.
**Two consequences:**
1. The backup is not a reliable picture of the parent. A snippet being *present* in the
   backup does not mean its *content* matches the parent's.
2. "Copy it down and edit it" silently reverts parent improvements — filter fixes,
   accessibility work, new card fields — with nothing in the diff to notice, because the diff
   is against the stale copy.
**Rule:** Before overriding any snippet, get its current source from the parent admin or from
the rendered page, never from the child's backup. Then change only what must change and leave
the rest verbatim.
**Prefer not to override at all when the change is presentational.** Scoped CSS on the
section wrapper does the job without freezing hundreds of lines of parent logic, and stays
correct when the parent snippet moves on.
**Corollary:** the parent-first rule (a child can only override a snippet the parent already
has) still holds, but the inverse does not — a snippet **absent** from the child's backup may
well exist on the parent and be perfectly legal to override for the first time. Confirm from
the parent admin rather than treating absence as proof it does not exist.

### Parent defects that silently disable a setting (2026-08-26)

1. **`font-body` is matched lowercase only.** `html.head` loads the Google Fonts link when
   `admin/checklist/font-body` is exactly `lato`. `pages/setup/storefront` writes `lato` /
   `open-sans`, but **`pages/manage/branding` writes `Lato` / `Open Sans` with capitals**, so
   setting the body font from the branding page does nothing.
2. **The kiosk idle-screen logo test is inverted.** `kiosk/idle-screen` tests
   `has_logo != blank`, but the parent ships `update-website-logo` = `FALSE`, and
   `'FALSE' != blank` is true — so the idle screen renders `header/logo` on every site that
   explicitly said it has no logo. The correct test is `has_logo == 'TRUE'`.
3. **`admin/checklist/admin/checklist/kiosk-picker-idle-seconds` exists** as a double-prefixed
   snippet path, a creation typo. `kiosk/style` also has a doubled `}` closing the token block.

### A trailing newline in a value snippet silently breaks every flag (2026-08-24)

**A value snippet written into a CMS tar must be byte-exact against the parent, including its
trailing newline or the absence of one.** Checklist and other value snippets on shopper24 carry
**no trailing newline**: the body of `admin/checklist/search` is exactly `TRUE`, four bytes.

The parent reads them by capture-and-compare, and `capture` does not trim. A generator that
writes `"TRUE\n"` produces a capture of `TRUE\n`, `TRUE\n == 'TRUE'` is false, and **every
comparison of that flag fails, silently, forever.**

The signature sends you the wrong way: the tar imports with no error, the snippets are present
in admin at the right paths with the right descriptions and values that look correct on screen,
the logo and theme colours and contact details are all visibly right — and the homepage, the
promotion bar and the logo position are all still the parent template's. The natural reading is
"checklist flags do not import". They import perfectly; they just never match.

**The tell:** interpolated values are unaffected, because a trailing newline in printed output
is invisible. Only compared values break. So if the brand colours took and the flags did not,
this is the bug, every time.

Anything read through `{% capture %}` and tested with `==` is affected — in practice the whole
of `admin/checklist/*`, and any `style/*` or `website/*` value used in a conditional rather
than emitted. When unsure, assume compared and write it byte-exact. The one-line generator fix
is to carry the seed's own trailing whitespace:

```python
seed_tail = seed_body[len(seed_body.rstrip("\r\n")):]
body = body.rstrip("\r\n") + seed_tail
```

Two companions found in the same session:

- **Which navigation style renders is an admin setting a tar cannot read or set.** Overriding
  one `navigation/style*` and writing "confirm the style" into a checklist has produced a
  wrong-navigation first delivery repeatedly. Override **every** navigation style the parent
  ships, and put anything that sits in the header beside the nav — currency picker, language
  switcher — into all of them too.
- **`admin/checklist/no-index` ships `TRUE` on the parent.** Any live storefront needs it set
  to `FALSE`. Nothing on the page shows it.
- **Build stamps.** Every structural override should carry
  `<!-- <slug>-build <date> :: <snippet name> -->` as its first body line. It survives into the
  rendered page and turns "is this surface mine or the parent's?" into a view-source check, and
  it names which navigation style is actually rendering.

### `default-delivery-option` values are `public` / `private` (2026-08-24)

`admin/checklist/default-delivery-option` controls which delivery method is preselected on
`/site/checkout`.

| Value | Effect |
|---|---|
| `public` | Preselects **In-store pickup** |
| `private` | Preselects **Deliver to my address** |
| `none` | Nothing preselected — the shopper must choose (parent default) |

The naming is **address type, not delivery type**: a public address is a store or pickup
location owned by the site, a private address is the customer's own. Anyone guessing from the
checkout UI would try `pickup` / `delivery` and get silent no-ops, because the radios carry ids
`checkoutPickup` / `checkoutDelivery` and `value="on"`.

Proved by `account/v2/order-details`
(`{% if order.address.is_public %}Pickup Address{% else %}Shipping Address{% endif %}`) and by
`checkout/shipping-options`, which renders shipping services only
`{% unless cart.address.is_public %}`.

**No Liquid snippet in the shopper24 tree consumes this key** — the platform's own checkout
page reads it, like the `/site/cart` shell. It cannot be traced by grepping snippets. The
parent ships it as `none` with an **empty Description**, which is why the value set is
undiscoverable from the CMS; any site override should carry a Description listing the accepted
values.

Two traps: the trailing-newline rule above applies, and the importer is wipe-and-replace, so
setting this in admin and later importing a CMS bundle that does not carry the override
silently reverts it to `none`. Related keys: `pickup-in-store` (`TRUE` enables the pickup
option at all), `disable-delivery`, `display-shipping-options`,
`dont-require-pickup-contact-details`. Setting a default does not remove the other option.

Sites migrated to the r3w settings architecture read this through `shopper/config` and the
`shopper_settings` Custom Type delta rather than the checklist snippet — same values, different
storage.


### Live `selectors:` re-renders patch the DOM, they do not replace it

A live form re-render (`selectors:` on an async form) is applied with a DOM-diffing patch. **An unchanged node survives**, so a one-shot hook on it (an `onload` on a `<style>`, an insertion-triggered script) fires once and never again while the node is unchanged. Any automatic write driven this way must (a) wait for `.px-live-fragment-loading` to clear before posting, because concurrent `cart_update` posts can each save a stale `custom` hash and wipe each other's field, and (b) re-check its own result and retry a capped number of times rather than relying on re-insertion. Give the hook element a stable `id` so a retry can find the current node. *Verified by reading the live platform bundle, 2026-09-21.*

## 18. Generated CSS Is Appended After `style/custom.css`

`/site/custom.css` is not just the `style/custom.css` snippet. Shopper serves one stylesheet
containing that snippet **followed by** rules it generates from the `style/color-*` value
snippets. The generated rules carry `!important` and use three-class selectors. Confirmed in
the live DOM, all from the same sheet, in this order:

| Order | Selector | Value | Source |
|---|---|---|---|
| 1 | `a` | site token | the snippet |
| 2 | `.navbar .nav-link` | site token `!important` | the snippet |
| 3 | `.brand-nav-cta .nav-link` | `#ffffff !important` | the snippet |
| 4 | `a` | generated colour | generated |
| 5 | `.navbar-light .navbar-nav .nav-link` | generated colour `!important` | generated, from `style/color-font` |

Rule 5 is `(0,3,0)` with `!important` and comes last, so it beats rules 2 and 3, which are
`(0,2,0)`. **Adding `!important` to a two-class selector does nothing about this** — the
generated rule already has it and wins on specificity.

**The rule: any nav colour set in `style/custom.css` must out-specify
`.navbar-light .navbar-nav .nav-link`.** Scope under `.navbar-light .navbar-nav` to reach
`(0,4,0)`:

```css
.navbar-light .navbar-nav .nav-link { color: var(--brand-ink) !important; }
.navbar-light .navbar-nav .brand-nav-cta .nav-link { color: #ffffff !important; background-color: var(--brand-accent) !important; }
```

Observed cost of getting this wrong: a nav CTA rendering the generated font colour on the
brand background at a contrast ratio of **2.28:1**, live and unnoticed for six days. After the
specificity fix, 5.88:1.

Assume the same applies to anything else the platform generates from a value snippet. Before
assuming a custom colour has landed, read the computed style off the live element rather than
the snippet. Thirty-second diagnostic in the console on the live page:

```js
const el = document.querySelector('.brand-nav-cta .nav-link');
Array.from(document.styleSheets).flatMap(ss => { try { return Array.from(ss.cssRules) } catch(e) { return [] } })
  .filter(r => r.selectorText && r.style && r.style.color && el.matches(r.selectorText))
  .map(r => [r.selectorText, r.style.color, r.style.getPropertyPriority('color')]);
```

The last matching rule with the highest specificity wins. Reading the snippet source will not
tell you this, because the snippet is only half of the served file.

Related, same cause: `theme.min.css` carries
`.navbar-light .navbar-nav .nav-link { text-transform: capitalize }` at higher specificity, so
a nav label written in sentence case renders title-cased unless the override matches that
specificity.

---

### 18.1 The parent's own CSS in `pages/custom.css` also outranks a child

The ordering in § 18 is not limited to generated color rules. The parent's `custom.css` page also carries the parent's own large feature-build CSS (one observed block ran to about 7,000 lines), printed **after** the child's `style/custom.css`. At equal specificity the parent's rule wins, including over a child's `!important` when the parent's rule is also `!important`. Observed offenders include an unscoped `h1, h2, h3, h4 { margin: 0 }` reset and `!important` colors on `.btn-primary` and form inputs.

**Rule for a child theme:** prefix a global element or Bootstrap-class override with one extra type selector (`body h1`, `body .btn-primary`) to outrank the later parent rule. A class of the child's own is unaffected. Confirm with the live computed style, not by reading either file. *Verified by reading source, shopper24, 2026-09-23.*

**The unscoped heading rule sits in the film builder block and does more than reset margins.** The parent `custom.css` page carries, inside the film builder (`pxfb`) CSS but not scoped to `.pxfb`:

```css
h1, h2, h3, h4 { color: inherit; font-family: var(--font-display); font-weight: var(--display-weight); letter-spacing: var(--display-tracking); line-height: 1.08; margin: 0; }
```

The three variables are defined only inside `.pxfb` scopes. Everywhere else they are undefined, so on every Shopper 24 page headings fall back to the inherited body font, `font-weight` falls back to normal (400), `margin` is 0 and `line-height` is 1.08. A child `h1, h2, h3 { font-family: ... }` loses to it at equal specificity.

- **Child workaround:** define the variables on `:root` in the child `style/custom.css` (`--font-display: <display stack>; --display-weight: 700; --display-tracking: -0.01em;`). The `.pxfb` scopes still override them for the film builder. The `body h1` prefix above also works.
- **Proper fix (parent, needs Alex):** scope the rule to `.pxfb h1, .pxfb h2, .pxfb h3, .pxfb h4`, then check Shopper 24 sites for headings that lost weight or spacing since the block was added.
- Same file: at `max-width: 767.98px` the parent sets `.btn { font-size: 90% }`, which beats a child button class of equal specificity. Use `.btn.<class>` in child CSS.

Template-level (Shopper 24 parent). *Verified by query (CSSOM rule list on a live child page) and by reading source (parent backup), 2026-09-29.*

### 18.2 Dark child themes: the parent styles the tokens do not reach

Template-level (Shopper 24 parent CSS), so every dark child hits these. The fixes live in the **child's** `style/custom.css`; nothing changes on the parent. Found on a dark child, 2026-10-01; the full checkout was confirmed reading correctly on dark after the fixes. **The checkout is high risk: after any change here, walk the whole checkout on the live site** (address, delivery, payment, store-closed notice) before handing over.

| Where | What the parent hard-codes | Child fix |
|---|---|---|
| Checkout page | `.checkout-page { background-color: #f6f6f6; }`, so every heading and label in the ink color disappears on a dark child | `html body .checkout-page { background-color: var(--color-bg) !important; }`, plus `.card` and `.list-group-item` borders and `.custom-control-label::before` for the radios |
| Checkout delivery icons | `icons/store.svg` uses `fill="currentColor"`, but `icons/ship.svg` has paths with no fill, so the truck renders black | `html body .checkout-page .list-group-item .ml-auto svg { color: var(--color-ink); }` and the same selector with `svg path:not([fill]) { fill: currentColor; }` (verified live) |
| Bootstrap modals | `.modal-content` is white, so the cart's store-closed notice and the password reset pop-up print ink on white | Style `.modal-content` dark site-wide |
| Cart quantity stepper | `.main.cart .quantity input { color:#111 }` and `.main.cart .input-group button { color:#222 }` (specificity 0,3,1) beat an `html body .quantity ...` override (0,2,3); the minus icon's path has no `fill="currentColor"` | Use `html body .main.cart .quantity ...` and set `svg path { fill: currentColor }` |
| Two-way toggle (`px-toggle`) | Selected label `#191c1d` and a white track, so on dark the selected label vanishes | Restyle the label names, the track and the knob |
| Photo upload window (`dialog.px-upload-dialog`) | White; its back and close icons are background-image SVGs with a dark fill, so `color` does nothing | `filter: invert(1)` on those icons. The QR code `img` sits in a fixed 180 px box; adding padding needs `box-sizing: border-box` or the bottom is clipped |
| Image swatches | Two outlines: the parent's ink outline on the hovered `img` plus the selected outline on `span.label-img` | On round swatches use one ring on the span with `border-radius: 50% !important` |

**General rule for dark children:** any parent SVG icon whose paths carry no `fill` renders black. When an icon vanishes on dark, check the snippet for `fill="currentColor"` before touching colors.

**Finding the winning rule** when a cross-origin stylesheet hides it from `document.styleSheets`: query the browser DevTools protocol (`CSS.getMatchedStylesForNode`), for example from Playwright.

## 19. Account v2 (`acv2`) Theming

Applies to any Shopper site running the v2 account area (`admin/checklist/account-v2-*`).

The account area is its own token-driven design system. It does **not** use Bootstrap card or
list markup; it renders a bespoke `acv2-*` class system from
`account/v2/{sidebar,dashboard,orders,projects,carts,galleries,addresses,info,dates,calendars,order-details,empty-state}`.
Its CSS is generated into `/site/custom.css` **after** the site's own `style/custom.css`
snippet (§18), so equal-specificity overrides lose. Site CSS must out-specify:

- tokens: `div.acv2 { … }` `(0,1,1)` beats their `.acv2` `(0,1,0)`
- components: `div.acv2 .acv2-card` `(0,2,1)` beats `.acv2-card` `(0,1,0)`
- stateful: `div.acv2 .acv2-sidebar__item.is-active` `(0,3,1)` beats `(0,2,0)`

**One rule retokenises the whole area.** The platform block defines `--acv2-accent`,
`--acv2-accent-soft`, `--acv2-text`, `--acv2-text-muted`, `--acv2-bg-page`, `--acv2-bg-card`,
`--acv2-border`, `--acv2-radius`, `--acv2-radius-lg`, `--acv2-shadow`, `--acv2-shadow-sm` on
`.acv2`. `--acv2-accent` and `--acv2-text` are generated from `style/color-primary` and
`style/color-secondary`, so brand colour and ink already inherit; usually only the surface
tokens need restating. Stock defaults that clash with most brand systems:
`--acv2-bg-page: #f5f6f8`, `--acv2-bg-card: rgba(255,255,255,0.55)` with
`backdrop-filter: blur(14px)` (a glass look), `--acv2-radius-lg: 18px`. Glass must be removed
separately — `backdrop-filter` is a property, not a token:
`div.acv2 .acv2-card, div.acv2 .acv2-sidebar { backdrop-filter: none; }`

**Platform defect — `--acv2-accent-soft` is malformed.** The generator builds it by appending
an alpha suffix to the primary colour. Because the `style/color-primary` value snippet body
ends with a newline, the emitted token is literally `#RRGGBB\r\n1a` — an invalid colour. On
every site whose colour snippet ends with a newline (likely all of them) the stock active-nav
tint (`.acv2-sidebar__item.is-active`) and `.acv2-pill--primary` background both resolve to
transparent. It reads as "the active nav item has no highlight" rather than as an error. The
fix belongs in the generator (trim the value before concatenation); the site-side workaround is
to define the active state yourself.

**Buttons.** The theme emits generated button colours as
`.btn-dark { background-color: … !important; border-color: … !important }` and the same for
`.btn-primary`, so selector weight alone will not override them — those two need `!important`
too. `.btn-outline-secondary` and `.btn-light` carry no `!important` and override normally.
`btn-dark` does double duty in the account markup: `<a class="btn btn-dark">` is navigation and
`<button type="submit" class="btn btn-dark">` is the page's primary action. Split them by
element selector rather than restyling both.

**Inline styles in the parent snippets.** `account/v2/dashboard` writes `border-radius: 12px`,
`background: #f5f6f8` and `background: #eee` inline on the project thumbnail and gallery tile
wrappers. Only an attribute selector plus `!important` reaches them:
`div.acv2 [style*="border-radius: 12px"] { border-radius: 4px !important; }`

**The stock dashboard types the user's name out.** `.acv2-typing` sets `max-width: 0` and
reveals the name via the `acv2Type` keyframes with a blinking caret. Disabling it requires
resetting the width too — `animation: none` alone leaves `max-width: 0` and the name
disappears: `div.acv2 .acv2-typing { max-width: none; animation: none; border-right: 0; }`

**Testing gotcha — transitions mask computed values.** `.acv2-sidebar__item` and `.btn` both
transition `background-color`. Injecting or mutating CSS on a live page and reading
`getComputedStyle` immediately returns the transition's **start** value, which looks exactly
like "my rule lost the cascade" and persists long enough to fool a delayed re-read. Even inline
`!important` appears to fail, because transitions sit above `!important` in the cascade.
Reliable check: append a **fresh** element carrying the target classes and read its computed
style — new elements get initial style computation with no transition, which is the page-load
case you are trying to predict.

**Copy, not CSS.** The stock account strings are consumer photo-lab voice and come through
`{{ '…' | t: ns: 'account' }}`. On a B2B or trade site they read wrong and no amount of CSS
fixes them. Whether they can be retargeted through the translations mechanism rather than by
overriding the parent snippets is **UNCONFIRMED** — do not promise it.

---

## 20. Analytics — GA4 Tagging Defects on the Parent

Audited 2026-08-22 against the shopper24 parent. All of these are parent-level and affect every
child site unless overridden.

**`view_item` is double-counted on a GTM site.** Shopper ships two tagging paths and both fire:
an inline `gtag("event","view_item",{…})` call, and a
`dataLayer.push({event:"view_item", ecommerce:{…}})`. On a site with a GTM container GA4
receives `view_item` twice per product-page view — once with a numeric Google taxonomy category
code and once with the readable category. Any product-view count or view-to-cart rate is
inflated roughly 2×. The three snippets carrying the duplicate are `product/product-details`,
`product/product-details-filter` and `product/details-filter-dual-mode`: each already renders
`product/design-now`, which owns the dataLayer push, **and** also ends with
`{% snippet 'integrations/google/event/view-item' %}`. `product/product-details-fullpage` is
already correct.

**Photo-prints pages emit no ecommerce event at all.**
`product/product-details-prints` does not render `product/design-now`, so its only `view_item`
is the gtag include — dead on any GTM site.

**`purchase` re-fires on refresh.** It is pushed unconditionally on render on `thank-you`,
`payment_success` and `confirmedorder`. A refresh re-fires with the same `transaction_id` and
GA4 counts it as new revenue. All three also fall back to
`user.orders | where: 'status', 'C' | first` when `order` is unset.

**Design products never joined item-level funnels.** `pages/editor-scripts.js` sent `item_id`
as `theme.code:product.code` — a different SKU from every other event. The theme belongs in
`item_variant`.

**`integrations/google/gtag` is orphaned.** Nothing includes it, yet its Description reads
*"Enter in your Analytics account id — replacing 'ABCDE12345' below"*. Editing it does nothing.
**The snippet that works is `website/gtag`.** The same applies to
`integrations/google/event/add-to-cart`, `…/begin-checkout` and `…/purchase`; keep only
`…/view-item` as the no-GTM fallback.

**Free funnel step.** GA4 enhanced measurement fires `form_start` with
`form_id=project_create` when a shopper launches the design tool — a "started designing" step
with no tagging work.

**More parent facts, from the 2026-09-28 parent backup** (verified by reading source):

- `html.head` loads GTM when the container ID is set, and gtag.js plus `gtag('config')` when `website/gtag` is set, **independently**. Nothing stops both.
- `integrations/google/event/view-item` calls `gtag()` and is included on `product/product-details`, `product/details-filter-dual-mode`, `product/product-details-filter` and `product/product-details-prints`. On a GTM-only site (the standard below) there is no `gtag` function, so it throws **`gtag is not defined`** in the console on every product page. Harmless to the dataLayer path, but noise in every console check.
- gtag.js ignores the plain `dataLayer.push({event: ...})` objects Shopper emits, which is why a `website/gtag`-only site records nothing past product views.
- The cookie banners (`cookie-consent/banner`, `gdpr-banner`) do **not** gate any tag, and there is no Consent Mode.
- `integrations/google/ads-id` is not in the parent backup and `html.head` never reads it. An Ads ID entered there loads nothing; Google Ads goes through the container.
- `ga_client_id` and `ga_session_id` are captured by `checkout/utm-cart-code`, which `pages/cart` includes. The UTM landing cookie `_pf_utm` is set by `integrations/google/utm-capture`. See `85_GA4_SERVER_SIDE_PURCHASE.md` § 2.

**Rule:** the dataLayer is already complete, so a new ecommerce event is added as a `dataLayer.push` only, never as a parallel gtag snippet.

### The technical standard: GTM only (decided 2026-09-02)

**Set the GTM container ID. Leave `website/gtag` blank.** Both fields sit on the Shopper custom admin page `manage/integrations` (layout `shopper-admin`), which also carries `integrations/google/ads-id` and `website/meta-pixel`. `setup/integrations` has no Google fields, and the older `setup` and `setup/dashboard` pages hide the GA4 field with `d-none` and point to GTM. *Verified by reading source (shopper24 backup), 2026-09-28.* The admin menu path **"Setup and Manage → Integrations"** used in earlier notes is **not verified**: Alex did not recognise it (2026-09-28). Do not give it to a customer until the real click path is confirmed.

| Field | Checklist key / snippet | Standard value |
|---|---|---|
| Google Tag Manager | `setup-google-tag-manager` → renders `integrations/google/tag-manager` | `GTM-XXXXXXX` |
| Google Analytics 4 tag ID | `website/gtag` | **blank** |

Why, in order:

- **The gtag path cannot carry the funnel.** Only `view_item` is wired through `gtag()`.
  `add_to_cart`, `begin_checkout` and `purchase` exist as gtag snippets but are **orphaned**
  (see the entry above). A `website/gtag`-only site gets page views and product views and
  nothing past them.
- **The dataLayer path is complete** — `view_item`, `add_to_cart`, `view_cart`,
  `begin_checkout`, `purchase` all push.
- **Both fields set is roughly 2× double counting**, which is the duplicate `view_item`
  recorded above.
- **Google Ads conversion tracking has no Shopper preset** and must go through GTM anyway.
  Search Console also verifies through the container with nothing pasted into the theme.

**The half-a-chain trap, and it is the first thing to check on any "analytics is connected but
I see nothing" report:** a container on the site is only **half** the chain. If the container
carries no GA4 event tags listening for those five events, GA4 shows traffic and **no revenue**.
The container ID being present in the page proves nothing about whether anything is being
recorded.

Related: when a site arrives with an unidentified tag, read the live page to find which
container is loading and which measurement ID that container carries **before creating
anything**. A new property created next to a working one is the expensive mistake, not a week
of lost history.

*Verified by live inspection of a client storefront, 2026-09-02.*

### Cross-reference: `purchase` is also sent server-side on myPixfizz brands

On brands wired to myPixfizz, `purchase` is **additionally** sent server-side through the GA4
Measurement Protocol from the Pixfizz order webhook. See `70_MYPIXFIZZ_OVERVIEW.md` and
`85_GA4_SERVER_SIDE_PURCHASE.md`.

**Consequence for this file:** on those brands, leaving `purchase` in the GTM eCommerce trigger
regex **double-counts revenue**. Check the container's trigger regex before trusting any
revenue figure from a myPixfizz-wired brand.

**Missing `_ga` cookie — a storefront fault that looks like a server-side one.** When the
storefront never captured a `_ga` cookie, the server-side function invents a `client_id`
(`pixfizz-<order_id>`) and omits `session_id` entirely. Those purchases land in GA4 as
**Direct / (not set)**. Revenue still counts; attribution does not.

The root cause is a **broken storefront analytics bootstrap** — no gtag stub, no dataLayer
push, so GA4 never initialises and there is no cookie to read. **Fixing the storefront snippet
fixes that brand's attribution, and nothing server-side can.** One client lab in the window had
18 of 19 purchases in this state.

*Verified by query against the myPixfizz database, 2026-09-02.*

### GA4 bridge for gtag-only sites (built, not deployed)

**Status, 2026-09-28: built and unit-tested, NOT deployed to the parent and NOT tested on baseline. Do not describe it as live.** The design: a new `integrations/google/ga4-bridge` rendered in `html.head` only when `website/gtag` is set and the GTM container is blank. It wraps `dataLayer.push` and forwards Shopper's GA4 ecommerce events to `gtag('event')`, sends `view_item` once per product per page and `purchase` once per transaction, skips `purchase` when a new checkbox `admin/checklist/ga4-server-side-purchase` ("myPixfizz sends purchases to GA4") is `TRUE`, and offers `?pf_ga4_debug=1` for DebugView. Not verified: that the real gtag.js keeps the wrapped push. A rollout check found every myPixfizz-wired brand already in GTM mode, so the bridge would stay off for them. The GTM-only standard above still applies.

### 20.1 Klaviyo integration and email consent

Template-level (Shopper 24). *Verified by reading source (shopper24 backup 2026-09-24), and live on a client site and its Klaviyo account, 2026-09-30.*

- **Loading and identify.** `layouts/index` loads `klaviyo.js?company_id=<integrations/klaviyo/api-key>` when `admin/checklist/activate-klaviyo` = `TRUE`, and identifies logged-in users with email, first name and last name only.
- **What is not sent.** No consent flag, no Started Checkout, no Placed Order, and no Fulfilled, Cancelled or Refunded events. Order events in Klaviyo need a separate feed from the order webhook.
- **Viewed Product** is included only in `product/design-now`, so static product pages send nothing. Its `ProductID` is `product.code`.
- **Added to Cart** is inline in `pages/cart` (identical to `integrations/klaviyo/added-to-cart`) and fires on **every cart page view**, not on add. `AddedItemPrice` reads `itemPrice` (wrong case, always undefined); `ProductCategories`, `ImageURL` and `ProductURL` are never set; `ProductID` is the numeric `product_id`, which does not match Viewed Product.
- **Escaping.** `product.name` and `user.first_name` are printed unescaped inside JS strings, so a double quote in a name breaks the script.
- **Email consent.** `user[custom][newsletter]` comes from a pre-ticked registration checkbox that is commented out in both `account/login` and `account/login-form`, so it never renders: **registration collects no marketing consent**. `user[custom][email_opt_in]` and `sms_opt_in` are on the account info pages, gated by `admin/checklist/activate-email-opt-in` and `activate-sms-opt-in`. Neither field syncs to Klaviyo, and the footer newsletter form is a hidden placeholder.

---

## 21. Parent-Safe Changes to Shopper 24

The shopper24 parent drives every lab. Everything in this section exists because a change there
is a change to ~50 storefronts at once.

### 21.1 `collection_filters` has two syntaxes

**This was a KB gap and it cost most of an afternoon.** Recorded here so it does not cost
another one.

| Consumer | Page | Fields |
|---|---|---|
| `collection/collection-filters` | the standard shop page | **three** fields |
| `product/details-filter-dual-mode` | a collection using `pdp_layout` | **five** fields |

The five-field form is:

```
label | url_name | filter_attribute | default_value | snippet_args
```

`snippet_args` is a set of `key: value` pairs joined by pipes, and is consumed by
`product/filter-controls`. With `asset_images: true` the **field value must be an asset
filename, not a label**.

Using the three-field form where the five-field form is expected fails in the way filter
configuration always fails — quietly, with the control rendering and doing nothing.
*Verified live on a client PDP, 2026-09-08.*

**Scope: wall art style ranges only, never everyday prints** (standard stated by Alex,
2026-10-04). Five-field `collection_filters` size pickers are for ranges sold one size per
product on a `pdp_layout` shop page: canvas, fine art, metal, acrylic, wood and similar.
Everyday (cut) prints never get `collection_filters` and never go through a `pdp_layout` shop
page; they go straight into the photo prints bulk workflow (for example `/site/prints`).

**`show_prices: true` on the size line, as standard** (stated by Alex, 2026-10-04). Every
`pdp_layout` `collection_filters` setup carries `show_prices: true` on the size line, with
radio tiles, so each size tile shows its price. It also works on dropdowns (`8x10 ($X.XX)`).
The tile price is the **base price** of the product the option leads to, before variants (so a
surcharged default variant is not included, see § 17). The code is on the shopper24 parent
since 2026-10-04 (`product/details-filter-dual-mode` and `product/filter-controls`, plus
`.px-filter-price` CSS on the parent `custom.css` page). A site with its own **Override
Snippet** of either snippet does not get it: list the site's overrides before relying on it.
After any setup, load the page logged out and look for `px-filter-price`. *Verified by query on
a child site, 2026-10-04.*

Reference setup (metal prints, size default chosen per orientation by Liquid):

```
Orientation | orientation | product.custom.orientation | metal-portrait-thumb.jpg | control_type: radio | asset_images: true
{%- if request.params['orientation'][0] == 'metal-portrait-thumb.jpg' or request.params['orientation'][0] == blank %}
    Size | size | design.name | 5x7 | control_type: radio | show_prices: true
{%- elsif request.params['orientation'][0] == 'metal-landscape-thumb.jpg' %}
    Size | size | design.name | 7x5 | control_type: radio | show_prices: true
{%- elsif request.params['orientation'][0] == 'metal-square-thumb.jpg' %}
    Size | size | design.name | 8x8 | control_type: radio | show_prices: true
{% endif %}
```

- `collection_filters` is Liquid-rendered.
- The orientation switch sends `orientation[]=<value>`, which is why the Liquid reads
  `request.params['orientation'][0]`.

**Size tile order is the collection's Design Products order**, not size order. Products added
later land at the end of the list, so a range built in two passes shows the second pass last
(4x6 after 40x60), and an alphabetical bulk add gives 16x20 before 8x10. Fix in admin:
Products > Collections > <collection> > Design Products. Each row's arrow handle opens a popover
with **Move to Top** and **Move to Bottom**, and rows can be dragged; every move saves
immediately (it POSTs `/site/<site>/admin/theme_categories/<id>/order`), with no Save button.
Fastest full re-sort: click Move to Bottom on every row in the target order (short side, then
long side, so each orientation ascends). When bulk-adding through the API, add in size order,
because rows append in call order. Add to every launch audit: load each `pdp_layout` collection
logged out and check the size tiles ascend in every orientation. Platform admin behavior plus
template rendering. *Verified by query on a child site (two collections, 67 rows), 2026-10-04.*

**Pasting a multi-line `collection_filters` value by hand can drop its first line.** On one
canvas collection the Orientation line was lost, and the page then listed every size of every
orientation. After any filters paste, count the orientation and size inputs on the live page.
*Observed live, 2026-10-01.*

### 21.2 A free-form arguments string is an opt-in that needs no code and no deploy

Parent snippets that take a free-form arguments string **parse it at the top into named
values**. `product/filter-controls` does exactly this, and the string comes from **admin data**,
not from a snippet body. So **adding a new argument name is an opt-in that needs no code change
and no deploy** on any site that does not use it — the sites that do not pass it are, by
construction, unchanged.

Three properties make such a change parent-safe, and each has to be **proven rather than
asserted**:

1. **Opt-in, not opt-out.** Grep the corpus to confirm the new flag string appears **nowhere**
   before shipping. If it already appears anywhere, it is not an opt-in.
2. **Degrade to byte-identical output.** Render the current parent against the patched parent
   across fixtures covering every branch, and assert a marker per fixture proving it reached
   the branch it names. (A harness that passes everything while every fixture silently falls
   through one branch is the normal first result — the marker assertions exist because of it.)
3. **Preserve parallel-array alignment.** Where the parser builds parallel arrays, push the new
   array **outside** the conditional, so an entry exists for every row whether or not the new
   argument was supplied.

**Unverified and it decides the design:** whether `{% snippet %}` passes **only** its named
arguments, or whether the callee inherits caller scope. If arguments are isolated, a generic
argument name is safe; if scope is inherited, every new name must be namespaced against
collisions with whatever the caller happens to have assigned. A two-minute test on baseline
settles it: assign a distinctively named variable in a caller, render a snippet that does not
declare it, and see whether it resolves. **Namespace until proven otherwise.**

**Partly answered (2026-10-06).** In **email templates** a snippet receives only its named
arguments plus `website`; a caller's assigns do not reach it (verified by query in admin
Preview, 2026-10-03, `50_LIQUID_REFERENCE.md`). The storefront case is still not tested, and
a Pages `page_content` caller behaved differently again (§ 14), so keep namespacing.

### 21.3 The admin action is **Override Snippet** — not copy, fork or duplicate

The action that creates a site-level version of a parent snippet is called **Override
Snippet**. Use that exact name in any instruction; the others send people looking for a control
that is not there.

What the action does, stated on client calls (2026-09-28): **Override Snippet** starts the site's copy from the parent snippet's current body, the override applies to that one site only, and it is live as soon as it is saved. A browser still showing the old version needs a hard refresh. Pages cannot be overridden or edited on a child at all (`13_TEMPLATE_BOUNDARIES.md`).

Two consequences to state **every time** an override is instructed:

- **An override pins that snippet.** The site stops inheriting parent changes to it, silently
  and for good. Every later parent fix to that snippet — filter fixes, accessibility work, new
  card fields, SEO corrections — stops arriving, with nothing in any diff to notice.
- **A promotion to the parent is only finished when the override is removed.** Moving a change
  up to the parent while leaving the child override in place means the child is still running
  the old copy and the parent version is never exercised.

Prefer **not to override at all** where the change is presentational — scoped CSS on a wrapper
does the job without freezing hundreds of lines of parent logic (see §17).

**On a snippet edit page the first `_method` form is DELETE.** Its inputs are `_method=delete`,
a submit button and the token, and nothing on it says it deletes. The content form is the one
carrying `_method=patch` and `snippet[name]`. Submitting the first form by script deleted a
child override on 2026-10-06. Before posting any form from a snippet edit page by script, assert
`_method === 'patch'`. Same trap as the template option edit page (`18_ADMIN_NAVIGATION.md`
§ Bulk Update Tools). Platform-level (Pixfizz CMS admin). *Verified by use, 2026-10-06.*

### 21.4 Everything ported into the parent ships gated, off by default

*"Shopper is being used by many sites. So let's make very safe upgrades that are always gated
behind an option we can turn on."*

Any change ported into the shopper24 parent ships **behind an option that is off by default**,
because the parent drives every lab. A child that changes nothing must render byte-identical
output.

The working method for deciding what to promote: hand over **three backups side by side** — the
current shopper24 parent CMS backup, the site's own Shopper CMS backup, and the non-Shopper
custom CMS backup where the upgrade was developed — diff the individual snippets, then propose
what to promote.

Note the mechanics that make the gate real: the flag snippet is created **on the parent** with
the off value and **Allow Override** ticked, then overridden `TRUE` on the child. See
`52_SNIPPET_INVENTORY.md`, *Creating a new snippet*.

### 21.5 A child that overrides `pages/custom.css` wholesale inherits no parent CSS

Any child site that overrides `pages/custom.css` wholesale **will not inherit** CSS blocks added
to the parent's copy, and needs them pasted into its own file. Check before promising a child
that a parent CSS change will reach it — a child with no `pages/` directory in its backup
inherits the parent file and is fine.

The parent `pages/custom.css` already calls `{% snippet %}`, so it **is Liquid-rendered** and
can carry gated blocks, not just static CSS. *Verified by reading source — shopper24 CMS
backup, 2026-09-08.*

### 21.6 A site-level checklist key set on the parent moves for every child

A key that is **site level** belongs on the **child site** or on the **product**, not on
shopper24, unless the value is genuinely the house default.

Observed consequence: a bleed key left empty on the parent meant artwork built to one bleed was
checked at another. Because that check is **warn level rather than fail level**, it generated
noise on valid files rather than rejecting them — which is probably why it went unnoticed for
weeks. A key that silently mis-checks is worse than one that is obviously off.

---

## 22. SEO Defects Found and Fixed at Parent Level (2026-09)

Recorded as a list because every one of them is **generic** — they were parent-level, so they
applied to every child site, and the same classes recur on any new template work.

- **Paginated listings were invisible to crawlers.** Paginated product and static listings —
  pages 2 and beyond — were being missed by every search-engine tool. Fixed on the parent.
- **A footer link hard-coded to `http://` multiplied into hundreds of phantom broken pages**,
  one per originating page. A single wrong scheme in a sitewide component scales with the page
  count.
- **The Google reviews widget ships without structured-data markup.** It renders, and it is
  correctly connected, but it is **never picked up as a rich snippet** because there is no
  markup for a crawler to read. Existing Google Reviews help articles predate this finding and
  are now incomplete.
- **Snippet headings used `h3` with no `h1`.** A hidden `h1` was added.
- **Preview modules and pop-up design tools are not recognised as product imagery** by
  crawlers, so the page reads as having **no images at all**. Remedy: ship two or three static
  images alongside any preview module.
- **Sitemap and robots.txt must be explicitly enabled.** They are not on by default. Shopper
  admin also carries AI-search settings that expose descriptions and summaries to AI
  assistants.
- **Legacy kiosk landing pages get crawled and scored as broken.** A split-screen film page
  with no menu links was indexed anyway. Anything reachable is crawlable regardless of whether
  it is linked.

*Stated from a working session, not independently verified against a live crawl.*

**Footer sitemap link with no protocol (2026-10-05).** Child footers copied from the parent
carry `href="{{ website.hostname }}/sitemap.xml"`, which has no protocol, resolves as a relative
path and 404s. Use `/sitemap.xml`. `/sitemap.xml` itself exists only after the first run of
Admin > Website Crawls. Template-level, likely on the parent; seen on two child sites.
*Verified live, 2026-10-05.* See `81_SEO_AND_GEO_REFERENCE.md` Part G.

---

## 23. Static Product Collections — Filtering and Aggregation

- **A parent collection does not aggregate its subcollections' products.** It lists its own static products plus subcollection cards; every product must sit directly in the collection meant to list it.
- **`collection/collection-filters` can filter on `category` with no custom field.** Its three-field syntax maps `filter_attribute` over `collection.static_products`, and `category` is a Product property the Static Product Importer already writes.
- **The collection still needs a `collection_filters` custom field definition (snippet type)** on the site before any collection there can be given a filter.
- **Collection paths are `/`-separated** (`film/processing`); `collection.path | split: '/' | first` gives the top-level parent.
- **Deep links that preselect every choice** on a collection rendered as a single product with filters (PDP layout): `<url_name>%5B%5D=<value>` per filter, for example `size%5B%5D=` and `orientation%5B%5D=`, with the filter value exactly as stored (including any stray text or trailing spaces, URL-encoded), plus `variants%5B<variant type>%5D%5Bvalue%5D=<variant value>` once per variant type. Portrait sizes resolved without the orientation parameter; landscape and square sizes needed it. Landing pages, emails and ads can link straight to a configured product. Platform-level. *Verified live on a child site, 2026-09-24.* Still not verified: the same shape on a three-field `collection/collection-filters` listing page.

*Verified by reading source (shopper24, 2026-09-18 backup) and a live catalogue build, 2026-09-23.*

**Static product page.** URL: `/site/product/c/<collection path>?product=<id>-<slug>`. The form posts to `/cart/add_product` with `product_id` and `variants[<code>]`. There is no quantity box on the page; quantity is set in the cart (`orderline[quantity]`). A POST that includes `quantity` adds that quantity. Template-level. *Verified 2026-10-05.*

## 24. Product URLs, Titles, Shop Tiles and Card Prices for Design Products

Template-level (Shopper 24) unless marked.

- **Clean product URLs.** The design custom field `url_path` (fallback `url_slug`) makes the collection card link `/site/product/<collection path>/<url_path>`, served by the parent wildcard page `product/:collection-level-1/:url-path`. Works in `collection-filters` and `collection-load-more`. Without it the link is `/site/product/c/<collection>?product=<id>-<slug>&theme=<id>-<slug>`. `url_path` must be unique on the site: clear it on the old design before giving it to a new one. Menu links pointing at the pretty URL then follow without a menu edit. *Verified by query, 2026-10-03 and 2026-10-06.*
- **Product page title and H1** come from the design's `meta_title` when set. Without it the wildcard page title is design name + product name (doubled when they match). *Verified by query, 2026-10-03.*
- **Collection `title_format` / `breadcrumb`** are text fields; `product` makes the breadcrumb's last item the product name. *Verified by query, 2026-10-03.*
- **Card "from" price.** `collection-load-more` prints "As low as" + product `from_pricing` whenever `from_pricing` is set. `collection-filters` only uses `from_pricing` when `product.price == 0`. Collection `load_more` switches the page to the load-more snippet. `from_pricing` is static: re-check it when the lab changes prices. *Verified by query, 2026-10-03.* (`52_SNIPPET_INVENTORY.md` corrected to match, 2026-10-06.)
- **Delivery / pickup box** (`product/shipping-available`, "This item can be picked up in our store"): every product page variant (product-details, details-filter, prints, dual-mode) skips it when collection `hide_delivery_options` is ticked. Per collection only; a site with no pickup needs it on every collection. *Verified by query, 2026-10-03.*
- **Shop tile image for design products.** `collection/collection-filters` draws a live preview when the design has preview pages, else `design.preview_images`, else `design.image` (the design's preview_img). The product image is not used. Set the design preview image, and give each product its own design when two products share one, or both tiles show the same picture. *Verified by reading source and by query, 2026-10-06.*
- **Two products that share one design share one URL:** the page shows the first product and the second is unreachable from the collection card. *Verified by query, 2026-10-05.*
- **Product `unit_intervals` takes a list:** `100,250,500,1000` gives exactly those quantity choices (custom design tools' quantity chips follow it). Platform-level. *Verified by query, 2026-10-06.*

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
- 2026-08-05: Corrected `page_path` to store the full slash-joined path at levels 2 and 3, not only the final segment, and corrected the constraint that wrongly stated level 1 only. Added Head-level dependencies must repeat the lookup, covering the `html.head` before `page.content` render order, the paste-ready head lookup block, the `!= blank` guard, and the per-page noindex pattern via a boolean `hide_from_index` custom field. Source: claude-chat.
- 2026-08-11: Added Value-bearing checklists to Section 5 — many `admin/checklist/*` keys hold interpolated values, and overwriting one with a boolean takes every page down; includes the known value-bearing key list, the case-sensitivity and `| strip` capture rules, and the `custom-X-page` flag against an empty target snippet. Added three Known Gotchas: the Add to Cart button carries no `type` attribute; reading the product price from JavaScript (`px-product-price`, the `regular_pricing` strikethrough trap, observer placement, and `unit-price="true"` instead of JS division); and a child's CMS backup can hold a stale inherited copy of a parent snippet. Source: claude-chat.
- 2026-08-29: Added §18 generated CSS is appended after `style/custom.css` (nav colour overrides must out-specify `.navbar-light .navbar-nav .nav-link`, with the console diagnostic and the `text-transform: capitalize` companion). Added §19 Account v2 theming — specificity ladder, token list, the malformed `--acv2-accent-soft` platform defect, `!important` on `.btn-dark`/`.btn-primary`, inline styles in the parent dashboard, the typing animation, and the transition-masks-computed-values testing gotcha. Added §20 GA4 tagging defects on the parent — duplicate `view_item`, photo-prints emitting nothing, `purchase` re-firing on refresh, the `item_id` mismatch for design products, and the orphaned `integrations/google/gtag`. Added §17 gotchas: the trailing-newline value-snippet rule with its signature and generator fix, the `default-delivery-option` `public`/`private` value set, and three parent defects (lowercase-only `font-body`, inverted kiosk idle-screen logo test, double-prefixed checklist path). Corrected the custom home page gate to the `admin/checklist/custom-home-page` capture with no `| strip`, flagged pending on whether the Storefront Settings checkbox is also required. Amended the §16 kiosk status. Source: claude-chat.
- 2026-09-09: Added §21 Parent-Safe Changes to Shopper 24 — the two `collection_filters` syntaxes (three fields for `collection/collection-filters`, five for `pdp_layout` via `product/details-filter-dual-mode`, with `snippet_args` consumed by `product/filter-controls` and `asset_images: true` requiring an asset filename); a free-form arguments string is an opt-in needing no code and no deploy, with the three properties that must be proven and the unverified `{% snippet %}` scope question; the admin action is Override Snippet, an override pins the snippet, and a promotion is unfinished until the override is removed; every parent port ships gated and off by default; a child overriding `pages/custom.css` wholesale inherits no parent CSS blocks, and the parent file is Liquid-rendered; a site-level checklist key set on the parent moves for every child. Added §22 SEO defects fixed at parent level. Added to §20 the GTM-only technical standard (`setup-google-tag-manager` set, `website/gtag` blank, both set is ~2x double counting, only `view_item` wired through gtag, Google Ads has no preset), the half-a-chain trap as the first check on any "connected but I see nothing" report, and the cross-reference to the myPixfizz server-side `purchase` pipeline including the double-count risk from leaving `purchase` in the GTM trigger regex and the missing `_ga` cookie landing purchases as Direct / (not set). Added to §16 the minimum kiosk-mode key set, the silent host-mismatch failure of `helpers/is-kiosk-mode`, and the fact that kiosk design tokens are defined on `.kiosk-touchscreen` inside `kiosk/style`. Added §17 gotchas: hide-don't-replace for a computed price with its drift conditions; always-visible gallery arrows and driving the platform gallery by dispatching a click on its own thumbnail; editor locale needs `editor`-namespace translations exported and imported. Source: claude-chat, slack-message, fireflies-call.
- 2026-09-24: CSS delivery split by site: child → `style/custom.css` snippet override, parent → `custom.css` page; the parent wins at equal specificity (and new § 18.1 for the parent's own feature CSS). Pages Custom Type accepts duplicate `page_path` values silently. Static Product Importer: header rule, 64-character caps, case-insensitive codes, description not stored, the hang before collection assignment. Live `selectors:` re-renders are DOM patches. Added § 23 Static Product Collections. Source: claude-chat.
- 2026-09-29: §9 favicon is the asset named exactly favicon.png. §13 custom-body-font takes no trailing semicolon. §14 check /site/<path> is a 404 before choosing a page_path. §17 native modal dialog top layer beats any z-index; move overlays inside the open dialog. §17 prices outside px-product-price must include default-variant surcharges. §18.1 the unscoped h1-h4 rule is in the pxfb block and strips heading font and weight; child :root workaround and parent fix. §20 CORRECTED where the GTM and GA4 fields live (manage/integrations); the Setup and Manage path is unverified. §20 view-item gtag include throws on GTM-only sites, no consent gating, ads-id unused, capture snippets named; new events go to the dataLayer only. §23 CLOSED the open item: query-string shape for deep links that preselect filters and variants. §21.3 Override Snippet copies the parent body, is per site, live on save, may need a hard refresh. Source: claude-chat, fireflies-call.
- 2026-10-06: §1 Shopper 24 has no layouts/main and no px-tag layout (Shopify integration host does); the sign-in modal on every page (`#modalLoginCheckout`, `?login_user=t`, never send mid-flow shoppers to /site/login). §2 `#cart-link-icon` duplicated on the search icon (open defect, parent fix, audit check). §3 check megamenu open state by `.show`, not opacity. §5 token table for radio-type keys and the token-not-label rule; CORRECTED rows `bottom-promotion-bar` (does nothing), `payment-gateway` (`authorizedotnet`), `prints-autoselect` (lowercase true/false); value sets on several rows; kiosk tip keys. §7 CORRECTED google-review-link (widget reads the checklist key); current-promotions not reusable; store address keys. §8 footer newsletter is a placeholder. §9 social image is `og-preview-image.jpg`. §11 email current state and pointer to 32. §12 Blog (asset-name fields, unpublished posts render, homepage section unfiltered, unescaped JSON-LD) and Promotions fly-out. §13 custom homepage delivery rules. §14 never a Pages instance under services/; Liquid inside page_content (renders; variables to called snippets did not arrive; config-stub pattern). §15 manage/* known defects and audit lesson; file inputs are styled by the shopper-admin layout (never override), sample CSV downloads built with a Blob. §16 kiosk checkout styling lessons. §17 search results linking into test collections. New §18.2 dark child themes (checkout high risk). §20 GA4 bridge status (not deployed); new §20.1 Klaviyo integration and email consent. §21.1 wall-art-only scope, `show_prices: true` standard, tile order = Design Products order, filters paste hazard. §21.2 snippet scope partly answered. §22 footer sitemap link. §4 dark child sites: override the token snippets, hard-coded `.form-control` color, inline swatch name color. §21.3 the first `_method` form on a snippet edit page is DELETE. §23 static product page form and quantity. New §24 product URLs (`url_path`), titles, breadcrumbs, card "from" price, delivery box, shop tile image, shared designs, `unit_intervals`. Source: claude-chat.
- 2026-10-09: § 14 settled which wins at `/site/photo-prints` (the real page) and added the `product/photo-prints-landing` gate for replacing it on a child, with the parent-stub paste incident. § 17: the live navbar is taller than a harness shows; an admin.pixfizz.com login does not open `user.is_admin` pages. Source: claude-chat.
