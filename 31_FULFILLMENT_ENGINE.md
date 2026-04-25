# 31 — Fulfillment Engine

**Authority Scope:** Job ticket schema and generated file logic only.

_Last updated: 2026-04-10_

---

# 08 — Fulfillment Templates (Job Tickets)

Fulfillment Templates generate **job tickets** for external labs and systems. A “job ticket” is any machine-readable payload (JSON/XML/TXT) plus the produced artwork files that a fulfiller needs to manufacture and ship an order.

This document covers:
- A clean **Pixfizz Default Fulfillment JSON** that’s safe to share with any integrator.
- How to map Pixfizz Liquid objects into that JSON (including **options/variants**).
- The three layers of a real integration: **asset generation**, **transport/auth**, and **payload schema**.
- Filename Template patterns and the operational settings that affect generated files.

---

## Mental model: the 3 layers

### 1) Asset generation (files)
Controls *what files exist* and *what they are named*.
Common settings you’ll see in Admin:
- **Filename Template**
- **Single Page Output**
- **Multiple Cut Print Copies**
- **Color Profile** (often sRGB)
- **Enable Perfectly Clear** (image enhancement)
- **Split By Orderline** (fan-out into multiple requests)

These settings affect:
- number of `generated_files`
- `file.type` values (e.g., `cover`, `pages1`, `pages2`)
- whether quantity is represented as **file duplication** vs **a numeric quantity field**
- whether one order generates **one request** vs **N requests** (one per orderline)

### 2) Transport + auth (delivery)
Controls *how the payload is delivered*:
- Hotfolder / shared drive (TXT manifests, file drops)
- HTTP API POST/PUT (JSON/XML payloads)

Auth commonly includes:
- Static headers (API keys, subscription keys)
- OAuth2 token retrieval + Bearer token injection

> Keep transport/auth configuration separate from the payload schema. Don’t bake auth into the JSON contract.

### 3) Payload schema (the job ticket)
The vendor-specific shape (JSON/XML/TXT).
Best practice is:
- Define a **canonical internal schema** (Pixfizz Default JSON).
- Build vendor adapters that map canonical → vendor payload.

---

## Pixfizz Default Fulfillment JSON (vendor-neutral)

This is the “baseline contract” for Starter Pack v4.

**Key goals**
- Simple enough to share with any vendor.
- Expressive enough for most products (prints, books, merch).
- Options/variants included as `{ key, value }` pairs.
- Assets included as typed file descriptors.

### Schema: `pixfizz.fulfillment.default.v1`

```json
{
  "schema": "pixfizz.fulfillment.default.v1",
  "order": { "..." : "..." },
  "shipping": { "..." : "..." },
  "jobs": [ { "..." : "..." } ]
}
```

### Field semantics

#### `order`
Commercial + customer context:
- ids, status, timestamps
- totals (numeric)
- notes (order-level)

#### `shipping`
Delivery context:
- `method` is the storefront shipping method label/code
- `ship_to` is the destination address

> For **pickup/collection flows**, many integrations treat `order.address.is_public` as “pickup” (system/public address).

#### `jobs[]`
One producible unit per orderline.
- `job_id` should be stable and unique within the order.
- `quantity` is the commerce quantity (for cut prints use `cut_print_quantity`).
- `print_quantity` is the production quantity (can differ when “Multiple Cut Print Copies” is enabled).

#### `options[]`
A normalized list of template options + product variants:
```json
{ "key": "paper", "value": "lustre" }
```

Rules:
- Exclude `image_upload` / `file_upload` options.
- Exclude blanks.
- Prefer stable identifiers (`code`) when available; fall back to `name`.

#### `assets[]`
A normalized list of generated files:
- `type` (e.g. `cover`, `pages1`, `pages2`, `print`)
- `url`
- `filename`
- `pages` (best-effort; often `1` for cover, and project page count for interiors)

---

