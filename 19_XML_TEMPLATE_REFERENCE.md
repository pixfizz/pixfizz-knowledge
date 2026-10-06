# 19 — XML Template Reference

**Authority Scope:** XML template page parameters, filters, and production output behaviour. Platform-level — not Shopper-specific.

_Last updated: 2026-09-09_

---

## What this file covers

XML Templates define the production specification for a Pixfizz design product. The XML controls:
- `<definition>` root attributes (unit, DPI, output format, page count limits)
- Page dimensions and bleed
- Visual guides shown to end users in the Design Tool (safe area, hinge, gutter, layflat spread)
- Snap point behaviour for alignment aids
- Production file grouping and output naming
- Growing spine behaviour for book covers
- PDF layer definitions and separate-file / separate-page fulfillment behaviour
- Set captions displayed in the Design Tool
- Sequential page type cycling in growable sets

Page layouts and page structure are configured in the Design admin — not in the XML. The XML controls production output behaviour only.

For calendar and planner-specific XML (date sequences, `foreachdate`, `<dategen>` etc.) see `23_XML_CALENDAR_REFERENCE.md`.

---

## `<definition>` Attributes

The root element of every XML template definition.

| Attribute | Description |
|---|---|
| `unit` | Unit system for all dimension values. Common values: `inch`, `mm`. |
| `dpi` | Target resolution for production output. |
| `output` | Output file format. Common values: `pdf`, `jpeg`. |
| `minimum-dpi` | Minimum acceptable image resolution. Used for image quality warnings in the Design Tool. |
| `pages` | Starting page count when a new project is created. Required for cut print (photo print) products — must be set to `1`. |
| `min` | Minimum page count. Prevents users from reducing pages below this value. |
| `add` | The increment by which pages are added when a user adds pages to the project. |
| `max` | Maximum total page count for the product. |
| `trimbox` | `true` embeds PDF trimbox metadata in the output file. Used for prepress workflows where the receiver needs trim information embedded in the PDF. |
| `background-transparent` | Seen as `output="png" background-transparent="true"` on a live clear-crystal product. With it, the design previews in WebP and PNG return alpha 0 outside the design; JPEG previews fill white (`61_PIXFIZZ_API.md` § 9). *Verified by query, 2026-09-29; the production file's alpha was not checked.* |

---

## Page Parameters

### Sample — Book Cover

```xml
<page type="cover" position="left-right" width="19.5" height="13.5" hinge="0.4" bleed="0.75" margin="0.5" snap="0,0.525,0.725,1.125">
	<filter type="binding" map="binding" />
</page>
```

---

### Parameter Reference

| Parameter | Description |
|---|---|
| `type` | Refers to the name of the Page in the Design. |
| `position` | `left-right` makes page captions visible on both sides of a single-page spread or cover. |
| `width` | Pre-trimmed width of the production file. Bleed is **not** added on top of this value. |
| `height` | Pre-trimmed height of the production file. Bleed is **not** added on top of this value. |
| `output-name` | Defines the type name of the production file and controls how production PDFs are grouped. Pages with the same `output-name` are grouped into a single multi-page PDF. Default behaviour groups similarly sized pages together. |

**`output-name` on JPEG output.** On `output="pdf"`, several pages sharing one `output-name` are grouped into one multi-page file. On `output="jpeg"` nothing is grouped, and an explicit `output-name` is used verbatim with no page index appended, so two pages carrying the same `output-name` resolve to the same filename and fulfillment fails on the second page with *Duplicate filename when fulfilling project*. On JPEG output, remove `output-name` from interior pages and keep it only on a page that is meant to be a single file of its own (a cover). *Verified by query (two live orders, controlled pair) and by reading source.*
| `hinge` | Used with a binding map (cover spine only). Renders a visible hinge line in the Design Tool and shifts the alignment aid center point. Common for hard cover products. |
| `bleed` | Virtual bleed displayed in the Design Tool to show users where the page will be trimmed. **Has no effect on the artwork output size.** Supports asymmetric values in `top bottom left right` order, e.g. `bleed="10 20 10 25"`. |
| `margin` | Displays a visual safe line to end users. The value is added on top of any bleed value — it is measured from the bleed line, not from the page edge. This is the mechanism for defining a **safe area**. |
| `snap` | Sets custom snap points for alignment aids. Default snap points: page edge and center (horizontal and vertical). If bleed is set but no margin, snap is at the bleed. If margin is set, snap ignores bleed and uses margin instead. If any snap values are defined, elements snap **exclusively** to those values. Multiple values separated by commas, e.g. `snap="0,0.525,0.725,1.125"`. |
| `gutter` | Sets a gutter value on internal pages. Hides artwork from end users in the Design Tool in the area between pages to prevent unwanted content loss within the binding gutter. |

---

## Safe Area

To display a safe area guide to end users in the Design Tool, use the `margin` attribute on the `<page>` element.

- The value is measured **from the bleed line inward**, not from the page edge.
- If no bleed is set, it is measured from the page edge.
- The safe area renders as a visual line in the Design Tool — it does not affect production output.

**Example — 0.5 unit safe area inside a 0.75 unit bleed:**

```xml
<page type="cover" width="19.5" height="13.5" bleed="0.75" margin="0.5">
</page>
```

---

## Filters

Filters are nested within a `<page>` element to add specialist behaviour.

### Growing Spine (Binding Filter)

Used for book covers with a spine that grows based on page count. The `binding` filter references a named `<map>` that defines spine width ranges.

```xml
<filter type="binding" map="binding" />
```

The map is defined separately in the XML definition:

```xml
<map name="binding">
	<val key="8..44">0.375</val>
	<val key="46..132">0.575</val>
	<val key="134..192">0.775</val>
	<val key="194..240">0.975</val>
</map>
```

- Key ranges use `..` notation (e.g. `8..44` = 8 to 44 pages inclusive).
- The value is the spine width for that page range.
- The `<filter>` must be nested inside the cover `<page>` element.
- The `map` attribute value must match the `name` attribute on the corresponding `<map>` element.
- **Every value the filter looks up must fall inside a key range.** A value with no range fails the render with `PrintBook::TemplateError (Entry for N not found in map "binding")`, where N is the value that had no entry (seen for 29 and 33 on a live photobook range). Fix it by adding the missing range to the template's `<map name="binding">`, and check that the ranges leave no gap between them. Adding the `<filter>` alone does not fix it. *Stated by the core developer ("95% sure", 2026-09-24); the errors cleared once the missing entries were added, 2026-09-25. Not verified by reading source.*

#### How the spine is inserted into a cover at render

The `binding` filter inserts the spine at the centre line of the cover page when it renders,
so the rendered cover is the cover page width plus the spine width for the project's page
count (in the storefront editor, `svg.px-page` viewBox width = cover width + spine). Element
coordinates in the cover XML are written against the cover without the spine. Then:

