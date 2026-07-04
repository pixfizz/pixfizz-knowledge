# 31 — Fulfillment Engine

**Authority Scope:** Job ticket schema and generated file logic only.

_Last updated: 2026-06-30_

---

# 08 — Fulfillment Templates (Job Tickets)

Fulfillment Templates generate **job tickets** for external labs and systems. A "job ticket" is any machine-readable payload (JSON/XML/TXT) plus the produced artwork files that a fulfiller needs to manufacture and ship an order.

This document covers:
- A clean **Pixfizz Default Fulfillment JSON** that's safe to share with any integrator.
- How to map Pixfizz Liquid objects into that JSON (including **options/variants**).
- The three layers of a real integration: **asset generation**, **transport/auth**, and **payload schema**.
- Filename Template patterns and the operational settings that affect generated files.

---

## Mental model: the 3 layers

### 1) Asset generation (files)
Controls *what files exist* and *what they are named*.
Common settings you'll see in Admin:
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

> Keep transport/auth configuration separate from the payload schema. Don't bake auth into the JSON contract.

### 3) Payload schema (the job ticket)
The vendor-specific shape (JSON/XML/TXT).
Best practice is:
- Define a **canonical internal schema** (Pixfizz Default JSON).
- Build vendor adapters that map canonical → vendor payload.

---

## Fulfillment Code Resolution and Precedence

A fulfillment code routes an orderline to the correct fulfillment destination. The code can be set at more than one level, and the platform resolves them by priority:

- **Template-level fulfillment code (highest priority).** A fulfillment code set on an individual template overrides everything else, including location-based fulfillment codes. (Added June 2026.)
- **Location-based fulfillment code.** Applied when no template-level code is set.
- **Product-level fulfillment code.** The base `fulfillment_code` product attribute (see `50_SHOPPER_TEMPLATE_REFERENCE.md`) used where not overridden above.

Practical effect: if a template carries its own fulfillment code, that code wins regardless of location or product configuration. Set a template-level code only when you intend to override location/product routing.

## Pixfizz Default Fulfillment JSON (vendor-neutral)

This is the "baseline contract" for Starter Pack v4.

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

> For **pickup/collection flows**, many integrations treat `order.address.is_public` as "pickup" (system/public address).

#### `jobs[]`
One producible unit per orderline.
- `job_id` should be stable and unique within the order.
- `quantity` is the commerce quantity (for cut prints use `cut_print_quantity`).
- `print_quantity` is the production quantity (can differ when "Multiple Cut Print Copies" is enabled).

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
- **Reverting to the default template flattens folders.** The output filename template controls folder structure. If a customized template that routes into product-category subfolders is reverted to the default, those category subfolders disappear and every file lands flat in the per-order folder. Re-check the template after any reset when a lab relies on category subfolders. Source: #development (2026-06-30).

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
- **Page count transforms** (e.g., "interior pages" excludes covers).
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

### Which accessor: chosen_variants, not chosen_template_options

Shopify identifiers (`shopify_product_id`, `shopify_variant_id`, `shopify_line_id`)
are stored on the orderline as **chosen_variants**, not `chosen_template_options`.
A fulfillment transformation or job ticket that reads them must use:

```liquid
{{ orderline.chosen_variants['shopify_line_id'].value }}
```

Reading them from `chosen_template_options` resolves to nothing, so the QR code
(or barcode, or any injected value) comes out blank with no error.

Diagnostic note: the admin orderline view lists every option under a single
generic "Options" label and does not distinguish variants from template options.
Confirm which bucket a value lives in by inspecting the product attribute
(Variants tab vs template Options tab) or by looping both collections in Liquid.

The `barcode_datauri` Liquid filter is a related but distinct capability — that
filter generates a barcode inline in Liquid-rendered templates (job tickets,
emails), whereas this pattern generates a per-order graphic on the production
artwork during fulfillment processing.

---

## Original Customer Files Are NOT Copied to FTP by Default