## Liquid mapping: Pixfizz Default Fulfillment JSON

Copy/paste template (Shopper + CMS environments). Uses **hard tabs** for indentation.

```liquid
{
	"schema": "pixfizz.fulfillment.default.v1",

	"order": {
		"order_id": "{{ order.code | escape_json }}",
		"external_order_id": "{{ order.id }}",
		"source": "pixfizz",
		"status": "confirmed",
		"ordered_at": "{{ order.confirmed_at }}",

		"customer": {
			"name": "{{ order.first_name | escape_json }} {{ order.last_name | escape_json }}",
			"email": "{{ order.email | escape_json }}",
			"phone": "{{ order.telephone | escape_json }}"
		},

		"totals": {
			"subtotal": {{ order.orderlines_total }},
			"shipping": {{ order.shipping }},
			"tax": {{ order.tax }},
			"discount": {{ order.discount }},
			"total": {{ order.total }},
			"currency": "{{ website.currency_code | escape_json }}"
		},

		"notes": "{{ order.notes | escape_json }}"
	},

	"shipping": {
		"method": "{{ order.shipping_method | escape_json }}",
		"ship_to": {
			"name": "{{ order.address.first_name | default: order.first_name | escape_json }} {{ order.address.last_name | default: order.last_name | escape_json }}",
			"address_1": "{{ order.address.street | escape_json }}",
			"address_2": "{{ order.address.street2 | escape_json }}",
			"city": "{{ order.address.city | escape_json }}",
			"region": "{{ order.address.region | escape_json }}",
			"postal_code": "{{ order.address.postcode | escape_json }}",
			"country_code": "{{ order.address.country.code | escape_json }}",
			"email": "{{ order.email | escape_json }}",
			"phone": "{{ order.address.telephone | default: order.telephone | escape_json }}"
		}
	},

	"jobs": [
		{%- for line in orderlines -%}
		{
			"job_id": "{{ order.code | escape_json }}-{{ forloop.index }}",
			"external_line_item_id": "{{ line.id }}",

			"product": {
				"name": "{{ line.product.name | escape_json }}",
				"code": "{{ line.product.code | escape_json }}",
				"category": "{{ line.product.category | escape_json }}"
			},

			"quantity": {% if line.is_cut_print %}{{ line.cut_print_quantity }}{% else %}{{ line.quantity }}{% endif %},
			"print_quantity": {% if line.is_cut_print %}{{ line.cut_print_quantity }}{% else %}{{ line.quantity }}{% endif %},

			"options": [
				{%- assign opt_first = true -%}

				{%- comment %} Template options {%- endcomment -%}
				{%- for opt in line.chosen_template_options -%}
					{%- if opt.template_option.type == "image_upload" or opt.template_option.type == "file_upload" or opt.value == blank -%}
					{%- else -%}
						{%- unless opt_first -%},{%- endunless -%}
						{ "key": "{{ opt.template_option.code | default: opt.template_option.name | escape_json }}", "value": "{{ opt.template_option_value.code | default: opt.value | escape_json }}" }
						{%- assign opt_first = false -%}
					{%- endif -%}
				{%- endfor -%}

				{%- comment %} Product variants {%- endcomment -%}
				{%- for v in line.chosen_variants -%}
					{%- if v.variant.type == "image_upload" or v.variant.type == "file_upload" or v.variant_value.name == blank -%}
					{%- else -%}
						{%- unless opt_first -%},{%- endunless -%}
						{ "key": "{{ v.variant.code | default: v.variant.name | escape_json }}", "value": "{{ v.variant_value.code | default: v.variant_value.name | escape_json }}" }
						{%- assign opt_first = false -%}
					{%- endif -%}
				{%- endfor -%}
			],

			"assets": [
				{%- for file in line.generated_files -%}
				{
					"type": "{{ file.type | escape_json }}",
					"url": "{{ file.url | escape_json }}",
					"filename": "{{ file.filename | escape_json }}",
					"pages": {% if file.type == 'cover' %}1{% elsif line.project and line.project.page_count %}{{ line.project.page_count }}{% else %}1{% endif %}
				}{% unless forloop.last %},{% endunless %}
				{%- endfor -%}
			]
		}{% unless forloop.last %},{% endunless %}
		{%- endfor -%}
	]
}
```

