# 82 — Mobile UX Reference

**Authority Scope:** Storefront mobile experience on Pixfizz Shopper — layout,
conversion patterns, the platform facts that constrain a mobile design, and an audit
checklist. **The design tool's own mobile mode is documented in 17_DESIGN_TOOL.md**
§ Editor CSS Customization § Mobile editor CSS; this file points at it rather than
duplicating it.

_Created 2026-08-29. Every claim below is labelled: **[verified]** read from the
template, the editor bundle or a live page; **[framework]** conversion guidance from
the Shopper UX framework, applied judgement rather than platform behaviour;
**[category]** observed across competitor products in an August 2026 audit, not
Pixfizz behaviour._

---

## 1. Scope, and why this file exists

Most photo and print storefront traffic is mobile, and mobile is where the
personalisation flow is hardest to hold together. Until 2026-08-29 the retrieval map
routed mobile questions to this file and to `83_MOBILE_UX_AUDIT.md`; neither had ever
been written, and the coverage that existed was four scattered sentences across three
files. This file consolidates what is actually known.

Three different things get called "mobile" on this platform and they are not the same
surface:

| Surface | What it is | Where it is documented |
|---|---|---|
| **Storefront on a phone** | Shopper template at a narrow viewport, Bootstrap 4.6 | This file |
| **Design tool mobile mode** | A **separate template** in the editor bundle, not a responsive layout | 17_DESIGN_TOOL.md |
| **Kiosk touchscreen mode** | A checklist-gated in-store mode, `.kiosk-touchscreen` on `<body>` | 50_SHOPPER_TEMPLATE_REFERENCE.md § 16 |

Designing one of these does not carry to the others. Kiosk in particular is a
touch surface but not a mobile one — large tiles, no scrolling, a fixed device.

---

## 2. Platform facts that constrain a mobile design

**[verified]** From the Shopper template and the editor bundle.

- **Navigation collapses to a hamburger toggler.** `navigation/style1` and
  `navigation/style3` both render logo-left plus toggler at mobile width, with nav
  links collapsing into an accordion. Which style renders is an admin setting a tar
  cannot read or set — see 50_SHOPPER_TEMPLATE_REFERENCE.md § 17.
- **`header/logo-height-mobile`** is a separate value snippet from the desktop logo
  height. A logo that reads well in the desktop header is frequently too tall on a
  phone; this is the control, and it is easy to miss because the desktop one is the
  obvious snippet.
- **`variant_columns_mobile`** sets how many variant columns render on mobile,
  independently of the desktop count. Getting this wrong is the most common cause of
  a product page whose options are either unusably narrow or absurdly tall.
- **The design tool's mobile mode is not width-driven.** The editor picks its
  template from device and touch detection, not a breakpoint, so narrowing a desktop
  browser to 390 px still renders desktop mode. See 17_DESIGN_TOOL.md for how to
  reach mobile mode for testing.
- **Generated colour CSS is appended after `style/custom.css`** and carries
  `!important` on three-class selectors, so a mobile nav restyle must out-specify
  `.navbar-light .navbar-nav .nav-link`. This has produced a live nav CTA at a 2.28:1
  contrast ratio. See 50_SHOPPER_TEMPLATE_REFERENCE.md § 18.

---

## 3. Conversion principles at a mobile viewport

**[framework]** From the Shopper UX conversion framework. These are design judgement,
not platform behaviour, and all of them are implementable in Bootstrap 4.6 plus Liquid
with no custom JavaScript.

- **Evaluate every layout at mobile width first**, not as an afterthought to a desktop
  comp.
- **The primary CTA must be reachable with a thumb** — bottom-anchored, or high enough
  in the viewport that it is not a stretch. Never bury it below the fold.
- **Touch targets: 44 px minimum height.** This is the floor, not the target.
- **Product card CTAs must be visible without hover.** A card whose action only
  appears on hover has no action on a phone. The same applies to any warning,
  tooltip or helper text: **[category]** hover-gated messaging is one of the most
  consistent mobile failures in this product category.
- **One dominant action per page.** If two actions feel equal in weight on a phone,
  one of them is wrong.
- **Price on the card, before the product page.** For variable-price products show a
  "from" anchor.
- **Trust signals near the CTA, not only at the page bottom** — print quality,
  delivery or collection timeframe, reviews. Personalised products carry purchase
  anxiety and a phone screen shows one thing at a time, so proximity matters more here
  than on desktop.
- **Lead with lifestyle imagery, then product detail.** Imagery must fill the width
  and load fast.
- **The personalisation preview must be legible at small size.** A preview that only
  reads at desktop scale is not a preview.

---

## 4. Mobile failures seen on real builds

**[verified]** Each of these was found on a live Pixfizz site.

### A baked hero image is not a mobile hero

A homepage hero delivered as a single flat JPEG with an HTML image map, click targets
positioned by computed coordinates, has no text in the DOM. Nothing is responsive,
selectable, indexable or translatable, and every copy change is an image edit — per
language. Replacing baked hero images with real responsive sections is usually the
highest-value mobile fix available on an inherited site, and it needs no rebranding:
the imagery and palette stay, only the delivery changes.

### Single-value variant types push the CTA below the fold

A variant type with exactly one value auto-selects, inherits the theme's *selected*
button styling, and renders as a large fixed pill. On a product with four fixed
specifications that is four stacked blocks of roughly 190 px each — measured on a
1512 px desktop viewport, where it already pushed quantity and Add to order below the
fold. On a phone it is far worse. The CSS fix is in
22_OPTION_VARIANT_RENDERING.md § A Single-Value Variant Type Renders as a Selected
Button, and it takes each group from about 190 px to 54 px.

