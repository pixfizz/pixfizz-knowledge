# 30 — Pricing Engine

**Authority Scope:** Ruby pricing formulas and price variables only.

_Last updated: 2026-09-09_

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
- **A formula referencing a price variable will not save until that variable exists.**
  Create the price variables first, then paste the formula; the save is rejected otherwise.
  Verified live, 2026-09-08.
- **Price Variables are not reachable via the API.** There is no API read or write for them.
  Confirmed by the core developer, 2026-09-07. This is a statement of the current limitation,
  not a roadmap commitment — do not scope work that assumes it will change.
- House convention for tier variables: store **multipliers**, not percentages off.

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

### `pages` on a product whose artwork is uploaded, not designed

On a design product where the customer never opens the editor — a custom upload tool writing
`file_upload` template options — the project keeps the template's **default** page count. A
formula of the form `5 + (pages - 20) / 4 * 1` therefore prices the template default, not the
uploaded PDF, however many pages that PDF actually has.

There is no error and no warning. The price is simply wrong, it is wrong *consistently*, and
that consistency is what makes it look deliberate rather than broken.

This is the pricing face of the rule already recorded for fulfillment: page count on an
upload-driven line must come from an explicit source, never from `project.page_count`. If
price depends on page count and the artwork arrives by upload, the tool has to write the
count into a `number` template option and the formula has to read that option instead of
`pages`.

Verified live on an upload-driven design product, 2026-09-09.

#### Variable choice: `quantity` vs `units`
- `quantity` — quantity within a single orderline only. Use when pricing is per-orderline.
- `units` — total units across all orderlines in the cart for this product. Use when
  tiered or stepped pricing should accumulate across multiple separate orderlines.

For the yard signs case: using `quantity` charges $25 for the first sign per orderline.
If a customer adds 1 sign, goes back, and adds another as a second orderline, they would
be charged $25 twice. Using `units` would recognise 2 total signs across the cart and
apply the $15 additional rate to the second. Choose based on the intended pricing behaviour.

`quantity` is scoped to a **single orderline** — two separate lines of the same product do
not combine. `units` aggregates across the cart. Switching a product from one to the other is
a find/replace across **every** formula on that product; miss one and the product prices
inconsistently between branches. It is also a commercial decision rather than a technical
one, so make it once, before the formulas are written. Verified by reading source.

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

## Variant Price Formula Validator — Narrower Than Ruby

The Price field on a variant type has its own validator, and it accepts less than Ruby does.
A rejected formula returns `Error saving: Price isn't valid` and nothing else — the message
names no token, no position and no reason.

Verified by live admin test on a copy-shop client's product, 2026-09-08. Four saves.

### What the validator accepts and rejects

| Formula body | Result |
|---|---|
| `value * <price_variable>` | saves |
| Range-hash + `.find` + block + `.last`, keyed on `quantity` | saves |
| Price Variables as the hash **values** (`{0..24=>tier_1, ...}`) | saves |
| A large finite top range (`1000..99999999`) | saves |
| Block params of any spelling (`\|r,m\|`, `\|range, mult\|`) | saves |
| An arithmetic expression as the `include?` argument — `range.include?(value * quantity)` | saves |
| `range.include?([1,(value * quantity).to_i].max)` | **rejected** |

The rejection was not isolated to a single token. The failing string carried an array literal
`[ ]`, `.max` and `.to_i` together; the passing string carries none of the three. Which one
is the offender is **not verified** — do not claim it without a further bisect.

A large top range saving in the editor proves only that the validator accepts it. Still
prefer a modest cap at runtime.

### Practical rule

In a variant price formula, stick to:

- arithmetic on the in-scope variables,
- the range-hash + `.find` + `.last` tier idiom,
- Price Variables.

No array literals. No method calls beyond `.find` / `.last` on the hash — in particular no
`.max` and no `.to_i`. Guard against a nil `.find` by **starting the first range at 0**, not
by wrapping the lookup in `[1, x].max`.

### Debugging rule

`Error saving: Price isn't valid` names nothing, so **bisect — do not rewrite**. Start from a
formula that is known to save, add one construct at a time, and save after each. Four saves
located the failure above; guessing at it first cost two rounds, including one confident
wrong diagnosis (the range cap) that a single test disproved in seconds.

---

## `value` and `quantity` Are Both In Scope in a Variant Price Formula