1. An element wholly in the back half (right edge at or left of the centre line) moves left by spine/2.
2. An element wholly in the front half (left edge at or right of the centre line) moves right by spine/2.
3. An element that crosses the centre line keeps its authored position and size. It is **not
   stretched**. It covers the spine only where it overlaps it, and a full-spread element ends
   spine/2 short of both outer edges.
4. Any part of the spine no element covers shows the page background.

Consequences for cover designs and layouts:

- A two-tone cover drawn as "back shape up to the centre line" leaves the spine in the page
  background color, not the back color.
- To color the spine, add a centre strip of the spine color, crossing the centre line, with
  half-width S at least the largest spine / 2 (plus about 1 mm), behind everything. Front and
  back elements sit above it.
- To ground the whole cover in one color: back-half shape + front-half shape + the same
  centre strip. One full-spread shape falls short of the outer edges.
- A full-spread (wraparound) photo frame: oversize it by S on each side (`x = -S`,
  `width = W + 2S`) so it reaches the edges at every spine width. The photo then runs
  continuously across the spine, so the middle of the photo lands on the spine: sample and
  default photos for a wraparound cover should have no central subject.
- A cover page background color cannot be set from a layout (§ Element Permission Flags);
  draw it as shapes.
- **Check the spine in the storefront editor, not the admin design tool.** The admin design
  tool (`/v1/editor?theme_templates=...`) does not insert the spine. In the storefront editor,
  `.px-binding` and `.px-hinge` are editor overlays at 0.5 opacity, not print.

*Verified by query in the storefront editor on a hardcover photobook template at 40 and 160
pages, 2026-10-02.*

### Layflat Spread Page Break

Adds a visual guide line down the center of a spread in the Design Tool. Also activates alignment aid center points to work within each half of the spread independently.

```xml
<filter type="binding-layflat" />
```

Nest this inside the relevant `<page>` element.

---

## Set Parameters

A `<set>` groups one or more `<page>` elements. For calendar/planner products, sets also accept `foreachdate` — see `23_XML_CALENDAR_REFERENCE.md`.

| Attribute | Values | Description |
|---|---|---|
| `count` | `false` / omit | `count="false"` excludes this set from the product page count. Commonly used for covers. Omit to include. |
| `grow` | `true` / omit | `grow="true"` marks this set as the one used when an end user adds new pages to the project. Typically used for interior pages in books or planners. Only one set should carry this. |
| `fulfillment` | `false` / omit | `fulfillment="false"` excludes this set from production artwork generation. Omit to include. |
| `editor` | `false` / omit | `editor="false"` hides this set from the end user in the Design Tool. Omit to show normally. |
| `preview` | `true` / omit | `preview="true"` designates this set as the project preview — the thumbnail shown to users in cart and saved projects. Not to be confused with a design preview. |

**Sample — hidden preview set (common pattern for calendar/planner products):**

```xml
<set fulfillment="false" editor="false" preview="true">
	<page type="preview" bleed="0" width="10" height="10" />
</set>
```

---

## PDF Layers

Layers are defined at the template level using a `<layers>` block. Once defined, elements in the design can be assigned to layers via the admin design tool — similar to Photoshop layer behaviour.

### Sample

```xml
<layers>
	<layer name="Barcode" />
	<layer name="Cutmarks" visibility="fulfillment" separate-file="true" />
</layers>
```

### Layer Attribute Reference

| Attribute | Required | Values | Description |
|---|---|---|---|
| `name` | Yes | Any string | The layer name. Used as default basis for separate filenames. |
| `visibility` | No | `on` (default), `off`, `fulfillment` | `fulfillment` = layer is visible in production files but hidden in the Design Tool and on previews. |
| `separate-file` | No | `false` (default), `true` | When `true`, the layer is fulfilled to a separate file during production. |
| `separate-page` | No | `false` (default), `true` | When `true`, the layer is fulfilled to separate pages appended to the end of the same PDF file. |
| `filename` | No | String with optional placeholders | Only applies when `separate-file="true"`. Overrides the default filename for the separate layer file. Supports `%prod_code%`, `%order_code%`, `%barcode%`. |

### Default Filename Behaviour

When `separate-file="true"` and no `filename` is specified, the separate file is named:

```
{base_name}_{layer_name}.{extension}
```

For example: if the main file is `pages1.pdf` and the layer is named `Cutmarks`, the separate file will be `pages1_Cutmarks.pdf`.

To fulfill multiple layers into the same separate PDF file, give them the same `filename` value.

### `separate-page` Behaviour

When `separate-page="true"`, the layer pages are appended to the back of the main PDF rather than written to a separate file.

Example: a 3-page product with a `Foil` layer set to `separate-page="true"` will fulfill to a 6-page PDF — pages 1–3 are the base layer, pages 4–6 are the Foil layer.

### Controlling Which Pages Generate Separate Layers

By default, every page in the product generates output for a separate-file or separate-page layer — even if that page contains no elements from that layer (resulting in blank pages).

To restrict which pages generate separate layer output, add the `separate-layers` attribute to the relevant `<page>` elements in the template definition:

```xml
<page type="page1" ... separate-layers="Foil">
```

To suppress separate layer output entirely for a page:

```xml
<page type="page2" ... separate-layers="">
```

### Design Tool Use

Layers can also be used as a design aid independent of fulfillment. Once defined, layer visibility can be toggled on and off in the admin design tool, making it easier to access and edit elements on different layers — similar to working with layers in Photoshop.

### PDF Layers in Practice — What Each Setting Does to the Customer and to Production

PDF layers are the platform's name for what PDF viewers call **Optional Content Groups**. They are declared in the `<layers>` block of the XML template definition, and elements in a design are then assigned to a layer in the admin design tool. The production PDF carries them as real layers, so they appear as a layer list when the file is opened in Illustrator or Acrobat.

A layer does one of three jobs depending on its attributes. Pick the job first, then the attributes:

| Job | Attributes | Customer sees it while designing | In the production file |
|---|---|---|---|
| **Design aid only** — group elements so they can be hidden while editing | none beyond `name` (default `visibility="on"`) | Yes | Yes, merged into the main file. The layer changes nothing about production |
| **Production-only artwork** — cut marks, registration lines, a die line, an order barcode | `visibility="fulfillment"` | **Never** — not in the editor, not on previews | Yes |
| **Split artwork into its own file or pages** — a digital foil or white-ink plate, a second printing pass | `separate-file="true"` (optional `filename`) or `separate-page="true"` | Depends on `visibility` | Written to a separate file, or appended as extra pages to the same PDF |

**Assigning and hiding in the admin design tool.** Select the element and assign it to a layer (named groups such as `background`, `artwork`, `ribbon`). Every layer can then be switched on or off for the current editing session under **View Settings** at the top of the design tool. Hiding a layer there is purely a working aid: it does not change the customer's view and does not change production. It is the answer to "I cannot click the element underneath without nudging the one on top." Set the layers up before building a range of designs; retrofitting them design by design costs more than planning them once.

