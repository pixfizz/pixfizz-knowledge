# 27: Live Finish and 3D Previews

**Authority Scope:** Product Previews as a class: Live Finish (the material and finish
preview for flat products) and 3D Preview (the rotatable model for products that wrap or
drape). What each is, how one is configured, mounted, installed and verified, the
per-family status, and the known defects.

**Not in scope:** custom design tools, which write to the orderline (`26_CUSTOM_DESIGN_TOOLS.md`);
the platform editor and its own mapped previews (`17_DESIGN_TOOL.md` § Mapped Previews);
the preview render endpoint itself (`61_PIXFIZZ_API.md` § 9 Dynamic Design Previews). The myPixfizz
setup tool that writes Live Finish mounts is described in the myPixfizz files
(`70_MYPIXFIZZ_OVERVIEW.md`, `71_MYPIXFIZZ_FEATURES_ROUTES.md`); this file only says what
it produces.

_Last updated: 2026-10-22_

---

## 1. What a product preview is

A **product preview** shows the finished product, with the customer's own artwork on it,
beside the standard editor. It **writes nothing to the orderline** and changes nothing in
production. That is the line between a product preview and a custom design tool: a
custom design tool takes the order; a preview only sells it.

Product Previews are their own category in the Pixfizz tool taxonomy (decided 25 Sep 2026),
next to custom design tools and platform extensions:

- It is **not a platform extension**: those are switched on by an `admin/checklist/...`
  key and never mount from a template option.
- It is **not a custom design tool**: it has no contract with the orderline.
- Like a custom design tool, it is **mounted from its own hidden Text template option**
  and **configured only by that mount**. Configuration travels with the template export.
  See `26_CUSTOM_DESIGN_TOOLS.md` § 4 for why configuration never lives in a product or
  design custom field.

There are two preview engines, plus the older platform mechanisms they replace.

| | Live Finish | 3D Preview |
|---|---|---|
| For | Flat or near-flat objects where the **material is the selling point**: metal, acrylic, paper and fine art prints, canvas, frames, ornaments, panels | Objects that **wrap around or are soft**: mugs, tumblers, bottles, pillows, totes, blankets, apparel, packaging |
| What the customer sees | The design on a lit panel that tilts with the mouse or the phone, with finish, mounting, corners and an "on the wall" room view | A rotatable model with the design mapped onto the print surface |
| Rendering | CSS 3D and canvas, no library | three.js r128 (pinned, from cdnjs, with SRI) |
| Parent asset / snippet | `live-finish.js`, `product/live-finish`, room and sample images | `preview-3d.js`, `product/preview-3d`, one `p3d-model-<name>.js` per model |
| Mount option | `lf_mount` | `p3d_mount` |
| Key prefix | `lf_` | `p3d_` |

**Which engine fits which gift** (decided 27 Sep 2026): ornaments, puzzles, acrylic
keepsakes and blocks, slate and hardboard panels, coasters, glass cutting boards, wall
tiles, magnets, license plates, bookmarks and key hangers are Live Finish. Mugs,
tumblers, water bottles, can huggers, pillows, totes, blankets and apparel are 3D Preview.
For metal prints a 3D model adds little: the finish is what sells, so metal is Live Finish.

### The older mechanisms

- **Mapped previews** (platform, per print product): a background photo plus a small GLB
  print surface that warps the flat artwork onto the photo. See `17_DESIGN_TOOL.md`
  § Mapped Previews. For mugs they are being **replaced by 3D Preview**: the per-size
  configuration cannot be copied between sizes, and a copied template can carry another
  size's GLB (see § 6).