---

## Filename Templates (default vs customized)

Filename Templates control how output files are named (and optionally which folders they land in).

### Common variables
- `order.code`
- `order.last_name`
- `orderline.id`
- `orderline.barcode`
- `orderline.quantity`
- `print_quantity` (production quantity; may differ from orderline quantity)
- `page_output_name`
- `layer_output_name`
- `idx` (useful for uniqueness across multiple outputs)
- `format` (file extension)

### Operational guidance
- Ensure **uniqueness**: include `order.code` + `orderline.barcode` + `idx`.
- Avoid unsafe filename characters: strip/replace `|`, `/`, `\`, `:` and quotes.
- Use folder routing sparingly (e.g., by category) when a lab watches many hotfolders.

### Example: adjusted filename template
A real-world example that routes into a category subfolder and forces PDF for a specific category:

```liquid
{{ orderline.product.category }}/{{ order.last_name }}_{{ page_output_name }}_{{ order.code }}_{{ orderline.product.category }}_{{ orderline.product.name | replace: "|" }}_{{ orderline.barcode }}_{{ idx }}{% if layer_output_name %}_{{ layer_output_name }}{% endif %}_Q{{ orderline.quantity }}.{% if orderline.product.category == 'MSP-Posters-Collage Prints' %}pdf{% else %}{{ format }}{% endif %}
```

> Note: Liquid `replace` normally takes two arguments: `replace: 'from', 'to'`. Sites often use this pattern to remove pipes from filenames; standardize your preferred sanitization approach per site.

---

## Provider patterns to expect (for adapters)

These patterns show up repeatedly across real integrations:

- **Barcode-first line IDs** (`line.barcode` is the primary key in payloads).
- **sold_to vs ship_to** split (billing identity vs delivery identity).
- **Customs value** blocks conditionally included by country.
- **Page count transforms** (e.g., “interior pages” excludes covers).
- **Component-level manufacturing** (per-file components with attributes like Paper/Finishing/Binding).
- **Pickup vs ship** driven by `order.address.is_public`.
- **Hotfolder manifests** that group by size/media/quantity and list file paths.

Keep the Pixfizz Default schema simple, then implement these as adapter-level transforms.

---

## JSON/XML/TXT formatting tips

- For JSON: use `escape_json` on strings; keep booleans as booleans and numbers as numbers.
- Avoid producing the string `"null"` when you mean JSON `null`.
- For XML: use `escape` (and CDATA where required by a provider).
- Validate the output (JSON validity, XML well-formedness) before sending to a provider.

---

## Worked Example — QR Code Element + Fulfillment Transformation (Oxford & Rose)

Source: Oxford & Rose, 2026-04-01. Concrete example of using a design element plus
a fulfillment transformation to inject a **per-order unique value** into the
production artwork at fulfillment time — without requiring the shopper to do
anything at design time.

### The pattern

1. **In the design tool** — a QR code design element is placed on the template.
   The element is tagged with a reference to the Shopify order line ID (a Liquid
   expression that evaluates to an empty placeholder in the design preview).
2. **At fulfillment time** — a fulfillment transformation detects the tagged
   element and **dynamically generates a unique QR code** on each production file,
   encoded with the real order line ID for that order.
3. **The shopper** never sees the QR code in the design tool preview. They see a
   normal card. The production file has the QR code embedded at the tagged
   location, ready for scanning in the fulfillment / delivery workflow.

### Why this matters

Before fulfillment transformations supported this pattern, injecting per-order
dynamic content (QR codes, barcodes, serial numbers, routing codes) required
either:

- Post-processing the generated PDF with an external script (extra moving parts,
  slower, hard to debug), or
- Generating the code in JavaScript inside the design tool (exposes the code to
  the shopper, who could remove or edit it).

The transformation approach is clean: the design template stays the same for
every customer, the shopper cannot tamper with the injected content, and the
logic lives in one well-defined place.

### When to reach for this pattern

Any time a customer order needs a **per-order dynamic production artefact** that
should not be editable by the shopper — QR codes for tracking, serialized
numbers, vendor routing codes, delivery confirmation codes, etc.

Use a tagged element in the design tool + a fulfillment transformation. Do not
embed the dynamic value in the design tool itself.

The `barcode_datauri` Liquid filter is a related but distinct capability — that
filter generates a barcode inline in Liquid-rendered templates (job tickets,
emails), whereas this pattern generates a per-order graphic on the production
artwork during fulfillment processing.

---

## `_additional_files.json` — Delivering User-Uploaded Files to FTP

The `_additional_files.json` fulfillment template controls which additional files are fetched and placed on the FTP/hotfolder alongside generated artwork. It supports two `source` formats:

- `"source": "https://..."` — fetch the file at this URL and deliver it
- `"source": { "content": "..." }` — write literal string content as a file

### Delivering `file_upload` option files

To loop all orderlines and deliver any user-uploaded files (from `file_upload` variant or template options) into a folder on the FTP:

```liquid
{% capture pdfinvoice_json_body %}...{% endcapture %}

