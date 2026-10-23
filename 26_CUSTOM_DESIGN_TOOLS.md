# 26 — Custom Design Tools

**Authority Scope:** custom design tools as a class — what they are, the estate that
exists, how one is configured, mounted, installed and verified. Per-tool behaviour
and current status.

**Not in scope:** the standard Design Tool (the platform editor) — that is
`17_DESIGN_TOOL.md`. The browser-side engineering rules shared by every tool stay in
`17_DESIGN_TOOL.md` § Custom Design Tools — Browser-Side Rules; this file points at
them rather than repeating them.

_Last updated: 2026-10-22_

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

### Price ladders read the platform price

Where the platform *can* price the product (variants and a product formula), a tool that
shows a quantity ladder reads the platform's price and never computes one. Platform-level.
*Verified by query on a client's live products, 3 Oct 2026.*

- Endpoint, the same one `px-product-price` uses:
  `GET /v1/products/<product id>/price_forecast.json?print_theme_id=<design id>&variants[<CODE>]=<value>&...&quantity=<n>`
- Response: `{ parameters, price, variants_applied: { <CODE>: amount }, template_options_applied }`.
  `price` is the **unit** price; `px-product-price` displays `price * quantity` unless
  `unit-price="true"`.
- Build the query with the element's own `priceQueryParameters()` (it skips invalid and
  non-pricing options and hidden triggered children), then override `quantity`. Fallback:
  checked, enabled `variants[...]` radios plus `print_theme_id`. The endpoint accepts
  `template_options[code]` parameters exactly like `variants[code]`, and
  `priceQueryParameters()` already includes them (see § 4, stores that price with template
  options).
- Variant amounts are additive to the base unit price. Amounts with `/quantity` in their
  formula (setups, lamination) change per quantity; the rest do not.
- About 160 ms per call; ten quantities in parallel are fine. Cache per variant set.
  Same-origin only; a collection custom field page can call it too.
- A ladder that shows `round(price * qty, 2)` matches the cart, because the cart line total
  rounds the product, not the unit. See `30_PRICING_ENGINE.md`.
- **Books counted in spreads take system pages.** On a book product with the product custom
  field `spreads_as_pages` on, `product/design-now` shows printed pages but submits system
  pages: the `book[pages]` option value is the count and its label is `(count - 1) * 2`
  (label 16 = value 9). The pricing formula and `price_forecast.json?pages=N` both work in
  system pages. A tool that prices a book by page count reads `custom.spreads_as_pages` from
  `/v1/products/<id>.json` and converts: system = printed / 2 + 1. `pages` alone is enough for
  a product priced by formula with no paper option. Template-level (`spreads_as_pages`) and
  platform-level (`pages`). *Verified by reading a live page and by query, 2026-10-21.*

### When the tool computes the price

Where the tool computes the price from a rate card (the uploaders), the tool writes one figure
to a number variant whose formula is `value`. **The rate card lives in the product custom field
`px_rates`** (Corrected 2026-10-13: it previously rode in the mount), because OrderHub and the
point of sale read product custom fields but not template options, and must price from the same
numbers as the website. Decided by Pixfizz, 2026-10-13.

- `px_rates` is a **snippet-type** product custom field (text fields cap at 256 characters,
  `51_CUSTOM_FIELDS_REFERENCE.md`). One fixed name on every site and every tool; the card's
  `mode` key names the pricing engine. Description: "Price card (JSON) for the custom design
  tool on this product. Read by the tool and by OrderHub. Leave empty on products without a
  tool."
- The definition must exist on each site (Product object) before a value can be stored; it
  belongs in the shopper24 custom fields kit. The mount stays: it switches the tool on and holds
  display configuration only.
- The tool cannot read `product` inside its mount, so the parent snippet `product/design-now`
  prints the field as `<script type="application/json" data-px-rates data-px-source="product">`
  when it is not blank. Tools read that tag first and fall back to a card in the mount. A
  `px_rates` value that is not valid JSON disables pricing; the tool never falls back to the
  mount card in that case.
- A snippet-type field prints byte-identical through `{{ product.custom.px_rates }}`.
  *Verified by query, 2026-10-13.*
- **Mode `area`** prices anything sold by size: graduated or whole tiers per stock, a minimum
  per item, and extras priced each, per area, per linear foot or per grommet, in `sq_in` or
  `sq_ft`. Its engine file is `pricing-area.js` (version 1). The tool writes the size and
  choices to generic variants without a tool prefix (`px_width_in`, `px_height_in`,
  `px_stock`, `px_price`, plus `px_hanging`, `px_depth`, `px_edge`, `px_edgecolor`,
  `px_grommets`), and OrderHub reprices an order line from those values. A counter-only
  product (not published) can carry the same card so staff price a custom size in OrderHub
  from width and height. This replaces per-stock area Number variants with range-hash Ruby
  formulas, which OrderHub cannot evaluate. The three pricing types side by side are in
  `30_PRICING_ENGINE.md`. Shopper standard. *Stated in the tool build, 2026-10-22; OrderHub
  repricing not verified here.*

**One price engine for the tool, OrderHub and the POS: `pricing.js`** (standard agreed with
OrderHub 2026-10-13 and 2026-10-14; not verified, the first tool on it is still a prototype).
- The engine is its own file, plain JS with no dependencies and no DOM, running in the browser
  and in Node. Entry point `PxPricing.price(card, selections)` returns
  `{ unit, total, qty, breakdown }`; `card` is the `px_rates` JSON. The tool bundles the file
  unchanged; OrderHub and the POS load the same file.
- `PxPricing.version` is a whole number starting at 1. Bump it only when a change could change a
  price for the same card and choices; a rate change is a `px_rates` edit, never a version bump.
  Every version is kept, so an order reprices on the version that sold it.
- The card declares the minimum engine version it needs (`"engine": n`, missing = 1). A tool or
  OrderHub holding a lower version refuses to price rather than guess.
- OrderHub and the POS price from variants (custom and POS orders carry variants only), so the
  tool writes the computed price to `px_price`
  (number variant, formula `value`) and the choices as text variants (`px_size`, `px_paper`,
  `px_sides`, ...), with `pos_hidden` on anything the POS should not show. Because variant
  values are customer-facing labels, `pricing.js` owns both directions: it writes the variant
  values and reads them back to reprice, so the tool and OrderHub cannot drift.

When a lab supplies its own Ruby pricing formulas for such a product, keep its
tables unchanged in the rate card and run the formulas as an **acceptance oracle**: every
selection must match to the cent. Tool-level. *Verified by test, 5 Oct 2026.*

- Compute each term in the same order as the formula (`run * sheets * rate / units`), sum,
  then round in two steps: `Math.round(Number(sum.toFixed(9)) * 100)` in JS,
  `(sum.round(9) * 100).round` on the Ruby side.
- Why: Ruby `Array#sum` uses compensated (Kahan-Babuska) summation and JS `+` does not. Without
  the 9-decimal step, 218 of 97,200 booklet cases differed by one cent on half-cent sums such
  as 12.495.
- How the platform itself rounds a Ruby pricing field is **not verified**.

---

## 2. The estate — registry

Every tool currently in existence. **Status was recorded by Alex on 19 September 2026
and brought up to date on 6 October 2026** from the tool build documents, the 29 September
install sweep (below) and live queries on shopper24 and experience.pixfizz.com between 3 and
6 October. Versions marked "on shopper24" were read from the parent asset (verified by query);
the rest are from the asset source or the build document. A tool absent from this table does
not exist.

| Tool | Asset | Version | Prefix | Mount option | Status |
|---|---|---|---|---|---|
| Sticker Designer | `sticker-designer.js` | 5.30.0 on shopper24 (banner `2026.09.15-1`, 6 Oct) | `sticker_` / `stk_` args | `sticker_artwork` | **Live** on several client sites and on experience.pixfizz.com. Needs updates (§7). |
| Gang Up (DTF gang sheet) | `gang-up.js` | 1.2.0 on shopper24 (5 Oct) | `gangup_` / `gu_` args | `gangup_artwork` | **Live.** Auto-trim on every site; cut path built, off unless a site's mount turns it on. |
| Business Cards | `business-card.js` | 1.0.1 on shopper24 (`@version 2026.09.15-2`, 6 Oct); 2.0.0 built for the recall | `bc_` | `bc_artwork_front` (live installs); `bc_mount` from 2.0.0 | **Live** on 16 templates across 12 client sites, plus experience.pixfizz.com. In-place recall to the variant standard prepared 29 Sep. |
| Cover Studio | `cover-studio.js` | `1.8.0` | `cvs_` | template option per install (`cvs_mount` planned) | **In test.** Installed on two client album and layflat ranges. |
| Publication Upload | `publication-upload.js` | 1.13.2 | `pu_` | `pu_text` (`pu_mount` planned) | Built. Installed on two client sites (publication programs). Production status not restated since 19 Sep. |
| Document Uploader | `document-uploader.js` | 1.3.2 on shopper24 (6 Oct) | `du_` | `du_mount` | **Live** on two client sites (go-live 27 Sep; a third site reported, not verified) and on experience.pixfizz.com, test order placed 6 Oct. To be replaced by Uploader 2.0. |
| Booklet Uploader | `booklet-uploader.js` | 1.5.1 on shopper24 (6 Oct) | `bu_` | `bu_mount` | **Live** on one client site and on experience.pixfizz.com, test order placed 6 Oct. Fork of Document Uploader; to be merged into Uploader 2.0. |
| Uploader 2.0 (documents, booklets, forms) | one asset replacing the two uploaders | prototype (5 to 6 Oct) | `px_` codes, `tool=uploader` in `px_spec` | reads `du_mount` and `bu_mount` | **Prototype.** Modes `flat`, `booklet`, `forms`. Nothing built on the platform. |
| Flyer / Brochure (flyer-fold) | `flyer-fold.js` | 0.5.2 on shopper24 (3 Oct); 0.6.0 in progress | `flyer_` / `fld_` args | one mount per template | **Live on one client site** (hidden pages). Not yet proven on a second site (§7). Demo template on experience.pixfizz.com. |
| Fan Face | `face-fan-designer.js` | delivered 30 Jul 2026 | `facefan_` | `facefan_artwork` | **Not ready.** Head cut needs real AI segmentation to be sellable. Installed on two client sites (29 Sep sweep). |
| Design Brief (Design Service) | `design-brief.js` | `1.0.0` | `dsn_` | `dsn_mount` | **Unfinished.** Demo install on experience.pixfizz.com. |
| Film Order Builder | `film-builder.js` | prototype, no version | `pxfb_` | host element `#pxfb` | **In progress** (status of 19 Sep). Rate card in the asset is demo data. |
| Custom Framing | not recorded | - | - | - | Demo install on experience.pixfizz.com only (29 Sep sweep). |
| Wall Designer | not built | prototypes 0.2.1 (metal and acrylic) and 0.3.0 (magnetic wall tiles) | - | - | **Prototype**, 6 Oct. Baseline test template built 26 Sep. Nothing installed. |
| Board Engraver | not built (`board-engraver`) | prototype 0.0.1 (27 Sep) | `be_` | `be_mount` (proposed) | **Prototype.** Awaiting review. |
| Custom Sized Prints | not recorded | 0.2.1 stated as shipped (22 Oct); 0.3.0 room scenes in local render | `cp-` classes; generic `px_` codes | not recorded | **In build.** Priced by `px_rates` mode `area` (§ 1). Where it is installed is not stated in the sources (22 Oct). |
| Restoration Estimator | not built | - | `rst_` | `rst_original` | **Specified 4 Aug 2026, never started.** Needs revisiting. |
| Acrylic Statue Designer | not built | - | `astat_` | `astat_artwork` | **Specified 21 Aug 2026, never started.** Needs revisiting. |
| Collage Wall | not built | - | - | - | **Unfinished.** Its template emitter is the reference shape for the Wall Designer baseline test. |

**Version banners are not reliable on every tool.** Three cases found on 19 Sep:
Sticker Designer's header banner still read `2026.08.26-11` while the asset was
v5.30; Cover Studio's file is named `cover-studio-v1.7.js` and its `VERSION`
constant reads `1.8.0`; Business Cards and Gang Up carried a `@version` banner but no
`VERSION` constant, so the deployed build could not be confirmed from the console.
Since then Gang Up 1.2.0 exposes `VERSION` as `window.GangUp.version` (verified by query,
5 Oct; 1.1.0 had bumped the constant and left `window.GangUp.version` at a date build, so
grep the asset for every exported version string when bumping), and Business Cards 2.0.0
carries a semver `VERSION` constant, but the live parent still runs 1.0.1 without one.
**Every tool should expose a `VERSION` constant and log it at init**, because an
asset is aggressively browser-cached and "did my change deploy" is otherwise
unanswerable. Document Uploader, Booklet Uploader, Publication Upload, Cover Studio
and Design Brief do this correctly.

### How the install list is found