- **Template preview pages (room scenes)**: a `preview` page in the design that places the
  production page (`<ipage template="print" .../>`) on a room photo with a shadow layer.
  Static, one per size. Two lessons from a 60-size metal range (1 Sep 2026, verified by
  reading source and by local render):
  - Derive the scene scale from one known object (the seed's print width in the photo),
    then cross-check it against two or three real objects in the photo before placing
    every size from it.
  - Use **one master plate and crop tiers** (a bottom-aligned center crop expressed as the
    background `width` and `x`/`y` in the XML) rather than one photo per size. Large
    sizes get a wider crop of the same image. Shadow layers compress well as WebP
    (a 6.8 MB PNG became 333 KB at no visible loss).
- **Editor previews**: whatever the gallery's preview pages show. Live Finish sits over
  gallery slide 1 and needs that slide to exist (§ 2.6).

---

## 2. The Live Finish standard

Level: **template-level (Shopper 24 parent)** unless marked otherwise.

### 2.1 Parts and where they live

| Part | Where | Note |
|---|---|---|
| `live-finish.js` | Site asset on the shopper24 parent | The whole engine. Injects its own styles with a versioned id (`lf-css-<version>`), so an older build's styles never linger. |
| `product/live-finish` | Parent snippet | Reads the mount, passes it as attributes, lists missing keys in `data-config-missing`. Some versions change the snippet (0.10.0 added eight pass-through attributes): install asset and snippet together. |
| Room and sample images | Parent site assets, WebP | `lf-sample-metal.webp`, `lf-room-<room>.webp`. WebP is about half the JPEG size at the same look. |
| `lf_mount` | Template option, Text, hidden, hide from cart | Its `custom_script` calls the snippet with keyword arguments. Per template. **Travels with a template export.** |

The tool source and version history live in the private tools repo
(`product-previews/live-finish/`), one folder per version for delivery.

### 2.2 Mount keys

Every value is a quoted string. Lengths carry their unit (`0.75in`, `6mm`). `lf_units` is
`in` on every mount so far.

- **Required:** `lf_page`.
- **Core keys** (a missing one is listed in `data-config-missing` and logged as a
  `console.error`): `lf_units`, `lf_sample`, `lf_finish_variant`, `lf_looks`, `lf_look`,
  `lf_corner_variant`, `lf_corners`, `lf_radius`, `lf_mounting_variant`, `lf_mountings`,
  `lf_mounting`, `lf_room`, `lf_room_label`, `lf_view`, `lf_tilt`, `lf_compare`, `lf_start`,
  the label keys (for example `lf_light_label`, `lf_compare_label`), `lf_render_width`,
  the standoff keys (`lf_standoff_variant`, `lf_standoff_colors` / `lf_standoff_color`,
  `lf_standoff`) and `lf_render_variants`. **Declare every core key in the mount, with
  `'none'` where unused** (`lf_corner_variant: 'none'`, `lf_mounting_variant: 'none'`,
  `lf_standoff_variant: 'none'`, `lf_render_variants: 'none'`) and the standoff color and
  size even when no standoffs are offered. 0.17.1 still logs an error for each undeclared
  key (seen: `lf_corner_variant`, `lf_mounting_variant`, `lf_standoff_color`,
  `lf_standoff`). *Seen 2026-10-05.*
- **Feature keys** (from 0.8.0): off when omitted, reported only when the mount uses
  their feature. `lf_holes_variant`, `lf_holes`, `lf_hole`, `lf_frame` (float frame),
  `lf_stand`, `lf_curve`; canvas (0.10.0): `lf_bar`, `lf_bar_variant`, `lf_bars`, `lf_wrap`,
  `lf_wrap_variant`, `lf_wraps`, `lf_wrap_color_variant`, `lf_wrap_colors`; picture frame
  (0.13.0): `lf_mat_variant` (takes a comma list, first choice shown wins); ornaments:
  `lf_ornament`, `lf_ornament_shape`, `lf_ornament_sides`, `lf_ribbon_variant`,
  `lf_ribbons`, `lf_ribbon`, `lf_backdrop`, `lf_sample_page`, `lf_material_filter`.
- `*_variant` keys name the **variant or option code** the preview follows; the plural key
  maps that choice's value codes to looks or settings (`value:look,value:look`). Two maps
  may read the same variant (corners and mounting both following a backing choice,
  because the backing decides whether the corners show). `lf_mat_variant` may name a
  template option (`template_options[code]`) as well as a variant. *Verified live, Live
  Finish 0.23.5, 2026-10-22.*
- **A printed border:** an `lf_mats` window ending `@flat` draws a printed border (no bevel,
  no shadow) in `lf_mat_color`, on the sample before any upload too. With no
  `lf_frame_filter`, the All colors captions use the frame variant's radio labels.
  Tool-level. *Verified live on two sites, Live Finish 0.23.5 served, 2026-10-22.*
- **A color wrap can be a design (template) option.** A child option `value_type: color` with
  `trigger_value_code: <wrap value code>` and an element substitution `background_color`
  `<implicit>` on the canvas page, next to a layout substitution on the wrap value, imports
  inside a template export, and the render shows the picked color on the sides. Live Finish
  reads it with `lf_wrap_color_variant: '<child code>'` (the engine also finds
  `template_options[code]`). Platform-level option, tool-level key. *Verified on
  experience.pixfizz.com, 2026-10-20.* Option setup: `22_OPTION_VARIANT_RENDERING.md`.

### 2.3 Looks, mountings and rooms

- **Looks are `<base>-<finish>`.** Bases: `white`, `clear`, `brushed`, `acrylic`, `paper`,
  `metallic`, `fineart`, `textured`, `etching`, `canvas`, and the ornament materials
  (for example `crystal`). Since 0.22.0 the bases `clearacrylic`, `crystal`, `glass` and
  `ceramic` are ornament-only: with any other mounting the engine logs `console.error`. Use
  `acrylic` for glass products that are not ornaments (for example glass cutting boards).
  *Verified by reading source, Live Finish 0.22.0, 2026-10-16.* Finishes: `gloss`, `semi`,
  `matte`. A finish alone means a white base; a base alone means gloss.
- **Mountings:** `flat`, `float`, `standoffs`, `easel`, `block`, `stand`, `floatframe`
  (needs `lf_frame`), `curved` (concave only, needs `lf_curve`), `loose` (a print lying on
  a table), `board`, `standout`, `picture` (framed, 0.13.0), `ornament`. Any value can
  carry its own depth: `float@0.5in`. Convex curves have no rendering.
- **Rooms** live in the asset's registry; `lf_room` is one room or a comma list
  (`sofa`, `bedroom`, `hallway`, `sideboard`) and a Change room button cycles them.
  The furniture in each room photo is calibrated to a real width, so the print hangs at
  true scale. The room photos are AI generated. How to calibrate a new room (from a height,
  on the wall plane) is in § 2.11.
- **Room plates are 4:5, with scale props inside the middle 85%.** The Live Finish stage is
  about 4:5 on desktop (528 x 669 at 1440 px) and about 1:1 on a phone (343 x 356 at 375 px).
  A 4:3 plate covering that stage keeps only the middle ~59% of its width, so side props that
  give scale (a mug, books) get cropped. Generate close-up plates at 4:5 with the props fully
  inside the frame and the surface line 66 to 75% down, so the phone crop (10 to 90% of the
  height) still shows it. Leave a clear span of at least 13 in at true scale for prints up to
  12 in on the long edge. Calibrate from a prop whose size the prompt states (12 oz mug
  3.75 in, hardcover 9.5 in) and record it as an assumption. *Stage size verified by query,
  2026-10-15; plate guidance derived from it.*
- **Material looks are tuned by eye**, not against lab photos. Treat every look, thickness
  and depth as an assumption until the lab confirms it.
- **Canvas texture reads as a fine weave:** high-frequency grain (threads at 1 px or finer at
  screen scale), low opacity, multiply. A repeating line grid a few pixels apart reads as graph
  paper and was rejected. Canvas sides stay a plain cloth color with shading, no texture.
  *Stated by Alex, 2026-10-07.*
- **Metal prints always draw rounded corners** (default 1/8 in radius, at least 3 px on
  screen), in close-up and in the room view. *Stated by Alex, 2026-10-07.*

### 2.4 Geometry comes from the template, never from the mount

- **The shape is never in the mount.** It is the `lf_page` page size less twice its bleed,
  read from `/v1/themes/<id>` and re-read whenever the theme changes.
- **`lf_page` is matched against `<page type="...">` in the template's XML definition**,
  not against a design page name. On a template whose definition page is `type="print"`
  the mount says `lf_page: 'print'`. *Verified by reading source, 26 Sep 2026.*
- **Choosing the page:** a page whose name starts with `preview` is the product picture,
  never the print. Prefer `photo`, `canvas`, `wall-decor`, `print`, `front`, `card`.
  Ornaments are the exception: the print page is `production` and `preview` is the
  backdrop (`lf_backdrop: 'page:preview'`).
- **Canvas:** the platform render draws the selected wrap (gallery, mirror, color) into
  the bleed itself, so the render's bleed is the truth for the sides. *Verified by reading
  renders, 27 Sep 2026.* Bar depth is not stored anywhere on most canvas templates; read
  it from the template code or name, and confirm it with the lab. Where the canvas page
  carries the wrap inside the page rather than as bleed (a 14 x 16 page for an 8 x 10
  canvas), the inset per side is `(page - finished size) / 2`.
- **The artwork** is the platform render of the production page with the customer's
  current options. For how that request is built see § 4.
- **The template mask shows the bleed shape, not how the product is made.** Before modelling
  a preview or tool for a supplier product, read the supplier's template or spec sheet and
  look at a real product photo. A pet bandana modelled from its mask alone as a sleeve over
  the collar had to be rebuilt: the print is the front panel only and the collar runs through
  a channel. *Observed, 2026-10-14.*

### 2.5 Behavior on the product page

- **Placement:** inside `.px-display-wrapper`, right after `.px-display`, positioned over
  gallery slide 1 only. It hides on other slides and in Standard view.
- **Fit to screen:** in a two-column layout it caps `.px-display` at the viewport height
  less the gallery top, never under 80% of the slide's own height or 480 px; removed at
  teardown.
- **Sample before upload:** a photo sample is shown until the customer uploads. The
  sample note is a button that clicks the platform's own Upload Image button, so the
  standard dialog opens. **Designed products** (a finished design on the card) show the
  design render before an upload, never the generic sample photo.
- **Phone tilt:** Android starts on its own; iOS shows a tilt button; with Reduce Motion a
  button is offered on both.
- **Compare** splits the panel at a divider; only finishes mapped in `lf_looks` and offered
  on the page take part.
- **Theme swaps:** a PDP layout replaces the product by AJAX after the size or orientation
  `change` event, so Live Finish watches `theme_id` itself (polls every 500 ms) rather than
  reading it in a change handler. See `17_DESIGN_TOOL.md` § Custom Design Tools,
  Browser-Side Rules. *Verified live, 26 Sep 2026.*
- `storeOptionSelectionIntoURL` writes `template_options[lf_mount][value]=` into the page
  URL. Harmless.
- **The stage resets `transform` on its own images.** A site rule such as
  `img:hover { transform: scale(1.02) }` in `style/custom.css` zooms every image inside a
  preview while the mouse is over it. Live Finish 0.24.2 guards it with
  `.lf-stage.lf-stage img:hover, .lf-stage.lf-stage img { transform: none }`; inline
  transforms still win. Other preview tools likely need the same guard (not checked). On any
  "it zooms when I hover" report, list the site's `:hover` rules that set `transform` first.
  *Verified live by computed style, 2026-10-22.* General rule: `26_CUSTOM_DESIGN_TOOLS.md` § 3,
  *Defend against the site's `style/custom.css`*.

### 2.6 Gating: what must exist before it shows

- **A gallery slide.** Live Finish sits `position:absolute; inset:0` over the wrapper and
  takes the height of slide 1. **A design with no preview page gives an empty gallery and
  a 0 px tall Live Finish.** The gallery renders one `px-design-preview` per
  `design.preview_page_names`; a design page counts only when the page itself carries
  `preview: true` (the Preview checkbox), with a matching definition page type. A
  `<set preview="true" ...>` in the definition alone is not enough. Working pattern: the
  definition adds `<set preview="true" fulfillment="false" editor="false"><page type="preview" width="10" height="10" /></set>`;
  the design adds a `preview` page with one centered `<ipage template="print" crop="true" .../>`,
  marked as preview. *Verified live, 27 Sep 2026.* Admin route for the flag:
  `18_ADMIN_NAVIGATION.md` § Bulk Update Tools.
- **The Custom Script field definition on template options** must exist on the site
  (`custom_script`, plus `hidden` and `hide_from_cart`). Check before importing a mount.
- **The boolean option flag definitions** must exist too. A mount option imported from
  another site's template export carries that site's flag keys (`kiosk_mode_only` and
  others); a site without boolean definitions for them stores the string `"false"`, which
  Liquid reads as true, so the option renders in kiosk mode only and the preview never
  mounts, with no error. See `22_OPTION_VARIANT_RENDERING.md` § 3.1.
- **Platform defect, open:** `px-design-preview` does not request its first render until
  the page is scrolled (see § 6). Anything layered on the gallery inherits that.
- A background browser tab reports the gallery item at 1 px tall; judge only a visible
  tab.

### 2.7 Dark child sites

Live Finish renders `.lf-stage.pxt` and takes its colors from the shared `--pxt-*`
tokens (pressed states use `--pxt-ink` with `--pxt-surface` text). On a dark child site
set the ink tokens on the class every pxt tool shares, not on a Live Finish root class
that may change between versions:

```css
html body .pxt { --pxt-ink: #191c1d; --pxt-ink-muted: #667085; }
```

The wider dark-child token list is in `50_SHOPPER_TEMPLATE_REFERENCE.md` § 4.

### 2.8 Query parameters

- `?lf_debug=1` shows the tilt state on the page.
- **Never use `preview` as a query parameter** for a review or debug switch on any
  storefront: the platform takes `?preview=` over and redirects to the editor, which
  returns 404. Platform-level. See `50_LIQUID_REFERENCE.md` § Request.

### 2.9 Live Finish outside Shopper

The Live Finish engine relies on Shopper-only parts: `.px-product-gallery`, a same-origin
`/v1/themes/<id>`, and the `lf_mount` option. On a non-Pixfizz page (WordPress, a Shopify
storefront domain) the script can show a preview `<img>` but cannot read its pixels or
the theme JSON: a `fetch` from another origin to `/v1/themes/<id>` and
`/v1/themes/<id>/preview.jpg` returned no `Access-Control-Allow-Origin` header. *Verified
by query on one client's WordPress domain, 4 Oct 2026.* Anything that samples the image
(WebGL, canvas) therefore needs CORS on the preview endpoint for allowed origins, or a
proxy. Site assets served from `cdn.pixfizz.com` do answer CORS (a WebGL texture from an
`asset_url` works). A standalone embed mode (one script tag, config in data attributes,
watching a given `<img>`) is not built.

### 2.10 Photo plates (Live Finish Plates)

Live Finish Plates puts the customer's design on a **photo of the real-looking product** (a
plate): a blank photo of the product, plus a map that says where the print goes and how it is
shaded (a green box or green print area marks the print surface in a second, edited copy of
the photo). Tool-level technique, platform-independent. *From prototype and plate builds,
17 and 18 Oct 2026, unless marked.* Generating the plate photos is in
`83_AI_IMAGERY_PRODUCTION.md` § B12.

