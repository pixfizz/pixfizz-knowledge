# Design Service (Design Brief): tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: beta, version 2.3.1 (live on the shopper24 parent; first client installs October 2026). Snippet: `product/design-brief`. Mount option: `px_mount` on new installs (older installs: `dsn_mount`). Prefix: `dsn_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

A "we design your photo book for you" order builder. The shopper picks a book type, size and length, sends photos (or an album link), marks must-have photos, tells the story, and orders. Your team designs the book. The price is the Photo Book Studio book price plus your design fee.

Example mount (the kit value; fill in `dsn_books` with your own product ids):
```liquid
{% snippet 'product/design-brief',
	dsn_skin: 'atelier',
	dsn_types: 'hard,soft,layflat',
	dsn_books: 'hard=8.5x8.5:1001,10x10:1002|soft=8.5x8.5:1003',
	dsn_paper: '',
	dsn_pages: '',
	dsn_auto_pages: '60',
	dsn_units: 'in',
	dsn_guidance: 'auto=150|40=100:50 to 70|60=120:75 to 100|80=180:120 to 140',
	dsn_min_photos: '20',
	dsn_max_must: '20',
	dsn_draft_days: '5',
	dsn_rounds: '2',
	dsn_approve_days: '21',
	dsn_occasions: 'Family year|Travel|Wedding|New baby|Birthday|Memorial|Something else',
	dsn_alt_url: '',
	dsn_alt_label: '',
	dsn_exit_url: '/',
	dsn_max_mb: '50',
	dsn_brand: '' %}
```
No apostrophes and no curly brackets anywhere in the mount values.

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `dsn_skin` | The look of the brief | `atelier`, `noir`, `fresh` | `atelier` | Yes |
| `dsn_types` | Book types offered, in order; the first is preselected. A type shows only if `dsn_books` gives it a product | any of `hard`, `soft`, `layflat`, comma separated | `hard,soft,layflat` | Yes |
| `dsn_books` | Sizes offered and the Photo Book Studio product that prices each one (sizes in inches, width x height) | `type=WxH:product id,...` per type, types split by `|` | none, required | With care: every id must be a live book product of that type and size |
| `dsn_paper` | Paper option code the book is priced and printed on | option code, or `type=code` per type split by `|`, or blank when priced by pages alone | blank | With care: must match the paper option code on your book products |
| `dsn_pages` | Page counts the books can be made at; lengths outside are not offered | `type=min-max` split by `|` | blank (no limit) | With care: must match what your book products allow |
| `dsn_auto_pages` | Page count "Let the designer decide" is priced at | whole number | `60` | With care: it sets the price for that choice |
| `dsn_units` | Units shown to the shopper (sizes in `dsn_books` stay in inches) | `in`, `cm` | `in` | Yes |
| `dsn_guidance` | Length choices: page count = most photos to send : typical photos used; `auto` lets the designer decide | `auto=most`, then `pages=most:text` entries split by `|` | see example | Yes; each page count is the length the shopper pays for |
| `dsn_min_photos` | Fewest photos before the shopper can continue (an album link also counts) | whole number | `20` | Yes |
| `dsn_max_must` | Most must-have photos | whole number | `20` | Yes |
| `dsn_draft_days` | Working days to the first draft, shown on review | whole number | `5` | Yes |
| `dsn_rounds` | Rounds of changes included | whole number | `2` | Yes |
| `dsn_approve_days` | Days to approve after the first draft | whole number | `21` | Yes |
| `dsn_occasions` | Occasion choices on the Story step | text split by `|` | see example | Yes |
| `dsn_alt_url`, `dsn_alt_label` | Optional link under Start for shoppers who prefer to make the book themselves; shown only when both are set | page URL; text | blank | Yes |
| `dsn_exit_url` | Where Exit takes the shopper | page on your site | `/` | Yes |
| `dsn_max_mb` | Largest photo accepted | whole number, MB | `50` | Yes |
| `dsn_brand` | Your store styling over the skin | `key=value` pairs split by `|`; keys `primary`, `on_primary`, `tint`, `ink`, `muted`, `faint`, `canvas`, `surface`, `stage`, `stage_hi`, `line`, `line_strong`, `accent`, `heading`, `body`, `mono` (Google font names), `heading_weight`, `radius` (px), `buttons` (`upper`) | blank | Yes |
| `dsn_variant_prefix` | Prefix of the brief's variants | text | `dsn_` | No, unless renamed |
| `dsn_spec_code`, `dsn_brief_code`, `dsn_preview_code` | Options the job record, brief JSON and cart preview are written to | option code | `px_spec`, `dsn_brief`, `px_preview` | No, unless renamed |
| `dsn_sizes` | Older size list, replaced by `dsn_books` | | blank | No; use `dsn_books` |

Things that are not set here
- Your design fee: the price of the "Photo Book Design Service" product attribute. Leave the hidden "Book price" variant (`dsn_book_price`) as it is.
- Book prices: they come from your Photo Book Studio products.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