On 29 Sep 2026 every Shopper site was swept from the super admin (verified by query): of 1,857
websites, 912 have products and 182 are Shopper sites (their `cms_snippets` list
`admin/checklist/*` or `shopper/*`). Every product and template on those 182 sites was read,
and the template export of every template whose product name or code matches a tool keyword,
and of every template id from 46000 up, was searched for `snippet 'product/<tool>'` inside
`custom_script`. **Limit:** a tool mounted on an older template whose product name matches no
keyword would be missed; none is known. Several subdomains can share one product list; each
such group is one install. Demo and test sites (experience, baseline, training and backups)
are not counted as installs.

### Related, but not custom design tools

- **Product Previews: Live Finish and 3D Preview** (`live-finish.js`, `preview-3d.js`; mounts
  `lf_mount`, `p3d_mount`) show the finished product beside the editor and write nothing to the
  orderline. Their own category, mounted and configured like a tool but not a tool. See
  `27_LIVE_FINISH_AND_3D_PREVIEWS.md`.
- **Photo Prints review modal (`prints-review.js`, prefix `prv-`)**: a parent asset
  that adds a review step to the platform's own photo prints flow. It mounts from
  `product/prints-review` and is gated on `admin/checklist/prv-enabled`, not on a
  `custom_script` template option. See `claude/PRINTS_REVIEW_MODAL_REFERENCE.md`.
- **1 Hour Frame (framing)**: shipped as eight templates and products with baked-in
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

**Tool assets: one stem per tool, a new asset per version.** The asset edit screen
(`admin/assets/<id>/edit`) has an Upload field that replaces the file in place, keeps the name
and gives no warning when the new file is a different tool (`61_PIXFIZZ_API.md` § 13c
*Assets*). A wrong pick there once replaced one live tool's asset with another tool's file, and
the product page rendered an empty root with no console error. *Incident verified by query,
2026-10-14; rules set after it.*
- Every tool asset name is unique to the tool and versioned (`<tool>-<major>-<minor>-<patch>.js`).
  Two tools never share a stem.
- A new version is always a **new** asset, then the mount or page script tag is switched to it.
  Use Upload on an existing asset only to roll back that same tool.
- After any asset upload on the parent, load every page that uses a file with that stem and
  check the served file names the right tool and version, and that the tool root has children
  (an HTTP 200 is not proof).
- Before a demo or webinar, run the link check after the last change, not before it.

### Open the platform upload dialog standalone

**Every custom tool that takes a customer file (photo, artwork, logo) must open the platform
dialog.** Never a tool-built file input, gallery picker or QR flow, so the shopper always gets the
same sources as the store's own upload buttons (This Device, My Galleries, phone by QR, and
whatever else the site enables). (Corrected 2026-10-22: previously "should open".) A plain
`<input type=file>` is only the fallback when `customElements.get('px-upload-dialog')` is
undefined on the page (a test harness), and it logs a `console.error`. Dropping a file on the
tool is a shortcut beside the dialog, not a replacement. Sources `local galleries`, plus `qr`
when `(min-width:768px) and (pointer:fine)`; `multiple` for several files;
`login-modal-enabled`. A local file is read from the capture-phase `change` and used at once;
a gallery or phone pick is fetched from `result.url` on `upload-success`, never from
`preview`. *Decided by Pixfizz, 2026-10-04, restated 2026-10-19 after two tools shipped their
own pickers.* Platform-level component; button classes and wording are template-level
(Shopper 24). *Verified by query, 2026-10-04, bundle 20261002143509, and on
experience.pixfizz.com, 2026-10-19; Google Photos behavior not verified.*

**A tool opens calm:** at most one sample on load, and only the first step open before the
customer's first file. *Decided by Pixfizz, 2026-10-19.*

- How: `const d = document.createElement('px-upload-dialog'); d.setAttribute('multiple',''); d.setAttribute('sources','local galleries qr'); d.setAttribute('gallery-id', gid); /* button classes and text attributes, below */ document.body.appendChild(d);`
  Listen for `upload-success`, `close` and `cancel`, and remove the element when done.
  **Without `sources` the dialog shows This Device only** (verified live, 2026-10-19). The
  platform's own `px-image-upload` builds the dialog the same way and copies `sources`,
  `max-size`, `gallery-id`, `dropbox-app-key`, `google-client-id` and every `upload-dialog-*`
  attribute onto it (verified by reading the live component source).
  It mounts and opens itself as a native modal dialog in the top layer (see the z-index
  gotcha in `50_SHOPPER_TEMPLATE_REFERENCE.md` § 17). Its markup is light DOM
  (`dialog.px-upload-dialog`, `.px-ud-*`), so a class on the element plus scoped CSS styles it.
- Attributes: `multiple`, `sources` (space-separated: `local`, `galleries`, `url`, `camera`,
  `qr`, `dropbox`, `google_photos`), `max-size`, `max-files`, `gallery-id`, the four
  `login-modal-*` attributes (`17_DESIGN_TOOL.md` § Login inside the upload dialog),
  `dropbox-app-key`, `google-client-id`, the text labels, and the button classes.
- **Pass the button classes, or the dialog looks unstyled.** Without them every source button
  renders as `class=""`, a browser-default grey button. The standard Shopper upload passes them
  through `px-image-upload` as `upload-dialog-*` attributes, copied onto the dialog with the
  prefix stripped. To match Shopper 24 set directly: `button-class="btn btn-xs"`,
  `source-button-class="btn-outline-secondary"`, `local-source-button-class="btn-primary"`,
  `confirm-button-class="btn-secondary"`,
  `picker-load-more-button-class="btn-xxs btn-outline-secondary"`,
  `google-photos-picker-link-button-class="btn-primary"`, plus the text attributes
  `main-panel-title`, `upload-info-text`, `source-button-local-text`, `galleries-picker-title`,
  `close-button-title`, `back-button-title` and `qr-picker-info-text`. All class attributes the
  dialog reads: `button-class`, `source-button-class`, `local-` / `galleries-` / `url-` /
  `camera-source-button-class`, `picker-load-more-button-class`, `confirm-button-class`,
  `select-all-button-class`, `ios-picker-notice-button-class`, `login-link-button-class`,
  `login-button-class`, `google-photos-picker-link-button-class`. *Verified by reading source
  (cms_bundle.js) and live, 2026-10-14.*
- **The dialog has no button look of its own.** `cms_bundle.css` gives `.px-upload-dialog`
  layout only; the source, close and footer buttons get their look only from the class
  attributes the caller passes, read into `data.button_classes`. Without them they are
  browser-default grey square buttons on every theme. A site whose `style/custom.css` styles
  `px-upload-dialog dialog button[class=""]` hides the gap on that site only. *Verified by
  reading cms_bundle 20261021074014 and by query on two live sites, 2026-10-21 and
  2026-10-22.*
- **The classes are read once, when the element connects.** Setting the attributes after
  `appendChild`, or patching the class prototype's `connectedCallback`, changes nothing
  (custom element callbacks are captured at define time). Set every attribute before
  `appendChild`; after the fact only a fix on the rendered light-DOM buttons works.
  Platform-level. *Verified by query, bundle 20261021152157, 2026-10-22.*
- **The canonical class set is the standard Shopper upload's** (`product/px-option`): the
  `btn btn-xs` / `btn-outline-secondary` / `btn-primary` / `btn-secondary` /
  `btn-xxs btn-outline-secondary` / `btn-primary` set above. On a dark child site these
  render correctly through the site's own `.btn` rules. Template-level (Shopper 24).
  *Verified by reading source and on a dark child site, 2026-10-22.* Every tool sets the full
  set, or styles its own scoped dialog class (`px-upload-dialog.<prefix>-pud`) completely.
  A repo check catches the miss: any source file containing `px-upload-dialog` must also
  contain `local-source-button-class`. Proposed for the parent and not built: a
  `helpers/px-upload-dialog-defaults` snippet, included in `layouts/index` before
  `integrations/custom-body-scripts`, whose MutationObserver gives any unclassed dialog button
  the Shopper classes and logs a console warning (a safety net only; tested by injection on
  one child site, 2026-10-22).
- **File types: This Device accepts a fixed list, `SUPPORTED_FILE_TYPES`:** `image/png`,
  `image/jpeg`, `image/webp`, `image/heic`, `image/heif`, `image/tiff`, `application/pdf`
  (`.png .jpg .jpeg .webp .heic .tif .tiff .pdf`). The dialog writes it straight into its file
  input; **`px-upload-dialog` has no `accept` attribute** (`accept` belongs to
  `px-image-upload` and `px-file-upload`). The phone QR page (`/upload/qr_uploader`) and
  Dropbox use the same list. A 2-page PDF uploaded through the dialog gave an id, page 1's
  size at 300 dpi as `width` / `height`, the original PDF as `url` and WebP thumbnails.
  Platform-level. *Verified by reading source and by query on baseline.pixfizz.com, bundle
  20261019115227, 2026-10-19; a PDF sent from a real phone through the QR code, and how an
  uploaded PDF shows in My Galleries, not verified.*
- **The default wording says photos.** `main-panel-title` "Upload Photos",
  `qr-picker-info-text` "Scan QR code to upload images from your phone:",
  `ios-picker-notice-title`, `ios-picker-notice-text` and `ios-picker-notice-button-text`
  ("Choose photos"). The dialog has no drop zone and no hint text; the local source is a
  button (`source-button-local-text`, default "This Device"). **A print-artwork tool sets all
  five text attributes in print wording** and leads with PDF: title "Upload your artwork",
  local choice "Choose your PDF", and "PDF recommended. PNG or JPG also accepted." on its own
  upload card and header hint, with no photo wording anywhere. Because the dialog lets every
  listed type through, the tool turns away in its own code the types the product cannot print
  (HEIC and TIFF for print artwork) with a message that names PDF first. *Decided by Pixfizz,
  2026-10-19.* Proposed to the platform, not built: an `accept` attribute on
  `px-upload-dialog` falling back to `SUPPORTED_FILE_TYPES`, and a hint label under the local
  source button.
- Events on the element: `upload-start`, `upload-progress`, `upload-success` (once per file;
  `detail.changedFile.result` = `{id,width,height,filename,preview,url,thumbnails}`),
  `upload-error`, `close`, `cancel`. The dialog closes itself when done. On `upload-success` the
  platform's own upload displays the last entry of `thumbnails[]` and stores `db:<id>` as the
  option value.
- `upload-progress` detail is `{files, changedFile}` with no percentage field: show an
  indeterminate state. (`changedFile.progress` as `{loaded,total}` was read from source and not
  tested.)
- **Closing the inner `<dialog>` right after the pick does not stop the upload:**
  `upload-progress` and `upload-success` still fire, then the element emits `close` and removes
  itself. A tool can show the photo at once from the File (capture-phase `change`, below) and let
  the upload finish in the background. *Verified by query, 2026-10-14.*
- **`result.preview` is a 250 px thumbnail** (`.../thumbnail/250/~/<file>`, 250 px on the **long
  edge**); `thumbnails` lists 150, 250, 600, 900 and 1200. Any size works by rewriting the number
  (`/thumbnail/2400/` gave 2400x600 from a 6000x1500 upload). A tool that draws the photo large
  on screen (wall previews, Live Finish, close-ups) must not draw from `preview`: use a working
  copy at long edge min(2400, photo long edge). Print files use `url`. *Verified by query,
  2026-10-14.* `url` and `preview` are on the site's own host (`https://<site>/fz/.../~/<name>`),
  so the full image draws to a canvas without tainting it and can be fetched same-origin: a
  tool can cut print files from `url` in the browser. A generated file (a print file) is
  uploaded without the dialog by POSTing FormData `data` to `/upload/image` with the CSRF
  token; the response carries the same fields. *Verified by query on experience.pixfizz.com,
  2026-10-19.*
- `gallery-id` routes uploads into that gallery; without it, uploads go to a hidden uploads
  area. Picks from My Galleries return the existing image id and are **not** copied into
  `gallery-id`: copy them server-side (`61_PIXFIZZ_API.md`, `POST /upload/image?gallery_id=`).
- To read EXIF or other local file data, add a capture-phase `change` listener on the dialog
  element; it receives the real File objects.
- Google Photos did not show with `google_photos` in `sources` and no `google-client-id` (not
  verified why).
- **Sign-in inside the dialog** (`login-modal-enabled`, verified by reading source): the form
  under My Galleries shows ONLY when `/v1/galleries/_mine.json` is empty and
  `/v1/users/me.json` says anonymous, so a guest who already owns a gallery sees it and gets no
  sign-in form. The form posts `/v1/session.json` `{email,password}` as JSON, then polls
  `/v1/users/me.json` every second until signed in, then loads galleries, with no page reload.
  Forgotten-password and registration links open in a new tab, and the poll notices a sign-in
  done there. With `login-modal-external-url` set it shows a link to that URL instead of the
  form. A guest who signs in keeps projects and cart lines (`20_SHOPPER_CART_RULES.md`). For
  sign-in outside the dialog see `50_SHOPPER_TEMPLATE_REFERENCE.md` § 1 (`#modalLoginCheckout`).
