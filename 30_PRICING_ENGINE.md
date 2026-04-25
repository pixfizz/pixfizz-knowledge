# 30 — Pricing Engine

**Authority Scope:** Ruby pricing formulas and price variables only.

_Last updated: 2026-04-10_

---

# 06 — Pricing Formulas (Ruby) & Price Variables (Locked Guidance)

## What pricing formulas are
- Ruby expressions evaluated in context.
- Must return a numeric price.
- Used for product pricing and option/variant pricing.

## Price Variables
- Admin-defined numeric constants available by name in formulas.
- Used to centralize pricing inputs across products/options.
- `whitelabel` is a **Price Variable** (not a system variable).

## Keep formulas basic
Use established patterns; avoid “optimized” Ruby.

## Canonical patterns (examples)
### Linear
`0.85 * cut_print_quantity`

### Tiered unit price × quantity
`{1..10=>0.59, 11..49=>0.49, 50..249=>0.44, 250..499=>0.39, 500..2000=>0.35}.find { |range,unit_price| range.include?(cut_print_quantity) }.last * cut_print_quantity`

### Base + incremental pages
`19.99 + (pages-16)/2 * 0.50`
or
`19.99 + extra_pages/2 * 0.50`

### Volume discount lookup (unit price)
`{1..1=>8.00, 2..5=>6.50, 6..10=>6.00, 11..20=>5.75, 21..30=>5.60, 31..50=>4.80, 51..1000=>4.15}.find { |range,unit_price| range.include?(units)}.last`

### Volume discount + per-page pricing (per unit)
`{1..1=>(19.99 + (pages-16)/2 * 0.50), 2..5=>(18.99 + (pages-16)/2 * 0.45), 6..10=>(17.99 + (pages-16)/2 * 0.40)}.find { |range,unit_price| range.include?(units)}.last`

### Threshold + blocks of pages
`49 + ([0, pages - 50].max / 4)`
or
`49 + ([0, pages - 50].max / 4)*2.5`

### Sheet-based production pricing
`10 + (((pages - 12.0) / 12.0).ceil) * 8`

## Photo Prints — Hard Rule

- `cut_print_quantity` is only valid for Photo Prints.
- Must always be used for cut prints.
- Must never be used for non-cut-print products.

### Enforcement Logic

IF product is Photo Print → use `cut_print_quantity`
ELSE → use `quantity` or `units`

No exceptions.

### First unit + each additional (total price formula)
`(25 + ([0, quantity - 1].max * 15.0)) / quantity`

Use when the first unit has a higher base price and each additional unit costs less.
The `/ quantity` is required because Pixfizz multiplies the returned value by quantity —
the formula must return a **per-unit price**, not a total.
Use `15.0` (not `15`) to force float division and avoid Ruby integer rounding errors.

#### Variable choice: `quantity` vs `units`
- `quantity` — quantity within a single orderline only. Use when pricing is per-orderline.
- `units` — total units across all orderlines in the cart for this product. Use when
  tiered or stepped pricing should accumulate across multiple separate orderlines.

For the yard signs case: using `quantity` charges $25 for the first sign per orderline.
If a customer adds 1 sign, goes back, and adds another as a second orderline, they would
be charged $25 twice. Using `units` would recognise 2 total signs across the cart and
apply the $15 additional rate to the second. Choose based on the intended pricing behaviour.

---

## Worked Example — Unified Base + Variant Adjustment + Quantity Breaks (Hite Photo)

Source: Hite Photo, 2026-04-10. First documented Pixfizz pattern that combines variant
price adjustments **and** quantity breaks under a single base price.

### Model
- One base price per product (e.g. a photo restoration service).
- "Color correction" is the **default** variant.
- "No color correction" applies a **negative variant adjustment** (≈30% off base).
- Quantity break tier discounts apply **only on the non-corrected path** (to protect
  labour margin on the corrected path).
- Customer-facing tier pricing is shown on the product page as a simple HTML table
  with a tooltip — no dynamic formula display.

### Why this pattern
It keeps a single base price in the admin (simple to manage, simple to report on)
while still letting a single product serve two distinct price ladders depending on
which labour-level variant the customer picks.

---

## Negative Extras / Discount Variants — Workaround

The platform does **not** currently support negative extra fees or negative variant
adjustments directly. Attempts to set a negative extra value will not price correctly.

### Variant Formula Editor — Leading Minus Sign Error

The variant formula editor does **not** accept a leading minus sign (`-`). Entering a formula that starts with `-` causes an editor error and the formula cannot be saved.

**Confirmed workaround:** multiply by `-1` at the end of the formula instead.

```ruby
# This errors:
-(base_price * tier_1)

# This works:
(base_price * tier_1) * -1
```

Confirmed on Hite Photo cut print pricing, April 2026. The `* -1` pattern is logically equivalent and the editor accepts it.

RATIONALE: Platform editor limitation with a non-obvious workaround. Affects any negative variant adjustment using cut print formulas.
SOURCE: "Claude pricing formula generation" chat, April 23

### Canonical workaround (Hite Photo pattern, 2026-04-08)
1. **Lower the base price** of the product to the discounted value.
2. **Add a positive surcharge** to the "standard" variant so the net price of the
   standard path matches the original intended base.
3. Customers who pick the "discounted" path (e.g. delayed service, no correction)
   land at the genuinely lower base price with no extra applied.

This inverts the mental model — the discounted option becomes the default — but it
is the only reliable way to get a discount-variant effect today.

### Use cases seen in production
- Rush fee **+20%** — rush is the surcharge variant, standard turnaround is the base.
- Delay discount **−10%** — delayed turnaround is the base, standard is the surcharge.

When documenting these for a client, always label the variants in customer-facing
terms ("Standard / Rush" or "Standard / Delayed — save 10%") — do not expose the
inverted base-price mechanics.

---

## Photo Enhancement Variants — Charge Once Per Photo, Not Per Quantity

Recurring support issue: photo enhancement add-ons (e.g. "enhance this image") must
charge once per image, not multiplied by the print quantity the customer orders.
The standard Pixfizz behaviour of multiplying the option price by `quantity` is
wrong for this case.

The formula must explicitly flatten the quantity multiplier — return a per-unit
value that divides out the quantity the engine will re-multiply by. Confirm the
exact snippet against the live site before reusing; the pattern is known but the
canonical formula has not yet been locked in to this reference.

---

## Test Orders with 100% Voucher — Billing Cap Removed

**Platform rule change, 2026-03-24.**

Previously, test orders paid with a 100% voucher were still billed at 50% of the
order value internally (a safety cap on the billing file). That cap has been
removed. Orders paid with a 100% voucher now report as fully zeroed in the
billing file.

Impact: any site using voucher codes for internal QA or for comped client orders
will now see those orders at **$0** in billing reports rather than at 50% of the
catalogue price. Update any reconciliation logic that assumed the old cap.

---

## Roadmap — Price Variable Bulk Export / Import

**Status: planned, not yet shipped (2026-03-24 / 2026-04-10).**

A bulk export/import flow for Price Variables is on the roadmap. This will allow
editing price variables in a spreadsheet and re-importing them, rather than editing
them one at a time in the admin.

Do not design around its absence — if a site needs bulk price variable edits today,
the current workflow is still manual. But flag this as a coming capability when
scoping any onboarding that involves hundreds of price variables.