- **One plate for many sizes.** Generate one green box at a known size (for example 16 x 20
  in), store `free={w,h,anchor}` in the plate meta, and at run time map each requested box
  through the green box's homography (unit square in plane inches). Coverage comes from the
  drawn polygon, not the map's G channel. Write the map's R shade over the whole photo, not
  only inside the green. With a resized box the area outside the green comes from the base
  photo, so the blank edit must replace a wide soft band around the green (about 6% dilate,
  3% blur) **plus** the narrow hard mask: the wide band alone leaves green at the old box edge,
  the narrow one alone a visible seam on a plain wall.
- **An object resting on the print** (a mouse on a mousepad): skip the blank edit. Coverage is
  the face hull minus the pixels that are not green (`G - max(R,B) < 22`, opened, small blobs
  dropped); the shade is the green's own G channel, which gives the contact shadow. Despill
  the object edge and the product rim (set G to `max(R,B)`).
- **Build the mask at full resolution.** An AI edit comes back at about 930 px; pasting it into
  a 4K base leaves a soft, tone-shifted rectangle. Generate the blank natively at 4K and use
  the green edit only to build the mask (components over 2% of the frame, close 15 px, convex
  hull). Scale the edit to the blank and align it with an ECC affine registration (green area
  masked out) before taking the mask: it costs a second and removes a shifted edge.