### Duplicated language assets

A bilingual site carrying separate `*ar.jpg` mobile hero images per language is
carrying the maintenance cost twice and the page weight twice. Text in the DOM plus
the translations mechanism removes both. See 50_LIQUID_REFERENCE.md § Translation Keys.

### The mobile editor page navigation

The design tool's mobile mode has no page thumbnail strip by default — navigation is
tap a page in the project list, swipe between pages, tap Project to go back. A
verified CSS recipe for a persistent bottom strip exists, along with the cached
measurement trap that makes a naive version render at full-screen size until the
device is rotated. All of it is in 17_DESIGN_TOOL.md.

---

## 5. Patterns from the wider category

**[category]** Observed across consumer photo-book and photo-gift editors in an
August 2026 competitor audit. These are not Pixfizz behaviour; they are what the
market has settled on, and where it has not.

**Worth adopting**

- **Named zoom levels rather than free pinch-and-pan** — all pages, spread, page. The
  clearest mobile answer to navigating a multi-page product.
- **Selection replaces the toolbar.** A persistent bottom toolbar for global actions;
  selecting an object swaps it for that object's controls, rather than nesting menus.
- **Autosave with no Save button.** The strongest apps in the category have removed
  the concept entirely.
- **A fixed corner badge for element-level warnings.** The only element-level warning
  pattern in the category that needs no hover and no adaptation for touch.
- **Expose the automation fork explicitly**, in the first person, with its parameters
  visible at the moment of choosing — "place manually" versus "fill automatically",
  with the size and density settings shown alongside rather than hidden behind the
  choice.

**Worth avoiding**

- **No aggregate review step.** Some major mobile apps ship with none at all; their
  own help documentation instructs the customer to "scroll through and check". On a
  phone that is not a review step, it is a hope.
- **Merging unrelated controls into one toolbar slot** to save horizontal space.
- **Forking the desktop and mobile interaction model for one feature**, so the same
  task is learned twice.
- **Colour as the only signal**, and dimming as the only indication of state.
- **Disabling checkout with no adjacent reason.** At least one major product has
  generated its own help article explaining why customers cannot check out.

---

## 6. Mobile audit checklist

Use when reviewing a storefront for mobile. Checks marked **[verified]** are platform
facts you can confirm on the site; the rest are judgement.

**Layout and reach**

- [ ] Primary CTA above the fold, or bottom-anchored and thumb-reachable
- [ ] One dominant action per screen; secondary actions visually subordinate
- [ ] All touch targets at least 44 px high
- [ ] No action, warning or helper text that requires hover to appear
- [ ] Product card CTAs visible without hover

**Platform settings** **[verified]**

- [ ] `header/logo-height-mobile` set, and the header checked on a real phone
- [ ] `variant_columns_mobile` set appropriately for the product's option count
- [ ] Correct navigation style confirmed by view-source on the live site, not assumed
- [ ] Nav and CTA colours out-specify `.navbar-light .navbar-nav .nav-link`; contrast
      ratio checked against the computed style, not the snippet
- [ ] Any single-value variant types rendered as spec rows, not selected buttons

**Content**

- [ ] Hero is real responsive markup, not a baked image or an image map
- [ ] Price visible on grid cards, with a "from" anchor for variable-price products
- [ ] Delivery or collection timeframe visible on the product page
- [ ] Trust signals sit near the CTA, not only in the footer
- [ ] Imagery fills the width and is sized for the rendered card, not the source file

**Personalisation**

- [ ] Preview legible at phone size
- [ ] Editor checked on a real device, not a narrowed desktop window — mobile mode is
      device-detected, not width-detected **[verified]**
- [ ] Page navigation usable on a multi-page product
- [ ] Customer can tell what will be printed before they reach the cart

**Checkout**

- [ ] Delivery method preselected where the site has one obvious answer
      (`default-delivery-option`, see 50_SHOPPER_TEMPLATE_REFERENCE.md § 17)
- [ ] Form fields use appropriate input types so the right keyboard appears
- [ ] Nothing essential is behind a hover state or a hidden accordion

---

## 7. Known gaps in this file

Honest list, so the next person does not assume coverage that is not here.

- **No measured Pixfizz mobile conversion data.** Everything in § 3 is framework
  guidance and category convention. No Pixfizz-specific mobile funnel numbers have
  been recorded anywhere in this knowledge base.
- **No documented mobile breakpoint set** for the Shopper template beyond Bootstrap
  4.6's defaults.
- **No mobile checkout audit.** The checkout section of § 6 is inferred from general
  practice, not from reading the platform's checkout at a mobile viewport.
- **Kiosk touch guidance lives in 50_SHOPPER_TEMPLATE_REFERENCE.md § 16** and has not
  been reconciled with this file.

---

## Changelog
- 2026-08-29: Created. Consolidates the mobile facts that were scattered across 17, 50 and 80 — nav collapse, `header/logo-height-mobile`, `variant_columns_mobile`, the device-detected editor mode, and the generated-CSS specificity trap — with the Shopper UX framework's mobile-first principles, four mobile failures observed on live builds, category patterns from the August 2026 consumer editor audit, and a mobile audit checklist. Written because `02_RETRIEVAL_MAP.md` had routed to this filename and to `83_MOBILE_UX_AUDIT.md` since before 2026-05-21 without either file ever existing. The audit checklist is folded in here rather than split into an 83 file, since 83 is taken by AI imagery production. Source: claude-chat, Shopper UX framework, competitor audit.
