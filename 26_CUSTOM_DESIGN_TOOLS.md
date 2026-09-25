# 26 — Custom Design Tools

**Authority Scope:** custom design tools as a class — what they are, the estate that
exists, how one is configured, mounted, installed and verified. Per-tool behaviour
and current status.

**Not in scope:** the standard Design Tool (the platform editor) — that is
`17_DESIGN_TOOL.md`. The browser-side engineering rules shared by every tool stay in
`17_DESIGN_TOOL.md` § Custom Design Tools — Browser-Side Rules; this file points at
them rather than repeating them.

_Last updated: 2026-09-19_

---

## 1. What a custom design tool is

A **custom design tool** is a browser-based tool that mounts on a Shopper product
page, takes a customer's own artwork or specification, checks it, prices it where the
platform cannot, and writes files and values onto the orderline. It replaces the
standard editor for products the editor was never meant to serve: a print-ready PDF
supplied by a designer, a contour-cut sticker, a gang sheet packed from many small
images, a document copied at a copy-shop rate card.

It is **not** a Design Tool Configuration, and it is not a theme. A product runs
either the standard editor or a custom tool, never both.

### When a custom tool is the right answer

Reach for one only when all of the following are true. Anything less is a template
with options and variants, which is cheaper to build and cheaper to install.

- The customer supplies a finished file, or specifies rather than designs.
- Something must be **checked in the browser** before the order is taken — page
  count, trim size, bleed, resolution, cut geometry.
- The price or the production file cannot be derived by the platform from variant
  selections alone.

The pricing point is the common one and it is worth stating plainly: **a Pixfizz
pricing formula sees `value`, `quantity`, `units`, the `pages` family and Price
Variables, and is blind to option and variant selections.** A rate card that depends
on paper × colour × sides × binding cannot be expressed as a formula. A tool resolves
it in the browser and writes one number to a single number variant whose formula is
`value`. See `30_PRICING_ENGINE.md`.

---

## 2. The estate — registry

Every tool currently in existence. **Status is Alex's, recorded 19 September 2026;
versions are verified by reading the asset source in the Custom Tools working
folder on the same date.** A tool absent from this table does not exist.

| Tool | Asset | Version | Prefix | Mount option | Status |
|---|---|---|---|---|---|
| Sticker Designer | `sticker-designer.js` | v5.30 | `sticker_` / `stk_` args | `sticker_artwork` | **Live.** Needs updates. |
| Gang Up (DTF gang sheet) | `gang-up.js` | `2026.09.15-1` | `gangup_` / `gu_` args | `gangup_artwork` | **Live.** |
| Business Cards | `business-card.js` | `2026.09.15-1` | `bc_` | `bc_artwork_front` | **Live.** |
| Cover Studio | `cover-studio.js` | `1.8.0` | `cvs_` | template option per install | **In test.** Almost there. |
| Publication Upload | `publication-upload.js` | `1.12.0` | `pu_` | `pu_text` | Built, **never tested through production.** |
| Document Uploader | `document-uploader.js` | `2026.09.15-7` | `du_` | `du_file` | Built, **never tested through production.** |
| Booklet Uploader | `booklet-uploader.js` | `2026.09.16-1` | `bu_` | `bu_file` | Built, **never tested through production.** Fork of Document Uploader. |
| Fan Face | `face-fan-designer.js` | delivered 30 Jul 2026 | `facefan_` | `facefan_artwork` | **Not ready.** Head cut needs real AI segmentation to be sellable. |
| Flyer / Brochure | `flyer-fold.js` | `2026.08.14-2` | `flyer_` / `fld_` args | `flyer_artwork` | **Never finished.** |
| Design Brief (Design Service) | `design-brief.js` | `1.0.0` | `dsn_` | `dsn_mount` | **Unfinished.** |
| Film Order Builder | `film-builder.js` | prototype, no version | `pxfb_` | host element `#pxfb` | **In progress now.** Rate card in the asset is demo data. |
| Restoration Estimator | not built | — | `rst_` | `rst_original` | **Specified 4 Aug 2026, never started.** Needs revisiting. |
| Acrylic Statue Designer | not built | — | `astat_` | `astat_artwork` | **Specified 21 Aug 2026, never started.** Needs revisiting. |
| Collage Wall | not built | — | — | — | **Unfinished.** |
| Custom Sized Prints | not built | — | — | — | **Just started**, spec 18 Sep 2026. |

