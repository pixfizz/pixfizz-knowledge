# 17 — Design Tool

**Authority Scope:** Design Tool Configurations, feature toggles, and customer-facing editor behavior.

_Last updated: 2026-09-09_

---

## What is the Design Tool?

The Design Tool is the interactive editor customers use to personalize products. It is embedded in the storefront and provides a visual interface for uploading images, adding text, applying layouts, and previewing the finished product.

The Design Tool is highly configurable through **Design Tool Configurations**.

---

## Design Tool Configurations

Each Pixfizz environment can have one or more Design Tool Configurations. A configuration controls the design tool's appearance and available features for a specific context. Templates reference a configuration, so different product types can offer different design experiences.

Configured in admin under: **Settings > Design Tool**.

### Multiple configurations per site

A site can hold several configurations side by side. A configuration is assigned to a **Template** or to a **Design** — it is not attached to a product, a collection, or a category. Whichever object the customer enters the editor through determines the configuration that loads.

Running several configurations is the intended pattern where different product types need genuinely different editors (a card versus a photo book versus a wall-art product). Assign each Template or Design to the configuration that suits it rather than trying to serve everything from one configuration by toggling features on and off.

### Per-configuration help modals

The in-editor help modal content is a snippet, and a different snippet can be assigned per configuration through the Trip JS tutorial configuration. This means each configuration can carry its own tutorial and instructions without duplicating the editor.

Trip JS tutorial blocks carry separate desktop and mobile sections. When adapting desktop content for mobile, use fluid dimensions (`width: 100%`, `max-height: 60vh`, `overflow-y: auto`, `box-sizing: border-box`) rather than fixed pixel width/height. Desktop copy wraps to far more lines at phone width, so a fixed height clips the lower paragraphs and a fixed width sits inset inside the Trip modal leaving white gaps.

---

## Configuration Settings

### Branding
- Name, logo, and favicon
- Brand color
- Page title

### Feature Toggles (30+)

#### Image Features
- Image Rotation
- Crop Toggle
- Color Adjustments
- Filters
- Flip
- Image Borders
- Image Size Overlay
- QR Uploader
- PDF Imports

#### Layout & Design Features
- Autofill Button — auto-populate images into zones
- Two Page Spread — enable spread view for multi-page products
- Crop Bleed — show/hide bleed area
- Alignment Aids — snap guides and alignment helpers
- Background Colors — allow background color changes
- CMYK Color Picker — professional color selection
- Border Radius — rounded corner controls
- Shape Button — add shapes to designs
  - Shapes support color palettes (selectable from the configured color palette)
  - Shapes work with fulfillment transformations and calendar transformations
- QR Code Button — generate QR codes in designs

#### UX Features
- Unedited Warning — alert if zones haven't been customized
- Project Options — show project-level settings
- Display Price — show pricing in the design tool
- Copy Shared Projects — allow copying shared project links
- Highlight Editable Elements — visual indicators for editable zones
- Use Mapped Preview — advanced preview rendering

#### Specialist Features
- Large Format / Wall-Art mode
- Cut Print Autorotate

**AI image tools (URL flag):** the AI image-manipulation tools in the editor are gated behind a URL parameter. Append `&aitools=true` to the design-tool URL (`?aitools=true` if there are no other query params) to expose them. Platform-level; works wherever the editor loads. Source: #development (2026-07-01).

**AI restyle presets (launch set).** Seven styles ship in the Restyle section:

- Watercolor
- Pencil Sketch
- Oil Painting
- Cartoon
- Pop Art
- 3D Character
- Anime

Vintage Film was drafted during development and is **not** in the launch set. Generation is billed per image to the lab, not to Pixfizz, so usage carries a daily limit with a site default and per-user overrides.

