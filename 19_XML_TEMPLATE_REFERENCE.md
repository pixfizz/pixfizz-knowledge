# 19 — XML Template Reference

**Authority Scope:** XML template page parameters, filters, and production output behaviour. Platform-level — not Shopper-specific.

_Last updated: 2026-05-27_

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

## Changelog
- 2026-04-03: Created from platform documentation provided by AdeB. Covers page parameters, safe area, growing spine, and layflat spread.
- 2026-04-03: Added PDF Layers section — layer attributes, separate-file, separate-page, per-page layer control, filename placeholders.
- 2026-04-03: Added Set Parameters section — count, grow, fulfillment, editor, preview.
- 2026-04-03: Added definition attributes, captions, sequential page types, and four annotated product examples (photo prints, canvas, photobook, greeting card).
- 2026-05-27: Added FTP Fulfillment Behavior section — FTP path prefix behavior (originals/ vs /originals/), _additional_files.json for sending original uploads to FTP, escape_json filter requirement for JSON job tickets, Job Tickets folder naming rule. Source: Fireflies calls, Slack #dev.