- **Round products map a cylinder, not a quad.** Koozies, a gaiter, a mask on a face cannot be a
  homography. Fit the cylinder from the green silhouette (center, half width, top and bottom
  edges with their bow) and map `u = 0.5 + asin(dx / rx) * r / w` (r = radius, w = print width,
  both in inches), v between the edges. For anything worn on a face keep the edges flat (v from
  the top to the bottom of the silhouette) and let the silhouette clip the print.
- **Fabric:** no displacement on flat prints (a fine-weave displacement jitters faces and
  doubles edges); set warp dx / dy to 0 on aprons and T-shirts and let the multiply carry the
  creases. Fill the design canvas white before drawing, so transparent areas mean no ink.
- **Banners on a straight-on wall need no green.** A close-up uses a set box with a cap (for
  example 24 x 24 paper, 48 x 24 vinyl) so every size keeps its shape; a scene uses measured
  pixels per inch (§ 2.11). Light the print from the wall's blurred luminance, never from the
  photo itself, or brick and wood grain print through the banner. Add a soft drop shadow;
  vinyl gets grommets every 24 in.
- **The stage takes the plate's aspect ratio.** Drawing a 4:5 plate into a 4:3 canvas squashes
  the product.
- **Plate weight:** ship plates as WebP (photo about q82, map lossless). Strip map data the
  material never reads: fabric takes its shade from the photo, so its map R can be flat 128;
  free-box plates draw their own coverage. A map that compresses badly is carrying photo noise
  in R (one batch went from 10.9 MB to 1.4 MB with no visible change). A sharp textured
  background (a wood fence close-up, 814 KB at q82) drops below 300 KB by softening only the
  band the print covers (bilateral filter) and shipping at 1320 px wide around q60 to q66.
- **Replacing a plate asset:** uploading a file with the same name does not overwrite an asset;
  upload the replacement on that asset's own edit page (`26_CUSTOM_DESIGN_TOOLS.md` § 3, tool
  assets).
- **Preview tools share one UI standard.** Live Finish Plates matches Live Finish for every
  shared control: the sample note is a top-left `pxt-chip` button that opens the platform
  upload dialog (the first enabled `px-image-upload` button), the views pill sits top right,
  Standard view bottom right. Before changing any on-stage control in one preview tool, copy
  the behavior from the Live Finish source rather than inventing a variant. *Decided by
  Pixfizz, 2026-10-17.*

### 2.11 Room scenes at true scale

Applies to Live Finish rooms, Live Finish Plates scenes and any tool that draws art at true
size on a room photo (wall designs, custom sized prints). Tool-level.

- **Measure the scale from a dimension that barely varies.** Sideboard or table height (30 to
  34 in), seat height (17 to 18 in), door height (80 in). A plate that assumed a sideboard was
  60 in wide made every piece 50% too big; measured on its height (about 34 in), the scale was
  10.5 px/in. Place each size in a slot that avoids props; never stretch a green box to the
  requested size. *Measured on a plate, 2026-10-17.*
- **Calibrate on the wall plane, not on the front of the furniture.** A bench or credenza
  measured at its front edge reads 7 to 15% too large at the wall. Solve the wall scale from
  the object's front and back floor lines with the camera height at 60 in:
  `s_wall = s_front - (y_front - y_wall) / 60` (pixels per inch; y in pixels, the front floor
  line minus the line where wall meets floor). For public spaces, center the art at 60 in
  from the floor and keep its bottom edge at least 6 to 10 in above the furniture.
  *From a prototype, 2026-10-19.*
- **Brick walls:** modular brick courses are 2 2/3 in; measure the course period with an FFT on
  the row profile, on the vertical courses only (AI bricks are not true proportion
  horizontally).
- **A room that holds any size.** A room photo shows about 55 to 60 in of wall above the
  furniture, so a 96 in piece cannot fit. Extend the room by outpainting (pad the original onto
  a mid-grey canvas at the same pixels per inch, bottom-centered, sized for the largest piece
  plus margins, and ask for a taller wall with a ceiling line; generation details in
  `83_AI_IMAGERY_PRODUCTION.md` § B12). The model re-lights the original region, so paste back
  only the furniture area (feathered about 70 px), color-matched to the new lighting with a
  heavily blurred ratio: the wall stays generated, the furniture stays sharp, and the old
  calibration carries over (offset only). Check the placement with template matching on a
  furniture crop, not the whole photo: a plain wall correlates almost anywhere.
- **Keep-outs.** Every scene carries keep-out boxes (lamps, plants, side tables, windows,
  downspouts, benches) in master pixels. Fit order: the usual hang, then raise up to 30 in,
  then lower the hang to 2 in, then side spots, then switch scene with a one-line notice. The
  piece never covers a keep-out. Zoom out in fixed steps (0.84 per step) so the scene holds
  still while the size changes within a step, and keep 12 in of the furniture in view above
  the stage controls. *Verified by local render at 1440 and 390 px with an automated keep-out
  overlap test, 2026-10-22; not verified on a live site.*
- **Scene imagery is generated photography**, never drawn in code; a person for scale is a
  silhouette (vector). Scene files are calibrated in master pixels. *Decided by Pixfizz,
  2026-10-22.*

---

## 3. The 3D Preview standard

Level: **template-level (Shopper 24 parent)**.

### 3.1 Parts and architecture

- Parent asset `preview-3d.js` and snippet `product/preview-3d`. **Styles are injected by
  the asset**: there is no style snippet.
- A **shape registry** in the asset: each shape declares the keys it needs, its views and
  its plausibility checks. The mug is the first shape; tumbler, canvas, framed print,
  pillow, ornament and acrylic block are on the roadmap.
- **Models** (from 0.3.0) ship as parent site assets `p3d-model-<name>.js`, each a base64
  GLB plus its facts, because **a `.glb` upload is refused as a site asset** (platform,
  tested 24 Sep 2026). The engine vendors the r128 GLTFLoader.
- **The artwork** is the platform render of the production page: `/v1/themes/<id>/preview.jpg?template_name=<p3d_page>&product_id=...`
  plus the customer's current options (§ 4).
- The viewer uses the `--pxt-*` tokens. 0.2.0 opened a dialog from a button on the main
  gallery image; **0.4.0 adds `p3d_placement: 'inline'`**, which puts the live model in
  place of the main product image and re-renders on every option change. Inline is the
  preferred placement for new installs (decided 5 Oct 2026).

### 3.2 Mount keys and geometry rules

Mount option `p3d_mount` (Text, hidden, hide from cart), whose `custom_script` calls
`product/preview-3d` with keyword arguments. Keys used by the mug shape: `p3d_page`,
`p3d_trim_w`, `p3d_trim_h`, the body diameter and `p3d_height`, `p3d_print_bottom`,
`p3d_model` (0.3.0) and `p3d_placement` (0.4.0).

