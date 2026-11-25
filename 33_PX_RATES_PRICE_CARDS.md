# 33 — Price Cards (`px_rates`): Reference for Labs and Their Claude

**Authority Scope:** How a `px_rates` price card is built, where it goes and how to check it prices the same on the website and in OrderHub. Tool-level (Uploader 2, Custom Sized Prints, Sticker Studio) on a platform product custom field. Formulas, variants and price variables stay in `30_PRICING_ENGINE.md`.

_Last updated: 2026-11-24_

---

## 1. What a price card is

Some Pixfizz tools do not use a pricing formula. They price from a **price card**: one JSON block holding every choice and every rate. The tool works out the price per item, writes it to the hidden Number variant `px_price` (Price formula `value`) and the cart charges `px_price` x quantity. Every other choice the customer made is written to `px_` variants (`px_pages`, `px_paper`, `px_color`, `px_binding`, ...), so OrderHub and the till can see and reprice the line.

- **Where it lives:** the product custom field `px_rates` ("price card") on the Product Attribute. Paste the JSON there and press Save. *Verified by reading source (Uploader 2.6.0), 2026-11-02.*
- **Fallback:** Uploader 2.6.0 and later read `px_rates` first and the card inside the template's `px_mount` option second. A generic template can carry a sample card in its mount; each site prices on its own product's `px_rates`. OrderHub reads `px_rates` only, so a product priced only from the mount cannot be priced at the till.
- **Rates change in the card only.** Never in the tool settings, never in a formula. New rates are not a new engine version.
- The field needs a definition on the site before a value can be stored (`51_CUSTOM_FIELDS_REFERENCE.md`).

## 2. One engine, three places

One file, `pricing.js`, does the arithmetic. The storefront tool bundles it, OrderHub runs it, the POS runs it. The same card and the same choices give the same price in all three, to the cent.

| `pricing.js` version | Adds | Status (24 Nov 2026) |
|---|---|---|
| 1 | `flat` (documents, copies, flyers) | live |
| 2 | `booklet` (saddle stitch, separate covers) | live |
| 3 | simple booklets (per page rates, bindings) | live |
| 4 | `area` (custom sizes: prints, metal, acrylic, canvas, banners, counter jobs) | live in OrderHub and the POS from 5 Nov 2026 |
| 5 | `prints` (Flat Prints), `forms` (NCR), `sheet` (stickers) | built 20 Nov 2026, flat, booklet and area prices identical to v4. **Not installed in OrderHub yet** (OrderHub runs v4, confirmed 24 Nov 2026): Flat Prints, NCR forms and sticker products sell on the website but cannot be priced at the till until it is |

**The `"engine"` number on a card** is the lowest version that can price it. Rules:
1. A card is refused by any engine older than its `"engine"`. On the storefront the page then shows "This product is not set up yet" and Add to cart is off. *Verified live, 2026-11-10.*
2. Keep `"engine"` at the lowest number that has the mode: flat 1, booklet 2, simple booklet 3, area 1 (metric units 2), prints, forms and sheet as their starter card says. Do not raise it "to be current".
3. The storefront must never be older than any card on the site, and OrderHub must run a version that has the card's mode. A mode the website prices but OrderHub's `pricing.js` lacks works for web orders and fails at the till.
4. OrderHub refuses a card with no `"engine"` (from 9 Nov 2026). Always include it.

## 3. Modes at a glance