**Version banners are not reliable on every tool.** Three cases found on 19 Sep:
Sticker Designer's header banner still reads `2026.08.26-11` while the asset is
v5.30; Cover Studio's file is named `cover-studio-v1.7.js` and its `VERSION`
constant reads `1.8.0`; Business Cards and Gang Up carry a `@version` banner but no
`VERSION` constant, so the deployed build cannot be confirmed from the console.
**Every tool should expose a `VERSION` constant and log it at init**, because an
asset is aggressively browser-cached and "did my change deploy" is otherwise
unanswerable. Document Uploader, Booklet Uploader, Publication Upload, Cover Studio
and Design Brief do this correctly.

### Related, but not custom design tools

- **Photo Prints review modal (`prints-review.js`, prefix `prv-`)** — a parent asset
  that adds a review step to the platform's own photo prints flow. It mounts from
  `product/prints-review` and is gated on `admin/checklist/prv-enabled`, not on a
  `custom_script` template option. See `claude/PRINTS_REVIEW_MODAL_REFERENCE.md`.
- **1 Hour Frame (framing)** — shipped as eight templates and products with baked-in
  custom fields, not as a browser tool. The `framing_*` product fields in
  `51_CUSTOM_FIELDS_REFERENCE.md` are from the earlier framing-tool spec.

---

## 3. Anatomy of a tool

Five parts, and **all five live on the shopper24 parent**. A child site can only
override a snippet the parent already has; it cannot create one. A lab admin cannot
install any of this, and an instruction telling them to would fail with a blank
render rather than an error.

| Part | Where | Note |
|---|---|---|
| `<tool>.js` | Site asset on the parent | The whole tool. |
| `product/<tool>` | Parent snippet | Markup, config resolution, the script tag. |
| `style/<tool>` | Parent snippet | Scoped CSS. |
| The mount | `custom_script` custom field on one template option | Per template. **Travels with a template export.** |
| Template options | Per template | The tool's contract: what it writes to the orderline. |

The shared foundation is installed **once** and consumed by every tool: `px-notify.js`,
`style/px-notify`, `product/px-notify-boot`. It is not part of a tool's delivery. A
tool must fall back to a plain banner and log `[<tool>] px-notify.js did not load`
rather than aborting, and must **never load the shared asset from inside its own boot
block** — one parse error there takes down the shared layer and points the symptom at
the wrong file.

`product/px-artwork-place-boot` and the shared px-artwork-place engine are the same
shape: shared, loaded separately, and every call into them fails open.

The `pixfizz-internal:custom-design-tool` skill carries the skeleton and the
`check-tool.mjs` verifier. Use it to build one; use this file to know what exists.

---

## 4. Configuration — the mount argument list, and nothing else

> **A custom design tool reads its configuration from its mount argument list, and
> from nowhere else.** No product custom field, no design custom field, no site
> checklist snippet.

