# Cover Studio: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: in test, version 1.8.3 (asset; style updates numbered 1.9.1, as of the CHANGELOG). Snippet: `product/cover-studio`. Mount: `{% snippet 'product/cover-studio' %}` with no arguments, in the `custom_script` field of an imprint template option on each album template (there is no `cvs_mount` option yet). Prefix: `cvs_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

A foil imprint configurator for album and book covers: the shopper picks position, lines of text, foil and font for the front, back and spine, and sees the cover before ordering. It writes option values only; there is no print file because the foil is struck from a physical die.

Cover Studio does not use mount settings yet. It still reads the older settings sources below. A move to the standard mount is planned; until then, the mount line itself has nothing to change.

Example mount:
```liquid
{% snippet 'product/cover-studio' %}
```

Where its settings live today:

| Setting | Where it is set | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|---|
| `cvs-rate-per-line` | Site checklist `admin/checklist/cvs-rate-per-line` (Custom Admin, Cover Studio settings page) | Price per line of cover imprint | A number with no currency sign | `0` | **With care**: must match the price your lab charges |
| `cvs-rate-spine` | Site checklist `admin/checklist/cvs-rate-spine` | Price for a spine imprint | A number with no currency sign | `0` | **With care**: must match the price your lab charges |
| `cvs-design-btn-selector` | Site checklist `admin/checklist/cvs-design-btn-selector` | Which page button Cover Studio holds back until the cover is set | A CSS selector, or empty | empty: finds Design now, Start design or Personalize | **With care**: only if your theme renamed that button |
| `cvs-config` | Site checklist `admin/checklist/cvs-config`, or the product custom field `tool_config` (the product field wins) | Positions, characters per line, allowed characters, fonts, foils, preview spine and board size, fallback colors, hiding the platform imprint fields, and the option codes the tool reads | JSON with the keys `positions`, `optionCodes`, `maxCharsPerLine`, `charPattern`, `fonts`, `foils`, `previewSpineIn`, `previewBoardIn`, `fallbackColors`, `hideNativeImprint` | built-in: positions `center`, `bottom-right`; 30 characters per line; foils `gold`, `silver`, `copper`, `black`, `white`, `red`, `emboss`; font Times New Roman; spine preview 1.0 in; `optionCodes` family `leather-option`, color `leather-color` | **With care**: invalid JSON falls back to the built-in defaults. `optionCodes` must match your template option and variant codes. Ask Pixfizz before editing |
| `cover_dimensions` | Product custom field | Size of one cover board (the book trim size), with its unit | For example `10x10in`, `254x254mm`, `25.4x25.4cm` | read from the size variant, then the product heading | **With care**: must match the real book trim |
| `cvs_family` | Product custom field | Cover family the preview uses | Text | `leather` | **With care** |

Never put an apostrophe in any of these values.

Things that are not set here:
- The shopper's choices (position, text, foil, font): template options on the album template, sent with the order.
- Cover material and color: the product variants.
- Album prices other than the imprint rates: the product pricing.

Not yet confirmed: The behavior of some `cvs-config` keys is not confirmed; ask Pixfizz before editing that JSON.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