**Replacing an element that sits on a layer.** If a background or other element does not respond to clicks, check which layer it is on and whether that layer is switched off under View Settings. Switch it on, select the element, then replace or delete it. A replacement element must be assigned to the same layer if it is meant to keep the same production behavior. *Not independently verified step by step — walk it on the site before relying on it in support.*

**The opposite job is not a layer setting.** An element that the customer should **see while designing but that must never print** (a colored backing behind white text, a guide) is not done with a PDF layer. It is done by making the element **uneditable and a placeholder**. See `17_DESIGN_TOOL.md` § Show in the Editor, Never Print — the Uneditable Placeholder.

**Front and back printing on a transparent substrate (acrylic).** Production output cannot be mirrored by a setting. Splitting the back-print artwork into its own file with `separate-file` is supported; mirroring that artwork for reverse printing is not a documented capability. An inline page that is itself flipped has been discussed as a route, with the caveat that elements layered over it (text) would not flip with it. **Unconfirmed — needs a worked example built against the production file the lab wants before it is offered to a customer.**

*Verified by reading source (the platform help page on PDF layers, and a live admin walkthrough with a photo lab client, 2026-09-24), except where marked.*

---

## Set Captions

A `<captions>` element can be nested inside a `<set>` to display label text in the Design Tool for the pages in that set. Useful for orienting users in multi-page products.

Three position options:

```xml
<captions>
	<left>Page {{n}}</left>
	<right>Page {{n}}</right>
	<center>Page {{n}}</center>
</captions>
```

- `{{n}}` is replaced with the sequential page number at runtime.
- Use `<left>` and `<right>` for spread sets (two pages side by side).
- Use `<center>` for single-page sets.

---

## Sequential Page Types in Grow Sets

When a `<page>` inside a `grow="true"` set has a comma-separated `type` value, the platform cycles through the listed types sequentially each time the user adds a new set.

```xml
<set grow="true">
	<page type="page01,page02" bleed="0.125" width="11.25" height="8.75" output-name="Pages"/>
	<page type="page03,page04" bleed="0.125" width="11.25" height="8.75" output-name="Pages"/>
</set>
```

- On the first addition: `page01` and `page03` are used.
- On the second addition: `page02` and `page04` are used.
- The cycle then repeats.
- This allows different layout types to alternate in a predictable order without requiring separate set definitions.
- Only relevant when the page is in a set with `grow="true"`.

---

## Product Examples

Common product types with annotated XML definitions.

### Photo Prints (Cut Prints)

```xml
<definition pages="1" unit="inch" add="1" max="1500" dpi="300" minimum-dpi="200" output="jpeg">
	<set count="true" grow="true">
		<page type="print" bleed="0" width="6" height="4" />
	</set>
</definition>
```

- `pages="1"` is required for cut print products — sets the starting project to 1 page.
- `output="jpeg"` — photo prints produce JPEG files, not PDF.
- `add="1"` — user adds one print at a time.
- `max="1500"` — supports high-volume print orders.
- `bleed="0"` — photo prints typically have no bleed.
- `count="true"` and `grow="true"` on the same set — every added print is counted and uses the grow set.

---

### Canvas

```xml
<definition unit="inch" dpi="300" minimum-dpi="150" output="pdf">
	<layers>
		<layer name="scale" visibility="off" />
		<layer name="shadows" visibility="on" />
		<layer name="grid" visibility="off" />
		<layer name="cutmarks" visibility="fulfillment" />
	</layers>
	<set>
		<page type="canvas" bleed="1.25" margin="0" width="18.5" height="22.5" />
	</set>
	<set preview="true" fulfillment="false" editor="false">
		<page type="preview" width="10" height="10" />
	</set>
</definition>
```

- Large bleed (`1.25`) is typical for canvas — the wrap around the frame.
- `margin="0"` explicitly disables the safe area guide.
- `scale`, `grid`, `shadows` layers are design aids with `visibility="off"` or `"on"` — used for visual reference in the editor without being visible to the end user in fulfillment.
- `cutmarks` layer uses `visibility="fulfillment"` — hidden in the editor, present in production output.
- Hidden preview set is the standard pattern for a custom project thumbnail.

---

### Photobook

```xml
<definition pages="24" min="24" max="120" add="4" dpi="300" unit="inch" output="pdf" minimum-dpi="100" trimbox="true">
	<layers>
		<layer name="cover" visibility="on" />
		<layer name="finishing" visibility="fulfillment" />
		<layer name="order-data" visibility="fulfillment" />
	</layers>
	<map name="binding">
		<val key="24..37">0.35</val>
		<val key="38..73">0.35</val>
		<val key="74..93">0.39</val>
		<val key="94..121">0.43</val>
	</map>
	<set fulfillment="false" count="false" preview="true" editor="false">
		<page type="preview" bleed="0" width="5" height="5" />
	</set>
	<set count="false">
		<page type="cover" width="24" height="10.10" bleed="0.8" hinge="0.2" snap="0,0.4,0.8" output-name="Cover">
			<filter type="binding" map="binding" />
		</page>
	</set>
	<set>
		<captions>
			<center>Page {{n}}</center>
		</captions>
		<page type="title" bleed="0.125" width="11.25" height="8.75" output-name="Pages"/>
	</set>
	<set grow="true">
		<captions>
			<left>Page {{n}}</left> <right>Page {{n}}</right>
		</captions>
		<page type="page01,page02" bleed="0.125" width="11.25" height="8.75" output-name="Pages"/>
		<page type="page03,page04" bleed="0.125" width="11.25" height="8.75" output-name="Pages"/>
	</set>
</definition>
```

- `pages="24"` and `min="24"` — book starts at 24 pages and cannot go below it.
- `add="4"` — pages are added in increments of 4 (typical for sheet-based binding).
- `trimbox="true"` — embeds PDF trimbox metadata for prepress.
- Cover is `count="false"` and uses a growing spine via the `binding` map.
- `finishing` and `order-data` layers use `visibility="fulfillment"` — production-only overlays hidden from the customer.
- The grow set contains two page entries, each with two comma-separated types — cycles through `page01→page02` and `page03→page04` sequentially as users add pages.
- `output-name="Pages"` groups all interior pages into a single multi-page PDF; `output-name="Cover"` keeps the cover separate.

---

### Greeting Card

```xml
<definition unit="inch" dpi="300" output="pdf" minimum-dpi="150">
	<set fulfillment="false" editor="false" preview="true">
		<page type="preview-front" bleed="0" width="10" height="10" />
	</set>
	<set fulfillment="false" editor="false" preview="false">
		<page type="preview-back" bleed="0" width="10" height="10" />
	</set>
	<set>
		<captions><center>Front</center></captions>
		<page type="front" bleed="0.125" width="5.25" height="7.25" />
	</set>
	<set>
		<captions><center>Back</center></captions>
		<page type="back" bleed="0.125" width="5.25" height="7.25" />
	</set>
</definition>
```