Decided 11 September 2026 and **built for Business Cards the same day. This
supersedes the settings cascade** ("product custom field → site checklist → built-in
default") described in every tool spec written before that date, and the four-step
install order previously carried in this knowledge base. See
`claude/CUSTOM_TOOL_CONFIG_ONE_SOURCE.md` and
`claude/CUSTOM_TOOL_CONFIG_MUST_TRAVEL.md`.

### Why

**1. It travels.** A template export carries template options and their custom
fields. A custom field on a product or a design does not: its *definition* must be
created per object per site before a value can even be stored, and a missing
definition is indistinguishable from an unset value — both read as blank and the tool
falls back to a default that looks plausible. That is the single most painful part of
every install and the step most often half-done.

**2. Most of the old cascade could never fire.** `product` is nil inside a
`custom_script` mount unless the URL carries `?product=<id>`, and a Shopper
collection-style product page does not. Inside the snippet the mount calls, **both
`product` and `design` are nil** — only keyword arguments cross that boundary.
Verified live on cartolinaonmain.com, 10 Sep 2026. So `product.custom.*` in a tool
snippet was dead code that read like a working cascade.

**3. A site checklist is another object that does not travel**, and it puts product
geometry in Custom Admin, a screen nobody associates with a product.

### The one exception

`custom_script` itself — a custom field **on the template option**, which is why it
travels. Its definition still has to exist on the child site before a template
carrying a value for it can be imported. Nothing else works until it does.

### The mount gets its own hidden option (standard from 2026-09-20)

A tool's mount block no longer sits on its artwork upload option. It gets a **dedicated option**: type Text, hidden, hide from cart, code `<prefix>_mount` (for example `bc_mount`, `bu_mount`, `cvs_mount`), name "`<Tool>` Mount". Its `custom_script` holds only the `{% snippet 'product/<tool>', … %}` mount block. Reasons: the artwork option has its own required flag and upload behavior, and a tool with no artwork option has nowhere else to mount. The "Mount option" column in § 2 lists where each tool mounts today; tools move to the dedicated option as they are rebuilt. *Decided by Alex, 2026-09-20.*

### The counter test

A custom tool is not finished until someone at the counter, with the customer's file in hand, can build the same order from the Product Attribute's variants alone, with no template, no design and no browser tool. Artwork and anything derived from it are exempt. Tools whose price is computed in the browser from a rate card and written to a number variant (the booklet and document uploaders) do not pass yet; that is open. *Decided by Alex, 2026-09-20.* See `22_OPTION_VARIANT_RENDERING.md` § Pricing and POS-Relevant Choices Belong on Variants.

### Configuration is not choice

| | What it is | Where it lives |
|---|---|---|
| **Configuration** | how this product behaves — size, bleed, dpi, output mode, which controls are on | mount arguments |
| **Shopper choice** | every customer selection and every commercial value — sides, paper, finish, binding, quantity | **variants**, read live by the asset |

Sides, stock, corners and binding stay variants because they carry price, and price
is a product-level concern. The mount only says which variant *code* to read, for a
site that named one differently.

### Rules for a mount block

- **Pass every argument, every time**, even where the value is the default. A tool
  must list what the mount did not pass in `data-config-missing` and report it as a
  console error naming each one. A built-in default may exist so a missing argument
  cannot kill a live storefront, but it may never be quiet.
- **Blank is a statement, not an omission.** Where the mount genuinely cannot know a
  value — Business Cards' `bc_sides`, which differs per product on one template — it
  passes blank, the snippet names it in `data-config-missing`, and the asset resolves
  it from the variant. A probe that guesses turns "I cannot see the product" into a
  confident wrong answer.
- **Apostrophes are not escapable in Pixfizz Liquid.** Never put one in a mount
  value. See `feedback_liquid_no_escape`.
- **Emit `data-product-seen`** and a source marker beside every resolved value. A
  live tool reporting `productSeen: "false"` is working only because the fallbacks
  happen to be right.
- Numbers arrive as strings. `| plus: 0` before any comparison.

---

## 5. Install order on a new site

Corrected 19 September 2026 for the mount-argument model. The old four-step order,
which ended in product custom field definitions and checklist values, is retired.

1. **Template-option custom field schema.** Export it from a site that already runs a
   tool and import it at the **template option** object. This is what creates
   `custom_script`. Definitions do not inherit from the parent. Nothing below works
   until this exists. In admin this is the "template-option custom field schema
   export" — the phrase "the `custom_script` custom field definition" matches nothing
   a person sees on screen.
2. **Import the template export.** It carries the tool's template options, their
   `hidden` / `read_only` / `hide_from_cart` flags, the variants, and the mount block
   in `custom_script`.
3. **Set prices.** Bundles ship at 0.00. Ranges must cover the full quantity span
   with no gaps — a gap returns nil and breaks the product page, not just the price.
4. **Verify the mount on the live URL.** Open the product page and confirm the tool
   renders, then read the console line for the resolved configuration and
   `data-config-missing`.

Steps 1 and 2 used to be steps 1 to 4. That is the whole benefit of the change.

**Re-check `hidden` and `read_only` on every template option and variant after any
import.** Variant exports write unset booleans as quoted strings, so "unset" imports
as `'false'`, which reads as true. Never set `read_only` on anything the tool writes
to: it renders a display chip plus a hidden input, and the tool finds nothing to
drive. No `number` option may ship with a default value, or Add to Cart fails with
"value is a required field".

---

## 6. Shared surfaces a tool must reach

A tool's output has to appear in six places. Five of them carry a **hardcoded list of
preview option codes**, which is wrong by construction: the list is out of date the
day the next tool ships.

| Surface | Snippet | Variable |
|---|---|---|
| Cart | `shopper/cart` | `flat-preview-codes` (hyphen) |
| Cart fly-out | `modals/shopping-cart` | `flat_preview_codes` (underscore) |
| Saved projects | `account/v2/projects` | `flat_preview_codes` |
| Product gallery | `product/gallery` | `flat_preview_codes` |
| Order details | `checkout/orderline-preview` | **no list** — prefer this |
| Confirmation email | order email template | must include `file_upload`, and must read `hide_from_cart` |

**Where a surface is long-lived, move it to `checkout/orderline-preview`, which
carries no list.** Where a list stays, adding the tool's preview code to it is part
of that tool's definition of done. See `claude/ORDERLINE_PREVIEW_SURFACES.md` and
`20_SHOPPER_CART_RULES.md`.

Two live defects to know about: `modals/shopping-cart` assigns the list with an
underscore and tests it with a hyphen, so the fly-out preview has never rendered for
any custom tool; and the confirmation email excluded `image_upload` and not
`file_upload`, which is what every tool artwork option actually is.

---

## 7. Per-tool reference

Each entry states what the tool does, what it writes, and what is open. Build specs
and session history live in project docs; this is what a session needs to work.

### Sticker Designer — live, needs updates

Contour-cut sticker design in the browser: upload, background removal by flood fill,
contour trace with RDP simplification and Chaikin smoothing, chamfer dilation for the
cut offset, minimum cut radius guarantee, and a PDF assembled with the cut path as a
spot colour. Size and cut type come from the product's **variants** (codes like
`3x2`) and the artboard resizes live. Two-way: the modal drives the page control and
the page control drives the modal.