- **Dimensions have no default.** A missing or implausible dimension switches the preview
  off and logs a `console.error`. Other defaults are listed in `data-config-missing`.
- **Cross-check at init:** `p3d_trim_w` / `p3d_trim_h` against the template page size from
  `/v1/themes/<id>`, tolerance 0.02 in. Copy the page size exactly; do not round. Each
  template carries its own mount, because two templates for the same product can have
  different page sizes.
- **`p3d_page` must equal the page name in the template's designs**, because it becomes
  `template_name` on the render request. Mug templates differ (`front` on one, `fullwrap`
  on another): read a design's `<page name="...">` before writing the mount. *Verified by
  reading source, 5 Oct 2026.* **Symptom of a wrong `p3d_page`:** with `p3d_placement`
  `inline`, a "View in 3D" button on the product image is the fallback: the first artwork
  request (`preview.jpg?template_name=<p3d_page>`) failed and the engine restored the
  standard gallery. The console logs `[preview-3d] p3d_page ... is not a page in this
  template definition`; requesting `preview.jpg` with each page name gives 404 vs 200. Most
  common cause: a mount copied from another site. *Verified live, 2026-10-15.* Seen again on
  the msphub 11oz mug template, 2026-10-22: its definition names the print page `front`
  (8 x 3.5 in) plus two preview sets, and a mount copied from a `fullwrap` mug config gave
  `preview.jpg?template_name=fullwrap` 404 (`front` 200) and the fallback button on every lab
  site consuming it. Base a mug mount on the definition it will run on, with
  `p3d_placement: 'inline'`. *Verified live on a lab site, 2026-10-22.*
- **Rim rule:** `p3d_print_bottom + p3d_trim_h` must not exceed `p3d_height`; the engine
  switches the preview off when it does. Center the print vertically unless the product
  spec says otherwise: `p3d_print_bottom = (p3d_height - p3d_trim_h) / 2`.
- **A mount that declares the body sizes runs on both 0.2.0 (mug built in code) and 0.3.0
  (Blender model)**, so a child template can be fixed before the parent is upgraded.
  *Verified live, 5 Oct 2026.*
- The model carries its own facts in the GLB scene extras (trim, diameter, height, print
  bottom) and the engine checks them against the mount and the page.

### 3.3 Models are built in Blender

Standing rule (29 Sep 2026): **every 3D product model is built in Blender.** No
hand-written three.js geometry for product models. A packaging model took five guessed
versions in hand-written geometry; built in Blender and checked against photos of the
real product, it was right in one pass.

- Build from a script (`build.py`), parameterised, so it is repeatable. Geometry from the
  flat sheet (dieline) in real units; **UVs = sheet position**, so the print file lands
  exactly as printed.
- Folding products: the basis is the finished product; shape keys hold the partly folded
  and flat states, exported as glTF morph targets.
- **Render and check in Blender first** (front, end, 3/4, compared with photos of the real
  product) before anything reaches the viewer.
- Export GLB with `export_morph=True, export_morph_normal=True, export_materials='NONE'`.
  In three.js r128 the materials need `morphTargets: true, morphNormals: true`, and the
  canvas texture needs `flipY = false` for glTF UVs.
- **Mug mesh contract:** meshes `body`, `inside`, `handle`, `print`; `print` UV 0..1 is the
  whole production page, u left to right, centered opposite the handle, handle on -X.
- **Check face winding.** A body wound inside out with a single-sided material culls the
  outer wall and shows the inside through it (fixed in the 0.3.1 mug models).
- Keep the `.blend`, `.glb`, `build.py`, `render.py` and the sheet data together with the
  tool.
- **The same pipeline serves a custom design tool's own preview** (a 3D product beside the
  tool). When a 2D canvas fake of a 3D product looks cheap, stop iterating on it: build the
  model in Blender and render it in the tool's own WebGL. Tool-level. *From a prototype round,
  2026-10-19.*
  - Blender runs headless from Python: `pip download bpy` (the wheel matching the Python
    version), install it in a virtual environment and run the model script with that Python.
    Models are scripts (bmesh, lathe or spin for round products), so a supplier size change is
    a re-run.
  - Contract: named materials (`print`, `print_back`, `ceramic`, `webbing`, `plastic`,
    `metal`); the print surface's UVs map exactly onto the print page, (0,0) top left, so the
    customer's artwork canvas is the texture. Bake ambient occlusion to a **POINT**-domain
    color attribute (CORNER domain splits every vertex and triples the file).
  - Pack with `gltfpack -kv -kn -km -vtf` (`KHR_mesh_quantization`, no meshopt compression).
    `-kv` is required: without a texture on the material gltfpack drops the UVs. Quantization
    offsets live on node transforms, so the reader must apply node TRS.
  - A 100-line GLB reader and one shader cover it, with no three.js; 127 to 202 KB per model.
  - Site assets refuse `.glb` (§ 3.1), so a model ships base64 inside an asset the platform
    accepts, as `p3d-model-<name>.js` does. Never `fetch()` an embedded `data:` URL: a strict
    `connect-src` CSP blocks it and the stage stays blank while a local test passes; decode
    the base64 with `atob` into an ArrayBuffer.
- **Never replace an approved, live preview with a new technique.** A prototype round that
  swapped a locked, live 2D preview for a new GLB model came out clearly worse. New models are
  for products with no approved preview yet. Before publishing, screenshot the approved preview
  next to the new one and ship only if the new one is at least as good. *Decided by Pixfizz,
  2026-10-19.*

### 3.4 Design rules for wrap products (mugs)

- Front center = middle of the page, opposite the handle.
- Readable content stays inside the **front zone: page center ± 0.83 × radius** (about
  2.7 in on an 11oz mug, 2.82 in on a 15oz), so it reads in one view of the curved body.
- Keep 0.75 in clear at each end of the page (the handle side).

### 3.5 Version standard (both engines)

- Every build exposes its version and logs one ready line at init:
  `[live-finish] ready {version: '<v>' ... missing: []}` and
  `[preview-3d] ready <v> page <page>`. In the console, `LiveFinish.version` and
  `window.Preview3D.version`. Assets are aggressively browser-cached, so this is the only
  reliable answer to "did my change deploy".
- One version per build, taken from the top of the changelog at build time. Two builds
  made in parallel from the same base must not share a number: the second is merged onto
  the first and takes the next number.
- A version that contains another (0.10.0 canvas contains 0.9.0 paper) is installed
  instead of it, never alongside it.
- Every new Live Finish version is also taken into the myPixfizz setup tool, so its
  preview runs the same asset the storefronts run.

---

## 4. Platform facts both engines rely on

Level: **platform (Pixfizz CMS)**.