- **Feeding files to the dialog in a test:** set `files` on the open dialog's
  `input[type=file]` from a `DataTransfer` and dispatch `change`; the files upload exactly
  like a user pick (12 to 14 canvas-generated JPEGs in one run). Useful for hand-off tests
  without real photos. Platform-level. *Verified by query, 2026-10-19.*

**Upload dialog check before a studio install is done** (decided by Pixfizz, 2026-10-22):
This Device, My Galleries and the phone QR all show; the buttons carry `btn` classes with
readable contrast; the layout is right at 375 and 1280 px, on a light and a dark theme; and
upload progress is visible near the dialog or the tool.

### Attaching a file to an option: `file-url` is content addressed

Uploading the same bytes twice through a `px-file-upload` sets the **same** `file-url` both
times (`/fz/<hash>/0/<token>/~/<name>`); the attribute does not change on the second upload.
Platform-level. *Verified by query on experience.pixfizz.com, 2026-10-22.*

- So a tool that detects "upload done" by waiting for `file-url` to differ from its value
  before the upload **hangs** when the customer re-uploads an identical file, or when the tool
  rebuilds a byte-identical print file. One uploader waited out two 300 s timeouts and read as
  "froze in review".
- Fix: remove the host's `file-url` attribute before injecting the file, then wait for it to
  become non-empty. Proven red and green against a content-hash stub (30 s timeout before the
  fix, about 1 s after). Every tool with its own file injection needs the same check.
- **Edit from cart** (project-edit, § 6): restore from the original file's `file-url` and the
  `px_spec` / notes input values before the first write, then submit with
  `.save-to-cart-button`. *Confirmed in a tool build, 2026-10-22; not verified on a live
  order.*

### Using the platform crop tool standalone

The photo prints crop is the platform custom element `<px-image-adjust-tool>` in
`cms_bundle.js` (not Cropper.js; the editor bundle has its own crop and does not include it). A
tool can open it itself, following `px-image-upload`. Platform-level (CMS bundle). *Verified by
reading source, cms_bundle 20261007100426.*
- Create the element with `value`, `img-src`, optional `img-width` / `img-height`,
  `crop-aspect-ratio` (for example `8in/10in`; without it no crop box shows), `minimum-dpi`,
  `shrink-to-fit`, `image-filters` and text overrides. Put it in a
  `div.px-image-upload-crop-modal-overlay` prepended to `body`. Listen for `confirm` (read
  `el.value`) and `cancel`.
- The output is a string, not a box: `db:<imageId>@{l:..,t:..,z:..,r:..,crop:0,...}`, zero
  values omitted. It needs the platform image id (`db:`), not a URL.
- The bundle is minified and unversioned: check the element is defined, wrap the call, and fall
  back to the tool's own UI.
- Attributes: `22_OPTION_VARIANT_RENDERING.md` § 5.1. Not verified: that the prints flow's
  `l` / `t` / `zoom` values equal the editor's units on a book image.

### The fulfillment contract: one set of `px_` codes (decided 4 Oct 2026)

Every custom design tool hands its production files to fulfillment the same way, so an order
that mixes a sticker, a brochure and business cards goes out correctly on any site, whatever
its fulfillment route. Every tool writes the **same generic codes**, so a new tool never
needs a change to any site's fulfillment files. Decided by Alex, 3 and 4 Oct 2026. Live Finish
and preview-only extensions are out of scope: they make no production file.

| What | Where the tool writes it | Code |
|---|---|---|
| Production file (what gets printed) | template option, type file upload | `px_print_file` |
| Second production file on the same line | template option, type file upload | `px_print_<part>`, for example `px_print_cut`, `px_print_cover` |
| Cart thumbnail | template option, type file upload | `px_preview` |
| Customer's original upload, if kept | template option, type file upload | any other code (the tool's own, for example `sticker_original`) |
| Job record | template option, type text | `px_spec`, starting `tool=<prefix>; v=<version>;` |

1. **The production file always goes into `px_print_file`**, including a pass-through tool
   that prints the customer's PDF unchanged. The `_print_` token decides the routing.
2. A second production file uses `px_print_<part>`; the site files add the part to the file
   name, so the files never collide.
3. Which tool made a line is read from `px_spec` (`tool=`) or the product code, never from the
   option code.
4. The same code on many templates is fine: option codes are per template, so a site-wide
   clash cannot happen (verified by query on baseline: `du_file` sits on four templates).
5. Every set in the template XML carries `fulfillment="false"`; the `px_print_` option is the
   only production source (see `31_FULFILLMENT_ENGINE.md` § Where No Production File Is
   Rendered At All).
6. `px_spec` uses one format in every tool: `key=value; key=value`, keys `tool`, `v`, `size`,
   `fold`, `paper`, `finish`, `sides`, `pages`, `qty`, `approved` (plus tool-specific keys such
   as `mode`, `fit`, `cover_mode`).
7. No tool adds a custom field for fulfillment. Everything travels on template options in the
   template export.

**Old codes.** New installs use the `px_` codes from each tool's next release. Sites already
live keep their old codes (`du_file`, `bu_file`, `sticker_artwork`, `sticker_cutfile`,
`gangup_artwork`, `pu_text`, `flyer_file`, ...) until that site's templates are updated with
the bulk updater, and the site's fulfillment files list them meanwhile. **Never rename live
codes across sites in one pass:** an option code is platform data outside the tar. Where an
old code also carries the mount (`sticker_artwork`, `gangup_artwork`, `pu_text`), the rename
moves the mount and waits for a planned per-site update. The site side (FTP, OrderHub Desktop,
HTTP push files, and the legacy list) is in `31_FULFILLMENT_ENGINE.md` § Custom Tool
Fulfillment Standard.

### The shared look: `style/px-tool-theme`

Tools take the site's look through one shared contract, `style/px-tool-theme`, with `--pxt-*`
tokens declared in `:where(.pxt)`, and a per-site retoken through `.pxt-<tool>` in the site's
`style/custom.css` override. A tool ships no palette and no fonts of its own (body text uses
`--pxt-font-body`). Template-level (Shopper).

**Dark-theme sites break contrast.** `--pxt-ink` maps from `style/color-font` and
`--pxt-ink-muted` from `style/color-font-secondary`, while `--pxt-surface` is tool-owned
`#ffffff`. On a dark-theme site the font color is light, so tool text and pressed chips come
out light on white and are unreadable. *Verified by query on a dark-theme child site,
30 Sep 2026.* Check `style/color-font` before installing any `pxt` tool on a dark site.

**Two contract class names are taken.** `style/px-tool-theme` defines `.pxt-step` (an 18 px
numbered circle) and `.pxt-shell` (the modal shell, with radius and shadow) on `:where(.pxt)`.
A tool that uses either name for its own layout inherits those rules and its panel collapses.
Use the tool prefix for layout classes (`fan-step`, `fan-shell`); use `.pxt-*` only for the
contract components as the contract defines them. A local harness must load the real
`style/px-tool-theme`, not a stub. *Found 2026-10-06 and 2026-10-08.* The other DOM rules a
mounted tool lives under are in `17_DESIGN_TOOL.md` § A tool mounted by `custom_script` lives
inside the variant selector.

**Retoken every `.pxt` element, not the tool root.** A tool can put `.pxt` on several separate
elements, and each re-declares the tokens through `:where(.pxt)`, so an override on one never
inherits into another. Target them with a descendant selector. The site fix used for a
gallery-mounted tool:

```css
/* ===== START: Live Finish ink on dark theme ===== */
.px-product-gallery .pxt {
	--pxt-ink: #191c1d;
	--pxt-ink-muted: #667085;
}
/* ===== END: Live Finish ink on dark theme ===== */
```

When verifying computed colors, controls that transition `color` and `background` do not
progress in a hidden or background browser pane, so `getComputedStyle` keeps the old value:
inject `transition: none !important` before reading. A parent-level fix (stop mapping ink from
the page font colors, or only on light sites) is a candidate and not done.

**Text a tool draws outside its own surfaces sits on the page background.** A tool header
(studio label, moved H1, intro line) rendered on the page rather than on a tool card keeps the
tool ink and becomes invisible on a dark site (contrast about 1.0). Either give that text the
store text color or make the whole tool dark. **Full dark mode is one site block of tokens:**
override `--pxt-ink`, `--pxt-ink-muted`, `--pxt-ink-faint`, `--pxt-surface`, `--pxt-canvas`,
`--pxt-stage`, `--pxt-line`, `--pxt-line-strong`, `--pxt-secondary`, `--pxt-on-secondary`,
`--pxt-accent-soft` and the state background/ink pairs, on a selector that also matches dialogs
the tool reparents to `body` (for example `html body .pxt[class*="<tool prefix>-"]`). Then run a
contrast scan: primary-colored text on `--pxt-primary-tint` was the one pair that failed and
needs a lighter tint of the brand color. *Verified by query (computed style and contrast scan),
2026-10-15.*

**Write a tool's element resets with `:where()`.** `.<tool> button { font: inherit }` has
specificity 0,1,1 and beats the tool's own class rules (0,1,0), so every button renders at the
body size. Write it as `.<tool> :where(button, input) { font: inherit; }` (0,1,0), placed above
the class rules, so later class rules win while a theme's bare `button {}` (0,0,1) still loses.
*Verified by test (computed font size), 2026-10-14.*

**Defend against the site's `style/custom.css`.** A child site's own stylesheet reaches every
component on its pages: tools, previews, landing pages and homepage embeds. Three cases seen
on live child sites, October 2026 (verified by query and computed style):
- A global link rule with high specificity, for example
  `html body a:not(.btn):not([class*="..."]) { color: ... }` (0,3,3) on a dark site, beat a
  tool's light pill button `.<tool> .<tool>-btn` (0,2,0): white text on a white button.
  Scope button and link colors under an id (`html body #<tool root> a.<tool>-btn`).
- Bare-element and generic-class rules (`h1..h6 { color: <dark> }`, `.ph { border-radius;
  background; display: flex }`) give dark headings on dark bands and rounded photos where none
  were designed. Set heading colors explicitly inside every dark band, and reset any generic
  class the component reuses (`.ph`, `.card`, `.close`, `.btn`) under its own wrapper.
- `img:hover { transform: scale(1.02) }` (and `.gallery img:hover` at 1.05) zooms every image
  inside a product preview while the mouse is over it. The fix belongs in the tool: its stage
  resets `transform` on its own images (`.<stage>.<stage> img, .<stage>.<stage> img:hover
  { transform: none }`; inline transforms still win). On any "it zooms when I hover" report,
  list the site's `:hover` rules that set `transform` before touching motion code.

Before delivery, list the site `custom.css` rules that match the component's elements (a CSSOM
walk in the browser) and override each one that changes the look; then read the computed
color **and** background of every button and link on the live page and fail any pair below
4.5:1 contrast. Template-level lesson from site-specific findings.

**Wording and stage standards** (decided by Pixfizz, 2026-10-14):
- It is the "design tool", never the "design editor", in customer copy and docs.
- No "Live preview" label on the stage.
- Drag to turn follows the hand: dragging left turns the object left. With an orbiting camera
  at `eye.x = D*sin(yaw)`, that means `yaw = startYaw - dx * k`. A product with a visible back
  turns all the way round and offers a "See the back" button.
- Photo designs always use `px-upload-dialog` (§ Open the platform upload dialog standalone) and
  a real crop with zoom, pan and pinch. A local `<input type=file>` with a single position
  slider is off standard.
- **Show resolution only when it is a problem.** A red chip "N dpi, will look blurry" on the
  file card, and one Resolution row in the check-and-approve step, only when the artwork is
  under 100 dpi at the chosen print size. At 100 dpi and above say nothing about resolution
  ("333 dpi" or "300 dpi or better" chips are noise). The dpi still goes into `px_spec` for
  production. *Decided by Pixfizz, 2026-10-19.*
- **Quiet drags.** During a drag (a resize handle, a slider) only numbers move: fields, ruler,
  price. Messages, suggestions and notes update on release; updating them on every step makes
  them flash. *Decided by Pixfizz, 2026-10-22.*
- **A panel does not jump after a tap.** A panel re-rendered with `innerHTML` that restores
  its old `scrollTop` before reordering or inserting sections lands on the wrong layout, and
  the browser clamps it: the panel jumps back to the top. Restore scroll only after every DOM
  change in that render (reorders, inserted sections, text fitting). Better, anchor on the
  tapped control: record its selector and `getBoundingClientRect().top` in a window
  capture-phase `click` listener, find it again after the render and scroll the panel
  (desktop) or the page (phone) by the difference; repeat once in `requestAnimationFrame`.
  Tool-level. *Seen in a studio prototype on a client call, 2026-10-21.*
