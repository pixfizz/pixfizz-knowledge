# 17 — Design Tool

**Authority Scope:** Design Tool Configurations, feature toggles, and customer-facing editor behavior.

_Last updated: 2026-06-30_

---

## What is the Design Tool?

The Design Tool is the interactive editor customers use to personalize products. It is embedded in the storefront and provides a visual interface for uploading images, adding text, applying layouts, and previewing the finished product.

The Design Tool is highly configurable through **Design Tool Configurations**.

---

## Design Tool Configurations

Each Pixfizz environment can have one or more Design Tool Configurations. A configuration controls the design tool's appearance and available features for a specific context. Templates reference a configuration, so different product types can offer different design experiences.

Configured in admin under: **Settings > Design Tool**.

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

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-23: Added font licensing rule for editor embedding (digital/print embedding license required, not web font license).
- 2026-05-27: Added shape color palette support and fulfillment/calendar transformation support under Shape Button toggle. Added Login Modal section — default behavior, trigger (Save & Continue only), optional links, and Shopify External Login URL setup. Also consolidated duplicate Changelog sections into one. Source: Notion Dashboard (May 2026 updates).
- 2026-06-01: Added Editor CSS Customization section, Admin Mode Editor note, and pdf_import Image Sources requirement. Source: claude-chat/slack.
- 2026-06-30: Documented element substitution types added June 2026 — shape border width/color/radius and the Image effects (grayscale/sepia) substitution, including the required `placeholder` Name-field value. Source: notion-dashboard (2026-06-22), slack-message (#development).
- 2026-07-04: Documented the `&aitools=true` URL flag that exposes the editor AI image tools. Source: slack-message (#development).
- 2026-07-20: Corrected Image Sources to the full allowed value set and noted it controls icon order and defaults to device. Added Google Photos setup. Source: help-article + admin tooltip.