Original customer-uploaded files (photos, artwork) are **not** automatically copied to the FTP/hotfolder alongside generated production artwork. They require a separate `_additional_files.json` fulfillment template to be added to the fulfillment settings.

If a client asks "why aren't the originals on the FTP?", the answer is always: `_additional_files.json` is missing or not configured.

---

## `_additional_files.json` — Delivering Additional Files to FTP

The `_additional_files.json` fulfillment template controls which additional files are fetched and placed on the FTP/hotfolder **alongside** the generated production artwork. Use this for anything that needs to land on the FTP beyond the standard rendered files — PDF job tickets, user-uploaded artwork files, text manifests, etc.

The template outputs a single JSON array. Each entry specifies a `source` (where to get the file) and a `destination` (where to put it on the FTP).

### Source formats

`_additional_files.json` supports three `source` formats:

#### 1. Simple URL (string)

Fetch the file at this URL and deliver it directly:

```json
{
	"source": "https://example.com/path/to/file.pdf",
	"destination": "/Artwork/ORDER-123/file.pdf"
}
```

Use for: delivering user-uploaded files, fetching existing assets by URL.

#### 2. HTTP request (object with `url`, `method`, `headers`, `payload`)

Make an HTTP request (typically POST) and deliver the response as a file:

```json
{
	"source": {
		"url": "https://yoursite.pixfizz.com/custom/craftmypdf/pdfinvoice/12345.pdf",
		"method": "post",
		"headers": {"Content-Type": "application/json"},
		"payload": "{{ captured_json_body | strip | escape_json }}"
	},
	"destination": "/PDF Job Tickets/ORDER-123.pdf"
}
```

Use for: generating PDF job tickets or invoices via external API (e.g. CraftMyPDF), calling any web service that returns a file.

#### 3. Literal content (object with `content`)

Write literal string content directly as a file:

```json
{
	"source": { "content": "Order ORDER-123 ready for production" },
	"destination": "/Manifests/ORDER-123.txt"
}
```

Use for: simple text manifests, trigger files, status markers.

---

### Pattern 1: PDF Job Ticket via CraftMyPDF

This pattern generates a PDF invoice/job ticket by POSTing order data to a CraftMyPDF API endpoint. The resulting PDF lands on the FTP alongside the production artwork.

**How it works:**

1. A `{% capture %}` block builds the full JSON payload containing order details, customer info, shipping/pickup logic, line items, options, and generated filenames
2. The captured body is passed through `| strip | escape_json` (double-encoding — the JSON is itself embedded inside the outer `_additional_files.json` JSON)
3. The `source` object POSTs to the CraftMyPDF endpoint, which returns a rendered PDF
4. The PDF is delivered to the specified FTP destination path

**Template structure:**