- `/v1/themes/<design>/preview.<ext>?product_id=<id>&template_name=<page>&variants[...]&template_options[...]`
  renders the named page with substitutions and the customer's photo. **`product_id` is
  what makes `variants[...]` apply.** `template_name` avoids the 404 a dotted page name
  gives in the path form. Request it from the site's own host. The render includes the
  bleed and is capped at 1200 px on the longest side (`61_PIXFIZZ_API.md` § Preview
  resolution). *Verified by query, 21 Sep 2026.*
- The uploaded photo is in the product form as `template_options[photo]=db:<image id>`.
  Read option values the way the platform does (`17_DESIGN_TOOL.md` § Custom Design
  Tools, Browser-Side Rules).
- `/v1/themes/<id>` returns the template definition (page types, sizes, bleed) to the
  site's own storefront without admin; another site's theme returns 403.
- `.glb` files cannot be uploaded as ordinary site assets (§ 3.1). Mapped-preview GLBs
  travel inside template exports under `glb_files/`.
- Site assets on `cdn.pixfizz.com` answer CORS; storefront `/v1/themes/*` endpoints did not
  answer CORS to a foreign origin (§ 2.9).
- **CDN images as WebGL or canvas textures: two observations, one safe rule.**
  - 2026-10-21, *verified by query on experience.pixfizz.com:* an `Image` with
    `crossOrigin = "anonymous"` pointing at a `cdn.pixfizz.com` site asset loaded, drew to a
    canvas and `getImageData` worked (canvas not tainted).
  - 2026-10-18, *seen on experience.pixfizz.com:* a three.js `TextureLoader` load (crossOrigin
    anonymous) of a shopper24 asset failed, blocked without a CORS header, and `onError` may
    not fire before the scene renders.
  - Likely reconciliation (inferred, not verified): the URL form differs. Asset URLs are
    host-signed: the `/fz/<hash>/0/<token>/~/<name>` path taken from admin.pixfizz.com is not
    valid on `cdn.pixfizz.com` with the same hash, while the URL `asset_url` prints on the
    storefront is.
  - Safe rule: take the URL from `{{ '<name>' | asset_url }}` in Liquid, never build it; set
    `crossOrigin = "anonymous"` before `src`; and keep a fallback that needs no cross-origin
    read: the image as the stage's CSS background (`background-position` and
    `background-size` in px from the same cover math) behind a renderer created with
    `alpha: true` and `scene.background = null`.
- `cdn.pixfizz.com` serves a `.js` asset as `text/javascript` with CORS, so `import(asset_url)`
  works as an ES module from a Shopper page. JS responses carry no `Content-Length`; images
  do. *Verified by query (headers), 2026-10-21.* Alpha WebP files uploaded to shopper24 came
  back from the CDN with an extra C2PA chunk appended (image data byte-identical), so CDN file
  sizes can differ from the uploaded file. *Verified by query, 2026-10-21.*
- **Experimental `canvas` format for the `px-preview` components (not for production use).**
  Release commits from 16 to 20 Oct add an experimental `canvas` format to the `px-preview-*`
  web components (drawn in the browser on an offscreen canvas, with text support and a
  `ctx.roundRect()` polyfill), JSON responses on the book and theme preview endpoints, and a
  URL parameter to test the canvas renderer. The parameter name and the JSON shape are not
  recorded. Do not build a preview or a tool on it until the developer marks it stable.
  *Observed in release commits 2026-10-16 to 2026-10-20; behavior not verified.*

---

## 5. Status by product family

Status as recorded in the sources on the dates given. Builds move fast: before relying on
a row, read the version from the live console (§ 3.5).

### Live Finish

| Family | State | Notes |
|---|---|---|
| Metal prints | **Live** on client sites | Finishes, corners, mountings, standoffs, holes, float frame, curved. 0.8.3 verified live on the parent, 27 Sep 2026. |
| Paper and fine art prints | Built (0.9.0), not verified live at the time | `loose`, `board`, `standout` mountings; `textured`, `etching` bases. |
| Canvas | Built (0.10.0); **needs fixes** | Reported still needing fixes for finishes, stretched edges and frames (§ 6). |
| Picture frames | Built (0.13.0, `picture` mounting), not installed on 29 Sep | Moulding-corner color selector, mat, sideboard room. Landscape needs template work first (§ 6). |
| Ornaments | **Reported ready** in metal, glass, ceramic, acrylic and wood | Preset-based setup; drilled crystal round first. Needs a parent version with ornament support (0.15.0 or later). Not verified live by this file. |
| Folded cards | In the setup tool | The setup tool generates folded card mounts; no live verification recorded here. |
| Glass cutting boards, puzzles, acrylic blocks, slate and hardboard panels, coasters | Planned | Gift-season order (27 Sep): cutting boards, ornaments, puzzles, acrylic, panels, coasters. |
| Blankets | **Demo only** | A standalone WebGL page (three.js r128) on experience.pixfizz.com, built as a pages Custom Type instance on that child, not a mounted product preview. Knit and fleece. A real knit product whose texture is the design render is not built. |
| Pillows | Planned | Prototype first. |

### 3D Preview

| Family | State | Notes |
|---|---|---|
| Mugs, 11oz and 15oz | **Live on the parent (0.4.0, inline)**; mug design range in rework | 0.3.1 Blender models live on shopper24 (verified 6 Oct 2026). 0.4.0 logs `ready 0.4.0` on baseline. Mapped previews removed from the mug templates that use it. The first mug design range failed review on 6 Oct (§ 6). |
| Canvas, View in 3D | Shopify path only, unverified | An earlier CSS 3D snippet (`canvas/view-3d`, 0.4.x) for the Shopify personalization modal: mounts only when the form has a `variants[wrap]` input and the upload carries a crop ratio. 0.4.1 seen live on the Shopify test site; 0.4.2 not verified. On Shopper, canvas is Live Finish. |
| Packaging (gable box) | Prototype | First model built under the Blender rule. |
| Tumblers, framed prints, pillows, ornaments, acrylic blocks | Planned | Shape roadmap. |

---

## 6. Known defects and open issues

- **Design previews wait for a scroll (platform, open).** `px-design-preview` gates its
  first render on an IntersectionObserver plus `checkVisibility({contentVisibilityAuto:true})`.
  On desktop product pages no preview request is made until the shopper scrolls (3 of 3
  loads on one client site, 23 to 84 s with no request; renders within 3 s of one scroll
  tick). Collection grid previews paint blank for 5 to 25 s. Until it is fixed: never
  report a preview as missing from a screenshot taken without scrolling; scroll one tick
  and wait 3 s. *Verified live on one site, 29 Sep 2026; other Shopper sites not verified.*
