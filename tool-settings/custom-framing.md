# Custom Framing: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: built, version 0.4.0; not yet moved to the mount standard. Snippet: `product/framing`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

No mount settings yet. Settings come from product custom fields `framing_*` first, then the "Framing Settings" checklist under Custom Admin > Tools (keys `framing-units`, `framing-size-mode`, `framing-min-in`, `framing-max-in`, `framing-max-united-in`, `framing-preset-sizes`, `framing-mat-modes`, `framing-mat-min-in`, `framing-mat-max-in`, `framing-mat-default-in`, `framing-mat-weighted-bottom`, `framing-min-dpi`, `framing-dpi-action`, `framing-max-file-mb`, `framing-show-price`, `framing-room-scenes`, `framing-lab-prints`, `framing-preview-disclaimer`, `framing-preview-disclaimer-detail`, `framing-fallback-url`, `framing-fallback-message`, `framing-pricing`, `framing-config`), then built-in defaults. Mouldings are Custom Type instances; mats, glazing and finishing are variant values; rates are the `tool_pricing` JSON. Ask Pixfizz before changing `framing-pricing`, `framing-config` or `tool_config`.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