- **Use the Pixfizz admin terms** in every tool interface a lab or staff member sees, including
  myPixfizz tools: product, variant, variant value, child variant, template option. Never
  rename them for "friendlier" words ("Choice prices" for "Variant prices" was rejected).
  **Variant trees are collapsed by default at every level** (product, variant, child variant);
  each variant is its own labelled card (a VARIANT chip, name, value count, price range);
  child variants sit indented under the value that triggers them, with a connector line, a
  different tint and a "Shown when <variant> is <value>" line. Never show a whole product's
  variants as one flat list. *Decided by Pixfizz, 2026-10-18.*

### Page layout: an inline tool owns its page

Standing rules for every tool, stated by Alex, 3 and 6 Oct 2026.

- **Inline, not a modal**, for almost every tool. A modal feels like a separate application.
- **The tool is the main feature of its product page.** Nothing above it except the site
  header and the breadcrumb: no category line, separate product title, description or tabs.
  The page H1 stays (SEO) and becomes the tool's own heading. **The product description and
  the delivery box stay in the tool's right column, under the controls** (Corrected
  2026-10-14 after a hands-on test: dropping them to full width below the tool is wrong). Move
  the non-form children of `.product-details` (`.product-description`, `.dbx-wrap`) into a slot
  at the end of the tool panel, skipping `h6.product-category`, the empty `.d-flex`,
  `.product-footer`, scripts and styles. Other tabs, the quote link and the product footer go
  below the tool, if kept. Live structure verified by query, 2026-10-14: `.product-details`
  children are `.product-category`, `.d-flex`, `.product-description`, `.product-form`,
  scripts, style, `.dbx-wrap`, `.product-footer`.
  The gallery column is hidden and the tool runs full width. Reference implementation:
  flyer-fold 0.5.1 (`ff-owns-page`): the JS moves `h1.product-name` into the tool head; the
  CSS sets `.product-details.sticky` to `display:flex;flex-direction:column`, `.product-form`
  order 1, `.product-description` order 2, everything else order 3, and hides
  `.product-category`. The uploaders do the same with `du-owns-page` / `bu-owns-page`. A
  shared `.pxt-owns-page` contract is a candidate for the parent.
- **No flash of the plain product page.** When the takeover classes (`<prefix>-owns-page`) are
  added by the asset after it loads (async), the plain page (gallery, H1, price, variant
  controls) shows first and then jumps. Standard for every inline tool: the `product/<tool>`
  snippet's light-DOM style block also does the takeover in CSS alone with
  `.product-single:has(#<tool root id>) ...` (hide the `.col-md-7` gallery, make `.col-md-5`
  full width, hide `h1.product-name` and the mirrored variant `px-option`s, clip
  `.product-price` and `.add-to-cart-button`). It applies as soon as the style block is
  parsed. Template-level (Shopper). *Decided by Pixfizz and first shipped in a tool,
  2026-10-22.*
- **Measure the first heading inside the tool for the frame check**, not the tool root. A
  shell that draws a tinted card with inner side padding (14 px phone, 20 px desktop on the
  B2B family shell `style/flyer-fold`) has its outer edge on the page frame while the tool
  heading and stage sit 14 to 20 px inside it, so the frame check (logo, first heading, first
  footer link within 2 px) fails. Fix used: zero the shell's side padding and background in
  the tool's own style snippet (`.pxt-<tool>.ff-root { padding-left: 0; padding-right: 0;
  background: transparent }`); after it heading, stage and footer matched at 375, 768, 1200,
  1600 and 2560 px on two sites. Other tools on the same shell were not re-measured.
  Template-level. *Verified by query, 2026-10-20.*
- **Where to start is never in doubt.** The first call to action (upload, choose a photo,
  start designing) sits right under the tool heading as the primary button, with one line
  saying exactly what to supply, and is repeated inside the main preview as a card that is
  also the drop target while the sample is shown. Same label and action everywhere it
  appears (top button, preview card, step panel, phone sticky bar). After the first step the
  top button turns secondary ("Replace your PDF" + file name) and the preview card goes.
  Dragging a file anywhere over the tool highlights the preview as the drop zone. Reference
  implementation: flyer-fold 0.5.2.
- **One set of controls.** When a tool runs inline, the page's own quantity, Add to cart and
  price header, and any page control the tool already offers, are duplicates. The tool still
  writes to and clicks the page's controls, so hide them, never remove them. *Done with site
  CSS on experience.pixfizz.com, 6 Oct; belongs on the parent at the next tool release.*

### The product form refuses silently

A tool mounts inside `form#project_create`, so browser form validation and the platform's
option validation both apply to everything it renders or leaves behind there.

- **A stepped number input blocks Add to cart.** A tool field `type=number step=0.25` holding
  1.68 made the browser refuse the submit: no request, no visible message, the tool stuck on
  "Adding to cart...". Any number input a tool renders inside the product form uses
  `step="any"` (or sits outside the form, or carries `form=""`), unless every value it can
  hold is on the step grid. Before submitting, check `form.checkValidity()`; on false,
  `console.error` each invalid field with its `validationMessage` and tell the customer, never
  hang. Browser behavior plus Shopper 24 structure. *Verified by query on
  experience.pixfizz.com, 2026-10-19.* For the general Add to Cart diagnostic see
  `20_SHOPPER_CART_RULES.md`.
- **A retired multiple choice option that posts empty is rejected.** When a tool version
  stops writing an option that is still published on the template (hidden, read only,
  multiple choice), it posts empty and `/v1/books` answers with the plain text "Template
  Option '<code>' trying to be set with invalid value": the platform rejects an empty value for
  a multiple choice option. Platform-level. *Verified by query, 2026-10-19.* When a tool
  version retires an option, **unpublish it on every template in the same step as the
  import**; never delete it (old orders keep their values). A test page includes the retired
  options.
- **Custom flags can arrive as the string `"false"`.** A field with no definition on the site
  (an imported value only) or a field typed as text comes through as a string; a field with a
  real boolean definition is safe. A product's `spreads_as_pages`, declared boolean, read back as
  `"false"` on one site whose definition was not checked (pending confirmation). When the
  definition cannot be guaranteed, tool code tests `=== true || === 'true'`, never truthiness.
  *Seen on experience.pixfizz.com, 2026-10-21.* Details: `22_OPTION_VARIANT_RENDERING.md` §
  Unset Booleans Export as the String `'false'`.

### Tools that show a design gallery

- **`/v1/themes/<id>/preview.webp` is not a usable design tile.** It returns a 150 x 150 image
  of the design with empty placeholders, so a photo-led cover is almost blank white (with and
  without `hide_placeholders=false`). A design-picker tool needs its own tile per design: the
  collection image, or a per-design image map in the mount. Platform-level (storefront API).
  *Verified by query on 12 designs on a client site, 2026-10-21.*
- **`/v1/themes/<id>/layouts.json`** returns the design's layouts with the page XML in `data`
  (count `<image` elements for frames) and tags such as "1 Photo" to "6+ Photo". A tool that
  auto-fills a design depends on these; a design with 0 layouts cannot be auto-filled.
  *Verified by query, 2026-10-21.*
- **`template` is nil in the mount scope, like `product`** (§ 4). A mount that passes
  `template.width`, `template.height` or `template.minimum_dpi` renders all three empty, so the
  XML size and dpi cannot reach a tool through its mount. The tool falls back to mount values
  (trim, bleed, minimum dpi), which must match the XML definition by hand. Platform-level.
  *Verified by query on a live product page, 2026-10-20.*

### Drawing a design in the browser

- **An SVG drawn to a canvas cannot use the page's web fonts.** Through an `Image` (`data:` or
  `blob:` URL) its text falls back to a system face with no error. Draw the artwork as SVG
  without text, then each text line with canvas `fillText` after `document.fonts.load()` for
  every family and weight used. Baseline used: `y + size * ((lineHeight - 1) / 2 + 0.8 + line
  * lineHeight)`. Measure each line with `measureText` to flag text that runs past its box or
  the safe area. Browser behavior. *Found building a prototype, 2026-10-19.*
- **A studio preview can be drawn straight from the page XML** the template uses, so the
  preview is the template, not a CSS copy: positions as a percentage of trim (subtract bleed
  first), font sizes in `cqw` (font size / trim width x 100) on a `container-type:
  inline-size` page. Shrink-to-fit text (`shrink="true"`) needs a pass after the HTML is in
  the document: measure `scrollWidth` against the box and scale the font size down. Give
  rendered pages a namespaced class (a short class collided with an existing studio rule and
  squashed every page). Tool-level. *Verified against production PNGs, 2026-10-19.* Calendar
  page rules are in `23_XML_CALENDAR_REFERENCE.md`.
- **Design XML can carry `cmyk(c,m,y,k)` colors and placeholder cover photo frames.** A tool
  that renders design XML converts CMYK and does not assume the cover photo is a
  non-placeholder image. *Seen on a client site's designs, 2026-10-21.*
- **pdf.js in a sandboxed page:** set `workerSrc` to a `blob:` URL that `importScripts` the CDN
  worker; a cross-origin worker URL is refused.

### Design first: a prototype of an existing tool starts from the live tool

When a tool already runs live (on experience.pixfizz.com or a client site), a prototype that
extends it to more products, or puts it on another site, is **the real tool patched**, running
with that site's data, never a new standalone mock. Two standalone mocks drifted from the live
layout and were rejected on sight, 19 and 21 Oct 2026. To try it on another site without
saving anything: fetch the live parent asset in a browser tab on the target site, apply the
changes in the page, and mount the tool root in place of the page content (site header and
footer stay). Before the first prototype build, also check the preview corner standard:
scene or view switch top right, change-the-object actions (turn over, see the back) bottom
left as an icon, upload chip only where the chosen design needs a photo. Tool convention.
*Decided by Pixfizz, 2026-10-19 and 2026-10-21.*

### Testing a tool before the live check

- **A test page must load the Shopper host CSS.** A stand-in page without the theme passes
  layout checks that fail live. Host rules that broke one tool live (Shopper 24, verified by
  query, 2026-10-19): Bootstrap 4.6 `label{margin-bottom:.5rem}` (a field built on `<label>`
  sits 8 px lower than one built on `<div>`); `.product-form select{padding:.75rem 5rem .75rem
  1rem;margin:0 0 10px;appearance:auto;background-color:whitesmoke}`; and a site rule
  `input{border-radius:10px!important}` (only a `!important` radius in the tool wins). Every
  harness loads Bootstrap 4.6.2 and those rules before the tool CSS, plus the site's
  `--pxt-*` tokens, and asserts alignment by measurement (label tops equal, controls one
  height, one radius, one field style). Then verify on the live URL anyway.
- A harness submits the product form for real and counts the `submit` event, not the button
  click (see *The product form refuses silently*).
- Test a scroll jump with `element.click()` from script: a test runner's own click scrolls the
  element into view and hides the jump.

### Generated template art

Rules for card template art generated by Pixfizz's own template generator (the greeting card
library): layers `artwork_base` (under the photo windows) and `artwork_top` (over them).
Template-level, not platform rules. *Verified by reading source and by review, 2026-10-18.*

- **Anything that touches or crosses a photo window's bounding box goes in `artwork_top`**,
  never `artwork_base`: a frame line in the base is covered by the top PNG from the rectangle
  edge down and leaves a hairline seam that prints.
- **Never feather an `artwork_top` edge that sits on a photo window edge.** A feathered rim
  made the top semi-transparent exactly there (a faint photo-colored rectangle showed) and
  cut zero-alpha seams through lettering. Start from the original top PNG; where a top edge
  lies on the window edge, extend the top layer 16 px outward using the base art's own pixels
  (full alpha for 8 px, then fade).
- **Overlapping photos:** the photo meant to sit on top owns its frame in `artwork_top` (its
  frame with a hole for its window and a soft shadow), and so does any ornament that must sit
  over a photo.
- **Motif art is illustrated, not vector shapes:** generated images placed as image elements,
  sized for print at 320 dpi of the box they fill, then WebP. Key by subject: dark or colored
  art on pure white (unmultiply against white); light or glowing art on pure black (alpha
  from the max channel); dark art with highlights on solid magenta (color-distance key plus
  despill); satin or glossy art by flood-filling the background from the edges so highlights
  stay opaque. Whether generated motif art may sit in a template that reaches production is
  an open question against `83_AI_IMAGERY_PRODUCTION.md` § A5.
- **Patterns keep their physical size in landscape.** Pattern CSS in `cqw` (percent of card
  width) makes every tile 40% larger on a 7 in landscape card than in portrait: scale the
  pattern by 5/7 in landscape. Bands that need whole repeats are generated per box as an SVG
  on a strict grid.
- **Card backs are restrained.** Never place a motif on top of a pattern (a pattern is a frame,
  a band, or a texture with nothing on it). The back photo repeats the front frame shape. One
  small accent between message and sign. The sign is one line; script display fonts need 1.5x
  the size and thin display fonts a heavier weight. A weight or style change uses only a font
  already mapped in the generator and uploaded on the site. Confetti or star patterns behind
  text need a fade or a panel under the text zone.