| Mode | Tool | Price is driven by | Key card parts |
|---|---|---|---|
| `flat` | Uploader 2, documents | sheets or pages x copies, per size, color, sides, paper | `sizes[]` with `print` bands and `papers[]`, `finishing[]`, `labels`, `defaults` |
| `booklet` | Uploader 2, booklets | pages, copies, paper, color, cover, binding (see § 4) | `trim`, `binding`, `interior`, `cover` or `bindings`, `overrun`, `max_pages` |
| `area` | Custom Sized Prints, counter products | width x height in `unit` (sq_in, sq_ft, sq_cm, sq_m), per stock | `unit`, `tiering` (`whole` or `graduated`), `stocks[]` with `tiers` (`to` is exclusive: an area equal to `to` falls in the next band; the last band has no `to`), `min`, `base`, `extras[]` |
| `prints` | Uploader 2, Flat Prints | press sheets used for the cheapest layout, plus cutting | `sizes`, stocks, `press`, `cutting`, `geometry` |
| `forms` | Uploader 2, NCR forms | a price per quantity for finishing x parts x color x size | `finishing[]` with `qtys`, `parts[]`, `sizes[]`, `prices`, `numbering` |
| `sheet` | Sticker Studio | pieces per press sheet, setup plus per sheet ladder, minimum | `sheet`, `setup`, `min`, `tiers`, `size` |

Start every new card from a starter card or from the product's current card, never from a blank page. Pixfizz has starters for documents, letter booklets, simple booklets, NCR forms, Flat Prints, custom sizes and stickers.

## 4. Booklet cards in detail

A booklet card has `"mode": "booklet"`. It has one of two shapes.

### 4a. Simple booklet (`"engine": 3`)
No cover choices: the cover is the first and last pages of the customer's file, on the same paper. Recognised by `interior.rate_per: "page"` and a `bindings` list, and no `cover`.

- **Price per booklet = pages x page rate + binding price.**
- Page rate: `interior.print.<color>` bands, or the chosen paper's own `print.<color>` bands when the paper has them. Bands are looked up on **all pages in the job** (pages x copies), so later rows can lower the rate for big jobs.
- Binding: `bindings[].bands`, a `price` per booklet looked up on **copies ordered**. 0 means included.
- Pages are padded to `binding.multiple_of` (4); the padded count is priced. `binding.min_pages` is the fewest pages.

```json
{"name":"Booklets 5.5 x 8.5","mode":"booklet","engine":3,
 "trim":{"w":5.5,"h":8.5,"label":"5.5 x 8.5 in"},"bleed":0.125,"printable_margin_in":0,
 "binding":{"code":"booklet","label":"Booklet","multiple_of":4,"min_pages":4,"pages_per_sheet":4},
 "overrun":{"pct":0},
 "interior":{"rate_per":"page",
  "colors":[{"code":"bw","label":"Black & White","desc":"..."},{"code":"color","label":"Full Color","desc":"..."}],
  "print":{"bw":[{"min":1,"max":9999999,"rate":0.14}],"color":[{"min":1,"max":9999999,"rate":0.24}]},
  "papers":[{"code":"regular","label":"Regular (20 lb.)","desc":"..."},
   {"code":"gloss100","label":"Gloss (100 lb.)","desc":"...","print":{"bw":[{"min":1,"max":9999999,"rate":0.18}],"color":[{"min":1,"max":9999999,"rate":0.28}]}}]},
 "bindings":[{"code":"staple","label":"Staple Binding","desc":"...","bands":[{"min":1,"max":9999999,"price":0}]},
  {"code":"coil","label":"Coil Binding","desc":"...","bands":[{"min":1,"max":4,"price":3.75},{"min":5,"max":9,"price":3.0},{"min":10,"max":24,"price":2.63},{"min":25,"max":9999999,"price":1.88}]}],
 "volume_steps":[1,5,10,25,50,100],
 "defaults":{"int_color":"bw","int_paper":"regular","binding":"staple","copies":1,"blank_at":"obc"}}
```
Example rates are illustrative. Worked check with this card: 24 pages, 10 copies, Full Color, Gloss, Coil = 24 x 0.28 + 2.63 = **$9.35** a booklet, $93.50 total. *Verified by running pricing.js v5, 2026-11-24.*

