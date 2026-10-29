# kbsync 2026-10-29: what changed

Sources: 99 internal working notes written 22 to 29 Oct (75 applied in full or in part, 24 skipped), and the internal feature log (no new entries since 22 Oct). Base: `main` at 9709cca, checked again before delivery. Each group of files was written against the source notes and re-checked against the existing text; conflicts between a note and the KB were settled from the note's evidence and dates, or recorded as pending.

Upload every changed .md file in this folder to github.com/pixfizz/pixfizz-knowledge, main branch, replacing the existing ones. 20 files. Nothing in `orderhub/` is touched.

## Corrections (marked "Corrected 2026-10-29" in the files)
- 18 Templates: the Versions page diff is not a source for the current template XML, because a diff can show only the changed hunk. Read the template page's own content.
- 18 Template Provider and 50 § 24: an asset on the provider site alone works as the consumer's shop tile image. No parent copy is needed.
- 19 Element Permission Flags: `radius="50%"` renders a rounded square. A numeric mm radius of half the width makes a circle.
- 50 § 5: `home-page-login-form` gates the whole site, not only the home page.
- 50 § 15: the email kit reads `email-logo.png`; the old default templates do not.
- 27 § 4: the theme render draws print borders when the border variants are passed in the query.
- 83 A5 and 26 Generated template art: generated artwork is allowed in Pixfizz-made templates that reach production (decision of 26 Oct). Customer-placeable asset libraries stay under A5.

## Reconciled, not corrected
- 61 `/fz/` URLs: a URL loads cross-site from the host that issued it. The earlier 403 came from an admin-issued URL rewritten onto the storefront host.
- 26 and 61 `/upload/image`: both `data` and `Filedata` were seen to work. New code sends `data`.

## New rules and sections, by file
- 50, 20, 21, 30, 51, 41: mega menu panels; brand color from `.btn-primary`, not `--primary`; `add-to-cart` renders inside the layout; `project-edit` structure and the `px_mount` full-width rule; email kit logo; whole-site login gate and the password pages; product-landing engine (Liquid in config, frame, redirects); animated GIF tiles; prints page server cost, `x-cms-cache-hit` and the cache after snippet writes; audit for child overrides of the filter snippets; POD-published products that render empty in a collection; a multi-design product in a `pdp_layout` collection shows designs as sizes; `cart_product_name_format` and `checkout/extra-cart-code`; switching between pickup and delivery; converting cut prints to a base price plus per-tier differences, `price_forecast` and cut print quantity; a boolean custom field drops false and 0; canvas hit tests with `object-fit: contain`.
- 26, 27, 83: registry rows for Greeting Card Studio, Photo Book Studio and Wall Design Studio; upload size limits (server, `px-image-upload` `max-size`, admin dialog, `pu_max_file_mb`); upload throughput; upload pool ids are strings; `.container-fluid` frame; sticky under `overflow-x: hidden`; the takeover flag; the platform `popstate` morph and `data-px-no-rerender`; muted ink contrast; pdf.js for customer PDFs and its boot; Business Cards known defects; Live Finish label keys, mounting depth before edge color, flat products from one photo, binding sprites, one mount shared by two ranges, option import by POST; Nano Banana reserved space and moving lettering.
- 61, 31, 32, 70, 71, 80: bulk user create; a cart line from template options only, orderline delete and `/cart/clear`; `POST /upload/file`; PDFs rasterized at 300 dpi; font palettes and element geometry from the theme JSON; the 50 MB admin upload ceiling; cross-site asset library; `asset_url` hashed per file; resizing an asset in place; adding an existing product to another collection and writing collection Ace fields; the product `fulfillment` write; option choices in fulfillment file names (harness-tested only); email template form writes; a myPixfizz push to `main` does not deploy edge functions or run migrations; store health data can be stale; how Etsy orders appear; ask for the site backup without assets.
- 18, 22, 19: Templates list paginates and leaves out provider templates; remove products on the collection page; a snippet override gets a new id after revert and re-override; `print_themes=0` publishes the first design only; `product_id=new` creates at price 0; Liquid in default fulfillment; variant label images served as PNG; color controls on uploads; crop ratio per template, crop value geometry, no change event from `px-image-upload`; file option values are `db:<id>`; `multi_upload_group` heading, adjacency, reorder and progress; option URLs and the Back button; array custom field filters; `pdf_layer`; masks white where the photo shows; point-size `fontsize`; greeting card fold bleed and converter rules; building design exports outside the admin.
- 02: 48 new routes, every target file checked.

## Not added
- Internal tooling and delivery conventions (governance, not KB).
- Commercial: prices, AI token costs, client share counts.
- Site-specific configuration: one client's link color override, one tool's asset name collision, studio text keys per orientation.

## Pending confirmation
13 questions for the core developer are pending (kept outside this repo).
