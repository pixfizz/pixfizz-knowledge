# 30 — Pricing Engine

**Authority Scope:** Ruby pricing formulas and price variables only.

_Last updated: 2026-07-03_

---

# 06 — Pricing Formulas (Ruby) & Price Variables (Locked Guidance)

## What pricing formulas are
- Ruby expressions evaluated in context.
- Must return a numeric price.
- Used for product pricing and option/variant pricing.

## Where to paste a pricing formula
Pricing formulas are set on the **Product Attribute** in the admin:
- Go to **Products** → open the product → open the **Product Attribute** (the specific size/variant, e.g. "4x6 Glossy")
- Paste the formula into the **Pricing Formula** field on that attribute
- If the same formula applies to multiple sizes, paste it on each Product Attribute individually (prices per size typically differ, but the formula structure may be shared)

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
`19.99 + ((pages - uncounted_pages) - 16) / 2.0 * 0.50`
or
`19.99 + extra_pages / 2.0 * 0.50`

**Critical:** The `pages` variable counts ALL pages in the project, including pages
in XML sets with `count="false"` (covers, preview pages, etc.). For book pricing
that charges per interior page, always use `(pages - uncounted_pages)` to exclude
uncounted pages from the calculation. Using raw `pages` will overcharge customers.

Use `2.0` (not `2`) to force float division and avoid Ruby integer rounding.

RATIONALE: `pages` includes uncounted XML pages (covers, previews). Without subtracting `uncounted_pages`, book pricing formulas overcharge. Confirmed in client support case, May 2026.
SOURCE: "Pixfizz customer support agent" chat, May 3 2026
SOURCE TYPE: claude-chat

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

---

## `pages` Includes Uncounted Pages — Use `(pages - uncounted_pages)` for Books

The `pages` variable in pricing formulas counts **every** page in the project,
including pages in XML template sets marked with `count="false"` (typically covers,
preview thumbnails, and other non-interior pages).

For any book or multi-page product where pricing is based on interior page count,
always use `(pages - uncounted_pages)` instead of raw `pages`.

```ruby
# WRONG — includes covers/preview pages in the count
74.90 + (pages - 42) / 2.0 * 1.90

# CORRECT — excludes uncounted pages
74.90 + ((pages - uncounted_pages) - 42) / 2.0 * 1.90
```

Using raw `pages` causes the formula to calculate more "extra" pages than the
customer actually added, resulting in overcharging.

This applies to all page-based pricing patterns: base + incremental, threshold +
blocks, sheet-based, and volume discount + per-page.

RATIONALE: High-impact pricing gotcha. Raw `pages` silently includes uncounted XML pages, causing overcharging on book products. Not previously documented.
SOURCE: "Pixfizz customer support agent" chat, May 3 2026
SOURCE TYPE: claude-chat

#### Variable choice: `quantity` vs `units`
- `quantity` — quantity within a single orderline only. Use when pricing is per-orderline.
- `units` — total units across all orderlines in the cart for this product. Use when
  tiered or stepped pricing should accumulate across multiple separate orderlines.

For the yard signs case: using `quantity` charges $25 for the first sign per orderline.
If a customer adds 1 sign, goes back, and adds another as a second orderline, they would
be charged $25 twice. Using `units` would recognise 2 total signs across the cart and
apply the $15 additional rate to the second. Choose based on the intended pricing behaviour.

---

## Worked Example — Unified Base + Variant Adjustment + Quantity Breaks

Source: client implementation, 2026-04-10. First documented Pixfizz pattern that combines variant
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

Confirmed on client cut print pricing, April 2026. The `* -1` pattern is logically equivalent and the editor accepts it.

RATIONALE: Platform editor limitation with a non-obvious workaround. Affects any negative variant adjustment using cut print formulas.
SOURCE: "Claude pricing formula generation" chat, April 23

### Canonical workaround (2026-04-08)
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

## Automatic Discounts (Liquid-Based Cart Discounts)

**Feature type:** Platform-level. Configured in Main Admin.

Automatic Discounts apply cart-level discounts without requiring a promo code. The discount is calculated using a **Liquid formula** that has access to the full cart and user context, and the result appears automatically at checkout.