- Two hidden preview sets — one for front, one for back. Both are excluded from fulfillment and editor. Only the front is designated as the project preview (`preview="true"`); the back has `preview="false"`.
- Captions ("Front", "Back") orient the user in the Design Tool without using page numbers.
- Simple two-page product — no `min`, `max`, `add`, or `grow` needed.

---

## Multi-Page Product Page-Count Rules

- **Booklets (stapled / coil-bound) must have a page count divisible by 4.** The design tool auto-detects page count on upload and warns on bleed or divisibility errors. Pricing for these products is driven by page count alongside size, colour, paper type, and binding.
- **Old softcover templates can carry a page-count “ghost” bug.** Page-count metadata in older softcover templates can become corrupted, letting customers delete or add pages beyond the defined limits. There is no server-side fix yet. Mitigation: copy the affected customer project onto a fresh template and reshare it. Build new softcover products on current templates going forward.

---

## FTP Fulfillment Behavior

### FTP Path Prefix: `originals/` vs `/originals/`

The leading slash makes a significant difference in where files land on the FTP server.

| Path value | Result |
|---|---|
| `originals/` (no leading slash) | Files placed in a subfolder named `originals` **inside** the per-order folder (e.g. `order-1234/originals/`) |
| `/originals/` (leading slash) | Files placed in a top-level `originals` folder at the **FTP root**, independent of the order folder |

Use the relative form (`originals/`) for the standard pattern of keeping original files alongside production files in the per-order folder. Use the absolute form (`/originals/`) only when the lab's FTP structure requires files at a fixed root-level path.

### Sending Original Customer Files to FTP (`_additional_files.json`)

Original customer-uploaded files are **not** copied to FTP by default — only the generated production PDFs/JPEGs are sent. To include the original uploads in the fulfillment output, a fulfillment template named `_additional_files.json` is required.

This template must be named exactly `_additional_files.json` (including the leading underscore). It is configured in the same fulfillment template area as the main job ticket template. Contact Pixfizz support for the template body format, as the exact payload schema is environment-specific.

### JSON Job Tickets: `escape_json` Filter

When outputting custom order or orderline fields into a JSON fulfillment template, always pass the value through the `escape_json` Liquid filter. Without it, any custom field value containing double quotes, backslashes, or newlines will produce invalid JSON and cause job ticket generation to fail silently or error.

```liquid
"customer_notes": "{{ order.custom.notes | escape_json }}"
```

**Rule:** Every custom field value inserted into a JSON string in a fulfillment template must use `| escape_json`. Do not assume the value is safe — customers enter unpredictable content.

### FTP Folder Naming for Job Ticket Routing

When using multiple fulfillment templates that route to FTP, the folder name in the job ticket template must be exactly `Job Tickets` (that exact capitalisation) for the FTP routing logic to work correctly. Non-standard folder names cause routing failures.

---

## Canvas Wrap Geometry — `borderwrap` and `<ipage> zoom`

Derived 2026-08-22 from real exports and confirmed on three samples.

### `borderwrap` is the mirror-wrap depth

```xml
<image borderwrap="44.45" width="393.7" height="393.7" .../>
```

Measured **inward from the image element's own edge, in millimetres**, so

```
print area = image element - 2 x borderwrap
```

Confirmed on two exports (15.5 - 2(1.75) = 12; 17 - 2(2.5) = 12).

**This is not a fulfillment transformation.** `fulfillment_transformations` is `[]`
in both exports — the mirror lives entirely in the layout element.

A canvas is four numbers: print area `W x H`, bleed `b`, mirror depth `m`
(`m <= b`; the remainder prints white).

| Layout | image element size | position | extra |
|---|---|---|---|
| `gallery` | `(W+2m) x (H+2m)` | `(b-m, b-m)` | — |
| `mirror` | `(W+2m) x (H+2m)` | `(b-m, b-m)` | `borderwrap="m"` |
| `color` | `W x H` | `(b, b)` | wrap takes the chosen colour |

`design_options[].crop_aspect_ratio` equals the print area and **must be rewritten
on any size change** — it is the field that silently mis-crops every customer
upload.

**When asking a lab for wrap depth, expect them to answer with stretcher-bar
thickness** (3/4 inch, 1 1/2 inch) instead. That is the bar, not the wrapped
material. Ask again.
**Templates model the fold differently: compare page size less twice the bleed with the nominal size before trusting either.** Two canvas templates in one 10x10 range read: one page 13.25 in with bleed 1.5 in, so page less twice the bleed is 10.25 in and the 0.125 in fold sits inside the trim; the other page 11.75 in with bleed 0.875 in, trim exactly 10 in, fold inside the bleed. Another range put an 8x10 on a 14x16 page with a 2 in bleed, so the trim is 10x12, 1 in a side over the face. A preview or tool that takes the trim as the face must inset by the difference. *Verified by reading source (template exports), 2026-09-27 and 2026-09-29.*

**The platform preview render draws the selected wrap (gallery, mirror, color) into the bleed itself**, so the render's bleed shows what the sides will print. *Verified by reading renders on three sites, 2026-09-27.*

### `<ipage> zoom` is derived, and `crop="true"` means cover, not fit

```
cover(box, X) = max( box_w / X_w , box_h / X_h )
zoom = ( cover(box, print_area) / cover(box, page) - 1 ) x 100
```

At `zoom=0` the referenced page is scaled to **cover** the box. `zoom` is the extra
scale that makes the *print area* cover the box instead, so bleed and wrap fall
outside and are cropped.

Confirmed on three samples including a 5x7 layflat cover (39.024390243902).

**A simpler-looking form is wrong.** Taking `max` over the two axes of
`page / print_area` reproduces the square samples by coincidence and gives 37.5 on
a 16x20 where the answer is 30.

`zoom` does not vary with box size. `left` and `top` are the pan offset and are 0
whenever the box aspect matches the print-area aspect.
**Sign of `left` and `top`.** They move the referenced page under the box, so a box that shows a region right of (or below) the referenced page's center needs a **negative** offset, in percent of the referenced page:

```
left = (refW / 2 - regionCenterX) / refW * 100
top  = (refH / 2 - regionCenterY) / refH * 100
```

The case that settled it: panel pages with an `<ipage>` onto one wall page. With the sign flipped, the two outer panels rendered each other's artwork; symmetric panel sets hide the error. *`left` verified by render on a test template, 2026-09-27. `top` assumed to follow the same convention, not verified.*

### Image elements: `crop="false"` fits, the default fills

On an image element, `crop="false"` fits the image inside the frame (the setting for
logos); the default crops it to fill the frame. **An image upload option overrides this** and
crops to fill unless an `image_crop_flag` substitution turns cropping off
(`17_DESIGN_TOOL.md` § Image crop flag, `22_OPTION_VARIANT_RENDERING.md` § 5.1).
`target_element_name` limits and substitution types: `22_OPTION_VARIANT_RENDERING.md`
§ Template Option Substitutions. A template import, like a design import (§ Design Import),
adds the archive's fonts to the site under new ids and remaps `font=` in the page XML
(`16_PRODUCT_HIERARCHY.md` § Import Behavior). *Verified by query on baseline.pixfizz.com,
2026-10-05.*