- **QA at native resolution**, never from browser thumbnails (they add false seams): composite
  base, placeholder and top and inspect every window edge; scan every top PNG for thin
  zero-alpha lines; check decorative pieces for contact with letters and for half-hidden
  pieces under photo frames; run a shrink scan (flag text under 72% of its nominal size, or a
  last line under 42%).

The `pixfizz-internal:custom-design-tool` skill carries the skeleton and the
`check-tool.mjs` verifier. Use it to build one; use this file to know what exists.

---

## 4. Configuration — the mount argument list, and nothing else

> **A custom design tool reads its configuration from its mount argument list, and
> from nowhere else.** No product custom field, no design custom field, no site
> checklist snippet.

(`px_rates`, § 1, is a tool's price card, read by OrderHub and the POS. It is pricing data, not
configuration, and is the one product custom field a tool reads.)

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
Verified live on a client child site, 10 Sep 2026. So `product.custom.*` in a tool
snippet was dead code that read like a working cascade. **`template` is nil there too:**
`template.width`, `template.height` and `template.minimum_dpi` in a mount render empty
(verified by query on a live product page, 2026-10-20), so a mount cannot hand the tool the
XML size or dpi (§ 3, *Tools that show a design gallery*).

**3. A site checklist is another object that does not travel**, and it puts product
geometry in Custom Admin, a screen nobody associates with a product.

### A tool mounted on a Pages page

A tool that runs on a Pages custom type page rather than on a product (a studio landing page)
can take a **site-level JSON block** in the instance's `page_content`:
`<script type="application/json" id="<tool>-config">` with `{{ '<name>' | asset_url }}`
values. It is a per-site switch with no custom field and no snippet change, and it travels with
the page, not with a template; it applies only where there is no template option to mount from.
`{{ '<name>' | asset_url }}` in a child site's `page_content`
resolves an asset that exists only on the shopper24 parent (five parent-only assets all
answered 200 from `cdn.pixfizz.com`). Template-level (Shopper 24). *Verified by query on
experience.pixfizz.com, 2026-10-21.*

### The one exception

`custom_script` itself — a custom field **on the template option**, which is why it
travels. Its definition still has to exist on the child site before a template
carrying a value for it can be imported. Nothing else works until it does.

### The mount gets its own hidden option (standard from 2026-09-20)

A tool's mount block no longer sits on its artwork upload option. It gets a **dedicated option**: type Text, hidden, hide from cart, code **`px_mount`** for every tool, name "`<Tool>` Mount" (Corrected 2026-10-14: the code was `<prefix>_mount`, for example `bc_mount`; tools already live keep their codes until their next planned update). Tool-specific template options and variants follow the same `px_` pattern (`px_spec`, `px_print_file`, `px_preview`, `px_price`, `px_pages`...), with no tool prefix. Snippet keyword arguments keep a tool prefix (`up_mode`, `ps_size`), because they become Liquid variables in the snippet scope and the prefix prevents collisions. *Decided by Pixfizz, 2026-10-14.* Its `custom_script` holds only the `{% snippet 'product/<tool>', … %}` mount block. Reasons: the artwork option has its own required flag and upload behavior, and a tool with no artwork option has nowhere else to mount. The "Mount option" column in § 2 lists where each tool mounts today; tools move to the dedicated option as they are rebuilt. *Decided by Alex, 2026-09-20.*
**Rolling a mount onto a range that is already live:** copy the mount option from one configured template to the rest with **Admin → Advanced → Bulk Update Tools** (Template Options; super admin only). Never delete and re-import a live range to add or change a mount. The mount's `custom_script` is stored and exported with CRLF line ends, so compare mounts after folding CRLF to LF. See `18_ADMIN_NAVIGATION.md` § Bulk Update Tools. *Stated by Alex, 2026-09-26 and 2026-09-29; CRLF verified by query, 2026-09-26.*

A mount on a template runs on every product that uses that template, so list the product lines sharing it before adding one (`16_PRODUCT_HIERARCHY.md`).


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

### Every customer choice on a variant: the recall (29 Sep 2026)

Every product that can be sold custom, including through OrderHub and the POS, carries each
customer choice and everything price-related on **product variants**. Tools installed before
the standard are recalled, rebuilt on it and re-added. *Decided by Alex, 29 Sep 2026.*

- **`pos_hidden`**, a boolean custom field on the VariantType, is the standard way to hide a
  variant in the POS (agreed by Alex, 26 Sep). Every **choice** variant has `pos_hidden`
  unchecked; tool-written values (`<tl>_price`, `<tl>_pages`, `<tl>_spec`, the mount) have it
  checked.
- **Before a site can take a recalled product** it needs these custom field definitions
  (definitions do not inherit): on template options `custom_script`, `hidden` and
  `hide_from_cart`; on VariantType `pos_hidden` and `hidden`.
- **Pattern** (proven on the uploaders, 27 Sep): build the new MAJOR version on baseline with
  the mount option, choices on variants and `pos_hidden` set; the tool reads choices from
  variants and still accepts the old option codes for one release, so an unrecalled product
  keeps working. Prove it with one baseline order, the job opened in OrderHub with every choice
  present, and the POS acceptance test. Per-site go-live sets are generated from each site's
  own current export, keeping its codes, names, prices, stock values and images; prices are
  carried over, never re-priced. The parent asset and snippet go up once, before the first
  site.
- **Where the change is small, recall in place** rather than by re-import, because a re-import
  creates `-1` codes beside the old product and breaks links and anything that knows the
  product by its code (POS, OrderHub). Business Cards is recalled in place (§ 7).
- **Order:** Business Cards, Sticker Designer, Gang Up, Publication Upload, Fan Face, Cover
  Studio. The per-tool moves are in § 7.

Admin routes used by the recall (verified by query, 29 Sep): a product's variant types export
at `variant_types/export_all` returns `__variant_types.yml` **including** the `custom` flags
(`hidden`, `read_only`, `hide_from_cart`); a template's options export at
`/admin/templates/<id>/options/export_all`; a variant type import onto one product at
`/admin/products/<id>/variant_types/import`.

### Stores that price with template options

Not every store prices with product variants. Some price with template options whose values
carry the price (the product itself at 0.00, `price_forecast` returning
`template_options_applied`), and a tool that only reads `variants[code]` shows no choices
there. Tool rule (verified by query on a live client store, 5 Oct 2026):

- Read a configured choice code as `variants[code]` first, then `template_options[code]`
  (radios, `px-toggle` radios, selects). `px-toggle` radios have no wrapping label and an
  empty `label[for]`; the value name is in `aria-label`.
- Where the store already has a file upload option that its fulfillment reads, point the tool
  at it (for flyer-fold, `flyer_file_code`) rather than adding a new file option, and hide the
  store's own upload block while the tool drives the page. `px-file-upload` sets `file-url` on
  success, so the standard attach wait works.

Template options remain the wrong place for new builds: this rule is about installing on a
store that already prices that way.

### Lengths accept inches and millimeters

Any length a lab types or is asked for, in a tool, a setup page or a configuration question,
has an **Inches | Millimeters** switch. Tool-level. *Stated by Alex, 30 Sep 2026.*

- Start in the unit the site's templates use (`<definition unit="...">`); inches when the
  definition does not say.
- Store and write the value exactly as the lab gave it, with its unit (`0.225in`, `5mm`).
  Convert only for display; no rounding trip through the other unit.
- Questions to a lab ask in the lab's unit, or in both.
- Internal defaults may stay in millimeters.

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
- **A mount that carries a JSON script (a rate card) must be written back with a real
  `</script>`.** The admin edit page embeds an ace-editor field's value as a JS string, and
  only turns `<\/script>` back into `</script>` at runtime. Parsing that string yourself
  leaves the backslash; writing it back stores a literal `<\/script>`, the mount's JSON script
  never closes, and the tool prices from the wrong card. Replace `<\/script>` with `</script>`
  before posting. Platform-level. *Verified by query, 6 Oct 2026 (happened and fixed on
  experience.pixfizz.com).*
- **Re-read the mount's data attributes when they change, not only when the root node is replaced.** When a Shopper 24 PDP filter swaps to a sibling product whose template also carries a `custom_script` mount, the page keeps the **same root node** and rewrites its `data-*` attributes in place (`root === oldRoot` is true, the values change). A tool that re-initialises only when its root leaves the document keeps the first product's settings until a reload. Keep a signature of the root's `data-*` config attributes and re-run init when it differs. Template-level (Shopper). *Verified by query on a live child PDP, 2026-09-29.* Same mechanism as `50_SHOPPER_TEMPLATE_REFERENCE.md` § 17, *Live `selectors:` re-renders patch the DOM, they do not replace it*.
- **Filter-switched siblings can share one template, and so one mount.** A Shopper 24 product page can switch between sibling products with a collection filter (a radio such as `name="sport"`, handled by `Px.CMS.ThemeFilters.updateRadioButton`), each value being its own product, all on the same template, plain variant included. The mount runs for every one of them. Anything that must differ per sibling reads the filter field from the page (`[name="<filter>"]:checked`), not `variants[...]`, and falls back to the plain behavior when the field is absent. To see which template page a filter page uses without admin, read the preview request `/v1/themes/<id>/preview.jpg?...&template_name=<page>`. Template-level (Shopper). *Verified by query on a live child product page, 2026-10-16.*

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
   The same schema must include the **boolean** option flags, `kiosk_mode_only` above
   all: without them an imported mount option stores `"false"` as text and Shopper hides
   it. See `22_OPTION_VARIANT_RENDERING.md` § 3.1.
2. **Import the template export.** It carries the tool's template options, their
   `hidden` / `read_only` / `hide_from_cart` flags, the variants, and the mount block
   in `custom_script`.
3. **Set prices.** Bundles ship at 0.00. Ranges must cover the full quantity span
   with no gaps — a gap returns nil and breaks the product page, not just the price.
   The first range must start at quantity 1: `{100..250=>...}` is rejected with "Price isn't
   valid", even when the product only sells from 100. Set the quantity choices with the
   product's `unit_intervals` instead (a list such as `100,250,500,1000` gives exactly those
   choices). *Verified by query, 6 Oct 2026.*
4. **Verify the mount on the live URL.** Open the product page and confirm the tool
   renders, then read the console line for the resolved configuration and
   `data-config-missing`.

Steps 1 and 2 used to be steps 1 to 4. That is the whole benefit of the change.

**Check the child's TemplateOptionType definitions before importing any tool kit.** A mount
runs only when the child has a TemplateOptionType custom field `custom_script` of type
**snippet**. On one older child with no definition, the imported value was kept and shown in
admin, but `{{ option.custom.custom_script }}` printed the Liquid as raw text on the product
page. The same site also lacked `hide_from_cart`, `read_only` and `edit_from_cart`: imported
values exist without definitions and Liquid reads them as strings (`"false"` is truthy). Open
`/site/<site>/admin/custom_field_definitions?type=TemplateOptionType` and make sure
`custom_script` (snippet), `hidden`, `hide_from_cart`, `read_only` and `edit_from_cart`
(boolean) exist. experience.pixfizz.com has the full set; shopper24 has `custom_script` as
snippet. Site-specific definitions (each child has its own schema). *Verified live,
2026-10-21.*

**A site's display-only price gate does not cover a tool's own prices.** A tool that renders
its own price text (chips, the button label) is not hidden by a site gate written for
`.product-price`, `px-product-price` and similar. Before mounting a tool on a price-gated
site, add the tool's price selectors to the gate. Template-level. *Verified by query,
2026-10-20.*

**Check the child's `product/px-option` override before installing a tool.** One child site
overrode `product/px-option` with an older copy that has no `{{ option.custom.custom_script }}`
output and no `number` branch. On that site every tool mount and every `*_price` `number`
variant failed silently. Install check for any child: open Website → Snippets → product →
px-option. If it is an override, confirm it contains `custom_script` and
`option.type == 'number'`. If not, add both from the parent before importing a tool.
Template-level (Shopper 24). *Verified by reading source, 2026-10-02.*

**Strip false keys from every tool kit before packaging.** Export only the custom keys that are
true or carry a value. A key exported as `false` (for example `kiosk_mode_only: false` on the
mount option, `hide_from_cart: false` on a visible variant, `hide_label: false`) is stored as
the string `"false"` on a site without the boolean definitions and reads truthy, so the mount
option never renders or the variant is hidden in the cart. See
`22_OPTION_VARIANT_RENDERING.md` § 3.1 and `51_CUSTOM_FIELDS_REFERENCE.md` § Key Notes.
*Verified by query, 2026-10-02.*

**Re-check `hidden` and `read_only` on every template option and variant after any
import.** Variant exports write unset booleans as quoted strings, so "unset" imports
as `'false'`, which reads as true. Never set `read_only` on anything the tool writes
to: it renders a display chip plus a hidden input, and the tool finds nothing to
drive. No `number` option may ship with a default value, or Add to Cart fails with
"value is a required field". **`hidden` is not `hide_from_cart`:** a tool-written
configuration value (a spacing setting, for example) that carries `hidden` alone still prints
in the cart and in the order emails. Set `hide_from_cart` on it in every template and in the
master exports. *Seen on a supplier's tool templates, 2026-10-22.*