Mount arguments (`stk_*`): display mode (`modal` or `inline`), cut spot name, stroke
width, cut layer, overprint, split cut file, max MB, target dpi, price footer,
quantity tiers, type toggle, artwork check, requires design.

Writes: `sticker_artwork`, `sticker_preview`, `sticker_cutfile`, `sticker_original`,
plus read-only geometry (`sticker_cut_type`, `sticker_cut_offset`, `sticker_rotation`,
`sticker_pan_x`, `sticker_pan_y`, `sticker_bg_removed`, `sticker_bg_tolerance`).

`inline` display mode renders the designer open in the page with no launch button,
for a dedicated design page; `modal` is the default and an unknown value falls back
to it. The Roll / Die-Cut toggle parks the design in IndexedDB across the
collection-filter navigation and restores it on the other side.

### Gang Up — live

Gang sheet builder for roll goods (DTF transfer rolls). Sheet **width** is fixed by
the product; **length grows** to fit what the customer adds, then snaps up to the
next available value of the length variant, which is what drives price. The lab
defines what lengths it sells by creating variant values; the asset carries no
hardcoded increment.

Mount arguments (`gu_*`): mode, sheet width and height, target dpi, margin, gap, min
and max length, max file MB, min artwork size, accepted formats, background, manual
arrange, requires design, price footer, price and Add to Cart selectors.

Writes three files, each scoped to its own option — `gangup_artwork` (flattened print
PNG with alpha), `gangup_preview` (cart thumbnail), `gangup_originals` (zip of source
files plus `manifest.json`, for re-edit) — and read-only `gangup_source`,
`gangup_designs`, `gangup_copies`, `gangup_sheet_length`, `gangup_output_dpi`.

### Business Cards — live

Artwork upload, preflight and proof for business cards. On the default **NORMALISED**
output mode the tool *builds* the production PDF at the template's page size and the
customer's upload is kept as an asset on the order; on **PASSTHROUGH** the customer's
file is the production file. The customer can drag and resize their artwork on the
proof, and the proof, the cart thumbnail, the review image and the print file all
render from that one placement. A 3D preview turns the trimmed card.