```liquid
{% capture pdfinvoice_json_body %}
{
	"template_ID": "YOUR_CRAFTMYPDF_TEMPLATE_ID",
	"store_image_url": "https://{{ website.hostname }}/path/to/logo.svg",

	{%- comment %} Site-specific label strings go here {%- endcomment -%}
	"order_label": "Order",
	"customer_details_label": "Customer Details",
	"order_no_label": "Order #",
	"order_date_label": "Order Date",
	"order_delivery_method_label": "Delivery Method",
	"special_instructions_label": "Special Instructions",
	"order_sub_total_label": "Order Sub Total",
	"discount_total_label": "Discount",
	"shipping_total_label": "Shipping",
	"tax_total_label": "Tax",
	"total_label": "Total",
	"source_label": "Source",
	"orderline_description_label": "Description",
	"orderline_options_label": "Options",
	"orderline_qty_label": "QTY",
	"orderline_each_label": "Each",
	"orderline_total_label": "Total",

	{%- comment %} Standard order data — reusable across sites {%- endcomment -%}
	"pixfizz_order": "{{ order.code }}",
	"order_confirmed_date": "{{ order.confirmed_at | date: "%Y-%m-%d %H:%M" }}",
	"order_total": "{{ order.total | currency }}",
	"order_orderlines_total": "{{ order.orderlines_total | currency }}",
	"order_discount": "{{ order.discount | currency }}",
	"order_shipping_cost": "{{ order.shipping | currency }}",
	"order_tax": "{{ order.tax | currency }}",

	{%- comment %} Order code — use Shopify order number if available {%- endcomment -%}
	{%- if order.custom.shopify_order_number == blank %}
	"order_code": "{{ order.code }}",
	{%- else %}
	"order_code": "{{ order.custom.shopify_order_number }}",
	{%- endif %}

	{%- comment %} Pickup vs Ship logic {%- endcomment -%}
	{%- if order.address.is_public or order.custom.shopify_shipping_service == "" %}
	"flash_header": "Pick Up",
	"ship_to_label": "",
	"order_shipping_method": "Pick Up (in store)",
	"delivery_firstname": "",
	"delivery_lastname": "",
	"delivery_email": "",
	"delivery_address": "",
	{%- else %}
	"flash_header": "SHIP",
	"ship_to_label": "Ship to:",
	{%- if order.custom.shopify_shipping_service == blank %}
	"order_shipping_method": "{{ order.shipping_method | escape_json }}",
	{%- else %}
	"order_shipping_method": "{{ order.custom.shopify_shipping_service | escape_json }}",
	{%- endif %}
	"delivery_firstname": "{{ order.first_name | escape_json }}",
	"delivery_lastname": "{{ order.last_name | escape_json }}",
	"delivery_email": "{{ order.email | escape_json }}",
	"delivery_address": "{%- if order.address.street != blank %}{{ order.address.street | escape_json }}<br>{%- endif %}{%- if order.address.street2 != blank %}{{ order.address.street2 | escape_json }}<br>{%- endif %}{%- if order.address.city != blank %}{{ order.address.city | escape_json }}<br>{%- endif %}{%- if order.address.region != blank %}{{ order.address.region | escape_json }}<br>{%- endif %}{%- if order.address.postcode != blank %}{{ order.address.postcode | escape_json }}<br>{%- endif %}{%- if order.address.country.name != blank %}{{ order.address.country.name | escape_json }}{%- endif %}",
	{%- endif %}

	{%- comment %} Payment status {%- endcomment -%}
	{%- if order.payment_reference == blank %}
	"payment_status": "Payment Due",
	{%- else %}
	"payment_status": "Paid",
	{%- endif %}

	"source": "Pixfizz",
	"customer name": "{{ user.first_name | escape_json }} {{ user.last_name | escape_json }}",
	"customer_email": "{{ user.email }}",
	"email": "{{ order.email }}",
	"phone": "{{ order.telephone }}",
	"order_notes": "{{ order.user_notes | escape_json }}",

	{%- comment %} Rush fee extraction from extra_fees {%- endcomment -%}
	"rush_flash": "{% for fee in order.extra_fees %}{{ fee.name | replace: 'Rush Fee','RUSH' }}{% endfor %}",
	"rush_fee": "{% assign rush_fee = order.extra_fees | first %}{% if rush_fee and rush_fee.amount > 0 %}{{ rush_fee.amount | currency }}{% else %}{{ 0 | currency }}{% endif %}",

	"lines": [
		{%- for line in order.all_orderlines %}
		{
			"product_name": "{{ line.product.name | escape_json }}",
			"project_id": "{{ line.project.id }}",
			"product_code": "{{ line.product.code | escape_json }}",
			"qty": "{% unless line.is_cut_print %}{{ line.quantity }}{% else %}{{ line.cut_print_quantity }}{% endunless %}",
			"unitprice": "{% unless line.is_cut_print %}{{ line.unit_price | currency }}{% else %}{{ line.unit_price | divided_by: line.cut_print_quantity | currency }}{% endunless %}",
			"subtotal": "{{ line.price | currency }}",
			"options": "{%- for option in line.chosen_template_options %}
				{%- if option.template_option.type == "image_upload" or option.value == blank %}{%- else %}<b>{{ option.template_option.name | escape_json }}:</b> {{ option.value | escape_json }}<br>
				{%- endif %}
				{%- endfor %}
				{%- for option in line.chosen_variants %}
				{%- if option.variant.type == "image_upload" or option.value == blank %}{%- else %}<b>{{ option.variant.name | escape_json }}:</b> {{ option.value | escape_json }}<br>
				{%- endif %}
				{%- endfor %}",
			"files": "{%- for file in line.generated_files %}
				{%- if line.is_cut_print %}{%- else %}{{ file.filename }}<br>{%- endif %}
				{%- endfor %}"
		{%- unless forloop.last %}},{% endunless %}
		{%- endfor %}
	}
	]
}
{% endcapture %}
```