## Template Import — `products[].price` Validates Presence

Verified 2026-08-28 by a real import of `__print_product.yml` (Manage Products →
Templates → Import):

| Field | Value |
|---|---|
| Type | `ActiveRecord::RecordInvalid` |
| Extra info | `Validation failed: Price can't be blank` |

`products[].price` had been emitted as `''`. **An empty string is not accepted** —
unlike `products[].image`, where `''` is the fix and `nil` is the failure. The two
adjacent traps want opposite values:

| Path | `nil` | `''` |
|---|---|---|
| `products[].image` | **fails** — `Column 'image' cannot be null` | passes |
| `products[].price` | untested | **fails** — `Price can't be blank`; `'0'` passes |
| `products[].variant_types[].price` | passes | — |
| `products[].variant_types[].variant_values[].price` | — | passes |

**Emit `price: '0'`** on the product row when no formula is wanted. Quoted, per the
digits-only quoting rule.

**Status: verified 2026-08-29.** The archives were regenerated with `price: '0'`
and the template import succeeded.

**Generator lesson.** The two-direction archive diff nearly caught this and did
not, because it asked "nil here, filled there" and the generated file had `''`.
Widen it to flag any path that is non-blank in every reference and
**blank-or-empty** in the generated file, treating `nil` and `''` as the same
condition — then decide which of the two the platform wants, **per path**. They are
not interchangeable and the accepted value has to be recorded per path, not per
type.

Two restatements from the same build, both easy to get wrong when resizing by hand:

- Page geometry inside `print_themes[].templates[].data` is **in millimetres
  regardless of the definition's `unit`**. An inch-unit definition of
  `width="36" height="24"` pairs with a page XML of `width="914.4" height="609.6"`.
- **Cut print size naming is landscape-first in the XML.** A template named `8x12"`
  carries `width="12" height="8"` — the second number is the width. Follow the
  seed, not the name.

## Page Count and `minimum-dpi` Are Definition-Level Decisions

**Photobook page count lives on the first line of the template definition, not on the
product.** Minimum pages, maximum pages and the starting page count are all attributes of
`<definition>` (`min`, `max`, `pages` — see the attribute table above). The starting count
normally equals the minimum. There is no product setting that changes any of them: to change
a book's page range you edit the definition, not the product attribute.

_Verified by reading source (live definitions) and stated on a client call, 2026-09-09._

**Set `minimum-dpi` to the product's real floor, not a reflex 300.** `minimum-dpi` drives the
image quality warning, so a value that does not match the product's actual production
tolerance produces warnings nobody can act on. On a document-copy product — a copy shop
reprinting customer documents rather than doing press work — 150 is the right floor; a 300
gate would reject most of what the shop actually prints and blocks nothing worth blocking.
Press-ready products (business cards, flyers) legitimately sit at 300.

_Stated, not independently verified — the 150 floor is a build decision, not a measured
platform threshold._

---

## Sets With No Fulfillment Output

`fulfillment="false"` on a `<set>` excludes that set from production artwork generation (see
the Set Parameters table). The case worth stating explicitly is the one where **every** set
in a definition carries it:

```xml
<definition unit="inch" dpi="300" output="pdf" minimum-dpi="300">
	<set fulfillment="false" preview="true">
		<page type="design" bleed="0.125" margin="0.125" width="3.75" height="2.25"/>
	</set>
</definition>
```

**The platform renders no production file at all for a product shaped like this.** Nothing is
broken — it is the correct shape for a product whose print file is produced by a custom
browser tool rather than by the editor — but it means anything that tool produces has to
reach production through `_additional_files.json`. If that template is missing or does not
carry the tool's output option, the orderline delivers nothing to production and no error is
raised anywhere. Cross-reference `31_FULFILLMENT_ENGINE.md`.

_Verified by reading source (live definition plus the fulfillment templates on the same
site), 2026-09-09._

Note the sheet convention in that definition: **sheet size = trim + bleed per edge.** The
page above is a 3.50 x 2.00 in trim with 0.125 in of bleed on all four edges, written as
`width="3.75" height="2.25"`. The `bleed` attribute is the guide shown in the editor; it does
not add to the output size (see the parameter table).

### Two-sided output is one file

Both sides of a two-sided product share an `output-name`, so they group into a single PDF.
A double-sided card therefore writes a **2-page PDF into the single print option** — there is
no second output option per side, and adding one would only create another file to exclude
from the fulfillment package.

**Not verified — no double-sided order has been put through the normalised path.** Worth one
test order on a double-sided product before it meets a customer.

---

## Preview Sets — Layers, Scene Scale and Plate Placement

The standard preview-set pattern is a hidden set carrying a `type="preview"` page, with the
production page in a separate plain set:

```xml
<set preview="true" fulfillment="false" editor="false">
	<page type="preview" width="10" height="10" />
</set>
<set>
	<page type="print" margin="0" width="8" height="10" />
</set>
```

The room scene therefore never reaches production; production receives the plain print page.

### Layer roles in a real preview definition

Read from a live framed-print definition:

| Layer | Asset | Role |
|---|---|---|
| background | room photograph, 2500 x 2500 | The scene |
| `grid` | measurement overlay, `visibility="off"` | QA aid only — never shown, never fulfilled |
| `shadows` | semi-opaque tint plate, `z="1"` | Scene lighting, drawn over the placed art |

**The `shadows` plate is scene shadow, not the artwork's own.** The `ipage` carries its own
`shadow_opacity`, `shadow_ox`, `shadow_oy` and `shadow_stdev` (e.g. `0.5`, `1.27`, `1.27`,
`1.27`), so the placed art casts its own drop shadow whether or not a shadow plate is
present. A replacement scene generated with flat diffuse light needs no companion shadow
plate.

_Verified by reading source (layout definition of a live template)._

### A template page can reference a WebP asset directly

`src="db:<id>"` on a page image resolves a WebP asset — the platform already renders WebP
from template data. _Verified by reading source, not by render._

### On clear products, the page mask is the print area, not the outline

A page mask defines where ink goes. On clear or cut products (acrylic, crystal) the print
mask can be smaller than the physical piece: on one round crystal ornament the mask was
87.7% of the page and the lab's facet overlay 96.9%. Do not use the mask as the product
outline in a preview. Where the mask image itself is needed, the SVG design preview carries
its CDN URL (`61_PIXFIZZ_API.md` § 9). _Verified by query, 2026-09-29._

### Preview scene scale is derivable from the ipage

The preview page's `ipage` width in mm, against the trim it represents in inches, gives
mm-per-inch, and everything else follows:

```
mm_per_inch     = ipage_width_mm / trim_inches
scene_width_in  = page_width_mm / mm_per_inch
pixels_per_inch = background_px / scene_width_in
```

Worked example from a live 20 x 30 in preview: `57.996202993793 / 20 = 2.899810` mm per inch;
`254 / 2.899810 = 87.592` in of scene; `2500 / 87.592 = 28.5414` px per inch.