A variant price formula can read its own `value` — the number the customer entered on that
variant — **and** `quantity`, the orderline quantity, and can combine them arithmetically.

This does not contradict the rule that a Ruby formula is blind to option values. That rule is
about reading a *different* template option, variant code or product custom field. The
variant's **own** `value` is in scope.

Confirmed saving on a `number` variant type, 2026-09-08:

```ruby
value * unit_rate * {0..24=>tier_1, 25..49=>tier_2, 50..99=>tier_3,
100..249=>tier_4, 250..499=>tier_5, 500..999=>tier_6,
1000..1000000=>tier_7}.find { |range, mult| range.include?(value * quantity) }.last
```

`value * quantity` drives the tier lookup, so the ladder tiers on **a customer-entered number
multiplied by the orderline quantity** — two independent axes. That is the shape a
document-copies or multi-page-upload product needs: pages in the file x number of copies.

The first range starts at 0 so a zero entry cannot return nil from `.find`.

> **A save proves the validator accepts the string. It does not prove the engine prices it.**
> The cart test is separate and mandatory: confirm that the product page display and the
> orderline agree, and that changing quantity in the cart re-tiers. Never report a tiered
> variant formula as done on a successful save. Runtime behaviour for this shape is
> **not verified — pending a cart test**.

### Whole-order stepped ladders are not monotonic

Applying a tier to the whole order rather than marginally means the total can **fall** as the
order grows across a band boundary. Using neutral figures — a first band at $1.00 per unit up
to 100 units, a second band at $0.90 from 101:

- 100 units x $1.00 = **$100.00**
- 101 units x $0.90 = **$90.90**

One more unit costs less. This is inherent to whole-order stepped tiering, not a formula bug,
and it is normally exactly what the client's own published table specifies. **State it to the
client; do not fix it.** The alternative is marginal tiering, which is a different commercial
model and a different formula — and a client decision, not ours.

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
- `cart.custom.<field>` — any cart custom field written by the storefront **does** resolve
  inside an Extra Fee formula. A production `rush` fee has been gated on
  `{% if cart.custom.rush %}` for months. Verified by reading source (admin screenshot of the
  live fee), 2026-09-09. No test order needed.
- `orderlines_total` and `orderlines_discount` are available **unprefixed** inside the
  returned expression — see *The formula is a Liquid wrapper around a bare arithmetic
  expression* below.
- This context list records **what has been observed so far. It is not exhaustive.** Absence
  from the list means "not yet tried", not "not available" — reading it as complete is what
  parked a build for six weeks on a question a live fee had already answered.
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

### The formula is a Liquid wrapper around a bare arithmetic expression

The documented patterns above all show `{{ ... | times: ... }}` filter chains. The live
production fee does not use them. The house form is:

```liquid
{% if <liquid condition> %}
	<bare arithmetic expression>
{% endif %}
```

The output line is evaluated as **arithmetic**, with `orderlines_total` and
`orderlines_discount` available as bare identifiers — no `cart.` prefix, no `{{ }}`, no
Liquid filters. The live `rush` fee is exactly:

```liquid
{% if cart.custom.rush %}
	(orderlines_total - orderlines_discount) * 0.25
{% endif %}
```

This is the same shape as the Automatic Discount examples above (`orderlines_total * 0.20`),
which confirms the two mechanisms share one evaluator.

**For anything percentage-based, interpolate the percentage with Liquid** rather than
referencing the custom field inside the arithmetic. Liquid renders the literal before the
expression is evaluated, so the expression only ever sees numbers:

```liquid
{%- assign px_pct = cart.custom.tip_percent | plus: 0 -%}
{%- if px_pct > 0 -%}
	(orderlines_total - orderlines_discount) * {{ px_pct }} / 100.0
{%- endif -%}
```

**Not verified:** whether a custom field referenced *inside* the arithmetic
(`orderlines_total * cart.custom.tip_percent`) resolves. The interpolation form above
sidesteps the question entirely, and it should be the default recommendation, which makes the
question moot in practice.

Verified by reading source (admin screenshot of the live fee), 2026-09-09.

### Charge percentage fees on the post-discount subtotal

The live fee uses `orderlines_total - orderlines_discount`, not `orderlines_total`.

A percentage fee computed on the pre-discount subtotal charges the customer a percentage of
money they did not pay. Use `(orderlines_total - orderlines_discount)` as the default base for
any percentage-based Extra Fee, and state it explicitly when documenting one for a client.