Mount arguments (`bc_*`): sides (blank when the mount cannot know), sides variant
code, units, trim width and height, bleed, safe, min dpi, max MB, output mode,
placement mode, show bleed, preview, reposition, price footer, requires design, block
on error, designer link, human check, display mode.

Writes: `bc_artwork_front`, `bc_artwork_back`, `bc_preview`, `bc_preflight`,
`bc_proof_state`, `bc_print_front`; reads and writes the `bc_stock`, `bc_shape`,
`bc_corners` and `bc_sides` variants.

**Known defect:** the print file embeds the original bytes, which pdf-lib places
without the EXIF Orientation tag, so a tagged phone photo reaches the press turned a
quarter turn from the proof. See `claude/BC_EXIF_ORIENTATION.md`.

### Cover Studio — in test

Foil imprint configurator for album and photo book covers. **Produces no print file:**
the cover is `fulfillment="false"` and the foil is struck from a physical die, so the
output is option values only, written to the product or project form. Mounts inline
on the project-edit hub as well as on the PDP; the same asset runs in both places and
field names are suffix-matched (`variants[code]` on the PDP, `book[options][code]` on
project-edit).

Mount arguments (`cvs_*`): cover dimensions, size, family, rate per line, rate per
spine line, design button selector. Writes `cvs_preview` and `cvs_sheet`.

### Publication Upload — built, untested in production

Print-ready PDF upload with browser preflight, for fixed-specification publications
(assessment books, workbooks, perfect-bound titles). Checks: opens, not encrypted,
page count against the title spec, page size against trim plus bleed in both
orientations, bleed presence and consistency, lowest placed-image resolution,
non-embedded fonts (best effort). Measures geometry from the **page boxes the file
declares** (TrimBox, BleedBox, MediaBox) when present, so an export carrying crop
marks and colour bars is measured on its trim, falling back to overhang inference.

Does **not** check colour space, overprint, transparency or trapping — pdf.js cannot
see them reliably, so they are reported as not checked and left to prepress.

Mount arguments (`pu_*`): pages, trim width and height, cover mode, bleed, min dpi,
soft dpi floor, max file MB, block on fail, partner, generic, upload required, isbn,
file, file name. Option codes (`pu_text_code`, `pu_cover_code`, `pu_preview_code`,
`pu_preflight_code`, `pu_state_code`, `pu_changes_code`) are the contract and are
passed only by a site that renamed one.

`pu_pages` blank means no page count check runs at all. That is the tool's only hard
fail, so blank is a real decision rather than an omission — and `pu_generic: true`
declares a title deliberately unspecified, which reads differently to a setup nobody
finished. **"Generic by design" and "someone forgot to fill the fields in" are
different states and must not be inferred from absence.**

### Document Uploader — built, untested in production

Upload a document, confirm its page count, configure, price, add to cart. No
personalization, no editor, no page replacement. **The tool computes the price**,
because a pricing formula cannot read the selected paper, colour, sides or finishing:
it resolves the rate card in the browser, computes one per-copy figure, and writes it
to a single number variant whose formula is `value`. **Rates never live in the asset**
— they live in the rate card snippet, which is the only thing that changes per lab.

Mount arguments (`du_*`): max file MB, min dpi, full width, single cart, omit empty,
merge size into paper, thumbs max, brand, write preview, confirm pages, block on
error, and the option code overrides. Writes `du_file`, `du_preview`, `du_price`,
`du_pages`, `du_spec`, `du_size`, `du_color`, `du_sides`, `du_paper`, `du_sheets`,
`du_notes`, `du_finishing`.

### Booklet Uploader — built, untested in production

Fork of Document Uploader `2026.09.15-7`. Four things differ and everything else is
inherited: a folded sheet carries **four** pages, so saddle stitch is
`ceil(pages / 4)` where coil, wire and perfect stay at `ceil(pages / 2)`, and the
sheet mode comes from the chosen **binding**, so binding feeds the price; only
bindings compatible with the detected page count are offered; the divisible-by-four
rule **pads, it never rejects**, and the padded count is what is printed and priced;
and the preview shows reading order, spreads and the binding drawn over the spread.

