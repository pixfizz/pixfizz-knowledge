# Creative Studio (Make your own notebook cover): tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.1.5 live on the Shopper parent (5 Nov 2026); 0.1.8 built.

As recorded on 3 Nov 2026: Status: live, version 0.1.2 (as of the Stationery Studio CHANGELOG). Snippet: `product/creative-studio-assets`. Mount: none of its own. Prefix: `crs-` (CSS only).

*Not re-checked on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

Runs inside Stationery Studio on notebook pages: the shopper builds a notebook cover from drawings, a layout, a name and colors.

There are no settings for Creative Studio itself. It is switched on per notebook template in the Stationery Studio mount:
```liquid
	ss_creative: 'true',
	ss_bleed: '0.125'
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `ss_creative` (Stationery Studio mount) | Show the Make your own card on this notebook | `true`, `false` | `false` | **With care**: the collection must hold `pxt-notebook-in-<size>-creative` |
| `ss_bleed` (Stationery Studio mount) | Bleed of the Make your own print file, in inches | A number in inches | `0.125` | **With care**: must equal the bleed in the template XML definition |

Things that are not set here:
- Size, binding, pages, quantity and the cart: Stationery Studio and the product variants.
- The drawing library: built into the tool.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