**Filter selection auto-applies.** Selecting a filter (Restyle preset or Image effects filter) now applies it immediately. Previously, selecting a filter only showed a preview — if the customer went straight to cart without pressing an explicit **Apply** button, the filter was silently discarded. This has been changed to auto-apply on selection so the result can no longer be lost this way. Deployed 2026-07-27. Source: slack-message (#development), commit 8e2736ec.

**Known issue — AI token usage counted globally, not per site (fix deploying to production the week of 2026-08-03).** Enabling AI tools on one site could show token usage that included consumption from other sites on the same account, rather than being scoped to that site alone (e.g. enabling AI on `photosynthesis.pixfizz.com` showed tokens consumed elsewhere). A fix to scope AI usage trackers by website has been verified on staging and is pending production deploy. Until then, do not treat a site's displayed AI token count as an accurate per-site figure. Source: slack-message (#development), commit 9615afd5.

### Typography & Color Defaults
- Default Font, Font Palette, Font Size
- Text Color Palette
- Default Image Border Color Palette
- Background Color Palette

### Page & URL Settings
- Homepage URL
- Product Display Name
- Cart Page Name
- Account Page Name

### Integrations
- Google Tag Manager ID
- Image Sources — comma-separated list that specifies the **order of the source icons** in the editor's Images tab (not merely which sources are on). Allowed values: `device`, `galleries`, `public_galleries`, `groups`, `clipart`, `pdf_import`, `dropbox`, `google_photos`. Defaults to `device` when the field is empty. To expose a PDF import toolbar button you must do two things: enable the PDF Imports feature toggle (under Image Features) AND add `pdf_import` to this Image Sources field. `dropbox` and `google_photos` need provider OAuth setup before they work (see Google Photos as an Image Source below).
- Help URL
- Custom JS — inject custom JavaScript
- Custom CSS — inject custom styles

> Some Design Tool Configuration settings are only visible to Pixfizz staff. These control platform-level behaviors and are managed during onboarding or through support requests.

### Google Photos as an Image Source

`google_photos` is an allowed Image Sources token, but unlike `device`/`galleries` it needs OAuth setup before it works. Per storefront:

1. In Google Cloud, create an API Console project and a **Web Browser** OAuth Client ID. When asked for the calling domain, enter the storefront's own domain (e.g. `clientsite.pixfizz.com`). Guide: https://developers.google.com/identity/oauth2/web/guides/get-google-api-clientid
2. Complete Google's OAuth consent / API access configuration so the Client ID is authorised to fetch photos via the Google Photos API.
3. Copy the Client ID into the storefront's **Super Admin** panel, **Google OAuth 2** field (left-hand settings column).
4. Add `google_photos` to the **Image Sources** field on the relevant Design Tool Configuration.

Once the Client ID is in place and Google Cloud permissions are correct, customers can select images from Google Photos inside the editor.

## Admin Mode Editor

The "Open in Editor" action on an order, which re-opens a customer's project in
the design tool from the admin, is only visible when the Admin Mode Editor toggle
is ON. This toggle lives in the Design Tool Configuration under Editor & Templates
and is Super Admin only. It does not appear in the standard admin panel.

---

## Login Modal in the Design Tool

The design tool can be configured to show a login modal when an anonymous user attempts to save a project or access the galleries tab.

### Default behavior

- **Existing configurations:** The login modal is **not enabled by default** on existing design tool configurations. It requires additional configuration when used from an external site (such as Shopify).
- **New configurations:** Enabled by default on any new design tool configuration created in admin.

### Trigger

The login modal fires when the user clicks **"Save & Continue"**. It does **not** fire on "Save & Exit" — the Save & Exit flow relies on the target page (typically the account page) to detect whether the user is logged in and prompt login if not.

### Optional links

To show "Forgot password?" and/or "Register here" links next to the email/password fields in the modal, configure the corresponding URLs in the design tool configuration settings.

### Shopify and other external site behavior

When the design tool is embedded in an external site (such as Shopify), the login flow does **not** use the in-tool modal. Instead it opens a new browser window/tab pointing to the URL configured under **"External Login URL"** in the design tool configuration.

**Setup for Shopify:**
1. Set the **External Login URL** in the design tool configuration to a Shopify page that handles login.
2. After successful login, the user is redirected to a page that must include the Pixfizz setup code. The default Shopify integration includes the setup code on all pages — verify this for any custom theme integrations.
3. Recommended: create a custom Shopify confirmation page that says "You are now logged in. You may close this window and continue in the design tool."

The redirect-back-to-design-tool flow depends on the Pixfizz setup code running on the post-login landing page. If using a custom Shopify theme, confirm the setup snippet is present on every page before enabling the login modal.

### Login inside the upload dialog

`<px-upload-dialog>` can render the same login form, so an anonymous user can sign in from the
upload dialog to reach their galleries. It is **off unless explicitly enabled**, because an
external site such as Shopify needs the external login URL configured.

Four attributes, matching the design tool's login modal settings:

```
login-modal-enabled
login-modal-external-url
login-modal-forgotten-password-url
login-modal-registration-url
```

- **On `<px-upload-dialog>`** — use the attributes as written above.
- **In the PhotoPrints component config** — replace dashes with underscores
  (`login_modal_enabled`).
- **On `<px-image-upload>` and `<px-multi-image-upload>`** — prefix with `upload-dialog-`
  (`upload-dialog-login-modal-enabled="true"`).

The forgotten-password and registration URLs are site-specific; do not assume every Shopper
site uses the same paths. The dialog added ten translation keys, listed at the bottom of the
`<px-upload-dialog>` table in the design tool translation reference. Announced on the Notion
Dashboard 2026-08-17; not verified live.

---

## Editor CSS Customization

The editor can be re-themed with CSS. Where the CSS lives depends on deployment:
- Full Pixfizz / Shopper: the `editor.css` page.
- Shopify integration: the `shopify/custom-styles` snippet (loaded into the editor
  from the Pixfizz side, not the Shopify theme).
- Either path: a file named in the **Custom CSS** field of the Design Tool
  Configuration (see below).

Storefront `style/custom.css` does NOT reach inside the editor iframe. Use one of
the locations above instead.

**The Custom CSS field is a list of files, not a CSS text box** (Corrected 2026-10-06).
Its admin help text: "A list of custom CSS files to inject into the Editor. Each file can be
given as an absolute URL, name of a CMS page, or name of an asset wrapped in @ characters."
With the field empty, no custom stylesheet is loaded, even when the `editor.css` page
exists. On a Shopper child the working pair is:

1. an Override Snippet of `style/editor.css` (the parent's `editor.css` page renders it at
   `/site/editor.css`), and
2. `editor.css` in the configuration's Custom CSS field.

The editor shell then carries `<link rel="stylesheet" href="https://<site>/site/editor.css">`.
Iterate by editing the override and reloading the editor. Roll back by deleting the override
(the parent content returns) and clearing the field. *Verified by query on
experience.pixfizz.com (shopper24 child), 2026-10-02. Whether the Shopify
`shopify/custom-styles` route also needs naming in the field is not verified.*

Reusable techniques confirmed in production:

- Variable aliasing. The editor exposes internal CSS custom properties (for
  example `--bright-sky-blue`, `--seaweed`). Repointing them to the brand palette
  re-themes the whole editor without targeting individual elements.
- Asset references. `@filename@` (an uploaded asset name wrapped in @ signs) is how the
  Custom CSS field names an asset file to inject (Corrected 2026-10-06). The earlier note
  here called it an in-CSS URL helper that the platform replaces at render time (seen as
  `@Beatrice-Regular.woff2@` for a font). Unconfirmed: whether `@name@` inside CSS text is
  also replaced. See § Asset references below.
- Button classes. Editor buttons carry both legacy classes (`px-blue`, `px-green`)
  and newer classes (`px-primary-color`, `px-secondary-color`). Target both to
  reskin all buttons reliably.
- Tab show/hide. Tabs can be hidden via their `data-id` attribute plus `display:none`.
- Option reordering. Options can be reordered with `data-option-code` plus the CSS
  `order` property on a flex container.
- Gallery captions. Gallery folders use `px-gallery-item px-gallery`; individual
  images use `px-gallery-item px-image`. To hide image filenames while keeping
  folder names visible:

```css
.px-gallery-item.px-image .px-caption {
	display: none;
}
```

- Action button overlay gotcha. A `.btn::after` pseudo-element set to
  `position: absolute` creates an invisible overlay that swallows clicks and
  blocks hover states. Action buttons can sit in different containers
  (`px-action-buttons-container`, `px-edit-buttons-container`,
  `px-reset-button-container`, `px-controls-container`), so widen selectors across
  all relevant containers when styling them.
- Default tab on load (JS). `editor.store.ui.expandTab` sets which tab is open when
  the editor loads.
- Save and Continue callback (JS). The Save and Continue button can be
  monkey-patched to chain an action after the original behavior, using an
  `exit_target` URL parameter to control where the user lands.
- Placeholder icon position — avoid `transform` on `.px-element-icon`. Setting a
  CSS `transform` on `.px-element-icon` breaks the position of the placeholder
  (upload) icon inside image zones. To restyle that icon, use properties that do
  not create an offset/stacking context (size, color, background) rather than
  `transform`.
- Layout category order. Layout categories in the designer sort **alphabetically
  by default**. They can be re-ordered visually with the CSS `order` property on
  the category container (same flex-`order` idiom as option reordering above); the
  underlying order is not configurable in admin.

### Theming variables and selectors (editor bundle 20261001104917)

Read from the live DOM and `editor_bundle.css` on desktop. *Verified by query, 2026-10-02,
unless marked.*

- **Variables:** `--editor-selection-color`, `--mobile-editor-selection-color`,
  `--icon-background-highlight-color`, `--icon-warning-color` (`--yellow`),
  `--icon-danger-color`, `--editor-background-color`, `--primary-*`, `--secondary-*` and
  `--action-button-*`, `--gallery-sidebar-width` (276px), `--thumb-size` (96px),
  `--caption-height` (24px), `--standard-fonts` (`Satoshi, "Open Sans"`). They join the
  aliasable set above.
- **Brand color.** The configuration's brand color field (`editor_configuration[brand_color]`)
  emits an inline `:root { --brand-color }` in the editor shell, so CSS built on
  `var(--brand-color)` follows each site's own setting.
- **Selection ring.** The page SVG writes `stroke="var(--editor-selection-color)"` on
  `rect.selected`, so one variable recolors the selection box, handles and move/rotate
  discs. The same variable colors the ring on the open spread in the page strip
  (`.px-page-set[data-selected=true] .px-page-thumbs`, stock).
- **Trap: never set `border-color` on `.px-page-set`.** Its `border-right: 12px solid
  transparent` is the spacer between spreads; coloring it draws a solid block.
- **Open panel tile:** `--icon-background-highlight-color` plus
  `.px-inspector-sidebar .px-tab[data-expanded="true"]`.
- **Photo usage.** The editor renders one `<a data-page-id>` per placement inside
  `.px-gallery-item .px-image-usage` ("Used on: Page 1"), which is the stock hover overlay;
  its links already go to the page. A usage count can be drawn in CSS: `counter-reset` on
  the item, `counter-increment` on each `a`, `content: counter()` on
  `.px-gallery-item:has(.px-image-usage a)::after`, and hide `svg.px-tick-mark`. Count 1
  verified; a count of two or more not yet seen live.
- **Square gallery tiles:** `.px-thumbnail { height: 100% }` with `img { object-fit: cover }`.
- **Image source tabs** are `button[data-tab-id="device"]`, `button[data-tab-id="albums"]`,
  selected state `data-selected`.
- **Page chrome:** page shadow on `.px-page-display-page > svg.px-page`, nav arrows
  `.px-page-prev button` / `.px-page-next button`, bleed guide `svg.px-page rect.px-bleed`.
- **Warnings today:** low resolution is a `g.px-element-icon` on the element plus a
  notification when selected; bleed and safe-area are a timed notification only; text
  overflow is a notification when selected. No badge carries a number, so a DPI figure or
  a count needs script (§ Driving the Editor From a Script). Warning wording is
  translatable through `Px.t` keys.
- **`editor_bundle.css` is cross-origin:** `document.styleSheets[].cssRules` throws on it;
  `fetch()` the file to read its rules.

### The "unedited placeholders" gate is a configuration switch

`editor_configuration[unedited_warning]` on the Design Tool Configuration controls the
"You have unedited placeholders" confirmation (stock buttons "Cancel and Fix", `.px-cancel`,
and "Continue Anyway", `.px-ok`, in `.px-confirmation-modal`). With it at `0`, the cart
button went straight to the cart with an empty cover photo frame and untouched title text.
Check this field before demonstrating error handling on a site. *Verified by query (gate
off), 2026-10-02. The gate's behavior with the field on was not tested in that session.*

---
## Show in the Editor, Never Print — the Uneditable Placeholder

**Platform rule: a placeholder the customer never edits is not fulfilled.** A placeholder is an element the customer is expected to replace. If it is still untouched at order time, production drops it on the reasoning that it was meant to be edited and was not. The editor preview flags it as *placeholder missing*.

That rule gives a clean way to show something **in the editor only**:

1. Add the element (a shape is simpler than a copy of the artwork; a mid-tone at partial opacity keeps it readable without dominating).
2. Mark it **not editable** and **placeholder**.
3. Send it to the back and align it to the artwork.

Because the customer can never edit it, it is always an unedited placeholder, so it always renders in the design tool and is always removed from the production file. Typical use: a colored backing behind white text so the customer can see what they are typing, on a product that prints on a transparent or white substrate. Check it by opening the project in the editor and using the preview, which reports the placeholder as missing, or by rendering the production file from Orders → Projects.

The reverse job, **print but never show**, is a PDF layer with `visibility="fulfillment"` (cut marks, registration lines). See `19_XML_TEMPLATE_REFERENCE.md` § PDF Layers in Practice.

*Verified live in the admin design tool with a photo lab client, 2026-09-24.*

## Locking an Element: `edit="false"`

**Platform-level (Pixfizz CMS design tool).** How a locked element behaves for the customer
in the storefront editor:

- An element with `edit="false"` renders normally, but a click on it selects nothing: no
  selection box, no Edit Shape panel, no delete button. The editor gives locked elements
  `pointer-events: none`, so elements above it stay fully editable, and a locked overlay
  placed above a photo does not stop the customer selecting the photo.
- `move="false" resize="false"` alone do **not** lock an element. The same shape with
  `edit="true"` still opens Edit Shape (color, border, radius, opacity, rotation) and shows
  a delete button.
- Layout swaps work in every direction (locked to unlocked, unlocked to locked, locked to
  locked): the old layout's elements are removed and the new ones placed.
- A design import keeps `edit="false"` and the `e*` permission flags verbatim.
- **Do not combine `edit="false"` with `placeholder="true"` on artwork that must print.**
  That pair is the uneditable placeholder above, which production drops. Confirm on the
  first proof that locked backgrounds print.

The XML side (the flag list, shape opacity) is in `19_XML_TEMPLATE_REFERENCE.md` § Element
Permission Flags. *Verified by query in the storefront editor, 2026-10-02 and 2026-10-04.*

## Grouping Elements to Hide Them While Editing — View Settings

Elements can be assigned to named layers (for example `background`, `artwork`, `ribbon`) and each layer switched on or off under **View Settings** at the top of the design tool. This is only a working aid for whoever is building the design: it gets covered elements out of the way so they can be selected without nudging the element on top. It changes nothing for the customer and nothing in production. The layers themselves are declared in the XML template definition; see `19_XML_TEMPLATE_REFERENCE.md` § PDF Layers.

*Verified live in the admin design tool, 2026-09-24.*

## Editor Buttons and Per-Site CSS

- **Autofill button.** Desktop: `.px-project-gallery-panel .px-gallery-actions .px-action-buttons button[data-onclick="autofill"]` (no class of its own). The mobile editor uses a different element, `button.px-autofill`. It renders only when Autofill is on in the Design Tool Configuration, outside cut-print mode, and when the project gallery has images; it is `disabled` when every uploaded image is already placed, so a faint button usually means all photos are used. Stock styling is a low-visibility text link. `.px-gallery-actions` is flex, so `flex-wrap: wrap` plus `flex: 0 0 100%` on `.px-action-buttons` gives a full-width button. View-size toggles are `.px-gallery-actions .px-gallery-size button[data-size]`, selected state `[data-selected=true]`. *Verified by reading source; the styling recipe was tested on a mock, not yet on a live project.*
- **Where the CSS goes.** Per-site editor styling such as the autofill button goes in a stylesheet that the Design Tool Configuration's **Custom CSS** field names, with no template change. The field takes file names, not CSS text (Corrected 2026-10-06); on a Shopper child that is an Override Snippet of `style/editor.css` plus `editor.css` in the field. See § Editor CSS Customization.
- **AI photo filters** in the design tool consume AI credits billed to the merchant; the merchant can cap daily uses per customer. *Stated on a client call, 2026-09-24.*

## Driving the Editor From a Script

**Platform-level (Pixfizz editor).** Everything here uses internal editor objects. Underscore
methods are internal and a bundle update can rename them: record the bundle stamp
(`cdn.pixfizz.com/dist/prod/<stamp>/editor_bundle.js`) with every test. *Verified by query on
a shopper24 child, bundle 20261002143509, 2026-10-03, unless marked.*

### Where a script can run

- **`editor/scripts.js` is not loaded by default on a Shopper child.** The snippet renders
  inside the parent page `/site/editor-scripts.js`, and the editor loads that page only when
  a staff-only Design Tool Configuration field names it. Check the editor HTML for the script
  tag before planning a feature on that snippet. This mirrors the Custom CSS field above.
- **A same-origin iframe works without any editor hook.** `/v1/editor?book=<id>` sends
  `X-Frame-Options: SAMEORIGIN` and `frame-ancestors 'self' <site host>`, so a page on the
  same storefront host can load the editor in an iframe and reach
  `iframe.contentWindow.editor.store`. This is enough to prefill a project from a storefront
  page.

### Store paths

- Editor store: `window.editor.store`. `editor.store.ui` holds UI state
  (`expandTab`, `showNotification`).
- Project image store: `store.galleries.project` (an object with `project`, `user`,
  `clipart`, `groups`, `pdfs`, `theme_resources`). `store.galleries.image_sources` lists the
  configured upload sources. (An earlier note read it with
  `Array.from(store.galleries.values())`; that is wrong on this bundle.)
- Pages: `store.project.page_sets[].pages[]`, each with `image_elements[]`,
  `unedited_elements` and `fillable_placeholders`. An image element's
  `dpi_resolutions` is its effective `[x, y]` dpi; `store.project.minimum_dpi` is the
  floor from the definition. *Verified by query, bundle 20261001104917, 2026-10-02.*

### Autofill in a chosen order

The editor's own autofill can be driven with any photo order; gallery order does not matter.

1. Per image, in the wanted order: `item = galleries.project._addDbImage(imgJson)` gives
   id `db:<id>`; then `store.images.register(item.id, item.data)`. Skipping `register`
   fills the frames but they render blank.
2. `store.project.autofill(ids)` fills placeholders in array order.
3. Overflow: `store._autofillWithNewPages(rest)` adds spreads without a prompt.
   `store.autofill(ids)` with leftovers raises a browser `confirm()` ("Add more pages?").
4. `await store.saveProject()` persists. Read back with `GET /v1/books/<id>/pages.json`
   (20 pages per response).

- A design image without `placeholder="true"` (a typical cover photo) is skipped by
  autofill. Call `el.update({placeholder: true})` on it first.
- Text: `textElement.update({text})` persists on save.

Not verified: layout of pages added by `_autofillWithNewPages`, and logged-in projects.

Further detail, *verified by query, bundle 20261002143509, 2026-10-04*:

- `store.project.autofill(ids)` fills in page order and **returns the ids it could not
  place**; pass those to `store._autofillWithNewPages(rest)`. A placeholder is fillable when
  `is_editable_master_element && replace && (placeholder || id === null)`.
- In a hidden same-origin iframe, wait for `loaded` and `galleries.project.loaded` before
  touching the store. It works even when the browser tab itself is hidden.
- **Only the project's own gallery feeds the editor tray.** Images in the project gallery
  (`GET /v1/books/<id>/gallery.json`) appear in the tray for that project. `_addDbImage`
  plus `saveProject()` does **not** add an image to the project gallery on the server: the
  image can be placed on a page but is missing from the tray on reopen. There is no
  server-side copy between galleries; upload the images to the project gallery with
  `POST /upload/image?gallery_id=` (`61_PIXFIZZ_API.md`).
- **Creating a project by posting the product page's `#project_create` hidden inputs needs
  the `book[pages]` select on every product's page.** One product lacked it until fixed;
  check every product in the flow, not just one.

## Mapped Previews — What "Use Mapped Preview" Actually Is

A mapped preview is not a full 3D model. Each entry in the print product's mapped previews pairs a background photo (`bg_url`) with a small GLB mesh (`glb_url`, served from `/fz/...` with open CORS): a partial surface plus a camera, used to warp the flat production artwork onto the photo at render time. The mapped previews travel with the template export as `glb_files/`, referenced from `__print_product.yml` by `mapped_preview` and `glb_blob_hash_key`. `GET /v1/themes/<id>/preview.<ext>?...&preview_section=left|center|right` returns the rendered composite for a named preview section; `template_name=<page>` returns the flat production page. A `.glb` cannot be uploaded as an ordinary site asset; the mapped-preview upload on the print product is the route that accepts it.

The older per-size "preview section" configuration used for mug previews (arc, rotation, scale per size) cannot be copied between sizes and has to be rebuilt by hand when a size is missing it. It is being replaced by 3D Preview: see `27_LIVE_FINISH_AND_3D_PREVIEWS.md`. A template copied from another size can carry that size's mapped-preview GLB hash keys and render on the wrong model; compare `glb_blob_hash_key` values across sizes. *Verified by reading source, 2026-10-05.*

*Verified by query (a live mug product, 2026-09-24); the legacy-mechanism note is stated by Alex, 2026-09-22.*

## Zero-Width or Zero-Height Shapes Corrupt PDF Output

A shape element with a width or height of 0 causes `NaN` to be written into the generated PDF,
producing a corrupt output file. The platform no longer writes NaN for zero-size shapes (fix
deployed Aug 2026), but any zero-width/height shapes already present in live projects should be
removed manually.

**Rendering difference by viewer:**
- Chrome PDF viewer: renders a 1px black line
- Acrobat and Firefox: render nothing

**Source:** #development Slack, Matjaz, 2026-08-17.

---
## Font Licensing: Editor Fonts Require Embedding License

Fonts used inside the Pixfizz editor are **embedded into personalised product renders** (print-ready files, previews). This requires a **digital embedding** or **print embedding** license — not a standard web font license.

Many foundries do not offer embedding licenses, or charge significantly more for them. A foundry that sells a web font license may explicitly prohibit embedding in rendered output — this is the most common reason client font purchases fail.

When advising clients on font sourcing for the editor:
- Describe the use case as: "fonts loaded into a web-based product personalisation editor, embedded into rendered output (print-ready files or customer previews)"
- Recommended sources:
  - **Paratype** (paratype.com) — strong Cyrillic catalogue, commercially experienced, generally supports embedding
  - **MyFonts** (myfonts.com) — look for Desktop / Digital Ad / App license tiers, which typically cover embedding scenarios; confirm with their support before purchasing
- Do not recommend Google Fonts for editor use: Google Fonts licenses permit web use but do not cover embedding in commercial rendered products

Note: this is separate from fonts used on the storefront (navigation, product names, body text) — those only require a web font license.

---

## Element Substitution Types

Element substitutions re-style a design's elements per template/option without editing the design itself. Alongside the existing types (element color, blend mode), the following were added in June 2026:

### Shape border substitutions

Three substitution types target shape element borders:

- **Shape border width**
- **Shape border color**
- **Shape border radius**

### Image effects substitution

A substitution type named **Image effects** applies a filter to image elements. Supported filters are **grayscale** and **sepia**.
Sepia is the standard CSS `sepia` filter. *Stated by the core developer, 2026-09-28.*

- Configuration gotcha: set the substitution's **Name** field to `placeholder`. An earlier build where this was misconfigured threw an application error in the design tool (Canvas and More views) that broke the whole design. The `placeholder` Name value is the correct, confirmed configuration.

### Admin preview and embedded inline page behavior

Element substitutions now run on all admin page previews and on embedded inline pages. Previously, admin previews skipped element substitutions unless `fulfillment=true` was set, so a preview could look different from the actual customer-facing/fulfillment render. Deployed 2026-07-31. Source: slack-message (#development), commits dc282140, b406e757, 55667a2c.
### Substitutions bind by element name

A substitution targets an element by its name (for example `standoffs`, `placeholder`). A variant set exported from one product and imported onto another only acts where the target template's pages carry elements of those names, and any image it swaps in (a size-specific drawing) was made for the source template's geometry. Read the target template's element names before importing a variant set with substitutions. *Verified by reading source (a variant export), 2026-09-29; the no-match behavior is inferred, not verified by render.* See also `19_XML_TEMPLATE_REFERENCE.md` § Preview Sets for the background element name color substitution binds to.

**Name and tags.** In the editor a substitution key has the form `name@[tag1,tag2]`: an element matches when it has that name (if one is given) **and** every listed tag. Tags come from the element's `tags` attribute (a comma list). *Verified by reading source (editor bundle 20261002143509), 2026-10-06. Server-side matching by tags is not verified.*

### Image crop flag (`image_crop_flag`)

**An image upload option crops the customer's image to fill, whatever the element says.** The upload option stores `db:<id>`, with any crop props appended as `db:<id>@{l:..,t:..,z:..,r:..,crop:0,...}`. With no props, the server crops to fill even when the page XML element carries `crop="false"`. *Verified on baseline.pixfizz.com (server preview, design tool, saved project page XML), 2026-10-05.*

The fix that travels with the template: a hidden multiple-choice template option with one default value (for example `fit`), whose value carries an **image_crop_flag** element substitution with Cropping Enabled off, targeting the element name or element tags. Wide, tall and transparent logos then fit whole, and the saved project page carries `crop="false"` with the customer image, which is what production reads. A value written as `db:<id>@{crop:0}` also works. *Verified on baseline.pixfizz.com by server render and saved page XML, 2026-10-06; the production PDF was not rendered.*

- In the editor, image_crop_flag sets crop on or off; the "off" value also resets left, top and zoom to 0. *Verified by reading source, 2026-10-06.*
- image_crop_flag is offered only on option values and Website substitutions. An image upload option has no substitution panel (the new-substitution form returns 500 for it); color options offer only color types; text options offer only text and `qrcode_content`.
- Admin route: `GET /admin/element_substitutions/new?owner_type=TemplateOptionValue&owner_id=<value id>&substitution_type=image_crop_flag`, fields element name, element tags, content (checkbox, `0` = cropping off).
- Crop Aspect Ratio and the customer's crop box: `22_OPTION_VARIANT_RENDERING.md` § 5.1. Its interaction with image_crop_flag is not tested.

### Color options and multi-element targets

Color option behavior, comma-separated Target Element Names, the 255-character `target_element_name` limit and the full substitution type list: `22_OPTION_VARIANT_RENDERING.md` § Template Option Substitutions. One comma-separated image target reached 43 tiles by server render on baseline.pixfizz.com, 2026-10-05.

### Known issue: colour substitutions import as black

When a template with designs is exported and imported into another site, colour element substitutions arrive as **black** rather than the assigned colour. All other substitution data comes across.

Check and reset every colour substitution manually on the destination site after any template export/import. Do not assume the values carried over because the substitution records themselves are present.

### Known issue: element substitutions on the photo prints interface

Element substitutions do not apply correctly on the newer bulk photo prints interface.

A related symptom is white borders on prints that should be borderless (or the reverse). The cause is the **layout switch resetting the crop** — moving between a bordered and borderless layout re-runs the crop and discards the previous state.

Workaround until this is fixed: organise print products into two separate categories, **with borders** and **without borders**, so the customer never switches layout mid-flow.

---

## Page Border Radius — Bleed and Margin Guides

There is no native page border-radius setting in the editor. Where a designer applies rounded corners to a page, the **bleed and margin guide lines remain square** — they are drawn against the rectangular page bounds, not the visible rounded shape.

This is a display limitation of the guides, not a production problem: the actual bleed and margin values are unaffected.

Workaround for a genuinely rounded page appearance: use **page masks** rather than attempting a border radius.

---

## Known Issues — Mobile

### Multi-page products: no indicator of which page is being edited

On mobile, the card editor gives no clear indication of which page (e.g. front vs. inside) is currently being edited — the same view is clear on desktop. This has been reported via support ticket and is not yet resolved; no fix has been confirmed as of 2026-07-31. Source: support ticket #18343.

**Update 2026-08-20:** a CSS-level workaround now exists — a persistent bottom page strip, in § Mobile Editor CSS below. The stock prev/next hints are decorative and cannot be turned into navigation from CSS. The underlying platform fix is still outstanding.

## Element Substitutions — Opacity Control

Element opacity can be controlled through element substitutions, enabling opacity values to be
driven by product option selections or other configuration values. Useful for calendar and
photo book products where transparency effects need to vary by option choice.

SOURCE: Fireflies call (Shaun Bowen / Rapid Studio, 2026-08-18). Confirmed live: 2026-08-21.

## Editor Gallery Folders — Per-Tag Theming

Verified 2026-08-25 by live DevTools inspection of an editor Clipart tab.

Gallery folders carry the **tag name** in `data-gallery-id`:

```html
<div class="px-gallery-items" data-item-size="medium">
  <div class="px-gallery-item px-gallery" data-gallery-id="Hearts" data-onclick="onGalleryClick">
    <div class="px-thumbnail">
      <svg viewBox="0 0 320 320"><path fill="currentColor" ...></svg>
    </div>
    <div class="px-caption" data-px-tooltip="Hearts">Hearts</div>
  </div>
</div>
```

Three facts worth having:

- **`data-gallery-id` is the literal tag name.** Every clipart tag folder is
  individually targetable — `[data-gallery-id="Hearts"]` — with no reliance on
  child order. This retires the `:nth-child()` / alphabetical-order workaround
  documented for layout categories.
- **The stock folder graphic is an inline SVG using `fill="currentColor"`.** It
  is not a missing thumbnail and there is nothing to set in admin. It recolours
  from a single `color` declaration on `.px-thumbnail`, which is the cheapest
  per-tag treatment available.
- **`.px-caption` also carries the tag** in `data-px-tooltip`, so captions are
  targetable per tag independently of the tile.

Ancestors: `.px-gallery-panel > .px-gallery-items > .px-gallery-item.px-gallery`.
`.px-gallery-items` carries `data-item-size` (`medium` observed), which appears to
be the small/medium/large view toggle — **inferred from the control above the
grid, not confirmed in the bundle.**

### Replacing the folder glyph with a per-tag image

```css
/* ===== START: Clipart folder thumbnails ===== */
.px-gallery-item.px-gallery[data-gallery-id="Hearts"] .px-thumbnail {
	background-image: url({{ website.assets['clipart-hearts.webp'] | asset_url: 256, format: 'webp' }});
	background-repeat: no-repeat;
	background-position: center;
	background-size: cover;
}

/* visibility, not display -- the SVG is what gives .px-thumbnail its height,
   so display:none collapses the tile. */
.px-gallery-item.px-gallery[data-gallery-id="Hearts"] .px-thumbnail svg {
	visibility: hidden;
}
/* ===== END: Clipart folder thumbnails ===== */
```

- **Background image, not a `::after` pseudo-element** — the action-button overlay
  gotcha in this section applies here too.
- **`background-size: cover` survives the view-size toggle.** Use `contain` for
  line art or logos that must not crop.
- **It fails gracefully.** Renaming a tag in admin stops the selector matching and
  the folder reverts to the stock glyph. Comment the block with the tag names it
  depends on, because nothing else records the coupling.
- The `visibility` versus `display` choice is reasoning, not a tested result —
  worth confirming on a site with a non-square tile.

**Open, and worth closing before using this on a live site:** the Galleries tab
almost certainly renders folders with the same classes and its own
`data-gallery-id` values, so a customer photo gallery sharing a name with a
clipart tag would pick up the clipart styling. Nobody has inspected the Galleries
tab DOM to confirm whether the two panels are distinguishable by an ancestor
attribute. If one exists, scope every recipe above to it as standard.

### Two more editor CSS custom properties

Read from the computed rule for `.px-caption` in `editor_bundle.css`:

```css
.px-gallery-panel .px-gallery-items .px-gallery-item .px-caption {
	color: var(--neutral-grey-2);
	height: var(--caption-height);
	line-height: var(--caption-height);
}
```

`--neutral-grey-2` (caption text) and `--caption-height` (caption row height) join
`--bright-sky-blue` and `--seaweed` in the aliasable set. Caption colour and row
height are the two things a lab most often wants to change in a gallery panel.

### Asset references — which syntax belongs where

The `@filename@` wrapper is the fallback for the one location that is **not**
Liquid-rendered, not the house style for all editor CSS.

| Location | Liquid? | Asset reference |
|---|---|---|
| `editor.css` page (Full Pixfizz / Shopper) | Yes — CMS page | `asset_url` |
| `shopify/custom-styles` snippet (Shopify) | Yes — CMS snippet | `asset_url` |
| Custom CSS field on the Design Tool Configuration | **No**, and it holds file names, not CSS (Corrected 2026-10-06) | `@filename@` names an asset file to load |

**Status.** The Custom CSS field row is verified from the field's admin help text
(2026-10-02). Still unconfirmed: that a Liquid `asset_url` call in `editor.css`
renders as a resolved URL at `/site/editor.css`, and whether `@filename@` written
inside CSS text is replaced (the only evidenced use is a font). Until checked: use
`asset_url` in the Liquid-rendered locations and never mix the two syntaxes in one
block.

## Design Theme Layouts — Export Format

Verified 2026-08-26 against two real design-theme exports.

- `layouts[]` is a **sibling** of `templates[]`; layout entries carry
  `layout: true`.
- An empty layout is `<page ...></page>` with `tags: []` — that is what admin
  writes.
- **`left` and `top` are on every element and are always `0`. They are NOT the
  position.** `x` / `y` are, and they are **omitted entirely when zero**.
- Emitted attribute order is `edit height left placeholder top width x y`.
  Coordinates are **always millimetres**, whatever the definition's `unit`.
- `edit="true" placeholder="true"` is what makes a frame a customer photo slot.
- `pages=""` (empty) means the layout is offered on all pages; a value such as
  `page01,page03` targets it. Never rewrite this attribute on the user's behalf.
- The tag vocabulary is a fixed list read off the theme: `1 photo`, `2 photos`,
  `3 photos`, `4 photos`, `5+ photos`. **`5+ photos` is the catch-all** — a
  16-frame layout carries it and there is no `5 photos` string. Generating a tag
  string creates a picker group of one.

**Import behaviour.** (Corrected 2026-10-06.) A design import through the template's
Import Design button creates a **new design with a new id**, and remaps the page,
layout, image, asset and font ids carried in the tar. Layouts inside the tar get new
ids, so the earlier workaround of creating empty layouts in admin first is not
needed. A page's `layout_id` is kept only when it points at a layout already on the
site. *Verified by query (several imports re-exported and read back), 2026-09-29 and
2026-10-02.* The earlier statement here that a design-theme import overwrites by id
is withdrawn. Format, fonts, and the Linked Layouts step that an import does not do:
`19_XML_TEMPLATE_REFERENCE.md` § Design Import. `__print_product.yml` (template
import) does **not** remap `layout_id`; import creates new records there. Still open:
whether importing a design whose `code` already exists on the site duplicates it.

## Mobile Editor CSS

Verified 2026-08-20 by live inspection of a Shopify + Pixfizz client's editor plus
on-device testing. Relates to support ticket #18343.

### The mobile editor is a separate template, not a responsive desktop layout

`editor_bundle.js`:

```
this.store.ui.editor_mode === "mobile"
	? this.mobile_mode_template()
	: this.desktop_mode_template()
```

Two entirely different component trees. Consequences:

- **Mode is not width-driven.** Two `@media` rules exist in roughly 139 KB of editor
  CSS. Loading the editor at a 390 px viewport on desktop still renders
  `px-desktop-mode` — detection is device and touch, not a breakpoint. **You cannot
  reproduce mobile mode by narrowing a window.**
- **To reach mobile mode on desktop**, run `editor.store.ui.setEditorMode('mobile')`
  in the console. This is the only reliable way to develop or test mobile editor CSS
  without a device.
- The editor URL returns a ~5.5 KB shell; everything is client-rendered, so fetching
  the HTML tells you nothing about the DOM.

Class map:

| Desktop (`px-desktop-mode`) | Mobile (`px-mobile-mode`) |
|---|---|
| `.px-main-area` grid `"page-display" "page-navigation"`, rows `1fr 148px` | `.px-main-area` grid `"header" "pages" "mobile-toolbar"` |
| `.px-page-display` | `.px-mobile-page-display` |
| `.px-page-navigation > .px-page-sets` (bottom thumb strip) | `.px-mobile-page-list > .px-page-sets` (full-screen project list) |
| `.px-page-prev` / `.px-page-next` — real buttons, `data-enabled`, `goToNextSet` | `.px-prev-set-hint` / `.px-next-set-hint` — decorative only |
| — | `.px-mobile-toolbar` |

**`.px-page-navigation`, `.px-page-display`, `.px-page-prev` and `.px-page-next` do not
exist in the mobile DOM.** CSS written against them to "fix mobile" is dead code.
`.px-page-sets` does exist, but inside `.px-mobile-page-list` — the project overview
screen, not a strip.

Stock mobile navigation is: tap a page in the project list, swipe left and right
between pages, tap **Project** to go back.

### `setDimensions()` caches its measurement — the trap

`setDimensions()` measures the `.px-mobile-page-list` box into component state, and
`pageSetScale` derives from that cache. It drives **two** things: the rendered size of
the page thumbnails, and an **inline `width` written onto `.px-page-captions`**.

**It re-runs on mount and on window resize only — never when CSS changes the box.** The
list mounts in the full-height project view, so the cached scale is permanently
"full-screen page". Any CSS that shrinks the list into a strip therefore inherits a
stale scale: thumbnails render at full-screen page size and burst out of the strip, and
each caption set carries an inline width of roughly 330 px so the strip scrolls
horizontally.

Rotating the device fires a resize, `setDimensions()` re-runs against the strip, and it
snaps into place — the observed "wrong until I rotate, then perfect" symptom.

> **Rule: when CSS resizes any container the mobile editor measures, pin the child
> sizes in CSS too. Fixing only the thumbnails or only the captions looks worse than
> fixing neither.** Pinning also makes the result rotation-proof, because CSS beats the
> cached scale either way.

### Recipe — persistent bottom page strip on mobile

Verified on device. Goes in a stylesheet named in the Design Tool Configuration
Custom CSS field (the field lists files, not CSS text; Corrected 2026-10-06), the
`shopify/custom-styles` snippet, or the `editor.css` page — storefront
`style/custom.css` does not reach the editor.

Scope everything with `:has(.px-mobile-page-display[data-expanded="true"])` so it
applies only while a page is open. Without that guard, tapping **Project** leaves a
blank `pages` row with the strip stranded below. `:has()` is safe here — the bundle
already ships `.px-mobile-mode:has([data-active-tab="project->cut-print-quantities"])`.

```css
/* ===== START: Mobile editor page strip ===== */
.px-editor.px-mobile-mode .px-main-area:has(.px-mobile-page-display[data-expanded="true"]) {
	grid-template-areas: "header" "pages" "page-strip" "mobile-toolbar";
	grid-template-rows:
		calc(var(--header-height-mobile) + var(--subheader-height-mobile))
		calc(var(--page-area-height-mobile) - 96px)
		96px
		var(--toolbar-height-mobile);
}

.px-editor.px-mobile-mode .px-main-area:has(.px-mobile-page-display[data-expanded="true"]) .px-mobile-page-list {
	grid-area: page-strip;
	overflow-x: auto;
	overflow-y: hidden;
	background: #fff;
	border-top: 1px solid rgba(0, 0, 0, 0.12);
}

.px-editor.px-mobile-mode .px-main-area:has(.px-mobile-page-display[data-expanded="true"]) .px-mobile-page-list .px-page-sets {
	display: flex;
	align-items: center;
	justify-content: safe center;
	min-width: 100%;
	width: max-content;
	height: 96px;
}

.px-editor.px-mobile-mode .px-main-area:has(.px-mobile-page-display[data-expanded="true"]) .px-mobile-page-list .px-page-set {
	flex: 0 0 auto;
	padding: 4px 8px;
}

/* 1 of 2 — overrides the stale thumbnail scale. Without this the strip renders at
   full-screen page size until the device is rotated. */
.px-editor.px-mobile-mode .px-main-area:has(.px-mobile-page-display[data-expanded="true"]) .px-mobile-page-list .px-page-thumbs .px-page-wrapper .px-page {
	width: auto !important;
	height: 58px !important;
}

/* 2 of 2 — overrides the inline caption width written from the same stale scale.
   Without this each set is ~330px wide and the strip scrolls. */
.px-editor.px-mobile-mode .px-main-area:has(.px-mobile-page-display[data-expanded="true"]) .px-mobile-page-list .px-page-captions {
	width: auto !important;
	height: 16px !important;
	line-height: 16px !important;
	font-size: 10px !important;
	margin: 2px auto 0 !important;
}

.px-editor.px-mobile-mode .px-main-area:has(.px-mobile-page-display[data-expanded="true"]) .px-mobile-page-list .px-page-caption {
	flex: 0 1 auto !important;
	max-width: 76px;
	padding: 0 3px;
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}
/* ===== END: Mobile editor page strip ===== */
```

**Optional hardening, UNVERIFIED.** Only needed if a customer rotates while a page is
open: that fires a resize, the cache becomes strip-sized, and the project overview
afterwards shows tiny thumbnails. Pin the project view the same way, sized to the
viewport rather than the cache, using `width: 100%` and `height: auto` on the SVG so
the aspect ratio comes from its `viewBox`.

### The prev/next hints are decorative — do not sell them as buttons

`.px-prev-set-hint` and `.px-next-set-hint` ship at `opacity: 0` with `z-index: -1`.
`prevSetArrowStyle` / `nextSetArrowStyle` write an inline `opacity` only while a swipe
is in progress, gated on `canGoToPreviousSet` / `canGoToNextSet`; at rest the inline
style is empty, so nothing shows.

They can be forced visible with `opacity`, `z-index` and `!important` — the
`!important` is needed because of that inline style — but:

- they are `<div>` wrappers around an SVG with **no click handler**; tapping does
  nothing, confirmed by raising `z-index` and clicking;
- they carry **no enabled/disabled attribute**, so a blanket override shows an arrow on
  the first and last page too.

Do not propose them as a navigation fix. They are a swipe hint at best.

### Recommended platform changes

Two changes would retire this whole workaround:

1. **Re-measure on element resize, not just window resize.** A `ResizeObserver` on
   `.px-mobile-page-list` in place of the current mount-plus-window-resize trigger makes
   `pageSetScale` correct at all times and lets any lab restyle the mobile editor
   without fighting a stale cache. This is the root cause of the rotate-to-fix
   behaviour.
2. **Make the prev/next hints real controls.** They are already most of the way there:
   a persistent state gated on the existing `canGoToPreviousSet` / `canGoToNextSet`
   plus a click handler onto `goToPreviousSet` / `goToNextSet` closes #18343 for every
   lab at once.

## Custom Design Tools — Browser-Side Rules

Everything above this heading describes the platform Design Tool. This section covers
**custom design tools**: browser-based tools that mount on a Shopper product page, take a
customer file, and write a print file and its supporting records onto the orderline
(Sticker Designer, Gang Up, Business Cards, Cover Studio and the upload-and-price tools built
on the same shape). The rules here are platform-level unless stated otherwise — they follow
from how the CMS re-renders pages, and from how pdf.js and pdf-lib behave in a browser.

**Which tools exist, what each one does, how one is configured, mounted and installed:
`26_CUSTOM_DESIGN_TOOLS.md`.** This section is the browser-side engineering only.

---

### A dialog reparented to `<body>` survives an AJAX partial re-render, and wins

**Platform-level.** Affects every custom design tool that reparents its modal on a page the
CMS re-renders.

**Reparenting is necessary and correct.** A `filter`, `transform` or `will-change` on any
ancestor creates a containing block and traps a `position: fixed` dialog under the backdrop.
That is not a z-index problem and no z-index fixes it; moving the dialog to `<body>` does.
Every tool on this platform does it.

**The consequence nobody had drawn.** A filter control on the product page re-renders a
container over AJAX — typically the innerHTML of `<main>` — and injects a fresh tool root
carrying the new product's data attributes. That part works. But the reparented modal is
**outside** the container being re-rendered, so the swap cannot remove it. Two modals with
the same id now exist, and **both `getElementById` and Bootstrap's `data-target` resolve to
the first in document order — the stale one.** They accumulate, one per filter change.

Symptom as the shopper experiences it: change the product size, press the button, and the
tool opens built for the size they just changed away from. Refreshing the page clears it.

**The rule.** A tool that reparents its dialog to `<body>` has taken ownership of a node the
CMS can no longer see. **It must remove its own previous instance on every boot.**

Three parts of the fix, each load-bearing:

1. **Presence of an uninitialised root is the re-render signal.** Query for a tool root that
   does not yet carry the tool's own ready flag. No flags to maintain, no MutationObserver,
   no coupling to the CMS's AJAX. On first load nothing is detached yet, so the sweep is a
   no-op. Tag each modal on reparent (e.g. `data-<tool>-detached`) so the sweep can find them,
   and skip any detached node that contains the fresh root.
2. **Removing a modal Bootstrap thinks is open leaks the backdrop and the scroll lock**,
   because the teardown never runs on an element that no longer exists. After the sweep, and
   only when no `.modal.show` remains, clear the `modal-open` body class, remove the
   scroll-lock padding on `<body>`, and remove any orphaned `.modal-backdrop`.
3. **Select the uninitialised root, not by id.** With duplicates present the id resolves to
   the stale one, which is the whole bug. Have `init()` likewise resolve its own modal by
   walking up from the root rather than by id.

Nothing changes in the parent filter snippet. It drives every lab; the tool has to survive
what it does, not the reverse.

**The verification lesson.** A correct boot log is not evidence the shopper is looking at the
new instance — throughout this fault the tool booted, initialised and resolved the new size
correctly while the button opened the old board. The assertion that catches it is **which
element `data-target` resolves to, and what that one rendered**. Two further details worth
copying into any such test: lift the mount block out of the `.liquid` by regex rather than
retyping it, so the test cannot pass against a mount block that no longer ships; and perform
**two** size changes, because the failure compounds (two modals, then three) and a
single-change test understates it.

_Verified by reading source and verified live on the affected site, 2026-08-30. The same
reparenting pattern is used by the other tools; **not verified** on them — worth ten minutes
each: open a product page with an AJAX filter, change it, count
`document.querySelectorAll('.modal[id]')`._

---

### pdf.js detaches the buffer it is handed

pdf.js takes ownership of the `ArrayBuffer` passed to `getDocument` and **detaches it**. A
second `getDocument` call on the same buffer throws, and in a tool that surfaces as a generic
"something went wrong" plus a blank proof — nothing points at the buffer.

Pass a fresh copy to every call:

```js
var data = (buffer instanceof Uint8Array ? buffer : new Uint8Array(buffer)).slice();
```

Worth a regression test asserting that no two `getDocument` calls share a buffer.

_Verified by test (the guard is in shipped tool code), 2026-09-08._

---

### pdf.js and pdf-lib do not read the same page box

On any PDF where **CropBox differs from MediaBox**, pdf.js measures the CropBox and pdf-lib
places the MediaBox. Measured proof, built deliberately:

```
PDF authored with MediaBox 360 x 252 pt, CropBox 342 x 234 pt

pdf-lib embedPdf drawable size -> 360 x 252     (MediaBox)
pdf.js  getViewport({scale:1}) -> 342 x 234     (CropBox)
```

**Consequence.** A press-ready export — which is what CropBox-at-trim, MediaBox-at-bleed
looks like coming out of InDesign — measures as trim-sized in preflight and triggers a false
"no bleed" warning on a file that has perfectly good bleed. Any tool that measures with one
library and writes with the other is placing a page it did not measure.

Guessing at MediaBox from the raw bytes is worse than leaving it. **The clean fix is to
render the review image from the built print PDF itself** rather than re-deriving it from the
source file. The review screen carries the acknowledgement click, so it above all should show
the actual file rather than a reconstruction of it. That retires the whole class of defect
rather than one instance.

_Verified by test (measured on a purpose-built PDF), 2026-09-09._

---

### Browser preflight — what can and cannot be concluded

Reliable in the browser:

| Check | Notes |
|---|---|
| Page count | Reliable. Where it drives price it is mandatory |
| Page size, per page | Reliable. Report a mismatch against the product size as a **warning** |
| Mixed page sizes within one PDF | Reliable, and worth surfacing — common in scanned documents |
| Rotation | Reliable |
| Embedded raster resolution | Reliable |
| Per-page colour vs greyscale | Detectable by rasterizing at low DPI (~24) and sampling |

**Not detectable via pdf.js: colour space / CMYK.** pdf.js normalizes every fill to RGB and
content streams are usually compressed. **Report it as _unchecked_, never as "RGB".** If a
signal is genuinely needed, detect a PDF/X output intent instead.

Three companion rules:

- **Warn, do not block.** Every serious competitor warns and lets the customer proceed after
  an explicit acknowledgement. Make blocking a **per-site setting, off by default**.
- **A check that cannot conclude returns `true` or `null`, never `false`.** Never fail a
  customer's file on an inconclusive test.
- **The acknowledgement click is where liability transfers — record it**, with a timestamp,
  in the tool's own state option on the orderline.

_Verified by reading source (pdf.js behaviour) and by build, 2026-09-08._

#### `.docx` page count cannot be determined in the browser

Page breaks in a word-processor document are a **rendering result, not a document property**.
There is no honest way to read a page count from a `.docx` in the browser. Three positions,
all defensible:

1. **Accept the file and ask the customer to declare the count**, showing it as declared
   rather than detected, and marking the source as `declared` in the tool's state record.
2. **Accept and convert server-side** to PDF, then detect. Platform work.
3. **PDF only.**

**Never silently guess.** Half the document-print market runs on declared counts and every
operator that does has to publish a reconciliation warning; declaring it explicitly is what
makes that safe.

_Stated, not independently verified — position 1 is the recommendation, not a measured
platform behaviour._

#### If every hard failure comes from one check, disabling that check disables blocking

Worth asking as a design-audit question of any preflight implementation: **which findings can
produce a hard fail?** In one shipped tool every `fail`-severity finding in the whole file was
a page-count finding, so on a product with no page count configured the tool could not
produce a hard fail at all — and the site's `block-on-fail` setting, which read as `TRUE`,
had nothing to act on. Nothing was wrong with the behaviour; what was wrong was that nobody
knew blocking was inert.

_Verified by reading source, 2026-09-09._

---

### Coordinate systems — proof and pointer versus engine and writer

**The proof canvas and the pointer are canvas space** — origin top-left, y down. **The
placement engine and pdf-lib are page space** — origin bottom-left, y up.

Convert **at the engine boundary**, with a single flip function that is its own inverse, and
**pin the sign convention with a test**: drag the artwork up on screen, the engine must report
more lost off the top. Two flips scattered through the code is how a proof ends up mirroring
the customer's drag.

A manual placement rect must be **post-rotation**, with rotation decided **once**, at
preflight, and passed to the engine rather than re-derived.

_Verified by test (geometry suite against the real engine), 2026-09-09._

---

### One placement, decided once, consumed by every surface

Four surfaces draw the artwork: the **proof canvas**, the **cart thumbnail**, the **review
image** and the **print writer**. All four must render from a single resolved placement. Where
only the print writer knows the placement, the customer and the press see different things —
measured case: the customer saw their file pinned at native size over the sheet with most of
the design greyed out as off-cut, while the press received the whole design scaled to fill.
Neither party saw what the other had.

Solve the placement at preflight time, hold it on state, and have every surface render from
it. A deterministic solver may be called twice rather than caching a result — the proof runs
long before a print file exists — provided both calls are fed the same numbers.

**This explicitly corrects an earlier position.** An earlier decision held that "preflight and
proof behaviour should not fork on the output mode; only the file written to the print option
should." **That is wrong.** The proof's job is to show what will print, and what prints now
differs by mode, so **the proof must fork on the output mode**: a normalised mode draws the
placed result, while a passthrough mode keeps the own-size-plus-off-cut view, because under
passthrough that view is the truth.

_Verified live (fault observed on a real order) and verified by test (both call sites solve
to an identical rect across 7 fixtures x 5 modes), 2026-09-09._

---

### Warning copy must match the output mode

Copy written for a passthrough tool becomes false under normalised output, and it then sits
directly above a canvas showing the file placed correctly — the panel contradicting the
picture.

> "Your file is 5 x 3.5 in. This product needs 3.75 x 2.25 in including bleed."

That is right for a passthrough tool, where the customer's file **is** the print file and only
they can fix it. Under normalised output it should state what the tool will do and what it
costs — scaled to fill, roughly this much trimmed off the edges, the preview shows exactly
this — and reserve "needs fixing" for content landing outside the safe zone **after**
placement.

Two severity rules that fall out of this:

- **Amber only when the crop reaches past the bleed and into the product.** A crop the size of
  the bleed **is** the bleed. Colouring it as a defect trains people to ignore the panel.
- **Resolution must be judged at the placed size, not the file's own size.** A file reading
  "154 dpi at print size" against its own dimensions can print at 205 dpi once placed. Judge
  it after placement and say so in the copy.

Keep the acknowledgement checkbox on warnings. The liability transfer is worth keeping; the
false alarm is not.

_Verified by test (copy generated from the solver across fixtures); **not verified** on a live
page as of 2026-09-09._

---

### Browser PDF dependency pinning

For any tool loading PDF libraries in the browser:

- **pdf.js and pdf-lib from cdnjs**, each **pinned to an exact version** and carrying an **SRI
  hash**.
- The **worker URL and the cMap URL pinned to the same pdf.js version** as the library itself.
- The **pdf-lib pin kept aligned across tools** on the same site, so two tools on one page
  cannot disagree about which build is loaded.

Reference implementation in the wild: pdf.js 3.11.174 and pdf-lib 1.17.1, both from cdnjs,
both SRI-pinned, worker and cMap on the same pdf.js version.

See also `41_IMPLEMENTATION_PATTERNS_UPDATED.md` § Browser PDF Preflight for what those
libraries can and cannot read, and for the SRI-verification caveat.

_Verified by reading source, 2026-09-09._

---

### Reading the product page from a tool (Shopper 24)

- **Read the customer's option values the way the platform does:** call `values({skipInvalid: true, skipNoElementSubstitutions: true})` on every `px-option-selector` and merge the results. That is what `px-design-preview` sends to the render; the second flag leaves out variants that carry no element substitution. *Verified by reading the component source, 2026-09-26.*
- **A PDP layout size or orientation change swaps the product in place, after the `change` event.** The product is replaced by AJAX: `theme_id` in the form changes, the option markup re-renders (a template option root is replaced), the gallery element stays. A `change` handler that reads `theme_id` sees the old template; watch `theme_id` itself. *Verified live, 2026-09-26.*
- **Gallery state:** `.px-product-gallery[data-selected-idx]` is the selected slide (0-based), updated after the slide animation. A positioned element with no z-index inserted right after `.px-display` paints over the slides and under the arrows (`.px-next-arrow` has `z-index: 1`; `.px-prev-arrow` is `display: none` on slide 0). *Verified by query, 2026-09-26.*
- **Theme ids without admin:** from a storefront tab, `GET /v1/themes/<id>` returns 200 for that site's themes and 403 for another site's. Behind a PDP layout collection, fetch the collection page with `?orientation[]=<o>&size[]=<s>` and read `theme_id` from the returned form. *Verified by query, 2026-09-27.*

---

### An inline `<svg>` ignores `el.hidden = false`

`hidden` is an `HTMLElement` property. On an `SVGElement` the assignment only sets a plain
JS property and the `hidden` attribute stays, so an SVG overlay (guide lines, a hang guide)
never appears. Toggle it with `setAttribute('hidden', '')` and `removeAttribute('hidden')`.

_Verified by render, 2026-10-06._

---

### PDF pages longer than 200 in (roll goods)

- **Acrobat caps a page side at 14,400 units (200 in).** That is an Acrobat limit, not a PDF rule.
- **`/UserUnit` (PDF 1.6) is the standard way round it, but a reader that ignores it shows the page at a fraction of its size.** poppler (`pdftoppm`) renders a UserUnit 2 page of 36 x 263 in at 18 x 131.5 in. A RIP that ignored it would print and cut a half-size roll without complaint.
- **A page written at full size in points** is read at the right size by every reader tested except Acrobat's display. A RIP that refuses it fails loudly on import.
- Rule for a tool that writes its own PDF for roll goods: default to full size in points, keep UserUnit as a per-lab switch, and prove the lab's RIP with a file longer than 200 in before go-live.

_Verified by test, 2026-09-28._

---

### Build and install rules for a custom tool

Carried from shipped builds. Each of these has cost time at least once.

- **Resolve per-product configuration in the browser, not in Liquid.** A snippet mounted
  through a template option's `custom_script` may have no `product` in scope at all, and every
  Liquid fallback built on it then returns nil without error. **Emit source markers alongside
  every resolved value and emit `data-product-seen`**, and ask for that diagnostic line first
  when a tool misbehaves. A live tool reporting `productSeen: "false"` is working only because
  the site-level fallbacks happen to be right; a per-product override would be dropped
  silently.
- **DPI is read from the XML definition, never hardcoded.** A tool shipping a hardcoded target
  DPI in its asset is a large part of why one lab's output came out soft.
- **Never write a local alert.** Consume the shared notify layer — do not copy it, wrap it or
  reimplement it.
- **Never load a shared asset from inside a tool's boot block.** One parse error takes down
  the shared layer and points the symptom at the wrong file. Load the shared layer separately.
- **Load the tool's own dependencies from the product snippet**, not from a widely-overridden
  layout include, and inject with a `data-<tool>-src` marker so a re-render cannot
  double-load. See `41_IMPLEMENTATION_PATTERNS_UPDATED.md` § Custom Tool Dependency Loading.
- **Configuration rides in the mount argument list, not in custom fields.** Since 11
  September 2026 a tool reads every setting from the arguments passed to its snippet by the
  `custom_script` mount, because a template option's custom field travels with a template
  export and a product or design custom field does not. The one definition that must exist
  on the child site before the template import is `custom_script` itself, on the template
  option object. See `26_CUSTOM_DESIGN_TOOLS.md` § 4.
- **Measure the print file on a real order** — format, colour mode, alpha, pixel dimensions,
  embedded DPI. **None of them are visible in a proof.**
- **Place one real order end to end before handing over. Every defect found on this platform
  so far survived every check short of that.**

_Verified by build and by live order, 2026-09-08 / 2026-09-09._

---

### "Generic by design" and "someone forgot to fill the fields in" are different states

A tool must degrade cleanly when its specification fields are absent — no page count
configured, no trim size configured, no hard fail possible. That is correct behaviour and it
is what makes one tool serve both a tightly specified product and a generic upload product.

**But absence must not be inferred as misconfiguration.** A tool that raises

> "This title has no page count or trim size configured, so preflight can only report what it
> finds. Ask <partner> to complete the title setup."

is telling a customer their product is broken, on a product that is deliberately generic.
Distinguish the two states with an **explicit opt-out flag** — a boolean product custom field
or a site checklist key — not by inferring from absence. Then the flagged state swaps the
banner for a neutral line and leaves every other behaviour alone.

**One thing must change and it is copy, not logic.** The degrade path already works; only the
message is wrong.

_Verified by reading source, 2026-09-09._

---

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-23: Added font licensing rule for editor embedding (digital/print embedding license required, not web font license).
- 2026-05-27: Added shape color palette support and fulfillment/calendar transformation support under Shape Button toggle. Added Login Modal section — default behavior, trigger (Save & Continue only), optional links, and Shopify External Login URL setup. Also consolidated duplicate Changelog sections into one. Source: Notion Dashboard (May 2026 updates).
- 2026-06-01: Added Editor CSS Customization section, Admin Mode Editor note, and pdf_import Image Sources requirement. Source: claude-chat/slack.
- 2026-06-30: Documented element substitution types added June 2026 — shape border width/color/radius and the Image effects (grayscale/sepia) substitution, including the required `placeholder` Name-field value. Source: notion-dashboard (2026-06-22), slack-message (#development).
- 2026-07-04: Documented the `&aitools=true` URL flag that exposes the editor AI image tools. Source: slack-message (#development).
- 2026-07-20: Corrected Image Sources to the full allowed value set and noted it controls icon order and defaults to device. Added Google Photos setup. Source: help-article + admin tooltip.
- 2026-07-20: Added editor-CSS gotchas — `transform` on `.px-element-icon` breaks the placeholder icon position; layout categories sort alphabetically by default and can be reordered with CSS `order`. Source: slack-message (#development), loom-video.
- 2026-07-25: Clarified that a design tool configuration is assigned to a Template or a Design (not to a product or category) and that several configurations can run on one site. Added the confirmed seven-style AI restyle launch set (Vintage Film excluded) with per-lab billing and daily limits. Added per-configuration help modal snippets via Trip JS (including mobile fluid-dimension rule for Trip blocks). Added two known issues — colour element substitutions import as black after template export/import, and element substitutions failing on the bulk photo prints interface with white-border symptoms caused by the layout switch resetting the crop (workaround: split print products into with-borders / without-borders categories). Added page border-radius limitation: bleed and margin guides stay square, use page masks. Source: slack-message (#development), fireflies-call, loom-video, claude-chat.
- 2026-07-31: Documented AI restyle/filter auto-apply on selection (fixes filter loss when going to cart without pressing Apply). Added known issue: AI token usage counted globally instead of per site, fix verified on staging and pending production deploy. Documented that element substitutions now run on all admin previews and embedded inline pages (previously skipped unless `fulfillment=true`). Added known issue: no front/inside page indicator in the mobile card editor. Source: slack-message (#development), support ticket.
- 2026-08-29: Added Editor Gallery Folders — per-tag theming via `data-gallery-id` (the literal tag name), the `currentColor` inline-SVG folder glyph, the `data-px-tooltip` caption hook, a per-tag thumbnail recipe, and the open question of whether the Galleries tab is distinguishable from Clipart. Added `--neutral-grey-2` and `--caption-height` to the aliasable variable set. Clarified that `@filename@` is the fallback for the non-Liquid Design Tool Configuration Custom CSS field, while `editor.css` and `shopify/custom-styles` are Liquid-rendered and take `asset_url` — marked inferred pending two checks. Added Design Theme Layouts export format (layouts as a sibling of templates, `left`/`top` always 0 with `x`/`y` omitted when zero, mm coordinates, `edit`+`placeholder` photo slots, the fixed tag vocabulary with `5+ photos` as the catch-all) and what is and is not verified about layout import. Source: claude-chat (live editor inspection, photobook layout build).
- 2026-08-29: Added Mobile Editor CSS — the mobile editor is a separate template chosen by device and touch detection rather than a breakpoint (so it cannot be reproduced by narrowing a window; use `editor.store.ui.setEditorMode('mobile')`), the desktop/mobile class map, the cached `setDimensions()` measurement that makes any CSS resize of the page list render at a stale scale until the device is rotated, a verified persistent-bottom-page-strip recipe, why the prev/next hints are decorative, and the two platform changes that would retire the workaround. Cross-referenced from the Known Issues — Mobile entry for #18343. Source: claude-chat, live editor inspection, on-device testing.
- 2026-09-09: Added Custom Design Tools — Browser-Side Rules. A dialog reparented to `<body>` survives an AJAX partial re-render and wins document-order resolution, so a tool that reparents must sweep its own previous instance on every boot (with the backdrop and scroll-lock cleanup, the uninitialised-root selector, and the verification lesson that a correct boot log proves nothing). pdf.js detaches the buffer it is handed, with the copy guard. pdf.js and pdf-lib do not read the same page box — measured CropBox versus MediaBox proof, the false no-bleed consequence, and rendering the review image from the built print PDF as the clean fix. Browser preflight — what is reliable, that colour space/CMYK is not detectable via pdf.js and must be reported unchecked, warn-do-not-block, a check that cannot conclude returns true or null, the acknowledgement click as liability transfer, `.docx` page count as undeterminable in the browser, and the audit question of whether one check carries every hard failure. The canvas-space versus page-space coordinate rule with a single self-inverse flip and a pinned sign convention. One placement decided once and consumed by proof, cart thumbnail, review image and print writer — explicitly correcting the earlier decision that the proof should not fork on output mode. Warning copy must match the output mode, with the amber-only-past-the-bleed and judge-resolution-at-placed-size rules. Browser PDF dependency pinning. Build and install rules for a custom tool, ending in placing one real order end to end. And that generic-by-design must be an explicit opt-out flag rather than inferred absence. Source: claude-chat, fireflies-call.
- 2026-09-16: Added login inside `<px-upload-dialog>` — four attributes, the underscore form for PhotoPrints config and the `upload-dialog-` prefix for `<px-image-upload>` / `<px-multi-image-upload>`; off by default. Missed in the 2026-08-17 sync. Source: notion-page (Dashboard).
- 2026-09-19: Pointed the Custom Design Tools section at the new `26_CUSTOM_DESIGN_TOOLS.md` for the estate, configuration and install. Replaced the "create the two shared custom fields before the template import" build rule with the mount-argument-list rule decided 11 Sep 2026. Source: kbsync (custom tool estate).
- 2026-09-24: Added "Show in the Editor, Never Print" (unedited placeholders are not fulfilled; the uneditable-placeholder technique), "Grouping Elements to Hide Them While Editing — View Settings", "Editor Buttons and Per-Site CSS" (autofill selector and states, the Design Tool Configuration Custom CSS field, AI filter credits) and "Mapped Previews". Source: fireflies-call, claude-chat, slack-message.
- 2026-09-29: Sepia is the CSS sepia filter. Substitutions bind by element name. Reading option values, PDP layout swaps and gallery state from a tool; PDF pages over 200 in. Source: claude-chat, slack-message.
- 2026-10-06: Editor CSS Customization: corrected the Custom CSS field to a list of files (URL, CMS page name, `@asset@`), not CSS text, with the Shopper child pair (`style/editor.css` override plus `editor.css` in the field); corrected the `@filename@` notes and the asset-reference table; added theming variables and selectors (brand color, selection ring, `.px-page-set` spacer trap, usage count, warnings) and the `unedited_warning` gate switch. Added Locking an Element (`edit="false"`). Added Driving the Editor From a Script (`editor/scripts.js` not loaded by default on a Shopper child, same-origin iframe, store paths, ordered autofill, the autofill return value and fillable rule, only the project gallery feeds the tray, `#project_create` needs `book[pages]` on every product). Element Substitution Types: name-and-tags matching, the image crop flag (image upload options crop to fill), a pointer to 22 for color options and multi-element targets. Mapped Previews: pointer to 27 and the copied GLB hash key trap. Corrected Design Theme Layouts import behaviour (a design import creates a new design and remaps ids). Custom tools: inline SVG ignores `el.hidden`. Source: claude-chat, vault-doc.