### Taxable is a per-fee checkbox in the admin

The Extra Fee admin screen (Main Admin -> Shipping -> Extra Fees) carries three fields above
the formula: **Name**, **Code**, and a **Taxable** checkbox.

Whether a fee is taxed is therefore **configuration on the fee** — per fee, per site. It is
not a code change and it is not a platform-wide policy, so the same fee code can be taxable in
one jurisdiction and not in another. Unchecked means the fee is excluded from the tax base.

Verified by reading source (admin screenshot), 2026-09-09.

**Open — whether the checkbox reaches what the shopper is shown.** `pages/checkout` sums every
fee into its VAT tax base with no reference to the flag, and the template is never handed a
`fee.taxable` to test against. See `21_SHOPPER_CHECKOUT_POLICY.md`, *Extra fees and the VAT tax
base*. Until that is settled, do not assume the checkbox changes the tax displayed on a Shopper
checkout. **Not verified — pending one live test.**

### A minimum-order fee survives a 100%-off promo code

A cart-level Extra Fee of the "if the order total is under $X, add $Y" shape is applied
**after** discounts. A 100%-off promo code therefore still leaves the fee sitting on the order,
and a "free" item is never quite free.

The same mechanism seen from the other side: a percentage promo code applies to the **whole
cart**, so adding a second item can take the entire order to zero.

Excluding a product code inside the fee formula has been proposed as the fix. It has **not**
been written and **not** verified — do not offer it to a client as something that exists.

Stated from a client call, not independently verified.

### Key rules

- The formula returns an **amount to add**, in the site currency, as a raw number (no
  `currency` filter on the output — the engine formats it).
- **Verify on first test order (pending confirmation with Matjaz):** that `cart.orderlines`
  is the correct loop handle inside an Extra Fee formula, and that a bare `{{ number }}`
  output is read as the fee amount. Both are inferred from Automatic Discounts behaviour;
  confirm before relying on the orderline-iteration pattern in production.
- Admin location: **Shipping → Extra Fees** (confirmed via admin screenshot).

---

## Price Variable Bulk Export / Import

**Status: shipped (2026-07-28).** Previously on the roadmap as planned/not-yet-shipped; this is now live.

Price Variables can be exported and imported, including across sites — export to a spreadsheet, edit in bulk, and re-import rather than editing them one at a time in the admin. This is useful for onboarding scoping involving hundreds of price variables, and for replicating a pricing setup from one site to another.