**Key implementation notes for the CraftMyPDF pattern:**

- The `template_ID` is a CraftMyPDF template identifier — each site gets its own branded template designed in the CraftMyPDF dashboard.
- `order.all_orderlines` is used inside the capture block (includes all lines for the complete job ticket), while `orderlines` is used in the outer file delivery loop.
- The `| strip | escape_json` on the captured body is critical — `strip` removes trailing whitespace from the capture, then `escape_json` escapes the entire JSON string so it can be embedded inside the outer JSON's `payload` field. This is a double-encoding pattern.
- Pickup vs Ship logic uses `order.address.is_public` — when `true`, the order is a collection/pickup (system address); when `false`, it's a delivery with a real shipping address.
- For Shopify sites, shipping service name comes from `order.custom.shopify_shipping_service`; for Full Pixfizz sites, from `order.shipping_method`.
- Payment status is inferred from `order.payment_reference` — blank means unpaid.
- Rush fee detection loops `order.extra_fees` and replaces "Rush Fee" text with "RUSH" for visual flagging on the printed job ticket.
- Cut print orderlines need special handling: quantity comes from `cut_print_quantity`, and unit price must be divided by `cut_print_quantity` to get the per-unit price.
- Preview image URLs can be constructed from `line.preview_url` with modified height parameters and a `share` query parameter using `line.project.share_code`.
- Label strings (e.g. `order_label`, `customer_details_label`) are site-specific and should be customized per deployment. For multilingual sites, translate these values.
- The `show_payment_status` field should be set to `"false"` for Shopify sites (payment is handled externally).

---

### Pattern 2: Delivering user-uploaded files (file_upload options/variants)

This pattern delivers files that end users uploaded via `file_upload` type template options or product variants. These are files the customer attached during the ordering process (e.g. a custom logo, a PDF of their own artwork, a photo for engraving).

**How it works:**

1. Loop through all `orderlines`
2. For each orderline, check both `chosen_template_options` and `chosen_variants`
3. If an option/variant has type `file_upload` and `uploaded_file.url` is not blank, add it to the array
4. The file is fetched from its Pixfizz-hosted URL and delivered to the FTP

**Template (standalone version — without a fixed first entry):**

```liquid
[
	{%- assign af_first = true -%}

	{%- for line in orderlines -%}

		{%- comment %} Template options — file_upload type {%- endcomment -%}
		{%- for opt in line.chosen_template_options -%}
			{%- if opt.template_option.type == "file_upload" and opt.uploaded_file.url != blank -%}
				{%- unless af_first -%},{%- endunless -%}
				{
					"source": "{{ opt.uploaded_file.url | escape_json }}",
					"destination": "/Artwork/{{ order.code | escape_json }}/Uploaded_Files/{{ forloop.parentloop.index }}-{{ opt.uploaded_file.filename | escape_json }}"
				}
				{%- assign af_first = false -%}
			{%- endif -%}
		{%- endfor -%}

		{%- comment %} Product variants — file_upload type {%- endcomment -%}
		{%- for v in line.chosen_variants -%}
			{%- if v.variant.type == "file_upload" and v.uploaded_file.url != blank -%}
				{%- unless af_first -%},{%- endunless -%}
				{
					"source": "{{ v.uploaded_file.url | escape_json }}",
					"destination": "/Artwork/{{ order.code | escape_json }}/Uploaded_Files/{{ forloop.parentloop.index }}-{{ v.uploaded_file.filename | escape_json }}"
				}
				{%- assign af_first = false -%}
			{%- endif -%}
		{%- endfor -%}

	{%- endfor -%}
]
```

