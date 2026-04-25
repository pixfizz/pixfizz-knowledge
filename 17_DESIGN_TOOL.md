# 17 — Design Tool

**Authority Scope:** Design Tool Configurations, feature toggles, and customer-facing editor behavior.

_Last updated: 2026-04-23_

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
- Image Sources (external image providers)
- Help URL
- Custom JS — inject custom JavaScript
- Custom CSS — inject custom styles

> Some Design Tool Configuration settings are only visible to Pixfizz staff. These control platform-level behaviors and are managed during onboarding or through support requests.

---

## Changelog
- 2026-03-30: Created from master platform documentation export.

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

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-23: Added font licensing rule for editor embedding (digital/print embedding license required, not web font license).
