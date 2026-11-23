# Live Finish Plates: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.5.0 live; 0.6.0 built for the Jewelry Box, not on the parent yet.

As recorded on 3 Nov 2026: Status: live, version 0.3.0 on the shopper24 parent (0.4.x in the CHANGELOG is a prototype, not installed). Snippet: `product/live-finish-plate`. Mount option: `lfp_mount` (hidden Text template option, custom_script field). Prefix: `lfp_`.

## Changes since 3 November 2026

- 0.6.0: a side view per plate (`lfp_turn`, `lfp_turn_images`) with a Turn button. Not offered by the standard `product/live-finish-plate` snippet yet; only the Jewelry Box uses it.

*Checked against the CHANGELOG and docs/releases on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

A companion to Live Finish for products that need a real photo: it places the shopper's design onto product photos (plates) such as a mousepad on a desk or a T-shirt on a hanger, one view per plate. It writes nothing to the order.

Example mount:
```liquid
{% snippet 'product/live-finish-plate',
	lfp_page: 'wall-decor',
	lfp_plates: 'mousepad-studio,mousepad-desk',
	lfp_views: 'Studio,On the desk',
	lfp_sample: 'lf-plate-sample-dog.jpg',
	lfp_label: 'Live Finish',
	lfp_off_label: 'Standard view',
	lfp_light_label: 'Move the light',
	lfp_sample_note: 'Sample photo. Upload yours to see it here.',
	lfp_render_width: '1200' %}
```

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lfp_page` | Production page rendered onto the plate | a page name in the template | none, preview off (unless `lfp_pages` is set) | **With care**: must name a real page of the XML template |
| `lfp_pages` | One page per design, for a plate showing several designs (a set of coasters). Replaces `lfp_page` | comma list of page names | `lfp_page` | **With care**: must name real pages |
| `lfp_plates` | The plates, one per view | comma list of plate names | none, preview off | **With care**: each plate is two Pixfizz-made assets (`lf-plate-<name>` photo and map); only use plate names that exist on the parent |
| `lfp_views` | Label of each view, same order as the plates | comma list of text | the plate names | **Yes** |
| `lfp_sample` | Sample photo shown before an upload; a comma list gives one per design | asset name(s) or path(s) | none | **Yes**: upload your own asset first |
| `lfp_label` | Button that switches the preview on | text | `Live Finish` | **Yes** |
| `lfp_off_label` | Button that switches back to the platform preview | text | `Standard view` | **Yes** |
| `lfp_light_label` | Close-up hint (not shown on fabric) | text | `Move the light` | **Yes** |
| `lfp_sample_note` | Note over the sample photo | text | `Sample photo. Upload yours to see it here.` | **Yes** |
| `lfp_render_width` | Render width in px | number, platform caps at 1200 | `1200` | **Yes** |
| `lfp_box` | Print size in inches, only for plates drawn at the template size (T-shirts, wall stickers) | `WxH`, for example `16x20` | the plate's own print box, with a console error | **With care**: must match the printed size of that template |

Things that are not set here
- Where the design sits on the photo. That is built into each plate's map image by Pixfizz.
- New plates. A plate is a photo plus a map made by Pixfizz; ask Pixfizz for one.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