### 4b. Saddle-stitch booklet with covers (`"engine": 2`)
Recognised by `cover.prints` and a `bindery` table. Per booklet:
- **Inside pages:** a rate per sheet (one sheet = `pages_per_sheet` pages, 4), from `interior.print.<color>` or the paper's own bands, looked up on **sheets in the whole run**. With a separate cover the cover's 4 pages are not counted.
- **Separate cover** (only when the cover paper is not `"self": true`): `cover.papers[].print.<cover print code>` bands, a price per cover looked up on copies ordered (`cover.tier_on: "copies"`) or on the run (`"run"`). A self cover prints with the inside pages and the cover print choice does not change the price.
- **Stitching:** `bindery` bands, a price per booklet looked up on **sheets per booklet**. With a separate cover an 8-page booklet has 1 sheet: the table must start at 1 if that size is sold, or it is not priced.
- **Overrun:** `overrun.pct` extra copies when copies are at least `copies_gte` or pages are more than `pages_gt`; their cost is spread over the copies ordered.
- `max_pages[<cover paper or "same">][<inside paper>]` limits pages; papers in `max_pages.provisional` warn instead of stopping.

## 5. Rules for editing a card

1. **Change rates, keep codes.** `code` values are what the tool and the card use internally. Changing one breaks defaults and saved choices.
2. **Labels are written on the order line** (`px_paper` "Gloss (100 lb.)") and OrderHub matches them back to the card. Renaming a label is fine for new orders, but an old order line with the old label can no longer be repriced from the card (website orders still carry what was paid in `px_price`). Rename before launch, not after.
3. **Bands:** each list starts at 1, rows run on without gaps or overlaps (`next.min = previous.max + 1`), and the last row ends high (9999999). A value outside every row has **no price**, never the neighbour's price.
4. Rates are dollars (or the site currency) as plain numbers: `0.28`, not `"$0.28"`.
5. Booklet page and sheet bands use `"rate"`; binding bands use `"price"`.
6. Keep the card one JSON object. Valid JSON only: double quotes, no trailing commas, no comments.
7. **Area cards: every `extras[].values` entry needs a `"label"` that matches the variant value name on the product** (for example `"2ft":{"grommet":0.75,"label":"Every 2 ft"}` for the value "Every 2 ft"). OrderHub matches the order line's value to the card by code or label; a value whose name is not its code and has no label is refused with reason `order-line`, while the website still prices it. *Verified by test on pricing.js v4 and v5, 2026-11-24 (an extra whose values "None" and "Corners" priced while "Every 2 ft" and "Every 3 ft" were refused until labels were added; prices unchanged).*
8. **Before saving, run the order-line check:** for each choice, build the line the website writes (`PxPricing.variants`) and price it back with `PxPricing.priceOrderLine`; every combination must give the same total as `PxPricing.price`.

## 6. Check before and after saving

1. The JSON parses and `PxPricing.canPrice(card).ok` is true (reasons: `bad-engine`, `engine`, `mode`, `unpriced`).
2. Price three real jobs you have quoted before and compare.
3. Paste into `px_rates`, Save, open the product page: the uploader shows the same price.
4. Sync OrderHub, then a till test: a custom order in OrderHub for the product gets the same price. `30_PRICING_ENGINE.md` § Uploader Price Cards, `45_ORDERHUB.md` § Repricing Custom Tool Lines.

## 7. Rounding

Whole cents inside the engine. Booklets: the per-booklet price is rounded half up to the cent and the total is that x copies. A card priced per job (`"price_per": "job"`, documents and Flat Prints) writes total / copies to `px_price` at 4 decimals. The customer always pays `px_price` x quantity; never replace that with a repriced figure on a website order.

## Changelog
- 2026-11-24: OrderHub runs v4 (v5 not installed); area band `to` is exclusive (90 sq in on a `to: 90` band prices at the next band, tested).
- 2026-11-24: Editing rules 7 and 8: area extras need labels matching the variant value names, and the order-line round-trip check.
- 2026-11-24: Page created. Price card location and fallback, engine versions 1 to 5 and the compatibility rules, the six modes, booklet cards (simple and saddle stitch with covers) with a worked check, editing rules and the check list.