- **In a collection grid some design previews never load (platform, open).** With
  `loading="lazy"`, `px-design-preview` observes its own shadow `<img>`, which is 0 x 0 until
  it has a src. In a Shopper 24 grid at 1440 px the third column never intersected, so 16 of
  40 cards stayed blank permanently, not just late; the same SVG URLs render on their own.
  Site workaround (verified by injection, 40 of 40): observe the whole `.card`, set
  `has_intersected` and `is_visible`, then call `scheduleUpdate()`. The proper fix belongs in
  the component (observe the host box, or give the img a minimum size). *Verified by query on
  a client site, 2026-10-06.*
- **The wall view overflows on oversize prints (Live Finish 0.17.1).** Rooms are calibrated to
  a real wall width and the print hangs at true scale with no zoom-out, so a print wider than
  the room's calibrated span (seen at 48 x 96 in) fills and overflows the photo. A range above
  roughly 60 in wide needs a room that covers it or a zoom-out rule before it ships with the
  wall view on. A zoom-out that only triggers where true scale fails is built in 0.18.0, not
  installed. *Verified by query, 2026-10-09.*
- **Crop-ratio swap with a landscape upload.** Uploading a landscape image into the live
  preview swaps the crop ratio. Reported 29 Sep 2026; cause and fix not recorded.
- **Canvas still needs fixes** for finishes, stretched edges and frames (reported).
  Unmatched edge or finish values (for example "Stretched Edge", "Metallic") have no
  look of their own and must be mapped by hand or by a setup-tool question.
- **Picture frame landscape.** Presentation and orientation both need a layout on the same
  print element, so they cannot be two independent options. Plan: orientation top-level,
  presentation as a child option per orientation. Whether an upload crop, Adjust and
  fulfillment handle a placeholder turned 90 degrees is **not verified**: test on baseline.
- **Mapped previews carry another size's GLB after a copy.** A 15oz mug template carried
  the 11oz template's `glb_blob_hash_key` values and rendered on the 11oz model. Compare the
  hash keys across sizes when a preview shows the wrong product. *Verified by reading
  source, 5 Oct 2026.*
- **Mug design range failed review (6 Oct 2026).** Causes found: (a) an uploaded logo is
  cropped to fill unless the element's crop is turned off through an option value's
  image crop flag substitution (`17_DESIGN_TOOL.md` § Element Substitution Types);
  (b) the main product image was static because the design's only preview was a linked
  thumbnail, fixed by the 0.4.0 inline placement; (c) content off-center was the design,
  not the model; (d) a hollow patch near the handle was inverted face winding, fixed in
  the 0.3.1 models. Rework under the design-first rule.
- **Multiple three.js instances.** `WARNING: Multiple instances of Three.js being imported`
  appears on some product pages without the 3D Preview installed: another script on the
  page loads three.js. Harmless, not caused by the tool. experience.pixfizz.com loads three.js
  r142 globally (`window.__THREE__ = "142"`), so any tool importing its own copy logs the
  warning there. *Verified by query, 2026-10-21.*
- **Not verified on either engine:** Safari's luminance mask (Live Finish), the Shopify
  modal, the photo-upload refresh while the 3D viewer is open, and a real phone for every
  version.

### Engineering lessons (CSS 3D and three.js)

- **Perspective must sit on the direct parent of the `preserve-3d` element.** A
  transformed wrapper without `preserve-3d` in between flattens the subtree to an
  orthographic, skewed box.
- **A CSS filter on a 3D panel flattens it**: a sibling at `translateZ(-1px)` then draws in
  front. A filter on a leaf element is fine.
- Sides turned away from the viewer get `backface-visibility: hidden` rather than relying
  on depth sorting; thin sides off the middle of the view need more tilt or a longer
  perspective to read.
- **Never step a drawing loop by a size read from layout.** A hidden stage returns 0 from
  `getBoundingClientRect` and the loop never ends. Floor the step and fall back to a
  default size.
- A canvas weave overlay above about 0.05 opacity reads as a grid, not canvas. Long random
  streaks for brushed metal read as wood.
- three.js r128: hex colors given to materials must be `.convertSRGBToLinear()` when
  `outputEncoding = sRGBEncoding`, or the scene washes out.
- three.js r128 uses **one UV transform per material, taken from `.map`**: a normal map's
  own `repeat` is ignored, so a small tiling normal map stretches once across the mesh.
- Vertices moved on the CPU leave a stale bounding sphere and the mesh is frustum-culled
  in close-ups: set `frustumCulled = false` or recompute the sphere.
- A single-sided cloth casts no shadow toward the light unless
  `material.shadowSide = THREE.DoubleSide`.
- **Loading a newer three.js without an import map:** three 0.160.0 from jsDelivr `+esm`
  (core, and addons as `examples/jsm/.../X.js/+esm`) loads with dynamic `import()` from a
  classic script on a Shopper page; the addons resolve to the same core module, so there is
  one THREE instance. *Verified by query, 2026-10-18.*
- three.js r160 `ShaderMaterial`: the program prefix already defines the tone mapping
  functions, so `#include <tonemapping_pars_fragment>` in a custom fragment shader is a
  redefinition error; `colorspace_fragment` is fine.
- For printed products in WebGL, Khronos PBR Neutral tone mapping keeps photo colors true;
  ACES washes them out. r160 has no `NeutralToneMapping`, so apply it in the final pass.
- A browser pane that is hidden throttles `requestAnimationFrame`: a WebGL page looks
  frozen mid-animation. Expose a manual step hook for tests and recordings.
- **A turned `preserve-3d` container still takes the clicks.** A container rotated 180
  degrees hit-tests its own flat box, and the visible back face sits behind that plane, so
  `elementFromPoint` returns the container and clicks never reach the face.
  `backface-visibility: hidden` does not help (it hides faces, not the container plane).
  Give every 3D container (wrapper, tilt layer, card, leaves) `pointer-events: none`, give
  `pointer-events: auto` only to the faces, and switch off the faces pointing away per state
  (for example `[data-side=back] .front-face { pointer-events: none }`). Test after each turn
  with `elementFromPoint` at the center of an editable element on the visible face.
  *Verified locally in Chromium, 2026-10-16.*
- **Brushed metal that reads as metal:** an SVG `feTurbulence` data URI with stretched
  frequency (`baseFrequency="0.002 0.9"`, 4 octaves, desaturated, full contrast) as a
  background layer over a metal gradient (`#a9adb2` to `#d4d6d8`) with
  `background-blend-mode: overlay`, drawn over the photo with `mix-blend-mode: darken` at
  about 0.95. Light areas turn to metal with fine grain; darks stay. A striped
  `repeating-linear-gradient` reads as rough lines. *Verified by local render, 2026-10-12.*

---

## 7. Installing on a site

Both engines follow the same pattern. Everything shared lives on the **shopper24
parent**: a child site can only override a snippet the parent already has, it cannot
create one (`13_TEMPLATE_BOUNDARIES.md`). Parent changes are delivered as paste blocks,
never as a tar.