Source: slack-message (#development), commit e954d0b3.

---
## Packaging Tab — Do Not Use

The **Packaging** tab exists in the product admin but is a legacy feature — almost no live clients
use it, and it is not recommended for new setups.

**Why:** In mixed-product orders, per-package prices aggregate unpredictably (multiple products go
into one package, but the formula can't know what else is in the order at pricing time). The result
is often an expensive and confusing checkout experience.

**Recommended approach:** Always use **shipping pricing formulas** instead of the Packaging tab.
If a client asks about Packaging, redirect them to shipping formula configuration.

RATIONALE: Repeat signal — Loom video in #kb-sync + #development question same week.
SOURCE TYPE: loom-video + slack-message
---
## Per-Order Charges Must Never Sit on a Variant

A variant price adjustment is **per orderline** and multiplies by orderline
quantity. A flat per-order amount placed on a variant therefore multiplies by the
item count — the trap that bit the flyer tool, where postage multiplied by the roll
count.

- **Per-order charge** (postage, a handling fee, a rush fee): a separate orderline
  against a dedicated product, or an Extra Fee. Never a variant.
- **Per-item charge** (a finish, a substrate upgrade): safe on a variant, precisely
  because orderline quantity is the item count.

### A per-order charge must never multiply by copies either

The same trap reappears inside a custom design tool. Where a fee is configured `per: "order"`
and the tool writes a **per-unit** number into the price option, the platform multiplies that
number by the orderline quantity and the per-order charge is levied once per copy.

Where `per` is `"order"`, the tool divides the amount by the unit count before folding it into
the per-unit number, **and carries a comment saying why**. Without the comment the division
reads as a bug and gets "fixed" by the next person to open the file.

Stated in a build spec, consistent with the variant behaviour above; not independently
verified in production.

### Custom tools: who owns the quantity break

A `number` template option whose pricing formula is `value` makes the entered
number the price. The division of labour with a custom design tool is fixed:

- **the tool writes a quantity-1 unit figure**
- **the formula owns quantity breaks**

Writing an already-tiered price double-discounts. Writing a finished total freezes
the price at add-to-cart, and the cart quantity stepper then stops re-pricing.

### The escape hatch when the tier driver is not quantity

The division of labour above assumes the tier driver **is** `quantity`. Where it is instead a
product of two inputs — `input_a x input_b`, for example a customer-entered page count
multiplied by the number of copies — the variant / price-variable model can only express it by
smuggling one input into `value` on every branch, which multiplies the formula count by every
new size and every new option branch.

The established escape hatch, proven on a custom framing tool: **compute the price in
JavaScript inside the tool, write one number to a `number` template option, and set that
option's formula to `value`.** The tool owns the whole ladder; the platform only multiplies by
orderline quantity.

Record its cost rather than discovering it later. Two costs, both real:

- **The number is frozen at add-to-cart.** Changing the quantity in the cart will not re-tier.
  Mitigate by owning the quantity control inside the tool on the product page, by making the
  cart stepper read-only for that product, or by prompting the shopper to reopen the tool to
  re-price. This is the honest weakness of the approach and it needs a decision per product,
  not a default.
- **The obvious alternative is worse.** Setting cart quantity to the total page count instead
  lets the formula own the ladder and re-tiers correctly, but then everything downstream keyed
  on quantity — shipping formulas, Extra Fees, per-order charges, packaging, order management
  job counts — starts reading page counts as item counts. That is the same trap as a per-order
  charge on a variant, running in the other direction.

Do not mix the two. Pick one per product.

The JS-computed route is verified in production on one tool. The frozen-price mitigations are
**not verified** — they are design options, not tested behaviour.

---

## Additive Pricing via Hidden `number` Child Variants

A product can be priced additively from hidden `number` child variants — one per selection
branch, each returning `value * <price variable>`. The platform sums every branch's result and
multiplies the sum by the orderline quantity.

Worked shape, a per-page document product: a base rate variant, a color uplift variant, a
heavier-stock uplift variant and a second-side uplift variant, each `value * <rate>`. The
customer enters the page count once; the branches they did not choose return `value * 0`.

Those `value * 0` siblings are **deliberate no-ops**, not leftover configuration. Tidying them
away breaks the branch the moment a rate is set on it.

The model works and is in production, but it has a ceiling: the formula count is the product of
every axis, so each new size or option multiplies the number of formulas to maintain. See *The
escape hatch when the tier driver is not quantity* above for when to stop extending it.

Verified by reading source (product export), 2026-09-08.

---

## A Transcribed Price Table on the PDP Will Drift From the Formula

Any storefront display of a price ladder that is **transcribed** from a pricing formula rather
than derived from it is a mirror, and nothing enforces that the two agree.

This is not hypothetical. A tiered product shipped with a price table that had been generated
from a **different** product's formula, and the product detail page then advertised prices the
cart did not charge. It was caught only because someone compared the header price against the
table.

**Interim rule:** any change to a tiered product's pricing formula must be mirrored into its
price-table field **in the same sitting**. No exceptions, and no "I will update the table after
the cart test".

**This technique has a ceiling, and the fix is a platform capability.** Named platform ask:
expose a product's price ladder to Liquid, so a tier table can be derived from the formula
instead of transcribed from it. Until that exists, every published tier table is a maintenance
liability.

Verified live (the incident), 2026-09-01.

### Withdrawn claim — `product.price` does not evaluate in a mid tier

Recorded because it was stated with confidence and it was wrong.

The claim was that `product.price` evaluates in a middle tier, so every tiered product on every
Shopper site advertises a mid-tier headline price. **Withdrawn.** The product header showed the
formula's correct value at quantity 1. There is no platform fault here — the discrepancy was
entirely a stale transcribed price table, the incident above.

If a tiered product's header price disagrees with its published table, suspect the table first.

---

## Catalogue and Category Pages Show $0 for Variant-Driven Prices

Listing and category pages show **$0** for any product whose price lives entirely on variants
rather than on the product itself. Seen independently at two clients in the same window.

The fix is the **product-level starting-price field**: populate it and the listing shows a real
"from" figure instead of zero.

### `Custom pricing` is the only product custom field that takes free text

- `From pricing`, `To pricing` and `Starting at` are **number** fields. Where the real price
  lives on variants, they render as `$0`.
- `Custom pricing` accepts a **string**, so it is the only one of the four that can carry a
  worded price label. It is **display only** and does not interfere with the pricing formula —
  the product still charges what its formula returns, whatever the label says.

Open, unresolved: the `Custom pricing` label renders at a smaller font than the sibling price
label. Suspected parent snippet CSS. **Not verified.**

Stated from client calls, not independently verified.

---

## Changelog
- 2026-05-19: Added Automatic Discounts section — Liquid-based cart discounts with tiered, user category, and seasonal patterns. Source: Claude chat (webinar prep).
- 2026-07-03: Added Extra Fees (Liquid-Based Cart Fees) section — fee-side twin of Automatic Discounts (adds instead of subtracts), configured under Shipping → Extra Fees. Includes per-duplicate-orderline surcharge pattern (seen-string + contains idiom); orderline-iteration specifics pending live confirmation. Source: Claude chat.
- 2026-07-20: Added property-name trap — `cart.promocode` (without `_code`) is nil and silently defeats promo-code guards; always use `cart.promocode_code`. Source: claude-chat.
- 2026-07-31: Price Variable Bulk Export/Import shipped (2026-07-28) — moved out of Roadmap, now documented as a live feature. Source: slack-message (#development).
- 2026-08-21: Documented Packaging tab as legacy feature; shipping formulas recommended instead. Source: loom-video + slack-message.
- 2026-08-29: Added per-order charges must never sit on a variant — a variant adjustment is per orderline and multiplies by orderline quantity, so postage or a flat fee on a variant multiplies by the item count; use a separate orderline or an Extra Fee. Added the custom-tool division of labour for a `number` option priced as `value`: the tool writes a quantity-1 unit figure and the formula owns quantity breaks, since a pre-tiered price double-discounts and a finished total freezes the price at add-to-cart. Source: claude-chat.
- 2026-09-09: Corrected the Extra Fee context list — `cart.custom.<field>` does resolve inside an Extra Fee formula, confirmed on a live production `rush` fee, and the list is now marked as observed-so-far rather than exhaustive. Added the bare-arithmetic-expression formula shape (`orderlines_total` / `orderlines_discount` unprefixed, same evaluator as Automatic Discounts) with the Liquid interpolation pattern for percentage fees, the rule to charge percentage fees on the post-discount subtotal, the per-fee Taxable checkbox in Main Admin -> Shipping -> Extra Fees with its open question about the Shopper checkout display, and the note that a cart-level minimum-order fee survives a 100%-off promo code. Source: claude-chat.
- 2026-09-09: Added the variant price formula validator section — the field's validator is narrower than Ruby and rejects with `Error saving: Price isn't valid`, with the accepted/rejected table, the practical grammar rule, and the bisect-from-a-known-good-formula debugging rule. Source: claude-chat.
- 2026-09-09: Recorded that `value` and `quantity` are both in scope in a variant price formula and can be combined (`range.include?(value * quantity)`), with the working formula shape, the warning that a successful save does not prove the engine prices it, and the note that whole-order stepped ladders are not monotonic. Source: claude-chat.
- 2026-09-09: Added `pages` behaviour on upload-driven design products — the project keeps the template default page count when the editor is never opened, so a page-count formula prices the default; cross-referenced to the `project.page_count` fulfillment rule. Source: claude-chat.
- 2026-09-09: Added that a price formula will not save until the price variables it references exist, and that Price Variables are not reachable via the API (confirmed by the core developer, 2026-09-07). Source: slack-message.
- 2026-09-09: Extended the quantity/units note with the switching cost — a find/replace across every formula on the product, and a commercial decision. Source: claude-chat.
- 2026-09-09: Added the escape hatch for a tier driver that is a product of two inputs (JS-computed price into a `number` option priced as `value`) with its two costs, and the rule that a per-order charge written by a tool must be divided by the unit count. Source: claude-chat + fireflies-call.
- 2026-09-09: Added additive pricing via hidden `number` child variants (`value * 0` siblings are deliberate no-ops), the transcribed-price-table drift hazard with the same-sitting mirroring rule and the platform ask to expose a price ladder to Liquid, and the withdrawal of the claim that `product.price` evaluates in a mid tier. Source: claude-chat.
- 2026-09-09: Added zero-priced catalogue/category pages for variant-driven prices (fix is the product-level starting-price field) and the note that `Custom pricing` is the only product custom field accepting free text for a price label. Source: fireflies-call.
