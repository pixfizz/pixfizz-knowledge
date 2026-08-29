# 17 — Design Tool

**Authority Scope:** Design Tool Configurations, feature toggles, and customer-facing editor behavior.

_Last updated: 2026-07-31_

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

---

## Editor CSS Customization

The editor can be re-themed with CSS. Where the CSS lives depends on deployment:
- Full Pixfizz / Shopper: the `editor.css` page.
- Shopify integration: the `shopify/custom-styles` snippet (loaded into the editor
  from the Pixfizz side, not the Shopify theme).
- Either path: the Custom CSS field in the Design Tool Configuration.

Storefront `style/custom.css` does NOT reach inside the editor iframe. Use one of
the locations above instead.

Reusable techniques confirmed in production:

- Variable aliasing. The editor exposes internal CSS custom properties (for
  example `--bright-sky-blue`, `--seaweed`). Repointing them to the brand palette
  re-themes the whole editor without targeting individual elements.
- Asset URL syntax. Inside editor CSS, wrap an uploaded asset filename in @ signs
  (for example `@Beatrice-Regular.woff2@`). The platform replaces it with the full
  asset path at render time. This is the editor equivalent of the storefront
  asset_url convention.
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

---
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

- Configuration gotcha: set the substitution's **Name** field to `placeholder`. An earlier build where this was misconfigured threw an application error in the design tool (Canvas and More views) that broke the whole design. The `placeholder` Name value is the correct, confirmed configuration.

### Admin preview and embedded inline page behavior

Element substitutions now run on all admin page previews and on embedded inline pages. Previously, admin previews skipped element substitutions unless `fulfillment=true` was set, so a preview could look different from the actual customer-facing/fulfillment render. Deployed 2026-07-31. Source: slack-message (#development), commits dc282140, b406e757, 55667a2c.

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
| Custom CSS field on the Design Tool Configuration | **No** — admin field on the config record | `@filename@` |

**Status: inferred, NOT confirmed.** Two checks close it: paste a Liquid
`asset_url` call into `editor.css` and confirm the rendered stylesheet at
`/site/editor.css` carries a resolved URL rather than literal Liquid; and paste
`@filename@` into the Design Tool Configuration Custom CSS field and confirm it
resolves for an **image** — the only evidenced use of that syntax is for a font.
Until both are checked: pick one location and use its own syntax, never mix the
two in one block.

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

**Import behaviour.** A design-theme import overwrites by id — verified.
`__print_product.yml` does **not** remap `layout_id`; import creates new records
there. Whether a blank or invented **layout** `id:` creates a new record is **not
verified** (blank `id:` is documented as accepted at template, template-option and
print-theme level, but nobody has proven it one level down). The workflow that
sidesteps the unknown: have the site owner create N empty layouts in admin and
export, so every blank carries a real platform id before the re-import. Also open:
whether re-importing a design theme whose `code` already exists updates in place
or duplicates.

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