1. **Parent, once per version:** upload the asset (and model or image assets), paste the
   snippet if the version changed it. Verify the version in the console of a product page
   that already carries a mount.
2. **Site check:** the site is a shopper24 child, and its template options have the
   `custom_script`, `hidden` and `hide_from_cart` custom field definitions.
3. **Template check:** the design has a preview page marked as preview (Live Finish needs a
   gallery slide, § 2.6); read the definition page type (`lf_page`) or the design page name
   (`p3d_page`) and the page size before writing the mount.
4. **Mount, a template without the option:** import a **template option archive** that
   carries `lf_mount` or `p3d_mount` (Text, hidden, hide from cart, sorted after the
   customer's options). Do not hand a lab a mount to paste.
5. **Mount, a template that already has the option:** paste the new mount into its Custom
   Script. **The import is create-only**: importing onto a template that already has the
   option makes a second one (`51_CUSTOM_FIELDS_REFERENCE.md`).
6. **A whole range:** import on one template, then copy the option to the rest with
   **Admin → Advanced → Bulk Update Tools** (`18_ADMIN_NAVIGATION.md`). Never delete and
   re-import a live range.
7. **experience.pixfizz.com**, the headline demo site, gets every new Live Finish version
   and every preview family in its best version, alongside any customer install, for
   sales calls, tests, webinars and recordings.

**MSP products share msphub's templates.** On a lab site an MSP product's template link points
at the msphub site's own template, so a template option added there (`lf_mount`, `p3d_mount`)
reaches every site listed under **Product Attributes on Other Sites** on that msphub template
page (one holiday card template already carried `lf_mount` on 23 sites). Enumerate that list
before adding any mount to an MSP template. *Verified by query, 2026-10-06.* So the fix for
an MSP product's preview goes on the msphub template's own option: the lab site has no
template of its own to edit. *Verified live, 2026-10-22.*

**A mount copied from another site carries that site's ids.** `lf_backdrop:
'page:preview-2@<theme_id>/<product_id>'` and the `lf_material_filter` links
(`/site/product/c/<collection>?product=<id>-...&theme=<id>-...`) name the source site's
designs, products and collections, and they do not change on import. An ornament mount
imported this way rendered on a plain background with no backdrop scene, and its material
links pointed at the source site's paths. After any import, rewrite both to the target site's
ids. The 3D Preview equivalent is a wrong `p3d_page` (§ 3.2). *Verified live, 2026-10-14.*

**Test a mount before installing it:** on the live product page, build `#lf-root` or
`#p3d-root` with the mount's data attributes inside the product form's `px-option-selector`,
load the parent asset and call its init. Nothing is written to admin. *Verified 2026-10-06.*

The myPixfizz **Live Finish setup tool** (Tools, `/tools/live-finish`) reads an uploaded
template export, detects the product type (folded card, ornament, card, canvas, metal,
print), asks only what the file cannot answer (finished canvas size, bar depth, which
choice drives the edge), and outputs either the template option import file or the
settings text to paste. Its 1 Oct 2026 redesign was not published at the time of writing.

---

## 8. Definition of done

A preview is not finished until all of this is true.

- **Verified on the live product page, not only in a harness.** A local render verifies
  the file, not the site.
- The ready line shows the expected version and `missing: []`, with no `console.error`
  from the engine.
- The geometry cross-check passes: the shape matches the template page (Live Finish), or
  trim, page name and rim rule pass (3D Preview).
- A real customer upload reaches the preview, and every mapped choice (finish, mounting,
  corners, wrap, color) changes it.
- Checked at desktop and at phone width (375 to 390 px): no horizontal scroll, the
  preview fills the width, tilt or rotate works on a real phone.
- Screenshots taken after one scroll tick (§ 6).
- A PDP layout size or orientation swap re-reads the new template.
- Designed products show the design render before upload, not the sample photo.
- Material looks, depths and thicknesses either confirmed by the lab or stated as
  assumptions.
- Installed on experience.pixfizz.com as well, and the setup tool updated for the version.
- An approved preview it replaces is screenshotted beside it, and the new one is at least as
  good (§ 3.3).

---

## Changelog

- 2026-10-06: New file. Product Previews had no home in this knowledge base. Added § 1 what a product preview is and how it differs from custom design tools, mapped previews and template room-scene pages; § 2 the Live Finish standard (parts, mount keys, looks and mountings, geometry from the template, page behavior, gallery-slide gating, dark sites, reserved query parameters, use outside Shopper); § 3 the 3D Preview standard (architecture, mount and rim rules, Blender models, wrap design rules, version standard); § 4 platform facts both rely on; § 5 status by product family; § 6 known defects, open issues and CSS 3D / three.js lessons; § 7 install pattern; § 8 definition of done. Also § 2.2 declare every core mount key (0.17.1 errors on undeclared keys) and § 2.6 the string "false" flag trap on imported mount options. Source: claude-chat, vault-doc.
- 2026-10-09: § 2.3 canvas weave and metal corner rules. § 6 grid design previews that never load (lazy observer on a 0 x 0 img) and the wall view overflow on oversize prints (0.17.1). § 7 MSP templates are shared from msphub, enumerate the sites before a mount; test a mount on the live page before installing. Source: claude-chat.
- 2026-10-17: § 2.3 ornament-only bases from Live Finish 0.22.0 (`acrylic` for non-ornament glass); room plates at 4:5 with props inside the middle 85%. § 2.4 read the supplier spec sheet before modelling a preview. § 3.2 the "View in 3D" fallback button as the symptom of a wrong `p3d_page`. § 6 engineering lessons: hit testing on turned `preserve-3d` containers, brushed metal with `feTurbulence`. § 7 a mount copied from another site carries that site's backdrop and material-filter ids. Source: claude-chat.
- 2026-10-22: § 2.2 `lf_mat_variant` may name a template option, `lf_mats` `@flat` printed border, color wrap as a design option read by `lf_wrap_color_variant`. § 2.5 stage resets `transform` on its images (site `img:hover` zoom). New § 2.10 photo plates (one plate for many sizes, objects on the print, full-resolution masks, cylinders, fabric, banners, plate weight, shared preview UI standard) and § 2.11 room scenes at true scale (scale from a height, wall-plane calibration, outpainted rooms, keep-outs). § 3.2 msphub mug `p3d_page` `front` case. § 3.3 Blender GLB pipeline for tool previews; never replace an approved preview. § 4 CDN textures: the 21 Oct and 18 Oct observations reconciled into one safe rule; CDN JS as ES modules; C2PA chunk. § 6 three 0.160.0 `+esm` loading, r160 tone mapping notes, r142 on experience. § 7 MSP preview fixes go on the msphub template. § 8 approved-preview comparison. § 4: experimental `canvas` format for `px-preview` components, JSON on book and theme preview endpoints (commit titles only, not for production use). Source: claude-chat, slack-message.