### Installing from another site's live template

A lab's live tool template can be copied to another site without a file on disk: export it
with its designs and products (`16_PRODUCT_HIERARCHY.md`, the `print_theme_ids[]` and
`product_ids[]` parameters; without them the export carries no products or designs), blank
the ids, rename names and codes, **strip every custom key set to `false`** (an imported
`"false"` string reads as true), and import it. If any product in the import fails
validation, the import returns 500 **after** the template and its options are already
created; read the reason in Admin > Error Query with the reference on the 500 page. Then edit
the mount on the new site for that site's card and settings. *Verified by query, 6 Oct 2026,
installing the B2B tool set on experience.pixfizz.com.*

After the import, for the shop page: give the design a preview image (the shop tile reads the
design, not the product, see `50_SHOPPER_TEMPLATE_REFERENCE.md`), set `meta_title` so the page
title does not print the name twice, set the design `url_path` for the clean product URL
(unique per site: clear it on an old design before giving it to the new one), and take the
old product out of its collection rather than deleting it.

### Self-install from myPixfizz (in progress)

The direction (Alex, 3 and 5 Oct 2026): every Shopper tool is listed in a **Shopper tools**
catalog in myPixfizz, filterable by business type, and installs from a tool page with the
same steps for every tool: **Sizes** (formats in mm or inches), **Options** (point at the
lab's own variants from an uploaded export; no prices created), **Look** (match the store or
custom, with a live preview running the real asset), **Install** (per-format template import
file with the mount, the `.pxt-<tool>` CSS block for `style/custom.css`, shop page fields,
numbered steps). Everything that can live on shopper24 does; a lab brings only sizes, its
variants and its look. A **Fulfillment** step comes first on every tool page and is done once
per store (§ 3, and `31_FULFILLMENT_ENGINE.md` § Custom Tool Fulfillment Standard).

State: the catalog and the Flyer / Brochure setup page are built in myPixfizz and not
published; the fulfillment wizard is a prototype. Template imports are one file at a time
through admin, and template option and variant writes over the API are not available
(not verified for the latest API). A child site cannot create snippets, so every install step
on a child is an import, a template option, a variant, a collection field, or an edit of an
existing override.

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

### The tool chooses its own cart image

For a custom design tool line, the image on the cart, checkout order summary, account order
details and order emails is whatever file the tool writes to its preview template option:
`checkout/orderline-preview` draws any option whose code contains `_preview` and carries an
uploaded file, ahead of `static_preview` and `px-project-preview`. Template-level (Shopper).
*Verified by reading source, shopper24 backup of 24 Sep 2026.*

- So "the checkout should show the product image" is a **tool change, not a checkout
  change**: have the tool fetch the catalog image (design image, then product image, then the
  first design preview image) and write that as its preview file, behind a mount argument.
  No checkout code is touched. First applied in Publication Upload 1.13.2
  (`pu_cart_image: 'product'`).
- **A fallback must exist.** If the catalog image cannot be fetched, write the tool's own
  preview. A line with no preview file falls through to `px-project-preview`, which has
  nothing to draw for a project-less tool line.
- **Do not enable Add to cart until the preview file is attached.** A preview rendered on an
  animation frame does not render in a hidden tab, so a customer who switches tab during
  upload reaches the cart with no thumbnail (seen on the uploaders, 6 Oct).
- Guard image properties with `!= blank`, not bare truthiness: an empty string is truthy in
  Liquid, so `{% if design.image %}` on an empty value passes a blank into `asset_url`.

### The project edit page has no price

Cart Edit opens `pages/project-edit`, and the tool's `custom_script` mount boots there too
(form and input names in `22_OPTION_VARIANT_RENDERING.md` § 5.2c). Template-level (Shopper 24).
*Verified by reading source, shopper24 backup 2026-10-07.*
- Layout: `section.project-edit > .row` with two `col-md-6` columns: `product/gallery/standard`,
  then the product link, the design name (h3) and `.product-form` holding `form#project_update`
  with `px-option`s for variants (`book[options]`) and template options
  (`book[template_options]`), and `.save-to-cart-button` ("Save and return to cart" with
  `?cart=t`, otherwise back to saved projects).
- There is **no `px-product-price` and no `product_id` input**, so a tool mounted there cannot
  read the page price or call `price_forecast.json` from the page alone. Carry what it needs in
  its own saved state, and give money formatting a fallback.
- The saved projects page (`pages/account-saved-projects`) draws `px-project-preview` from the
  template's mapped previews, not from a tool's `px_preview` file.

### A tool can reopen its own saved state on project-edit

Save the tool's full state as a JSON file in a file upload template option. On project-edit the
option renders `px-file-upload` with a `file-url` the tool can fetch, so it rebuilds from that
file and opens on its summary. Detect project-edit as: no `#project_create` and the root inside
`form#project_update`. Read the saved values before the first write, then write with exact
`book[options][<code>]` names first, suffix match second, never `custom[...]`. Custom-tool
standard. *Verified by query on a live project, 2026-10-13.*

---

## 7. Per-tool reference

Each entry states what the tool does, what it writes, and what is open. Build specs
and session history live in project docs; this is what a session needs to work.
Brought up to date 6 Oct 2026. Every tool moves to the `px_` fulfillment codes (§3) on its
next release; the codes listed per tool are the ones live installs carry today.

### Sticker Designer: live, needs updates

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
Under the fulfillment standard: `px_print_file`, `px_print_cut`, `px_preview`.

`inline` display mode renders the designer open in the page with no launch button,
for a dedicated design page; `modal` is the default and an unknown value falls back
to it. **`inline` does not make the tool full width:** it still sits in the right-hand
`col-md-5` beside the empty gallery column, and the tool has no full-width setting. A
dedicated sticker page is full width only through that site's own CSS, which does not travel
with a template export. On a normal product page, scope site CSS by
`.product-single:has(.stk-root)` in the child `style/custom.css`: hide the gallery column,
widen the details column to 100%, unstick `.product-details`, and hide the duplicate price
header and `.product-form` (the tool still clicks it). Prefix selectors with `body` and use
`!important`, because child CSS prints before the parent's. A shared `.pxt-owns-page` parent
rule (§ 3, Page layout) would remove the need. *Verified by reading source and live DOM,
2026-10-13.* The Roll / Die-Cut toggle parks the design in IndexedDB across the
collection-filter navigation and restores it on the other side.

Already variants on live installs: `size`, `finish`, `sticker_cut_type`, `backing-style`. The
recall adds `sticker_source` (designer or print-ready upload) as a variant and a dedicated
`stk_mount`.

Known defects in 5.30.0 (verified by query on experience.pixfizz.com, 6 Oct):
- With the product's `display_each_pricing` on, the quantity ladder reads the unit price
  before it repaints, so every chip shows the first tier's unit price. With it off the tool
  divides the total and the ladder is right; use that until fixed. The ladder cache key is
  collection path + size + finish, not product.
- The last step is a review **modal** even when the tool runs inline.
- The review shows the finish **code** (for example `matte-white-vinyl`), not its name.

### Gang Up: live

Gang sheet builder for roll goods (DTF transfer rolls). Sheet **width** is fixed by
the product; **length grows** to fit what the customer adds, then snaps up to the
next available value of the length variant, which is what drives price. The lab
defines what lengths it sells by creating variant values; the asset carries no
hardcoded increment.

Mount arguments (`gu_*`): mode, sheet width and height, target dpi, margin, gap, min
and max length, max file MB, min artwork size, accepted formats, background, manual
arrange, requires design, price footer, price and Add to Cart selectors, plus `gu_trim`
and the `gu_cut*` set below.

Writes three files, each scoped to its own option: `gangup_artwork` (flattened print
PNG with alpha), `gangup_preview` (cart thumbnail), `gangup_originals` (zip of source
files plus `manifest.json`, for re-edit), and read-only `gangup_source`,
`gangup_designs`, `gangup_copies`, `gangup_sheet_length`, `gangup_output_dpi`. With a split
cut file it also writes `gangup_cutfile`. Live installs keep these codes; the `px_print_*`
rename waits for a planned per-site update.

**1.2.0 (live on shopper24 since 5 Oct, verified by query): auto-trim.** Every upload is
cropped to its first non-transparent pixel on each side, so a design exported on an oversized
artboard no longer sizes and prices the sheet for the artboard.
- Trims on alpha only (above 8/255) with a 2 px safety margin. **Never trims white**, because
  DTF prints white ink.
- Skipped when the saving is under 3%, on fully transparent files, and above 120 megapixels.
- Automatic, with **Undo per design**. Undo keeps the physical scale (clamped to the roll
  width), so the sheet visibly grows. Background removal and trim both re-derive from the
  uploaded file, so either can be undone in any order.
- The sources zip holds the trimmed file and `manifest.json` records
  `trimmed: { box, fromW, fromH }`. A reopened sheet is never re-trimmed.
- On for every site; per-site off switch `gu_trim: 'false'` on the mount.

**Cut path round each design (built in 1.1.0, shipped in 1.2.0, off by default).** For labs
that cut DTF sheets on a cutter. A configuration option, not a customer choice: mount
arguments `gu_cut*`.
- Outside edge only, holes not cut. Separate pieces are bridged into one outline (closing up
  to 0.5 in), else a convex hull, so each design comes off as one transfer.
- Default offset 1/16 in, clearance 1/16 in. With cutting on, the gap rises to 0.20 in and the
  margin to at least 0.13 in; designs closer than that block the order.
- Default output `combined`: one PDF, lossless image (Flate + soft mask, never JPEG), the cut
  line as `/Separation /CutContour` on its own layer, overprint. `split` keeps the PNG and adds
  `gangup_cutfile`.
- Pages over 200 in are written in points by default; `userunit` is a mount switch. See
  `17_DESIGN_TOOL.md` § PDF pages longer than 200 in.
- Cut geometry is copied from Sticker Designer 5.30.0 into the asset so it ships as one file.
- **Turning cutting on changes every print file on that product from PNG to PDF.** Not live on
  any site: it waits for the lab's RIP to pass three test sheets (CutContour routed to the
  cutter and not printed; a page over 200 in written in points; the UserUnit fallback) and to
  confirm the spot name is literally `CutContour`. A cut line that "does not show" on a site
  without the cut mount is switched off, not broken.

**Planned 1.3.0: sheet preview background.** A swatch control on the sheet (checker, white,
grey, black) so white artwork is visible in the builder. Viewer only: excluded from the print
render and the cart thumbnail. The existing `background` configuration flattens the **print**
file and `previewBackground` is the cart thumbnail only, so neither is the right lever. Not
built.

**Known limit: long sheets lose resolution silently.** Gang Up renders the print file on one
browser canvas, capped at 16,000 px per side and 200 MP (`computeRenderScale()`). On a 36 in
roll at a 300 dpi target the cap holds 300 dpi only up to about 53 in long; 100 in comes out at
about 160 dpi and 300 in at about 53 dpi, with no warning. iPhone Safari allows a smaller canvas
(not verified on a device). Rule for any tool whose output can exceed about 16,000 px on a side:
write a PDF with each source image embedded once at its own resolution and placed per copy,
never one flattened canvas; record the achieved dpi in `px_spec`, and surface any canvas
fallback as a defect. *Verified by reading source (gang-up.js 1.2.0), 2026-10-16; not measured
on a real order.*

### Business Cards: live

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
`bc_corners` and `bc_sides` variants. Under the fulfillment standard: `px_print_file` and
`px_preview` (routing already works, because `bc_print_front` carries `_print_`).

**Install state (29 Sep sweep, verified by query).** 16 templates on 12 client sites share
one option set (`bc_artwork_front` holds the `custom_script`). Two generations: newer installs
carry `bc_sides`, `bc_shape`, `bc_corners` and `bc_stock` as hidden variants; older ones have
no `bc_sides` variant and show shape and corners on the single-sided product only. Some older
installs have **zero values on `bc_stock`**, and a few lack `bc_print_front`. Configuration
still sits in product custom fields (`bc_sides`, `bc_trim_width`, `bc_trim_height`) on every
install; the live snippet reads the mount only, so these are dead values. Quantity is the
orderline quantity, priced by each site's own product formula.

**2.0.0 and the in-place recall (29 Sep).** Built in the repo, not stated as installed.
`business-card.js` 2.0.0 writes the supply mode to a `bc_supply_mode` **variant** when the
product has one, otherwise to the old option, and carries a semver `VERSION` constant. Sides
stay the product: single and double are separate products, each with its own formula, so no
`bc_sides` variant is added where it is missing. Per template: clear `custom_script` on
`bc_artwork_front`, import a standalone `bc_mount` option (blank id, the site's own mount
moved verbatim), import a `bc_supply_mode` variant type onto each product (blank ids,
`hidden: true`, `pos_hidden` unset so the POS shows it), then delete the `bc_supply_mode`
option. This is done **in place** because the re-import pattern would give every recalled
product a `-1` code beside the old one. Mounts differ by generation (a pre-11-Sep sides
argument, bare mounts with no sides argument, one in mm with `bc_sides: 'auto'`); each is
moved verbatim and standardising arguments is a separate step.

