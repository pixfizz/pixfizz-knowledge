# 21 — Shopper Checkout Policy

**Authority Scope:** Checkout engine logic only.

_Last updated: 2026-04-23_

---

# 05 — Shopper Checkout Policy Engine

Shopper checkout resolves UI/flow by computing derived flags from cart orderlines and admin checklist toggles.

## Kiosk mode
- Alternate kiosk domain + helper snippet detection.
- Pay-in-store can be shown only in kiosk mode.
- Auto-logout after successful checkout.

## Shipping unavailable
If any orderline has:
- `orderline.product.custom.shipping == 'false'` OR
- `orderline.product.custom.shipping_unavailable`

## Pickup unavailable
If any orderline has:
- `orderline.product.custom.pickup_unavailable`

## Film processing
If not disabled by `admin/checklist/hide-film-order-checkout`, and any orderline has:
- `orderline.product.custom.film_processing`

## Digital-only detection (if enabled)
If `admin/checklist/digital-only-delivery == 'TRUE'` and cart non-empty:
- All orderlines must have `chosen_variants['version']`
- All must equal `digital-only`
Then checkout:
- Ignores shipping
- Applies hidden public/system address
- Shows digital delivery state

## Tax model: US-style checkout tax vs European VAT expectations

Pixfizz uses a **US-style tax-at-checkout model**: tax is calculated and added at checkout, not included in displayed prices.

For European stores, this creates a UX mismatch — customers expect VAT-inclusive pricing (the price shown is the price paid). This is a known platform-level structural difference.

Current workarounds:
- **VAT exemption** (e.g. cross-border orders to Switzerland): apply an automatic discount as a negative fee to remove the VAT that was added at checkout. The storefront price remains unchanged.
- **Postal code-based tax rules** are managed via a CSV file that maps postal code ranges or prefixes to tax rates. Complex postal code formats (multi-region countries like France) require range or prefix matching logic in the CSV.

Flag this to clients during onboarding if they are European and expect tax-inclusive display prices.

## Guest checkout: configurable fields

Guest checkout supports the following configuration:
- **Billing address**: can be set as mandatory
- **Customer ID field**: optional — can be added as an optional field (relevant for in-store or lab-account workflows)
- **Registration toggle**: controls whether the guest is offered account registration at checkout

These are checkout configuration options, not hardcoded behaviour.

## Changelog
- 2026-02-26: Initial checkout policy content.
- 2026-04-23: Added tax model note (US-style vs European VAT, postal code CSV, automatic discount workaround). Added guest checkout configurable fields note.