Every size in a range then places by formula against that one scale:

```
w_mm = inches x mm_per_inch
x_mm = page_centre_mm - w_mm / 2
y_mm = (bottom_px x page_mm / image_px) - h_mm
```

The consequence worth having: **a whole size range can be a bottom-aligned centre crop of one
master plate**, expressed in the template XML as a background `width` plus `x` / `y` offset.
No second image, no per-size plate. Preview crops become data rather than assets.

_Verified by measurement, and by re-rendering previews from each generated archive's own XML._

### Plate aspect must be preserved when placing an image into a page rect

Placing a plate of one aspect into a rect of another **stretches it**, and — worse — any rect
measured in plate pixels changes shape on the way into page space, so nested geometry lands
wrong even where the distortion itself is not obvious. Derive the rect from the plate's own
aspect and recentre:

```
H = <rect height in mm>
W = H * (plate_px_w / plate_px_h)
X = page_centre_mm - W / 2
```

Measured case: a 1650 x 2100 plate (aspect 0.7857) placed into a 185.74 x 219.87 mm rect
(aspect 0.8447) drew about 7.5% too wide.

_Verified by measurement on the regenerated archives._

### A preview page's background image element name is what colour substitution binds to

On a two-sided product with per-side previews, the **name of the background image element**
on each preview page is what a colour substitution resolves against. A back-preview page
whose background element carried the front's name painted a front image onto the back
preview. Renaming the element fixed it with no new imagery generated and no change to the
substitution bindings.

_Verified live (fault reproduced and cleared on the live product), 2026-09-08._

---

## Editing an Exported Template Definition — Attribute Order Is Alphabetical

**Attribute order in a Pixfizz template export is alphabetical, not authored order.** Never
anchor a regex on one attribute in order to reach another. Match the tag, then edit inside
it.

The failure is silent. Substitutions shaped like this:

```
<ipage[^>]*template="print"[^>]*height="..."
```

require `template` to appear before `height`. The exporter writes attributes alphabetically,
so `height` sits ahead of `template` and the substitution matches nothing. In the case that
established this, **three of four attribute writes were dropped** and the preview came out
badly misplaced, with no error from the generator and valid XML in the output.

The order-independent shape is: match the tag, rewrite each attribute inside the matched text
wherever it sits, and append the attribute if it is absent.

**Every write must be followed by a read-back assertion** on each attribute — re-parse the
generated archive and compare the values. Nothing else catches this class of miss.

_Verified by reading source and by re-parsing the regenerated archives, 2026-09-08._

---

## Seed Archives — Audit Before Cloning Across a Size Range

A seed template exported from a live product carries stale and size-specific leftovers.
Cloning it across a size range multiplies every one of them. Check for all of the following
before generating:

- **Hardcoded per-size coordinates on the print page.** Decorative or hardware images placed
  at absolute x/y computed for the seed's page size must be recomputed per size, or dropped.
- **Stale preview pages** whose `ipage` geometry describes a different product — e.g. a
  square `ipage` and a scale caption from a square product carried forward onto a 2:3 one.
  Carry it or drop it, but never treat it as a source of truth.
- **"Zoom" preview pages that are not a true multiple of their base.** One measured case was
  2% under a true 1.5x — placed by eye. Regenerate from the derived scale rather than scaling
  the seed's numbers.
- **Accordion / `details` copy inherited from the wrong product**, and the markup faults
  inside it. Three named faults found in one seed, all of which would have been inherited
  into every clone: a stray `</div>` closing the accordion early so the last card sat outside
  it, a duplicated `heading*` id, and two duplicated `collapse*` ids.

_Verified by reading source (seed archive) and by parsing every generated archive._

---

## Layout Content Requires `layout="true"` and a Matching `layout_id`

Applying a different layout (the layout picker, or a variant element substitution of type layout) only replaces elements that came from a layout. The `<page>` must carry `layout_id="<id of a layout in the design>"` and each element that belongs to the layout must carry `layout="true"`. A plain element without `layout="true"` is not replaced: the new layout's content is added on top, and the customer sees two placeholders stacked. A generator that rebuilds pages from layout XML must copy both attributes forward; seed exports already carry them.

A template can hold several named layouts (for example `full` and `matted` for the same frame), and a variant with an element substitution of type layout switches between them. That is the right shape when two sellable versions share one physical product: one product with a variant, not two products that double-count stock.

*Verified live in the design tool (2026-09-21) and stated on a client call (2026-09-22).*
## Element Rotation in Page XML: `rotate`

**Platform-level (Pixfizz CMS).** `rotate="90"` on an element turns it a quarter turn **clockwise about its own center**. Write the box at its unrotated size, positioned as if unrotated, and add the attribute. Typical use: a landscape layout on a portrait print page. A 254 x 203.2 mm image at x -25.4, y 25.4 on a 203.2 x 254 mm page renders turned and filling the page.

- `rotation="90"` and `angle="90"` are ignored (render unchanged).
- `erotation="false"` is the editor permission flag, not a rotation.
- A placeholder holding a customer upload fills the unrotated box and turns with it in the platform render.

*Verified by render, 2026-09-29. Not verified: the Adjust crop dialog and the production file for a rotated placeholder (needs a test order).*

## Element Permission Flags

**Platform-level (Pixfizz CMS).** The element flags seen across real design exports are
`edit`, `layout`, `move`, `resize`, `eborder`, `ebordercolor`, `eborderradius`, `eopacity`,
`erotation`, `elayer`, `emask` and `z`. There is **no `ecolor` flag**: to stop a customer
recoloring or deleting a shape, use `edit="false"`. `move="false" resize="false"` alone do
not stop either. Editor behavior of a locked element: `17_DESIGN_TOOL.md` § Locking an
Element.

A locked background shape in a layout looks like:

```
edit="false" layout="true" move="false" resize="false" z="-31"
```

- **Shape `opacity`** (e.g. `opacity="0.023"`) is rendered by the editor as a group opacity
  and honored by the fulfillment renderer. Stacked translucent locked shapes make a gradient
  that prints. *Verified by query (editor and fulfillment render), 2026-10-04.*
- **`bgcolor` on a layout's page tag is not applied on a layout swap.** The page keeps the
  design's background color, so a background color that must change with the layout has to
  be drawn as shapes. *Verified by query, 2026-10-02.*

*Flag list verified by reading 16 design exports; locked-element behavior verified by query
in the storefront editor, 2026-10-02.*

## Text Elements: `shrink`, `valign` and Font Size

**Platform-level (Pixfizz CMS).**

- **`fontsize` uses the page geometry unit** (millimetres, like `x`, `y`, `width`,
  `height`). *Verified by query, comparing to rendered proofs, 2026-10-04.*
- **`shrink="true"` renders the text at the largest size at which the wrapped text fits the
  box.** In a name box one line high and 48 mm wide: a short name prints full size, a longer
  name slightly smaller on one line, a very long name wraps to two lines at about half size;
  long job titles dropped to about 6 pt. Size the box and the option's `max_length` together
  so the longest accepted text still prints at a readable size (9 pt or more is the working
  floor). *Verified by server render on baseline.pixfizz.com, 2026-10-05.*
