# 10 — Core Mental Model

**Authority Scope:** Platform-level conceptual architecture only.

_Last updated: 2026-03-30_

---

# 01 — Core Mental Model (Locked Canon)

## Four-layer architecture
Pixfizz is organized around four capability layers:
1. **Personalization Layer** — product configuration, design tools, XML templates. Core of every deployment.
2. **Commerce Layer** — storefront, checkout, catalog, payments. Only active in Full Pixfizz.
3. **Workflow Layer** — OrderHub: order management, artwork generation, asset delivery, production jobs.
4. **Website Layer** — Shopper, custom frontend, theming, domain/SSL. Only in Full Pixfizz.

See **15_PLATFORM_ARCHITECTURE.md** for deployment models and full layer details.

## Platform vs Shopper vs Custom
- Pixfizz CMS provides objects, identity, pricing engine, admin/custom fields.
- Shopper is an opinionated storefront implementation (cart/checkout UI rules).
- Custom storefronts may differ from Shopper even while using the same CMS.

### How to identify if a site is on Shopper
Click the ellipsis icon at the bottom of the Pixfizz admin sidebar. The modal shows "Source Websites" — `shopper24.pixfizz.com` confirms the site is a Shopper child site. Child sites do not expose the full CMS pages list, so wildcard page detection is not a reliable method.

## Cart vs checkout
- Cart does not change based on payment method.
- Payment options vary on checkout only.

## Digital-only
- Digital-only is a static product with variant `version` == `digital-only`.
- Checkout ignores shipping and applies a public/system address internally (hidden); customer sees digital delivery.
- No special cart behavior for digital-only.

## Photo prints
- Photo prints quantities are managed in the Photo Prints UI and priced via `cut_print_quantity`.
- Cart quantity is not editable for photo prints.