**Known defects:**
- The print file embeds the original bytes, which pdf-lib places without the EXIF
  Orientation tag, so a tagged phone photo reaches the press turned a quarter turn from the
  proof. See `claude/BC_EXIF_ORIENTATION.md`.
- 1.0.1 (verified by query on experience.pixfizz.com, 6 Oct): "Review & Continue" opens a
  review **modal** even inline; the review omits Paper and Quantity; inline on a phone it is a
  100dvh panel with its own scroll inside the page scroll. When the page's own Paper and
  Corners controls are left visible they show twice, once in the tool and once below it, and
  "Printed sides" exists only as a page control below the tool. Site CSS can hide the page
  duplicates and move Printed sides above the tool; the review fixes belong in the tool.
- Product `unit_intervals` takes a list (`100,250,500,1000` gives exactly those choices) and
  the tool's quantity chips follow it; a 100 to 1000 in steps of 50 setting gives 19 chips.
  Verified by query, 6 Oct.

### Cover Studio: in test

Foil imprint configurator for album and photo book covers. **Produces no print file:**
the cover is `fulfillment="false"` and the foil is struck from a physical die, so the
output is option values only, written to the product or project form. Mounts inline
on the project-edit hub as well as on the PDP; the same asset runs in both places and
field names are suffix-matched (`variants[code]` on the PDP, `book[options][code]` on
project-edit).

Mount arguments (`cvs_*`): cover dimensions, size, family, rate per line, rate per
spine line, design button selector. Writes `cvs_preview` and `cvs_sheet`.

Recall to the variant standard (planned, last in order because it needs a generator for a
range of about 38 templates): material is already on variants; `cover-imprint`,
`cover-line-1`, `cover-line-2`, `spine-imprint`, `spine-line`, `foil-color`, emboss and
`imprint-font` move to variants; `cvs_preview`, `cvs_sheet`, `cover-configured` stay template
options; new `cvs_mount`.

### Publication Upload: built, installed on two client sites

Print-ready PDF upload with browser preflight, for fixed-specification publications
(assessment books, workbooks, perfect-bound titles). Checks: opens, not encrypted,
page count against the title spec, page size against trim plus bleed in both
orientations, bleed presence and consistency, lowest placed-image resolution,
non-embedded fonts (best effort). Measures geometry from the **page boxes the file
declares** (TrimBox, BleedBox, MediaBox) when present, so an export carrying crop
marks and colour bars is measured on its trim, falling back to overhang inference.

Does **not** check color space, overprint, transparency or trapping: pdf.js cannot
see them reliably, so they are reported as not checked and left to prepress.

Mount arguments (`pu_*`): pages, trim width and height, cover mode, bleed, min dpi,
soft dpi floor, max file MB, block on fail, partner, generic, upload required, isbn,
file, file name, and from 1.13.2 `pu_cart_image`. Option codes (`pu_text_code`,
`pu_cover_code`, `pu_preview_code`, `pu_preflight_code`, `pu_state_code`,
`pu_changes_code`) are the contract and are passed only by a site that renamed one.
Under the fulfillment standard: `px_print_file`, `px_print_cover`, `px_preview`.

`pu_pages` blank means no page count check runs at all. That is the tool's only hard
fail, so blank is a real decision rather than an omission, and `pu_generic: true`
declares a title deliberately unspecified, which reads differently to a setup nobody
finished. **"Generic by design" and "someone forgot to fill the fields in" are
different states and must not be inferred from absence.**

**1.13.2: `pu_cart_image: 'product'`** makes the tool write the catalog image (design image,
then product image, then first design preview image) as its preview file, so the cart and
checkout show the product picture instead of the uploaded page. Falls back to the tool's own
preview when the catalog image cannot be fetched. See §6.

Recall (planned): has no variants today; `cover-paper`, `content-paper`, `cover-finish` and
`binding-type` move to variants; new `pu_mount` (today the mount sits on `pu_text`).

### Document Uploader: live

Upload a document, confirm its page count, configure, price, add to cart. No
personalization, no editor, no page replacement. **The tool computes the price**,
because a pricing formula cannot read the selected paper, colour, sides or finishing:
it resolves the rate card in the browser, computes one per-copy figure, and writes it
to a single number variant whose formula is `value`. **Rates never live in the asset.**
Since 22 Sep the rate card rides **in the mount**, one card per product (it previously
lived in a rate card snippet). All customer choices are on variants.

Mount arguments (`du_*`): max file MB, min dpi, full width, single cart, omit empty,
merge size into paper, thumbs max, brand, write preview, confirm pages, block on
error, the rate card, and the option code overrides. Writes `du_file`, `du_preview`,
`du_price`, `du_pages`, `du_spec`, `du_size`, `du_color`, `du_sides`, `du_paper`,
`du_sheets`, `du_notes`, `du_finishing`. 1.3.1 refuses to sell an all-zero card; 1.3.2
states the expected size under the upload step.

The parent copy of `document-uploader.js` 1.3.2 and `product/document-uploader` is
byte-identical to the baseline override, so a child site needs no asset and no override for
the uploaders (verified by query, 6 Oct). Test order on experience.pixfizz.com, 6 Oct: a
12-page Letter job produced a 12-page print PDF at 8.75 x 11.25 in with trim 8.5 x 11 in,
plus a preview.

**Known defects (shared with Booklet Uploader):**
- **Bleed is read from PDF boxes only.** A file with no TrimBox or BleedBox is measured on its
  MediaBox as the trim, so a correctly bleed-sized page is reported as the wrong size and gets
  a white border. Verified by reading source (1.3.2 / 1.5.1).
- **The preview renders only on a visible page.** The render waits for an animation frame; in
  a hidden tab the print file attaches but the preview does not, and Add to cart is already
  enabled, so the line reaches the cart with no thumbnail. The tool should not enable Add to
  cart until the preview is attached. Verified by query, 6 Oct.
- **Phones:** the parent rule `.du-owns-page .du-col-left` (and `.bu-owns-page .bu-col-left`)
  has `min-width: 440px`, so at 390 px every step card is cut off on the right, on every site.
  The fix belongs in the parent stylesheet (drop the min-width under 768 px); a site can
  override it in `style/custom.css` meanwhile. Verified by query, 6 Oct.
- The cart line shows no paper, color or sides, because the spec variants are hidden.
- Replacing a document could leave a silent "Preparing your print file"; fix planned in
  Uploader 2.0.

### Booklet Uploader: live

Fork of Document Uploader `2026.09.15-7`. Four things differ and everything else is
inherited: a folded sheet carries **four** pages, so saddle stitch is
`ceil(pages / 4)` where coil, wire and perfect stay at `ceil(pages / 2)`, and the
sheet mode comes from the chosen **binding**, so binding feeds the price; only
bindings compatible with the detected page count are offered; the divisible-by-four
rule **pads, it never rejects**, and the padded count is what is printed and priced;
and the preview shows reading order, spreads and the binding drawn over the spread.

**Never impose or reorder pages.** The RIP does imposition; the tool supplies the
flat PDF in reading order and states the page count. Adds `bu_binding` to the
Document Uploader option set. The rate card (interior and bindery bands, covers, an
`overrun_pct`) rides in `bu_mount` and the tool writes the per-book price to the `bu_price`
number variant. An overrun shown to the customer ("Printed, including 5% overrun") reads as an
error, which is why Uploader 2.0 keeps it internal. Same known defects as Document Uploader.
Test order on experience.pixfizz.com, 6 Oct: 16-page Letter and 8-page Half Letter print PDFs
at the trim plus 0.125 in bleed each side.

### Uploader 2.0 (documents, booklets, forms): prototype

**One tool, three modes**, decided 5 Oct: the booklet uploader was forked from the document
uploader and shares its print composer, preflight, pricing engine and rate card model; only the
preview and the sheet maths differ. Uploader 2.0.0 is one asset, one stylesheet and one
snippet, with mount argument `mode: 'flat'` (documents, copies, flyers), `'booklet'` (spreads,
bindings, cover) or `'forms'` (carbonless forms). MAJOR version, because the option codes move
to `px_print_file`, `px_print_cover`, `px_preview` and `px_spec` (`tool=uploader; v=2.0.0;
mode=...` first). The old assets stay on the parent untouched for live mounts until each site
is moved; the new asset reads both `du_mount` and `bu_mount`, so no live mount code needs a
rename.

Carries every standard set since the 27 Sep go-live: customer choices on variants, rates in
the mount, the tool owns its page, an obvious first call to action, the shared look layer, the
cart image via the preview option, the mobile standard, generic and installed on shopper24.

Booklet mode adds, generically: cover as paper x print; per-paper print bands; a conditional
overrun rule (applied only above a copy or page threshold, never shown to the customer); a
max-pages matrix by cover paper x interior paper (marked cells are warnings, not limits); a
blank placement picker shown only when padding is needed (all padding blanks together at the
chosen spot); a grayscale preview for black and white pages; **two production files when the
cover is on its own stock** (`px_print_file` interior and `px_print_cover`), one under a self
cover; a fit choice when the file is not the trim size (scale to fit, or original size and
trim, recorded as `fit` in `px_spec`); and a customer summary without production metrics
(the volume panel shows price per book at other quantities, never sheets). Under a one-sided
cover with the cover in the main file, pages 2 and N-1 of the file do not print, and the
prototype warns.

Flat mode adds: the page count read from the PDF is confirmed automatically, with a Confirm
step only when it cannot be read; replacing a document re-finds the platform input after a
re-render, retries once, then shows a plain error.

Forms mode (carbonless forms, 5 to 6 Oct, not a new tool): a one-page PDF with forms
preflight (exactly one page blocks; size, margins and binding edge, raster resolution,
embedded fonts, RGB and bleed warn), a **matrix rate card** (price by finishing x size x parts
x color x quantity, also useful for flyers, postcards, envelopes and letterhead), a numbering
add-on, a `#####` placeholder found in the PDF text layer and previewed as the starting number,
and a book or loose-set preview. The tool writes the job total to the one price variant with
cart quantity locked at 1. **Request a quote** is a zero-price line flagged `quote=yes` in `px_spec`,
which needs an exception to the all-zero rule for that line only; zero-total orders can check out
(stated by Alex, 6 Oct). Whether staff can price a placed order in admin afterwards is not
verified.

Pricing method: where a lab supplies its own Ruby pricing formulas, they go into the rate
card unchanged as tables and are run as an **acceptance oracle**: every selection must match
the formula to the cent (see § 1, *When the tool computes the price*).

Status: prototypes published 5 to 6 Oct, the booklet engine checked against a lab's 20
formulas (97,200 combinations, 0 differences) and the documents engine against 1.3.2
(32,256 combinations, 0 differences) [test]. Nothing built on the platform. A lab-reachable
rate card editor that covers the new fields is still a gap.

### Flyer / Brochure (flyer-fold): live on one site

Takes a PDF, PNG or JPG for a flat or folded piece, reads the fold and size from the
product's own variants and keeps both in sync, computes panel widths with the shared
fold engine, and renders the fold in 3D. Writes the artwork, the geometry and the
artwork-check verdict to the orderline (`flyer_file`, `flyer_preview`, `flyer_spec`; under
the standard `px_print_file`, `px_preview`, `px_spec`).

**The first tool on the generic model** (0.5.0, 3 Oct): parent asset and snippets, one mount
per template, the store's own variants listed by code in `flyer_variants`, the platform price
for its quantity ladder (§1), and a `.pxt-flyer` look. 0.5.1 is the reference implementation
of *the tool owns the page* and 0.5.2 of *where to start is never in doubt* (§3). 0.5.2 is
live on shopper24 and on a client's hidden brochure pages (verified by query, 3 Oct).

Not yet proven on any other site. In the order they close:
1. A generic install end to end on experience.pixfizz.com through the myPixfizz setup page
   (sizes, options from that site's own export, look, download, import, shop page blocks, one
   test order). The template download has only been built from one client's exports; a size
   that differs from the source export has never been imported, and the generic banner HTML
   and collection footer have never run on a generic site [not verified].
2. A lab with no brochure product cannot install, because every file is built from the lab's
   own product export. Needs a neutral starter export per fold (C, Z, half) with no variants
   and no prices.
3. Flat flyers: the tool only knows `flyer_fold` c, z and h. A flat type (1 or 2 sides) is
   planned for 0.6.0 with `flyer_hide`.
4. Feedback from a client call (5 Oct): options do not read as a dropdown; a black and white
   variant should switch the preview to grayscale; the approval message is too long.