**When combined with CraftMyPDF (or any fixed first entry):**

When a fixed entry always occupies position 1 in the array, the comma handling simplifies — hardcode the comma inside each conditional block rather than using a flag variable:

```liquid
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

---

### Key rules for `_additional_files.json`

- The entire template must output a **single `[...]` array**. Multiple arrays or objects outside the array is invalid JSON and will cause a silent parse error.
- When a fixed entry (e.g. a PDF job ticket) always occupies position 1, **hardcode the comma inside each conditional block** rather than using a flag variable. This avoids the leading-comma problem entirely.
- Do not use generic variable names like `needs_comma` — they can collide with variables set in other fulfillment templates rendered in the same context. Use a unique prefix (e.g. `af_first`) if a flag variable is needed.
- Use `uploaded_file.filename` (not the option code/name) in the destination path to **preserve the original file extension** (e.g. `.pdf`, `.psd`, `.ai`).
- Always use `order.code` as the default folder identifier — only use `order.custom.shopify_order_number` when explicitly required for a Shopify site.
- `order.all_orderlines` includes all orderlines (use inside payload capture blocks for complete data). `orderlines` is the standard loop variable in the fulfillment template context (use for file delivery).
- The double-encoding pattern (`capture` → `| strip` → `| escape_json`) is required when embedding a JSON payload inside another JSON structure. The `strip` removes trailing whitespace from the capture block; `escape_json` escapes the entire string for safe embedding.

---

### FTP Path Prefix: `originals/` vs `/originals/`

The path prefix in a fulfillment filename or directory template determines where files land on the FTP server **relative to the order folder**:

- `originals/` — places files in a subfolder **inside** the order folder (e.g. `/ORDERFOLDER/originals/filename.jpg`)
- `/originals/` — places files in the **root** of the FTP server under `originals/`, outside the order folder hierarchy

This distinction is silent — both are valid syntax and neither produces an error. Wrong choice results in files landing in an unexpected location.

**Both the main fulfillment template AND the `_additional_files.json` template must be updated** when changing path structure. Updating only one will produce inconsistent delivery — production files in one place, original files in another.

### Job Tickets Folder Naming Consistency

When FTP routing uses a `Job Tickets` folder (e.g. to deliver PDF job tickets alongside production files), the **folder name must be consistent** across all sites and fulfillment templates in the same FTP setup.

A mismatch (e.g. `Job Tickets` on one site, `job-tickets` on another) causes routing to fail silently — the FTP receives the file but the production system cannot find it.

Establish a naming convention at the start of each FTP integration and apply it identically to every template that writes to that folder.

---

## Changelog
- 2026-04-10: Initial content from platform documentation export.
- 2026-05-21: Restructured _additional_files.json section — documented three source formats (simple URL, HTTP request object, literal content), added full CraftMyPDF PDF job ticket worked example with capture block and implementation notes, separated file_upload delivery as distinct Pattern 2 with standalone and combined versions, added double-encoding and orderlines scope rules. Source: claude-chat.
- 2026-06-01: Added chosen_variants accessor note to the QR Code worked example. Source: claude-chat.
- 2026-06-30: Documented fulfillment code resolution/precedence — template-level codes (added June 2026) have highest priority and override location-based codes. Source: notion-dashboard (2026-06-22).
- 2026-07-04: Noted that reverting the output filename template to default flattens product-category subfolders (files land flat in the per-order folder). Source: slack-message (#development).