- **`valign="top|center|bottom"`** sets vertical alignment in the box; the default is `top`.
  With `valign="center"` and `shrink="true"`, one line and two wrapped lines both sit
  centered. *Verified by reading source (editor bundle 20261002143509) and by server render
  on baseline.pixfizz.com, 2026-10-06.*

## Mug Wrap Designs: The Readable Front Zone

**Template-level (the Pixfizz Templates mug range).** Content a customer must read sits
within page centre ± 0.83 × mug radius, about 48 degrees each side of the front. That is
2.7 in wide on an 11oz mug (3.25 in diameter) and 2.82 in on a 15oz mug (3.4 in diameter).
*Rule set for the mug template range, 2026-10-05.*

## Layouts Travel With a Template Export

**Corrected 2026-09-24.** The entry previously here said a template export does not carry the layouts linked to the template. That is wrong for the admin template export.

A template export (`__print_product.yml`) carries the design's pages (`templates:`) **and** its `layouts:` in the same file, together with every product on the template, the referenced images, fonts and, where the print product has them, the mapped-preview GLB files (`glb_files/`). Ask for the template export alone when the template, its designs, layouts or products are needed; do not ask separately for a design theme export or per-product exports.

What still holds:

- **There is no API route for layouts.** They cannot be read or written through the admin API, so they cannot be moved by script.
- **Relinking can still be needed.** A layout that the template uses but that belongs to another design theme is not guaranteed to come across; check the layout picker after import.
- **Layouts cannot be exported on their own.** The export is the template (or the design theme), not a layouts-only file.
- **Deleting layouts from designs removes the links to them.** A design can use layouts linked from another design on the same template, and in some cases from another template. A scripted cleanup that deleted layouts from a range's designs left the linked layouts unlinked, and relinking them by hand took about two hours. Export the template before any scripted layout cleanup. *Observed on a client range and stated by the core developer, #development, 2026-09-22.*

*Verified by reading source (a real template export, 2026-09-22) and stated on a client call, 2026-09-22. The 2026-09-12 #development statement this replaces was probably describing a layout shared from another theme, or the API.*

### Linked Layouts drive the layout picker, separately from `layout_id`

What the customer editor offers in its layout picker comes from the design's **Linked
Layouts** (admin, design page, Linked Layouts > Link). A link is to a whole source design:
every layout in it is offered, and Unlink removes them all ("Unlink all layouts from
<design>").

- **A design import does not create these links,** even when every page carries a correct
  `layout_id`. Imported designs show no layouts in the picker until they are linked.
- Deleting a layout from the source design removes it from every design that links that
  source.
- **Deleting a design is a soft delete.** Its admin page still opens by URL, its code is
  cleared, and it stays in the Link Layouts modal list with a blank code. **Links to a
  deleted source design are not removed:** designs that linked it keep offering its layouts.
  After deleting a layouts source, unlink it from every design that linked it. Never link a
  row with a blank code in the Link Layouts modal: it is a deleted design.
- Admin form endpoints (paths relative to `admin.pixfizz.com/site/<site>/`), as called by the
  modal and the Unlink buttons:
  - Link: `POST admin/print_theme/link_layouts?theme=<design id>&inherited_print_theme_id=<source id>`.
    Returns 200; returns 500 when that source is already linked.
  - Unlink: `POST admin/print_theme/unlink_layouts/<design id>?inherited_print_theme_id=<source id>`
    (the form on each Linked Layouts row; send its inputs).
  - Read back: count the `form[action*="unlink_layouts/"]` per `inherited_print_theme_id` on
    the design page.
  - The Link Layouts button (`button[data-onclick=openLinkLayoutsModal]`) stays disabled
    until the page previews load.

*Verified by query, 2026-10-02.*
## Template Changes Do Not Reach Existing Projects

**Platform-level (Pixfizz CMS).** A project keeps the page geometry it was created with. Changing the template afterwards (page size, bleed) does not update projects that already exist:

- Existing projects keep the old dimensions.
- A refulfill (delete the generated file, force refulfill) does resend to the fulfiller, but with the project's old page size.
- New projects pick up the changed template.
- The only fix for an existing project is editing that project's page XML, one project at a time. A spec change on a live range therefore leaves every open project on the old spec.

The same holds when a design is copied to a new size: its layouts stay at the old size until **Resize** is used on the layouts.

*Stated by the core developer (#development, 2026-09-25) and on calls, 2026-09-28 and 2026-09-29. Not verified by test.*

**Design pages do not follow a size change either.** Changing a size on a template that
already has designs leaves every design page and layout at the old geometry, each to be
edited on its own. Changing the hardcover size on one existing photobook template meant
editing about 500 pages individually. Cost a size change on a live range before agreeing
to it. The page XML can be rewritten by script through the admin page form (§ Editing
Design Pages and Layouts in Place), which turns the work into a batch job; that has not
yet been done for a size change. *Reported by Alex (kbsync Phase 1 scan, October 2026). Not verified by test.*

## Editing Design Pages and Layouts in Place

**Platform-level (Pixfizz CMS).** A design page or layout can be changed without
re-importing the design:

- Read: `GET /site/<site>/admin/print_pages/<id>/edit`. The XML is in the Ace mount script as
  `const content = "..."` (a JSON string), not in a textarea.
- Write: `POST /site/<site>/admin/print_pages/<id>` with `_method=patch`, the
  `authenticity_token` from that page's form, and `page[data]`. Route list:
  `18_ADMIN_NAVIGATION.md`.
- Layouts in a layouts design are print pages, so the same path edits them.
- The saved XML comes back with CRLF line endings. Normalize before diffing.

*Verified by query, 2026-10-03 and 2026-10-04.*

## Design Import (`__print_theme.yml`)

**Platform-level (Pixfizz CMS).** A design is imported from the template page's **Import
Design** button (form `POST /site/<site>/admin/print_theme/import_print_theme/<template id>`,
field `exported_file`).

### Format

*Verified by reading a real design export, 2026-09-29.*

- A `.tar.gz` holding `./assets/`, `./fonts/`, `./glb_files/`, `./images/` and `./pdfs/`
  (files named by id), plus `./__print_theme.yml` at the root.
- `__print_theme.yml` is **one design mapping** at the root (the same keys as one entry of
  `print_themes` in a template export), followed by `__asset_map`, `__image_map`,
  `__pdf_map` and `__font_map`.
- Map keys are quoted ids; inner keys are Ruby symbols written unquoted (`:name:`,
  `:original_filename:`, `:description:`). A quoted `':name'` loads as a string, not a
  symbol.

### What an import does

*Verified by query (designs imported, re-exported and read back), 2026-09-29, 2026-10-02 and
2026-10-05.*

- Creates a **new design with a new id**. Page, image, asset and font ids in the tar are
  remapped. Layouts carried inside the tar get new ids, so a page cannot link to them by
  `layout_id` in the same tar.
- **Fonts:** files in `fonts/` with `__font_map` entries are created as site fonts (named
  from `__font_map`), and `font=` attributes in the page XML are rewritten to the new site
  font ids. A font whose name matches an existing site font is not duplicated. With `font=`
  already set to the site's own font ids and the entries under the existing names, the ids
  come back unchanged and no new fonts are created.
- **Images:** every `db:<id>` image is copied to a new id, including an image that already
  exists on the site. Element substitutions match by element name, so the copy does no harm.
- **`layout_id`** on a page is kept when it points at a layout already on the site.
- `linked_assets` + `__asset_map` become the design's preview images; `preview_img` (an
  asset name) becomes the design preview.
- `edit="false"` and the `e*` element flags are kept verbatim.
- An import does **not** create Linked Layouts (§ Layouts Travel With a Template Export).

### Workflow for a generated batch of designs

1. Import one design first, carrying every new font. Read the new font ids from
   Admin > Fonts, then build the rest with `font=` set to those site ids and only the fonts
   each design uses. Never ship the same new font in 30 tars before the first import has
   created it.
2. If the designs use new layouts: import the layouts design first, export it (or the
   template with that design) and read the new layout ids.
3. Build the designs with each page's `layout_id` set to those ids (or to layouts already on
   the site) and every layout element carrying `layout="true"`.