**Never impose or reorder pages.** The RIP does imposition; the tool supplies the
flat PDF in reading order and states the page count. Adds `bu_binding` to the
Document Uploader option set.

### Fan Face — not ready

Head cut-out on a stick. Ported from the Sticker Designer with MediaPipe selfie
segmentation and face detection added for the head clip, neck line and radius
guarantee. Built and delivered 30 Jul 2026, **and not sellable as it stands: the head
cut needs real AI segmentation, not MediaPipe.** Treat it as blocked on that, not as
a tool awaiting an install.

The two legacy MediaPipe solutions must be initialised strictly one after the other —
constructing the second before the first has finished aborts with
`Module.arguments has been replaced with plain arguments_`.

### Flyer / Brochure — never finished

Takes a PDF, PNG or JPG for a flat or folded piece, reads the fold and size from the
product's own variants and keeps both in sync, computes panel widths with the shared
fold engine, and renders the fold in 3D. Writes the artwork, the geometry and the
artwork-check verdict to the orderline.

### Design Brief — unfinished

The Design Service wizard: collects the brief, drives the customer gallery, and
writes the whole thing onto the orderline. The book itself is not priced in the tool
— the customer buys it after approving the design, so everything captured is a
preference for the designer, not a locked specification.

### Film Order Builder — in progress

Film processing order builder: formats, services, scan resolutions and returns
assembled into one order. **The rate card in the current asset is demo data**, blended
from two real customer exports for the prototype. It is not a Pixfizz price list and
must not be quoted to a client. Respecced 16 Sep 2026.

### Restoration Estimator — specified, not started

Instant restoration estimate: the customer uploads a photo of the photo, the tool
measures what it can in the browser and returns a price band, a turnaround date and
an honest list of what restoration cannot fix. Tier 1 (browser only) ships without
any server; the vision-model tier needs a Pixfizz-owned proxy to hold the API key,
which is an open decision. **A vision API key may never ship to the browser.**

### Acrylic Statue Designer — specified, not started

Flat acrylic figure, UV printed and laser cut to a contour, standing in a separate
base. The new work over Fan Face is **support geometry**: a figure whose silhouette is
too narrow at the bottom cannot stand, so the die adds a clear support region and a
tab. A tool that only traces the silhouette produces a piece that falls over.

### Collage Wall — unfinished

### Custom Sized Prints — just started

---

## 8. Definition of done

A tool is not finished until all of this is true. Each item has cost time at least
once.

- **One real order placed end to end.** Every defect found on this platform so far
  survived every check short of that.
- **The print file measured on a real order** — format, colour mode, alpha, pixel
  dimensions, embedded DPI. None of them are visible in a proof.
- DPI read from the XML definition, never hardcoded.
- Configuration resolved in the browser with source markers and `data-product-seen`.
- The shared notify layer consumed, never reimplemented.
- Dependencies loaded from the **product snippet**, not from
  `integrations/custom-body-scripts` or the layout, both of which child sites
  routinely override, and injected with a `data-<tool>-src` marker so a re-render
  cannot double-load.
- The preview code added to every surface in §6 that still carries a list.
- A `VERSION` constant logged at init.
- The registry row in this file written or updated.
- Rendered against every orderline shape — this product, the other tools, an ordinary
  project product, empty options, nil `template_option` — with **byte-identical
  output for everything that is not this product**.

---

## Changelog

- 2026-09-19: File created. The custom design tool estate had no home in this
  knowledge base: no list of which tools exist, no per-tool reference, and the
  install order in three other files still taught the retired product-custom-field
  cascade. Registry status supplied by Alex; versions verified by reading asset
  source. Sources: `claude/CUSTOM_TOOL_CONFIG_ONE_SOURCE.md`,
  `claude/CUSTOM_TOOL_CONFIG_MUST_TRAVEL.md`, `claude/CUSTOM_TOOL_DOC_SET_SPEC.md`,
  `claude/ORDERLINE_PREVIEW_SURFACES.md`, the per-tool build specs, and the tool
  assets themselves.
- 2026-09-24: Added the dedicated hidden mount option standard (`<prefix>_mount`) and the counter test; widened "Shopper choice" to every customer selection and commercial value. Source: claude-chat, fireflies-call.