Reference: [https://help.pixfizz.com/triage/automatic-discounts](https://help.pixfizz.com/triage/automatic-discounts)

### How it works

- The formula is a Liquid template that must return a **numeric discount amount** (not a percentage — the output is the actual value to subtract).
- The formula has access to `cart`, `user`, `orderlines_total`, and other standard Liquid objects.
- The discount appears automatically in the cart/checkout — no customer action required.
- Multiple automatic discounts can be active simultaneously.

### Available context variables

- `cart.orderlines_total` — subtotal before discounts
- `cart.promocode_code` — the applied promo code (blank if none)
- `cart.orderlines` — the orderlines collection
- `user.category` — user category label (e.g. "VIP", "Wholesale")
- `user.orders_count` — number of completed orders (if available on the user object — confirm with Matjaz)
- Standard Liquid filters: `date`, math operators, etc.

### Canonical patterns

**Tiered cart discount (spend more, save more):**

```liquid
{%- if cart.orderlines_total >= 250 %}
    orderlines_total * 0.20
{%- elsif cart.orderlines_total >= 150 %}
    orderlines_total * 0.15
{%- elsif cart.orderlines_total >= 75 %}
    orderlines_total * 0.10
{%- endif %}
```

**User category conditional discount with promo code guard:**

```liquid
{% if user.category == 'VIP' and cart.promocode_code == blank %}
    orderlines_total * 0.10
{% endif %}
```

The promo code guard (`cart.promocode_code == blank`) prevents stacking a category discount with a manual promo code. Whether to include this guard depends on the business intent.

> **Property-name trap:** the property is `cart.promocode_code`, not `cart.promocode`. `cart.promocode` (without `_code`) is not a valid Cart property and resolves to nil, which is falsy. A guard written as `{%- unless cart.promocode -%}` therefore never blocks, and the discount fires even when a promo code is applied. Always use `cart.promocode_code`.

**Seasonal / time-based discount:**

```liquid
{%- assign current_month = 'now' | date: '%m' | plus: 0 -%}
{%- if current_month == 1 or current_month == 2 -%}
    orderlines_total * 0.15
{%- endif -%}
```

Runs automatically during slow months, turns itself off when the month changes.

### Key rules

- The formula must return a numeric value. If the formula returns nothing (no branch matches), no discount is applied.
- The discount is an **amount**, not a percentage — the formula does the percentage math itself.
- Automatic discounts are separate from promo codes and extra fees. They are a distinct discount mechanism.
- Admin location: confirm exact admin path with Matjaz (likely under Discounts or Pricing in Main Admin).

---

## Extra Fees (Liquid-Based Cart Fees)

**Feature type:** Platform-level. Configured in Main Admin under **Shipping → Extra Fees**.

Extra Fees are the fee-side twin of Automatic Discounts. Each fee can be driven by a
**Liquid formula** with full cart context, and the result is **added** to the order
(Automatic Discounts subtract; Extra Fees add). This is the supported way to add a
conditional surcharge — the platform does not support negative discounts or negative
variant adjustments to achieve the same effect (see "Negative Extras / Discount
Variants — Workaround" above).

Confirmed in production use for minimum-order-value fees and extra shipping / oversize
surcharges.

### How it works

- Each Extra Fee has a **Code** and **Name** (for example `rush` / "Rush Fee",
  `oversize` / "Oversize shipping charge") plus a Liquid formula.
- The formula returns a **numeric amount** — the actual fee value to add, in the
  site's currency (not a percentage).
- Same cart / user context as Automatic Discounts: `cart.orderlines`,
  `cart.orderlines_total`, `cart.promocode_code`, `user.*`, and standard Liquid filters.
- Multiple Extra Fees can be active at once; each is evaluated independently.
- If the formula returns nothing (no branch matches, or empty output), no fee is added
  — mirrors the Automatic Discounts "return nothing = nothing applied" behaviour.

### Canonical use cases

- **Minimum order fee:** add a flat handling fee when the cart subtotal is below a threshold.
- **Extra shipping / oversize surcharge:** add a fixed amount when the cart contains an oversized product.

### Pattern — Per-Duplicate-Orderline Surcharge (count distinct lines)

Use case: charge a flat amount for each **additional** order line of the same product
beyond the first (for example, multiple separate cut-print lines of the same size).
Documented from a photo lab client implementation.

The reusable technique is a **seen-string + `contains`** idiom to count distinct
products across `cart.orderlines`. The first line of a given product is the original
(no charge); every later line of the same product adds the per-duplicate amount.

```liquid
{%- assign per_duplicate = 1 -%}
{%- assign fee = 0 -%}
{%- assign seen = '' -%}
{%- for orderline in cart.orderlines -%}
	{%- if orderline.is_cut_print -%}
		{%- assign token = '|' | append: orderline.product.id | append: '|' -%}
		{%- if seen contains token -%}
			{%- assign fee = fee | plus: per_duplicate -%}
		{%- else -%}
			{%- assign seen = seen | append: token -%}
		{%- endif -%}
	{%- endif -%}
{%- endfor -%}
{%- if fee > 0 -%}{{ fee }}{%- endif -%}
```

- Five cut-print lines of one product returns 4. Add three lines of a second product
  and it returns 6 (each product is counted independently).
- **Grouping key:** `orderline.product.id` groups by product. Correct only when each
  print size is its own Product. If size is a variant on a single product, every size
  shares one product ID — include the chosen size variant in the `token` to keep sizes
  distinct.
- Drop the `orderline.is_cut_print` guard to apply the same duplicate-line logic to any
  product type, not just cut prints.

### Key rules

- The formula returns an **amount to add**, in the site currency, as a raw number (no
  `currency` filter on the output — the engine formats it).
- **Verify on first test order (pending confirmation with Matjaz):** that `cart.orderlines`
  is the correct loop handle inside an Extra Fee formula, and that a bare `{{ number }}`
  output is read as the fee amount. Both are inferred from Automatic Discounts behaviour;
  confirm before relying on the orderline-iteration pattern in production.
- Admin location: **Shipping → Extra Fees** (confirmed via admin screenshot).

---

## Roadmap — Price Variable Bulk Export / Import

**Status: planned, not yet shipped (2026-03-24 / 2026-04-10).**

A bulk export/import flow for Price Variables is on the roadmap. This will allow
editing price variables in a spreadsheet and re-importing them, rather than editing
them one at a time in the admin.

Do not design around its absence — if a site needs bulk price variable edits today,
the current workflow is still manual. But flag this as a coming capability when
scoping any onboarding that involves hundreds of price variables.

---

## Changelog
- 2026-05-19: Added Automatic Discounts section — Liquid-based cart discounts with tiered, user category, and seasonal patterns. Source: Claude chat (webinar prep).
- 2026-07-03: Added Extra Fees (Liquid-Based Cart Fees) section — fee-side twin of Automatic Discounts (adds instead of subtracts), configured under Shipping → Extra Fees. Includes per-duplicate-orderline surcharge pattern (seen-string + contains idiom); orderline-iteration specifics pending live confirmation. Source: Claude chat.
- 2026-07-20: Added property-name trap — `cart.promocode` (without `_code`) is nil and silently defeats promo-code guards; always use `cart.promocode_code`. Source: claude-chat.
