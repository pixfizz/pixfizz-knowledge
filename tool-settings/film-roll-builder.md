# Film Roll Builder: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
As recorded on 3 Nov 2026: Status: phase 1, version 0.1.1, live on the demo site. Snippet: `film/roll-builder`, mounted on a Pages page whose content is only `{% snippet 'film/roll-builder' %}`. Prefix: `frb_`.

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

No mount settings: it reads the checklist key `film-roll-builder-products` (your film product code or codes, comma separated, for example `COLOR-FILM-DEV`; must match the product code exactly) and `account-v2-film-prefs` (`TRUE` prefills each roll from the shopper's saved film preferences). Options and prices come from the product itself; a text variant with code `roll_name` on the product turns on per-roll names.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG.