4. Import the designs and read one back by export.
5. Link the layouts designs to every new design under Linked Layouts, unlink any deleted or
   superseded source, and read the links back from each design page.

## Open Platform Question — `fulfillment` on Layers

Logged 2026-08-24 and **not implemented**. Recorded here only so it is not re-proposed as an
existing capability:

> Possible to add a `fulfillment=false` parameter on **layers** in XML, alongside
> `visibility=on/off/fulfillment/editor` and split-pdf?

Today, layer-level control is `visibility` plus `separate-file` / `separate-page` as
documented above. There is no layer-level `fulfillment` attribute. Do not document one.

---

## Changelog
- 2026-04-03: Created from platform documentation provided by AdeB. Covers page parameters, safe area, growing spine, and layflat spread.
- 2026-04-03: Added PDF Layers section — layer attributes, separate-file, separate-page, per-page layer control, filename placeholders.
- 2026-04-03: Added Set Parameters section — count, grow, fulfillment, editor, preview.
- 2026-04-03: Added definition attributes, captions, sequential page types, and four annotated product examples (photo prints, canvas, photobook, greeting card).
- 2026-05-27: Added FTP Fulfillment Behavior section — FTP path prefix behavior (originals/ vs /originals/), _additional_files.json for sending original uploads to FTP, escape_json filter requirement for JSON job tickets, Job Tickets folder naming rule. Source: Fireflies calls, Slack #dev.
- 2026-06-15: Added Multi-Page Product Page-Count Rules — booklet page count must be divisible by 4; old softcover templates can carry a page-count ghost bug (mitigation: rebuild on a fresh template). Source: slack-kb-sync (booklet rules; softcover bug).
- 2026-08-29: Added Canvas Wrap Geometry — `borderwrap` is the mirror-wrap depth measured inward from the image element in mm (`print area = element - 2 x borderwrap`), the three canvas wrap layouts fully parameterised from print area / bleed / mirror depth, the `crop_aspect_ratio` rewrite requirement on any size change, and the derivation of `<ipage> zoom` (with the plausible-but-wrong simpler form called out). Added Template Import — `products[].price` validates presence, so `''` aborts the import with `Validation failed: Price can't be blank`; emit `price: '0'`, note this is the opposite of `products[].image`, and widen the archive diff to treat `nil` and `''` as one blank condition recorded per path. Restated mm page geometry regardless of definition unit, and landscape-first cut print naming. Source: claude-chat.
- 2026-08-29: Confirmed by re-import that `price: '0'` on the products row is accepted — the template import succeeds. Source: claude-chat.
- 2026-09-09: Added that photobook page count (min, max, starting) lives on the first line of the template definition and is not a product setting, and that `minimum-dpi` should be the product's real floor rather than a reflex 300. Added the all-sets-`fulfillment="false"` case — the platform renders no production file, so a tool's output must travel via `_additional_files.json` — with a real definition, the sheet = trim + bleed per edge convention, and two-sided output as one file via a shared `output-name` (not verified). Added Preview Sets — layer roles read from a live definition, the `ipage`'s own drop-shadow attributes, WebP via `src="db:<id>"`, the derivation of scene scale and per-size placement from the ipage so a whole range is one master plate plus offsets, plate-aspect preservation when placing into a page rect, and the background image element name as what colour substitution binds to. Added that attribute order in a template export is alphabetical, so a regex anchored on one attribute to reach another silently matches nothing — match the tag, edit inside it, read back every write. Added the seed-archive audit before cloning across a size range. Recorded the layer-level `fulfillment` parameter as an open, not-implemented platform question. Source: claude-chat, fireflies-call, notion-dashboard.
- 2026-09-19: Added that layouts are excluded from template exports and that the API cannot manage or transfer them, so layouts must be recreated and relinked after any template import; a design theme export (`__print_theme.yml`) is the route for a bulk layout move. Source: slack-message (#development).
- 2026-09-24: Added "PDF Layers in Practice": the three jobs a layer does (design aid, production-only, split file), View Settings for hiding layers while editing, replacing an element on a layer, the pointer to the uneditable-placeholder technique for show-but-never-print, and front/back acrylic marked unconfirmed. Replaced "Layouts Do Not Travel With a Template Export" with the corrected rule: the template export carries layouts. Added the `output-name` JPEG duplicate-filename rule and the `layout="true"` + `layout_id` requirement for layout swaps. Source: fireflies-call, claude-chat.
- 2026-09-29: Binding map missing-entry error and fix. `<ipage>` left/top sign convention. Canvas fold inside trim vs bleed; render draws the wrap into the bleed. `rotate="90"` element rotation. Layout links break when layouts are deleted from designs. Template changes do not update existing projects; refulfill resends old geometry. Source: claude-chat, slack-message.
- 2026-10-06: Added `background-transparent` to definition attributes. Growing Spine: how the spine is inserted at render and the consequences for cover designs. Preview Sets: the page mask is the print area, not the outline, on clear products. Added Element Permission Flags (flag list, no `ecolor`, shape `opacity`, `bgcolor` not applied on layout swap), Text Elements (`shrink`, `valign`, `fontsize` unit) and the mug readable front zone. Layouts: Linked Layouts drive the picker, import does not create them, soft-deleted designs keep their links, link/unlink endpoints. Template Changes: design pages do not follow a size change. Canvas Wrap Geometry: image elements, `crop="false"` fits and image upload options override it. Added Editing Design Pages and Layouts in Place, and Design Import (`__print_theme.yml` format, id remapping, fonts, batch workflow). Source: claude-chat, vault-doc.