On a store that prices with template options rather than variants, the tool must read
choices from `template_options[code]` too (§4), and can point at an existing file upload
option that fulfillment already reads (`flyer_file_code`) instead of adding its own.

### Fan Face: not ready

Head cut-out on a stick. Ported from the Sticker Designer with MediaPipe selfie
segmentation and face detection added for the head clip, neck line and radius
guarantee. Built and delivered 30 Jul 2026, **and not sellable as it stands: the head
cut needs real AI segmentation, not MediaPipe.** Treat it as blocked on that, not as
a tool awaiting an install, although two client sites carry an install (29 Sep sweep).
Recall (planned): `size` is already a variant; `facefan_shape` and `facefan_source` move to
variants; new `ff_mount`.

The two legacy MediaPipe solutions must be initialised strictly one after the other:
constructing the second before the first has finished aborts with
`Module.arguments has been replaced with plain arguments_`.

### Design Brief: unfinished

The Design Service wizard: collects the brief, drives the customer gallery, and
writes the whole thing onto the orderline. The book itself is not priced in the tool:
the customer buys it after approving the design, so everything captured is a
preference for the designer, not a locked specification.

### Film Order Builder: in progress

Film processing order builder: formats, services, scan resolutions and returns
assembled into one order. **The rate card in the current asset is demo data**, blended
from two real customer exports for the prototype. It is not a Pixfizz price list and
must not be quoted to a client. Respecced 16 Sep 2026.

### Wall Designer: prototype

Multi-panel wall art: splits (one photo across several panels), clusters and gallery walls.
Inline and full width on the product page, not a modal (an agreed departure from the shell
standard). Two prototypes since 6 Oct, one engine: **0.2.1** for metal and acrylic panels, and
**0.3.0** for a print-on-demand supplier's magnetic wall tiles, glass panels and skateboards,
with a mixed option. Count first, then arrangement; layout cards show piece count and price,
not dimensions; step 1 is the customer's photos, with samples on the wall to play with first;
modes are one photo across all pieces or one photo per piece; drag to swap; a hang guide; a
per-piece sharpness check at print size; in/mm switch; minimum-dpi 100 from the XML.

Data model (standard): layout, panel sizes, material, finish, mounting and the priced quantity
are product **variants**; the price carrier is a number variant, not a number template option.
Below minimum dpi the prototype warns and asks for review; whether to block Add to cart is not
decided.

**Baseline test template (26 Sep):** a triptych, 24 x 16 in panels, built from the Collage
Wall template export as the reference shape. A wall page (`fulfillment="false"
preview="true"`) plus three `editor="false"` panel pages at 24.25 x 16.25 in (0.125 in bleed
each side, taken from the wall around the panel), jpeg, 300 dpi, minimum-dpi 100. Panel pages
use `<ipage crop="true" template="wall">` with zoom 0. Two themes: one placeholder over the
whole wall (split), and one placeholder per panel (cluster). If it imports and renders as
calculated, production gets one platform-rendered file per panel. **Not verified:** that the
import accepts a blank-id product row, that `preview="true"` behaves, and that `<ipage>` with
zoom 0 on a non-square wall crops as calculated. Open with the core developer: whether a
custom tool can write the photo and its placement onto the wall page itself.

Open before platform work: the cart model (one line per piece, or one wall line with several
print files), print window per piece against the supplier's spec, hanging gap. Nothing
installed.

### Board Engraver: prototype

Laser-engraved wooden cutting boards. Kebab `board-engraver`, prefix `be_`, JS global
`BoardEngraver`, proposed mount option `be_mount`. Replaces the editor and produces the laser
file. Prototype 0.0.1 (27 Sep): the board drawn as the product with real thickness, a burn
color per wood multiplied into the grain, tilt on hover or drag; preset designs that adapt to
landscape and portrait engraving areas; values carried between designs by slot id; text that
fits itself to width and blocks below a minimum capital height; up to four extra texts or
elements; a review of the board or the 1:1 laser file; a spelling acknowledgement before Add
to cart.

Proposed, awaiting Alex: **one product per blank, designs chosen inside the tool** (instead of
one product per design); board (wood and size) as the variant and price carrier; template
options `be_mount`, laser file, preview and design state JSON; production file SVG or PDF at
1:1 with fonts converted to outlines in the browser, black engrave, board outline on a named
non-engrave layer. Board geometry from the XML definition where it can; the outline shape is
tool-side. Proposed mount keys (none built): `be_board_variant`, `be_boards`, `be_designs`,
`be_fonts`, `be_min_cap`, `be_extras_max`, `be_output`, labels.

Laser output constraints from research (not platform facts): vector for text and line art,
raster only for photo engraving at 300 dpi minimum; bold sans and slab survive small sizes,
thin serifs and delicate scripts fail; keep strokes at least about 0.5 mm. Edge and groove
margins, laser software and file format must come from the lab.

### Custom Sized Prints: in build

The customer uploads an image, sees a preview, then chooses the exact print size; the tool
detects the photo's ratio and suggests size groups. The preview shows the print at its
relative size against standard objects on a wall. Scope decisions (Alex, 27 to 28 Sep):
inline, not a modal; **never mixes paper, metal and acrylic in one tool**: one product per
material family, each with its own configuration (for example 1 in steps for canvas, 1/4 in
for prints); size range, increment and bleed configurable per product (plain prints need no
bleed, metal and acrylic usually 1/8 in per side); default unit in the configuration with an
optional switch; stock choice and **all pricing, including the rate per square inch, on
variants**. Writes the print file to a template option file upload (`px_print_file` from its
first installed release) and keeps the original. Banners need a different interface, later.

**Pricing moved to `px_rates` (22 Oct 2026).** The build moved from per-stock area Number
variants with range-hash Ruby formulas (which OrderHub cannot evaluate) to a `px_rates` card in
mode `area`, with the generic codes `px_width_in`, `px_height_in`, `px_stock` and `px_price`
plus `px_hanging`, `px_depth`, `px_edge`, `px_edgecolor` and `px_grommets` (§ 1). (Corrected
2026-10-22: the 27 to 28 Sep scope put the rate per square inch on variants.) A counter-only
product can carry the same card for OrderHub. 0.2.1 is the first tool with the CSS-only page
takeover and quiet drags (§ 3). Room scenes that hold any size are in
`27_LIVE_FINISH_AND_3D_PREVIEWS.md` § 2.11.

### Restoration Estimator: specified, not started

Instant restoration estimate: the customer uploads a photo of the photo, the tool
measures what it can in the browser and returns a price band, a turnaround date and
an honest list of what restoration cannot fix. Tier 1 (browser only) ships without
any server; the vision-model tier needs a Pixfizz-owned proxy to hold the API key,
which is an open decision. **A vision API key may never ship to the browser.**

### Acrylic Statue Designer: specified, not started

Flat acrylic figure, UV printed and laser cut to a contour, standing in a separate
base. The new work over Fan Face is **support geometry**: a figure whose silhouette is
too narrow at the bottom cannot stand, so the die adds a clear support region and a
tab. A tool that only traces the silhouette produces a piece that falls over.

### Collage Wall: unfinished

See Wall Designer: its template emitter is the reference for the Wall Designer baseline test.

### Custom Framing: demo only

A framing tool mounted on one template on experience.pixfizz.com (29 Sep sweep). No build
document in the sources; moves to `px_print_file` from its first installed release.

---

## 8. Definition of done

A tool is not finished until all of this is true. Each item has cost time at least
once.

- **One real order placed end to end.** Every defect found on this platform so far
  survived every check short of that.
- **The print file measured on a real order** — format, colour mode, alpha, pixel
  dimensions, embedded DPI. None of them are visible in a proof.
- DPI read from the XML definition, never hardcoded. The mount cannot pass `template.*`
  (§ 4), so until the page or an endpoint exposes the definition, the mount's dpi and size
  values are checked against the XML by hand.
- The upload dialog check passes (§ 3), and the dialog wording and accepted types match what
  the product takes.
- Configuration resolved in the browser with source markers and `data-product-seen`.
- The shared notify layer consumed, never reimplemented.
- Dependencies loaded from the **product snippet**, not from
  `integrations/custom-body-scripts` or the layout, both of which child sites
  routinely override, and injected with a `data-<tool>-src` marker so a re-render
  cannot double-load.
- The preview code added to every surface in §6 that still carries a list.
- Production files on the `px_` codes (§3), with `px_spec` in the standard format.
- Every customer choice and price input on a variant, `pos_hidden` set on tool-written values
  (§4).
- Inline, owning its page, with an obvious first call to action (§3), and checked at 1440 and
  390 px wide with no horizontal overflow.
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
- 2026-09-29: Pointer: roll a mount across a live range with Bulk Update Tools; custom_script stored with CRLF. §4 a PDP swap keeps the custom_script root and rewrites its attributes; tools must re-read config on attribute change. Install step 1 must include boolean option flag definitions. Source: claude-chat.
- 2026-10-06: §2 registry and §7 per-tool reference brought up to current state (versions read on shopper24 3 to 6 Oct, install footprint from the 29 Sep sweep, new rows for Uploader 2.0 with forms mode, Wall Designer, Board Engraver, Custom Framing; Gang Up 1.2.0 auto-trim, cut path and planned preview background; Business Cards 2.0.0 in-place recall; uploader and sticker known defects; Flyer / Brochure install gaps; Custom Sized Prints scope); §2 how the install list is found, pointer to 27 for Live Finish and 3D previews. §1 price ladders read price_forecast; tool-computed prices with a Ruby formula oracle and 9-decimal rounding. §3 the px_ fulfillment code contract, the shared px-tool-theme look and dark-site ink, page layout rules (inline, owns the page, obvious first action, one set of controls). §4 the variant recall standard with pos_hidden, stores that price with template options, inches and millimeters, writing back a mount with a JSON script. §5 first price range starts at 1, installing from another site's live template, myPixfizz self-install direction. §6 the tool chooses its own cart image. §8 new done items. Applied spills: Product Previews pointer (B1); §5 px-option override check and strip-false-keys rule (D1); §3 open the platform upload dialog standalone, with events, gallery routing and sign-in behavior (C). Source: claude-chat, fireflies-call, vault-doc.
- 2026-10-09: § 3 the shared look: `.pxt-step` and `.pxt-shell` are contract names a tool must not reuse; pointer to the variant-selector rules in 17. Source: claude-chat.
- 2026-10-17: § 1 the rate card moves from the mount to the snippet-type product custom field `px_rates` (printed by `product/design-now` as a JSON script tag), and the agreed `pricing.js` engine contract shared with OrderHub (versioned, `px_price` plus text choice variants); § 4 pointer that `px_rates` is pricing data. § 3 tool assets one stem per tool and a new asset per version; upload dialog button classes and text attributes, `upload-success` storage, progress detail, closing the dialog mid-upload, `preview` is a 250 px long-edge thumbnail; new § Using the platform crop tool standalone (`px-image-adjust-tool`); shared look: text outside tool surfaces on dark sites and full dark mode by tokens, `:where()` element resets, wording and stage standards. CORRECTED § 3 page layout: description and delivery box stay in the tool's right column. CORRECTED § 4 mount option code is `px_mount` for every tool, `px_` codes without tool prefix; filter-switched siblings share one mount. § 6 the project edit page has no price; a tool can reopen its saved state on project-edit. § 7 Sticker Designer `inline` is not full width (site CSS pattern); Gang Up canvas resolution ceiling on long sheets. Source: claude-chat, slack-message.
- 2026-10-22: § 1 books counted in spreads price in system pages; `px_rates` mode `area` with generic `px_` size codes and counter-only products. § 2 and § 7 Custom Sized Prints moved to in build, priced by `px_rates` (CORRECTED: rate per square inch was on variants). § 3 upload dialog CORRECTED to mandatory for every tool taking a customer file, a tool opens calm, `sources` required, no button look of its own, classes read once at connect, canonical Shopper class set and repo check, `SUPPORTED_FILE_TYPES` and no `accept`, print wording for the five photo-worded attributes, `url` same-host and untainted, feeding files in a test, studio install upload dialog check; new § attaching a file (`file-url` is content addressed, edit-from-cart restore); defend against site `custom.css` (link overrides, generic classes, `img:hover` zoom, contrast check); wording and stage standards (resolution only under 100 dpi, quiet drags, no panel scroll jump, admin terms and collapsed variant trees); page layout (CSS-only takeover with `:has()`, measure the first heading inside the tool); new § the product form refuses silently (`step="any"`, `checkValidity`, retired multiple choice options), § tools that show a design gallery (`preview.webp` tiles, `layouts.json`, `template` nil in the mount), § drawing a design in the browser, § design first from the live tool, § testing a tool (host CSS), § generated template art. § 4 `template` is nil in the mount scope; a tool on a Pages page takes a `page_content` JSON block (parent-only asset resolves from a child). § 5 TemplateOptionType definitions check, price gate covers tool prices, `hidden` is not `hide_from_cart`. § 8 dpi from XML by hand, upload dialog check. Source: claude-chat.
