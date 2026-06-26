# 21 — Shopper Checkout Policy

**Authority Scope:** Checkout engine logic only.

_Last updated: 2026-06-26_

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

### Tax CSV format

The tax CSV has a single header row, then one row per rule. Header:

	country,region/state,zipcode,city,tax%

Rules are matched against the shopper's shipping address. Specificity stacks:
- Country + state: applies anywhere in that state.
- Country + state + zip: applies when state and zip both match.
- Country + state + zip + city: applies when state, zip, and city all match.

This only applies when product prices are NOT tax-inclusive. If prices already
include tax, no rules are needed and the price shown is the price paid. When
prices exclude tax, the matched rate is added on top of the orderlines at cart and
checkout. A customer-facing help article exists ("Setting Up Taxes").

Flag this to clients during onboarding if they are European and expect tax-inclusive display prices.

## Guest checkout: configurable fields

Guest checkout supports the following configuration:
- **Billing address**: can be set as mandatory
- **Customer ID field**: optional — can be added as an optional field (relevant for in-store or lab-account workflows)
- **Registration toggle**: controls whether the guest is offered account registration at checkout

These are checkout configuration options, not hardcoded behaviour.

## Payment method selection and labels

**Which method is selected by default.** The default selected payment method is resolved in two layers:
- A URL parameter takes priority: `request.params['payment-method']`. If present and valid, it persists the customer's selection across page reloads.
- If no valid method is set via URL, the system falls back to the **first entry** in the `checkout/available-payment-methods` snippet.

So the effective default is determined by the **ordering** in `checkout/available-payment-methods`. To change the default, reorder that snippet - do not modify the `site/checkout` file.

The payment-method radio selector only renders when **more than one** method is enabled. With a single enabled method it is auto-selected and the selector is hidden.

**Renaming a payment method label.** Payment method display labels (e.g. "Cash on Delivery") are set via the admin **Translations** system, not a dedicated config field. To rename a method - for example "Cash on Delivery" to "In House Billing" - edit the translation string for that label. See `18_ADMIN_NAVIGATION.md` section Built-in Translation Support.

## Changelog
- 2026-02-26: Initial checkout policy content.
- 2026-04-23: Added tax model note (US-style vs European VAT, postal code CSV, automatic discount workaround). Added guest checkout configurable fields note.
- 2026-06-01: Added tax CSV format and matching specificity. Source: claude-chat/help-article.
- 2026-06-26: Added payment method selection (URL param priority, then first entry in checkout/available-payment-methods; radio only renders for 2+ methods) and label renaming via Translations. Source: claude-chat/fireflies-call.