[
    {
        "source": {
            "url": "https://{{ website.hostname }}/custom/craftmypdf/pdfinvoice/{{ order.id }}.pdf",
            "method": "post",
            "headers": {"Content-Type": "application/json"},
            "payload": "{{ pdfinvoice_json_body | strip | escape_json }}"
        },
        "destination": "/PDF Job Tickets/{{ order.code }}.pdf"
    }

	{%- for line in orderlines -%}

		{%- comment %} Template options — file_upload type {%- endcomment -%}
		{%- for opt in line.chosen_template_options -%}
			{%- if opt.template_option.type == "file_upload" and opt.uploaded_file.url != blank -%}
			,
				{
					"source": "{{ opt.uploaded_file.url | escape_json }}",
					"destination": "/Artwork/{{ order.code | escape_json }}/Uploaded_Files/{{ forloop.parentloop.index }}-{{ opt.uploaded_file.filename | escape_json }}"
				}
			{%- endif -%}
		{%- endfor -%}

		{%- comment %} Product variants — file_upload type {%- endcomment -%}
		{%- for v in line.chosen_variants -%}
			{%- if v.variant.type == "file_upload" and v.uploaded_file.url != blank -%}
			,
				{
					"source": "{{ v.uploaded_file.url | escape_json }}",
					"destination": "/Artwork/{{ order.code | escape_json }}/Uploaded_Files/{{ forloop.parentloop.index }}-{{ v.uploaded_file.filename | escape_json }}"
				}
			{%- endif -%}
		{%- endfor -%}

	{%- endfor -%}
]
```

### Key rules

- The entire `_additional_files.json` must be a **single `[...]` array**. Multiple arrays separated by commas is invalid JSON and will cause a parse error.
- When a fixed entry (e.g. a PDF job ticket) always occupies position 1, hardcode the comma **inside** each conditional block rather than using a flag variable. This avoids the leading-comma problem entirely.
- Do not use generic variable names like `needs_comma` — they can collide with variables set in other fulfillment templates rendered in the same context. Use a unique prefix if a flag variable is needed.
- Use `uploaded_file.filename` (not the option code/name) in the destination path to preserve the original file extension (e.g. `.pdf`, `.psd`, `.ai`).
- Always use `order.code` as the default folder identifier — only use `order.custom.shopify_order_number` when explicitly required for a Shopify site.