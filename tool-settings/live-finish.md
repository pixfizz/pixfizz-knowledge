# Live Finish: tool settings

_Part of the Pixfizz Tool Settings Reference. How the mount works, what Yes / With care / No mean and how to change a setting safely: `28_TOOL_SETTINGS_REFERENCE.md` sections 1 to 5. Template-level (Shopper) unless a row says otherwise._

_Last updated: 2026-11-24_

## Current version
0.29.0 live on the Shopper parent since 17 Nov 2026; 0.29.1 (wording in the store's language) built for one site.

As recorded on 3 Nov 2026: Status: live, version 0.26.1 (as of the CHANGELOG, live on the shopper24 parent). Snippet: `product/live-finish`. Mount option: `lf_mount` (hidden Text template option, custom_script field). Prefix: `lf_`.

## Changes since 3 November 2026

- `lf_effects` (from 0.28.0): an effect per product, for example grayscale for a black and white product. **Yes.** Value format not confirmed in this reference: ask Pixfizz for the line.
- Framed ornaments (from 0.29.0): `lf_frame_backs` (`value:asset` pairs, comma separated, or `none`) gives the back face per frame color; `lf_orn_hook` (an asset name or `none`) draws the wire hook. A mount with `lf_frame_faces` on an ornament is a framed ornament. **With care**: asset names must exist on your site.
- `lf_backdrop` accepts `<backdrop>|ribbon:<page>` and, from 0.28.1, `<backdrop>|ribbon:<page>@<theme>/<product>` to take the ribbon images from another design's page. **With care.**
- 0.29.1, not on the parent: the preview's own words follow the store language (`<html data-px-locale>` or `data-lf-lang`).

*Checked against src/product__live-finish.liquid on 24 Nov 2026.*

## Settings

Verified against the tool source on 3 Nov 2026, with later corrections dated in the changelog of `28_TOOL_SETTINGS_REFERENCE.md`. Where the section above disagrees, the section above is newer.

Shows the shopper their own design on the main product image as the finished product: the finish catching the light, the corners, the mounting, canvas sides, a frame or an ornament, and optionally on a wall in a room. It writes nothing to the order.

Example mount (a fine art paper print):
```liquid
{% snippet 'product/live-finish',
	lf_page: 'photo',
	lf_units: 'in',
	lf_sample: 'lf-sample-metal.webp',
	lf_finish_variant: 'paper-type',
	lf_looks: 'smooth:fineart-matte,textured:textured-matte,etching:etching-matte',
	lf_look: 'fineart-matte',
	lf_corner_variant: 'none',
	lf_corners: '',
	lf_radius: '0in',
	lf_mounting_variant: 'none',
	lf_mountings: '',
	lf_mounting: 'loose',
	lf_standoff_variant: 'none',
	lf_standoff_colors: '',
	lf_standoff_color: 'silver-satin',
	lf_standoff: 'diameter:0.75in,inset:1in,depth:0.5in',
	lf_render_variants: 'none',
	lf_room: 'none',
	lf_room_label: 'Change room',
	lf_view: 'closeup',
	lf_tilt: 'on',
	lf_compare: 'on',
	lf_start: 'live',
	lf_label: 'Live Finish',
	lf_off_label: 'Standard view',
	lf_light_label: 'Move the light',
	lf_sample_note: 'Sample photo. Upload yours to see it here.',
	lf_closeup_label: 'Close-up',
	lf_wall_label: 'On the wall',
	lf_compare_label: 'Compare papers',
	lf_tilt_label: 'Tilt to light',
	lf_render_width: '1200' %}
```

How values are written: most map settings are comma lists of `value:result` pairs, where `value` is the variant value exactly as it is on the product (matched exactly, then ignoring case). A value may contain spaces but never a comma, a colon or an apostrophe. Looks are `<base>-<finish>`. Bases: `white`, `clear`, `brushed`, `acrylic`, `paper`, `metallic`, `fineart`, `textured`, `etching`, `canvas`, `wood`, and for ornaments `ceramic`, `glass`, `clearacrylic`, `crystal`. Finishes: `gloss`, `semi`, `matte`, and `pearl` (canvas only). A finish alone means the white base; a base alone means gloss.

Core settings (every mount declares these; anything left out is listed in `data-config-missing` and logged as a console error):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lf_page` | The production page that is rendered and measured. The preview shape comes from this page size less its bleed | a page name in the template | none, the preview stays off | **With care**: must name a real page of the XML template |
| `lf_units` | Unit for bare numbers in `lf_corners` and `lf_radius` | `in`, `mm` | `in` | **Yes** |
| `lf_sample` | Sample photo shown before the shopper uploads | asset name, full URL, or `none` | none (the render with placeholders shows) | **Yes**: upload your own asset first |
| `lf_finish_variant` | Variant code the finish is read from. A comma list is allowed: the first listed variant on the page wins | variant code(s) or `none` | `none` | **With care**: must match the variant code on your products exactly |
| `lf_looks` | Which look each finish value shows | `value:look` pairs | none | **With care**: values must match your variant values; an unmapped value uses the first look |
| `lf_look` | Look used when no finish variant decides. `off` means no preview on a page without a listed finish variant | a look, or `off` | `white-gloss` | **Yes** |
| `lf_corner_variant` | Variant code the corners are read from | variant code or `none` | `none` | **With care**: must match the variant code |
| `lf_corners` | Corner radius per corner value | `value:radius` pairs, for example `rounded:0.125in,square:0in`; `value:scallop@0.375in` for scalloped | none | **With care**: values must match your variant |
| `lf_radius` | Corner radius when no corner variant decides | a length | `0` | **Yes**: match the real product |
| `lf_mounting_variant` | Variant code the mounting is read from | variant code or `none` | `none` | **With care**: must match the variant code |
| `lf_mountings` | Mounting per value, with optional depth and edge color | `value:mounting`, `value:mounting@depth`, `value:mounting#edge`. Mountings: `flat`, `float`, `standoffs`, `easel`, `block`, `stand`, `floatframe`, `curved`, `curvedout`, `loose`, `board`, `standout`, `card`, `folded`, `edgeframe`, `recessed`, `floater`, `picture`, `ornament`. Edge colors: `#white`, `#black`, `#silver`, `#pewter`, a hex; floater colors `#walnut`, `#ebony`, `#maple`, `#black`, `#white`, `#silver` | none | **With care**: values must match your variant |
| `lf_mounting` | Mounting when no mounting variant decides | one mounting, optional `@depth` and `#edge` | `flat` | **Yes**: match the real product |
| `lf_standoff_variant` | Variant code the standoff cap color is read from | variant code or `none` | `none` | **With care**: must match the variant code |
| `lf_standoff_colors` | Cap per value | `value:cap`. Caps: `silver-satin`, `chrome`, `gold-satin`, `gold`, `black` | none | **With care**: values must match your variant |
| `lf_standoff_color` | Cap when no variant decides | a cap name | `silver-satin` | **Yes** |
| `lf_standoff` | Standoff size: cap diameter, inset from the trim edge to the cap center, depth off the wall | `diameter:<len>,inset:<len>,depth:<len>` | `diameter:0.75in,inset:1in,depth:0.5in` | **Yes**: match the real hardware |
| `lf_render_variants` | Variant values forced on the preview render only, for templates whose production page draws the mounting into the artwork | `variant:value` pairs or `none` | `none` | **With care**: must match variant codes and values on the template |
| `lf_room` | Rooms for the wall view, first one shown first. `desk` and `shelf` show small prints standing on a surface | `sofa`, `bedroom`, `hallway`, `desk`, `shelf` (comma list) or `none` | `none` | **Yes**: each room needs its `lf-room-<room>.webp` asset on the site |
| `lf_view` | View shown first | `closeup`, `wall` | `closeup` | **Yes** |
| `lf_tilt` | The light follows a phone's tilt | `on`, `off` | `on` | **Yes** |
| `lf_compare` | The Compare button (shows only when the product offers two or more looks) | `on`, `off` | `on` | **Yes** |
| `lf_start` | What shows first | `live` (Live Finish), `standard` (platform preview) | `live` | **Yes** |
| `lf_render_width` | Render width in px. The platform caps it at 1200 | number | `1200` | **Yes**: lower only to save bandwidth |
| `lf_label` | Button that switches Live Finish on | text | `Live Finish` | **Yes** |
| `lf_off_label` | Button that switches back to the platform preview | text | `Standard view` | **Yes** |
| `lf_light_label` | Light slider label | text | `Move the light` | **Yes** |
| `lf_sample_note` | Note over the sample photo | text | `Sample photo. Upload yours to see it here.` | **Yes** |
| `lf_closeup_label` | Close-up view button | text | `Close-up` | **Yes** |
| `lf_wall_label` | Wall view button | text | `On the wall` | **Yes** |
| `lf_compare_label` | Compare button | text | `Compare` | **Yes** |
| `lf_tilt_label` | Tilt button (iPhone) | text | `Tilt to light` | **Yes** |
| `lf_room_label` | Button that cycles the rooms | text | `Change room` | **Yes** |

Feature settings (only needed when the mount uses the feature; left out means the feature is off or uses the default shown):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lf_trim_inset` | Length taken off each side of the page trim, when the template trim box is larger than the product | a length | `0` | **With care**: must match the XML template and the finished size |
| `lf_design_looks` | Look (and optional mounting) chosen by design code, for a template shared by products with no finish variant. First match wins | `pattern:look` or `pattern:look/mounting`; `*` matches any characters; look may be `off` | none | **With care**: patterns must match your design codes |
| `lf_design_code` | Hint for the current design code | pass `design.code` (a Liquid variable, no quotes) | empty | **No**: leave as `design.code` |
| `lf_holes_variant` | Variant code that decides drilled holes | variant code or `none` | off | **With care**: must match the variant code |
| `lf_holes` | Hole set per value | `value:none`, `value:top`, `value:corners` | none | **With care**: values must match your variant |
| `lf_hole` | Hole size | `diameter:<len>,inset:<len>` | `diameter:0.1875in,inset:0.5in` | **Yes**: match the real product |
| `lf_frame` | Frame dimensions for `floatframe`, `recessed`, `floater` and `picture` | `width`, `gap`, `depth`, `lip`, `recess` as `key:<len>` pairs | floatframe `width:0.75in,gap:0.25in,depth:1.25in`; picture `width:0.75in,lip:0.25in,depth:0.875in` | **Yes**: match the real frame |
| `lf_stand` | Clear acrylic stand size for the `stand` mounting | `width:<len>,height:<len>` | `width:4in,height:0.5in` | **Yes** |
| `lf_curve` | Curve of a curved print | `radius:<len>` or `arc:<angle>deg`, plus optional `shape:concave` / `shape:convex` or `variant:<code>` | `radius:12in` | **Yes**: match the real product |
| `lf_edge_variants` | For a mounting value, the variant that holds its frame | `mounting value:variant code` pairs | off | **With care**: must match variant codes and values |
| `lf_edges` | Frame depth and color per frame value | `value:<depth>#<color>` pairs | off | **With care**: values must match your variant |
| `lf_border` | A border printed round the photo from the product choices (not on canvas) | `variant:<choice>,on:<value>|<value>,width:<choice or length>,color:<choice or hex>,stroke:<choice>=<value>@<width>#<hex>` | no border | **With care**: must match your option codes and values |

Canvas settings (needed when a `canvas` look is used):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lf_bar` | Depth of the canvas sides drawn | a length, for example `0.75in`, `1.5in` | `1.5in`, reported | **Yes**: match your stretcher bars |
| `lf_bar_variant` | Variant code the bar depth is read from | variant code or `none` | reported | **With care**: must match the variant code |
| `lf_bars` | Bar depth per value | `value:<len>` pairs | none | **With care**: values must match your variant |
| `lf_wrap` | Wrap when no variant decides | `image`, `mirror`, `stretch`, `color`, `color@<hex>` | `image`, reported | **Yes** |
| `lf_wrap_variant` | Variant code the wrap is read from | variant code or `none` | reported | **With care**: must match the variant code |
| `lf_wraps` | Wrap per value | `value:wrap` pairs | none | **With care**: values must match your variant |
| `lf_wrap_color_variant` | Variant a bare `color` wrap takes its color from (a color picker or named colors) | variant code or `none` | reported when used | **With care**: must match the variant code |
| `lf_wrap_colors` | Named colors for that variant | `name:#hex` pairs, or `none` for a color picker | none | **Yes** |

Picture frame settings (needed with the `picture` mounting):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lf_frame_variant` | The frame color variant | variant code | first face, reported | **With care**: must match the variant code |
| `lf_frame_faces` | Frame face image and side color per frame value | `value:asset#side-hex` pairs; an entry with `/` is a path | plain frame, reported | **With care**: each image must be uploaded and drawn to the frame spec |
| `lf_frame_filter` | The page color filter whose swatches show a corner of each frame | filter name or `none` | reported | **With care**: must match the filter name |
| `lf_mat_variant` | The presentation (mat) choice, or a list | variant code(s) | no mat, reported | **With care**: must match the variant code |
| `lf_mats` | Mat window per value, as the print hangs portrait | `value:none` or `value:<w>x<h>[in]` | no mat | **With care**: must match the real mat |
| `lf_mat_color` | Mat color | hex | `#f7f5ef` | **Yes** |
| `lf_orient_variant` | The orientation choice | variant code | as the page, reported | **With care**: must match the variant code |
| `lf_orients` | How far the frame hangs turned from the page | `value:0`, `90`, `-90`, `180` | as the page | **With care**: values must match your variant |
| `lf_sample_wide` | Sample photo for a frame hanging wide | asset name or path | `lf_sample` | **Yes** |
| `lf_all_label` | Label of the all colors view | text | `All colors` | **Yes** |
| `lf_crops` | Crop shape set on the photo upload per mat value, so Adjust crops what prints | `value:<w>/<h>` pairs (`*` for any value), or `none` | nothing set | **With care**: must match the printed opening |

Ornament settings (needed with the `ornament` mounting):

| Setting | What it controls | Accepted values | Default | Can you change it? |
|---|---|---|---|---|
| `lf_ornament` | Ornament thickness, ribbon width, optional disc and drilled hole | `thickness`, `ribbon`, `disc`, `hole`, `inset` as `key:<len>` pairs | `thickness:0.04in,ribbon:0.24in`, reported | **Yes**: match the real ornament |
| `lf_ornament_shape` | Where the outline and hole come from | `mask`, `drilled`, `crown` | `mask`, reported | **With care**: must match the template mask |
| `lf_ornament_sides` | One side, two sides on one page, or each side on its own page | `one`, `front,back`, `back,front`, `pages:<front>,<back>` (cards: `pages:<front>,<inside>,<back>`) | `one`, reported | **With care**: must match the template pages |
| `lf_sample_page` | Page shown with placeholders before an upload, or the sample image filling it | a page name, `none`, `fill`, `fill:all`, `frames` | `lf_page`, reported | **With care**: must name a real page |
| `lf_ribbon_variant` | Ribbon color variant | variant code or `none` | reported | **With care**: must match the variant code |
| `lf_ribbons` | Ribbon color per value | `value:#hex` or `value:#hex/#trim` pairs | `lf_ribbon` | **With care**: values must match your variant |
| `lf_ribbon` | Ribbon when no variant decides | hex | first `lf_ribbons` entry, else `#b0232c` | **Yes** |
| `lf_backdrop` | Background behind the ornament | `page:<page>`, `page:<page>@<theme id>/<product id>`, an asset name or path, or `none` | plain stage, reported | **With care**: pages and ids must exist |
| `lf_material_filter` | The materials view across materials | filter name, or `links:<Label>=<url>|<Label>=<url>` | reported | **With care**: must match the filter or real URLs |
| `lf_material_label` | Materials view label | text | `Materials` | **Yes** |
| `lf_sides_label` | Both sides view label, or side names for cards | text, or a comma list such as `Front,Back` | `Both sides` | **Yes** |
| `lf_flip_label` | Turn over button | text | `Turn over` | **Yes** |

Things that are not set here
- The preview size and shape. They are always read from the page of the XML template named in `lf_page`.
- Price. Live Finish shows the product; it never changes the price or the order.
- Room photos, frame faces and sample photos are site assets. The mount only names them.

Not yet confirmed: The full list of `lf_sample_page` values is not confirmed beyond those shown.

## Changelog

- 2026-11-24: Page created from `28_TOOL_SETTINGS_REFERENCE.md` section 6 (one page per tool). Current version and changes since 3 Nov added from the tool source and CHANGELOG. Source: claude-chat, tool source.
