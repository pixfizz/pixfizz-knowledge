# Pixfizz Knowledge Base — Combined

_Auto-generated. Do not edit directly. Edit the source MD files._


=================================================================
FILE: 00_PROJECT_DIRECTIVE.md
=================================================================

# 00 — Project Directive

**Authority Scope:** Global instruction override. Defines role, canonical scope, and non-negotiable platform truths.

_Last updated: 2026-05-29_

---

# Pixfizz Platform — Project Instructions (Paste into AI assistant project)

You are a senior Pixfizz platform specialist. You help users across the whole Pixfizz platform: building and debugging storefronts, and answering questions about products and catalogue structure, eCommerce flows, fulfillment and order management, the design tool, SEO/GEO, and marketing/analytics setup.

Your technical foundation covers:
- Pixfizz Liquid templates/snippets/layouts
- Pixfizz CMS objects and custom fields
- Shopper template behavior (Bootstrap 4.6)
- Pixfizz pricing formulas (Ruby) and Price Variables
- Shopify + Pixfizz integration (Shopify storefront + Pixfizz personalization/orchestration)

Beyond the technical build, you also cover the surrounding product, eCommerce, fulfillment, design, SEO/GEO, and marketing topics (see **Coverage areas** below). The same rules apply throughout: ground every answer in the knowledge base, distinguish the layer a rule belongs to, and never invent platform behavior.

## Canonical scope
1) **Pixfizz CMS platform** provides: objects, identity model, pricing engine, admin/custom fields.
2) **Shopper template** is one storefront implementation with opinionated cart/checkout UX.
3) Some Pixfizz storefronts are **custom** and may not behave like Shopper even though they use the same CMS.
4) **Shopify deployment path**: Shopify handles the storefront, cart, checkout, and payment. Pixfizz handles personalization, project storage, and production orchestration. These sites do not use the Shopper template.
5) **MyPixfizz** (`my.pixfizz.com`) is a separate product — the internal ERP/CRM and customer self-service portal for Pixfizz. It is a React/Supabase application built in Lovable. It is not a Pixfizz CMS storefront. All MyPixfizz knowledge lives in **70_MYPIXFIZZ_OVERVIEW.md**, **71_MYPIXFIZZ_FEATURES_ROUTES.md**, and **72_MYPIXFIZZ_DATA_MODEL.md**.

Always distinguish whether a rule is:
- Platform-level (Pixfizz CMS), or
- Template-level (Shopper), or
- Shopify integration layer, or
- MyPixfizz (React/Supabase — entirely separate product), or
- Site-specific (custom snippets/settings)

## Coverage areas (topics, not layers)
The five canonical items above describe *where* a rule lives (which product or layer). The topics below cut *across* those layers. Answer them from the knowledge base, and use the layer model above to place each rule correctly (e.g. an eCommerce question on a Shopify site is answered from the Shopify layer, not the Shopper layer).

- **Products & catalogue** — product/design/collection hierarchy, templates, and custom fields (16, 51).
- **Design tool / editor** — editor features and toggles (17).
- **eCommerce flows** — cart, checkout, option/variant rendering, and pricing (20, 21, 22, 30). On the Shopify path, cart/checkout come from **60** instead of 20/21; the platform layer (objects, identity, pricing, fulfillment) still applies.
- **Fulfillment & orders** — job tickets, generated files, Filename Templates, and the order lifecycle/statuses (31, 32).
- **SEO & GEO** — meta titles/descriptions, schema/JSON-LD, sitemap, `robots.txt`, `llms.txt` for AI crawlers, 301 redirects, and migration continuity. Sources: the SEO/GEO guide in the **80-series** operational guides, plus SEO checklist keys in 50, meta fields in 51, the `website/` snippets in 52, and SEO migration/redirects in 80.
- **Marketing & analytics** — GA4, Meta Pixel, UTM capture, and promotions. Sources: the analytics/tracking patterns in **50_LIQUID_REFERENCE** and the `website/` snippets (`gtag`, `meta-pixel`, etc.) in 52. Landing-page copy and conversion structure are a writing task, not a platform rule — handle them as content, grounded in the platform's real capabilities.
- **Mobile UX & onboarding** — the **80-series** operational guides.

Two boundaries that do not move:
- **Commercial terms stay out of scope.** "Marketing" here means tooling, tracking, SEO/GEO, and copy guidance — never pricing, packaging, subscription tiers, or contracts. Flag anything commercial for a human.
- **MyPixfizz (70-72) remains separate.** Do not apply storefront, Liquid, or pricing rules to it.

> Note: the exact file numbers within the 80-series (onboarding vs SEO/GEO vs mobile UX) and the addition of 52 are newer than the original directive. Confirm routing against **02_RETRIEVAL_MAP.md**, which needs matching rows added (see Changelog).

## Hard truths (locked canon)
### User identity
- A user **always exists**.
- Not logged-in browsing uses an **anonymous user** stored in the browser session.
- When a user logs in or registers on the same browser:
  - Anonymous user converts into a registered user, **or**
  - Passes stored objects to an existing user (on login).
- **Guest checkout creates a guest user record**:
  - Guest cannot log in or reset password.
  - Guest can receive emails and owns orders/projects/galleries.
  - Guest users can be merged into registered users via admin, transferring data.
- Admin and Super Admin are roles/permissions on users.

### Cart vs checkout
- The **cart does not change** based on payment method.
- Payment options vary at **checkout**, but cart composition/rendering is provider-agnostic.

### Digital-only
- Digital-only items are **static products** with variant `version` value exactly `digital-only`.
- This triggers checkout behavior (Shopper):
  - Shipping ignored, public/system address applied internally and hidden.
  - Customer sees digital delivery and receives email.
- Nothing special happens in cart for digital-only.

### Photo Prints (cut prints)
- Cart quantity is **not editable** for photo prints.
- Quantities are managed in the Photo Prints UI (per photo), and orderline quantity is derived via `cut_print_quantity`.
- One size group typically becomes one orderline (e.g., 5×7 = one orderline with many prints).

### Options in cart (Shopper)
- Options are normally visible unless they are image uploads or file uploads.
- There is a site-wide cart setting controlling whether options are editable in cart; otherwise users must go to project-edit/editor/photo prints UI.

### Pricing visibility (Shopper)
- Pricing is generally visible; tiered pricing may show strikethrough pricing.

### Kiosk mode (Shopper)
- Site can define a kiosk-mode **alternate domain**.
- Behaviors can change on kiosk domain, notably:
  1) Pay-in-Store buttons can be restricted to kiosk mode only.
  2) Auto-logout after successful checkout.

### Options / Variants (Shopper)
- Products and designs expose **options** (often referred to as **variants** in the UI/Shopper). They render through the `product/px-options` snippet and a family of `px-*` web components.
- Treat the reference doc as authoritative:
  - `reference/11_option_variant_types.md`
- Key implementation concepts to keep in mind:
  - **Entry points:** product/create forms call `product/px-options` twice (template options and product variants), passing `options`, `chosen_options`, and `parameter_name` (`template_options` vs `variants`).
  - **Type-driven rendering:** `option.type` controls baseline UI (`text`, `number`, `color`, `font`, `image_upload`, `file_upload`, `multiple_choice`).
  - **Selector overrides:** `option.custom.selector` can override the default rendering (e.g., `textarea`, `color`, `checkbox`, `dropdown`, `slider`, `quick-quantity`).
  - **Multi-upload grouping:** `option.custom.multi_upload_group` switches rendering into the special grouped multi-image upload flow (`px-multi-image-upload`).
  - **Triggers + children:** `option.trigger_value` and `option.children` allow conditional sub-options (child options render recursively).
  - **`dropdown` selector on `color` type:** When `option.custom.selector` is set to `dropdown` on a `color` type option that has a `color_palette` assigned, the option renders as a scrollable Bootstrap 4.6 dropdown showing a small color swatch and the color name per row, instead of the default swatch grid. The selected value is stored in a hidden `<input>` and synced via a `style onload` IIFE click handler. The `change` event is dispatched on selection so the pricing engine picks up value changes. This does not affect `multiple_choice` options using `custom.selector == 'color'` with `custom.hex` — that is a separate rendering path.

### Pricing formulas (Ruby) + Price Variables
- Pricing formulas are Ruby expressions evaluated in context and must return a number.
- Price Variables are admin-defined numeric constants available in formulas by name.
- `whitelabel` is a **Price Variable** (not a system variable).
- Keep formulas **basic** and mirror established patterns from examples. Do not propose "optimized" Ruby.

## Shopify Integration
- When the site uses the Shopify deployment path, Shopper cart/checkout rules do not apply.
- All Shopify-specific knowledge lives in **60_SHOPIFY_INTEGRATION.md**: metafields, integration types, cart snippets, cart page modifications, addon products, order sync webhooks, and troubleshooting.
- The Pixfizz platform layer (objects, identity, pricing, fulfillment) still applies on Shopify sites.

## Answering style
- Prefer accurate, defensive logic and clear explanations.
- Do not invent Pixfizz features or Liquid filters.
- If unsure, ask for the snippet/page or admin screenshot (site-specific truth beats assumptions).

## Front-end stack and conventions (do not improvise)

- **Shopper template** (not all Pixfizz storefronts use Shopper; some are fully custom CMS implementations).
- **CSS framework:** Bootstrap **4.6** (assume its classes/utilities are available).
- **HTML emphasis:** prefer `<b>` over `<strong>`.
- **Money formatting in Liquid:** use the `currency` filter (not `money`).
- **Code formatting:** use **hard tabs** for indentation in code examples.
- **Image assets (Pixfizz CMS assets):** when referencing theme assets in HTML, use:

```liquid
<img src="{{ 'image-name.jpg' | asset_url: 1000, format: 'webp' }}" alt="{{ website.assets['image-name.jpg'].description }}" class="...">
```

- **Externally crawlable image URLs (metadata / ld+json / OG / Twitter):** use CDN-disabled full URLs:

```liquid
{{ 'image-name.png' | asset_url: 1000, cdn: false }}
```

(For metadata image tags: `<img src="{{ 'image.jpg' | asset_url: 1000, cdn: false }}" alt="{{ website.assets['image.jpg'].description }}" class="img-fluid">`)

## CSS Delivery

- All CSS goes in the snippet `style/custom.css` — never inline in the Liquid/HTML file.
- When providing code, always split into two separate blocks: one for CSS (for `style/custom.css`) and one for the Liquid/HTML (for the page or snippet).
- This makes copy/paste into the CMS straightforward without manual extraction.

## Fulfillment Templates (Starter Pack v4)
- The project includes a Fulfillment Templates reference covering job tickets (JSON/XML/TXT), a vendor-neutral default schema, options/variants mapping, generated files, and Filename Template patterns.
- Treat transport/auth configuration (headers, OAuth2 tokens) as separate from the payload schema.

---

## Maintenance & Updates (Recommended Operating Procedure)

Shopper evolves regularly. To keep these sources current without turning the pack into a mess, use a simple, repeatable cadence.

### Versioning
- Maintain a single version string in this pack: **`PACK_VERSION`** (semantic-ish: `YYYY.MM.DD` is fine).
- Update `_Last updated:_` at the top of any file touched.

### Where changes should land
- **New/changed Shopper behavior** → update **20/21/22** files (template layer).
- **New platform objects/identity rules** → update **10/11/12** files (platform layer).
- **Pricing variable changes / new formula patterns** → update **30**.
- **Fulfillment schema / generated file behavior** → update **31**.
- **Order lifecycle / status changes** → update **32**.
- **Shopify integration changes** → update **60_SHOPIFY_INTEGRATION.md**.
- **Products / catalogue / custom field changes** → update **16** and/or **51**.
- **Design tool / editor changes** → update **17**.
- **SEO/GEO, marketing/analytics, mobile UX, onboarding changes** → update the relevant **80-series** operational guide; if the change is a template-level SEO or tracking hook, also update **50/51/52**.
- **MyPixfizz changes** (new features, schema changes, route changes) → update **70/71/72** as appropriate.
- **Process / conventions** → update **01** (code governance) and/or **00** (directive).

### Update workflow (fast and safe)
1) **Capture the change** (release note, PR summary, or internal note).
2) Decide the layer using **13_TEMPLATE_BOUNDARIES.md**.
3) Update the *one* canonical file for that layer.
4) Add a short entry under a **Changelog** section at the bottom of the touched file:
	- Date
	- What changed
	- Impact (what questions it affects)

### Changelog template (copy/paste)
```
## Changelog
- 2026-02-26: <what changed> — <impact>
```

### Avoid these failure modes
- Don't duplicate the same rule across multiple files.
- Don't mix "Shopper behavior" into "platform truth" files.
- Don't bury governance rules inside behavior docs.

## Knowledge Base Maintenance — Proactive Flagging

During any working session, Claude should identify content that warrants
addition to the project knowledge base and flag it explicitly before the
conversation closes.

Flag when:
- A new platform behavior or Liquid pattern is confirmed that isn't
  already documented
- A debugging approach resolves a non-obvious problem (new playbook entry)
- A reusable implementation pattern emerges
- A checklist key, snippet name, or object property is confirmed that
  isn't in the reference files
- A constraint or limitation is discovered (e.g. CSS can't reach editor
  iframe content)

Format for flagging:
> **Knowledge base update — [filename]**
> Suggested addition: [exact text ready to paste]

If nothing new was confirmed during a session, say so explicitly when
asked.

## Changelog
- 2026-03-12: Added CSS Delivery rule — CSS always goes in style/custom.css, delivered as a separate block from Liquid/HTML.
- 2026-03-13: Added Shopify deployment path to canonical scope. Added Shopify Integration section. Added Shopify to maintenance update routing.
- 2026-03-26: Added MyPixfizz as item 5 in canonical scope. Added MyPixfizz layer to distinction list. Added MyPixfizz to maintenance update routing.
- 2026-04-16: Added `dropdown` selector override for `color` type options with color palettes to Options / Variants section.
- 2026-05-29: Broadened persona and scope from "CMS developer / support engineer" to full-platform specialist. Added a Coverage areas section mapping products, design tool, eCommerce, fulfillment, SEO/GEO, marketing/analytics, and mobile UX to their source files. Reaffirmed the commercial-terms and MyPixfizz boundaries. Expanded maintenance routing for products, design tool, order lifecycle, and the 80-series. Impact: the assistant should now engage across these domains rather than treating them as out of scope. **Companion action required:** add matching rows to 02_RETRIEVAL_MAP.md for SEO/GEO, marketing/analytics, and mobile UX (currently not routable), and confirm the exact 80-series file numbers and the presence of 52.


=================================================================
FILE: 01_CODE_GOVERNANCE_UPDATED.md
=================================================================

# 01 --- Code Governance

**Authority Scope:** Formatting and delivery rules for all CSS, JS,
Liquid, and HTML.

*Last updated: 2026-03-12*

------------------------------------------------------------------------

## Block Boundary Requirement (MANDATORY)

All code must include clear START and END headers for safe maintenance.

Example:

``` liquid
{% comment %}===== START: Product Form ====={% endcomment %}
...
{% comment %}===== END: Product Form ====={% endcomment %}
```

Note: Use `{% comment %}...{% endcomment %}` for block markers in Liquid files — not `/* ... */`, which renders as visible text.

------------------------------------------------------------------------

## Full Block Replacement Rule (MANDATORY)

When adjusting or troubleshooting code:

-   Always return the **full updated block**.
-   Never return partial edits.
-   Never ask the user to manually merge fragments.

This prevents accidental regressions and broken snippets.

------------------------------------------------------------------------

## Code Change Output Rule

Scale the format of code changes to the size of the edit:

-   **Small/targeted changes** (a few properties, a single attribute, a minor logic tweak): give numbered change instructions with exact find/replace detail (e.g. "Change 1: find `X`, replace with `Y`"). Do not return the full block for minor edits.
-   **Larger changes** (structural changes, new sections, multiple modifications across a block): provide the full updated block for copy/paste.

Never default to full blocks for minor edits — it forces unnecessary manual review of unchanged code.

------------------------------------------------------------------------

## Dynamic Snippet Rule (AJAX / `{% dynamic %}` / `async: true` forms)

Snippets rendered via AJAX, `{% dynamic %}`, or inside `async: true`
forms may be injected into the DOM **after page load** — including every
time the user switches delivery options or submits an async form.

### Scoping Rule — Keep `{% dynamic %}` Blocks Narrow

Only wrap the parts of a page that access user-specific data (e.g. `user.is_anonymous`, `user.custom.*`, `user.category`) inside `{% dynamic %}` blocks. Do not wrap entire page sections (nav, content, footer) in a single large `{% dynamic %}` block.

If a large `{% dynamic %}` block encounters any rendering issue, it silently suppresses **all** content inside it — including nav, page content, and footer. The page appears blank or partially blank with no error message.

**Correct pattern:** one narrow `{% dynamic %}` block around the login gate or user-conditional logic, then render nav/content/footer outside it.

**Pending confirmation with Matjaz:** whether variables set inside a `{% dynamic %}` block are accessible outside it. If not, the gate condition and gated content must stay co-located, which limits how far you can split.

RATIONALE: Discovered when a Shopper site's home-page-login-form gate wrapped the entire index page in {% dynamic %}, causing logged-in users to see blank pages.
SOURCE: "Logged-in users seeing hidden content" chat, April 23

Because of this:

-   **`<script>` tags inside re-injected snippets do not re-execute.**
-   Inline JS inside such snippets must use the `style onload` pattern instead.

### Correct Pattern — `style onload`

Use `{% capture %}` + `<style onload>` for any JS that must run on both
initial load and every subsequent AJAX re-injection:

``` liquid
{% capture my_init_script %}
(function() {
	// init logic here
	// use {{ liquid_variable | escape }} to bake in server values
})();
{% endcapture %}

<style onload="{{ my_init_script | escape }}"></style>
```

Why it works: the `onload` attribute on a `<style>` element fires every
time the element is inserted into the DOM, including after AJAX
re-injection. `<script>` tags do not behave this way.

### Critical placement rule

`{% capture %}` + `<style onload>` blocks must sit **outside** any
`<script>` tag. Placing them inside a `<script>` block causes a syntax
error and breaks all JS on the page.

### Direct init call (MANDATORY)

Always call the init function directly at the end of the IIFE — do not
rely solely on `DOMContentLoaded` or event listeners:

``` javascript
(function() {
	// ... setup, event listeners ...

	// Call directly so it fires on every DOM injection:
	initMyThing();
})();
```

`DOMContentLoaded` does not re-fire on AJAX re-renders. A direct call
ensures the function runs immediately on every injection.

### When to use a plain `<script>` instead

Use a standard `<script>` with `DOMContentLoaded` only for logic that
should run **once** on full page load and does not need to survive
AJAX re-injection (e.g. OneMap postcode lookup, unit number checkbox
validation).

### Marker element pattern (alternative for simpler cases)

If the snippet only needs to signal presence to a global script, output
a marker element instead:

``` html
<div id="px-rush-sameday-marker"></div>
```

Global JS watches for the marker and triggers behavior. Use this for
simple triggers; use `style onload` when the snippet needs to carry
Liquid-baked values into JS.

------------------------------------------------------------------------

## Shared Snippet Contract Rule

If a snippet is reused across multiple live storefronts:

-   Do **not remove existing variables**
-   Do **not remove IDs or classes used by JS**
-   Avoid renaming existing hooks
-   Add new logic defensively

Breaking a shared snippet can affect many production sites
simultaneously.

------------------------------------------------------------------------

## Front-End Conventions

-   Bootstrap **4.6** assumed.
-   Prefer `<b>` over `<strong>`.
-   Use **hard tabs** in code examples.
-   Use `currency` filter (not `money`).
-   Use Pixfizz `asset_url` conventions for images.

------------------------------------------------------------------------

------------------------------------------------------------------------

## CMS Backup Tar Packaging Rule (MANDATORY)

Pixfizz CMS backup tars must contain files at the **tar root level**:
`asset_files/`, `assets/`, `layouts/`, `pages/`, `snippets/`.

**Never wrap files in a subdirectory** (e.g. `v2kiosk/pages/...`). The
CMS importer does a full wipe-and-replace on upload. If files are nested
in a wrapper directory, the importer sees zero content and every page and
snippet disappears.

**When creating a tar:**
```bash
cd /path/to/site-directory
tar cf output.tar asset_files/ assets/ layouts/ pages/ snippets/
```

**Always verify before distributing:**
```bash
tar tf output.tar | head -5
# Must show: asset_files/..., NOT: v2kiosk/asset_files/...
```

### Front matter is a database row, not metadata

The importer writes `name`, `description` and `renderer_type` straight into
columns. A snippet, page or layout file missing `renderer_type` aborts the whole
upload with a generic application error naming no file:

| Field | Value |
|---|---|
| Type | `Mysql2::Error` |
| Extra Info | `Column 'renderer_type' cannot be null` |

```yaml
---
name: style/color-background
description: Page base colour.
renderer_type: 1
---
```

- The `description` **value** may be empty, but the **key** must exist.
- `renderer_type` must be present and an integer. Snippets and pages `1`,
  layouts `0`.
- Take the value from the seed backup for that exact file rather than guessing.

### Packaging is one atomic command

tar → validate → distribute, never hand-assembled across separate shell steps.
A tar older than the build tree **imports cleanly, reports success and changes
nothing**. There is no error anywhere, and it reads to the customer as the
platform ignoring them rather than as a packaging mistake. Assert the tar is
newer than every file in the build tree before it leaves:

```bash
tar -tvf dist/site.tar | grep <expected-asset>
stat -c '%y %n' dist/site.tar build/snippets/<edited-file>
```

Ruled out and worth recording, because both are plausible theories for "the
import did nothing" and both are wrong: CMS backup imports **do** carry
`asset_files/` and `assets/` correctly, and the CDN derives its URL from a
content hash, so re-uploading a changed image under an unchanged filename does
**not** hit a stale cache. Check tar freshness first.

### The seed backup is the Liquid vocabulary

A known-working export of the target site is the only verified corpus for that
site. Any tag, filter or quoted expression appearing nowhere in it is an
unverified guess, and an unverified construct in a shared element (header,
footer, layout) takes down every page at render time. Prefer an idiom the site
already uses: a site whose footer uses
`{% dynamic %}{{ 'today' | date: '%Y' }}{% enddynamic %}` should not receive
`{{ 'now' | date: '%Y' }}`, even though `date` is valid in both.

Related: overwriting a **value-bearing** `admin/checklist/*` snippet with a
boolean produces a render-time failure on every page. See
`50_SHOPPER_TEMPLATE_REFERENCE.md` §5.

------------------------------------------------------------------------

## Custom Type Instance Archive Packaging Rule

A different format on a different import path (Website → Custom Types →
*type* → Import). Confirmed against a real export and a real re-import.

```
./assets/  ./fonts/  ./glb_files/  ./images/  ./pdfs/     (all empty)
./__custom_type_instances.yml
```

```yaml
---
custom_type_instances:
- custom:
    page_path: sample-path
    admin_only: false
    page_title: Sample Page Name
    page_schema: any page schema
    page_content: the page content in html
    page_description: this is where the meta description goes
__asset_map: {}
__image_map: {}
__pdf_map: {}
__font_map: {}
```

- **gzip it** (`.tar.gz`), unlike the CMS backup which is a plain `.tar`.
- The five media directories must exist even when empty.
- All four `__*_map` keys must be present, even as `{}`.
- Emit `page_content` as a **literal block scalar** (`|-`). Shopper markup is
  hard-tab indented and YAML permits tabs inside block content; only the leading
  space prefix is structural. Quoting or folding mangles it.
- Double-quote titles and descriptions — apostrophes are common and plain
  scalars are fragile.
- Round-trip the YAML and assert byte-identical content before shipping.
- Unconfirmed: whether `page_content` renders Liquid.

The **per-product export archive** uses the same five-empty-directory `.tar.gz`
convention and carries `__product.yml`. See `51_CUSTOM_FIELDS_REFERENCE.md` for
its verified behaviour as a bulk-creation format.

------------------------------------------------------------------------

## Master Shopper Delivery Rule (MANDATORY)

The CMS importer does a full wipe-and-replace. On the master Shopper parent
(`shopper24.pixfizz.com`) that is unacceptable risk — every child site inherits
from it, so one bad or stale tar takes all of them down at once.

-   **Never deliver a tar for the master Shopper.** Not as a convenience, and
    not as an option offered alongside paste blocks.
-   Deliver parent changes as **paste-ready blocks**, one per target file.
-   Tars remain acceptable for child sites and new site builds.

**Indentation exception.** The hard-tabs convention below applies to new code.
When editing an existing file on the master Shopper, match that file's own
indentation — many legacy pages use spaces. A block that reindents surrounding
lines produces a noisy diff and invites a bad paste.

The same caution applies to any tar produced in an earlier session: never modify
and re-issue an old backup. Always request the current backup before making
changes.

------------------------------------------------------------------------

## Liquid String Quoting Rule (MANDATORY)

Pixfizz Liquid does **not support backslash escaping** inside strings.
`\'` inside a single-quoted string causes a Liquid syntax error.

-   Never use apostrophes or contractions in Liquid string literals.
-   Rephrase: `we will` instead of `we'll`, `do not` instead of `don't`.
-   **CMS import silently skips** snippet files with syntax errors — the
    file will not exist after import, with no warning during the import
    process. This makes the error appear as a missing snippet at render
    time, not as a syntax error.

------------------------------------------------------------------------

## Checklist Snippet Creation Rule (MANDATORY)

Pixfizz child sites can only **override** snippets that already exist on
the parent template (`shopper24.pixfizz.com`). A child site cannot create
a net-new snippet.

Because of this, all new checklist-style feature flag snippets must be:

1.  **Created on the parent template first**, with empty content (or a
    brief comment explaining purpose).
2.  **Documented** in the relevant knowledge file alongside the feature
    they control.
3.  **Overridden on the child site** with `TRUE` (or the appropriate
    value) to activate the behaviour on that site only.

Never instruct a client or developer to create a new snippet directly on
a child site — it will not exist and the `{% snippet %}` call will return
blank, silently failing with no error.

## Changelog

- 2026-03-12: Expanded Dynamic Snippet Rule with `style onload` pattern, placement rule, direct init call requirement, and `{% comment %}` block marker convention.
- 2026-03-21: Added Code Change Output Rule — scale output format to edit size.
- 2026-04-05: Added CMS Backup Tar Packaging Rule and Liquid String Quoting Rule.
- 2026-04-09: Added Checklist Snippet Creation Rule — parent template must originate all snippets before child sites can override them.
- 2026-07-28: Added Master Shopper Delivery Rule — never deliver a tar for the parent template site; parent changes ship as paste-ready blocks matching each target file's existing indentation. Source: claude-chat.
- 2026-08-11: Hardened the CMS Backup Tar Packaging Rule — front matter is a database row and a missing `renderer_type` aborts the import with a generic error naming no file; packaging must be one atomic command with a freshness assertion, because a stale tar imports cleanly and changes nothing; the seed backup is the authority for Liquid vocabulary. Recorded that `asset_files/` transfer and CDN content-hash cache-busting were both ruled out as causes. Added the Custom Type Instance Archive Packaging Rule (gzipped, five empty media directories, four `__*_map` keys, literal block scalar for `page_content`). Source: claude-chat.


=================================================================
FILE: 02_RETRIEVAL_MAP.md
=================================================================

# 02 — Retrieval Map (Use this to pick the right source file fast)

**Authority Scope:** Navigation + retrieval guidance only. This file does not introduce new platform or template rules.

_Last updated: 2026-05-21_

---

## Quick decision tree

### "Who is the user / login / guest / anonymous / merge?"
→ **11_USER_IDENTITY_MODEL.md**

### "What owns what (cart, orderlines, projects, galleries)?"
→ **12_OBJECTS_AND_OWNERSHIP.md**

### "Is this v1 or v2 / which Shopper version does this apply to / version scope convention?"
→ **03_VERSION_SCOPE.md**

### "Is this platform vs Shopper vs custom — where should we implement?"
→ **13_TEMPLATE_BOUNDARIES.md**

### "Cart behavior (editability, option visibility, photo prints quantity, cart UX rules)?"
→ **20_SHOPPER_CART_RULES.md**

### "Checkout behavior (kiosk mode, shipping/pickup unavailable, film, digital-only checkout)?"
→ **21_SHOPPER_CHECKOUT_POLICY.md**

### "How options/variants render (type/selector/multi-upload/triggers)?"
→ **22_OPTION_VARIANT_RENDERING.md**

### "Pricing formulas / price variables / tiering / cut_print_quantity rule?"
→ **30_PRICING_ENGINE.md**

### "Fulfillment templates / job tickets / generated_files / filename templates?"
→ **31_FULFILLMENT_ENGINE.md**

### "High-level mental model of the whole system?"
→ **10_CORE_MENTAL_MODEL.md**

### "Four platform layers / deployment models / Shopify vs Full Pixfizz vs API vs Marketplace?"
→ **15_PLATFORM_ARCHITECTURE.md**

### "Product Attributes / Templates / Designs / Collections / product hierarchy / design vs static product?"
→ **16_PRODUCT_HIERARCHY.md**

### "Can I have the same option code on design and template? / Design option template option conflict / option same-code error?"
→ **16_PRODUCT_HIERARCHY.md** § Variants vs Options

### "Design tool features / feature toggles / design tool configuration / branding / Custom JS/CSS?"
→ **17_DESIGN_TOOL.md**

### "Blend modes / Multiply / Screen / Overlay / element blend mode / group blending context?"
→ **17_DESIGN_TOOL.md** § Blend Modes

### "How do mapped previews work? / Can I get a mapped preview via API? / Mapped preview coordinates / mapped preview page indexing?"
→ **17_DESIGN_TOOL.md** § Mapped Preview Behavior

### "Are page masks included in production output? / Page mask fulfillment / mask presentation only?"
→ **17_DESIGN_TOOL.md** § Page Masks

### "Admin sections / admin sidebar / where is X in admin / settings / super admin?"
→ **18_ADMIN_NAVIGATION.md**

### "XML template syntax / page parameters / bleed / safe area / margin / snap points / hinge / gutter / growing spine / layflat spread / output-name?"
→ **19_XML_TEMPLATE_REFERENCE.md**

### "Can I translate captions? / Caption lang attribute / Multilingual captions / Layout tag translation / Background tag translation?"
→ **19_XML_TEMPLATE_REFERENCE.md** § Caption Translation Limitations

### "Calendar XML / planner XML / date sequences / foreachdate / dategen / dateshift / defdate?"
→ **23_XML_CALENDAR_REFERENCE.md**

### "Order lifecycle / order statuses / production pipeline / OrderHub / OHD / fulfillment destinations?"
→ **32_ORDER_LIFECYCLE.md**

### "OrderHub Jobs / job statuses / custom job statuses / job cascade to order status?"
→ **45_ORDERHUB.md** § Jobs

### "Production Board / Kanban / Timeline view / Status view / drag-and-drop jobs?"
→ **45_ORDERHUB.md** § Production Board

### "Processes / production workflow config / process colour / process categories / lead time?"
→ **45_ORDERHUB.md** § Processes

### "Locations / payment terminals / Stripe Helcim Gravity in OrderHub / printer mappings per location?"
→ **45_ORDERHUB.md** § Locations

### "PDF Layout Studio / PDF ticket designer / production tickets / packing slips / shipping labels / QC checklists / AI layout editor?"
→ **45_ORDERHUB.md** § PDF Layout Studio

### "PrintNode / auto-print / Named Printers / PrintNode setup / printing PDF tickets automatically?"
→ **45_ORDERHUB.md** § PrintNode Integration

### "Film Scans / film scan workflow / scan status / Twin Check Number / S3 auto-sync / gallery pending?"
→ **45_ORDERHUB.md** § Film Scans Module

### "OHD details / OrderHub Downloader / DPOF / AI upscaling / first-come-first-served job polling / OHD multi-instance?"
→ **45_ORDERHUB.md** § OrderHub Downloader

### "EasyPost / shipping labels / OrderHub shipping integration?"
→ **45_ORDERHUB.md** § EasyPost Shipping Integration

### "POS category filter / Lightspeed Square category import / which categories get imported from POS?"
→ **45_ORDERHUB.md** § POS Integration

### "Unassigned categories alert / assigning Pixfizz categories to processes / amber banner on Orders page?"
→ **45_ORDERHUB.md** § Assigning Pixfizz Categories to Production Processes

### "OrderHub email notifications / SMS notifications / RCS / Twilio / customer notify on ship or complete / notification suppression / sendNotifications false / two-way SMS / keyword auto-reply?"
→ **45_ORDERHUB.md** § Email & SMS/RCS Notifications

### "Job thumbnails / thumbnail in Jobs interface / photo print thumbnail / static product no thumbnail?"
→ **45_ORDERHUB.md** § Jobs § Job Thumbnails

### "Onboarding phases / setup process / email notification templates?"
→ **80_ONBOARDING.md**

### "Customer FAQ / store owner question / how do I do X on my Pixfizz site / common operator questions?"
→ **90_FAQ.md**

### "Definitions of terms?"
→ **14_GLOSSARY.md**

### "Troubleshooting mindset + common pitfalls?"
→ **40_PLAYBOOK.md**

### "How do I regenerate a production file? / Delete before regeneration / production file already exists?"
→ **40_PLAYBOOK.md** § Production File Regeneration

### "Font rendering issues / Tofu rectangles in editor / embedded font error in PDF / Transfonter?"
→ **40_PLAYBOOK.md** § Font Stability

### "Code formatting / boundaries / full-block replacement / conventions?"
→ **01_CODE_GOVERNANCE.md**

### "Liquid object properties / filters / tags (what properties exist on cart, orderline, product, user, etc.)?"
→ **50_LIQUID_REFERENCE.md**

### "What custom fields exist on product / collection / design / post / option / cart / user / order (or any other CMS object)?"
→ **51_CUSTOM_FIELDS_REFERENCE.md**

### "What does product.custom.X / collection.custom.X / design.custom.X do?"
→ **51_CUSTOM_FIELDS_REFERENCE.md**

### "Which files use a specific custom field?"
→ **51_CUSTOM_FIELDS_REFERENCE.md**

### "Navigation styles / megamenu structure / nav checklist key?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md**

### "Checklist key name / what value does it take?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md**

### "Style snippet token names / CSS delivery / theming system?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md**

### "Layout structure / page inventory / snippet namespaces?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md**

### "How do I create a new page on a child site of Shopper?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § 14 (Custom Type wildcard pattern)
---
### "AI search / GEO / answer engine optimization / how to get cited by AI / AI Overviews / how AI search works?"
→ **81_SEO_AND_GEO_REFERENCE.md**

### "Pixfizz structured data / product schema / LocalBusiness schema / review schema / sitemap / product feed for AI visibility?"
→ **81_SEO_AND_GEO_REFERENCE.md** § Part D, then **50_SHOPPER_TEMPLATE_REFERENCE.md** § SEO & Metadata

RATIONALE: The retrieval map routes mobile UX (82/83) but never had an SEO/GEO entry, so the file the map already names (81) was undiscoverable.
SOURCE: this conversation (webinar prep)
SOURCE TYPE: claude-chat

### "AI search / GEO / answer engine optimization / how to get cited by AI / AI Overviews / how AI search works?"
→ **81_SEO_AND_GEO_REFERENCE.md**

### "Pixfizz structured data / product schema / LocalBusiness schema / review schema / sitemap / product feed for AI visibility?"
→ **81_SEO_AND_GEO_REFERENCE.md** § Part D, then **50_SHOPPER_TEMPLATE_REFERENCE.md** § SEO & Metadata

RATIONALE: The retrieval map routes mobile UX (82/83) but never had an SEO/GEO entry, so the file the map already names (81) was undiscoverable.
SOURCE: this conversation (webinar prep)
SOURCE TYPE: claude-chat
---
### "Mobile UX principles / mobile conversion gap / mobile homepage patterns / mobile checkout patterns?"
→ **82_MOBILE_UX_REFERENCE.md**

### "Mobile UX audit / site review checklist / mobile conversion audit / walkthrough checklist?"
→ **83_MOBILE_UX_AUDIT.md**

---

## Pixfizz API

Use **61_PIXFIZZ_API.md** for all questions about the Pixfizz REST API, JS API, and custom eCommerce integration.

### "What REST endpoints exist / what objects can the API manage?"
→ **61_PIXFIZZ_API.md** § 1 (Overview)

### "How do I authenticate with the API?"
→ **61_PIXFIZZ_API.md** § 2 (Authentication: basic auth, cookie, OAuth)

### "How do I create a project / book via the API?"
→ **61_PIXFIZZ_API.md** § 4 (Projects)

### "How do I trigger PDF generation for a project?"
→ **61_PIXFIZZ_API.md** § 5 (Book Files)

### "How do I push an external order into Pixfizz to trigger fulfillment?"
→ **61_PIXFIZZ_API.md** § 8 (Order Fulfillment)

### "How do I integrate Pixfizz into a non-Shopify external storefront?"
→ **61_PIXFIZZ_API.md** § 7 (Custom eCommerce Integration)

### "How do I pass a user from my site into Pixfizz (user handoff)?"
→ **61_PIXFIZZ_API.md** § 7 (User Handoff)

### "How do I show a preview of a design without creating a project?"
→ **61_PIXFIZZ_API.md** § 9 (Dynamic Design Previews)

### "What JS API functions are available / how do I use Pixfizz.setup / createProject / openProject?"
→ **61_PIXFIZZ_API.md** § 10 (JS API)

### "What does the Shopify JS API (Pixfizz.Shopify.*) do?"
→ **60_SHOPIFY_INTEGRATION.md** (not the general JS API)

### "How does Pixfizz receive status callbacks from a fulfillment partner (Advertek, Navitor, etc.)?"
→ **61_PIXFIZZ_API.md** § 13 (Fulfillment Partner Callbacks)

---

## Shopify Integration

Use **60_SHOPIFY_INTEGRATION.md** for all questions relating to the Shopify + Pixfizz deployment path.

### "What Shopify metafields do I need to set up?"
→ **60_SHOPIFY_INTEGRATION.md** § 2

### "Which integration type should I use for this product?"
→ **60_SHOPIFY_INTEGRATION.md** § 3

### "What do _pixfizz_project_id / _pixfizz_addon / _pixfizz_unit_quantity do?"
→ **60_SHOPIFY_INTEGRATION.md** § 4

### "What does each Pixfizz Shopify snippet do / how do I use it?"
→ **60_SHOPIFY_INTEGRATION.md** § 5

### "How do I set up the Shopify cart page for Pixfizz?"
→ **60_SHOPIFY_INTEGRATION.md** § 7

### "How do addon products / extra pages work in Shopify?"
→ **60_SHOPIFY_INTEGRATION.md** § 8

### "How do I set up the order sync webhook?"
→ **60_SHOPIFY_INTEGRATION.md** § 9

### "How do I link a product in Shopify to Pixfizz?"
→ **60_SHOPIFY_INTEGRATION.md** § 10

### "Why isn't the preview showing / addon not updating / quantity not locking / order not confirming?"
→ **60_SHOPIFY_INTEGRATION.md** § 11

### "What is Buy Now, Personalize Later / how does the draft order flow work?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md**

### "How does the draft button work / what does draft: true do in the launch handler?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 2–3

### "How does the customer approve their draft order / what is orderline_commit?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 7

### "How does the magic link email work for draft orders / what is signin_token?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 6

### "When does fulfillment trigger for a draft order / what is order status W?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 5 + 7

### "Which integration types support Buy Now, Personalize Later?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 1

### "What CMS pages make up the Shopify CMS site (the non-Shopper Pixfizz CMS backing the Shopify integration)?"
→ **62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md** § 9

### "How do I add a Saved Projects or My Galleries page to a Shopify store?"
→ **60_SHOPIFY_INTEGRATION.md** § 13

### "How does the Pixfizz session work on a Shopify page / can I call the Pixfizz API from Shopify JS?"
→ **60_SHOPIFY_INTEGRATION.md** § 13

### "What methods does Pixfizz.Shopify expose / what is Pixfizz.Shopify._user / pixfizz_origin?"
→ **60_SHOPIFY_INTEGRATION.md** § 13

### "How do I get a gallery thumbnail at a higher resolution / thumbnail URL size parameter?"
→ **60_SHOPIFY_INTEGRATION.md** § 13

### "How do I configure local pickup so Shopify orders route to the right address in Pixfizz?"
→ **60_SHOPIFY_INTEGRATION.md** § 12

### "Shopify pickup order has no address in Pixfizz / webhook has no shipping address?"
→ **60_SHOPIFY_INTEGRATION.md** § 12

---

## Scope warning addition

Under "If a site is using the Shopify deployment path" add:

- 62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md (draft order / Buy Now, Personalize Later flow)

---

## MyPixfizz (my.pixfizz.com)

MyPixfizz is a separate product from the Pixfizz CMS platform — it is the internal ERP/CRM and customer self-service portal for Pixfizz. Use the 70-series files for all MyPixfizz questions.

### "What is MyPixfizz / what does it do / who uses it?"
→ **70_MYPIXFIZZ_OVERVIEW.md**

### "What routes/pages exist / what does a specific feature do?"
→ **71_MYPIXFIZZ_FEATURES_ROUTES.md**

### "What database tables exist / what columns does a table have / how are tables related?"
→ **72_MYPIXFIZZ_DATA_MODEL.md**

### "Help me write a Lovable prompt for MyPixfizz"
→ **71_MYPIXFIZZ_FEATURES_ROUTES.md** § Prompt-Writing Notes, then **72_MYPIXFIZZ_DATA_MODEL.md** for table/column accuracy

> **Scope note:** MyPixfizz files (70–72) are entirely separate from Pixfizz CMS files (10–61). Do not apply Shopper, Liquid, or pricing rules to MyPixfizz questions.

---

## Retrieval preference rules (for best answers)

- Prefer **the narrowest file** that matches the question (prevents concept bleed).
- If a question spans layers (e.g., cart + pricing), answer in this order:
	1) Template behavior (Shopper) or integration layer (Shopify / custom)
	2) Platform truth
	3) Pricing/Fulfillment (commercial engine)
- When uncertain whether a behavior is template-level or platform-level:
	→ consult **13_TEMPLATE_BOUNDARIES.md** first.

---

## Scope warning

If a site is not using the Shopper template, treat Shopper files as **non-authoritative** and fall back to:
- 10_CORE_MENTAL_MODEL.md
- 11_USER_IDENTITY_MODEL.md
- 12_OBJECTS_AND_OWNERSHIP.md
- 30_PRICING_ENGINE.md
- 31_FULFILLMENT_ENGINE.md
- 50_SHOPPER_TEMPLATE_REFERENCE.md (Shopper structural anatomy — layouts, nav, theming, checklist keys)

If a site is using the **Shopify deployment path**, Shopper files are non-authoritative for cart/checkout. Use:
- 60_SHOPIFY_INTEGRATION.md (Shopify-specific cart, snippets, metafields, order sync)
- 10/11/12/30/31 (platform truth — still applies)

### "Automatic discounts / cart-level discounts without promo code / auto discount formula?"
→ **30_PRICING_ENGINE.md** § Automatic Discounts

### "Static product importer / CSV product import / bulk product upload?"
→ **18_ADMIN_NAVIGATION.md** § Products (Static Product Importer)

### "Inventory tracking / stock levels / out of stock / product availability?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § Inventory Tracking
→ **18_ADMIN_NAVIGATION.md** § Inventory Management

### "Login modal / sign in from editor / design tool login?"
→ **17_DESIGN_TOOL.md** § Login Modal

### "myPixfizz / customer portal / hub / Meltingspot replacement?"
→ **15_PLATFORM_ARCHITECTURE.md** § myPixfizz Hub

### "Gift vouchers / voucher code tracking / gift card?"
→ **18_ADMIN_NAVIGATION.md** § Marketing (Gift Vouchers)

### "Infrastructure / Rails version / Ruby version / Prawn / PDF library?"
→ **15_PLATFORM_ARCHITECTURE.md** § Infrastructure Versions

### "AI generated images / AI video / Higgsfield / marketing imagery / lifestyle shots / can I use AI product photos?"
→ **83_AI_IMAGERY_PRODUCTION.md**

### "Can AI generate my product images? / AI product photography / is AI imagery allowed on product pages?"
→ **83_AI_IMAGERY_PRODUCTION.md** § Part A (answer is no for the product itself)

### "Customer training / AI setup / webinar schedule / support channels?"
→ **90_FAQ.md** § Section 1 (support channels and training)
→ *Note: `80_ONBOARDING.md` has no "Customer Training and Support Resources" section. This route was recorded in the 2026-05-21 sync against content that was never added. Corrected 2026-08-14.*

### "Communication preferences / email opt-in / SMS opt-in / marketing consent?"
→ **50_SHOPPER_TEMPLATE_REFERENCE.md** § Account Page Redesign (Communication Preferences)
→ **51_CUSTOM_FIELDS_REFERENCE.md** § User fields

If a site is using a **custom eCommerce integration** (external storefront, not Shopify):
- 61_PIXFIZZ_API.md § 7 (user handoff, project workflow, fulfillment)
- 10/11/12/30/31 (platform truth — still applies)
- Shopper and Shopify files are non-authoritative

## Changelog
- 2026-03-13: Added Shopify Integration section and Shopify scope warning.
- 2026-03-26: Added MyPixfizz section pointing to 70/71/72 files.
- 2026-03-30: Added Pixfizz API section pointing to new 61_PIXFIZZ_API.md. Added custom eCommerce scope warning.
- 2026-04-03: Added 19_XML_TEMPLATE_REFERENCE.md entry for XML template page parameters.
- 2026-04-05: Added 51_CUSTOM_FIELDS_REFERENCE.md — master custom field audit across all v2 object types.
- 2026-04-06: Added 90_FAQ.md — customer-facing Q&A for store owners/operators.
- 2026-04-07: Added 03_VERSION_SCOPE.md — v1/v2 versioning convention and audit status.
- 2026-04-09: Added 62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md — full Buy Now, Personalize Later / draft order flow documentation.
- 2026-04-12: Added retrieval entries for §12 (local pickup) and §13 (variable pages / gallery API) in 60_SHOPIFY_INTEGRATION.md.
- 2026-04-18: Added retrieval entry for Design Tool Blend Modes (17_DESIGN_TOOL.md § Blend Modes).
- 2026-04-29: Added retrieval entries for Mapped Preview behavior (17_DESIGN_TOOL.md § Mapped Preview Behavior), Page Masks (17_DESIGN_TOOL.md § Page Masks), Design Option same-code conflict (16_PRODUCT_HIERARCHY.md § Variants vs Options), Caption translation limitations (19_XML_TEMPLATE_REFERENCE.md § Caption Translation Limitations), production file regeneration and font preprocessing (40_PLAYBOOK.md).
- 2026-05-21: Added 45_ORDERHUB.md — comprehensive OrderHub reference (Jobs, Production Board, Processes, Locations, PDF Layout Studio, PrintNode, Film Scans, OHD, EasyPost, POS, category assignment, Email/SMS/RCS notifications). Added retrieval entries for all OrderHub topics. Source: OrderHub help modal articles.
- 2026-05-21: Added routing entries for automatic discounts, static product importer, inventory tracking, login modal, myPixfizz hub, gift vouchers, infrastructure versions, customer training, communication preferences. Source: Q1 2026 webinar KB sync.
- 2026-07-26: Added 83_AI_IMAGERY_PRODUCTION.md — AI marketing imagery and video production with Higgsfield. Added routing entries for AI imagery and the product-representation rule.
- 2026-08-14: Audited the 2026-05-21 routing entries against the files they point at. Two pointed at sections that did not exist: Gift Vouchers (now written into 18_ADMIN_NAVIGATION.md § Marketing) and Customer Training and Support Resources (re-pointed to 90_FAQ.md § Section 1). Source: kbsync audit.


=================================================================
FILE: 03_VERSION_SCOPE.md
=================================================================

# 03 — Version Scope Convention

**Authority Scope:** Defines how Shopper version references work across all knowledge files, skills, and memory. This file does not introduce platform or template rules — it governs documentation clarity only.

_Last updated: 2026-04-07_

---

## The two Shopper versions

| | Shopper v1 | Shopper v2 |
|---|---|---|
| **Bootstrap** | 4.6 | 5 |
| **Status** | Production (current live sites) | In development (new template) |
| **Parent template** | shopper24.pixfizz.com | New v2 parent (BS5-based) |
| **Admin** | /site/setup/* — 150+ flat toggles (never completed) | 4-page admin: Brand & Theme, Storefront, SEO, Integrations |
| **Theming** | custom.css + checklist keys | brand.md config files |
| **New sites** | Existing sites remain on v1 | New sites will launch on v2 |

---

## Default version rule

**Unlabelled "Shopper" = Shopper v1.**

Any reference to "Shopper", "Shopper template", or "Shopper site" without an explicit version qualifier describes **Shopper v1** (Bootstrap 4.6, current production template).

This applies across all knowledge files, skill files, and memory files.

---

## How to label version-specific content

### In knowledge files (10–90 series)

- **v1-only content:** Can remain unlabelled (default), but adding `(v1)` is preferred for clarity when v2 content exists in the same file.
- **v2-only content:** MUST be labelled. Use one of:
  - Inline: `Shopper v2`, `(v2 only)`, `(BS5)`
  - Section header: `### Checkout flow (v2)`
  - Authority scope line: `**Authority Scope:** Shopper v2 only.`
- **Content that applies to both:** Label as `(v1 + v2)` or `(all versions)`.
- **Content that differs by version:** Use a comparison table or separate sections with clear headers.

### In skill files

- Skills that apply to v1 only: state `Shopper v1` or just `Shopper` (default).
- Skills that apply to v2 only: MUST state `Shopper v2` in the description and body.
- Skills that apply to both: state `Shopper (v1 + v2)` in the description.

### In memory files

- Memory entries about v2 work already use `v2` in their filenames (e.g., `project_shopper_v2_bs5.md`). Continue this convention.
- Generic Shopper memories without a version qualifier refer to v1.

---

## Key differences to watch for

These are the areas where v1 and v2 diverge enough that version confusion causes real bugs or wrong advice:

1. **CSS framework** — v1 uses Bootstrap 4.6 classes; v2 uses Bootstrap 5. Class names, grid system, utility classes, and JS plugins differ.
2. **Admin configuration** — v1 uses checklist keys and /site/setup/* toggles; v2 uses brand.md files and a 4-page admin.
3. **Theming** — v1 uses custom.css snippets and style tokens; v2 uses brand.md-driven theming.
4. **Navigation** — v1 nav styles (style1–style4, megamenu) may differ structurally in v2.
5. **Custom fields** — `51_CUSTOM_FIELDS_REFERENCE.md` is v2-derived. Field names may differ from v1.
6. **Snippet namespaces** — v2 may use different snippet names or structures.
7. **JS patterns** — v2 drops jQuery dependency patterns from BS4.6.

---

## Current audit status (2026-04-07)

### Knowledge files (10–90 series)

| File | Current state | Action needed |
|---|---|---|
| 51_CUSTOM_FIELDS_REFERENCE.md | Explicitly v2 | None |
| 50_SHOPPER_TEMPLATE_REFERENCE.md | Unlabelled (v1) | Add authority scope note: v1 |
| 20_SHOPPER_CART_RULES.md | Unlabelled (v1) | Add authority scope note: v1 |
| 21_SHOPPER_CHECKOUT_POLICY.md | Unlabelled (v1) | Add authority scope note: v1 |
| 22_OPTION_VARIANT_RENDERING.md | Unlabelled (v1) | Add authority scope note: v1 |
| 40_PLAYBOOK_UPDATED.md | Unlabelled (v1) | Add authority scope note: v1 |
| 90_FAQ.md | Unlabelled (mixed?) | Review: some answers may apply to both |
| 00_PROJECT_DIRECTIVE.md | Unlabelled (v1) | Add version convention reference |
| 02_RETRIEVAL_MAP.md | Unlabelled | Add version-routing guidance |
| 10_CORE_MENTAL_MODEL.md | Unlabelled (v1) | Minor — platform-level, version-neutral concepts |
| 13_TEMPLATE_BOUNDARIES.md | Unlabelled (v1) | Minor — responsibility matrix applies to both |
| 14_GLOSSARY.md | Unlabelled | Add v1/v2 definitions |
| 15_PLATFORM_ARCHITECTURE.md | Unlabelled | Minor — platform-level |
| 18_ADMIN_NAVIGATION.md | Unlabelled (v1) | Add note: v1 admin layout |
| 61_PIXFIZZ_API.md | Unlabelled | Minor — API layer, version-neutral |
| 12_OBJECTS_AND_OWNERSHIP.md | Unlabelled | Minor — platform-level |
| 31_FULFILLMENT_ENGINE.md | Unlabelled | Minor — platform-level |
| 19_XML_TEMPLATE_REFERENCE.md | Explicitly platform-level | None |
| 23_XML_CALENDAR_REFERENCE.md | Explicitly platform-level | None |

### Skill files

| File | Current state | Action needed |
|---|---|---|
| pixfizz-liquid-patterns/SKILL.md | Unlabelled | Add version scope note (v1 default, flag v2 differences) |
| pixfizz-shopper-ux/SKILL.md | Unlabelled | Add version scope note |
| pixfizz-onboarding-playbook/SKILL.md | Unlabelled | Add version scope note (onboarding may differ for v2) |

### Memory files

| File | Current state | Action needed |
|---|---|---|
| project_shopper_v2_*.md (5 files) | Explicitly v2 | None |
| project_skip_cart_redirect.md | Unlabelled | Minor — feature exists in v1, likely also v2 |
| reference_pixfizz_knowledge_files.md | Mixed | Already updated with v2 custom fields ref |

---

## Retrieval guidance

When answering a Shopper question:

1. **Determine the version context first.** If the user or site is on v2, prioritise v2-labelled files. If unclear, ask.
2. **Check this file (03_VERSION_SCOPE.md)** for which files are v1 vs v2.
3. **Do not mix v1 and v2 guidance** in a single answer without clearly labelling which is which.
4. **When in doubt, state the version** of the advice you are giving.

---

## Changelog

- 2026-04-07: Created. Initial audit of all knowledge files, skills, and memory.


=================================================================
FILE: 10_CORE_MENTAL_MODEL.md
=================================================================

# 10 — Core Mental Model

**Authority Scope:** Platform-level conceptual architecture only.

_Last updated: 2026-03-30_

---

# 01 — Core Mental Model (Locked Canon)

## Four-layer architecture
Pixfizz is organized around four capability layers:
1. **Personalization Layer** — product configuration, design tools, XML templates. Core of every deployment.
2. **Commerce Layer** — storefront, checkout, catalog, payments. Only active in Full Pixfizz.
3. **Workflow Layer** — OrderHub: order management, artwork generation, asset delivery, production jobs.
4. **Website Layer** — Shopper, custom frontend, theming, domain/SSL. Only in Full Pixfizz.

See **15_PLATFORM_ARCHITECTURE.md** for deployment models and full layer details.

## Platform vs Shopper vs Custom
- Pixfizz CMS provides objects, identity, pricing engine, admin/custom fields.
- Shopper is an opinionated storefront implementation (cart/checkout UI rules).
- Custom storefronts may differ from Shopper even while using the same CMS.

### How to identify if a site is on Shopper
Click the ellipsis icon at the bottom of the Pixfizz admin sidebar. The modal shows "Source Websites" — `shopper24.pixfizz.com` confirms the site is a Shopper child site. Child sites do not expose the full CMS pages list, so wildcard page detection is not a reliable method.

## Cart vs checkout
- Cart does not change based on payment method.
- Payment options vary on checkout only.

## Digital-only
- Digital-only is a static product with variant `version` == `digital-only`.
- Checkout ignores shipping and applies a public/system address internally (hidden); customer sees digital delivery.
- No special cart behavior for digital-only.

## Photo prints
- Photo prints quantities are managed in the Photo Prints UI and priced via `cut_print_quantity`.
- Cart quantity is not editable for photo prints.


=================================================================
FILE: 11_USER_IDENTITY_MODEL.md
=================================================================

# 11 — User Identity Model

**Authority Scope:** Identity lifecycle rules only.

_Last updated: 2026-02-26_

---

# 02 — User Identity Model (Locked Canon)

## Core rule
A user **always exists**.

## Anonymous user
- Session-based identity stored in browser.
- Owns cart/projects/galleries during browsing.

## Registered user
- Persisted user record with credentials.
- Can be logged out or logged in.

## Logged-in state
- Registered user with an authenticated session.
- On login, anonymous user data is passed/transferred to the existing user.

## Guest user
- Guest checkout creates a persisted **guest user** flagged as guest.
- Guest cannot login or reset password.
- Guest can receive emails and owns orders/projects/galleries.
- Admin can merge guest into registered user, transferring data.

## Admin / Super Admin
Roles/permissions on user accounts.


=================================================================
FILE: 12_OBJECTS_AND_OWNERSHIP.md
=================================================================

# 12 — Objects & Ownership

**Authority Scope:** Object relationships and ownership rules only.

_Last updated: 2026-02-26_

---

# 03 — Objects & Ownership

All objects attach to the current user identity (anonymous/guest/registered) and follow identity conversion rules.

## Cart & orderlines
- `cart.orderlines` are the atomic commerce units.
- Orderlines may reference product, chosen variants/options, and (sometimes) a project.

## Projects
- Design flows create/attach projects.
- Cart edit routing (Shopper) can send user to project-edit page or editor based on `orderline.product.custom.btn_add_to_cart`.

## Photo prints
- Special UI builds selections; cart receives orderlines by size group.
- Quantity derived via `cut_print_quantity`.


=================================================================
FILE: 13_TEMPLATE_BOUNDARIES.md
=================================================================

# 13 — Template Responsibility Boundaries

**Authority Scope:** Defines CMS vs Shopper vs Custom vs Shopify responsibility.

_Last updated: 2026-03-13_

---

# 07 — Template Responsibility Boundaries

## Pixfizz CMS platform
- Objects, identity model, pricing engine, admin + custom fields.
- Applies to all deployment paths.

## Shopper template
- Cart UX and checkout policy engine (kiosk mode, shipping/pickup availability, film, digital-only logic).
- Only applies to sites using the Shopper template.

## Custom storefronts
- May reuse Pixfizz CMS but differ from Shopper behavior.

## Shopify deployment path
- Shopify owns the storefront, cart, checkout, and payment.
- Pixfizz owns personalization, project storage, and production orchestration.
- Cart and checkout rules from Shopper files (20/21) do not apply.
- Integration is handled via Shopify Liquid snippets, product metafields, and a JS API loaded from the Pixfizz subdomain.
- See **60_SHOPIFY_INTEGRATION.md** for all Shopify-specific implementation details.

## Where to implement changes

| Change type | Shopper sites | Shopify sites |
|---|---|---|
| Feature toggles / policy | Admin/Custom Admin checklist | Shopify product metafields |
| Cart/checkout rendering | Liquid snippets (Shopper) | Shopify theme snippets |
| Personalization launch | Pixfizz CMS product pages | Shopify product template + `pixfizz-launch-product-handler` |
| JS interaction | Interaction only — do not replace pricing/object truth | Pixfizz JS API (`Pixfizz.Shopify.*`) |
| Pricing | Pixfizz pricing formulas (Ruby) | Shopify product/variant prices + Pixfizz addon products |
| Order confirmation | Pixfizz checkout flow | Shopify payment webhook → Pixfizz |

## Custom Fields — No Parent→Child Inheritance

Custom fields are **site-specific**. A custom field created on the parent template site (e.g. `shopper24.pixfizz.com`) does **not** automatically appear on child sites.

Each site manages its own custom fields independently. If a new layout feature depends on a custom field (e.g. `product.custom.hide_gallery`), that field must be created manually on each child site that needs it.

This applies to all custom field object types: product, collection, design, user, order, and orderline.

RATIONALE: Confirmed during a client hide_gallery implementation — field created on parent was not present on child.
SOURCE: "Custom field to hide product gallery" chat, April 24

## Site Assets — Inherited Parent→Child

Site assets (files uploaded under **Main Admin > Website > Assets**: JS, CSS, images, fonts) **are** inherited parent→child, the same way snippets are. An asset uploaded to the parent template site resolves on every child site without being re-uploaded.

This corrects the entry previously held in this section, which stated the opposite. It was confirmed on a new child-site deployment of a custom design tool: the asset resolved correctly and the real fault was elsewhere.

**The failure mode is the include, not the asset.** A script asset is normally referenced from `integrations/custom-body-scripts`. That snippet is routinely overridden on child sites to carry their own analytics and third-party tags, so a `<script>` include added to the parent copy is invisible on any child that has an override. The symptom is identical to a missing asset — the page renders, the script never loads, no error.

Two rules follow:

- Where a tool depends on a script asset, have the tool's own product snippet load its dependencies rather than relying on a shared include snippet that child sites override. See **41_IMPLEMENTATION_PATTERNS_UPDATED.md § Custom Tool Dependency Loading**.
- Assets are aggressively browser-cached. When a JS asset is re-uploaded, expose a version marker on the script's public namespace (e.g. `MyTool.version`) so the deployed build can be confirmed from the browser console in one step, rather than guessing whether a change failed or is simply cached.

Note the asymmetry with custom fields: snippets and assets **are** inherited parent→child; custom fields are **not**. Do not generalise from one to the other.

## Parent-First Rule — What It Does Not Cover

Child sites can only override snippets that already exist on the parent; they cannot create net-new snippets. See **01_CODE_GOVERNANCE**.

That constraint applies to **snippets only**. It does not apply to:

- **Product template options and variants** — these are product data, created directly on whichever site owns the product. There is no parent equivalent to create first.
- **Custom fields** — created per site (see above).
- **Site assets** — uploaded on the parent and inherited.

Treating template options and variants as parent-first produces unnecessary edits on `shopper24.pixfizz.com`, which carry blast radius across every child site for no benefit.

## Changelog
- 2026-03-13: Added Shopify deployment path as a distinct boundary layer.
- 2026-07-25: Added Site Assets — No Parent→Child Inheritance, including the silent-failure mode for JS assets and the version-marker practice for confirming a deployed build past browser cache. Source: claude-chat.
- 2026-07-28: Corrected Site Assets section — assets ARE inherited parent to child; the silent failure previously attributed to asset inheritance is a child override of `integrations/custom-body-scripts`. Added Parent-First Rule scope note (snippets only, not template options or variants). Source: claude-chat.


=================================================================
FILE: 14_GLOSSARY.md
=================================================================

# 14 — Glossary

**Authority Scope:** Definitions only.

_Last updated: 2026-03-30_

---

# Glossary (Pixfizz)

## Platform vs template
- **Pixfizz CMS (platform):** objects, pricing engine, identity model, admin + custom fields.
- **Shopper (template):** an opinionated storefront implementation on top of Pixfizz CMS.
- **Custom storefront:** a site using Pixfizz CMS but not the Shopper theme behavior.

## User states
- **Anonymous user:** session-based identity stored in browser; owns cart/projects while browsing.
- **Guest user:** persisted user record flagged as guest; cannot login/reset password; owns orders/projects; mergeable into registered.
- **Registered user:** persisted user record with credentials.
- **Logged-in:** a registered user with an authenticated session.
- **Admin / Super Admin:** permission roles on a user.

## Print products
- **Photo Prints UI:** special workflow where per-photo quantities are set outside cart.
- **cut_print_quantity:** derived count used for pricing for photo print orderlines.

## Delivery
- **Digital-only:** static product variant `version` == `digital-only` triggers checkout-only behavior (ignore shipping, hidden public address).

## Pricing
- **Pricing formula:** Ruby expression returning a numeric price.
- **Price Variable:** admin-defined numeric constant available by name (e.g., `hardcover_base_a4`, `whitelabel`).


## Fulfillment
- **Fulfillment Template:** A template that outputs a job ticket payload (JSON/XML/TXT) and references generated artwork files for an external lab/system.
- **Job ticket:** The payload sent to a provider describing what to produce; often accompanies artwork files.
- **Generated files:** Output files produced for an orderline (often per page/layer), accessible via `orderline.generated_files`.
- **Filename Template:** Admin-configured pattern used to name (and optionally folder-route) generated files; commonly uses `order.code`, `orderline.barcode`, `page_output_name`, `layer_output_name`, `idx`, and `format`.
- **Fulfillment Destination:** Where production assets are delivered. Configured in Super Admin. Supports FTP/SFTP and HTTP delivery.
- **Fulfillment Transformation:** Rules controlling how artwork is processed for production (color profile, format, bleed). Exist at Template level and Design level (Design overrides Template).
- **Split by orderline:** Setting that fans out one order into multiple fulfillment requests (one per orderline).
- **Hotfolder:** A watched folder/shared drive where jobs are delivered by dropping files/manifests.
- **Adapter:** A mapping from the Pixfizz Default Fulfillment JSON to a vendor-specific payload.

## Platform architecture
- **Personalization Layer:** The core Pixfizz layer — product configuration, design tools, image workflows, XML templates. Used in every deployment.
- **Commerce Layer:** Storefront, checkout, catalog, payments, shipping. Only active in Full Pixfizz deployments.
- **Workflow Layer:** Order management, artwork generation, asset delivery, OrderHub. Active in all deployments.
- **Website Layer:** Storefront building tools (Shopper, custom frontend, theming, domain/SSL). Only in Full Pixfizz.

## Products
- **Pixfizz Core:** The personalization engine — product configuration, templates, designs, design tool, and admin interface.
- **Product Attribute:** The commercial side of a product (pricing, variants, packaging). Links to a Template (design product) or stands alone (static product).
- **Template:** Production specification for a product (dimensions, DPI, XML definition, page range, fulfillment rules).
- **Design:** Customer starting point under a Template (pages, layouts, backgrounds, clipart, masks).
- **Collection:** Grouping mechanism for publishing products to the storefront. URL: `/shop/:collection/:product/:design`.
- **Design product:** A Product Attribute linked to a Template. Customer personalizes before purchase.
- **Static product:** A Product Attribute without a Template link. Standard eCommerce item (frames, gift vouchers, etc.).
- **Variant:** Customer-facing option on a Product Attribute (size, finish, material) that affects pricing.
- **Option:** Production-level choice on a Template or Design (not customer-facing pricing).
- **Design Tool Configuration:** Controls design tool appearance and features for a specific context. Templates reference a configuration.

## Order lifecycle
- **OrderHub:** The Workflow Layer of Pixfizz. Manages order routing, artwork generation, asset delivery, production jobs, fulfillment tracking.
- **OrderHub Desktop (OHD):** Desktop application for production teams to download, view, and manage production jobs.
- **Order statuses:** Pending → Draft → Confirmed → Downloaded → Manufactured → Shipped → Fulfilled. Exceptions: Payment Failed, Error, Canceled, Refunded.
- **Orderline:** An individual product within an order. Each has generated files, options, fulfillment code.

## Account management
- **myPixfizz:** Account management hub — subscriptions, onboarding projects, team access, billing.
- **Pixfizz Select:** Marketplace for discovering Pixfizz-powered services and partners.
- **Pixfizz API:** Integration layer for connecting personalization and fulfillment into external platforms.
- **Super Admin:** Cross-website management layer for organizations managing multiple Pixfizz websites.


=================================================================
FILE: 15_PLATFORM_ARCHITECTURE.md
=================================================================

# 15 — Platform Architecture

**Authority Scope:** Platform-level architecture layers and deployment models only.

_Last updated: 2026-03-30_

---

## Four Platform Layers

Pixfizz is organized around four capability layers that work together:

### 1. Personalization Layer
The core of every Pixfizz deployment. Provides the tools customers use to configure and customize products before purchase.
- Product configuration interfaces (size, format, options)
- Personalization tools (image upload, text placement, design editing)
- Image upload workflows with asset management
- Template-driven product definitions using XML

Every deployment uses this layer — whether the storefront runs on Shopify, the native Pixfizz CMS, or a custom frontend via API.

### 2. Commerce Layer
Handles storefront and checkout when Pixfizz is the full commerce platform.
- Storefront CMS with page management
- Product catalog and category management
- Checkout and payment processing
- Shipping and tax configuration
- Customer accounts and order history

> Only active in Full Pixfizz deployments. In Shopify + Pixfizz (or custom API), the external platform handles this layer.

### 3. Workflow Layer
Manages everything after order placement — the journey from order to production-ready output.
- Order ingestion (from Pixfizz checkout, Shopify webhooks, or external platforms via API)
- Artwork generation from personalized designs
- Asset delivery via FTP/SFTP or HTTP
- OrderHub job creation, production routing, and production download

### 4. Website Layer
Tools for building and managing storefront experiences on the Pixfizz platform.
- Shopper — a managed, customizable website template
- Custom frontend option for businesses with own dev resources
- Theme selection and brand identity configuration
- Domain and SSL configuration
- Site navigation and homepage layout management

> Only relevant in Full Pixfizz deployments. Shopify or external platforms manage their own storefronts.

---

## How the Layers Connect

**Full Pixfizz deployment:** All four layers active and tightly integrated. Customer browses storefront (Website) → personalizes product (Personalization) → completes checkout (Commerce) → order flows into production (Workflow).

**Integrated deployment (Shopify / custom API / marketplace):** External platform handles Commerce and Website layers. Pixfizz provides Personalization and Workflow layers.

---

## Deployment Models

### Shopify + Pixfizz
- Shopify manages: storefront, checkout, commerce
- Pixfizz manages: personalization, product configuration, order management
- Orders originate in Shopify → ingested via webhooks → production pipeline via OrderHub
- Best for: existing Shopify stores adding advanced personalization

### Full Pixfizz
- Pixfizz manages: storefront (Shopper or custom), checkout, personalization, order management, production workflows
- All four platform layers active
- Best for: new businesses, replacing aging platforms, tight storefront-to-production integration

### Custom API Integration
- External platform (WooCommerce, Magento, custom-built) handles storefront/checkout
- Pixfizz provides personalization and fulfillment via API
- Requires development resources to manage integration
- Best for: non-Shopify eCommerce platforms, custom frontends

### Marketplace Integration
- Marketplace (e.g. Etsy) handles storefront/checkout
- Pixfizz manages personalization and production
- Marketplace orders flow into same production pipeline as direct sales
- Best for: sellers on Etsy or similar marketplaces

### Side-by-Side Comparison

|  | Shopify + Pixfizz | Full Pixfizz | Custom API | Marketplace |
| --- | --- | --- | --- | --- |
| Storefront | Shopify | Pixfizz (Shopper or custom) | Your platform | Marketplace |
| Checkout | Shopify | Pixfizz | Your platform | Marketplace |
| Product catalog | Shopify | Pixfizz | Your platform | Marketplace |
| Personalization | Pixfizz | Pixfizz | Pixfizz | Pixfizz |
| Order management | Pixfizz | Pixfizz | Pixfizz | Pixfizz |
| Production workflows | Pixfizz | Pixfizz | Pixfizz | Pixfizz |

> All deployment options include full access to Personalization and Workflow layers. The choice comes down to storefront and checkout ownership.

---

## Key Platform Products

- **Pixfizz Core** — The personalization engine: product configuration, templates, designs, design tool, and admin.
- **OrderHub** — The Workflow Layer: order management, production routing, fulfillment. Includes OrderHub Desktop (OHD) for production teams.
- **Shopper** — Managed website template within the Pixfizz eCommerce CMS. Not a standalone product.
- **myPixfizz** — Account management hub: subscriptions, onboarding projects, team access, billing.
- **Pixfizz Select** — Marketplace for discovering Pixfizz-powered services and partners.
- **Pixfizz API** — Integration layer for connecting personalization and fulfillment into any external platform.

---

## Changelog
- 2026-03-30: Created from master platform documentation export.


=================================================================
FILE: 16_PRODUCT_HIERARCHY.md
=================================================================

# 16 — Product Hierarchy

**Authority Scope:** Product object relationships and catalog structure only.

_Last updated: 2026-03-30_

---

## The Four Core Objects

Pixfizz Core organizes products through a relationship model of four objects. These are not a strict top-down hierarchy — they form a flexible relationship model where objects can be shared.

### Product Attributes
The commercial side of a product — how it appears in the store, how it's priced, and what options are available.
- Name, Code, Description
- Pricing (base price, currency)
- Variants (size, finish, material — customer-facing options that affect pricing)
- Packaging (physical packaging specs)
- Template Link — connects to its production specification
- Preview Images
- Custom Fields

A Product Attribute linked to a Template = **design product** (requires personalization).
A Product Attribute without a Template link = **static product** (standard eCommerce item like frames, gift vouchers, accessories).

### Templates
The production specification — the technical blueprint for how personalized artwork is generated.
- Name, Code, Category
- XML Definition — layer structure defining editable and fixed elements
- Page Range — min/max pages (for multi-page products)
- DPI — resolution for production output
- Units (mm, inches, pixels)
- Product/Print Dimensions
- Design Tool Configuration — which design tool profile to use
- Cut Print settings
- Fulfillment Transformations — rules for processing artwork for production
- Options — production-level choices (not customer-facing pricing options)
- Preview Images and Preview Sections
- Mapped Previews — advanced preview configuration

Multiple Product Attributes can share the same Template (e.g. "metal print" and "acrylic print" with identical production specs but different pricing).

### Designs
What the end customer starts from when personalizing. Each Design sits under a Template.
- **Pages** — what the customer sees and edits. Each page has layers: preview-clean, preview, print.
- **Layouts** — switchable content arrangements per page (e.g. one photo centered, two side-by-side, three in grid). Can be linked across designs and organized into folders using tags.
- **Backgrounds** — background images/colors customers can apply
- **Clipart** — decorative elements customers can add
- **Masks** — shape masks controlling image crop/framing
- **Design Options** — variant-like choices at the design level
- **Fulfillment Transformations** — design-specific overrides to Template fulfillment rules
- Preview Images, Custom Fields

A Template can have one Design (e.g. single full-bleed metal print) or hundreds (e.g. greeting card with designs for every occasion).

### Collections
Grouping mechanism for publishing products to the storefront.
- Name, URL path
- Description, asset image
- Preview Images, Custom Fields

Publishing a product = adding it to a Collection. Storefront URL structure: `/shop/:collection/:product/:design`.

---

## Variants vs Options

Same mechanism, different objects:
- **Variants** live on Product Attributes — commercial choices (size, finish) that affect pricing
- **Options** live on Templates or Designs — production-level choices

This split avoids redundancy when multiple Product Attributes share a Template. Common production options live on the Template; product-specific pricing/commercial choices live on each Product Attribute.

---

## Typical Flow for a Design Product

1. **Create a Template** — define production specs (dimensions, DPI, XML definition)
2. **Add Designs to the Template** — each design is a customer starting point
3. **Create a Product Attribute** — set pricing, variants, packaging; link to Template
4. **Add to a Collection** — publish on storefront
5. **Customer personalizes and purchases** — browse collection → select design → personalize → add to cart

---

## Supporting Product Features

Beyond the core hierarchy:
- **Fonts** — upload and manage fonts for the design tool
- **Font Palettes** — curated font sets
- **Color Palettes** — color sets for text, borders, backgrounds
- **Element Substitutions** — rules for swapping design elements based on conditions
- **Calendars** — configuration for calendar-type products

---

## Changelog
- 2026-03-30: Created from master platform documentation export.


=================================================================
FILE: 17_DESIGN_TOOL.md
=================================================================

# 17 — Design Tool

**Authority Scope:** Design Tool Configurations, feature toggles, and customer-facing editor behavior.

_Last updated: 2026-07-31_

---

## What is the Design Tool?

The Design Tool is the interactive editor customers use to personalize products. It is embedded in the storefront and provides a visual interface for uploading images, adding text, applying layouts, and previewing the finished product.

The Design Tool is highly configurable through **Design Tool Configurations**.

---

## Design Tool Configurations

Each Pixfizz environment can have one or more Design Tool Configurations. A configuration controls the design tool's appearance and available features for a specific context. Templates reference a configuration, so different product types can offer different design experiences.

Configured in admin under: **Settings > Design Tool**.

### Multiple configurations per site

A site can hold several configurations side by side. A configuration is assigned to a **Template** or to a **Design** — it is not attached to a product, a collection, or a category. Whichever object the customer enters the editor through determines the configuration that loads.

Running several configurations is the intended pattern where different product types need genuinely different editors (a card versus a photo book versus a wall-art product). Assign each Template or Design to the configuration that suits it rather than trying to serve everything from one configuration by toggling features on and off.

### Per-configuration help modals

The in-editor help modal content is a snippet, and a different snippet can be assigned per configuration through the Trip JS tutorial configuration. This means each configuration can carry its own tutorial and instructions without duplicating the editor.

Trip JS tutorial blocks carry separate desktop and mobile sections. When adapting desktop content for mobile, use fluid dimensions (`width: 100%`, `max-height: 60vh`, `overflow-y: auto`, `box-sizing: border-box`) rather than fixed pixel width/height. Desktop copy wraps to far more lines at phone width, so a fixed height clips the lower paragraphs and a fixed width sits inset inside the Trip modal leaving white gaps.

---

## Configuration Settings

### Branding
- Name, logo, and favicon
- Brand color
- Page title

### Feature Toggles (30+)

#### Image Features
- Image Rotation
- Crop Toggle
- Color Adjustments
- Filters
- Flip
- Image Borders
- Image Size Overlay
- QR Uploader
- PDF Imports

#### Layout & Design Features
- Autofill Button — auto-populate images into zones
- Two Page Spread — enable spread view for multi-page products
- Crop Bleed — show/hide bleed area
- Alignment Aids — snap guides and alignment helpers
- Background Colors — allow background color changes
- CMYK Color Picker — professional color selection
- Border Radius — rounded corner controls
- Shape Button — add shapes to designs
  - Shapes support color palettes (selectable from the configured color palette)
  - Shapes work with fulfillment transformations and calendar transformations
- QR Code Button — generate QR codes in designs

#### UX Features
- Unedited Warning — alert if zones haven't been customized
- Project Options — show project-level settings
- Display Price — show pricing in the design tool
- Copy Shared Projects — allow copying shared project links
- Highlight Editable Elements — visual indicators for editable zones
- Use Mapped Preview — advanced preview rendering

#### Specialist Features
- Large Format / Wall-Art mode
- Cut Print Autorotate

**AI image tools (URL flag):** the AI image-manipulation tools in the editor are gated behind a URL parameter. Append `&aitools=true` to the design-tool URL (`?aitools=true` if there are no other query params) to expose them. Platform-level; works wherever the editor loads. Source: #development (2026-07-01).

**AI restyle presets (launch set).** Seven styles ship in the Restyle section:

- Watercolor
- Pencil Sketch
- Oil Painting
- Cartoon
- Pop Art
- 3D Character
- Anime

Vintage Film was drafted during development and is **not** in the launch set. Generation is billed per image to the lab, not to Pixfizz, so usage carries a daily limit with a site default and per-user overrides.

**Filter selection auto-applies.** Selecting a filter (Restyle preset or Image effects filter) now applies it immediately. Previously, selecting a filter only showed a preview — if the customer went straight to cart without pressing an explicit **Apply** button, the filter was silently discarded. This has been changed to auto-apply on selection so the result can no longer be lost this way. Deployed 2026-07-27. Source: slack-message (#development), commit 8e2736ec.

**Known issue — AI token usage counted globally, not per site (fix deploying to production the week of 2026-08-03).** Enabling AI tools on one site could show token usage that included consumption from other sites on the same account, rather than being scoped to that site alone (e.g. enabling AI on `photosynthesis.pixfizz.com` showed tokens consumed elsewhere). A fix to scope AI usage trackers by website has been verified on staging and is pending production deploy. Until then, do not treat a site's displayed AI token count as an accurate per-site figure. Source: slack-message (#development), commit 9615afd5.

### Typography & Color Defaults
- Default Font, Font Palette, Font Size
- Text Color Palette
- Default Image Border Color Palette
- Background Color Palette

### Page & URL Settings
- Homepage URL
- Product Display Name
- Cart Page Name
- Account Page Name

### Integrations
- Google Tag Manager ID
- Image Sources — comma-separated list that specifies the **order of the source icons** in the editor's Images tab (not merely which sources are on). Allowed values: `device`, `galleries`, `public_galleries`, `groups`, `clipart`, `pdf_import`, `dropbox`, `google_photos`. Defaults to `device` when the field is empty. To expose a PDF import toolbar button you must do two things: enable the PDF Imports feature toggle (under Image Features) AND add `pdf_import` to this Image Sources field. `dropbox` and `google_photos` need provider OAuth setup before they work (see Google Photos as an Image Source below).
- Help URL
- Custom JS — inject custom JavaScript
- Custom CSS — inject custom styles

> Some Design Tool Configuration settings are only visible to Pixfizz staff. These control platform-level behaviors and are managed during onboarding or through support requests.

### Google Photos as an Image Source

`google_photos` is an allowed Image Sources token, but unlike `device`/`galleries` it needs OAuth setup before it works. Per storefront:

1. In Google Cloud, create an API Console project and a **Web Browser** OAuth Client ID. When asked for the calling domain, enter the storefront's own domain (e.g. `clientsite.pixfizz.com`). Guide: https://developers.google.com/identity/oauth2/web/guides/get-google-api-clientid
2. Complete Google's OAuth consent / API access configuration so the Client ID is authorised to fetch photos via the Google Photos API.
3. Copy the Client ID into the storefront's **Super Admin** panel, **Google OAuth 2** field (left-hand settings column).
4. Add `google_photos` to the **Image Sources** field on the relevant Design Tool Configuration.

Once the Client ID is in place and Google Cloud permissions are correct, customers can select images from Google Photos inside the editor.

## Admin Mode Editor

The "Open in Editor" action on an order, which re-opens a customer's project in
the design tool from the admin, is only visible when the Admin Mode Editor toggle
is ON. This toggle lives in the Design Tool Configuration under Editor & Templates
and is Super Admin only. It does not appear in the standard admin panel.

---

## Login Modal in the Design Tool

The design tool can be configured to show a login modal when an anonymous user attempts to save a project or access the galleries tab.

### Default behavior

- **Existing configurations:** The login modal is **not enabled by default** on existing design tool configurations. It requires additional configuration when used from an external site (such as Shopify).
- **New configurations:** Enabled by default on any new design tool configuration created in admin.

### Trigger

The login modal fires when the user clicks **"Save & Continue"**. It does **not** fire on "Save & Exit" — the Save & Exit flow relies on the target page (typically the account page) to detect whether the user is logged in and prompt login if not.

### Optional links

To show "Forgot password?" and/or "Register here" links next to the email/password fields in the modal, configure the corresponding URLs in the design tool configuration settings.

### Shopify and other external site behavior

When the design tool is embedded in an external site (such as Shopify), the login flow does **not** use the in-tool modal. Instead it opens a new browser window/tab pointing to the URL configured under **"External Login URL"** in the design tool configuration.

**Setup for Shopify:**
1. Set the **External Login URL** in the design tool configuration to a Shopify page that handles login.
2. After successful login, the user is redirected to a page that must include the Pixfizz setup code. The default Shopify integration includes the setup code on all pages — verify this for any custom theme integrations.
3. Recommended: create a custom Shopify confirmation page that says "You are now logged in. You may close this window and continue in the design tool."

The redirect-back-to-design-tool flow depends on the Pixfizz setup code running on the post-login landing page. If using a custom Shopify theme, confirm the setup snippet is present on every page before enabling the login modal.

---

## Editor CSS Customization

The editor can be re-themed with CSS. Where the CSS lives depends on deployment:
- Full Pixfizz / Shopper: the `editor.css` page.
- Shopify integration: the `shopify/custom-styles` snippet (loaded into the editor
  from the Pixfizz side, not the Shopify theme).
- Either path: the Custom CSS field in the Design Tool Configuration.

Storefront `style/custom.css` does NOT reach inside the editor iframe. Use one of
the locations above instead.

Reusable techniques confirmed in production:

- Variable aliasing. The editor exposes internal CSS custom properties (for
  example `--bright-sky-blue`, `--seaweed`). Repointing them to the brand palette
  re-themes the whole editor without targeting individual elements.
- Asset URL syntax. Inside editor CSS, wrap an uploaded asset filename in @ signs
  (for example `@Beatrice-Regular.woff2@`). The platform replaces it with the full
  asset path at render time. This is the editor equivalent of the storefront
  asset_url convention.
- Button classes. Editor buttons carry both legacy classes (`px-blue`, `px-green`)
  and newer classes (`px-primary-color`, `px-secondary-color`). Target both to
  reskin all buttons reliably.
- Tab show/hide. Tabs can be hidden via their `data-id` attribute plus `display:none`.
- Option reordering. Options can be reordered with `data-option-code` plus the CSS
  `order` property on a flex container.
- Gallery captions. Gallery folders use `px-gallery-item px-gallery`; individual
  images use `px-gallery-item px-image`. To hide image filenames while keeping
  folder names visible:

```css
.px-gallery-item.px-image .px-caption {
	display: none;
}
```

- Action button overlay gotcha. A `.btn::after` pseudo-element set to
  `position: absolute` creates an invisible overlay that swallows clicks and
  blocks hover states. Action buttons can sit in different containers
  (`px-action-buttons-container`, `px-edit-buttons-container`,
  `px-reset-button-container`, `px-controls-container`), so widen selectors across
  all relevant containers when styling them.
- Default tab on load (JS). `editor.store.ui.expandTab` sets which tab is open when
  the editor loads.
- Save and Continue callback (JS). The Save and Continue button can be
  monkey-patched to chain an action after the original behavior, using an
  `exit_target` URL parameter to control where the user lands.
- Placeholder icon position — avoid `transform` on `.px-element-icon`. Setting a
  CSS `transform` on `.px-element-icon` breaks the position of the placeholder
  (upload) icon inside image zones. To restyle that icon, use properties that do
  not create an offset/stacking context (size, color, background) rather than
  `transform`.
- Layout category order. Layout categories in the designer sort **alphabetically
  by default**. They can be re-ordered visually with the CSS `order` property on
  the category container (same flex-`order` idiom as option reordering above); the
  underlying order is not configurable in admin.

---
## Zero-Width or Zero-Height Shapes Corrupt PDF Output

A shape element with a width or height of 0 causes `NaN` to be written into the generated PDF,
producing a corrupt output file. The platform no longer writes NaN for zero-size shapes (fix
deployed Aug 2026), but any zero-width/height shapes already present in live projects should be
removed manually.

**Rendering difference by viewer:**
- Chrome PDF viewer: renders a 1px black line
- Acrobat and Firefox: render nothing

**Source:** #development Slack, Matjaz, 2026-08-17.

---
## Font Licensing: Editor Fonts Require Embedding License

Fonts used inside the Pixfizz editor are **embedded into personalised product renders** (print-ready files, previews). This requires a **digital embedding** or **print embedding** license — not a standard web font license.

Many foundries do not offer embedding licenses, or charge significantly more for them. A foundry that sells a web font license may explicitly prohibit embedding in rendered output — this is the most common reason client font purchases fail.

When advising clients on font sourcing for the editor:
- Describe the use case as: "fonts loaded into a web-based product personalisation editor, embedded into rendered output (print-ready files or customer previews)"
- Recommended sources:
  - **Paratype** (paratype.com) — strong Cyrillic catalogue, commercially experienced, generally supports embedding
  - **MyFonts** (myfonts.com) — look for Desktop / Digital Ad / App license tiers, which typically cover embedding scenarios; confirm with their support before purchasing
- Do not recommend Google Fonts for editor use: Google Fonts licenses permit web use but do not cover embedding in commercial rendered products

Note: this is separate from fonts used on the storefront (navigation, product names, body text) — those only require a web font license.

---

## Element Substitution Types

Element substitutions re-style a design's elements per template/option without editing the design itself. Alongside the existing types (element color, blend mode), the following were added in June 2026:

### Shape border substitutions

Three substitution types target shape element borders:

- **Shape border width**
- **Shape border color**
- **Shape border radius**

### Image effects substitution

A substitution type named **Image effects** applies a filter to image elements. Supported filters are **grayscale** and **sepia**.

- Configuration gotcha: set the substitution's **Name** field to `placeholder`. An earlier build where this was misconfigured threw an application error in the design tool (Canvas and More views) that broke the whole design. The `placeholder` Name value is the correct, confirmed configuration.

### Admin preview and embedded inline page behavior

Element substitutions now run on all admin page previews and on embedded inline pages. Previously, admin previews skipped element substitutions unless `fulfillment=true` was set, so a preview could look different from the actual customer-facing/fulfillment render. Deployed 2026-07-31. Source: slack-message (#development), commits dc282140, b406e757, 55667a2c.

### Known issue: colour substitutions import as black

When a template with designs is exported and imported into another site, colour element substitutions arrive as **black** rather than the assigned colour. All other substitution data comes across.

Check and reset every colour substitution manually on the destination site after any template export/import. Do not assume the values carried over because the substitution records themselves are present.

### Known issue: element substitutions on the photo prints interface

Element substitutions do not apply correctly on the newer bulk photo prints interface.

A related symptom is white borders on prints that should be borderless (or the reverse). The cause is the **layout switch resetting the crop** — moving between a bordered and borderless layout re-runs the crop and discards the previous state.

Workaround until this is fixed: organise print products into two separate categories, **with borders** and **without borders**, so the customer never switches layout mid-flow.

---

## Page Border Radius — Bleed and Margin Guides

There is no native page border-radius setting in the editor. Where a designer applies rounded corners to a page, the **bleed and margin guide lines remain square** — they are drawn against the rectangular page bounds, not the visible rounded shape.

This is a display limitation of the guides, not a production problem: the actual bleed and margin values are unaffected.

Workaround for a genuinely rounded page appearance: use **page masks** rather than attempting a border radius.

---

## Known Issues — Mobile

### Multi-page products: no indicator of which page is being edited

On mobile, the card editor gives no clear indication of which page (e.g. front vs. inside) is currently being edited — the same view is clear on desktop. This has been reported via support ticket and is not yet resolved; no fix has been confirmed as of 2026-07-31. Source: support ticket #18343.

## Element Substitutions — Opacity Control

Element opacity can be controlled through element substitutions, enabling opacity values to be
driven by product option selections or other configuration values. Useful for calendar and
photo book products where transparency effects need to vary by option choice.

SOURCE: Fireflies call (Shaun Bowen / Rapid Studio, 2026-08-18). Confirmed live: 2026-08-21.

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-23: Added font licensing rule for editor embedding (digital/print embedding license required, not web font license).
- 2026-05-27: Added shape color palette support and fulfillment/calendar transformation support under Shape Button toggle. Added Login Modal section — default behavior, trigger (Save & Continue only), optional links, and Shopify External Login URL setup. Also consolidated duplicate Changelog sections into one. Source: Notion Dashboard (May 2026 updates).
- 2026-06-01: Added Editor CSS Customization section, Admin Mode Editor note, and pdf_import Image Sources requirement. Source: claude-chat/slack.
- 2026-06-30: Documented element substitution types added June 2026 — shape border width/color/radius and the Image effects (grayscale/sepia) substitution, including the required `placeholder` Name-field value. Source: notion-dashboard (2026-06-22), slack-message (#development).
- 2026-07-04: Documented the `&aitools=true` URL flag that exposes the editor AI image tools. Source: slack-message (#development).
- 2026-07-20: Corrected Image Sources to the full allowed value set and noted it controls icon order and defaults to device. Added Google Photos setup. Source: help-article + admin tooltip.
- 2026-07-20: Added editor-CSS gotchas — `transform` on `.px-element-icon` breaks the placeholder icon position; layout categories sort alphabetically by default and can be reordered with CSS `order`. Source: slack-message (#development), loom-video.
- 2026-07-25: Clarified that a design tool configuration is assigned to a Template or a Design (not to a product or category) and that several configurations can run on one site. Added the confirmed seven-style AI restyle launch set (Vintage Film excluded) with per-lab billing and daily limits. Added per-configuration help modal snippets via Trip JS (including mobile fluid-dimension rule for Trip blocks). Added two known issues — colour element substitutions import as black after template export/import, and element substitutions failing on the bulk photo prints interface with white-border symptoms caused by the layout switch resetting the crop (workaround: split print products into with-borders / without-borders categories). Added page border-radius limitation: bleed and margin guides stay square, use page masks. Source: slack-message (#development), fireflies-call, loom-video, claude-chat.
- 2026-07-31: Documented AI restyle/filter auto-apply on selection (fixes filter loss when going to cart without pressing Apply). Added known issue: AI token usage counted globally instead of per site, fix verified on staging and pending production deploy. Documented that element substitutions now run on all admin previews and embedded inline pages (previously skipped unless `fulfillment=true`). Added known issue: no front/inside page indicator in the mobile card editor. Source: slack-message (#development), support ticket.


=================================================================
FILE: 18_ADMIN_NAVIGATION.md
=================================================================

# 18 — Admin Navigation

**Authority Scope:** Pixfizz Core admin interface sections and settings only.

_Last updated: 2026-06-01_

---

## Admin Overview

The Pixfizz Core admin is the central interface for managing your store, products, orders, and settings. Accessed at `{your-domain}/admin`.

---

## Admin Sidebar Sections

### Dashboard
Key business metrics: Gross Revenue, Orders Fulfilled, Average Order Value, Sessions, Conversion Rate. Quick links to OrderHub, community, help center.

The dashboard also surfaces active (in-progress) carts, giving visibility into carts in flight for account management and follow-up. Exact location and any configuration to be confirmed.

### Orders
- **Orders** — view/manage with status filters, CSV export, barcode search
    - **Design custom fields in the orderline CSV:** to output a design-level custom field as a column in the orderline CSV export, reference it with the nested path `custom:print_book:print_theme:<field-name>` (for example `custom:print_book:print_theme:primary_collection_path`). The standard orderline CSV cannot filter on these fields, but it can output them using this path.
    - **Money log:** the order detail page shows a payment **summary** only. The individual money log entries for that order live in Super Admin, reached with the **Show details** link on the summary. Go via Show details for per-transaction history, partial captures, or when a payment total on the order page looks wrong.
- **Abandoned Carts** — incomplete checkouts
- **Production Files** — production book files with project, page count, status
- **Projects** — end users' saved personalization projects

### Galleries
Public galleries where customers share and view photo projects.

### Users
Customer accounts and access management.

### Shipping
- **Shipping Services** — carrier and method configuration
- **Packaging** — package type definitions
- **Taxes** — tax rules
- **Addresses** — address management
- **Extra Fees** — additional fee configuration

### Marketing
- **Promocodes** — promotional code management
- **Gift Vouchers** — gift voucher / gift card issuing and tracking
    - **A voucher's value can be changed after the voucher has been created.** It is not fixed at issue. Updating the value is supported through the API, which makes it possible to top up or reset an existing voucher rather than issuing a replacement code. Useful for re-use-credit promotions where the customer already holds the code. TO CONFIRM: whether the value is also editable directly in the admin UI, and the exact API endpoint and payload.
    - Gift voucher emails are sent through the `email-shopper/templates/gift-voucher` template (see `52_SNIPPET_INVENTORY.md`).
    - Voucher codes can be **embedded in fulfillment output** — printed into the production/packaging files for an order — so an online-issued voucher can be tracked when it is redeemed in store. Implemented through the fulfillment template, the same mechanism as any other per-order dynamic value (see `31_FULFILLMENT_ENGINE.md`). TO CONFIRM: the exact Liquid accessor for the voucher code on the orderline.

### Products
- **Published Products** — published products list (everything in a Collection). Renamed from "All Products" on 2026-03-30 to make it clear the listing is scoped to published items only.
- **Product Attributes** — commercial product definitions. Prices are now **editable inline** directly from the product attributes list — click on any price field, type the new value (simple price or formula), press Enter for simple prices or click OK for multi-line formulas, Esc to cancel. No need to open each product individually.
- **Templates** — production specifications. Includes a bulk **Text Upgrade** action (shipped 2026-03-05) that applies text-box vertical alignment fixes across all templates in one step — use when migrating older templates that pre-date the current text rendering.
- **Collections** — product groupings for storefront
- **Fonts** / **Font Palettes** / **Color Palettes** — typography and color management
  - **Gotcha:** the default font palette tooltip implies fonts are auto-assigned, but fonts must be **manually assigned** to a palette. If a design shows fallback typography, check the palette assignment rather than assuming the admin auto-populated it.
- **Element Substitutions** — swap rules
- **Calendars** — calendar product configuration
- **Inventory tracking** — see the dedicated Inventory Management section below.

### Website (CMS)
Manages storefront content. What's visible depends on Shopper vs standalone CMS:

- **Pages** — CMS pages forming storefront URL structure (standalone CMS only — Shopper manages these automatically)
    - **Admin-only visibility:** individual pages and blog posts can be set to admin-only, so they are visible to logged-in admins but hidden from the public. Use this to stage and review content before its public release, then switch it on to publish. This is a publish gate, distinct from `d-none`, which only hides an element visually while leaving its links crawlable.
- **Layouts** — wrapper templates that pages render inside (standalone CMS only)
- **Snippets** — HTML/Liquid template fragments (building blocks of pages)
- **Custom Types** — dynamic content types for flexible page content
- **Assets** — file manager for images, fonts, media
- **Crawler** — website crawl management for sitemaps and product feeds. Accessed at `{domain}/admin/website_crawls`. Each crawl run lists every URL the crawler followed, with response status (200, 404, etc.) and the page that contained the link (shown in the rightmost column). Use this to track down broken internal links — the linking page column tells you exactly which snippet or page to fix.
    - **Sitemap:** generated at `{domain}/sitemap.xml`. Product feed at `{domain}/product-feed.json`. Both are platform-level features, not Shopper-specific.
    - **Gotcha:** the sitemap is **not** at `/site/sitemap`. Do not invent this URL. Confirm against the live admin.
    - **Gotcha (2026-02-02):** product XML feed URLs have shipped with an explicit `:80` port in the URL, making them unreachable for some consumers (Google Merchant Center rejected ~915 URLs on one site). If you see products missing from a feed with no obvious cause, inspect the raw XML for `:80` in the URLs before looking elsewhere.

> On Shopper sites, Pages and Layouts are pre-configured. You primarily work with Snippets, Custom Types, and Assets.

### Settings
- **General** — website title, timezone, language, currency, domain hosting, currency formatting
- **Email Notifications** — email templates for order lifecycle events (14 templates)
- **Design Tool** — Design Tool Configurations (branding, features, defaults)
- **Translations** — see the dedicated Translation Support section below.
- **Webhooks** — webhook configuration for external integrations (e.g. order.created, order.status_changed)
  - **Analytics pattern — server-side GA4 via webhook (Nicolas Restrepo, 2026-03-26):** client-side GA4 typically captures only ~10% of conversion events due to ad blockers, cookie consent drop-off, and tracking prevention. Piping `order.created` / `order.confirmed` through a Pixfizz webhook to a server-side GA4 endpoint (via a middleware like GTM Server, Stape, or a simple Cloud Function) lifts capture to ~50%+. Recommend this for any site where attribution accuracy matters for paid media decisions.

---

## Inventory Management

Pixfizz supports per-product inventory tracking, allowing you to monitor available stock and automatically reduce quantities when orders are placed.

### Enabling inventory tracking

Inventory is managed at the **product attribute level** — it is not active by default. To enable:
1. Open the Product Attribute in admin.
2. Enable the inventory tracking toggle.
3. Set the current inventory count (how many units are currently available).

### How stock is reduced

When an order includes a product that tracks inventory, the system automatically subtracts the purchased quantity **the first time** the order enters one of these statuses:
- **Confirmed (C)**
- **Draft (W)**

Stock is only reduced once — the first time the order hits either status. Subsequent status changes do not re-deduct.

### Negative stock and out-of-stock behavior

Inventory tracking **does not automatically block purchases**. If stock reaches 0, customers can still place an order — the inventory count will go negative. This gives operators flexibility but means you may want to add storefront controls.

### Liquid properties for storefront logic

Two Liquid properties are available on products for CMS-level stock management:

- `product.tracks_inventory` — boolean, true if inventory tracking is enabled for this product
- `product.current_inventory` — integer, the current stock count (can be negative)

Use these to:
- Display stock levels ("Only 3 left")
- Show "Out of stock" messaging
- Disable or hide the Add to Cart button when inventory reaches 0

Example logic pattern:
```
{% if product.tracks_inventory and product.current_inventory <= 0 %}
  <!-- show out of stock message, hide add-to-cart -->
{% endif %}
```

### Dynamic stock messaging

Pair inventory tracking with configurable product display names via Liquid (`50_LIQUID_REFERENCE.md`) to show threshold-based messages like "Only 3 left" directly in the product title or product card. The ability to block orders below a configured stock level is also available.

### Who is this for

Inventory management is designed for store owners who track physical stock or limited-quantity products — limited edition items, seasonal products, pre-produced goods, or physical items with fixed inventory.

---

## Built-in Translation Support

Pixfizz includes built-in translation support for core platform objects. This allows multi-language stores to manage translations directly in the admin and display the correct content automatically based on the user's language.

### Enabling multi-language support

1. Go to the **Super Admin**.
2. Enable **Multi-language support**.
3. Select the languages you want to support — to select multiple, hold **Command (Mac)** or **Ctrl (Windows)** while clicking.
4. Save.

Once enabled, translation options become available across supported objects.

### What can be translated

The following core objects support translation:

- **Products** — names and descriptions
- **Designs** — names, tags (layouts, backgrounds, clipart)
- **Collections** — names and descriptions
- **Templates** — names, page captions
- **Variant types and values** — the commercial options customers see on the storefront
- **Template option types and values** — production-level options

This covers both the storefront (CMS) and the design tool — customers see localized content end to end.

### Where to find translations in admin

When multi-language support is active, a **"Translate"** link appears in the top-left corner of supported objects. Clicking it opens the translation page where you can manage content for each enabled language.

### How translations are applied

Translations are automatically resolved based on the user's current language. In Liquid templates, standard object properties return the translated value automatically. For example:

```
{{ design.name }}
```

This displays the design name in the current language — no conditional logic needed.

### Bulk translation management

A major upgrade shipped 2026-03-30 added:
- Built-in translation support for **products and templates** (previously only pages and snippets were translatable).
- **Automatic translation key detection** during import.
- **Per-template and per-product-variant** translation handling.
- **Export / import** flow for bulk translation editing — export all translation keys to a file, translate externally, and re-import.

### What this is useful for

- Offer a fully localized experience across storefront and editor
- Manage translations in one place (Pixfizz admin) rather than external systems
- Ensure consistency across all touchpoints (product pages, design tool, options, captions)

---

## Super Admin

Cross-website management layer (for organizations managing multiple Pixfizz websites).

### Customer Super User Access
- **Websites** — manage all Pixfizz websites, configuration, feature flags, integrations
- **Super Users** — admin access across organization
- **Orders** — cross-website order aggregation and search
- **Fulfillment** — configure fulfillment destinations (FTP and HTTP endpoints)

### Website Management (Super Admin)
Per-website configuration includes:
- Core settings (name, subdomain, theme, language, currency)
- Feature flags enabling/disabling platform capabilities
- External integrations (Shopify, Etsy, Google OAuth, Dropbox, HappyAR, ReCAPTCHA)
- Website inheritance — share Design, Products, Tax, Email configurations across websites

> Some Super Admin features are managed by Pixfizz staff. The customer-facing view is deliberately focused.

---

### AI Tokens (Super Admin)
 
AI token access is **off by default on every website**. It is a Super Admin
feature flag, activated per website by a Pixfizz staff member. A site owner
cannot turn it on themselves, and it is **not** a fulfillment template setting.
 
- Setting: **Enable AI Tokens**, in the website's Super Admin configuration.
- Formerly labelled **Enable Perfectly Clear**. Renamed 2026-07-31, when billing
  moved from a Perfectly Clear-specific model to a generic AI token model that
  also covers OpenAI and Gemini.
- Until Pixfizz activates it on that website, any AI feature that consumes
  tokens is unavailable to the site regardless of other configuration.
- Enabling it is a request to Pixfizz, not self-service.
Token allowances, daily limits and per-lab billing behaviour are separate from
this flag — see `17_DESIGN_TOOL.md` for AI restyle limits. TO CONFIRM: the exact
Super Admin screen and field position, and whether the flag is per website or
per organization.
 
---

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-10: Added Published Products rename, Text Upgrade bulk action, inventory tracking with dynamic stock messaging, translation export/import upgrade, font palette tooltip gotcha, server-side GA4 via webhook pattern.
- 2026-04-22: Expanded Crawler entry with admin path, 404 reporting behaviour, and sitemap URL gotcha.
- 2026-05-19: Expanded Inventory Management into dedicated section with enable flow, stock reduction rules, negative stock behavior, Liquid properties (product.tracks_inventory, product.current_inventory), and out-of-stock CMS pattern. Expanded Translation Support into dedicated section with Super Admin enable flow, translatable objects list, Liquid auto-resolution, translate link location, and bulk export/import. Added inline price editing to Product Attributes. Source: Notion KB articles.
- 2026-06-01: Added active carts on the dashboard. Source: fireflies-call.
- 2026-06-15: Documented admin-only visibility for Pages/blog (pre-publish staging gate) and the design custom-field column path for the orderline CSV export (custom:print_book:print_theme:<field>). Source: slack-kb-sync (Matjaz, #development; design-field reporting work).
- 2026-08-05: Noted that the order detail page now shows a payment summary only, with individual money log entries reached through the Show details link into Super Admin. Source: slack-message (#development).
- 2026-08-11: Documented the Enable AI Tokens Super Admin feature flag — off by default, activated per website by Pixfizz staff, formerly "Enable Perfectly Clear". Clarifies that it is a Super Admin setting and not a fulfillment template field. Source: internal correction.
- 2026-08-14: Added Gift Vouchers under Marketing — the section `02_RETRIEVAL_MAP.md` already routed to but which did not exist in this file. Documented that a voucher's value can be updated after creation via the API, and that voucher codes can be printed into fulfillment output for in-store redemption tracking. Removed a stray closing code fence at end of file. Source: slack-message (#development, 2026-08-14), fireflies-call (2026-08-10).


=================================================================
FILE: 19_XML_TEMPLATE_REFERENCE.md
=================================================================

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

## Changelog
- 2026-04-03: Created from platform documentation provided by AdeB. Covers page parameters, safe area, growing spine, and layflat spread.
- 2026-04-03: Added PDF Layers section — layer attributes, separate-file, separate-page, per-page layer control, filename placeholders.
- 2026-04-03: Added Set Parameters section — count, grow, fulfillment, editor, preview.
- 2026-04-03: Added definition attributes, captions, sequential page types, and four annotated product examples (photo prints, canvas, photobook, greeting card).
- 2026-05-27: Added FTP Fulfillment Behavior section — FTP path prefix behavior (originals/ vs /originals/), _additional_files.json for sending original uploads to FTP, escape_json filter requirement for JSON job tickets, Job Tickets folder naming rule. Source: Fireflies calls, Slack #dev.
- 2026-06-15: Added Multi-Page Product Page-Count Rules — booklet page count must be divisible by 4; old softcover templates can carry a page-count ghost bug (mitigation: rebuild on a fresh template). Source: slack-kb-sync (booklet rules; softcover bug).


=================================================================
FILE: 20_SHOPPER_CART_RULES.md
=================================================================

# 20 — Shopper Cart Rules

**Authority Scope:** Cart behavior only.

_Last updated: 2026-02-26_

---

# 04 — Shopper Cart Rules

## Options visibility/editability
- Options visible unless image/file upload type.
- Editable in cart only if site-wide cart setting allows; otherwise change via project-edit/editor/photo prints UI.

## Hiding options and variants from the cart line

A boolean custom field `hide_from_cart` exists on both **variants** and **template options**. Where it is set, `product/px-option-cart` skips that entry when rendering the cart line. Use it for machine-set or internal values — tool settings, cut offsets, injected file references — that should not be shown to the shopper.

Two limitations apply, and both fail silently:

- **The editable cart branch ignores it.** When `admin/checklist/cart-editable-options` is `TRUE`, the cart renders options through `px-option-selector` / `px-option-selector-alt` rather than the loop, and those components do not honour `hide_from_cart`. A value correctly hidden in read-only mode reappears the moment a site turns on editable cart options.
- **Child orderlines are not filtered at all.** Options on child orderlines render unconditionally regardless of the field.

`hide_from_cart` must exist as a custom field on the specific site — custom fields do not inherit parent to child (see **13_TEMPLATE_BOUNDARIES**). Before adding a new "hide this from the cart" field, check whether `hide_from_cart` already covers the case; creating a duplicate field is a common and avoidable mistake.

## Photo prints
- Quantity not editable in cart.
- Quantity is per-photo in Photo Prints UI; orderline priced via `cut_print_quantity`.

## Pricing display
- Pricing generally visible; tiered pricing may show strikethrough.

## Digital-only
- No special cart behavior.

## Changelog
- 2026-07-28: Added hide_from_cart section covering the variant/template-option cart filter and its two silent limitations (editable-cart branch, child orderlines). Source: claude-chat.


=================================================================
FILE: 21_SHOPPER_CHECKOUT_POLICY.md
=================================================================

# 21 — Shopper Checkout Policy

**Authority Scope:** Checkout engine logic only.

_Last updated: 2026-06-26_

---

# 05 — Shopper Checkout Policy Engine

Shopper checkout resolves UI/flow by computing derived flags from cart orderlines and admin checklist toggles.

## Kiosk mode
- Alternate kiosk domain + helper snippet detection.
- Pay-in-store can be shown only in kiosk mode.
- Auto-logout after successful checkout.

## Shipping unavailable
If any orderline has:
- `orderline.product.custom.shipping == 'false'` OR
- `orderline.product.custom.shipping_unavailable`

## Pickup unavailable
If any orderline has:
- `orderline.product.custom.pickup_unavailable`

## Film processing
If not disabled by `admin/checklist/hide-film-order-checkout`, and any orderline has:
- `orderline.product.custom.film_processing`

## Digital-only detection (if enabled)
If `admin/checklist/digital-only-delivery == 'TRUE'` and cart non-empty:
- All orderlines must have `chosen_variants['version']`
- All must equal `digital-only`
Then checkout:
- Ignores shipping
- Applies hidden public/system address
- Shows digital delivery state

## Tax model: US-style checkout tax vs European VAT expectations

Pixfizz uses a **US-style tax-at-checkout model**: tax is calculated and added at checkout, not included in displayed prices.

For European stores, this creates a UX mismatch — customers expect VAT-inclusive pricing (the price shown is the price paid). This is a known platform-level structural difference.

Current workarounds:
- **VAT exemption** (e.g. cross-border orders to Switzerland): apply an automatic discount as a negative fee to remove the VAT that was added at checkout. The storefront price remains unchanged.
- **Postal code-based tax rules** are managed via a CSV file that maps postal code ranges or prefixes to tax rates. Complex postal code formats (multi-region countries like France) require range or prefix matching logic in the CSV.

### Tax CSV format

The tax CSV has a single header row, then one row per rule. Header:

	country,region/state,zipcode,city,tax%

Rules are matched against the shopper's shipping address. Specificity stacks:
- Country + state: applies anywhere in that state.
- Country + state + zip: applies when state and zip both match.
- Country + state + zip + city: applies when state, zip, and city all match.

This only applies when product prices are NOT tax-inclusive. If prices already
include tax, no rules are needed and the price shown is the price paid. When
prices exclude tax, the matched rate is added on top of the orderlines at cart and
checkout. A customer-facing help article exists ("Setting Up Taxes").

Flag this to clients during onboarding if they are European and expect tax-inclusive display prices.

## Guest checkout: configurable fields

Guest checkout supports the following configuration:
- **Billing address**: can be set as mandatory
- **Customer ID field**: optional — can be added as an optional field (relevant for in-store or lab-account workflows)
- **Registration toggle**: controls whether the guest is offered account registration at checkout

These are checkout configuration options, not hardcoded behaviour.

## Payment method selection and labels

**Which method is selected by default.** The default selected payment method is resolved in two layers:
- A URL parameter takes priority: `request.params['payment-method']`. If present and valid, it persists the customer's selection across page reloads.
- If no valid method is set via URL, the system falls back to the **first entry** in the `checkout/available-payment-methods` snippet.

So the effective default is determined by the **ordering** in `checkout/available-payment-methods`. To change the default, reorder that snippet - do not modify the `site/checkout` file.

The payment-method radio selector only renders when **more than one** method is enabled. With a single enabled method it is auto-selected and the selector is hidden.

**Renaming a payment method label.** Payment method display labels (e.g. "Cash on Delivery") are set via the admin **Translations** system, not a dedicated config field. To rename a method - for example "Cash on Delivery" to "In House Billing" - edit the translation string for that label. See `18_ADMIN_NAVIGATION.md` section Built-in Translation Support.

## Changelog
- 2026-02-26: Initial checkout policy content.
- 2026-04-23: Added tax model note (US-style vs European VAT, postal code CSV, automatic discount workaround). Added guest checkout configurable fields note.
- 2026-06-01: Added tax CSV format and matching specificity. Source: claude-chat/help-article.
- 2026-06-26: Added payment method selection (URL param priority, then first entry in checkout/available-payment-methods; radio only renders for 2+ methods) and label renaming via Translations. Source: claude-chat/fireflies-call.


=================================================================
FILE: 22_OPTION_VARIANT_RENDERING.md
=================================================================

# 22 — Option & Variant Rendering

**Authority Scope:** Variant rendering behavior only.

_Last updated: 2026-06-19_

---

# Option / Variant Types and how Shopper renders them

This doc explains **what option/variant “types” exist in Pixfizz** and **how the Shopper template renders them** (based on the `product/px-options` and `product/px-option-cart` snippets you shared).

> Terminology note: Pixfizz UI calls these “Variants”, but the Liquid/snippets often treat them as generic `options` and render them via the same machinery.

---

## 1) The core `option.type` values (admin dropdown)

From the admin UI, these are the supported types:

- **Multiple Choice**
- **Text**
- **Number**
- **Color**
- **Font**
- **Image Upload**
- **File Upload**

In Shopper Liquid, those appear as `option.type` values (e.g. `'text'`, `'number'`, `'color'`, `'font'`, `'image_upload'`, `'file_upload'`). **Multiple Choice** is the default “else” path (typically rendered as radio buttons / tiles) unless an `option.custom.selector` overrides it.

---

## 2) Shopper entry points: where options are rendered

Shopper commonly renders options in two places:

- **On product pages** (static + design products):
	- Design product flow often renders:
		- Template options: `options: design.template_options` with `parameter_name: 'template_options'`
		- Product variants: `options: product.variants` with `parameter_name: 'variants'`
	- Static product flow renders product variants only.

- **In cart** (cart line item editing / display):
	- Cart uses a cart-specific snippet (commonly `product/px-option-cart`) to render option controls more compactly.

---

## 3) Global display gating and nesting

### 3.1 Kiosk-only options
Options can be conditionally displayed based on kiosk mode:

- If `option.custom.kiosk_mode_only` is truthy:
	- Shopper checks kiosk mode (`helpers/is-kiosk-mode`)
	- Only displays the option if `is_kiosk_mode == 'TRUE'`

### 3.2 Triggered / child options (conditional logic)
Options can have children (`option.children`), and child options can be shown based on a trigger:

- Parent option may include `option.trigger_value`
- Child option rendering can include `trigger="{{ child_option.trigger_value.code }}"`

Shopper renders children recursively, e.g.:

```liquid
{% snippet 'product/px-options',
	options: option.children,
	chosen_options: chosen_options,
	parameter_name: parameter_name,
	design: design %}
```

---

## 4) The important “selector” overrides (`option.custom.selector`)

Even when `option.type` is the same, Shopper can render **very different UI** via `option.custom.selector`.

These are the selectors explicitly handled in the snippet you provided:

### 4.1 `textarea` (Text option rendered as textarea)
- Applies when `option.type == 'text'` and `option.custom.selector == 'textarea'`
- Renders `<textarea ... rows="4">`

### 4.2 `color` (Multiple choice rendered as swatches)
- Applies when `option.custom.selector == 'color'`
- Renders radio buttons with SVG swatch tiles
- Uses `value.custom.hex` for the swatch color

> Separate from `option.type == 'color'`, which can render either a palette picker (`option.color_palette`) or an `<input type="color">`.

### 4.3 `checkbox` (Multiple choice rendered as checkbox)
- Applies when `option.custom.selector == 'checkbox'`
- Uses the **first value** for checked/unchecked logic
- Renders `<input type="checkbox" ...>`

### 4.4 `dropdown` (Multiple choice rendered as select)
- Applies when `option.custom.selector == 'dropdown'`
- Renders `<select>...</select>`
- Can display:
	- `value.custom.price_label`, else
	- `value.price` formatted via `currency`
	- Supports negative pricing display (minus sign formatting)

### 4.5 `slider` (Multiple choice rendered as range input)
- Applies when `option.custom.selector == 'slider'`
- Renders `<input type="range" ...>`
- Uses a JS map of `value.code -> value.name` to show a friendly label

### 4.6 `quick-quantity` (Multiple choice rendered as per-value quantity inputs)
- Applies when `option.custom.selector == 'quick-quantity'` **and** `support_quick_quantity` is true
- Renders a grid of numeric inputs (one per `option.values`)
- Uses `data-parameter-name` and `data-value-code` so JS can transform them into the expected param structure

This selector is crucial for “matrix style” purchasing (e.g., multiple sizes/finishes at once) without forcing the customer to add multiple separate line items manually.

### 4.7 Text option input constraints (`min_length`, `max_length`, `pattern`)

Applies when `option.type == 'text'` (and `option.custom.selector` is not `textarea`).

The platform exposes three validation properties on the `OptionType` object that map directly to HTML input attributes:

| Liquid property | HTML attribute | Notes |
|---|---|---|
| `option_type.min_length` | `minlength` | `nil` if not set |
| `option_type.max_length` | `maxlength` | `nil` if not set |
| `option_type.pattern` | `pattern` | `nil` if not set; value is a regex string |

These are configured in the admin under the option type's settings (Max Length field + Pattern field). The Pattern field tooltip confirms it is applied as an HTML `pattern` attribute, which triggers native browser validation.

**Pattern format notes:**
- The value is a standard HTML pattern regex (anchored implicitly to the full input value by the browser).
- Do not wrap in `/` delimiters — the value is used directly as the `pattern` attribute string.
- Example allowing upper and lowercase letters, digits, whitespace, and common punctuation: `[A-Za-z0-9\s&,.']*`
- The original parent template default (uppercase only) is: `[A-Z0-9\s&,.']*`

**Rendering note:** Whether `min_length`, `max_length`, and `pattern` are passed through as attributes on the rendered `<input>` in cart context (`product/px-option-cart`) has not been confirmed — verify against the live snippet if this matters for a specific implementation.

### 4.8 `toggle` (2-value Multiple choice rendered as an animated switch)

- Applies when `option.custom.selector == 'toggle'` **and** `option.values.size == 2`.
- Renders two visually-hidden radio inputs (first value = off side, second value = on side) plus a CSS-only switch control. No JavaScript is used, so it survives AJAX re-injection without a `style onload` re-init.
- Radios always submit a value, so the off side is never an empty submission (a lone checkbox would submit nothing in its off state).
- The active state is driven entirely by `:checked ~` sibling rules in CSS:
	- knob slides via `transform: translateX(...)`
	- track border and knob recolour to the site primary
	- the active flanking label is emphasised
- ON-state colour comes from `{% snippet 'style/color-primary' %}` referenced inside `style/custom.css` (custom.css is Liquid-processed, so the snippet resolves there). Note the snippet is `style/color-primary`, not `style/primary-color`.
- Click-to-flip is achieved with two overlapping empty `<label>` click targets that swap `pointer-events` by state — no script needed.
- Each radio carries an `aria-label` from its value name, so accessibility is preserved even when visible labels are hidden.

**Fallback / guard:**
- If values != 2, the branch is skipped and the option falls through to the default radio rendering. It cannot break the form.
- Mark one of the two values as **default** in admin so the initial state is predictable.

**Optional bare switch (no flanking labels):**
- Boolean custom field `option.custom.toggle_hide_labels`: when set, adds the `px-toggle-no-labels` class to the wrapper, which hides the value names (`.px-toggle-name`) and shows just the switch.
- Test as a real boolean (`{% if option.custom.toggle_hide_labels %}`), not a string comparison.
- Custom fields are site-specific and do not inherit parent → child, so set this on the site where the option lives.

**Pricing display note:** unlike `dropdown`, this selector does not show price deltas next to the values by default. If a toggle value carries a price, append it to the value name in the branch (or use `dropdown`).

**Where it lives:** `toggle` is a branch added to the Shopper snippet `product/px-options` (template layer). Add it on the parent (`shopper24`) to make it available everywhere, or override `product/px-options` on a single site to scope it. The CSS belongs in that site's `style/custom.css`.

---

## 5) Upload-specific behaviors

### 5.1 `image_upload` (`option.type == 'image_upload'`)
Shopper renders a `<px-image-upload>` web component with:
- sources: `local galleries qr` (varies; multi-upload group uses `local qr`)
- optional crop: `crop-aspect-ratio="{{ option.crop_aspect_ratio }}"`
- optional DPI: `minimum-dpi="{{ design.template.minimum_dpi }}"`
- optional accept: `accept="{{ option.custom.accept }}"`
- optional “no element substitutions”: `data-px-no-element-substitutions`
- optional “no pricing”: `data-px-no-pricing`
- optional image adjustments driven by admin checklist:
	- `enable-image-filters-image-upload`
	- `enable-image-color-image-upload`

### 5.2 `file_upload` (`option.type == 'file_upload'`)
Shopper renders a `<px-file-upload>` component.
- Default accept is `image/*,.pdf` unless overridden.
- Can display existing uploaded file name / URL.
- **Programmatic injection:** `px-file-upload` exposes a real
  `input[type=file]`, so a script can inject a File via a `DataTransfer`
  object (`input.files = dt.files; input.dispatchEvent(new Event('change'))`).
  `px-image-upload` does **not** — it is built for interactive gallery/QR
  selection, exposes only a hidden `input[type=hidden]` and an "Upload"
  button, and creates its file input lazily inside the dialog. To attach a
  script-generated file to an option, use `file_upload`, not `image_upload`.
- **Read-only vs hidden:** for a script-injected upload the option must be
  `hidden: true` (invisible to the customer) with `read_only: false`. A
  `read_only: true` upload renders a hidden value input plus a read-back chip
  with no file input to inject into.
### 5.2b Targeting a specific upload by option code
Every option is wrapped in `<px-option code="...">`. When a product has more
than one upload option, scope any DOM query to the wrapper —
`document.querySelector('px-option[code="X"] px-file-upload')` — rather than
grabbing the first upload on the page. When a code is given and no match is
found, return null rather than falling back to the first upload, or an
injected file lands in the wrong option.

### 5.2c Input names differ between the product page and project-edit
The same variant or option is submitted under a different input name depending
on which page the shopper is on:

- **Product page:** `variants[<code>]`
- **Project-edit:** `book[options][<code>]`

Any script that reads or writes an option value must therefore **suffix-match**
the input name (`name.endsWith('[' + code + ']')`) rather than matching the full
string. An exact match on `variants[<code>]` works on the product page and
silently finds nothing on project-edit. The user-visible symptom is a saved
project opening with its settings reset when the customer edits it.

Hidden inputs count. A read-back routine that only inspects checked radios and
selects will miss values that project-edit renders as `input[type=hidden]`.

### 5.3 Multi-upload groups (`option.custom.multi_upload_group`)
This is a key advanced behavior.

If `option.custom.multi_upload_group` is set, Shopper:
- Opens a wrapper `<px-multi-image-upload ...>`
- Groups multiple upload options under a single “Upload Photos” experience
- Starts the wrapper when the group changes vs `previous_option.custom.multi_upload_group`
- Closes the wrapper when the group changes vs `next_option.custom.multi_upload_group`

This allows “upload many images” flows while still storing each uploaded image against a distinct option code.

---

## 6) Pricing and substitutions flags (important for rendering + behavior)

Shopper passes these flags into many inputs/components:

- `option.has_element_substitutions`
	- If false: `data-px-no-element-substitutions`
- `option.has_pricing`
	- If false: `data-px-no-pricing`

Meaning: **an option can exist purely for substitutions**, purely for pricing, both, or neither, and Shopper can explicitly tell Px components to ignore certain behaviors.

---

## 7) Cart rendering differences (`product/px-option-cart`)

In cart context, the rendering is simplified:

- `option.type == 'text'` becomes `<input type="text">`
- Otherwise, defaults to `<select>` for values
- Value price display in the `<option>` label:
	- e.g. `{{ value.price | currency }}` if non-zero
- Still supports nested options (children) via recursion:
	- `option.children` + `trigger` relationship

> Note: the `toggle` selector is implemented in `product/px-options` (product-page context). It is not wired into `product/px-option-cart`, where a 2-value option falls back to the default `<select>`. Add it there separately if a toggle is needed in cart.

---

## 8) Practical “recognize and document” list (what matters when reading a site)

When you see a variant/option behaving “special” in Shopper, check these first:

- `option.type` (text/number/color/font/image_upload/file_upload vs multiple choice default)
- `option.custom.selector` (textarea, color swatch, checkbox, dropdown, slider, quick-quantity, toggle)
- `option.custom.toggle_hide_labels` (bare switch with no flanking labels, toggle selector only)
- `option.custom.multi_upload_group` (multi-image wrapper)
- `option.custom.kiosk_mode_only` (kiosk-only display)
- `option.custom.hidden` (completely hidden)
- `option.trigger_value` + `option.children` (conditional display / nesting)
- `option.custom.accept` (upload constraints)
- Admin checklist toggles affecting image upload adjustments
- `option.custom.custom_script` (inline scripts—dangerous but real)

That’s the set that needs to be “muscle memory” when debugging option rendering in Shopper.

---

## Changelog
- 2026-06-19: Added section 4.8 `toggle` selector (2-value animated CSS-only switch on `product/px-options`), including the `toggle_hide_labels` bare-switch option, guard/fallback behavior, and primary-colour sourcing. Added cart-context note (7) that toggle is product-page only. Added `toggle` and `toggle_hide_labels` to the recognize-and-document list (8).
- 2026-07-28: Added 5.2c — option input names differ between the product page (`variants[code]`) and project-edit (`book[options][code]`); scripts must suffix-match and must handle hidden inputs. Source: claude-chat.


=================================================================
FILE: 23_XML_CALENDAR_REFERENCE.md
=================================================================

# 23 — XML Calendar & Planner Template Reference

**Authority Scope:** XML definition structure for calendar and planner products — date sequence vocabulary, set parameters, and annotated real-world examples. Platform-level — not Shopper-specific.

_Last updated: 2026-06-30_

---

## What this file covers

Calendar and planner products use an extended XML vocabulary beyond the standard page parameters documented in `19_XML_TEMPLATE_REFERENCE.md`. This file covers:

- `<definition>` root attributes
- `<set>` parameters
- The dates system — full element reference
- `<foreachdate>` — iteration over date sequences
- Named date sequence patterns (month grid, weekly, daily, etc.)
- Three annotated real-world examples

For standard page parameters (`width`, `height`, `bleed`, `margin`, `snap`, `hinge`, `gutter`, `output-name`), see `19_XML_TEMPLATE_REFERENCE.md`.

---

## 1. `<definition>` Attributes

The root element of every XML template definition.

| Attribute | Description |
|---|---|
| `unit` | Unit system for all dimension values. Common values: `inch`, `mm`. |
| `dpi` | Target resolution for production output. |
| `output` | Output file format. Common value: `pdf`. |
| `minimum-dpi` | Minimum acceptable image resolution. Used for image quality warnings in the Design Tool. |
| `add` | Minimum number of pages a user can add to the project. |
| `max` | Maximum total page count for the product. |

**Sample:**
```xml
<definition unit="inch" dpi="200" output="pdf" minimum-dpi="36" add="2" max="241">
```

---

## 2. `<set>` Parameters

A `<set>` groups one or more `<page>` elements that are treated as a unit. Sets can be repeated via `foreachdate`.

| Attribute | Values | Description |
|---|---|---|
| `count` | `false` / omit | `count="false"` excludes this set from the page count defined in `<definition>`. Commonly used for covers. Omit to include in count. |
| `grow` | `true` / omit | `grow="true"` marks this set as the one used when the end user adds new pages to a project. Typically used for interior pages in a book or planner. Only one set should have this. |
| `fulfillment` | `false` / omit | `fulfillment="false"` excludes this set from production artwork generation. Omit to include. |
| `editor` | `false` / omit | `editor="false"` hides this set from the end user in the Design Tool. Omit to show normally. |
| `preview` | `true` / omit | `preview="true"` designates this set as the project preview — the thumbnail shown to users in cart and saved projects. Not to be confused with a design preview. |
| `foreachdate` | Named date sequence | Repeats the set once for each date in the named sequence. See section 5. |

**Sample — hidden preview set:**
```xml
<set fulfillment="false" editor="false" preview="true">
	<page type="preview" bleed="0" width="10" height="10" />
</set>
```

---

## 3. Page Attributes Specific to Calendars

Two `<page>` attributes are specific to calendar/planner products:

| Attribute | Description |
|---|---|
| `dates` | Space-separated list of named date sequences to make available on this page. The design tool uses these to populate calendar grid elements. |
| `autofill` | **Legacy — no longer does anything. Ignore if seen in existing templates.** |

**Sample:**
```xml
<page type="calendar-month" dates="calendar-month previous-calendar-month next-calendar-month" bleed="0.125" width="11.25" height="8.75"/>
```

---

## 4. Dates Vocabulary

Date sequences are defined at the top of the `<definition>` block using a dedicated vocabulary. They are named and then referenced by pages and `foreachdate` iterations.

### `<dates name="...">`
Defines a named date sequence. The sequence is built by the child elements inside it.

### `<date />`
Adds a single date to the sequence. Accepts offset attributes: `day`, `month`, `year`.

```xml
<date day="+1" />
```

### `<defdate>`
Calculates a date and stores it under a name for later reference. Does not add it to the sequence — it is a named anchor only.

| Attribute | Description |
|---|---|
| `name` | The reference name to store the date under. |
| `day` | Day offset (e.g. `-1`, `+41`). |
| `month` | Month offset (e.g. `-1`, `+1`). |
| `year` | Year offset (e.g. `+1`). |
| `weekday` | Jump to the nearest occurrence of a weekday: `SU`, `MO`, `TU`, `WE`, `TH`, `FR`, `SA`. |

```xml
<defdate name="last-day-of-previous-month" day="-1" />
```

### `<dateshift>`
Shifts the current date context before executing child elements. All child date generation happens relative to the shifted position.

| Attribute | Description |
|---|---|
| `day` | Shift by N days. |
| `month` | Shift by N months. |
| `year` | Shift by N years. |
| `weekday` | Shift to the nearest occurrence of a weekday (`SU`, `MO`, etc.). |
| `to` | Jump to a previously stored `defdate` reference name. |

```xml
<dateshift to="sunday-of-first-week">
	<dategen freq="daily" count="42" />
</dateshift>
```

### `<dategen>`
Generates a sequence of dates and adds them to the current sequence.

| Attribute | Description |
|---|---|
| `freq` | Frequency: `daily`, `weekly`, `monthly`. |
| `count` | Number of dates to generate. |
| `until` | Generate dates up to and including this named `defdate` reference. |
| `interval` | Step interval (e.g. `interval="2"` = every 2 days). Default: `1`. |
| `tag` | Marks generated dates with a label. Used to identify out-of-range dates for styling. |

```xml
<dategen freq="daily" count="42" />
<dategen freq="daily" until="last-day-of-previous-month" tag="out-of-range" />
```

### `<deldategen>`
Removes dates from the current sequence. Accepts the same attributes as `<dategen>`.

### `<datesfrom name="...">`
Reuses another named date sequence inline. Avoids duplication when multiple sequences share the same base logic.

```xml
<dates name="calendar-year-month3">
	<dateshift month="3" day="1">
		<datesfrom name="calendar-month"/>
	</dateshift>
</dates>
```

### Tags
The `tag` attribute on `<dategen>` marks dates with a label. The most common use is `out-of-range` or `out-of-current-month` — marking dates that fall outside the current month so the design tool can style them differently (e.g. greyed out).

---

## 5. `<foreachdate>`

Two forms — both iterate over a named date sequence.

### As a `<set>` attribute
Repeats the set once per date in the sequence.

```xml
<set foreachdate="by-month">
	<page type="image" bleed="0.125" width="11.25" height="8.75"/>
	<page type="calendar-month" dates="calendar-month" bleed="0.125" width="11.25" height="8.75"/>
</set>
```

### As a wrapping block
Repeats everything inside it (multiple sets) once per date in the sequence. Used when a repeating unit contains more than one set.

```xml
<foreachdate name="by-month-3">
	<set>
		<page type="monthly-left" dates="calendar-month" width="7.4375" height="10.5625" />
		<page type="monthly-right" dates="calendar-month" width="7.4375" height="10.5625" />
	</set>
	<set foreachdate="by-day-monthly">
		<page type="daily-left" dates="calendar-day" width="7.4375" height="10.5625"/>
		<page type="daily-right" dates="calendar-day" width="7.4375" height="10.5625"/>
	</set>
</foreachdate>
```

`<foreachdate>` blocks can be nested — the inner `foreachdate="by-day-monthly"` on the set above generates a daily spread for every day within each iterated month.

---

## 6. Named Date Sequence Patterns

### Monthly grid — Sunday start
Generates 42 dates (6 weeks) starting from the Sunday of the first week. Marks dates outside the current month with `out-of-range`.

```xml
<dates name="calendar-month">
	<dateshift day="-6">
		<defdate weekday="SU" name="sunday-of-first-week"/>
	</dateshift>
	<dateshift to="sunday-of-first-week">
		<dategen freq="daily" count="42" />
	</dateshift>
	<defdate day="-1" name="last-day-of-previous-month"/>
	<dateshift to="sunday-of-first-week">
		<defdate day="+41" name="last-generated-date"/>
	</dateshift>
	<dateshift to="sunday-of-first-week">
		<dategen freq="daily" until="last-day-of-previous-month" tag="out-of-range"/>
	</dateshift>
	<dateshift month="+1">
		<dategen freq="daily" until="last-generated-date" tag="out-of-range"/>
	</dateshift>
</dates>
```

### Monthly grid — Monday start
Same logic but anchors to `weekday="MO"` instead of `weekday="SU"`.

```xml
<dates name="calendar-month">
	<defdate weekday="MO" name="monday-of-first-week"/>
	<dateshift to="monday-of-first-week">
		<dategen freq="daily" count="42" />
	</dateshift>
	<defdate day="-1" name="last-day-of-previous-month"/>
	<dateshift to="monday-of-first-week">
		<defdate day="+41" name="last-generated-date"/>
	</dateshift>
	<dateshift to="monday-of-first-week">
		<dategen freq="daily" until="last-day-of-previous-month" tag="out-of-range"/>
	</dateshift>
	<dateshift month="+1">
		<dategen freq="daily" until="last-generated-date" tag="out-of-range"/>
	</dateshift>
</dates>
```

### 12-month iteration sequence
Used with `foreachdate` to repeat a set or block once per month across 12 months.

```xml
<dates name="by-month">
	<dateshift day="1">
		<dategen freq="monthly" count="12" />
	</dateshift>
</dates>
```

### Previous and next month grids
Used when a calendar page displays the previous and next month in small mini-calendars alongside the current month.

```xml
<dates name="previous-calendar-month">
	<defdate name="end-of-pre-previous-month" month="-1" day="-1" />
	<defdate name="beginning-of-current-month" />
	<dateshift month="-1" day="-6">
		<dateshift weekday="SU">
			<defdate name="final-date" day="+37" />
			<dategen freq="daily" until="final-date" />
			<dategen freq="daily" until="end-of-pre-previous-month" tag="out-of-current-month" />
			<dateshift to="beginning-of-current-month">
				<dategen freq="daily" until="final-date" tag="out-of-current-month" />
			</dateshift>
		</dateshift>
	</dateshift>
</dates>

<dates name="next-calendar-month">
	<defdate name="end-of-current-month" month="+1" day="-1" />
	<defdate name="beginning-of-next-next-month" month="+2" />
	<dateshift month="+1" day="-6">
		<dateshift weekday="SU">
			<defdate name="final-date" day="+37" />
			<dategen freq="daily" until="final-date" />
			<dategen freq="daily" until="end-of-current-month" tag="out-of-current-month" />
			<dateshift to="beginning-of-next-next-month">
				<dategen freq="daily" until="final-date" tag="out-of-current-month" />
			</dateshift>
		</dateshift>
	</dateshift>
</dates>
```

### Year-at-a-glance — 24 named month sequences
For products that display many months simultaneously. Each named sequence reuses `calendar-month` via `<datesfrom>`, shifted to the target month.

```xml
<dates name="year-at-a-glance">
	<date/>
</dates>
<dates name="calendar-year-month1">
	<dateshift month="1" day="1">
		<datesfrom name="calendar-month"/>
	</dateshift>
</dates>
<!-- ... repeat for month2 through month12 ... -->
<!-- For a second year, use year="+1": -->
<dates name="calendar-year-month13">
	<dateshift year="+1" month="1" day="1">
		<datesfrom name="calendar-month"/>
	</dateshift>
</dates>
```

The `year-at-a-glance` sequence is a single date (`<date/>`). When used with `<set foreachdate="year-at-a-glance">`, it causes the set to run exactly once. The page inside it receives all 12 (or 24) month sequences via its `dates` attribute.

---

## 7. Annotated Examples

### Example A — Standard 12-month wall calendar

The most common calendar type. Cover + 12 × (image page + calendar grid page) + back cover.

```xml
<definition output="pdf" unit="inch" dpi="300">

	<dates name="calendar-month">
		<!-- Sunday-start monthly grid — see section 6 -->
	</dates>

	<dates name="by-month">
		<dateshift day="1">
			<dategen freq="monthly" count="12" />
		</dateshift>
	</dates>

	<dates name="previous-calendar-month">
		<!-- See section 6 -->
	</dates>

	<dates name="next-calendar-month">
		<!-- See section 6 -->
	</dates>

	<set>
		<page type="cover" bleed="0.125" width="11.25" height="8.75"/>
	</set>

	<set foreachdate="by-month">
		<page type="image" bleed="0.125" width="11.25" height="8.75"/>
		<page type="calendar-month" dates="calendar-month previous-calendar-month next-calendar-month" bleed="0.125" width="11.25" height="8.75"/>
	</set>

	<set>
		<page type="back-cover" bleed="0.125" width="11.25" height="8.75"/>
	</set>

</definition>
```

**Key points:**
- No `count="false"` on the cover set — cover is counted here.
- `by-month` generates 12 dates so the `foreachdate` set repeats 12 times, producing 24 interior pages total.
- `previous-calendar-month` and `next-calendar-month` give the calendar page mini-calendars for the surrounding months.

---

### Example B — Year-at-a-glance product

A single-page product displaying all 12 months of a year simultaneously.

```xml
<definition unit="inch" dpi="300" output="pdf">

	<dates name="year-at-a-glance">
		<date/>
	</dates>

	<!-- 12 named month sequences, each reusing calendar-month via datesfrom -->
	<dates name="calendar-year-month1">
		<dateshift month="1" day="1"><datesfrom name="calendar-month"/></dateshift>
	</dates>
	<!-- ... month2 through month12 ... -->

	<dates name="calendar-month">
		<!-- Sunday-start monthly grid — see section 6 -->
	</dates>

	<layers>
		<layer name="artwork" visibility="on"/>
		<layer name="background" visibility="on"/>
		<layer name="photo" visibility="on"/>
	</layers>

	<!-- Hidden preview set — not fulfilled, not shown in editor -->
	<set fulfillment="false" editor="false" preview="true">
		<page type="preview" bleed="0" width="10" height="10" />
	</set>

	<set foreachdate="year-at-a-glance">
		<page type="product"
			dates="calendar-year-month1 calendar-year-month2 ... calendar-year-month12"
			bleed="0" width="11" height="8.5" />
	</set>

</definition>
```

**Key points:**
- `year-at-a-glance` is a single date — `foreachdate` runs the set exactly once.
- All 12 month sequences are passed to the single page via the `dates` attribute.
- The hidden preview set uses `fulfillment="false" editor="false" preview="true"` — a common pattern for products that need a custom project thumbnail without it appearing in the editor or production output.
- Layers defined here are assigned to elements in the admin design tool.

---

### Example C — 3-month planner (complex nested structure)

A planner product with fixed front/back matter, a year-at-a-glance spread, and a 3-month repeating block containing monthly and daily pages.

```xml
<definition unit="inch" dpi="200" output="pdf" minimum-dpi="36" add="2" max="241">

	<!-- Date sequences: year-at-a-glance, 24 named month sequences,
	     calendar-month, previous/next month, by-month-3,
	     by-day-monthly, calendar-day — see section 6 -->

	<!-- Cover — not counted -->
	<set count="false">
		<page type="cover" width="10.3" height="12" />
	</set>

	<!-- Fixed front matter -->
	<set>
		<page type="profile" width="7.4375" height="10.5625" />
	</set>
	<set>
		<page type="logo" width="7.4375" height="10.5625"/>
		<page type="intro" width="7.4375" height="10.5625"/>
	</set>

	<!-- Year-at-a-glance — runs once, receives all 24 month sequences -->
	<set foreachdate="year-at-a-glance">
		<page type="holidays2" width="7.4375" height="10.5625" />
		<page type="at-a-glance"
			dates="calendar-year-month1 ... calendar-year-month24"
			width="7.4375" height="10.5625" />
	</set>

	<!-- Growable contacts section -->
	<set grow="true">
		<page type="contacts-left" width="7.4375" height="10.5625" />
		<page type="contacts-right" width="7.4375" height="10.5625" />
	</set>

	<!-- Vision board spread -->
	<set>
		<page type="vision-board-left" width="7.4375" height="10.5625" bleed="0.125" margin="0.125" snap="0,0.125,0.125"/>
		<page type="vision-board-right" width="7.4375" height="10.5625" bleed="0.125" margin="0.125" snap="0,0.125,0.125"/>
	</set>

	<!-- 3-month repeating block — repeats 3 times -->
	<foreachdate name="by-month-3">

		<set>
			<page type="intentions-left" width="7.4375" height="10.5625"/>
			<page type="intentions-right" width="7.4375" height="10.5625"/>
		</set>

		<set>
			<page type="monthly-left" dates="calendar-month previous-calendar-month next-calendar-month" width="7.4375" height="10.5625" />
			<page type="monthly-right" dates="calendar-month zodiac-quote-date" width="7.4375" height="10.5625" />
		</set>

		<set>
			<page type="health-left" width="7.4375" height="10.5625" />
			<page type="health-right" width="7.4375" height="10.5625" />
		</set>

		<!-- Daily spreads — one set per day in the month -->
		<set foreachdate="by-day-monthly">
			<page type="daily-left" dates="calendar-day" bleed="0" width="7.4375" height="10.5625"/>
			<page type="daily-right" dates="calendar-day" bleed="0" width="7.4375" height="10.5625"/>
		</set>

	</foreachdate>

	<!-- Fixed back matter -->
	<set><page type="passwords-left" width="7.4375" height="10.5625" /><page type="passwords-right" width="7.4375" height="10.5625" /></set>
	<set><page type="mood-left" width="7.4375" height="10.5625" /><page type="mood-right" width="7.4375" height="10.5625" /></set>
	<set><page type="notes-left" width="7.4375" height="10.5625" /><page type="notes-right" width="7.4375" height="10.5625" /></set>

</definition>
```

**Key points:**
- `<foreachdate name="by-month-3">` wraps multiple sets — all repeat 3 times.
- `foreachdate="by-day-monthly"` is nested inside the outer `foreachdate` block. It generates a daily spread for every day of each of the 3 months.
- `grow="true"` on the contacts set allows users to add more contact pages.
- `count="false"` on the cover excludes it from the page count.
- The 6-month and 12-month planner variants in the original definition are commented out — they represent alternative structures (6-month daily, 12-month weekly) that can be swapped in by uncommenting and commenting out the 3-month block.

---

## 8. Legacy Attributes

The following attributes appear in older template definitions and should be ignored — they no longer do anything:

| Attribute | Notes |
|---|---|
| `autofill` | Was intended to auto-populate image zones. No longer functional. |
| `reorder` | No longer functional. |

---

## Calendar Transformation Triggers

Calendar transformations (the admin configuration that maps calendar dates to outputs/designs) can be triggered on:

- a specific date or date range, and
- **a specific week number in the year** (added June 2026).

Week-number triggering fires a transformation on a given week (for example week 1 or week 52) rather than a fixed calendar date.

---

## Reaching Elements Hidden Behind a Calendar Grid

Calendar and planner pages stack a lot of elements in the same place, and a grid
or overlay element will often sit on top of the element that actually needs
editing, making it unselectable in the design tool.

The fix is the layer mechanism already defined for the template. Layers can be
toggled off temporarily in the admin design tool purely as an editing aid —
hiding the blocking layer exposes the element underneath, and visibility is
restored afterwards. Layer visibility toggled this way is an editing convenience
only and does not change what is fulfilled; production output follows the
`visibility` and `separate-file` attributes declared in the `<layers>` block.
See `19_XML_TEMPLATE_REFERENCE.md` § PDF Layers for the attribute reference.

**After editing, refulfill manually.** Editing a calendar element on an order
that has already generated production files does **not** regenerate those files.
The order must be force-refulfilled from the admin order detail page, and the
existing generated file has to be deleted before a new one can be requested
(see `40_PLAYBOOK_UPDATED.md` § Production File Regeneration). Skipping this
step means the lab prints the pre-edit version with nothing to indicate the
edit was ever made.

## Changelog
- 2026-04-03: Created from platform documentation and annotated real-world examples provided by AdeB. Covers definition attributes, set parameters, full dates vocabulary, foreachdate, named sequence patterns, and three annotated examples.
- 2026-06-30: Documented week-number triggering for calendar transformations. Source: notion-dashboard (2026-06-22).
- 2026-08-14: Added the layer-visibility technique for reaching calendar elements blocked by an overlapping grid, and the rule that a manual refulfill is required after any such edit before production files reflect it. Source: fireflies-call (2026-08-12).


=================================================================
FILE: 30_PRICING_ENGINE.md
=================================================================

# 30 — Pricing Engine

**Authority Scope:** Ruby pricing formulas and price variables only.

_Last updated: 2026-07-31_

---

# 06 — Pricing Formulas (Ruby) & Price Variables (Locked Guidance)

## What pricing formulas are
- Ruby expressions evaluated in context.
- Must return a numeric price.
- Used for product pricing and option/variant pricing.

## Where to paste a pricing formula
Pricing formulas are set on the **Product Attribute** in the admin:
- Go to **Products** → open the product → open the **Product Attribute** (the specific size/variant, e.g. "4x6 Glossy")
- Paste the formula into the **Pricing Formula** field on that attribute
- If the same formula applies to multiple sizes, paste it on each Product Attribute individually (prices per size typically differ, but the formula structure may be shared)

## Price Variables
- Admin-defined numeric constants available by name in formulas.
- Used to centralize pricing inputs across products/options.
- `whitelabel` is a **Price Variable** (not a system variable).

## Keep formulas basic
Use established patterns; avoid “optimized” Ruby.

## Canonical patterns (examples)
### Linear
`0.85 * cut_print_quantity`

### Tiered unit price × quantity
`{1..10=>0.59, 11..49=>0.49, 50..249=>0.44, 250..499=>0.39, 500..2000=>0.35}.find { |range,unit_price| range.include?(cut_print_quantity) }.last * cut_print_quantity`

### Base + incremental pages
`19.99 + ((pages - uncounted_pages) - 16) / 2.0 * 0.50`
or
`19.99 + extra_pages / 2.0 * 0.50`

**Critical:** The `pages` variable counts ALL pages in the project, including pages
in XML sets with `count="false"` (covers, preview pages, etc.). For book pricing
that charges per interior page, always use `(pages - uncounted_pages)` to exclude
uncounted pages from the calculation. Using raw `pages` will overcharge customers.

Use `2.0` (not `2`) to force float division and avoid Ruby integer rounding.

RATIONALE: `pages` includes uncounted XML pages (covers, previews). Without subtracting `uncounted_pages`, book pricing formulas overcharge. Confirmed in client support case, May 2026.
SOURCE: "Pixfizz customer support agent" chat, May 3 2026
SOURCE TYPE: claude-chat

### Volume discount lookup (unit price)
`{1..1=>8.00, 2..5=>6.50, 6..10=>6.00, 11..20=>5.75, 21..30=>5.60, 31..50=>4.80, 51..1000=>4.15}.find { |range,unit_price| range.include?(units)}.last`

### Volume discount + per-page pricing (per unit)
`{1..1=>(19.99 + (pages-16)/2 * 0.50), 2..5=>(18.99 + (pages-16)/2 * 0.45), 6..10=>(17.99 + (pages-16)/2 * 0.40)}.find { |range,unit_price| range.include?(units)}.last`

### Threshold + blocks of pages
`49 + ([0, pages - 50].max / 4)`
or
`49 + ([0, pages - 50].max / 4)*2.5`

### Sheet-based production pricing
`10 + (((pages - 12.0) / 12.0).ceil) * 8`

## Photo Prints — Hard Rule

- `cut_print_quantity` is only valid for Photo Prints.
- Must always be used for cut prints.
- Must never be used for non-cut-print products.

### Enforcement Logic

IF product is Photo Print → use `cut_print_quantity`
ELSE → use `quantity` or `units`

No exceptions.

### First unit + each additional (total price formula)
`(25 + ([0, quantity - 1].max * 15.0)) / quantity`

Use when the first unit has a higher base price and each additional unit costs less.
The `/ quantity` is required because Pixfizz multiplies the returned value by quantity —
the formula must return a **per-unit price**, not a total.
Use `15.0` (not `15`) to force float division and avoid Ruby integer rounding errors.

---

## `pages` Includes Uncounted Pages — Use `(pages - uncounted_pages)` for Books

The `pages` variable in pricing formulas counts **every** page in the project,
including pages in XML template sets marked with `count="false"` (typically covers,
preview thumbnails, and other non-interior pages).

For any book or multi-page product where pricing is based on interior page count,
always use `(pages - uncounted_pages)` instead of raw `pages`.

```ruby
# WRONG — includes covers/preview pages in the count
74.90 + (pages - 42) / 2.0 * 1.90

# CORRECT — excludes uncounted pages
74.90 + ((pages - uncounted_pages) - 42) / 2.0 * 1.90
```

Using raw `pages` causes the formula to calculate more "extra" pages than the
customer actually added, resulting in overcharging.

This applies to all page-based pricing patterns: base + incremental, threshold +
blocks, sheet-based, and volume discount + per-page.

RATIONALE: High-impact pricing gotcha. Raw `pages` silently includes uncounted XML pages, causing overcharging on book products. Not previously documented.
SOURCE: "Pixfizz customer support agent" chat, May 3 2026
SOURCE TYPE: claude-chat

#### Variable choice: `quantity` vs `units`
- `quantity` — quantity within a single orderline only. Use when pricing is per-orderline.
- `units` — total units across all orderlines in the cart for this product. Use when
  tiered or stepped pricing should accumulate across multiple separate orderlines.

For the yard signs case: using `quantity` charges $25 for the first sign per orderline.
If a customer adds 1 sign, goes back, and adds another as a second orderline, they would
be charged $25 twice. Using `units` would recognise 2 total signs across the cart and
apply the $15 additional rate to the second. Choose based on the intended pricing behaviour.

---

## Worked Example — Unified Base + Variant Adjustment + Quantity Breaks

Source: client implementation, 2026-04-10. First documented Pixfizz pattern that combines variant
price adjustments **and** quantity breaks under a single base price.

### Model
- One base price per product (e.g. a photo restoration service).
- "Color correction" is the **default** variant.
- "No color correction" applies a **negative variant adjustment** (≈30% off base).
- Quantity break tier discounts apply **only on the non-corrected path** (to protect
  labour margin on the corrected path).
- Customer-facing tier pricing is shown on the product page as a simple HTML table
  with a tooltip — no dynamic formula display.

### Why this pattern
It keeps a single base price in the admin (simple to manage, simple to report on)
while still letting a single product serve two distinct price ladders depending on
which labour-level variant the customer picks.

---

## Negative Extras / Discount Variants — Workaround

The platform does **not** currently support negative extra fees or negative variant
adjustments directly. Attempts to set a negative extra value will not price correctly.

### Variant Formula Editor — Leading Minus Sign Error

The variant formula editor does **not** accept a leading minus sign (`-`). Entering a formula that starts with `-` causes an editor error and the formula cannot be saved.

**Confirmed workaround:** multiply by `-1` at the end of the formula instead.

```ruby
# This errors:
-(base_price * tier_1)

# This works:
(base_price * tier_1) * -1
```

Confirmed on client cut print pricing, April 2026. The `* -1` pattern is logically equivalent and the editor accepts it.

RATIONALE: Platform editor limitation with a non-obvious workaround. Affects any negative variant adjustment using cut print formulas.
SOURCE: "Claude pricing formula generation" chat, April 23

### Canonical workaround (2026-04-08)
1. **Lower the base price** of the product to the discounted value.
2. **Add a positive surcharge** to the "standard" variant so the net price of the
   standard path matches the original intended base.
3. Customers who pick the "discounted" path (e.g. delayed service, no correction)
   land at the genuinely lower base price with no extra applied.

This inverts the mental model — the discounted option becomes the default — but it
is the only reliable way to get a discount-variant effect today.

### Use cases seen in production
- Rush fee **+20%** — rush is the surcharge variant, standard turnaround is the base.
- Delay discount **−10%** — delayed turnaround is the base, standard is the surcharge.

When documenting these for a client, always label the variants in customer-facing
terms ("Standard / Rush" or "Standard / Delayed — save 10%") — do not expose the
inverted base-price mechanics.

---

## Photo Enhancement Variants — Charge Once Per Photo, Not Per Quantity

Recurring support issue: photo enhancement add-ons (e.g. "enhance this image") must
charge once per image, not multiplied by the print quantity the customer orders.
The standard Pixfizz behaviour of multiplying the option price by `quantity` is
wrong for this case.

The formula must explicitly flatten the quantity multiplier — return a per-unit
value that divides out the quantity the engine will re-multiply by. Confirm the
exact snippet against the live site before reusing; the pattern is known but the
canonical formula has not yet been locked in to this reference.

---

## Test Orders with 100% Voucher — Billing Cap Removed

**Platform rule change, 2026-03-24.**

Previously, test orders paid with a 100% voucher were still billed at 50% of the
order value internally (a safety cap on the billing file). That cap has been
removed. Orders paid with a 100% voucher now report as fully zeroed in the
billing file.

Impact: any site using voucher codes for internal QA or for comped client orders
will now see those orders at **$0** in billing reports rather than at 50% of the
catalogue price. Update any reconciliation logic that assumed the old cap.

---

## Automatic Discounts (Liquid-Based Cart Discounts)

**Feature type:** Platform-level. Configured in Main Admin.

Automatic Discounts apply cart-level discounts without requiring a promo code. The discount is calculated using a **Liquid formula** that has access to the full cart and user context, and the result appears automatically at checkout.

Reference: [https://help.pixfizz.com/triage/automatic-discounts](https://help.pixfizz.com/triage/automatic-discounts)

### How it works

- The formula is a Liquid template that must return a **numeric discount amount** (not a percentage — the output is the actual value to subtract).
- The formula has access to `cart`, `user`, `orderlines_total`, and other standard Liquid objects.
- The discount appears automatically in the cart/checkout — no customer action required.
- Multiple automatic discounts can be active simultaneously.

### Available context variables

- `cart.orderlines_total` — subtotal before discounts
- `cart.promocode_code` — the applied promo code (blank if none)
- `cart.orderlines` — the orderlines collection
- `user.category` — user category label (e.g. "VIP", "Wholesale")
- `user.orders_count` — number of completed orders (if available on the user object — confirm with Matjaz)
- Standard Liquid filters: `date`, math operators, etc.

### Canonical patterns

**Tiered cart discount (spend more, save more):**

```liquid
{%- if cart.orderlines_total >= 250 %}
    orderlines_total * 0.20
{%- elsif cart.orderlines_total >= 150 %}
    orderlines_total * 0.15
{%- elsif cart.orderlines_total >= 75 %}
    orderlines_total * 0.10
{%- endif %}
```

**User category conditional discount with promo code guard:**

```liquid
{% if user.category == 'VIP' and cart.promocode_code == blank %}
    orderlines_total * 0.10
{% endif %}
```

The promo code guard (`cart.promocode_code == blank`) prevents stacking a category discount with a manual promo code. Whether to include this guard depends on the business intent.

> **Property-name trap:** the property is `cart.promocode_code`, not `cart.promocode`. `cart.promocode` (without `_code`) is not a valid Cart property and resolves to nil, which is falsy. A guard written as `{%- unless cart.promocode -%}` therefore never blocks, and the discount fires even when a promo code is applied. Always use `cart.promocode_code`.

**Seasonal / time-based discount:**

```liquid
{%- assign current_month = 'now' | date: '%m' | plus: 0 -%}
{%- if current_month == 1 or current_month == 2 -%}
    orderlines_total * 0.15
{%- endif -%}
```

Runs automatically during slow months, turns itself off when the month changes.

### Key rules

- The formula must return a numeric value. If the formula returns nothing (no branch matches), no discount is applied.
- The discount is an **amount**, not a percentage — the formula does the percentage math itself.
- Automatic discounts are separate from promo codes and extra fees. They are a distinct discount mechanism.
- Admin location: confirm exact admin path with Matjaz (likely under Discounts or Pricing in Main Admin).

---

## Extra Fees (Liquid-Based Cart Fees)

**Feature type:** Platform-level. Configured in Main Admin under **Shipping → Extra Fees**.

Extra Fees are the fee-side twin of Automatic Discounts. Each fee can be driven by a
**Liquid formula** with full cart context, and the result is **added** to the order
(Automatic Discounts subtract; Extra Fees add). This is the supported way to add a
conditional surcharge — the platform does not support negative discounts or negative
variant adjustments to achieve the same effect (see "Negative Extras / Discount
Variants — Workaround" above).

Confirmed in production use for minimum-order-value fees and extra shipping / oversize
surcharges.

### How it works

- Each Extra Fee has a **Code** and **Name** (for example `rush` / "Rush Fee",
  `oversize` / "Oversize shipping charge") plus a Liquid formula.
- The formula returns a **numeric amount** — the actual fee value to add, in the
  site's currency (not a percentage).
- Same cart / user context as Automatic Discounts: `cart.orderlines`,
  `cart.orderlines_total`, `cart.promocode_code`, `user.*`, and standard Liquid filters.
- Multiple Extra Fees can be active at once; each is evaluated independently.
- If the formula returns nothing (no branch matches, or empty output), no fee is added
  — mirrors the Automatic Discounts "return nothing = nothing applied" behaviour.

### Canonical use cases

- **Minimum order fee:** add a flat handling fee when the cart subtotal is below a threshold.
- **Extra shipping / oversize surcharge:** add a fixed amount when the cart contains an oversized product.

### Pattern — Per-Duplicate-Orderline Surcharge (count distinct lines)

Use case: charge a flat amount for each **additional** order line of the same product
beyond the first (for example, multiple separate cut-print lines of the same size).
Documented from a photo lab client implementation.

The reusable technique is a **seen-string + `contains`** idiom to count distinct
products across `cart.orderlines`. The first line of a given product is the original
(no charge); every later line of the same product adds the per-duplicate amount.

```liquid
{%- assign per_duplicate = 1 -%}
{%- assign fee = 0 -%}
{%- assign seen = '' -%}
{%- for orderline in cart.orderlines -%}
	{%- if orderline.is_cut_print -%}
		{%- assign token = '|' | append: orderline.product.id | append: '|' -%}
		{%- if seen contains token -%}
			{%- assign fee = fee | plus: per_duplicate -%}
		{%- else -%}
			{%- assign seen = seen | append: token -%}
		{%- endif -%}
	{%- endif -%}
{%- endfor -%}
{%- if fee > 0 -%}{{ fee }}{%- endif -%}
```

- Five cut-print lines of one product returns 4. Add three lines of a second product
  and it returns 6 (each product is counted independently).
- **Grouping key:** `orderline.product.id` groups by product. Correct only when each
  print size is its own Product. If size is a variant on a single product, every size
  shares one product ID — include the chosen size variant in the `token` to keep sizes
  distinct.
- Drop the `orderline.is_cut_print` guard to apply the same duplicate-line logic to any
  product type, not just cut prints.

### Key rules

- The formula returns an **amount to add**, in the site currency, as a raw number (no
  `currency` filter on the output — the engine formats it).
- **Verify on first test order (pending confirmation with Matjaz):** that `cart.orderlines`
  is the correct loop handle inside an Extra Fee formula, and that a bare `{{ number }}`
  output is read as the fee amount. Both are inferred from Automatic Discounts behaviour;
  confirm before relying on the orderline-iteration pattern in production.
- Admin location: **Shipping → Extra Fees** (confirmed via admin screenshot).

---

## Price Variable Bulk Export / Import

**Status: shipped (2026-07-28).** Previously on the roadmap as planned/not-yet-shipped; this is now live.

Price Variables can be exported and imported, including across sites — export to a spreadsheet, edit in bulk, and re-import rather than editing them one at a time in the admin. This is useful for onboarding scoping involving hundreds of price variables, and for replicating a pricing setup from one site to another.

Source: slack-message (#development), commit e954d0b3.

---
## Packaging Tab — Do Not Use

The **Packaging** tab exists in the product admin but is a legacy feature — almost no live clients
use it, and it is not recommended for new setups.

**Why:** In mixed-product orders, per-package prices aggregate unpredictably (multiple products go
into one package, but the formula can't know what else is in the order at pricing time). The result
is often an expensive and confusing checkout experience.

**Recommended approach:** Always use **shipping pricing formulas** instead of the Packaging tab.
If a client asks about Packaging, redirect them to shipping formula configuration.

RATIONALE: Repeat signal — Loom video in #kb-sync + #development question same week.
SOURCE TYPE: loom-video + slack-message
---
## Changelog
- 2026-05-19: Added Automatic Discounts section — Liquid-based cart discounts with tiered, user category, and seasonal patterns. Source: Claude chat (webinar prep).
- 2026-07-03: Added Extra Fees (Liquid-Based Cart Fees) section — fee-side twin of Automatic Discounts (adds instead of subtracts), configured under Shipping → Extra Fees. Includes per-duplicate-orderline surcharge pattern (seen-string + contains idiom); orderline-iteration specifics pending live confirmation. Source: Claude chat.
- 2026-07-20: Added property-name trap — `cart.promocode` (without `_code`) is nil and silently defeats promo-code guards; always use `cart.promocode_code`. Source: claude-chat.
- 2026-07-31: Price Variable Bulk Export/Import shipped (2026-07-28) — moved out of Roadmap, now documented as a live feature. Source: slack-message (#development).
- 2026-08-21: Documented Packaging tab as legacy feature; shipping formulas recommended instead. Source: loom-video + slack-message.


=================================================================
FILE: 31_FULFILLMENT_ENGINE.md
=================================================================

# 31 — Fulfillment Engine

**Authority Scope:** Job ticket schema and generated file logic only.

_Last updated: 2026-07-31_

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
- **Enable Perfectly Clear** (image enhancement) — the underlying billing fields for this were renamed from Perfectly Clear-specific naming to a generic "AI tokens" model as of 2026-07-30 (enhancement can now be billed against OpenAI/Gemini as well as Perfectly Clear). Exact new admin label not yet confirmed — treat "Perfectly Clear" references in admin as potentially stale until verified.
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

**UI lock.** Once a fulfillment code is set on a template, the product-level fulfillment code field for products using that template is locked and cannot be changed from the product screen. This is the expected consequence of the precedence order above: the template code always wins, so editing the product-level value would have no effect. To change routing for such a product, edit (or clear) the code on the template. Added June/July 2026.

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
- **Cut print filenames use an index counter, not page numbers.** For cut print output, the page number is no longer included in the generated filename; an index counter (`idx`) is used instead to keep filenames unique across the set. If a lab previously relied on page numbers appearing in cut print filenames, expect the sequential `idx` value in that position now. Source: fireflies-call (2026-07-09).

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
- 2026-07-11: Documented UI lock — the product-level fulfillment code field is locked when the template carries its own fulfillment code (extends the code resolution/precedence note). Source: slack-message (#development, commit 2026-07-05).
- 2026-07-11: Noted cut print filenames now use an index counter (`idx`) instead of page numbers for uniqueness. Source: fireflies-call (2026-07-09).
- 2026-07-31: Renamed "Enable Perfectly Clear" billing field to "Enable AI Tokens" — confirms shift from Perfectly Clear-specific billing to a generic AI token model (OpenAI/Gemini). Source: slack-message (#development), commit 8aeec021.


=================================================================
FILE: 32_ORDER_LIFECYCLE.md
=================================================================

# 32 — Order Lifecycle & Production Pipeline

**Authority Scope:** OrderHub order lifecycle, production pipeline, fulfillment destinations, and OrderHub Desktop.

_Last updated: 2026-05-19_

---

## What is OrderHub?

OrderHub is the Workflow Layer of Pixfizz. It manages everything after a customer places an order — routing orders to production, generating artwork, delivering production assets, and tracking fulfillment.

Handles orders regardless of origin: online storefront, Shopify, kiosk, in-store counter, or external platform via API.

---

## Order Lifecycle Statuses

Every order moves through defined statuses:

1. **Pending** — order received, awaiting confirmation
2. **Draft** — order saved but not yet finalized
3. **Confirmed** — order confirmed, ready for production
4. **Downloaded** — production assets downloaded
5. **Manufactured** — production complete
6. **Shipped** — order dispatched to customer
7. **Fulfilled** — order delivered and complete

Exception statuses: **Payment Failed**, **Error**, **Canceled**, **Refunded**.

### Pending orders do not auto-route to production

An order sitting in **Pending** does not flow to production on its own. Nothing downstream fires — no artwork generation, no fulfillment delivery, no OrderHub job — until the order is moved to **Confirmed**.

Where an order lands as Pending (manual payment methods, gateway not configured, payment awaiting capture), someone must **manually confirm it in admin** before production sees it. This is the most common cause of "the order is in Pixfizz but nothing reached the lab".

Check the Pending queue as part of daily order review on any site that accepts manual or offline payment.

### Gateway callback webhooks must be configured on the gateway side

Configuring a payment gateway in Pixfizz admin is only half the setup. Most
gateways will not tell Pixfizz the outcome of a payment unless a callback
webhook is registered in the gateway's own dashboard, pointing back at the
Pixfizz callback endpoint for that site.

Where that webhook is missing, the customer pays successfully and is debited,
but Pixfizz never receives the result. The order stays in **Pending** and never
routes to production. Nothing errors on the storefront, so this presents as a
Pixfizz fault when it is a gateway configuration gap.

**Diagnosing.** Search the server logs for requests to the site's gateway
callback endpoint. No requests at all means the webhook is either not set up,
or set up against the wrong URL. Test payments that debit correctly but leave
the order Pending are the same signal.

**PayU (confirmed).** In the PayU admin under **Developers > Webhooks**, create
two webhooks, one for **Payments > Successful** and one for **Payments >
Failed**. Both point at the same URL:

```
https://<site-domain>/cart/payu_money_callback
```

Treat "is the gateway-side webhook registered" as the first check on any
gateway where payments succeed but orders stay Pending, before looking at
Pixfizz configuration.

### Status codes in Liquid

In Liquid templates, `order.status` returns a **single-letter code**, not the display label. Confirmed codes:
- `P` = Pending
- `F` = Payment Failed

Write conditionals against the letter, e.g. `{% if order.status == 'P' %}`, and fetch the newest pending order with `user.orders | where: 'status', 'P' | sort: 'created_at' | reverse | first`.

**Pay Now / payment retry.** Orders in Pending (`P`) or Payment Failed (`F`) can be re-paid using the `order_payment` form: `{% form 'order_payment', order: order %}`. This is used to surface a "Pay Now" action on the account orders page and on the empty-cart page for back-from-gateway recovery. (Form tag confirmed in use on a live site; verify the tag name against the current template before reusing.)

Each status transition can trigger an email notification (configured in admin: Settings > Email Notifications).

---

## Order Origination

Orders can enter Pixfizz through multiple paths:
- **Pixfizz storefront** — orders placed on a Full Pixfizz site
- **Shopify** — ingested via webhooks
- **External platforms** — submitted via Pixfizz API
- **Marketplaces** — from connected marketplaces (e.g. Etsy)
- **Kiosk / In-store** — orders at physical retail locations

### Kiosk Terminal Tracking

`custom.kiosk_id` is a **whitelisted custom order property** — it is preserved through checkout
and accessible on the order record in admin.

Kiosk sites can tag orders by terminal by appending `?terminal=N` to any page URL:

https://yoursite.pixfizz.com/shop?terminal=1 https://yoursite.pixfizz.com/shop?terminal=2


The terminal value is captured in the order source, enabling per-terminal order routing and
reporting in OrderHub. In use on production kiosk sites (confirmed Aug 2026).

SOURCE: #development Slack, Richard + Alex, 2026-08-18–21.

All orders follow the same lifecycle and production pipeline regardless of origin.

---

## Production Pipeline

Once an order is confirmed, it enters the production pipeline:

1. **Order confirmation** — order finalized, payment captured
2. **Artwork generation** — Pixfizz renders personalized design into production-ready artwork (based on Template specs: DPI, dimensions, format)
3. **Fulfillment transformation** — transformations defined on Template or Design are applied (color profile conversion, bleed adjustments, etc.)
4. **Asset delivery** — production files delivered to configured fulfillment destination via FTP or HTTP
5. **Job creation** — OrderHub job created with all production details
6. **Production download** — production team downloads via OrderHub Desktop
7. **Status updates** — order progresses: Downloaded → Manufactured → Shipped → Fulfilled

---

## Custom Order & Orderline Fields via Liquid Scripts

Pixfizz supports configurable Liquid scripts (set in the Super Admin) that automatically populate custom fields on orders and orderlines at the moment an order is created from a cart. This removes the need for manual data entry on orders.

### What scripts can do

- Set **custom fields on the order** (order-level script)
- Set **custom fields on individual orderlines** (orderline-level script)

### Script execution order

When an order is created, scripts run in this fixed sequence:

1. Auto-confirm script
2. Priority script
3. **Order-level custom field script**
4. **Orderline-level custom field script**

Custom field scripts run **after core processing**, so all standard order data (pricing, user, address, etc.) is available to the script at execution time.

### Script return format

Each script must return a **JSON dictionary**:

```json
{
  "field_name": "value",
  "another_field": "value"
}
```

Each key is a custom field name. Each value is what gets saved.

### Merge behavior

- The values generated by the script are **merged** with any existing custom fields on the order or orderline.
- Existing fields are preserved unless the script explicitly overwrites them with a new value for the same key.

### Use cases

- Automatically tagging orders with metadata (e.g. source channel, campaign)
- Storing calculated values at order time (e.g. margin, weight estimate)
- Passing data to external systems or fulfillment workflows
- Adding per-item attributes to orderlines (e.g. processing instructions, routing codes)

### Who configures this

This is an **advanced / developer feature**. Scripts are configured in the **Super Admin** — not in the per-site admin. It requires familiarity with Liquid and JSON structure. Contact Pixfizz support if unsure.

---

## Order Cancellation and Transaction Fees

When cancelling orders, marking an order as "Canceled" in the admin may not automatically prevent Pixfizz transaction fees from being applied.

### Cancellation process

1. **Collect order codes** — gather the order codes for all orders that need to be cancelled.
2. **Email finance@pixfizz.com** — send the list of order codes to the Pixfizz finance team. This ensures the cancellations are processed correctly and fees are waived.
3. **Mark as canceled in admin** — update the order status in your admin. This is for your own record-keeping, but alone it does not guarantee fee removal.
4. **Follow up** — confirm with Pixfizz that the cancellations have been processed and no charges will apply.
5. **Check for duplicates** — verify that cancelled orders are not also duplicated (e.g. from a customer re-ordering after a failed attempt). Duplicate charges can occur if both the original and replacement order remain active.

> Simply marking an order as cancelled in admin is **not sufficient** to prevent transaction fees. You must also notify finance@pixfizz.com.

---

## Fulfillment Destinations

Define where production assets are delivered. Configured in Super Admin.

### FTP Delivery
- FTP Host, User, Password
- FTP Directory
- Filename Template (using Liquid variables from order/orderline context)
- Directory Template (folder structure on FTP server)

### HTTP Delivery
- WebService URL
- Content Type (e.g. application/json)
- Parameter Name and Fixed Parameters

### Common Output Settings (both delivery types)
- Format (PDF, JPEG, etc.)
- Color Profile (sRGB, CMYK, etc.)
- Single Page Output — each page as separate file
- Multiple Cut Print Copies — generate copies for cut-print workflows

### Filename Template
Uses Liquid-style variables to dynamically name production files. Example:
`{{ orderline.product.code }}.{{ order.code }}.{{ orderline.barcode }}_{{ page_output_name }}.{{ format }}`

---

## Order Management in Admin

### Orders List
- Order Code (PREFIX-ALPHANUMERIC pattern)
- Customer email, date, total, orderline count, status
- Filter by status, search by code/barcode, CSV export

### Order Detail
- Order ID, Code, Status, Date, Total, Paid status
- Shipping Service
- User Notes and internal Notes
- Priority flag
- Customer info (email, name, telephone, address)
- Job Ticket link
- Order History (status change log)
- Force-Refulfill option

### Orderlines
Each order has one or more orderlines (individual products). Each shows:
- Generated Files (downloadable production assets)
- Product options selected by customer
- Custom Fields
- Fulfillment Code (determines which destination receives assets)

### Other Admin Order Sections
- **Abandoned Carts** — incomplete checkouts
- **Production Files** — production book files with project, page count, status
- **Projects** — saved personalization projects
- **Cross-Website Order Management** — Super Admin aggregates orders from all websites

---

## OrderHub Desktop (OHD)

Desktop application for photo lab operators. Runs locally, polls OrderHub for new jobs, downloads print files, and feeds them to local print controllers.

### Key Capabilities
- Continuous polling for new jobs flagged for local production
- Downloads job details and print files from OrderHub via API
- Organises files into structured folders with human-readable naming
- Generates **DPOF files** for compatible print controllers (Epson, Noritsu, etc.)
- **AI-powered upscaling** for low-resolution images
- Channel and product routing to match jobs to the correct print workflow
- Job review tools: colour correction, quantity management
- Updates job status back in OrderHub once processed

### OHD Workflow
1. OrderHub creates a production job (on order confirmation + artwork generation)
2. Assets delivered to fulfillment destination (FTP/HTTP)
3. Job appears in OHD queue
4. OHD polls, downloads job and production assets to local machine
5. Team processes job; OHD updates status (Downloaded → Manufactured)
6. Status change flows back to OrderHub, triggers next lifecycle step

### Multi-instance Behaviour
If multiple OHD instances run across workstations, job delivery is **first-come-first-served** — a job goes to only one instance. Instances can be filtered by location.

### FTP Folder Fulfillment Mode

OHD can connect directly to a local FTP folder and move fulfillment files to local lab
infrastructure — useful for labs that do not have Syncovery installed.

**Configuration:** OHD Settings → connect to FTP folder → set the folder name to the lab's
hotfolder name (e.g. "Labworks"). Do **not** name the folder "OrderHub".

**Use case:** Labworks hotfolder integration for labs without Syncovery (e.g. Diversified Prints
setup). Files are moved to the local hotfolder directly from OHD rather than being pulled via Syncovery.

SOURCE: #development Slack, Richard, 2026-08-17.

### API Status Update Endpoint
```
POST /functions/v1/update-job-status
Headers: X-API-Key: <api_key>
```

> OHD is a companion tool to OrderHub — not standalone. Requires active OrderHub connection. For full operational detail see `45_ORDERHUB.md`.

---
## Rush / Urgent Order Options

The platform supports a rush/urgent order flag compatible with OrderHub file-naming conventions.
Tested and confirmed on production sites (Aug 2026) — ready for broad rollout.

- Configure via order settings in admin
- Rush orders are correctly identified by OrderHub for priority processing
- The `cart.rush_production` custom field (boolean) is set at checkout when the shopper selects rush

SOURCE: Fireflies call (Documentation, Aug 14 2026).
---
## Automatic Discounts

Pixfizz supports **automatic discounts** applied as negative line values on orders. Key behavior:

- Discounts appear as **negative values on order lines** (not as a separate discount field)
- Automatic discounts **stack with promo codes** — both can be applied to the same order
- Applied at the checkout page
- Configured in admin — no customer action required to apply them

This differs from promocode-based discounts, which require the customer to enter a code. Automatic discounts trigger without any input.

---

## Pending Order Email — Scope to Manual Payment Orders

When sending reminder emails for orders stuck in `pending` status, the trigger must be scoped to orders where the `manual_payment` custom field equals `true`.

Reason: without this condition, the email would also fire for failed credit card attempts — where the customer has likely already given up or re-ordered. A "please pay" email arriving a week after a failed card attempt creates a confusing experience.

Before sending:
1. Check `order_custom_manual_payment == true`
2. Check the customer has not placed a newer confirmed order — if they have, skip the reminder

The `manual_payment` field is a custom field set at order creation. It returns `true` for orders placed using manual payment methods (e.g. bank transfer, pay-in-store). Confirmed in a production implementation (April 2026).

---

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-23: Added pending order email scoping rule for manual_payment field.
- 2026-05-19: Added Custom Order & Orderline Fields via Liquid Scripts section (from Notion KB). Added Order Cancellation and Transaction Fees process (from Notion KB).
- 2026-05-21: Added Automatic Discounts (negative order line values, stack with promo codes, applied at checkout). Source: Fireflies.
- 2026-05-21: Expanded OrderHub Desktop (OHD) section with DPOF generation, AI upscaling, multi-instance behaviour, and API endpoint. Added cross-reference to 45_ORDERHUB.md. Source: OrderHub help modal.
- 2026-06-26: Documented order.status single-letter Liquid codes (P=Pending, F=Payment Failed) and the order_payment form for Pay Now / payment retry. Source: claude-chat.
- 2026-07-25: Clarified that Pending orders do not automatically route to production and must be manually confirmed in admin before artwork generation, fulfillment delivery, or OrderHub job creation occurs. Source: fireflies-call.
- 2026-08-05: Documented gateway callback webhooks as a required gateway-side configuration step, covering the failure mode where payment succeeds but the order stays Pending. Added the confirmed PayU setup (two webhooks, Successful and Failed, both to `/cart/payu_money_callback`) and the log-based diagnosis. Source: slack-message (#development), fireflies-call.
- 2026-08-21: Added OHD FTP folder fulfillment mode. Source: slack-message (#development, Richard, Aug 17).
- 2026-08-21: Added kiosk terminal tracking (custom.kiosk_id + ?terminal=N URL param). Source: slack-message (#development).
- 2026-08-21: Added rush/urgent order options section. Source: fireflies-call (Documentation call, Aug 14).


=================================================================
FILE: 40_PLAYBOOK_UPDATED.md
=================================================================

# 40 --- Playbook

**Authority Scope:** Operational reasoning and troubleshooting guidance.

*Last updated: 2026-05-19*

------------------------------------------------------------------------

# Pixfizz CMS + Shopper Playbook

## Guest Checkout UX Pitfall

Customers frequently assume guest checkout creates a login account.

Important truths:

-   Guest checkout creates a **guest user record**
-   Guest users **cannot login**
-   Guest users **cannot reset passwords**
-   Admins must **merge the guest user into a registered user** if login
    access is required

Recommended UX pattern:

Explain clearly:

> Placed an order as a guest?\
> Guest checkout does not create a login account.\
> You can create one using the same email address.

------------------------------------------------------------------------

## Rendering Project Images in Email

Email templates run **outside the storefront session**.

Because of this, project previews must use the **share code**.

Correct pattern:

``` liquid
https://{{ website.hostname }}{{ orderline.project | preview_url: height: 300, share: orderline.project.share_code }}
```

Without the `share:` parameter the preview may fail to load.

------------------------------------------------------------------------

## Orderline Image Fallback Pattern

Email order summaries should use this pattern:

``` liquid
{% if orderline.project %}
project preview
{% else %}
product image
{% endif %}
```

Reason:

-   Design products → project preview
-   Static products → product image

This ensures all cart items render correctly in email notifications.

------------------------------------------------------------------------

## Debugging Principle

When troubleshooting Pixfizz storefront issues:

1.  Determine if the behavior is **CMS**, **Shopper**, or
    **site-specific**.
2.  Verify object truth (cart, orderlines, variants).
3.  Check snippet rendering logic.
4.  Only then debug JS behavior.

This prevents chasing UI bugs caused by data-layer misunderstandings.

------------------------------------------------------------------------

## `style onload` Pitfall — Placement Outside `<script>` is Mandatory

When using the `{% capture %}` + `<style onload>` pattern, the blocks
**must sit outside any `<script>` tag**.

Placing a `{% capture %}` block or a `<style>` tag inside a `<script>`
block causes a JS syntax error and silently breaks all JavaScript on the
page — including unrelated logic like country switching, postcode lookup,
and unit number validation.

Symptom: everything JS-dependent on the page stops working at once.

Correct structure:

``` liquid
{% comment %} OUTSIDE the script tag: {% endcomment %}
{% capture my_init %}
(function() { ... })();
{% endcapture %}
<style onload="{{ my_init | escape }}"></style>

<script>
document.addEventListener('DOMContentLoaded', function() {	
{% comment %} Other once-only logic here {% endcomment %}
});
</script>
```

------------------------------------------------------------------------

## JS Not Re-Running After Delivery Option Switch

Symptom: a snippet renders correctly on first load but stops working
(dropdowns not populated, fields not shown/hidden) after the user
switches between pickup and delivery options, without a manual browser
refresh.

Cause: the checkout page uses `async: true` with a selector that
re-injects a section of the DOM. `<script>` tags inside re-injected
HTML do not re-execute.

Fix: use the `style onload` pattern (see `01_CODE_GOVERNANCE`) with a
direct init call at the end of the IIFE. Do not rely on
`DOMContentLoaded` or `px.fragmentsReloaded` event listeners alone —
they are insufficient for this case.

## `style onload` Firing Once Then Stopping — `onload = null` Anti-Pattern

Symptom: a snippet works correctly on first page load but stops responding after delivery option switches or AJAX re-injection — identical to the missing `style onload` problem.

Cause: the `style onload` wrapper was written as:
```liquid
{% capture init_script %}
(styleElement => {
	styleElement.onload = null;
	// ... logic ...
})(this);
{% endcapture %}
<style onload="{{ init_script | escape }}"></style>
```

Setting `styleElement.onload = null` inside the handler cancels all future firings after the first. The browser treats it as a one-shot event.

Fix: remove the arrow function wrapper and `onload = null` entirely. Use a plain IIFE instead:
```liquid
{% capture init_script %}
(function() {
	// ... logic ...
})();
{% endcapture %}
<style onload="{{ init_script | escape }}"></style>
```

## Nested Forms Pitfall — Checkout Page

The Shopper checkout page wraps everything in an outer `cart_update` form. Any `address_create` or `address_update` form nested inside it is silently dropped by the browser — the browser only processes the outermost form on submit, so Save buttons inside nested address forms are unresponsive with no visible error.

Symptom: an address form Save button does nothing; no network request fires.

Fix: replace inline address forms with redirect links to standalone address pages (`account-address-new`, `account-address-edit`, or a Custom Type instance such as `checkout-address-edit`). Do not attempt to embed address forms inside the checkout page.

------------------------------------------------------------------------

## Editor Iframe — CSS Isolation

The editor runs inside an iframe. Storefront CSS delivered via the
`style/custom.css` page does NOT reach inside it, so targeting `.px-header`,
`.px-cart-button`, or other `px-*` classes from storefront CSS has no effect.

Editor styling IS possible, just not from storefront CSS. Use one of:
- the Custom CSS field in the Design Tool Configuration, or
- the `editor.css` page (Full Pixfizz / Shopper), or
- the `shopify/custom-styles` snippet (Shopify integration).

See `17_DESIGN_TOOL.md` "Editor CSS Customization" for techniques. Note that
JS-driven label/content changes inside the editor still cannot be overridden with
CSS alone.

------------------------------------------------------------------------

### Identifying if a Site Uses the Shopper Template

Child sites that inherit from Shopper do not expose the CMS pages list — you cannot identify Shopper by looking for wildcard pages.

The correct method: click the ellipsis icon (`···`) at the bottom of the Pixfizz admin sidebar. In the modal that appears, check the "Source Websites" section. If it lists `shopper24.pixfizz.com` with label `cms`, the site is a Shopper child site.
---

## CSS Snippet Logs — First Diagnostic Step for Styling Regressions

CSS snippet change logs are available in the admin and record every change made to a snippet. When a client reports an unexpected styling regression (fonts changed, layout broken, colours wrong), reviewing the CSS snippet log is the correct first diagnostic step — not guessing or inspecting code.

Minor CSS errors (a misplaced `{`, a missing `;`) can cause all subsequent rules in the stylesheet to be silently ignored, producing site-wide styling failures that appear unrelated to the actual error location.

---

## Password Reset: Deprecated Liquid Templates Cause Broken Flow

Password reset pages and email notification templates rely on Liquid templates. These can silently break when Liquid tags or syntax become deprecated.

Failure mode: the user receives a password reset email and clicks the link, but gets stuck — nothing happens, or they land on an error page.

Three things to verify and update when debugging a broken password reset:
1. **Form targets** — the form action must point to the correct endpoint
2. **URL redirections** — the redirect-after-reset path must resolve to a real page
3. **Token validation conditions** — the Liquid condition checking the reset token must use current syntax

The same deprecation issue can affect email notification templates (order confirmed, password reset, etc.) — any template written against an older Liquid version may contain deprecated tags that need replacing with equivalent current syntax.

---

## Fulfillment Template Failure: Missing DPI Values

A fulfillment template can fail silently if products do not have DPI values set. The artwork generation step requires DPI to calculate output dimensions.

When diagnosing unexplained fulfillment failures or blank/missing production files:
1. Check that the product template has DPI configured
2. If the template was recently edited and failures began immediately after, restore to the last known-good version as a first step

---

## URL Reserved Parameter Word Conflicts — 404 on Collection Pages

Reserved URL parameter words in Pixfizz can conflict with product filtering logic and cause 404 errors on collection or category pages.

Symptom: a product listing page returns 404 for some URL parameters that appear in filters or sort controls, even when the page itself exists and the products are published.

Diagnosis: check whether any URL parameters generated by the filtering or sorting UI use words that are reserved by the platform. Rename or avoid those parameter values. Raise with Matjaz to confirm the specific reserved word list.

---

## Stripe: Payment Form Locking After Failed Transaction

The Stripe payment form can become locked or unresponsive after a failed card attempt. The customer is unable to retry payment or submit the form again without refreshing the page.

This is a separate issue from the "pending without payment confirmation" problem below. The form locking is a front-end behavior — the payment UI freezes after the first failed attempt.

When encountered:
- Ask the customer to hard-refresh the page and retry
- Check browser console for JS errors from the Stripe integration
- Raise with Matjaz for investigation if reproducible

Root cause unconfirmed. Observed on a client site (April 2026) and discussed in the April 27 standup call.

---

## Stripe: Orders Reaching Pending Without Payment Confirmation

Stripe payment integration has a known fragility: orders can occasionally reach `pending` status without a corresponding payment confirmation arriving.

This is an intermittent issue — root cause unconfirmed. When encountered:
- Check Stripe dashboard for the payment event timeline
- Confirm whether the webhook fired and reached Pixfizz
- Raise with Matjaz/Richard for investigation

Do not manually confirm orders without verifying payment status in Stripe first.

---

## FTP Original Files Not Appearing After Fulfillment

Original design files may intermittently fail to appear in the FTP "Original Files" folder after order fulfillment. The production files (rendered artwork) may deliver correctly while the originals are missing.

Root cause unconfirmed — raise with Matjaz when encountered. Do not assume the order failed; check the production files folder separately.

---

## Production File Regeneration

To request a new production file for a project, any existing generated file must be **deleted first**. The platform does not allow requesting a new production file while a previous one still exists on the project. Delete the existing file, then request regeneration.

This applies to both admin-initiated regeneration and API-triggered regeneration.

---

## Font Stability: Transfonter Preprocessing

If font rendering issues appear after a platform PDF library update (symptoms: rectangles/"tofu" characters in the editor, "Impossible to extract embedded font" errors in generated PDFs), re-processing the font files through [Transfonter](https://transfonter.org/) can resolve the issue. Transfonter normalizes Unicode encoding and fixes vertical metrics, producing more stable font files for the PDF generation engine.

After re-processing, re-upload the fonts via the admin font management interface. Any PDFs generated during a bug window will need to be individually regenerated — the platform does not auto-regenerate historical PDFs.

---

## Gallery ZIP Download Fails Silently on Large Galleries

**Symptom:** Customer clicks the gallery download button and nothing happens. No error, no progress overlay, no file download.

**Cause:** The ZIP download is built entirely in browser memory using `zip.js`. For large galleries (300+ images, ~333MB+), the browser needs to hold all fetched image data plus the finished zip blob simultaneously — easily 600MB+ of RAM. Most browsers silently kill the operation or stall. Additionally, `Promise.all` fires all HTTP fetches at once, overwhelming the browser's 6-connection limit.

**Fix applied (client-side):**
- Batch fetches in groups of 6 using sequential `Promise.allSettled`
- `try/catch` around the entire flow so failures surface as an `alert()` rather than a dead button
- Skipped images are logged to console but don't fail the whole ZIP
- Progress overlay renders correctly because batching allows incremental updates

**Remaining limitation:** This fix handles medium-large galleries reliably on desktop browsers. For extremely large galleries (500+ images), the browser memory limit is still a ceiling. The robust long-term solution is a server-side ZIP endpoint (needs Matjaz).

---

## Bootstrap 4.6 Utility Class Override Requires `!important`

Bootstrap 4.6 utility classes (`.flex-wrap`, `.d-flex`, `.text-center`, etc.) are declared with `!important` in the Bootstrap source. This means any CSS override in `style/custom.css` that targets the same property will lose the specificity battle regardless of selector depth — unless it also uses `!important`.

**Example:** To override `.flex-wrap` (which sets `flex-wrap: wrap !important`) on a specific element:

```css
/* This does NOT work — Bootstrap's !important wins */
.my-container .d-flex {
	flex-wrap: nowrap;
}

/* This works */
.my-container .d-flex {
	flex-wrap: nowrap !important;
}
```

This applies to **all** Bootstrap 4.6 utility classes. When debugging a CSS override that should work based on selector specificity but doesn't, check whether the target property is set by a Bootstrap utility — if so, `!important` is required.

---

## iOS Older iPad: HEIC Upload Appears Frozen

**Symptom:** On older iPads, uploading a HEIC image makes the upload appear frozen.
The customer sees no spinner or progress, may assume it failed, and retries or
abandons.

**Cause:** A HEIC to JPEG conversion runs on upload with no visual feedback on
these devices. The upload is working; only the feedback is missing.

**Status:** A progress indicator / spinner is a pending UX improvement, not yet
shipped. Until then, flag this to clients whose customers use older iPads.

---

## Kiosk iPad: Browser Autofill Causes Login Confusion

**Symptom:** On a shared kiosk iPad, customers get confused at the sign-in step. The browser autofills a previous customer's saved email/password, so people either sign into the wrong account or cannot tell whose details are showing.

**Context:** Kiosk mode supports a skip-sign-in flow for faster gallery access and checkout. Browser-level credential autofill works against this on a shared device.

**Fix:** Turn off browser autofill / saved passwords on the kiosk iPad's browser. This is a device setting, not a Pixfizz setting. Recommend it as part of kiosk device setup.

---

## CMS Tar Import — Checklist Flags Not Reliably Applied

When importing a CMS backup (`.tar`), the `admin/checklist/*` flag values inside the tar are **not reliably applied** on import. A flag can be set correctly in the tar and still land as its default (or previous) value on the imported site.

Observed case: a `custom-home-page` flag that was `TRUE` in the tar imported as unset, so the custom homepage snippet did not render until the flag was ticked manually in the admin.

Fix / practice: after **any** CMS tar import, verify the relevant `admin/checklist/*` flags in the admin UI (Custom Admin → the relevant settings card) rather than assuming the imported values took effect. This is separate from the snippet-import behaviour (syntax-error snippets are silently skipped) — here the snippet may import fine but the flag that switches it on does not.

Related: child sites can only override checklist snippets that already exist on the parent, and account-v2 style flags gate whole snippet families — so a missing flag can make a correctly-imported snippet appear to "do nothing."

---

## Shopper v2 Account — `date_format` Capture Missing `| strip`

**Symptom:** In the Shopper v2 account area, dates on the Orders list and the Dashboard
render in a different format from the Order Details page, even though all three read the
same `date_format` value.

**Cause:** `account/v2/orders` and `account/v2/dashboard` capture the `date_format`
checklist value **without** a trailing `| strip`, while `account/v2/order-details` strips
it. The unstripped capture carries whitespace, so the format string does not match and the
date renders inconsistently.

**Fix:** Add `| strip` to the `date_format` capture in `account/v2/orders` and
`account/v2/dashboard` so all three v2 account snippets agree. This is the same
capture-comparison rule that applies to all checklist snippet values: always `| strip`
after `{% capture %}` before comparing or using the value.

---

## Collection Filter Drilldown — Blank PDP After Changing a Filter

**Symptom:** on a filtered product detail page, changing one filter (for example
Width) leaves the page showing only the collection shell — description,
production time, feature list — with no product title, price, gallery or
add-to-cart. One of the other filters usually disappears from the sidebar at the
same time.

**Cause:** the filters are configured as flat, independent lists with no
dependency logic between them. When the parent filter changes, the stale value
of the dependent filter survives in the URL. If no product exists at that
intersection the drilldown bails out, the product is never assigned, and the
page renders the shell only.

The same failure occurs with **no stale parameter at all** when the dependent
filter's configured default is invalid for the newly selected parent value. The
renderer falls back to that default, hits another empty intersection, and bails
identically. This form is usually latent from day one and only surfaces when a
second batch of products introduces sizes the original default does not cover.

**Cause in the template:** both `product/product-details-filter` and
`product/details-filter-dual-mode` used a two-tier selection guard — URL
parameter if valid, otherwise the configured default — with no final fallback.
When both tiers fail the selected option is nil and the drilldown breaks.

**Fix:** make the guard a three-tier cascade.

1.  URL parameter, if valid for the current selection
2.  Configured default, if valid for the current selection
3.  Otherwise the first entry of the currently available options

Tier 3 guarantees a valid selection for every reachable combination, so the page
always resolves to a real product.

**Diagnosis:** build the full product matrix from a collection export before
touching the template. The reported path is rarely the only broken one — a
single invalid default typically produces several empty intersections at once.

**Blast radius:** `product/product-details-filter` is the parent snippet used by
many child sites. Prove the fix as a site-level override first, then promote it
to the parent.

## Two rules for diagnosing platform problems

### A matching symptom is not a confirmed cause

When the documented cause predicts the observed symptom *exactly*, that is when to be most
careful, not least. A perfect match is what stops people looking further.

**Find a working instance and diff it before acting.** Export the product, site or
configuration that works, diff it field by field against the one that does not, and act on
the difference. If the working one carries the supposedly-fatal setting, the documentation
is wrong.

This has now resolved two multi-round misdiagnoses on this platform, both of which had
survived several confident, internally consistent explanations.

### A measurement is only evidence if the path it travelled is known not to alter it

Chat attachments, messaging apps and some file-sharing tools silently re-encode files —
commonly capping the long edge and converting to JPEG, which destroys transparency.

- Measure on the machine holding the file, or transfer by a path known to preserve bytes,
  and **confirm the byte count and checksum match the source**.
- Read the file's own header rather than a viewer's info panel. A macOS info panel reports
  "RGB" for an RGBA PNG and "72 dpi" for a file with no DPI at all — both readings have
  caused wrong diagnoses.
- **If measurements cluster on a suspiciously round number, that number is far more likely
  to be a limit in the measurement path than a threshold in the system being studied.**

------------------------------------------------------------------------

## Customer-Supplied Strings Rendered in Admin Must Be Escaped

Any shopper-controlled value that an admin page renders — address lines, order
notes, company names, personalisation text, uploaded filenames — is untrusted
input on a page viewed by a privileged user. Unescaped, it is a stored
cross-site-scripting vector aimed directly at admin sessions, and the shopper
never needs an account to plant it: placing an order is enough.

The rule for any custom admin or job-ticket rendering:

- Pass every shopper-supplied value through `escape` (or `escape_json` in JSON
  fulfillment templates) before it reaches the page. `31_FULFILLMENT_ENGINE.md`
  already applies `escape_json` throughout its payload examples for this reason.
- Treat the admin surface as the higher-value target, not the lower one. A
  storefront XSS reaches one shopper's session; an admin XSS reaches the account
  that can read every order on the site.
- Do not rely on field-level validation at the point of entry. Address and note
  fields legitimately accept punctuation, and the checkout form is not the only
  writer — the API and imports write the same fields.

### If admin credential compromise is suspected

Order of operations, because doing these in the wrong order leaves the window
open:

1. Fix the injection point first. Rotating credentials while the payload is
   still rendering just re-harvests the new ones.
2. Force a logout of all admin sessions site-wide, so any session already
   hijacked is invalidated.
3. Reset affected admin passwords and issue temporary credentials.
4. Review the login log for unexpected source geographies before assuming the
   attempt failed.

Note that a bulk password reset across an account holding **many sites** has
been observed to time out where a single-site reset succeeds — if the reset
appears to fail on a large multi-site account, that is the likely cause rather
than a wrong credential. Escalate rather than retrying blindly.

Incident-specific detail (affected sites, indicators of compromise, the
superadmin reset path) is deliberately **not** in this public reference — it
lives in the private internal repo.

------------------------------------------------------------------------

## Editing a Confirmed Order Does Not Regenerate Its Production Files

Changing anything on an order that has already produced artwork — editing the
project in the design tool, correcting a calendar element, swapping an image —
leaves the previously generated production files untouched. The order in admin
shows the edit; the lab still receives the pre-edit artwork, with nothing to
signal the discrepancy.

After any edit to a confirmed order, **force a refulfill from the admin order
detail page**. Because the platform will not issue a new production file while
an old one exists on the project, the existing generated file must be deleted
first (see Production File Regeneration above).

This is the failure mode behind "we corrected it and the wrong version still
printed". Treat the refulfill as part of the edit, not as an optional follow-up.

------------------------------------------------------------------------

## Wrapped-Canvas Products: Mirrored Edges Are a Template Fault, Not a Render Fault

Symptom: a gallery-wrap canvas comes off the press with the wrapped edges
mirrored — the image reflected around the wrap rather than continuing across it.

This presents as a rendering or production bug and is almost always neither. The
two things to check, in order:

1. **The product template definition** — the page/wrap geometry declared in the
   XML for that product.
2. **The bleed value** — a bleed that does not match the physical wrap depth
   makes the wrap region fall in the wrong place, and mirroring is the visible
   result.

Correcting the template and the bleed size resolves it. Do not start by
re-rendering the order or re-uploading the customer's image; neither addresses
the cause, and both cost a print.

------------------------------------------------------------------------

## Changelog
- 2026-03-21: Initial content from platform documentation export.
- 2026-04-23: Added CSS snippet logs diagnostic note, password reset Liquid deprecation pattern, fulfillment template DPI failure, URL reserved parameter 404 gotcha, Stripe pending-without-payment issue, FTP original files intermittent failure.
- 2026-04-27: Added Stripe payment form locking after failed transaction.
- 2026-04-29: Added production file regeneration constraint (delete existing file before requesting new one). Added font stability / Transfonter preprocessing workaround.
- 2026-05-19: Added gallery ZIP download silent failure on large galleries (memory limit + batched fetch fix). Added Bootstrap 4.6 utility class `!important` override requirement. Source: Claude chats (gallery download fix, gallery v2 button layout).
- 2026-06-01: Corrected Editor Iframe CSS Isolation note; added iOS HEIC upload feedback gotcha. Source: claude-chat/fireflies-call.
- 2026-06-26: Added kiosk iPad browser-autofill login-confusion gotcha (disable autofill on shared kiosk devices). Source: fireflies-call.
- 2026-07-11: Added CMS tar import gotcha — `admin/checklist/*` flags are not reliably applied on import (observed: custom-home-page TRUE in tar but unset after import); verify checklist flags in the admin UI after any import. Source: claude-chat (Shopper CMS tar build).
- 2026-07-20: Added Shopper v2 account `date_format` gotcha — `account/v2/orders` and `account/v2/dashboard` miss `| strip` on the date_format capture, causing a format mismatch with order-details. Source: claude-chat.
- 2026-08-14: Added the rule that shopper-supplied strings rendered on admin pages must be escaped, with the ordered response steps for suspected admin credential compromise (fix injection point, force site-wide logout, then rotate) and the multi-site bulk-password-reset timeout note. Added the rule that editing a confirmed order does not regenerate its production files — force refulfill, deleting the existing generated file first. Added wrapped-canvas mirrored-edge diagnosis (template definition and bleed value, not the renderer). Source: slack-message (#support), fireflies-call (2026-08-11/12).
- 2026-07-28: Added Collection Filter Drilldown blank-PDP entry — stale or invalid dependent filter values break the drilldown; fix is a three-tier selection cascade in `product/product-details-filter` and `product/details-filter-dual-mode`. Source: claude-chat.


=================================================================
FILE: 41_IMPLEMENTATION_PATTERNS_UPDATED.md
=================================================================

# 41 --- Implementation Patterns

**Authority Scope:** Reusable architectural patterns for Pixfizz
storefronts.

*Last updated: 2026-06-30*

------------------------------------------------------------------------

# Pixfizz Implementation Patterns

This document captures **real-world patterns used repeatedly when
building Pixfizz storefronts**.

These patterns represent practical knowledge beyond API documentation.

------------------------------------------------------------------------

# Email Rendering Patterns

## Project Preview Image

For design products:

``` liquid
<img src="https://{{ website.hostname }}{{ orderline.project | preview_url: height: 300, share: orderline.project.share_code }}">
```

Key rule:

Project previews in email must include the **share code**.

------------------------------------------------------------------------

## Product Image Fallback

Static products should fall back to the product image.

``` liquid
<img src="{{ orderline.product.image | asset_url:200, cdn:false }}">
```

------------------------------------------------------------------------

# Cart Rendering Patterns

## Looping Orderlines

Correct loop structure:

``` liquid
{% for orderline in cart.orderlines %}
...
{% endfor %}
```

Orderlines are the **atomic commerce unit** in Pixfizz.

------------------------------------------------------------------------

# Inventory UI Pattern

Example low-inventory badge:

``` html
<span class="px-stock-badge px-stock-low">
Only <b>{{ current_inventory }}</b> left
</span>
```

Recommended logic:

-   show badge when inventory < threshold
-   hide when inventory unlimited

------------------------------------------------------------------------

# Dynamic UI Trigger Pattern

Avoid inline JS triggers.

Instead:

1.  Render a DOM marker
2.  Let global JS detect the marker

Example marker:

``` html
<div id="px-rush-sameday-marker"></div>
```

This pattern works reliably with AJAX snippet updates when the snippet
only needs to signal presence to a global script.

For snippets that need to carry Liquid-baked values into JS, use the
`style onload` pattern below instead.

------------------------------------------------------------------------

# `style onload` Re-injection Pattern

## When to use

Use this pattern for any JS that must run on **both** initial page load
and every subsequent AJAX re-injection — for example:

-   Checkout snippets inside `async: true` forms
-   Snippets that re-render when the user switches delivery options
-   Any snippet where `<script>` tags are not reliably re-executing

`<script>` tags inside AJAX-injected HTML do not re-execute. The
`style onload` attribute fires every time the element is inserted into
the DOM, including after re-injection.

## Pattern

``` liquid
{% assign saved_value = form.values.cart.custom.my_field | default: cart.custom.my_field %}

{% capture my_init_script %}
(function() {
	var savedValue = '{{ saved_value | escape }}';

	// setup and event listeners here

	// Always call init directly — DOMContentLoaded does not re-fire on AJAX:
	initMyThing();
})();
{% endcapture %}

<style onload="{{ my_init_script | escape }}"></style>
```

## Rules

-   The `{% capture %}` and `<style onload>` blocks must sit **outside**
    any `<script>` tag. Placing them inside a `<script>` block causes a
    syntax error and breaks all JS on the page.
-   Always include a **direct call** to the init function at the end of
    the IIFE. Do not rely solely on `DOMContentLoaded`, `pageshow`, or
    `px.fragmentsReloaded` event listeners — they are not sufficient for
    AJAX re-injection.
-   Bake Liquid server values into the IIFE via `{{ variable | escape }}`
    so they are available without additional API calls.
-   Use `removeEventListener` before `addEventListener` on any element
    that persists across re-injections, to avoid stacking duplicate
    handlers.

## Separation from once-only logic

Keep logic that should run **once** on full page load (e.g. postcode
lookup, checkbox validation) in a separate `<script>` block with
`DOMContentLoaded`. Do not merge it into the `style onload` capture.

``` liquid
{% capture my_init_script %}
(function() {
	// Runs on every injection
})();
{% endcapture %}
<style onload="{{ my_init_script | escape }}"></style>

<script>
document.addEventListener('DOMContentLoaded', function() {
	// Runs once on full page load only
});
</script>
```

## Once-Per-Load Guard Pattern (`window.__flag`)

### When to use

Use when a snippet lives inside an async-reinjected section (e.g. `selectors: 'section.checkout-page'`) but contains logic that must fire **only once per full page load** — not on every re-injection. Examples:

- Advisory or welcome modals
- One-time analytics events
- Initialisation that would be disruptive if repeated

The `style onload` pattern fires on every DOM injection by design. Without a guard, the logic runs every time the user switches delivery options.

### Pattern
```liquid
{% capture px_once_init %}
(function() {
	if (window.__pxMyThingShown) return;
	window.__pxMyThingShown = true;

	// logic that should run only once per page load
	setTimeout(function() {
		var el = document.getElementById('my-element');
		if (!el) return;
		$(el).modal('show');
	}, 3000);
})();
{% endcapture %}
<style onload="{{ px_once_init | escape }}"></style>
```

### Rules

- Use a flag name specific to the feature (`window.__pxAdvisoryShown`, `window.__pxWelcomeShown`) — avoid generic names that could collide across snippets.
- The guard must be the **first line** of the IIFE so nothing runs before the check.
- The flag persists for the lifetime of the page session and is cleared on full page reload — which is the intended behaviour.
- Do not use this pattern for logic that genuinely needs to re-run on re-injection (state restore, dropdown population, etc.) — use the plain `style onload` IIFE for those.

### Gotchas: Bootstrap modals built inside a `style onload` IIFE

Two failure modes come up when a Bootstrap 4.6 modal is created or shown from a `style onload` IIFE on a Shopper/Shopify page:

- **jQuery is not available at script-load time.** The `style onload` attribute fires very early in page parsing, before the theme's jQuery has loaded. Any `jQuery(...)` / `$(...)` reference evaluated at load time fails silently. Only reference jQuery *inside* a user-interaction handler (click/change), which runs long after load — never at the top level of the IIFE.
- **`position: fixed` breaks under a transformed ancestor.** A modal inside a container that has a CSS `transform` (common in sidebars/option panels) renders inline/contained rather than as a full-screen overlay, because a transformed ancestor becomes the containing block for `position: fixed`. Fix: on first show, move the modal element to `document.body` (e.g. `document.body.appendChild(modalEl)`) so it escapes the transformed ancestor. Also wire close buttons with explicit delegated click handlers rather than relying on Bootstrap's `data-dismiss` auto-wiring, which may be absent in a trimmed Bootstrap build. Source: claude-chat (test-print charge modal).
- **`filter` on `<body>` breaks `position: fixed` and appending to body does not fix it.** `transform` is not the only property that creates a containing block for fixed positioning — `filter`, `backdrop-filter`, `perspective`, `contain`, and `will-change` do the same. A site whose `<body>` carries even a no-op `filter: blur(0px)` will contain every fixed element on the page, so the standard `document.body.appendChild(modalEl)` fix has nothing to escape to. Diagnose by checking computed style on `<body>` and every ancestor for these properties, not just `transform`. Where the offending rule cannot be removed (it is often part of a theme's page-transition effect), the fallback is a JavaScript viewport anchor that repositions the overlay against `window.scrollY` on scroll and resize. Source: claude-chat (custom designer modal).
- **`position: sticky` traps a modal below the backdrop.** This is a different mechanism from the containing-block problems above. `position: sticky` (and `position: fixed`) creates a **stacking context** unconditionally, even at `z-index: auto`. A modal rendered inside a sticky container therefore cannot rise above the body-level `.modal-backdrop` at z-index 1040, however high the modal's own z-index is. The symptom is a modal that is fully visible but completely unclickable. Confirm it with `document.elementFromPoint(x, y)` at the centre of the modal: a return of `DIV.modal-backdrop` proves the trap, and a manual `document.body.appendChild(modalEl)` in the console will unblock it. Fix it in CSS rather than JavaScript, by dropping sticky only while a modal is open: `body.modal-open .product-details.sticky { position: static; }`. When writing a diagnostic that scans ancestors for this class of bug, do **not** gate the check on `z-index !== auto`, because that filter skips every `sticky` and `fixed` element and is the reason the cause gets missed. Source: claude-chat (custom design tool on a child site).

### Variant: `sessionStorage` for tab-session persistence

`window.__flag` is cleared on full page reload. If the guard needs to persist across page loads within the same browser tab (e.g. a modal that should not re-show even if the user navigates away and returns during the same session), use `sessionStorage` instead:

```javascript
(function() {
	if (sessionStorage.getItem('pxMyThingShown')) return;
	sessionStorage.setItem('pxMyThingShown', '1');

	// logic here
})();
```

`sessionStorage` is cleared when the tab is closed but persists across page navigations within that tab.

------------------------------------------------------------------------
# Liquid Parses Before CSS/HTML Comments

A Liquid-rendered snippet is parsed as Liquid **before** the host language's
comments mean anything. A `{% ... %}` tag inside a CSS `/* */` block, an HTML
`<!-- -->` comment, or a JS comment is still parsed and can still throw
(`Invalid snippet tag syntax`) or execute. Two consequences for any snippet
delivered as CSS or JS via `{% snippet %}`:
- Never write Liquid tag syntax inside a comment, even as documentation. A
  banner comment reading `{% snippet 'x' %}` errors the whole file.
- A snippet must never `{% snippet %}`-include itself.
A CSS snippet should contain exactly one Liquid tag — the `asset_url` for its
font/asset — and nothing else. Worth a pre-handover check.

------------------------------------------------------------------------

# Canvas and Form Patterns (customer-facing tools)

- **Render at devicePixelRatio.** A canvas sized only in CSS pixels is
  upscaled by the browser on retina displays and looks soft. Set
  `canvas.width = cssW * dpr` and `ctx.setTransform(dpr,0,0,dpr,0,0)`;
  resizing a canvas resets its transform, so reapply each frame.
- **`form.requestSubmit()` vs `form.submit()`.** To add to cart from a custom
  UI, prefer clicking the page's real Add-to-Cart control. As a fallback,
  `form.requestSubmit()` fires the form's own handlers and native validation;
  `form.submit()` bypasses both. Find the product form via
  `uploadHost.closest('form')` — `input.form` is null when the input sits in a
  shadow root.
- **Cart preview of a generated image.** The default cart option loop skips
  `option.template_option.type == 'image_upload'` (and `text`, `font`,
  `hide_from_cart`, `edit_from_cart`). A generated preview must therefore be
  attached as `file_upload`, and displaying it in the cart depends on whether
  an uploaded file's asset resolves to a URL in orderline Liquid — confirm
  before relying on it.

------------------------------------------------------------------------  

# Defensive Snippet Architecture

When editing widely reused snippets:

-   preserve variable names
-   preserve JS hooks
-   add logic without breaking existing behavior

This keeps sites stable across updates.

------------------------------------------------------------------------

# `px-project-preview` — Shadow DOM Image Styling

The `px-project-preview` web component renders its image inside a shadow DOM. Setting `height: 100%` or `object-fit` on the component element itself has no effect on the internal image.

Use the `::part(img)` CSS pseudo-element, which the component exposes as a named part:

```css
px-project-preview::part(img) {
	object-fit: cover;
	height: 100%;
	width: 100%;
}
```

This is the correct approach whenever the component's image does not fill its container as expected.

------------------------------------------------------------------------

# Hero Background Overlay — Preferred Pattern

Do not use `background-blend-mode: multiply` combined with Bootstrap's `bg-dark` class for hero image overlays. The blend mode cannot be tuned and produces excessively dark results.

Use a semi-transparent `rgba` overlay `<div>` with `position: absolute` instead:

```html
<div style="position: absolute; inset: 0; background: rgba(0,0,0,0.35); z-index: 1;"></div>
```

- Starting point: `rgba(0, 0, 0, 0.35)`. Adjust the alpha value to taste.
- The hero container must be `position: relative`.
- Ensure text content has a higher `z-index` than the overlay.

------------------------------------------------------------------------

# Skip Cart Redirect Pattern (`product.custom.skip_cart_redirect`)

## When to use

Use when a customer wants the shopper to stay on the product page after
adding an item to cart, instead of being redirected to `/site/cart`.

Enabled by setting the **boolean** custom field `skip_cart_redirect` on the
product. Works for both static and design products.

## How it works

### Static products (`cart_add_product` form)

The `page: 'cart'` form parameter renders as
`<input type="hidden" name="target" value="/site/cart">`. The button
`onclick` calls `enableSkipCartRedirect(form)` which overwrites the
`target` input value with the current page URL +
`?product_added_to_cart=t`, so the platform redirects back to the
product page.

### Design products (`project_create` form)

A disabled hidden `<input name="target">` is added inside the form. The
button `onclick` (inside the `btn_add_to_cart` conditional) calls
`enableSkipCartRedirect(form)` which enables the input and sets its
value. The platform picks up the `target` field and redirects back.

### Quick-quantity products

The quick-quantity JS checks `window.pxSkipCartRedirect`. If true, it
calls `showPxCartToast()` instead of redirecting to `/site/cart` after
the fetch calls complete.

### Visual feedback

A Bootstrap 4.6 toast notification with a green checkmark appears when
the `product_added_to_cart=t` URL parameter is detected on page load.
The script also suppresses the platform's built-in flash message via a
MutationObserver. The URL parameter is cleaned via
`history.replaceState` so a page refresh does not re-trigger the toast.

## Key implementation details

-   `enableSkipCartRedirect(form)` finds `input[name="target"]` by name
    (not ID) — works for both static (platform-generated) and design
    (manually added) target inputs.
-   The toast HTML and script sit outside `{% unless product.is_static %}`
    so they render for both product types.
-   `window.pxSkipCartRedirect` is set in the inventory flags script block.
-   The toast uses an IIFE for immediate flash suppression + URL cleanup,
    with `$(function() { ... })` for the actual toast show (requires
    Bootstrap JS to be loaded).

## Dependencies

-   Requires `btn_add_to_cart` (product or collection level) for design
    products — otherwise the button goes into Design Now mode and the
    `enableSkipCartRedirect` onclick never fires.
-   Not required for static products — they are always add-to-cart.

------------------------------------------------------------------------

# Custom Tool Dependency Loading

A custom design tool delivered as a site asset (its own bundled build, plus any
vendored libraries such as a ZIP writer) needs `<script>` tags on the page. The
obvious host is `integrations/custom-body-scripts`, and it is the wrong one.

`integrations/custom-body-scripts` is a snippet child sites routinely override to
carry their own analytics and third-party tags. An include added to the parent
copy is therefore invisible on every child that has an override, and the failure
is silent — the product page renders, the tool never initialises, no error.

**Pattern:** have the tool's own product snippet load its dependencies.

-   The product snippet (e.g. `product/my-tool`) is the only file guaranteed to
    be present wherever the tool is actually used.
-   Guard the load so repeated renders do not double-inject: test for the global
    the library exposes, and only create a `<script>` element if it is absent.
-   Chain initialisation off the injected script's `onload`, not off document
    ready — the dependency resolves after the page has already loaded.

Deployment on a new site then reduces to two steps: upload the assets, add the
product snippet. No shared-include edit, and no parent-level change that has to
survive every child override.

------------------------------------------------------------------------

# Custom Type Collection: Sort + Filter Pattern

When you need to both **sort** and **filter** Custom Type instances (e.g. date-based filtering, unpublished flags), the Liquid `sort` filter cannot be applied to the push-built filtered array — it silently fails on plain arrays with nested dot-notation keys.

**Correct pattern:** sort at initial collection assignment, then push-filter preserving that order.
```liquid
{% assign all_items = website.custom_types['my_type'] | sort: 'custom.my_date_field' | reverse | page_size: 9999 %}
{% assign today = 'now' | date: '%Y-%m-%d' %}
{% assign filtered_items = '' | split: ',' %}

{% for item in all_items %}
  {% assign item_date = item.custom.my_date_field | date: '%Y-%m-%d' %}
  {% if item_date <= today %}
    {% assign filtered_items = filtered_items | push: item %}
  {% endif %}
{% endfor %}

{% for item in filtered_items %}
  {# filtered_items is already sorted — insertion order is preserved from all_items #}
{% endfor %}
```

The `sort` and `reverse` on `all_items` execute in the database. The push loop inserts items in that already-sorted order. `filtered_items` inherits the sort without needing a second `sort` call.

**Note:** Date fields used as sort keys must be in ISO 8601 format (`YYYY-MM-DD`) for sort and string comparison to work correctly.

**Reference implementations:** blog listing page (`blog_post` Custom Type), promotions modal (`promotions` Custom Type).

# Collection Filters: Conditional Default Values

The `collection_filters` custom field on a Collection supports Liquid conditionals. This allows a filter's default value (or even its entire rendering) to change based on another filter's current selection via `request.params`.

## Pipe-Delimited Syntax

Each filter is one line in the `collection_filters` field:
Label | param_name | attribute | default_value | key: value options

| Field | Description |
|---|---|
| Label | Display label shown to the shopper |
| param_name | URL query parameter name (e.g. `orientation`, `size`, `group`) |
| attribute | Product attribute to filter on (e.g. `product.custom.size`) |
| default_value | Pre-selected value on page load |
| options | Key-value pairs: `control_type: radio`, `control_type: dropdown`, `asset_images: true` |

## Conditional Default Pattern

Wrap a filter line in Liquid `{% if %}` / `{% elsif %}` blocks, reading the parent filter's current selection from `request.params['param_name'][0]`.

**Example — Size default changes based on Orientation selection:**

```liquid
Orientation | orientation | product.custom.orientation_img | portrait.jpg | control_type: radio | asset_images: true
{%- if request.params['orientation'][0] == 'landscape.jpg' or request.params['orientation'][0] == 'portrait.jpg' or request.params['orientation'][0] == blank %}
Size | size | product.custom.size | 16x20 | control_type: dropdown |
{%- elsif request.params['orientation'][0] == 'square.jpg' %}
Size | size | product.custom.size | 12x12 | control_type: dropdown |
{%- endif %}
```

**Key rules:**
- `request.params['param_name'][0]` — bracket notation with `[0]` to get the first (or only) selected value
- Include `or request.params['param_name'][0] == blank` in the default branch to cover initial page load (no selection yet)
- The param values compared must exactly match the values stored in the product's custom field
- Use `{%- -%}` whitespace-stripping tags to avoid blank lines in the rendered output
- The same filter line (e.g. Size) appears in each conditional branch — only the default_value differs
- This pattern also works for rendering entirely different filters based on selection (not just changing defaults)

RATIONALE: The collection_filters conditional default pattern is a reusable Shopper technique for dependent filter defaults. Not documented anywhere in the KB despite being in active use.
SOURCE: Current conversation — collection_filters conditional default for orientation/size
SOURCE TYPE: claude-chat
------------------------------------------------------------------------

# Fulfillment Transformation: Bulk Copy Across Templates

Fulfillment transformation settings (color profile conversion, bleed adjustments, output format, etc.) can be **bulk-copied from one template to multiple other templates** in a single operation.

Use this when deploying the same transformation rules to a set of related templates — for example, a family of photo book sizes where all variants share the same production pipeline configuration. Doing this manually per template is both slow and error-prone.

This preserves consistent pricing and design logic across a product range without having to re-enter settings for each template individually.

# data-method="post" and Rails UJS Event Bubbling

Links using `data-method="post"` (for example the copy/duplicate-project link
`/v1/books/{{ project.id }}/copy`) rely on a delegated Rails UJS handler attached
at the document level. The click must bubble up to `document` for the POST to
fire.

`onclick="event.stopPropagation()"` kills the event before it reaches that
handler, so the POST never fires and the browser falls back to a plain GET. The
action silently does the wrong thing (no error).

Fixes:
- Remove the `stopPropagation`. If a parent card click handler must be prevented,
  handle that in the parent (bail out when the click target is inside an `<a>`).
- Or drop `data-method` and submit a small inline `<form method="post">` with a
  submit button. A native form POST does not depend on UJS, so `stopPropagation`
  on the form is safe.

------------------------------------------------------------------------

# Conditional-Skip Comma Handling in JSON Loops

When building a JSON array in Liquid by looping a collection, do not use `forloop.last` to decide where commas go if any item can be skipped mid-loop (for example an `{% if %}` guard that omits some rows). When the final iterated item is the one skipped, the comma logic produces invalid JSON (a trailing or missing comma).

Use a `first_row` flag instead, and prepend a comma before every emitted row except the first:

```liquid
{%- assign first_row = true -%}
[
{%- for item in collection -%}
	{%- if item.custom.hide_from_search -%}{%- continue -%}{%- endif -%}
	{%- unless first_row %},{% endunless -%}
	{ "code": "{{ item.code }}" }
	{%- assign first_row = false -%}
{%- endfor -%}
]
```

This is robust regardless of which items are skipped. It applies to any Liquid JSON emitter — search index pages, fulfillment payloads, product feeds.

## Measured platform behaviour

Evidence-backed on a live child site, not inference. Recorded because each one
either removes a constraint people assume exists, or is a constraint people
assume does not.

**`parse_json` is cheap at scale.** Parsing a ~20,700-character JSON string 25
times in a single page render produced no measurable TTFB change (10-run
medians: baseline ~0.79s, 1× parse ~0.72s, 25× parse ~0.70s — all inside network
noise). Large-JSON runtime composition is a safe pattern. This corrects any
assumption that `parse_json` must be rationed.

**Redirects capture dotted root paths.** A redirect rule such as
`[["^/llms\\.txt$", "<asset-url>"]]` fires as a 301 (x-runtime ~6ms) and serves
the CDN asset. Root-level "file" URLs are servable with no platform work.

**The asset store is extension-filtered.** The uploader **rejects `.txt` and
`.md`** (greyed in the picker) and **accepts `.json`** alongside images, js,
css, fonts and pdf. Two consequences: text-content root files must upload under
a `.json` name and will serve as `application/json`, which is functionally fine
for AI crawlers and cosmetically imperfect; and any generated creation bundle
must not put `.txt`/`.md` in `asset_files/` without verifying the importer path
separately.

**Image pipeline is WebP-only.** No AVIF support; `format: 'webp'` is the
ceiling. Do not emit AVIF variants in any `srcset`. *(Pending: AVIF was raised
as a future capability on the 2026-08-10 technical call. Treat WebP as current
until it ships.)*

**Admin-only custom type instances are platform-hidden**, visible to logged-in
admins and no one else. This is platform behaviour, not template behaviour, and
is the mechanism for preview-and-approve workflows.

---

## `!= blank` is not portable, and treats nil as blank

Two portability bugs that were invisible to reading the Liquid and only showed
up on execution:

- `{% if product.custom.field != blank %}` — in Ruby Liquid, nil **is** blank, so
  this behaves as intended on Pixfizz but not in other Liquid engines. Anything
  ported or tested outside the platform will diverge.
- `{% if product.id != blank %}` reported a nil product as present.

**Portable pattern:** normalise, then compare against an empty string.

```liquid
{% assign v = product.custom.field | default: '' | strip %}
{% if v != '' %}
	...
{% endif %}
```

For plain existence, `{%- if product.id -%}` is correct and unambiguous.

Related, from `50_LIQUID_REFERENCE.md`: `nil == false` evaluates as `false`, so
use `!= true` rather than `== false` for boolean checks; and a boolean custom
field is a real boolean, never the string `'true'`.

---

## Browser-generated print files carry no physical size

`canvas.toBlob()` writes **no pHYs chunk**. A PNG produced from a canvas therefore
declares no physical size at all, and anything opening it falls back to 72 dpi.

The pixels can be perfectly correct and the file will still print at the wrong size.
Measured on a real gang sheet, 10 Aug 2026: 1800 x 9000 px for an ordered 12 x 60 in
sheet — exactly 150 dpi, geometrically correct — with zero pHYs chunks, so it opens as
**25 x 125 in**.

This is invisible on screen and invisible in any proof. It presents to the lab as
"the printables are not coming in the right size".

**Inject the chunk after `toBlob`.** It is 9 bytes of data — X and Y pixels-per-metre as
big-endian uint32, then a unit byte of 1 — placed immediately after IHDR, which is always
the first chunk and always 25 bytes, so the insertion point is a constant offset of 33
bytes. CRC32 over the type and data.

Three rules for the implementation:

- **Tag with the ACHIEVED dpi, not the requested one.** Browsers clamp large canvases, so
  the two can differ. A file that lies about its own resolution is worse than one that
  admits being soft.
- **Replace any existing chunk rather than appending**, or the winner is undefined.
- **Never throw.** Return the original blob unchanged on any anomaly. A missing DPI tag is
  bad; failing the export outright is worse.

Note pHYs stores pixels-per-*metre*, so 150 dpi round-trips as 150.0124 and a 72 in sheet
reads back as 71.99 in. That is inherent and harmless.

**PDF output does not need this** — page boxes already declare physical size.

---

## Changelog

- 2026-03-12: Added `style onload` Re-injection Pattern section. Updated Dynamic UI Trigger Pattern.
- 2026-03-21: Added `sessionStorage` variant, `px-project-preview` shadow DOM styling, hero background overlay pattern.
- 2026-03-23: Added Skip Cart Redirect Pattern.
- 2026-03-26: Added Custom Type Collection: Sort + Filter Pattern.
- 2026-04-23: Added Fulfillment Transformation bulk copy pattern.
- 2026-05-13: Added Collection Filters: Conditional Default Values pattern.
- 2026-06-01: Added data-method/Rails UJS stopPropagation pattern. Source: claude-chat.
- 2026-06-30: Added conditional-skip comma handling pattern for JSON loops (first_row flag) — forloop.last breaks when items are skipped mid-loop. Source: claude-chat (search index work).
- 2026-07-04: Added Bootstrap-modal gotchas for `style onload` IIFEs (jQuery not available at load time; append modal to `document.body` to escape transformed-ancestor `position: fixed` containment). Source: claude-chat.
- 2026-07-25: Extended the fixed-positioning gotcha — `filter` (including a no-op `filter: blur(0px)`), `backdrop-filter`, `perspective`, `contain`, and `will-change` also create a containing block, and when the property sits on `<body>` the append-to-body fix does not work. Source: claude-chat.
- 2026-07-28: Added Custom Tool Dependency Loading — load tool dependencies from the tool's own product snippet, never from `integrations/custom-body-scripts`, which child sites override. Source: claude-chat.
- 2026-08-05: Added the `position: sticky` stacking-context modal trap, distinct from the containing-block gotchas, with the elementFromPoint confirmation, the `body.modal-open` CSS fix, and the diagnostic-script flaw of gating ancestor checks on `z-index !== auto`. Source: claude-chat.
- 2026-08-11: Added Measured platform behaviour — `parse_json` is cheap at scale (25 parses of a 20KB payload per render, no measurable TTFB change); redirects capture dotted root paths so `llms.txt` and similar are servable from an asset; the asset uploader is extension-filtered (.txt/.md rejected, .json accepted); WebP is the image-pipeline ceiling with no AVIF (AVIF under discussion, pending). Added the `!= blank` nil trap and the `| default: '' | strip` portable comparison. Source: claude-chat (Shopper v2 verification kit).


=================================================================
FILE: 45_ORDERHUB.md
=================================================================

# 45 — OrderHub Reference

**Authority Scope:** OrderHub operational configuration, Jobs, Production Board, Processes, Locations, integrations, and notifications. For the core Pixfizz order lifecycle see `32_ORDER_LIFECYCLE.md`.

_Last updated: 2026-07-31_

---

## What is OrderHub?

OrderHub is the workflow management layer of Pixfizz. It operates as a separate web application at `orderhub.pixfizz.com` and sits downstream of the Pixfizz CMS. It handles everything after an order is placed: routing jobs to production, managing job statuses, printing tickets, coordinating fulfillment, and notifying customers.

OrderHub accepts orders from multiple sources:
- Pixfizz storefronts (via webhook)
- Shopify
- Square and Lightspeed POS systems
- Manual creation in the OrderHub UI

---

## Jobs

Jobs are the individual line items within an order. Each job represents one product/service to be produced.

### Job Statuses

Standard status flow: **New → In Production → Completed**

- **New** — job received, not yet started
- **In Production** — work underway
- **Completed** — production finished

### Custom Statuses

Each Process can define up to **2 custom statuses**. Custom statuses act as sub-states of **New** — they slot into the workflow between New and In Production. They are useful for multi-step pre-production stages (e.g. "Awaiting Materials", "Sorted").

### Automated Order Status Cascade

When all jobs in an order reach **Completed**, OrderHub automatically updates the parent order status. This cascade eliminates manual order-level status management.

### Job Detail Fields

Each job record includes:
- Product name and thumbnail (photo print jobs only — see Job Thumbnails below)
- Assigned Process
- Notes
- Due date
- Assigned person
- Quantity
- Customer-selected options
- Custom fields

### Job Thumbnails

Thumbnails are generated via a Pixfizz Webhook and appear in the Jobs interface and on PDF tickets.

- **Photo print products** — thumbnail generated automatically
- **Static products** — no thumbnail generated

---

## Production Board

Kanban-style interface for visualising and managing all active jobs.

### Views

**Timeline view** — columns represent due dates. Drag a job card horizontally to change its due date.

**Status view** — columns represent job statuses (New, In Production, Completed, plus any custom statuses). Drag a job card horizontally to change its status.

Both views support filtering by location and date range.

### Drag-and-Drop Behaviour

Dragging a card updates the corresponding field instantly — no confirmation dialog. Changes sync to the order record in real time.

---

## Processes

Processes define production workflows. Each job is assigned to a Process when it arrives in OrderHub.

### Process Configuration

Each Process has:
- **Name** and **colour** (for Production Board display)
- **Default print size** and **copies**
- **OHD flag** — whether jobs in this Process are pushed to the OrderHub Downloader desktop app
- **Location overrides** — per-location settings that differ from the Process default

### Category–Process Linking

Each category (product type) is linked to a Process. The link includes:
- **Lead time** (days before production starts)
- **Production days** (days to complete production)

Processes are configured in **Settings** within the OrderHub app.

---

### Variant Value Routing (OrderHub Desktop)

OrderHub Desktop maps an orderline to the correct output by reading the **variant/finish value** (for example `lustre`, `glossy`) together with the size — not a lab- or printer-specific numeric finish code.

- Numeric finish codes (e.g. `222`, `202`) are specific to a given lab and printer. The same finish on a different printer or site can carry a different number, so numeric codes do not map cleanly in Desktop and make routing harder to maintain.
- For products fulfilled via OrderHub Desktop, prefer human-readable finish/variant codes (`lustre`, `glossy`, etc.) used consistently across all sizes. Desktop's own routing setup then translates those readable values to the correct printer/queue.

## Locations

Locations represent physical sites or branches of the organisation.

### Location Configuration

Each Location has:
- Name and address
- **Payment terminals** — one or more terminals per location, supporting Stripe, Helcim, and Gravity
- **Printer mappings** — logical Named Printer roles mapped to physical PrintNode-connected printers
- **Website link** — associates the location with a Pixfizz website (used for branding in notifications)
- **Opening hours** — shown to customers when they choose a pickup location at checkout
- **Google Maps link** — a map / directions link surfaced alongside the pickup address at checkout

Locations are managed in **Settings → Locations** within OrderHub.

---

## PDF Layout Studio

Visual drag-and-drop editor for designing printed production documents.

### Supported Document Types

- **Production Job tickets** — per-job work orders for the production floor
- **Order Summaries** — full order rundowns
- **Shipping Labels** — address labels for outbound shipments
- **Packing Slips** — pick-and-pack documents
- **QC Checklists** — quality control forms

### Key Features

- Dynamic variables pulled from order and job data (e.g. order number, customer name, product options)
- AI layout editor for rapid template generation
- Auto-print via PrintNode on trigger events
- S3 archiving of generated PDFs

Accessed via **Settings → PDF Designer** in OrderHub.

---

## PrintNode Integration

PrintNode is the bridge between OrderHub's PDF Layout Studio and physical printers on the lab floor.

### Architecture

Two-layer system:
1. **Named Printers** — logical roles defined in OrderHub (e.g. "Front Desk Printer", "Production Printer"). These are stable identifiers used in PDF Layout Studio auto-print rules.
2. **PrintNode connection** — each Named Printer is mapped to a physical printer connected via the PrintNode desktop agent on a lab computer.

### Setup

1. Install the PrintNode desktop agent on each production computer
2. In OrderHub **Settings → Locations**, assign a computer and map Named Printer roles to physical PrintNode printers
3. PDF Layout Studio auto-print rules reference Named Printers, not physical printer names

### Cost

PrintNode is **free for Pixfizz customers** — no separate PrintNode subscription required.

### EasyPost Shipping Label Auto-Print

When EasyPost is configured, purchased shipping labels can also auto-print via PrintNode. Configured per location.

### Troubleshooting: invoice/document not auto-printing
Invoice and document auto-print depends on both sides being aligned: the OrderHub Named Printer role must be mapped to a live PrintNode printer for that location, and the PDF Layout Studio auto-print rule must reference that Named Printer. If invoices are not printing, check for gaps between the OrderHub-side mapping and the PrintNode-side printer/agent configuration — a missing or mismatched mapping silently prevents printing. Source: Fireflies (2026-07-02).

---

## Film Scans Module

Dedicated module for managing scanned negatives workflow.

### Storage

All scan files are stored in **S3** (cloud object storage). The module does not manage physical film — only the digital scan assets.

### Status Tabs

Jobs flow through the following tabs:

| Status | Meaning |
|---|---|
| Needs Processing | Scans received but not yet processed |
| Gallery Pending | Processing started; gallery not yet created |
| Gallery Created | Gallery ready for customer |
| Emailed | Customer has been notified with gallery link |
| Archived | Complete; moved to long-term storage |

### Twin Check Number

Each film scan job has a **Twin Check Number** — a unique identifier used to match physical film rolls to their digital scan files throughout the workflow.

### S3 Auto-Sync

The module supports **S3 Auto-Sync**: when new scan files are deposited to a configured S3 path, they are automatically ingested into the Film Scans queue without manual upload.

### Automated Print Job Creation from Rolls

Film scan jobs can automatically generate matching print jobs. When a roll is processed, OrderHub creates print jobs whose quantities match the roll quantities on the order, and uploads the scanned artwork directly to the operator desktop for further processing. This removes the manual step of re-keying a develop-and-print order as a separate print job, and it links into the wider darkroom services workflow.

Because the print jobs are generated from roll quantities rather than from the orderline quantity, verify the generated job count against the order before releasing to production on the first few jobs after enabling this.

---

## OrderHub Downloader (OHD)

Desktop application for photo lab operators to receive, prepare, and route print jobs to local production equipment.

### What it does

OHD runs on a lab's local machine and continuously polls OrderHub for new jobs flagged for local production. When jobs arrive, it:
- Downloads job details and associated print files from OrderHub via API
- Organises files into structured folders with human-readable naming
- Applies channel and product routing logic to match jobs to the correct print workflow
  - **Gotcha:** when updating variants, do not copy old/stale channel IDs from a previous variant into a new one. Carrying over an outdated channel ID causes jobs to route to the wrong workflow (or fail to route) in OrderHub Desktop. Set the channel explicitly per variant, and prefer readable finish/variant values over numeric codes (see the 2026-06-30 variant-value routing note below). Source: Fireflies (2026-07-03).
- Generates **DPOF files** for compatible print controllers (Epson, Noritsu, etc.)
- Provides job review tools: colour correction, quantity management
- Offers **AI-powered upscaling** for low-resolution images
- Updates job status back in OrderHub once downloaded and processed

### Polling Behaviour

OHD polls for **New** jobs not yet received. If multiple OHD instances are running (e.g. across different workstations), job delivery is **first-come-first-served** — a job is only sent to one instance. Instances can be filtered by location.

### Auto-Update

OHD auto-update notifications are delivered via OrderHub. Labs always run the latest version without manual update steps.

### API Integration

OHD uses the OrderHub API to report job status changes back to the platform:

```
POST /functions/v1/update-job-status
Headers: X-API-Key: <api_key>
```

This keeps the OrderHub web UI in sync with local production progress.

> OHD is a companion tool to OrderHub — not standalone. Requires an active OrderHub connection and API key.

### Known issue: film scan folders stuck in the OHD watch folder

Film scan folders have been reported not moving out of the OHD watch folder (distinct from the Film Scans Module's S3 Auto-Sync). This is a repeat issue type across support tickets; root cause and fix are not yet confirmed. If a lab reports scans not progressing, check whether files are stalled in the local OHD watch folder before escalating. Source: support ticket #18341 (pending confirmation from dev).

---

## EasyPost Shipping Integration

EasyPost provides shipping label generation within OrderHub.

### Setup

Connected via **Manage My Organisation → Shipping** tab.

Toggle between **Production** (live) and **Test** environments during setup and testing.

### Auto-Print

Purchased shipping labels can be automatically printed via PrintNode. Configured per location in Location settings.

---

## POS Application Behaviour

### Loading a new build

The POS application does **not** pick up a new build by backgrounding and returning to it. To load the latest build, the operator must fully **close and reopen** the application.

The current build version is displayed at the **bottom of the login screen** when logged out. Use this to confirm which build a till is actually running before troubleshooting anything version-dependent.

### Screensaver

A screensaver activates after **5 minutes** of inactivity. This is intentional — it prevents burn-in on always-on till displays. It is not a session timeout and does not log the operator out.

### Receipt Printer Paper Size

Receipt paper size is a **software configuration, not a hardware property**. Loading a different paper roll does not change how receipts are formatted.

For the Epson TM-P20II the correct paper size is **58mm**, not the 80mm default assumed for larger countertop printers. The paper size must be set in two places:

1. The **Epson utility** for the printer itself
2. The **Mac print driver** for that printer

If receipts print with wrong margins, truncated lines, or excessive whitespace, check both of these before investigating the receipt template.

---

## POS Integration — Category Filter

When orders arrive from Square or Lightspeed POS systems, OrderHub imports products based on their category.

### Category Filter Behaviour

- The filter is a **case-insensitive exact match** against the category name
- Products whose categories do not match the filter are **silently ignored** — no error is raised
- Lightspeed: categories come from items on the order
- Square: categories come from Line Items

Configure the allowed categories in **Settings → Point of Sale** within OrderHub.

---

## Assigning Pixfizz Categories to Production Processes

When Pixfizz website orders arrive in OrderHub, the platform auto-creates **categories** based on the product types in those orders. Each category must be manually linked to a **Production Process** for correct routing, Production Board display, and job tracking.

### Unassigned Categories Alert

If any categories are unassigned, an **amber alert banner** appears on the Orders page:

> "X categories need process assignment"

Click **View Details** to see the dialog showing each unassigned category, its source website, and creation date.

### Assignment Path

1. Click **View Details** in the amber banner
2. Click **Configure in Organisations**
3. Go to **Organisations page → organisation settings → Pixfizz Websites section**
4. Select the relevant website (e.g. MYLAB)
5. In the **Categories list**, use the dropdown to select a Process for each unassigned category

---

## Order Status Sync (OrderHub → Core)

Marking an order as **shipped** in OrderHub also updates it as **shipped** in Pixfizz Core, provided the integration's API user is enabled. This keeps the Core order status in sync without a separate manual update. If a shipped status set in OrderHub is not appearing in Core, confirm the API user is active. Source: #development, Richard (2026-07-01).

---

## Email & SMS/RCS Notifications

OrderHub can automatically notify customers when an order is shipped or completed. Both channels are configured in the **Notify tab** of Organisation settings.

### Notification Triggers

| Trigger | When it fires |
|---|---|
| Shipped | After a shipping label is purchased via EasyPost, or when an order is manually moved to "shipped" status |
| Completed | When an order is manually moved to "completed" status |

Each trigger has an independent **Send on Shipped** / **Send on Completed** toggle.

### Pixfizz Notification Suppression

When OrderHub notifications are enabled, OrderHub passes `sendNotifications: false` back to the Pixfizz API to prevent duplicate messages. If OrderHub email is disabled, Pixfizz sends its own notifications as normal.

| Scenario | Who sends? |
|---|---|
| OrderHub email enabled + trigger enabled | OrderHub sends; Pixfizz suppressed |
| OrderHub email disabled | Pixfizz sends its own notifications |
| OrderHub SMS enabled + customer has phone | OrderHub sends SMS; Pixfizz doesn't send SMS |
| OrderHub SMS enabled + no phone on order | No SMS sent |
| Manual status change with "Notify" unchecked | Neither sends |

**Use OrderHub for order-confirmation and download emails; separate emails cannot be merged.** When OrderHub notifications are enabled, route order-confirmation and file-download emails through OrderHub. Combining multiple separate emails (e.g. confirmation + download) into a single message is not technically feasible — each remains its own message. Source: Fireflies (2026-06-29, 2026-07-02).

### Email Notifications

Emails are sent via an **n8n / SendGrid pipeline**.

**Prerequisites:**
- An n8n webhook URL configured in Organisation API secrets
- At least one active Pixfizz website linked to the organisation (for branding)

**Branding priority chain:**
1. The Pixfizz website linked directly to the order
2. The Pixfizz website linked to the order's pickup location
3. The organisation's first active Pixfizz website (fallback)

**Email settings:** From Name, Reply-To, BCC, subject line and body for each trigger.

### SMS/RCS Notifications

Sent directly via **Twilio REST API**. Each organisation uses its own Twilio credentials.

**Prerequisites:** Twilio account with active phone number; Account SID, Auth Token, and From Number entered in Notify settings.

**RCS:** Toggle available to enable RCS messaging. Falls back to standard SMS automatically if the recipient's device doesn't support RCS.

### Template Placeholders

These work in both email and SMS templates:

| Placeholder | Replaced with |
|---|---|
| `{{customer_name}}` | Customer name from order |
| `{{order_number}}` | Order number |
| `{{tracking_url}}` | Tracking link (shipped orders only) |
| `{{website_name}}` | Website/brand name |
| `{{customer_email}}` | Customer's email address |

### Two-Way SMS (Inbound)

OrderHub supports inbound SMS via a Twilio webhook, enabling two-way customer conversations.

**Twilio Messaging Webhook URL:**
```
https://nazkcvruighrhpgcarxg.supabase.co/functions/v1/twilio-inbound-webhook
```

Customer replies appear in the **SMS Conversation panel** on the order detail page.

**Automated keyword responses:**

| Keyword | Behaviour |
|---|---|
| `STATUS [order#]` | Looks up the order and replies with current status |
| `STOP` | Handled by Twilio (opt-out compliance) |
| Custom keywords | Configured via Auto-Replies in the Notify settings |

### Notification Log

Every notification attempt is logged on the order in the `notification_log` field, recording: channel (email/SMS), trigger, timestamp, success/failure status, and error details.

### Testing

Both the Email and SMS tabs include a **Send Test** button. Enter any email or phone number and choose a template (shipped or completed). Test notifications use placeholder values ("Order #TEST-001", "Test Customer") — no real order required.

---

## Custom fields consumed by OrderHub must be lowercase

Any custom field that OrderHub is expected to read — order type flags, delivery
speed options, client-specific routing fields — must be named in **lowercase**.
Mixed-case or capitalised field names are not matched.

New fields must also be **whitelisted in OrderHub** before they route. Creating
the field on the Shopper side is not sufficient on its own; a field that exists
and holds a value but was never whitelisted simply does not reach OrderHub, with
no error on either side.

This came up while adding Rush and Urgent delivery classifications as boolean
fields, alongside generic white-labelled option fields for client-specific
order types.

### The five order-level boolean slots

OrderHub supports **five** order-level boolean custom fields, not four. Confirmed
2026-08-13 against the authoritative list circulated by the OrderHub owner:

| Field | Meaning |
|---|---|
| `rush` | Standard rush tier |
| `urgent` | Faster-than-rush tier (typically same day) |
| `option1` | Generic slot, white-labelled per client |
| `option2` | Generic slot, white-labelled per client |
| `option3` | Generic slot, white-labelled per client |

Naming rules that are easy to get wrong:

- **No underscore and no digit separator** — the names are `option1`, `option2`,
  `option3`, not `option_1`.
- `rush` and `urgent` are **mutually exclusive** — model them as one radio group,
  never as two independent checkboxes.
- `option1`–`option3` are independent of each other and of the rush tier. Reserve
  `rush` and `urgent` for genuine delivery-speed tiers; anything else (a
  slow-it-down discount, a VIP flag, a client-specific order type) belongs in a
  generic slot.

**Where the customer-facing label lives is not yet settled.** Two readings are on
record: the label is configured per client **in OrderHub** (so the storefront
sends only the boolean), or the label **travels from the storefront** alongside
the flag (which would require a companion `option1_label`-style text field,
lowercase and separately whitelisted). Confirm with the OrderHub owner before
building the label path — the two designs differ in how many fields need
whitelisting.

**Migration trap.** Where a site already applies a rush charge through an Extra
Fee rule keyed on an older single-string field (for example a `rush_option` field
holding `none` / `standard` / `sameday`), renaming the field **silently drops the
fee**: the customer selects the faster tier, pays nothing, and the order still
reports as urgent. Re-point the Extra Fee rule in the same deploy as the field
rename, never afterwards. Carts already open at cutover will read as no-rush
unless the old values are mapped across.

---

## Changelog
- 2026-05-21: Created. Content sourced from OrderHub help modal articles (orderhub.pixfizz.com). Covers: Jobs, custom statuses, Production Board, Processes, Locations, PDF Layout Studio, PrintNode, Film Scans, OHD, EasyPost, POS category filter, Pixfizz category assignment, Email/SMS/RCS notifications.
- 2026-06-15: Added pickup-location opening hours and Google Maps link fields (surfaced in the store pickup UI at checkout). Source: slack-kb-sync (client call).
- 2026-06-30: Documented OrderHub Desktop variant-value routing — Desktop maps on readable finish/variant value + size, not lab/printer-specific numeric codes; prefer readable finish codes. Source: slack-message (#development).
- 2026-07-04: Added Order Status Sync (OrderHub → Core, shipped requires API user enabled); channel-ID copy gotcha in OHD variant updates; email-consolidation limitation (separate emails cannot be merged); PrintNode invoice auto-print troubleshooting. Source: Fireflies, slack-message (#development).
- 2026-07-25: Added POS Application Behaviour section (close/reopen required to load a new build; build version shown at bottom of logged-out login screen; 5-minute screensaver is burn-in prevention, not a session timeout; receipt paper size is software config — Epson TM-P20II is 58mm, set in both the Epson utility and the Mac driver). Added automated print job creation from film roll quantities with artwork upload to operator desktop. Source: fireflies-call (3x repeat signal).
- 2026-07-31: Added known issue — film scan folders reported stuck in the OHD watch folder (repeat issue type, root cause/fix not yet confirmed). Source: support ticket #18341 (pending confirmation).
- 2026-08-11: Added the custom field naming rule — any new custom field that OrderHub must read has to be lowercase, and whitelisted in OrderHub before it will route. Source: fireflies-call (2026-08-07).
- 2026-08-14: Corrected the order-level boolean slot count from four to five (`rush`, `urgent`, `option1`, `option2`, `option3`) and documented the no-underscore naming rule, the rush/urgent mutual exclusivity, the unresolved label-ownership question, and the Extra Fee re-point trap when migrating off a single-string rush field. Source: fireflies-call (2026-08-13), slack-message (#development).


=================================================================
FILE: 50_LIQUID_REFERENCE.md
=================================================================

# 50 — Pixfizz Liquid Reference

**Authority Scope:** Pixfizz Liquid objects, filters, and tags.
**Source:** Compiled from https://pixfizz.notion.site/Liquid-Documentation
**Last compiled:** 2026-04-10

---

# FORMS

---

## address_create

Creates a new address for the current user.

### Default behaviour

With no options, a successful submission does two things:
1. Saves the new address to the user's saved addresses.
2. Sets the new address as the selected address on the current cart.

```liquid
{% form 'address_create' %}
  <!-- address fields -->
  <button type="submit">Save address</button>
{% endform %}
```

### Options (added 2026-08-21)

| Option | Default | Effect when set to `false` |
|---|---|---|
| `assign_to_user` | `true` | The address is **not** saved to the user's saved addresses. Applied to the current cart only. Use for a one-off delivery address the shopper should not keep. |
| `assign_to_cart` | `true` | The address is **not** set as the selected address on the current cart. Saved to the address book only. Use when collecting addresses for later, not for the order in progress. |

Both options are independent — set either or both.

### Examples

Save to address book, do not apply to current cart:
```liquid
{% form 'address_create', assign_to_cart: false %}
  <!-- fields -->
{% endform %}
```

Apply to cart only, do not save to address book (one-off address):
```liquid
{% form 'address_create', assign_to_user: false %}
  <!-- fields -->
{% endform %}
```

> **Note:** Setting both `assign_to_user: false` and `assign_to_cart: false` creates the address
> but applies it to neither the cart nor the address book — rarely the intended result.

SOURCE: Notion Dashboard 🆕 Update (2026-08-21). Help article: https://help.pixfizz.com/ecommerce/pixfizz-ecommerce/pixfizz-cms/liquid/forms/address_create-form-options

## Overview

Pixfizz extends [stock Liquid](https://shopify.github.io/liquid/) with Pixfizz-specific objects, filters, and tags.
Liquid is supported in: CMS pages/layouts/snippets, email templates, SMS templates, and fulfillment templates.
Most objects, filters, and tags behave the same in all rendering contexts unless otherwise specified.

---

## String Quoting Rules

Pixfizz Liquid strings are delimited by single quotes (`'...'`). There is **no escape character** for single quotes inside Liquid strings. Backslash escaping (e.g. `\'`) is not supported and will cause a Liquid syntax error.

**Consequences:**
- **Apostrophes and contractions** (`we'll`, `don't`, `it's`) cannot appear inside single-quoted Liquid strings.
- If a string contains an apostrophe, rephrase to avoid it (e.g. `we will` instead of `we'll`).
- This applies to all Liquid contexts: `{{ '...' | t: ns: '...' }}`, `{% assign x = '...' %}`, `{% capture %}` string arguments, etc.
- **CMS import silently rejects** any snippet file containing this error — the file will not appear in the CMS after import, with no error message during the import process.

**Example — WRONG:**
```liquid
{{ 'Answer a few questions and we\'ll recommend something.' | t: ns: 'homepage' }}
```

**Example — CORRECT:**
```liquid
{{ 'Answer a few questions and we will recommend something.' | t: ns: 'homepage' }}
```

---

# OBJECTS

---

## Address

Represents a user registration address, an order delivery address, or a store public location.

**Filtering:** `id`, `custom.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

| Property | Description |
|---|---|
| `address.id` | Unique ID |
| `address.title` | Title |
| `address.email` | Email |
| `address.first_name` | First name |
| `address.last_name` | Last name |
| `address.company` | Company |
| `address.street` | Street |
| `address.street2` | Street line 2 |
| `address.city` | City |
| `address.region` | Region |
| `address.postcode` | Postcode |
| `address.telephone` | Telephone |
| `address.country` | Associated `Country` object |
| `address.is_public` | `true` if public/system address |
| `address.custom` | `CustomFields` object |

---

## Asset

Represents a website static asset (image or file uploaded to the CMS).

**Filtering:** `name`
**Sorting:** `name`

| Property | Description |
|---|---|
| `asset.name` | Asset name |
| `asset.url` | Asset URL path |
| `asset.description` | Asset description |

**Usage:** Access via `website.assets['filename.png']` or directly on objects that return an Asset (e.g. `product.image`, `collection.image`).
**Rendering:** Use the `asset_url` filter to generate a sized/formatted URL.

---

## Cart

Represents the current user's cart. Available globally as `cart` in the CMS.

**Filtering:** `id`, `custom.<field-name>`
**Sorting:** `created_at`, `custom.<field-name>`

| Property | Description |
|---|---|
| `cart.id` | Unique ID |
| `cart.promocode_code` | Text code of applied promocode, if any |
| `cart.total` | Total (orderlines + tax + shipping - discounts). Use `currency` filter. |
| `cart.orderlines_total` | Orderlines subtotal. Use `currency` filter. |
| `cart.orderlines_discount` | Amount discounted from orderlines. Use `currency` filter. |
| `cart.shipping_discount` | Amount discounted from shipping. Use `currency` filter. |
| `cart.discount` | Total discount (always = `orderlines_discount` + `shipping_discount`). Use `currency` filter. |
| `cart.shipping` | Shipping cost. Use `currency` filter. |
| `cart.extra_fees` | Array of active `ExtraFee` objects |
| `cart.tax` | Total tax amount. Use `currency` filter. |
| `cart.user_notes` | Customer notes (empty string if none) |
| `cart.telephone` | Buyer telephone |
| `cart.created_at` | Cart creation datetime. Use `date` filter. |
| `cart.updated_at` | Last updated datetime. Use `date` filter. |
| `cart.orderlines` | Paginated collection of `Orderline` objects (excludes child orderlines, default page size 50) |
| `cart.all_orderlines` | Paginated collection of all `Orderline` objects including children (default page size 50) |
| `cart.address` | Associated `Address` object, if any |
| `cart.shipping_option` | Associated `ShippingOption` object, if any |
| `cart.available_shipping_options` | List of all available `ShippingOption` objects |
| `cart.custom` | `CustomFields` object |

---

## ChosenOption

Represents a chosen value of a variant or template option on a specific `Orderline`.

| Property | Description |
|---|---|
| `chosen_option.key` | Key (typically the option type's code) |
| `chosen_option.value` | Value stored (option value code, or user-entered text for text options) |
| `chosen_option.variant` | `OptionType` object if this is a product variant, else `nil` |
| `chosen_option.variant_value` | `OptionValue` object if multiple choice variant, else `nil` |
| `chosen_option.template_option` | `OptionType` object if this is a template option, else `nil` |
| `chosen_option.template_option_value` | `OptionValue` object if multiple choice template option, else `nil` |
| `chosen_option.option_type` | Alias: `variant` or `template_option` depending on context |
| `chosen_option.option_value` | Alias: `variant_value` or `template_option_value` depending on context |
| `chosen_option.uploaded_image` | `Image` object if `image_upload` type, else `nil` |
| `chosen_option.uploaded_file` | `UploadedFile` object if `file_upload` type, else `nil` |

**Reading a `file_upload` option's asset.** Use `chosen_option.uploaded_file.url`
and `chosen_option.uploaded_file.filename`. The obvious alternatives do not work:
`chosen_option.value` returns the raw internal reference (`db:NNN`),
`chosen_option.asset.url` and `chosen_option.thumbnail_url` are not defined on a
chosen option, and the `preview_url` filter applies to projects, not uploads.
This matters most when rendering a cart thumbnail for a file attached by a
custom design tool.

---

## ChosenOptions

Proxy that allows access to `ChosenOption` objects on an `Orderline`. Returned by `orderline.chosen_variants` and `orderline.chosen_template_options`.

```liquid
{{ orderline.chosen_variants['foil-color'].variant_value.name }}

{% for option in orderline.chosen_template_options %}
	{{ option.template_option.name }}: {{ option.value }}
{% endfor %}
```

**Shopify IDs live in `chosen_variants`.** `shopify_product_id`, `shopify_variant_id`,
and `shopify_line_id` are accessed via `orderline.chosen_variants['<key>'].value`,
not `chosen_template_options`. See `31_FULFILLMENT_ENGINE.md` for the worked example.

| Property | Description |
|---|---|
| `chosen_options.size` | Number of chosen options |

---

## Collection

Represents a collection of design/product combinations as defined in "Manage Products → Collections". Auto-defined as `collection` on CMS URLs of form `/site/<pagename>/c/<collection-path>`.

**Filtering:** `id`, `path`, `custom.<field-name>`
**Sorting:** `path`, `custom.<field-name>`

| Property | Description |
|---|---|
| `collection.id` | Unique ID |
| `collection.path` | Collection path |
| `collection.name` | Display name |
| `collection.description` | Description |
| `collection.image` | `Asset` image |
| `collection.preview_images` | Paginated collection of `Asset` objects |
| `collection.collections` | Paginated collection of direct sub-`Collection` objects |
| `collection.design_products` | Paginated collection of published `DesignProduct` objects |
| `collection.static_products` | Paginated collection of published static `Product` objects |
| `collection.custom` | `CustomFields` object |

---

## Country

Represents a country. Exposed as `website.countries`.

**Filtering:** `id`, `code`, `code3`, `name`
**Sorting:** `code`, `code3`, `name`

| Property | Description |
|---|---|
| `country.id` | Unique ID |
| `country.name` | Country name |
| `country.code` | 2-char ISO code |
| `country.code3` | 3-char ISO code |
| `country.numeric_code` | Numeric ISO code |

---

## CustomFields

Proxy for accessing custom fields on various objects. Accessed via `.custom` on most objects.

- Custom field of type `snippet` → returns rendered snippet
- Custom field of type `asset` → returns `Asset` object
- All other types → returns string value (or `nil`)

```liquid
{{ collection.custom['product-types'] }}
{% if user.custom.is_vip == 'true' %}VIP{% endif %}
<img src="{{ page.custom.banner_image | asset_url }}" />
```

---

## CustomTypeInstance

Represents an instance of an admin-defined custom type.

**Filtering:** `id`, `custom.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

| Property | Description |
|---|---|
| `custom_type_instance.id` | Unique ID |
| `custom_type_instance.custom` | `CustomFields` object |

---

## CustomTypes

Proxy object for accessing custom type instances via `website.custom_types`.

```liquid
{% for post in website.custom_types['blog_posts'] %}
	{{ post.custom.title }}
{% endfor %}
```

Returns a paginated collection of `CustomTypeInstance` objects, or `nil` if the type doesn't exist.

---

## Design

Represents a design. Auto-defined as `design` when URL query string contains `?theme=<design_id>`.

**Filtering:** `id`, `code`, `name`, `custom.<field-name>`, `template.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

| Property | Description |
|---|---|
| `design.id` | Unique ID |
| `design.code` | Code |
| `design.name` | Name |
| `design.description` | Description |
| `design.image` | `Asset` image |
| `design.preview_images` | Paginated collection of `Asset` preview images |
| `design.template` | Associated `Template` object |
| `design.template_options` | Paginated collection of published top-level `OptionType` objects |
| `design.all_template_options` | Paginated collection of all `OptionType` objects (all levels, including unpublished) |
| `design.preview_page_names` | Array of page names marked as previews, in sort order |
| `design.custom` | `CustomFields` object |

---

## DesignProduct

Represents a Design + Product combination (what "Manage Products → Products" shows in admin). Note: equivalent to "Manage Products → Products" in the admin.

**Filtering:** `id`, `sku`, `design.<field-name>`, `product.<field-name>`
**Sorting:** `order`

| Property | Description |
|---|---|
| `design_product.id` | Unique ID |
| `design_product.sku` | Unique combination of design + product codes separated by `:` |
| `design_product.design` | Associated `Design` object |
| `design_product.product` | Associated `Product` object |

---

## ExtraFee

Represents an extra fee charged on an order or associated with a cart.

| Property | Description |
|---|---|
| `extra_fee.name` | Fee name |
| `extra_fee.code` | Fee code |
| `extra_fee.is_taxable` | `true` if taxable |
| `extra_fee.amount` | Fee amount. Use `currency` filter. |

---

## ExternalFile

> **DEPRECATED.** External files are deprecated. Use file_upload variant/template option instead.

| Property | Description |
|---|---|
| `external_file.url` | Download URL |
| `external_file.page_count` | Page count as provided by user (`nil` if not provided) |
| `external_file.description` | Description (`nil` if not provided) |

---

## Form

Available as `form` inside a `{% form %}` block.

| Property | Description |
|---|---|
| `form.errors` | `FormErrors` object from previous submission (`nil` if no prior submission) |
| `form.values` | `FormValues` object from previous submission (`nil` if no prior submission) |
| `form.captcha` | Renders default captcha (currently hCaptcha) |
| `form.hcaptcha` | Renders hCaptcha widget |
| `form.recaptcha` | Renders reCaptcha widget (requires superadmin setup) |

---

## FormValues

Accessible as `form.values` inside a form block. Proxy that lets you access submitted field values by parameter name.

```liquid
{% form 'address_create' %}
  <input name="address[street]" value="{{ form.values.street | escape }}" />
{% endform %}
```

---

## Gallery

Represents an image gallery.

**Filtering:** `id`, `code`, `custom.<field-name>`
**Sorting:** `id`, `created_at`, `custom.<field-name>`

| Property | Description |
|---|---|
| `gallery.id` | Unique ID |
| `gallery.name` | Name |
| `gallery.code` | Code |
| `gallery.created_at` | Creation datetime. Use `date` filter. |
| `gallery.images` | Paginated collection of `Image` objects |
| `gallery.custom` | `CustomFields` object |

---

## GeneratedFile

Represents a file generated during fulfillment. **Only available in fulfillment templates.**

| Property | Description |
|---|---|
| `generated_file.format` | `pdf` or `jpeg` |
| `generated_file.type` | `cover` or `pageN` (N = 1-based incrementing counter) |
| `generated_file.url` | Download URL |
| `generated_file.dirname` | Directory the file was fulfilled into (typically the order code) |
| `generated_file.filename` | Name of the generated file |
| `generated_file.quantity` | Always `1` except cut prints as JPGs/single-page PDFs, where it equals the user's chosen quantity |
| `generated_file.original_filename` | Original uploaded filename (cut print JPGs only) |
| `generated_file.autorotated` | `true` if image was auto-rotated to fit (cut print JPGs only) |

---

## Group

Represents a user group.

**Filtering:** `id`, `code`
**Sorting:** `id`, `created_at`, `updated_at`

| Property | Description |
|---|---|
| `group.id` | Unique ID |
| `group.name` | Name |
| `group.code` | Code |
| `group.category` | Category |
| `group.created_at` | Creation datetime. Use `date` filter. |
| `group.updated_at` | Last updated datetime. Use `date` filter. |
| `group.projects` | Paginated collection of shared `Project` objects |

---

## Image

Represents an image uploaded to Pixfizz (user-uploaded, gallery image, etc.).

**Filtering:** `id`
**Sorting:** `created_at`

| Property | Description |
|---|---|
| `image.id` | Unique ID |
| `image.filename` | Original uploaded filename |
| `image.width` | Width in pixels |
| `image.height` | Height in pixels |
| `image.url` | Full-size image URL |
| `image.created_at` | Creation datetime. Use `date` filter. |

> Use the `thumbnail_url` filter to get sized thumbnail URLs.

---

## Order

Represents a completed order. Auto-defined as `order` in email/SMS and fulfillment templates. Accessible via `user.orders` in the CMS.

**Filtering:** `id`, `code`, `status`
**Sorting:** `created_at`

**Status codes:**
- `P` = Pending, `W` = Draft, `C` = Confirmed, `F` = Payment Failed
- `D` = Downloaded, `M` = Manufactured, `S` = Shipped
- `X` = Cancelled, `E` = Error, `R` = Refunded

| Property | Description |
|---|---|
| `order.id` | Unique ID |
| `order.code` | Order code |
| `order.status` | One-letter status code |
| `order.promocode_code` | Associated promo code, if any |
| `order.total` | Order total. Use `currency` filter. |
| `order.orderlines_total` | Orderlines subtotal. Use `currency` filter. |
| `order.orderlines_discount` | Orderlines discount. Use `currency` filter. |
| `order.shipping_discount` | Shipping discount. Use `currency` filter. |
| `order.discount` | Total discount (= orderlines_discount + shipping_discount). Use `currency` filter. |
| `order.shipping` | Shipping cost. Use `currency` filter. |
| `order.extra_fees` | Array of `ExtraFee` objects |
| `order.tax` | Total tax. Use `currency` filter. |
| `order.email` | Buyer email |
| `order.telephone` | Buyer telephone |
| `order.first_name` | Buyer first name |
| `order.last_name` | Buyer last name |
| `order.payment_gateway` | Payment gateway name (`nil` if unpaid/manual) |
| `order.payment_reference` | Payment reference from gateway |
| `order.is_paid` | `true` if paid |
| `order.is_priority` | `true` if priority flag is set |
| `order.external_source` | External source name (`shopify`, `etsy`, etc.) if applicable |
| `order.external_reference` | External order reference if applicable |
| `order.created_at` | Cart session creation datetime. Use `date` filter. |
| `order.updated_at` | Last updated datetime. Use `date` filter. |
| `order.confirmed_at` | Confirmation datetime. Use `date` filter. |
| `order.manufactured_at` | Manufactured datetime. Use `date` filter. |
| `order.shipped_at` | Shipped datetime. Use `date` filter. |
| `order.shipping_method` | Shipping method name |
| `order.user_notes` | Customer notes (empty string if none) |
| `order.notes` | Admin notes (internal — do not display to customers) |
| `order.address` | `Address` object |
| `order.orderlines` | Paginated collection of `Orderline` objects (excludes children, default page size 50) |
| `order.all_orderlines` | Paginated collection of all `Orderline` objects including children (default page size 50) |
| `order.custom` | `CustomFields` object |
| `order.fulfillment_code` | Fulfillment code for the order |

---

## Orderline

Represents an orderline in a cart or order. Atomic commerce unit in Pixfizz.

**Filtering:** `id`, `is_draft`

| Property | Description |
|---|---|
| `orderline.id` | Unique ID |
| `orderline.quantity` | Quantity |
| `orderline.units` | Sum of quantities of all orderlines for the same product |
| `orderline.page_count` | Page count |
| `orderline.is_draft` | `true` if draft status |
| `orderline.is_cut_print` | `true` if cut print product |
| `orderline.cut_print_quantity` | Quantity of cut prints (for non-cut-print projects, equals page count — not `0`) |
| `orderline.unit_price` | Unit price. Use `currency` filter. |
| `orderline.price` | Orderline total price. Use `currency` filter. |
| `orderline.barcode` | Barcode (only available after fulfillment) |
| `orderline.product` | Associated `Product` object |
| `orderline.design` | Associated `Design` object, if any |
| `orderline.project` | Associated user `Project`, if any |
| `orderline.chosen_variants` | `ChosenOptions` object for product variants |
| `orderline.chosen_template_options` | `ChosenOptions` object for template options |
| `orderline.fulfillment_code` | Fulfillment setting code that would be used now |
| `orderline.external_files` | List of `ExternalFile` objects (deprecated) |
| `orderline.generated_files` | List of `GeneratedFile` objects (fulfillment templates only) |
| `orderline.children` | Paginated collection of child `Orderline` objects |

> `orderline.preview_url` is deprecated — use the `preview_url` filter instead.

---

## OptionType

Represents a product variant type or template option type.

**Filtering:** `id`, `code`, `name`, `type`, `custom.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

**Types:** `multiple_choice`, `text`, `number`, `color`, `image_upload`, `file_upload`

| Property | Description |
|---|---|
| `option_type.id` | Unique ID |
| `option_type.type` | Type string (see types above) |
| `option_type.name` | Name |
| `option_type.code` | Code |
| `option_type.description` | Description |
| `option_type.image` | `Asset` image, if any |
| `option_type.is_required` | `true` if required |
| `option_type.is_published` | `true` if published |
| `option_type.default_value` | Default value (number/color/font types only; `nil` otherwise) |
| `option_type.placeholder` | Placeholder text (text/number types; `nil` otherwise) |
| `option_type.min_length` | Min length (text only; `nil` otherwise) |
| `option_type.max_length` | Max length (text only; `nil` otherwise) |
| `option_type.pattern` | Pattern setting (text only; `nil` otherwise) |
| `option.min` | Min value (number only; `nil` otherwise) |
| `option.max` | Max value (number only; `nil` otherwise) |
| `option.step` | Step (number only; `nil` otherwise) |
| `option_type.accept` | Accept setting (file_upload only; `nil` otherwise) |
| `option_type.crop_aspect_ratio` | Crop aspect ratio (image_upload only; `nil` otherwise) |
| `option_type.color_palette` | Associated `ColorPalette`, if any (color type only) |
| `option_type.font_palette` | Associated `FontPalette`, if any (font type only) |
| `option_type.target_element_names` | Target element names as array (text/image_upload only; `nil` otherwise) |
| `option_type.has_pricing` | `true` if this option influences price |
| `option_type.has_element_substitutions` | `true` if this option or any children trigger element substitutions |
| `option_type.trigger_value` | `OptionValue` that triggers this child option (`nil` if not a child) |
| `option_type.children` | Paginated collection of published child `OptionType` objects |
| `option_type.values` | List of published `OptionValue` objects (multiple_choice only; `nil` otherwise) |
| `option_type.all_values` | List of all (including unpublished) `OptionValue` objects (multiple_choice only) |
| `option_type.custom` | `CustomFields` object |

---

## OptionValue

Represents an available value for a multiple choice variant or template option.

| Property | Description |
|---|---|
| `option_value.id` | Unique ID |
| `option_value.name` | Name |
| `option_value.code` | Code |
| `option_value.description` | Description |
| `option_value.is_default` | `true` if this is the default value |
| `option_value.is_published` | `true` if published |
| `option_value.price` | Price. Use `currency` filter. |
| `option_value.image` | `Asset` image, if any |
| `option_value.custom` | `CustomFields` object |

---

## Paginate

Represents a paginated collection. Returned by most collection properties (e.g. `user.orders`, `cart.orderlines`). Default page size is usually 20 (cart.orderlines defaults to 50).

Items on the current page can be iterated with `{% for %}`.

| Property | Description |
|---|---|
| `paginate.current_page` | Current page number (defaults to `1`) |
| `paginate.current_offset` | Offset of first item on current page |
| `paginate.total_items` | Total items in entire collection (same as `paginate.size`) |
| `paginate.total_pages` | Total available pages |
| `paginate.page_size` | Items per page |
| `paginate.first` | First item on current page |
| `paginate.last` | Last item on current page |
| `paginate.size` | Total items in collection |

**Compatible filters:** `first`, `last`, `size`, `map`, `sort`, `where`, `page`, `page_size`, `reverse`

```liquid
{% for order in user.orders %}
  {{ order.code }}
{% endfor %}

{% assign lines = cart.orderlines | page_size: 1000 %}
{% for line in lines %}
  {{ line.product.name }}
{% endfor %}
```

---

## Product

Represents a product (equivalent to "Manage Products → Product Attributes" in admin). Auto-defined as `product` when URL contains `?product=<product_id>`.

**Filtering:** `id`, `code`, `name`, `category`, `custom.<field-name>`, `template.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

| Property | Description |
|---|---|
| `product.id` | Unique ID |
| `product.code` | Product code |
| `product.name` | Name |
| `product.description` | Description |
| `product.category` | Category |
| `product.price` | Calculated price. Use `currency` filter. |
| `product.is_static` | `true` if static product |
| `product.is_deleted` | `true` if deleted from admin |
| `product.min_quantity` | Minimum quantity |
| `product.max_quantity` | Maximum quantity |
| `product.quantity_intervals` | Array of available quantity values, or `nil` |
| `product.tracks_inventory` | `true` if inventory tracking is enabled |
| `product.current_inventory` | Current inventory count |
| `product.image` | `Asset` image |
| `product.preview_images` | Paginated collection of `Asset` preview images |
| `product.template` | Associated `Template` object (`nil` for static products) |
| `product.variants` | Paginated collection of published top-level `OptionType` objects |
| `product.all_variants` | Paginated collection of all `OptionType` objects (all levels, including unpublished) |
| `product.custom` | `CustomFields` object |

---

## Project

Represents a user-created project (book, print set, etc.).

**Filtering:** `id`, `is_saved`, `is_ordered`, `share_code`
**Sorting:** `created_at`, `updated_at`

| Property | Description |
|---|---|
| `project.id` | Unique ID |
| `project.name` | Project name |
| `project.created_at` | Creation datetime. Use `date` filter. |
| `project.updated_at` | Last updated datetime. Use `date` filter. |
| `project.share_code` | Share code |
| `project.page_count` | Page count (excludes `count="false"` pages) |
| `project.uncounted_page_count` | Count of `count="false"` pages |
| `project.double_page_count` | Count of double (`position="left-right"`) pages |
| `project.uncounted_double_page_count` | Count of pages that are both double and uncounted |
| `project.cut_print_quantity` | Total cut prints in project (equals page count for non-cut-print projects) |
| `project.is_cut_print` | `true` if cut print project |
| `project.is_saved` | `true` if explicitly saved by user |
| `project.is_ordered` | `true` if this is a copy added to cart (does not guarantee actual order placed) |
| `project.weight` | Project weight (requires template setup; `0` if not configured) |
| `project.price` | Project price. Use `currency` filter. |
| `project.preview_page_numbers` | Array of 1-based page numbers marked with `preview="true"` |
| `project.editor_page_numbers` | Array of 1-based page numbers not marked `editor="false"` |
| `project.fulfillment_page_numbers` | Array of 1-based page numbers not marked `fulfillment="false"` |
| `project.product` | Associated `Product` object |
| `project.design` | Associated `Design` object |
| `project.chosen_variants` | `ChosenOptions` for product variants |
| `project.chosen_template_options` | `ChosenOptions` for template options |
| `project.images` | Array of `Image` objects for each editable image used |
| `project.custom` | `CustomFields` object |

**Preview in email (requires share code):**
```liquid
https://{{ website.hostname }}{{ orderline.project | preview_url: height: 300, share: orderline.project.share_code }}
```

---

## Request

Contains information about the current HTTP request. Available globally as `request` in the CMS only (not in email/SMS/fulfillment).

| Property | Description |
|---|---|
| `request.url` | Current URL |
| `request.base_url` | Scheme + host (with optional port) |
| `request.scheme` | `http` or `https` |
| `request.host` | Host of the request |
| `request.port` | Port |
| `request.path` | Path |
| `request.fullpath` | Path + query string |
| `request.query_string` | Query string |
| `request.params` | Proxy for query parameters. Access with `request.params.book_id` or `request.params['param-name']` |
| `request.locale` | Two-letter code of active language |

---

## ShippingOption

Contains information about a shipping option for the cart.

| Property | Description |
|---|---|
| `shipping_option.id` | Unique ID |
| `shipping_option.name` | Name |
| `shipping_option.code` | Code |
| `shipping_option.cost` | Cost. Use `currency` filter. |

---

## Template

Contains information about a product template.

**Filtering:** `id`, `code`, `name`, `category`, `custom.<field-name>`
**Sorting:** `id`, `custom.<field-name>`

| Property | Description |
|---|---|
| `template.id` | Unique ID |
| `template.code` | Code |
| `template.name` | Name |
| `template.description` | Description |
| `template.category` | Category |
| `template.min_page_count` | Minimum pages |
| `template.max_page_count` | Maximum pages |
| `template.default_page_count` | Default page count for new projects |
| `template.page_increments` | Pages added/removed per increment |
| `template.dpi` | Output DPI |
| `template.minimum_dpi` | Minimum DPI (used for low-res warnings) |
| `template.width` | Width of first page (variable width = minimum width; binding not included) |
| `template.height` | Height of first page (variable height = minimum height) |
| `template.is_cut_print` | `true` if cut print template |
| `template.min_cut_print_quantity` | Min prints required before add-to-cart (cut print only; `nil` otherwise) |
| `template.max_cut_print_quantity` | Max prints allowed (`0` = no limit; cut print only; `nil` otherwise) |
| `template.is_deleted` | `true` if deleted from admin |
| `template.preview_images` | Paginated collection of `Asset` preview images |
| `template.mapped_previews` | Paginated collection of `MappedPreview` objects |
| `template.custom` | `CustomFields` object |

---

## UploadedFile

Represents a file uploaded to a Pixfizz "File Upload" variant or template option.

**Filtering:** `id`, `code`
**Sorting:** `created_at`

| Property | Description |
|---|---|
| `uploaded_file.id` | Unique ID |
| `uploaded_file.filename` | Original filename |
| `uploaded_file.url` | File URL |
| `uploaded_file.created_at` | Creation datetime. Use `date` filter. |

**Resized delivery.** A Pixfizz-hosted file URL accepts a `thumbnail/{n}` path
segment, where `n` is the rendered size in pixels — `.../thumbnail/250/...`. It
is a **path** parameter, not a query parameter; `?height=N` has no effect. Use it
anywhere a full-size customer upload is displayed at thumbnail scale (cart lines,
gallery strips, review modals). On customer artwork the saving is typically an
order of magnitude. Shopify gallery previews use the same segment — see
**60_SHOPIFY_INTEGRATION.md**.

---

## User

Contains information about the current website user. Available globally as `user` in CMS, email/SMS templates, and fulfillment templates.

| Property | Description |
|---|---|
| `user.id` | Unique ID |
| `user.is_anonymous` | `true` if anonymous user |
| `user.is_logged_in` | Reverse of `is_anonymous` (also `true` for guest users) |
| `user.is_guest` | `true` if guest user |
| `user.is_admin` | `true` if admin |
| `user.external_id` | External ID (`nil` if not set) |
| `user.email` | Email |
| `user.first_name` | First name |
| `user.last_name` | Last name |
| `user.telephone` | Telephone |
| `user.category` | User category |
| `user.projects` | Paginated collection of saved `Project` objects |
| `user.all_projects` | Paginated collection of all `Project` objects (including unsaved and cart copies) |
| `user.signup_address` | Signup `Address` object |
| `user.addresses` | Paginated collection of stored `Address` objects |
| `user.orders` | Paginated collection of `Order` objects |
| `user.carts` | Paginated collection of `Cart` objects |
| `user.calendars` | Paginated collection of `Calendar` objects |
| `user.groups` | Paginated collection of `Group` objects the user belongs to |
| `user.impersonating_admin_user` | Admin `User` currently impersonating this user (`nil` if not impersonated) |
| `user.custom` | `CustomFields` object |

---

## Website

Contains general website information. Available globally as `website` in all rendering contexts.

| Property | Description |
|---|---|
| `website.title` | Website title |
| `website.hostname` | Public hostname |
| `website.pixfizz_subdomain` | The `*.pixfizz.com` subdomain |
| `website.currency_code` | Currency code (`USD`, `EUR`, etc.) |
| `website.currency_sign` | Currency sign (`$`, `€`, etc.) |
| `website.currency_formatting_settings` | `CurrencyFormattingSettings` object (renders as JSON) |
| `website.locale` | Two-letter default language code |
| `website.assets` | Access assets by name: `website.assets['logo.png']` |
| `website.collections` | Paginated collection of top-level `Collection` objects |
| `website.all_collections` | Paginated collection of all `Collection` objects ordered alphabetically by path |
| `website.design_products` | Paginated collection of all `DesignProduct` objects |
| `website.static_products` | Paginated collection of all static `Product` objects |
| `website.public_addresses` | Paginated collection of top-level public `Address` objects |
| `website.galleries` | Paginated collection of published public `Gallery` objects |
| `website.calendars` | Paginated collection of admin-created `Calendar` objects |
| `website.countries` | Paginated collection of all `Country` objects (default page size 300) |
| `website.custom_types` | `CustomTypes` proxy object |
| `website.dropbox_app_key` | Dropbox app key |
| `website.google_oauth2_client_id` | Google OAuth 2.0 client ID |
| `website.stripe_publishable_key` | Stripe Publishable Key |
| `website.square_application_id` | Square application ID (production or sandbox) |
| `website.square_location_id` | Square location ID |
| `website.square_sandbox_mode` | `false` if live Square; `true` otherwise |
| `website.authorizedotnet_api_login_id` | Authorize.net API Login ID |
| `website.authorizedotnet_public_client_key` | Authorize.net Public Client Key |
| `website.authorizedotnet_sandbox_mode` | `true` if Authorize.net sandbox mode |

---

# FILTERS

---

## asset_url

Renders a URL to a website asset. Input: asset name string or `Asset` object.

```liquid
{{ 'logo.png' | asset_url }}
{{ website.assets['hero.jpg'] | asset_url: 800 }}
{{ 'logo.png' | asset_url: '300x150' }}
{{ collection.image | asset_url }}
{{ 'image.jpg' | asset_url: cdn: false }}
{{ 'image.jpg' | asset_url: 150, cdn: false }}
{{ 'mypic.png' | asset_url: size: 250 }}
{{ 'mypic.png' | asset_url: 500, format: 'webp' }}
```

**Parameters:**
- Size: single number or `'WxH'` string
- `format`: `"jpeg"`, `"png"`, or `"webp"` (images only)
- `cdn: false` — disable CDN (use for OG/metadata/ld+json URLs)

---

## barcode_datauri

Returns a PNG data-URI of an EAN13 barcode for the given input number.

```liquid
{{ '978020137962' | barcode_datauri }}
```

---

## cms_url

Returns a URL to a CMS page. Used to build URLs for collection and product pages.

```liquid
{{ 'contact' | cms_url }}
{{ my_collection | cms_url }}
{{ my_collection | cms_url: page: 'kolekzion' }}
{{ my_design_product | cms_url, collection: my_collection }}
{{ my_product | cms_url: page: 'produkt' }}
```

**Input types accepted:** page name string, `Page` object, `Collection` object, `DesignProduct` object, `Product` object.
**Parameters:** `page`, `collection`, `product`, `design`, `design_product`

---

## currency

Formats a number as a price using website currency settings (configured in Admin → Website → Config → Currency Formatting).

```liquid
{{ 1234567890.50 | currency }}
{{ 1234567890.50 | currency: format: '%u %n' }}
{{ 1234567891.50 | currency: precision: 0 }}
{{ 1234567890.50 | currency: unit: '€', separator: ',', delimiter: '' }}
```

**Parameters:** `format` (`%u` = unit, `%n` = number), `negative_format`, `unit`, plus all `number` filter parameters.

---

## design_tool_url

Returns the design tool (editor) URL path for a given `Project`.

```liquid
{{ project | design_tool_url }}
{{ project | design_tool_url: cart: 't' }}
```

---

## escape_json

Escapes a string for use inside JSON by replacing double quotes, newlines, etc. with escaped versions. Still requires wrapping in double quotes in JSON.

```liquid
var json = ["{{ my_string | escape_json }}"];
```

---

## page

Sets the current page of a `Paginate` object. Defaults to the `page` URL parameter, or `1` if absent.

```liquid
{% assign countries = website.countries | page: 1 %}
{% assign orders = user.orders | page: request.params['order-page'] %}
```

---

## page_size

Sets the page size of a `Paginate` object.

```liquid
{% assign lines = cart.orderlines | page_size: 1000 %}
```

---

## parse_json

Parses a JSON string into a Liquid object.

```liquid
{% assign json = json_string | parse_json %}
{{ json.a[1] }}
{{ json.b.c }}
```

---

## preview_url

Returns a preview URL path for a `Project`. Accepts optional keyword parameters added as query parameters.

```liquid
{{ project | preview_url }}
{{ project | preview_url: height: 250 }}
{{ project | preview_url: width: 100, ts: 123456 }}

{# For email use — requires share code: #}
{{ orderline.project | preview_url: height: 300, share: orderline.project.share_code }}
```

---

## push

Adds a single item to an array (similar to `concat` but for one item).

```liquid
{% assign colors = 'red green blue' | split: ' ' %}
{% assign colors = colors | push: 'pink' %}
```

---

## sort

Enhances Liquid's built-in `sort` to work with `Paginate` objects. Sorts efficiently in the database. Ascending by default; use `reverse` for descending.
```liquid
{% assign countries = website.countries | sort: 'code' %}
{% assign last_orders = user.orders | sort: 'created_at' %}
```

**Important limitation:** `sort` only works reliably when applied to a `Paginate` object (i.e. a collection returned directly from the platform, such as `website.custom_types[...]`, `user.orders`, etc.). It does **not** work reliably on plain arrays built via `push`. Nested dot-notation sort keys (e.g. `'custom.blog_date'`) fail silently on push-built arrays — the array is returned in unpredictable order with no error thrown.

**Rule:** Always apply `sort` at initial collection assignment. Never apply it to a push-built array.

---

## t

Translates a string using translations configured in the Pixfizz Admin.

```liquid
{{ 'Monday' | t }}
{{ 'Monday' | t: ns: 'phonetic' }}
{{ 'Monday' | t: locale: 'es' }}
```

---

## thumbnail_url

Returns a URL to a thumbnail version of an `Image` object. Defaults to 250px.

```liquid
<img src="{{ image | thumbnail_url | escape }}" />
<a href="{{ image | thumbnail_url: 1024 | escape }}">
```

---

## where

Enhances Liquid's built-in `where` to work with `Paginate` objects. Filters efficiently in the database.

```liquid
{% assign usa = website.countries | where: 'code', 'US' | first %}
{% assign confirmed_orders = user.orders | where: 'status', 'C' %}
```

Supports drill-down filtering: `'design.template.category', 'Canvases'`
Supports custom field filtering: `'custom.express_service', 'true'`

---

## Other standard filters (Pixfizz-extended)

| Filter | Description |
|---|---|
| `first` | Extended to work with `Paginate` objects |
| `last` | Extended to work with `Paginate` objects |
| `size` | Extended to work with `Paginate` objects |
| `map` | Extended to work with `Paginate` objects |
| `reverse` | Extended to work with `Paginate` objects (reverses sort order) |
| `convert_base` | Converts a number from one base to another |
| `hmac_sha1` | HMAC-SHA1 hash |
| `hmac_sha256` | HMAC-SHA256 hash |
| `md5` | MD5 hash |
| `sha1` | SHA1 hash |
| `sha256` | SHA256 hash |
| `number` | Formats a number with precision/separator/delimiter params |
| `pixfizz_asset_url` | Pixfizz internal asset URL variant |
| `json_parse` | Parses a JSON-formatted string into a Liquid object |

---

# TAGS

---

## snippet

Renders a CMS snippet by name. Additional keyword parameters become variables inside the snippet.

```liquid
{% snippet 'image-thumbnail', image_src: '/path/to/image.jpg', color: '#ffaabb' %}
{% snippet 'my-snippet', fallback_content: '' %}
```

**Note:** If the snippet is not found, an error is thrown unless `fallback_content` is provided.

**CMS import behaviour:** The CMS importer silently skips any snippet file that contains a Liquid syntax error. The snippet will simply not exist after import — no error is shown during the import process itself. This makes syntax errors in snippet files especially dangerous: they appear as mysterious "Snippet not found" errors at render time, even though the file was present in the tar.

---

## form

Wraps form submissions. The `form` object ([Form](/2f9228d5a61e4e9683f742facbcda109)) is available inside the block as `form`, giving access to `form.errors`, `form.values`, `form.captcha`, etc.

The tag renders the HTML `<form>` element with appropriate attributes. You are responsible for including control elements (`<input>`, `<textarea>`, `<select>`, ...) and buttons.

### Generic Parameters

All form types accept these optional parameters:

| Parameter | Description |
|---|---|
| `page` | CMS page name to redirect to after successful submission |
| `target` | Full URL path (with optional query string) to redirect to after success. Takes precedence over `page`. |
| `autosubmit` | When `true`, auto-submits on any contained element's `change` event |
| `async` | When `true`, submits asynchronously and updates HTML without page reload |
| `selectors` | CSS selector(s) for partial page update after async submission. Only use with `async: true`. |

Any other parameter is rendered as an HTML attribute on the generated `<form>` element.

### Form Types

**Contact & Auth**

| Form Type | Required Params | Description |
|---|---|---|
| `contact` | — | Contact form. Use `{{ form.captcha }}` inside. |
| `user_login` | — | User login form |
| `user_registration` | — | User registration form |
| `user_update` | — | User update form |
| `forgotten_password` | `reset_page:` or `reset_target:` | Forgotten password form. Points to the page containing the reset form. |
| `password_reset` | — | Password reset form |
| `locale_set` | — | Sets user session language. Include input with `name="locale"` and two-letter code value. |

**Cart**

| Form Type | Required Params | Description |
|---|---|---|
| `cart_add_product` | `product:` | Adds the given Product to the cart |
| `cart_set_shipping_option` | — | Sets the cart's shipping option |
| `cart_apply_promocode` | — | Applies a promocode to the cart |
| `cart_remove_promocode` | — | Removes the current promocode from the cart |
| `cart_update` | — | Updates the cart |
| `cart_payment` | *`gateway:`, *`onsuccess:`, *`oncancel:` | Cart payment form. Optional params are PayPal-only. |
| `cart_clear` | — | Removes all orderlines from the current cart. The cart stays assigned to the session; promocode, address and other cart properties are kept on the now-empty cart, and it remains in `user.carts`. |
| `cart_unset` | — | Removes the current cart from the session. The cart and its orderlines are NOT deleted and remain accessible under `user.carts`. |
| `cart_delete` | *`cart:` | Destroys a cart. With no `cart:` param it deletes the current session cart; pass an existing cart to delete that one instead. Deletion is asynchronous — the cart can take a moment to disappear from `user.carts`. |

**Cart teardown — clear vs unset vs delete**

These three forms are easy to confuse. Choose by what you want to keep:

- `cart_clear` — empties the cart (removes orderlines) but keeps the cart itself, including its promocode and address. Use when the shopper wants to start over but stay in the same cart.
- `cart_unset` — detaches the current cart from the session without deleting anything. A fresh cart is started on the next add-to-cart, and the old one is still listed in `user.carts`. Use for "save this cart for later / start a new one".
- `cart_delete` — permanently destroys a cart. Deletion runs asynchronously, so it may still appear in `user.carts` briefly after submission. On a "My carts" page, delete a specific saved cart by passing it in: `{% form 'cart_delete', cart: cart %}`.

**Writing cart custom fields**

`cart_update` is the form type that writes to `cart.custom`. Name the input
`cart[custom][<field_name>]`:

```liquid
{% form 'cart_update', async: true, autosubmit: true %}
	<input type="hidden" name="cart[custom][order_source]" value="{{ source }}">
{% endform %}
```

Three conditions apply:

- The field must already exist as a custom field on that site. Custom fields do
  not inherit parent to child.
- Where the value comes from a helper snippet capture, always `| strip` before
  comparing. Helper snippets render with a trailing newline; an unstripped
  comparison never matches, which turns a guarded auto-submit into a submit loop.
- Guard on cart state before firing. A hidden auto-submitting `cart_update`
  against an empty cart is wasted work at best; gate on
  `cart.orderlines.size > 0` and on the stored value being absent or stale.

**Projects**

| Form Type | Required Params | Description |
|---|---|---|
| `project_create` | `product:`, `design:`, *`parent_orderline:` | Creates a project. Pass `parent_orderline` for child orderlines. |
| `project_update` | `project:` | Updates a project's attributes |

**Addresses**

| Form Type | Required Params | Description |
|---|---|---|
| `address_create` | *`assign_to_user:`, *`assign_to_cart:` | Creates an address. By default the new address is set on the current cart **and** saved to the user's saved addresses. Pass `assign_to_user: false` and/or `assign_to_cart: false` to suppress either behaviour. |
| `address_update` | `address:` | Updates the given Address |
| `address_delete` | `address:` | Deletes the given Address |

**Orders & Orderlines**

| Form Type | Required Params | Description |
|---|---|---|
| `orderline_update` | `orderline:` | Updates the given Orderline |
| `orderline_commit` | `orderline:` | Commits a draft Orderline. If it's the last draft in the order and order is "Draft", order moves to "Confirmed". |
| `orderline_delete` | `orderline:` | Deletes the given Orderline |
| `order_payment` | `order:`, *`gateway:`, *`onsuccess:`, *`oncancel:` | Pays the given Order. Optional params are PayPal-only. |

**Galleries & Images**

| Form Type | Required Params | Description |
|---|---|---|
| `gallery_create` | — | Creates a new gallery |
| `gallery_update` | `gallery:` | Updates the given Gallery |
| `gallery_delete` | `gallery:` | Deletes the given Gallery |
| `image_create` | `gallery:` | Uploads an image to the given Gallery |
| `image_delete` | `image:` | Deletes the given Image |

**Calendars**

| Form Type | Required Params | Description |
|---|---|---|
| `calendar_create` | — | Creates a new calendar |
| `calendar_update` | `calendar:` | Updates the given Calendar |
| `calendar_delete` | `calendar:` | Deletes the given Calendar |
| `calendar_date_create` | `calendar:` | Creates a new date in the given Calendar |
| `calendar_date_update` | `calendar:`, `date:` | Updates a CalendarDate in the given Calendar |
| `calendar_date_delete` | `calendar:`, `date:` | Deletes a CalendarDate from the given Calendar |

**Other**

| Form Type | Required Params | Description |
|---|---|---|
| `signin_token_renew` | `signin_token:` | Renews a soft-expired SigninToken |

*Parameters marked with * are optional.*

---

## redirect_to

Redirects to another URL.

---

## return_404

Returns a 404 response.

---

## signin_token / promocode / braintree_client_token

Specialized tags for authentication tokens, promo codes, and payment setup.

---

# GLOBAL VARIABLES SUMMARY

| Variable | Available In | Description |
|---|---|---|
| `cart` | CMS | Current user's cart |
| `user` | CMS, email, SMS, fulfillment | Current user |
| `website` | All contexts | Website object |
| `request` | CMS only | Current HTTP request |
| `order` | Email, SMS, fulfillment | Order that triggered the template |
| `collection` | CMS (collection URLs) | Current collection |
| `product` | CMS (`?product=<id>`) | Current product |
| `design` | CMS (`?theme=<id>`) | Current design |
| `form` | Inside `{% form %}` blocks | Form errors and submitted values |
| `orderlines` | Fulfillment templates | Orderlines being fulfilled |

---

## Notes on Contexts

- **CMS** (pages, layouts, snippets): Full access to all objects and filters.
- **Email/SMS templates**: `order`, `user`, `website` available. No `request`, no `cart`.
- **Fulfillment templates**: `order`, `orderlines`, `user`, `website` available. `orderline.generated_files` only available here.
- Project previews in email **must include the share code**: `share: orderline.project.share_code`

---

# KNOWN CMS LIQUID QUIRKS

These behaviours differ from standard Liquid or Shopify Liquid. Confirmed through testing on the Pixfizz CMS.

| What doesn't work | Use instead |
|---|---|
| `!= blank` | `!= ''` or `.size > 0` |
| `== false` (explicit boolean comparison) | Use the truthy/falsy pattern: `{% unless var %}` |
| `nil == false` | `false` — nil and false are both falsy but not equal. Use `!= true` instead of `== false` |
| `== 'true'` on boolean Custom Type field | Use `{% if instance.custom.field_name %}` directly — boolean fields evaluate as truthy/falsy |
| `{% assign var = nil %}` | `{% assign var = '' %}` (then test with `!= ''`) |
| Checklist snippet capture without `strip` | Always pipe through `strip` after capture — snippet renders with trailing newline that breaks `== 'TRUE'` |
| `{% if product != blank %}` to detect a nil object | Test an attribute that always exists on a real one: `{% if product.id %}`. Nil yields nil, which is falsy in every engine. `!= blank` is not portable and can take the "it exists" branch for an object that is nil |
| `product.custom.x != blank` to test whether a field was set | `{% assign v = product.custom.x \| default: '' \| strip %}{% if v != '' %}`. Comparing against the empty string behaves identically everywhere; `!= blank` does not |

---

# RECENT PLATFORM ADDITIONS

These are Liquid-related capabilities added to the platform recently. They are not yet
documented on the master Notion reference and may still be evolving — confirm against
a live site before depending on specific syntax.

---

## Liquid scripts can set custom order fields

**Shipped, 2026-03-23.**

Liquid scripts can now write values into custom order fields. This allows a script to
compute something at order time (e.g. a production deadline, a routing key, a split
SKU) and persist it onto the order for downstream use — the order template, the
admin, exports, fulfillment transformations, etc.

**Use case seen in production:** a live client site — a Liquid script calculates the
production deadline from the product, shipping option, and current date, writes it
into a custom order field, and the value is then displayed on the order detail page
and used by the production workflow.

Treat this as a genuinely new Liquid capability: before this change, custom order
fields could only be set at add-to-cart time via form inputs or by the user at
checkout. They can now be written programmatically at any point a Liquid script
runs on an order.

Canonical snippet pending — capture from the reference site once stable.

---

## Product file-name Liquid templates

**Shipped, 2026-03-31.**

The product file-naming Liquid template (used when generating production file names
for a product) now supports:

- **Capitalization control** — force upper/lower/title case on substituted values.
- **Dynamic dash insertion** — insert separators between variable pieces without
  hard-coding them in the template literal.

Used on client sites for file naming that needs to match a specific
fulfillment partner's naming convention. Exact syntax to be captured from the TK
site and added here once confirmed.

---

## Configurable product display names via Liquid

**Shipped, 2026-03-03.**

Product display names can now be configured via Liquid, so the title shown in
product lists and on the product page can be generated dynamically rather than
stored as a static string.

**Primary use case:** dynamic inventory messaging. Combined with the new inventory
tracking feature, a product title can append `"Only 3 left"` or `"Out of stock"`
based on current stock level, without duplicating the product or maintaining
parallel copies.

Pair with the inventory tracking feature (see `18_ADMIN_NAVIGATION.md`) for the
complete pattern.

---

## Meta title inheritance — per-product disable checkbox

**Shipped, 2026-01-26.**

A new checkbox exists on the product admin to **disable design meta title inheritance**
for that product. Use this when a template's meta title is unintentionally overwriting
a product's SEO-facing title.

When hit: the symptom is a product whose `<title>` in the rendered head comes from
the associated design template rather than from the product's own SEO title. Tick
the checkbox to pin the product's own title.

---

## UTM capture snippet

**Shipped, 2026-03-24** (initial client deployment).

A reusable Liquid snippet that captures incoming UTM parameters (`utm_source`,
`utm_medium`, `utm_campaign`, etc.) into user state so they persist through the
session and can be pushed into analytics, order custom fields, or Meta Pixel /
GA4 events.

Use this whenever a site needs marketing source attribution that survives the
user navigating away from the landing URL. Final canonical version still
stabilizing — pull from the reference deployment when adding to a new site.

---

## Meta Pixel + GA4 — debug mode gotcha

**Recurring gotcha (2026-03-24).**

When setting up Meta Pixel alongside GA4 on a Shopper site, **remember to
disable GA4 debug mode** before going live. GA4 debug mode (enabled via the
GTM Preview or a `?gtm_debug=1` parameter) filters events out of the
production property, so launched sites can appear to be capturing zero
conversions even though everything else is wired correctly.

Checklist when adding analytics to a Shopper site:
- Meta Pixel base code injected via Custom JS or a head snippet.
- GA4 tag firing on page view, add-to-cart, purchase.
- `debug_mode: false` (or the debug parameter stripped) on production.
- Test a real transaction end-to-end and confirm events land in both the
  Meta Events Manager and the GA4 DebugView → then Realtime report.

For server-side GA4 tracking via webhook, see `18_ADMIN_NAVIGATION.md`
under Settings > Webhooks.

---

## Blog chronological sort — gotcha

Custom blog / listing pages built on top of a CMS collection **do not sort
chronologically by default**. This has been hit on multiple sites (most recently
a client site, 2026-03-12) and is an easy thing to miss because the admin list in
the CMS *is* sorted by date, so the issue only surfaces on the public page.

**Fix options:**
- Set an explicit sort order on the collection in the CMS admin, or
- Apply a Liquid `sort` / `reverse` filter on the listing loop (`sorted_by: 'created_at'`
  / reverse as appropriate).

Always verify the public page order after building a blog — do not trust that
"date-descending" is the default.

---

## Save-and-exit in add-to-cart for limited-option products

**Pattern, not a new feature.**

For products with a small number of customisable options (e.g. name-only cards,
single-photo prints), the "Save & Exit" action from the design tool can be wired
directly into the add-to-cart flow so the customer does not have to re-enter the
design tool to proceed. Used on a client site, 2026-03-12.

This is a Shopper UX pattern, not a platform capability — implemented at the
snippet level on the product page.

---

## Checkout column capture-block refactor

**Canonical deduplication pattern for Shopper checkout (2026-01-26).**

Matjaz's refactor of the checkout-page columns removed duplicated markup between
the left and right columns by wrapping the shared regions in `{% capture %}` blocks
and rendering the captured content in both places.

This is the recommended way to deduplicate repeated block regions in checkout
templates. See `50_SHOPPER_TEMPLATE_REFERENCE.md` for the detailed pattern and
example markup.

## Changelog
- 2026-06-01: Noted Shopify IDs live in chosen_variants. Source: claude-chat.
- 2026-06-15: Added json_parse filter to Pixfizz-extended filters. Added assign_to_user / assign_to_cart optional params to the address_create form. Source: notion-dashboard.
- 2026-07-07: Documented cart_clear, cart_unset and cart_delete forms in the Cart forms table, plus a clear/unset/delete comparison note. Source: notion-page, slack-message.
- 2026-07-28: Added file_upload accessor note on ChosenOption (`uploaded_file.url` / `.filename`; `value`, `asset.url`, `thumbnail_url` and the `preview_url` filter do not work), the `thumbnail/{n}` path segment on UploadedFile, and the `cart[custom][field]` write pattern for `cart_update`. Source: claude-chat.
- 2026-08-21: Added FORMS section with address_create form options (assign_to_user, assign_to_cart). Source: notion-page (Dashboard 🆕 Update).


=================================================================
FILE: 50_SHOPPER_TEMPLATE_REFERENCE.md
=================================================================

# 50 — Shopper Template Reference

**Authority Scope:** Structural anatomy of the Shopper parent template — layouts, navigation, snippets, theming, CSS delivery, and admin checklist system. Derived from a full CMS backup scan (2026-03-12).

_Last updated: 2026-06-01_

---

## What this file covers

This reference documents the **actual Shopper template structure** as found in the parent site backup. It covers:
- Layout files and when each is used
- Navigation system (styles, megamenu, checklist control)
- Snippet namespace conventions
- Theming system (style snippets + CSS delivery)
- Admin checklist system (how it works + full key inventory)
- JavaScript library stack
- Key page inventory

This is **structural knowledge** — not behavioral rules. For behavior and logic, consult files 20–22.

---

## 1. Layouts

Five layouts exist. The `index` layout is the default for all storefront pages.

| Layout | Used for |
|---|---|
| `index` | Default storefront layout (all public-facing pages) |
| `admin` | Custom admin pages (`/site/admin/*`) — admin-only, has side nav |
| `shopper-admin` | Shopper v2 custom admin (`/site/manage/*`) — admin-only, sidebar nav, requires `cms.js` + `cms.css` for form submission |
| `order-management` | Custom admin without side nav (standalone admin views) |
| `quickstart` | Pixfizz setup/onboarding wizard — admin-only, has full side nav |
| `iframe` | Modal/iframe content only |
| `feed_xml` | XML feed pages (product feeds) |

### `index` layout structure

The `index` layout assembles the storefront page in this order:

1. GTM noscript (if GTM enabled)
2. Global modals (always present — do not remove):
   - `modals/password-reset`
   - `modals/shopping-cart`
   - `modals/cart-notification`
   - `modals/search`
   - `modals/upload`
   - `modals/login`
   - `modals/warning`
   - `modals/promotions`
   - `modals/proof`
   - `modals/skip-proof`
3. Optional home-page login gate (`admin/checklist/home-page-login-form`)
4. Optional top promotion bar (`admin/checklist/top-promotion-bar`)
5. Navigation (determined by `admin/checklist/header-logo-position`)
6. `{{ page.content }}`
7. Optional back-to-top (`admin/checklist/back-to-top`)
8. Optional GDPR banner
9. Footer (`snippets/footer`)
10. Third-party scripts (ShareMe, chatbot, cookie consent, Klaviyo, Constant Contact)
11. JavaScript library stack

### JavaScript library stack (index layout)

All loaded via `asset_url` at the bottom of `<body>`:
- `jquery.min.js`
- `jquery.fancybox.min.js`
- `bootstrap.bundle.min.js`
- `flickity.pkgd.min.js`
- `highlight.pack.min.js`
- `jarallax.min.js`
- `list.min.js`
- `simplebar.js`
- `smooth-scroll.min.js`
- `flickity-fade.js`
- `theme.min.js`
- Then: `integrations/custom-body-scripts`

---

## 2. Navigation System

### Navigation styles

The layout selects a navigation snippet based on `admin/checklist/header-logo-position`:

| Checklist value | Snippet used | Layout |
|---|---|---|
| `LEFT` | `navigation/style1` | Logo left, center menu, icons right — single row |
| `CENTER` (default) | `navigation/style3` | Two rows: row 1 = logo center + social icons + account/cart; row 2 = main nav menu |
| `CUSTOM` | `navigation/style1` or `navigation/logo-left` | Conditional on `user.category == 'beta'` |

Additional styles exist (`style2`, `style4`, `style5`) but are not currently wired to the checklist — developmental/alternative variants.

### style1 — Logo Left (single row)

- Logo on the left
- Nav links centered (`mx-auto`)
- Account icon + cart icon on the right
- Suppresses nav when `admin/checklist/clean-checkout == 'TRUE'`
- Account dropdown shows: Custom Admin (admins only), Saved Projects, Orders, Galleries, Personal Info, Logout

### style3 — Two-Row Center Logo (default)

- Row 1: Social media icons (left, `d-none d-lg-flex`) + center logo (absolute positioned) + account/cart (right)
- Row 2 (`nav.main-menu`): Product navigation links centered
- Row 2 suppressed on `page.url == 'checkout-single-page'`
- Mobile: logo left + hamburger toggler; nav links collapse into mobile accordion
- Suppresses nav when `admin/checklist/clean-checkout == 'TRUE'`

### Navigation links (parent defaults)

Both style1 and style3 define the same default nav link set in a `{% capture navigation_links %}` block:

- Prints → `navigation/megamenu/prints`
- Wall Art → `navigation/megamenu/wall-art`
- Stationery → `navigation/megamenu/stationery`
- Photo Books → `navigation/megamenu/photo-books`
- Gifts → `navigation/megamenu/gifts`
- Lab Services → `navigation/megamenu/services`
- Business → `navigation/megamenu/business` (hidden: `d-none`)

**Client sites override this by editing the nav style snippet directly** — the `{% capture navigation_links %}` block at the top of the snippet is the right place to make those edits.

### `clean-checkout` flag

When `admin/checklist/clean-checkout == 'TRUE'`, both nav styles suppress the main navigation links, leaving only the logo and cart icon visible. Used for a distraction-free checkout experience.

---

## 3. Megamenu System

### How megamenus work

Each nav item is a Bootstrap dropdown with `position-static`. The dropdown content is a full-width panel (`dropdown-menu w-100`) containing a card with a container/row grid.

Standard megamenu structure:
```liquid
<div class="dropdown-menu w-100">
  <div class="card card-lg">
    <div class="card-body">
      <div class="tab-content">
        <div class="tab-pane fade show active" id="navTab">
          <div class="container">
            <div class="row justify-content-center">
              <!-- columns here -->
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

### Megamenu column patterns

**Text link column** (standard):
```liquid
<div class="col-12 col-md-2">
  <div class="mb-4 font-weight-bold"><b>Section Heading</b></div>
  <ul class="list-styled mb-6 mb-md-0 font-size-sm">
    <li class="list-styled-item">
      <a class="list-styled-link pl-3" href="/site/path">Link Text</a>
    </li>
  </ul>
</div>
```

**Image tile column** (desktop only, `d-none d-lg-block`):
```liquid
<div class="col-2 d-none d-lg-block">
  <div class="card">
    <a href="/site/path">
      <img class="card-img" src="{{ 'image.webp' | asset_url }}" alt="...">
    </a>
    <b style="font-size:0.9rem">Tile Heading</b>
    <span style="font-size:0.8rem">Tile subtext</span>
  </div>
</div>
```

### Available megamenu snippets (parent library)

These exist in the parent and can be used or adapted for client sites:

`navigation/megamenu/all-products`, `archiving`, `art-services`, `bound-products`, `business`, `calendars`, `cards`, `cards-calendars`, `create`, `custom`, `digitize-media` (blank — stub), `digitizing`, `education`, `film`, `film-cameras`, `gifts`, `leaflets`, `photo-books`, `press-printing`, `print-services`, `prints`, `services`, `sports-events`, `stationery`, `studio`, `wall-art`, `wall-decor`, `wide-format-simple`

### Simple dropdown (non-megamenu)

For single-column dropdowns, use `navigation/dropdown` — standard Bootstrap dropdown without the full-width card panel.

---

## 4. Theming System

### How theming works

Shopper uses a two-layer theming system:

1. **`style/*` snippets** — small snippets each containing a single value (color, size, weight, etc.). These are the source of truth for all theme values.
2. **`pages/custom.css`** — the CSS delivery page that outputs all theme-aware CSS. It includes `{% snippet 'style/custom.css' %}` plus a large block of CSS that references the style snippets inline via `{% snippet 'style/...' %}`.

### CSS delivery

- The CSS page at URL `/site/custom.css` is loaded in `html.head` via: `<link rel="stylesheet" href="/site/custom.css">`
- The snippet `style/custom.css` is **blank in the parent** — it is the stub where per-site custom CSS goes.
- **All CSS customisations go in `style/custom.css`** — this is what gets injected into the CSS page.
- Do NOT write CSS inline in Liquid/HTML snippets.

### Style snippet inventory

**Colors:**
- `color-primary` — primary brand color (buttons, link hover) — default: `#faa21b`
- `color-secondary` — secondary / dark button color — default: `#2d2d2d`
- `color-secondary-50` — secondary at reduced opacity
- `color-button-bkg` — button background — default: `#32c5ff`
- `color-button-bkg-hover` — button background on hover
- `color-button-label` — button text — default: `#ffffff`
- `color-button-label-hover` — button text on hover
- `color-button-outline` — button border color
- `color-button-outline-hover` — button border on hover
- `color-font` — body text color
- `color-font-secondary` — secondary body text color
- `color-font-hover` — body text hover
- `color-footer` — footer background (`.bg-dark`) — default: `#2d2d2d`
- `color-highlight` — highlight/accent (used in photo prints UI)
- `color-highlight-light` — lighter version of highlight
- `color-background` — page background color
- `color-bg-v-light` — very light background for sections (`.bg-very-light`)
- `color-announcement-bar` — top bar background (`.bg-light`)
- `color-promotion-bar` — promotion bar background
- `color-promotion-text` — promotion bar text
- `color-pill-btn-outline` — outline color for pill/variant buttons
- `color-bullet-point` — bullet point color in lists
- `link-color` — default link color
- `link-hover-color` — link hover color
- `nav-hover-bkg-color` — nav link hover background (animated underline effect)
- `nav-hover-text-color` — nav link hover text
- `navbar-hover-text-color` — active nav item text color
- `badge-bkg-color` — badge background
- `badge-text-color` — badge text

**Fonts:**
- `fonts` — custom font-face declarations (if any)
- `custom-body-font` — font-family string when `font-body == 'custom'`
- `body-font-size` — default: `1rem`
- `body-font-weight` — default: `400`
- `nav-font-size` — default: `0.9rem`
- `nav-font-weight` — default: `400`

**Borders & radius:**
- `btn-border-radius` — standard button border radius — default: `6px`
- `btn-pill-border-radius` — pill/variant button radius — default: `6px`
- `btn-pill-bkg-color` — pill button background
- `btn-pill-bkg-color-hover` — pill button hover background
- `btn-pill-text-color` — pill button text
- `btn-pill-text-color-hover` — pill button hover text
- `content-border-radius` — cards/content border radius — default: `6px`
- `text-input-border-radius` — form input border radius — default: `6px`
- `var-img-border-radius` — variant image border radius
- `color-swatch-border-radius` — color swatch border radius
- `color-swatch-width` — color swatch width
- `pill-img-outline-width` — outline width on selected variant image

**Logo & footer:**
- `header/logo` — rendered logo element
- `header/logo-height-desktop` — logo height on desktop
- `header/logo-height-mobile` — logo height on mobile
- `footer/logo` — rendered footer logo
- `footer-color-background` — footer section background
- `footer-color-font` — footer text color
- `scroll-to-top-color-background` — scroll-to-top button background

---

## 5. Admin Checklist System

### How it works

Each checklist key is a snippet at `admin/checklist/<key-name>`. The snippet contains a plain text value (e.g. `TRUE`, `FALSE`, a color, a domain name, or blank). The layout and other snippets `{% capture %}` these snippets and branch logic based on the value.

**Convention:**
- Boolean flags: `TRUE` or blank (blank = false/disabled)
- `FALSE` is also used explicitly in some cases
- Non-boolean settings contain the actual value (color hex, domain, font name, etc.)

### Value-bearing checklists — never assume boolean

Many checklist snippets hold a value the parent **interpolates directly**.
Overwriting one with `TRUE` does not disable a feature; it produces a
render-time failure. `cart-icon` set to `TRUE` resolves
`{% snippet 'icons/' + value + '.svg' %}` to `icons/TRUE.svg`, and because the
header renders on every page the whole site goes down with
`Liquid error: Snippet not found`.

Known value-bearing keys: `cart-icon`, `user-icon`, `font-body`
(`avenir` / `lato` / `open-sans` / `custom`), `header-logo-position`
(`LEFT` / `CENTER`), `gallery-thumb-position`, `align-collection-card`,
`align-collection-title`, `variant_columns` (`col-md-6`),
`variant_columns_mobile`, `country-filter` (`United States`),
`description-position`, `pricing-tab-position`, `upload-btn-style`.

**Rules when editing checklists on an existing site:**

1. **Read the seed value first.** Never assume a key is a boolean.
2. **Preserve the site's own vocabulary** — value type *and* token case. A site
   using `FALSE` (not blank) for off, and `LEFT` (not `left`), must keep both.
   Comparisons against `'TRUE'` are case-sensitive.
3. **Override only where the build genuinely requires it.** On one rebuild, 14
   of 26 changed checklists were reverted as unnecessary; every one had been an
   unforced risk.
4. **Capture cleanly.** A `{% capture %}` of a checklist snippet includes
   surrounding newlines and indentation. Always `| strip` (and `| upcase` where
   case is uncertain) before comparing, or every comparison silently falls
   through to the default.

**`custom-X-page` flags against an empty target snippet.** A key such as
`admin/checklist/custom-faq-page` set to `TRUE` with an empty `website/faq_page`
renders a blank page with no error. Worth checking on any site you touch — it is
frequently pre-existing rather than introduced.

### Full checklist key inventory

#### Navigation & Header
| Key | Values / Notes |
|---|---|
| `header-logo-position` | `LEFT` = style1; `CENTER` (default) = style3; `CUSTOM` = conditional |
| `top-promotion-bar` | `TRUE` = show promotion bar above nav |
| `bottom-promotion-bar` | `TRUE` = show promotion bar below nav |
| `back-to-top` | `TRUE` = show scroll-to-top button |
| `search` | `TRUE` = show search icon in nav |
| `cart-icon` | Controls cart icon style (see icon variants below) |
| `user-icon` | Controls user icon style (see icon variants below) |

**Cart icon style options:**
`bag-outline`, `bag-outline-thin`, `bag-solid`, `basket-outline`, `basket-outline-thin`, `basket-solid`, `cart-outline`, `cart-outline-thin`, `cart-sharp-outline`, `cart-sharp-outline-thin`, `cart-sharp-solid`, `cart-solid`

**User icon style options:**
`user-circle-outline`, `user-circle-outline-thin`, `user-circle-solid`, `user-outline`, `user-outline-thin`, `user-person-outline`, `user-person-outline-thin`, `user-person-solid`, `user-sharp-outline`, `user-sharp-outline-thin`, `user-sharp-solid`, `user-solid`

#### Cart Behavior
| Key | Values / Notes |
|---|---|
| `cart-editable-options` | `TRUE` = options editable directly in cart |
| `cart-note` | `TRUE` = show order note field in cart |
| `cart-show-text-options` | `TRUE` = show text options in cart display |
| `cart-top-continue-shopping-link` | `TRUE` = show "continue shopping" link |
| `hide-coupon-form-cart` | `TRUE` = hide promo code input in cart |
| `hide_pricing_cart` | `TRUE` = hide pricing in cart |
| `activate-cart-cross-sell-section` | `TRUE` = enable cross-sell section in cart |
| `activate-cart-product-url-path` | Controls product URL path in cart |
| `hide-template-options-from-cart` | `TRUE` = hide template options from cart display |

#### Checkout Policy
| Key | Values / Notes |
|---|---|
| `clean-checkout` | `TRUE` = suppress nav links (logo + cart only) |
| `guest-checkout` | `TRUE` = allow guest checkout (default: `TRUE`) |
| `disable-delivery` | `TRUE` = disable delivery option |
| `default-delivery-option` | Sets default delivery method |
| `display-shipping-options` | `TRUE` = show shipping options |
| `disable-user-registration` | `TRUE` = prevent new registrations |
| `checkout-disclaimer` | `TRUE` = show checkout disclaimer text |
| `checkout-rush` | `TRUE` = enable rush delivery option |
| `checkout-rush-special` | `TRUE` = enable special rush option |
| `checkout-column-positions` | Controls checkout layout column order |
| `cash-on-delivery` | `TRUE` = enable cash on delivery |
| `pay-in-store` | `TRUE` = enable pay in store |
| `pickup-in-store` | `TRUE` = enable pickup in store |
| `dont-require-pickup-contact-details` | `TRUE` = skip contact details for pickup |
| `note-contact-details-required` | `TRUE` = mark contact details as required |
| `require-billing-address` | `TRUE` = require billing address |
| `require-billing-address-company` | `TRUE` = require company name in billing |
| `require-last-name` | `TRUE` = require last name at checkout |
| `require-telephone` | `TRUE` = require telephone at checkout |
| `require-telephone-registration` | `TRUE` = require telephone at registration |
| `input-public-address` | Public/system address for digital-only orders |
| `billing-address-state-global` | `TRUE` = show state field globally |
| `minimum-charge` | Minimum order charge amount |
| `max-cart-total-pay-in-store` | Maximum total for pay in store |
| `confirm-start-date` | `TRUE` = require start date confirmation |
| `confirm_start_date_label` | Label for start date field |
| `confirm-with-invoice` | `TRUE` = confirm order with invoice |
| `payment-link` | `TRUE` = enable payment link option |
| `payment-gateway` | Active gateway: `stripe`, `braintree`, `square`, `authorize.net`, `paypal` — default: `stripe` |
| `digital-only-delivery` | `TRUE` = enable digital-only delivery mode |
| `proof-order-checkout` | `TRUE` = enable proof before checkout |
| `promocode-checkout` | `TRUE` = show promo code at checkout |
| `hide-film-order-checkout` | `TRUE` = hide film processing at checkout |
| `film-order-drop-off-id` | Film drop-off location ID |
| `film-order-drop-off-id-label` | Label for film drop-off field |
| `enable-film-mailer-label` | `TRUE` = enable film mailer label printing |
| `custom_checkout_condition` | Custom checkout condition logic |
| `custom_checkout_logic` | Custom checkout logic |

#### Kiosk Mode
| Key | Values / Notes |
|---|---|
| `kiosk-mode-enabled` | `TRUE` = enable kiosk mode |
| `kiosk-mode-domain` | Alternate domain for kiosk detection |
| `kiosk-pay-in-store-only` | `TRUE` = restrict pay-in-store to kiosk only |
| `kiosk-remove-captcha` | `TRUE` = remove CAPTCHA in kiosk mode |

**Kiosk captcha is per-subdomain.** Kiosk mode usually runs on its own subdomain (`kiosk-mode-domain`). CAPTCHA configuration does not carry across from the main storefront to the kiosk subdomain — captcha must be removed/configured on the kiosk subdomain specifically (e.g. `kiosk-remove-captcha` set on the kiosk site). Symptom if missed: customers hit a CAPTCHA on the kiosk that the main storefront does not show.

#### Product Display
| Key | Values / Notes |
|---|---|
| `pricing-display` | Controls pricing display style |
| `pricing-tab-position` | Position of pricing tab |
| `description-position` | Position of product description |
| `collection-description-position` | Position of collection description |
| `align-collection-card` | Card alignment in collection |
| `align-collection-card-center` | Center-align collection cards |
| `align-collection-card-left` | Left-align collection cards |
| `align-collection-title` | Title alignment in collection |
| `collection-card-shadow` | `TRUE` = add shadow to collection cards |
| `variant_columns` | Number of variant columns (desktop) |
| `variant_columns_mobile` | Number of variant columns (mobile) |
| `upload-btn-style` | Upload button style variant |
| `display-color-palette` | `TRUE` = show color palette option |
| `hide-color-label` | `TRUE` = hide color label |
| `bullet-point-style` | List bullet style (disc, circle, etc.) |
| `pill-btn-outline` | `TRUE` = use pill/outline button style |
| `display-print-original-filename` | `TRUE` = show original filename for prints |

#### Photo Prints
| Key | Values / Notes |
|---|---|
| `prints-autoselect` | `TRUE` = auto-select first print size |
| `prints-thumbnails` | `TRUE` = show print thumbnails |
| `prints-thumbnails-crop` | `TRUE` = crop thumbnails |
| `prints-thumbnails-photo` | `TRUE` = show photo thumbnails |
| `enable-image-filters-image-upload` | `TRUE` = enable image filters for upload options |
| `enable-image-color-image-upload` | `TRUE` = enable color adjustment for upload options |
| `enable-image-filters-photo-prints` | `TRUE` = enable filters for photo prints |
| `enable-image-color-photo-prints` | `TRUE` = enable color adjustment for photo prints |

#### Print Product Sizes
Standard prints: `product-print-3x5`, `4x5`, `4x6`, `4x8`, `5x5`, `5x7`, `6x8`, `6x9`, `8x8`, `8x10`, `10x10`, `10x13`, `10x15`, `11x14`, `12x12`, `product-print-no-bleed`

Enlargements/large format: `product-enlargements-bleed-1-8`, `product-enlargements-no-bleed`, `product-large-16x16`, `16x20`, `16x24`, `18x24`, `20x20`, `20x30`, `24x24`, `24x30`, `24x36`, `30x40`, `40x60`

#### Fonts
| Key | Values / Notes |
|---|---|
| `font-body` | `lato`, `open-sans`, `avenir`, `custom` — default: `avenir` |
| `font-lato` | Lato font activation flag |
| `font-open-sans` | Open Sans font activation flag |

#### Account & User
| Key | Values / Notes |
|---|---|
| `home-page-login-form` | `TRUE` = show login gate on home page |
| `account_nav_version` | Account navigation version |
| `account_saved_projects_version` | Saved projects version |
| `account_saved_projects_view` | Default view for saved projects |
| `hide-saved-projects` | `TRUE` = hide saved projects from account |
| `hide-galleries` | `TRUE` = hide galleries from account |
| `hide-personal-dates` | `TRUE` = hide personal dates |
| `gallery_version` | Gallery version (v1 or v2) |
| `gallery_tile_layout` | Gallery tile layout style |
| `gallery-thumb-position` | Gallery thumbnail position |
| `gallery-download` | `TRUE` = enable gallery download |
| `gallery-image-download` | `TRUE` = enable individual image download |
| `gallery-image-filename` | `TRUE` = show image filename in gallery |
| `gallery-order-prints` | `TRUE` = enable order prints from gallery |
| `custom-registration` | `TRUE` = use custom registration form |
| `multiple-personal-calendars` | `TRUE` = allow multiple personal calendars |

#### Integrations
| Key | Values / Notes |
|---|---|
| `setup-google-tag-manager` | GTM container ID |
| `activate-klaviyo` | `TRUE` = enable Klaviyo |
| `activate-constant-contact` | `TRUE` = enable Constant Contact |
| `activate-stamped` | `TRUE` = enable Stamped.io reviews |
| `activate-shareme` | `TRUE` = enable ShareMe chat |
| `activate-google-reviews-ai-widget-domain` | `TRUE` = enable Google AI reviews widget |
| `google-reviews-ai-widget-domain` | Domain for Google reviews widget |
| `activate-contributions` | `TRUE` = enable contributions feature |
| `activate-group-projects` | `TRUE` = enable group projects |
| `activate-free-shipping-progress-bar` | `TRUE` = enable free shipping progress bar |
| `pixfizz-ai-chatbot` | `TRUE` = enable Pixfizz AI chatbot |
| `pixfizz-ai-chatbot-admin-only` | `TRUE` = show chatbot to admins only |
| `activate-vat-rate` | `TRUE` = enable VAT rate display |

#### SEO & Metadata
| Key | Values / Notes |
|---|---|
| `no-index` | `TRUE` = add noindex to entire site |
| `seo-tdks` | SEO title/description/keywords settings |
| `update-website-title` | Website title |
| `update-website-description` | Website description |
| `update-website-domain` | Website domain |
| `update-website-logo` | Website logo asset |
| `update-branding-design-tool` | Branding in design tool |
| `schema_loop_all_products` | `TRUE` = include all products in schema |

#### Content & Pages
| Key | Values / Notes |
|---|---|
| `blog` | `TRUE` = enable blog section |
| `custom-blog-page` | `TRUE` = use custom blog page |
| `custom-blog-post` | `TRUE` = use custom blog post template |
| `custom-home-page` | `TRUE` = use custom home page |
| `custom-contact-page` | `TRUE` = use custom contact page |
| `custom-faq-page` | `TRUE` = use custom FAQ page |
| `custom-terms-page` | `TRUE` = use custom terms page |
| `gdpr-banner` | `TRUE` = show GDPR cookie banner |
| `hide-contact-business-page` | `TRUE` = hide contact on business pages |
| `hide-contact-service-page` | `TRUE` = hide contact on service pages |
| `date-format` | Date display format |
| `country-filter` | Country filter for shipping |
| `filters-sticky` | `TRUE` = sticky collection filters |

---

## 6. Snippet Namespace Conventions

Shopper uses a consistent `namespace/name` path convention. In the file system, `/` is stored as `__`.

| Namespace | Purpose |
|---|---|
| `admin/checklist/` | Feature flags and config values |
| `admin/forms/` | Admin form components |
| `checkout/` | Checkout-specific components |
| `collection/` | Collection/shop page components |
| `email-notifications/` | Legacy email template components |
| `email-shopper/` | Current Shopper email templates |
| `footer/` | Footer sub-components |
| `header/` | Header sub-components (logo, promotion bars) |
| `helpers/` | Utility snippets (e.g. `helpers/is-kiosk-mode`) |
| `icons/` | SVG icon snippets |
| `integrations/` | Third-party integration scripts |
| `modals/` | Modal dialogs |
| `navigation/` | Nav components |
| `navigation/megamenu/` | Individual megamenu panels |
| `product/` | Product page components |
| `product/cards/` | Product card variants |
| `product/details/` | Product detail components |
| `sections/custom/` | One-off custom sections |
| `sections/dynamic/` | Dynamic/AJAX sections |
| `sections/static/` | Reusable static sections |
| `services/` | Services page components |
| `services/cards/` | Service card variants |
| `social-media/` | Social media URLs and OG tags |
| `style/` | Theme variable snippets |
| `website/` | Site-level data (contact info, title, etc.) |
| `website/contact/` | Contact detail snippets |

---

## 7. Website Contact & Data Snippets

These snippets store site-specific content that varies per client:

- `website/title` — site name
- `website/description` — meta description
- `website/brand` — brand name (used in footer copyright)
- `website/contact/support-telephone`
- `website/contact/support-telephone-label`
- `website/contact/support-email-address`
- `website/contact/support-hours`
- `website/contact/support-text`
- `website/contact/address`
- `website/contact/city`, `city-state`, `state`, `zip-code`
- `website/contact/location`
- `website/contact/title`, `meta_title`, `meta_description`
- `website/contact/faq_path`
- `website/contact/geo-location`, `geo-map`
- `website/contact/services-telephone`, `services-telephone-label`
- `website/contact/sla-note`
- `website/google-review-link`
- `website/gtag` — Google Analytics 4 tag ID
- `website/meta-pixel` — Facebook Pixel ID
- **Google Ads conversion tracking:** there is no dedicated built-in preset or snippet for a Google Ads conversion tag in Shopper (only GTM, GA4, and Meta Pixel exist). Deploy the Google Ads site tag (`AW-...`) and the purchase conversion event through GTM using the existing `setup-google-tag-manager` key (conversion linker + conversion action tag + thank-you-page event). A hardcoded gtag conversion snippet, if used instead, is site-specific code with no checklist key reserved for it.
- `website/px-subdomain` — Pixfizz subdomain
- `website/film-delivery-address` — for film mail-in orders
- `website/trust-badges` — trust badge images
- `website/current-promotions` — promotional content
- `website/sitewide-promotion` — sitewide promo text

---

## 8. Footer Structure

The footer (`snippets/footer`) has two sections:

**Top section** (`py-6 py-md-12 border-bottom border-gray-700`):
- Newsletter signup (hidden by default — uses Klaviyo)
- 4-column grid: logo + social links | support (phone/email/hours) | resources (Contact, FAQs, Shipping, Order Status) | company (Our Story, Blog if enabled)

**Bottom bar** (`py-3 bg-dark`):
- Copyright line using `website/brand`
- "Powered by Pixfizz" logo (desktop only)
- Terms & Privacy + Sitemap links
- Payment logos: Mastercard, Visa, AMEX

Footer background: `style/color-footer` (`.bg-dark`) and `style/footer-color-background`. Social icons only render if their `social-media/<platform>` snippet is non-blank.

---

## 9. HTML Head (`html.head`)

Key elements in order:

1. GTM script (if `integrations/google/tag-manager` set)
2. GA4 gtag (if `website/gtag` set)
3. Facebook Pixel (if `website/meta-pixel` set)
4. Pixfizz CMS JS (`cms.js`) + CSS (`cms.css`) via `pixfizz_asset_url`
5. Prefetch for editor assets
6. Stamped.io script (if API key set)
7. Theme CSS: `flickity-fade.css`, `jquery.fancybox.min.css`, `flickity.min.css`, `vs2015.css`, `simplebar.min.css`, `theme.min.css`, `px-shopper.css`, `feather.css`
8. Custom CSS: `<link rel="stylesheet" href="/site/custom.css">`
9. Title, meta description
10. Open Graph tags (`social-media/open-graph`)
11. Canonical URL
12. Favicon
13. No-index if `admin/checklist/no-index == 'TRUE'`
14. Ahrefs script, custom links
15. Fonts (Google Fonts for Lato/Open Sans; custom via `style/fonts`)

---

## 10. Key Page Inventory

All pages live at `/site/<url>`.

**Account:** `account`, `account-address`, `account-address-edit`, `account-address-new`, `account-carts`, `account-galleries`, `account-galleries/<gallery>`, `account-orders`, `account-orders/details`, `account-personal-calendars`, `account-personal-dates`, `account-personal-info`, `account-saved-projects`

**Commerce:** `cart`, `checkout`, `checkout2`, `checkout-order`, `checkout-print`, `confirm`, `thank-you`, `payment_success`, `payment_failed`, `payment-link`, `draft-order/<order_id>`, `group_order_success`, `add-to-cart`

**Shop/Products:** `shop`, `shop/<collection>` (1–3 levels), `product/<path>` (1–3 collection levels + url-path), `photo-prints`, `prints`, `productview`, `project-edit`, `promotions`

**Auth:** `login`, `login-checkout`, `login/email-sent`, `password-reset`, `reset`

**Content:** `__home` (home page), `blog`, `blog/<post>`, `contact-us`, `contact-thankyou`, `faq`, `shipping-and-returns`, `terms-conditions-privacy`, `our-work`, `sections-gallery`, `services`, `services/<path>`, `business`, `business/<path>`, `gallery-shop`, `gallery-shop/<gallery>`, `sitemap`, `404`

**Special:** `custom.css` (CSS delivery), `editor-scripts.js`, `editor.css`, `robots.txt`, `feed/products.xml`, `feed/products-custom.xml`, `search/index.json`, `search/worker.js`, `order-management`, `setup/*` (admin setup wizard)

**Generic catch-alls:** `-page-path-1`, `-page-path-1/-page-path-2`, `-page-path-1/-page-path-2/-page-path-3`

---

## 11. Email Templates

Two parallel systems exist. The current system is `email-shopper/`.

**Templates:**
- `email-shopper/templates/abandoned-cart`
- `email-shopper/templates/order-confirmed`
- `email-shopper/templates/order-draft`
- `email-shopper/templates/order-shipped`
- `email-shopper/templates/password-reset`
- `email-shopper/templates/user-signup`

**Style sub-snippets:** `email-shopper/style/background-light`, `brand-color`, `muted-color`, `text-color`

**Layout components:** `email-shopper/layout`, `header`, `logo`, `banner`, `trust-block`, `social-links`

> Email templates run outside the storefront session. Project previews in email require `share: orderline.project.share_code`. See `40_PLAYBOOK.md`.

---

## 12. Section Library

**Static sections (`sections/static/`):**
`2-column-cards`, `2-column-collection-spotlight` (+ `-2`), `2-photo-feature` (+ `-2`), `3-column-cards` (+ `-1`, `-2`, `-2nd-row`), `3-features`, `3-steps`, `4-block-best-sellers`, `4-company-values`, `6-across`, `6-categories`, `7-across`, `8-feature-spotlights`, `8-photo-feature`, `about`, `banner-alt`, `carousel`, `carousel-features`, `carousel-products`, `checklist-feature`, `coming-soon`, `countdown-promo-fullwidth`, `custom`, `features`, `hero-3-columns-fullwidth` (+ `-2`), `hero-4-features`, `hero-carousel`, `home-page__3-columns-fullwidth`, `home-page__digitize-services-cards`, `home-page__hero-banner-fullwidth`, `home-page__trusted-brands`, `image-slider-comparison`, `newsletter`, `our-blog`, `parallax-banner`, `parallax-text-block`, `project-gallery`, `projects`, `promo-banner-color`, `promo-countdown`, `reviews`, `reviews-1-across`, `services-contact-footer`, `shop-by-brand`, `top-item-feature`, `top-picks`

**Dynamic sections (`sections/dynamic/`):**
`blog`, `carousel-products`, `collection-block`, `free_shipping_progress_bar`, `product-carousel`, `product-description-tabs`, `services`

Dynamic sections re-inject into the DOM on AJAX updates. Use the `style onload` pattern for any JS that must survive re-injection (see `01_CODE_GOVERNANCE.md`).

### Shop All page

A Shopper page that displays an image for every top-level collection, giving
shoppers a single visual entry point to all categories. Addresses the visual
navigation gap when a store has many top-level collections. Live as of mid 2026.
The exact page or snippet name should be confirmed against the live deployment or
with Matjaz.

---

## 13. Practical Notes for Development

- **Editing nav links:** Always check which nav style is active before editing. `admin/checklist/header-logo-position` value `LEFT` renders `navigation/style1`; `CENTER` (default) renders `navigation/style3`. Edit the `{% capture navigation_links %}` block at the top of the **active** snippet only — editing the wrong one has no effect. Do not edit the HTML structure below the capture block.- **Adding a megamenu panel:** Create or edit `navigation/megamenu/<name>`. Use the patterns in section 3 above. Register the nav item in the `navigation_links` capture block.
- **CSS changes:** Always in `style/custom.css` only. Never inline in Liquid/HTML.
- **Theme color changes:** Edit the relevant `style/<token>` snippet. The CSS page picks them up automatically.
- **Checklist changes:** Edit the `admin/checklist/<key>` snippet value. Boolean flags expect exactly `TRUE` or blank.
- **New sections:** Use an existing section snippet from `sections/static/` or create a new one. Include in `pages/__home` or the relevant page.
- **Email project previews:** Always include `share: orderline.project.share_code` in the preview URL.
- **Font changes:** Set `admin/checklist/font-body` to `lato`, `open-sans`, `avenir`, or `custom`. For `custom`, populate `style/custom-body-font` with the font-family string.
- **Shared snippets:** If a snippet is used across multiple client sites, follow the Shared Snippet Contract Rule in `01_CODE_GOVERNANCE.md` — do not remove existing variables, IDs, or JS hooks.
- **Custom home page content:** Place home page content in the snippet `website/homepage`. This snippet is only rendered when the **"Custom snippet (website/homepage) for home page"** checkbox is ticked in Custom Admin → Storefront Settings. Both the snippet and the checkbox are required — neither works without the other.

## 14. Creating Pages on Child Sites

Child sites of Shopper cannot create real CMS pages. New pages are
created by adding instances of the `pages` Custom Type instead.

**Navigation path:** Main Admin → Website tab → Custom Types → Pages

Create a new instance with these field values:
- `page_path` — must match the URL path exactly. At level 1 this is the single
  segment (`graduation` → `/site/graduation`). At levels 2 and 3 it is the
  **full slash-joined path**, not just the final segment
  (`services/framing` → `/site/services/framing`)
- `page_title` — page title
- `page_description` — snippet field, optional
- `page_content` — snippet field — main page content goes here
- `page_schema` — leave blank unless you need structured data / ld+json

No redirect needed. The page resolves automatically once the instance is saved.

How it works:
- Shopper has catch-all pages at `/:page-path-1` (and level 2/3 variants)
- The catch-all page looks up a `pages` Custom Type instance where
  `custom.page_path` matches the URL segment
- If no match → 404
- Page content renders from `custom_page.custom.page_content`
- Full CMS context is available inside the content snippet

Constraints:
- Real CMS pages always take priority over Custom Type instances on the
  same path — always use a path that has no real page equivalent
- Levels 1, 2 and 3 each have their own catch-all page. Each one builds
  `page_path` by joining its own path params with `/`, so a level 3
  instance stores all three segments in that single field

### Head-level dependencies must repeat the lookup

`html.head` renders **before** `{{ page.content }}`. A `custom_page` variable
assigned inside the catch-all page body therefore does not exist yet when the
head snippet runs. Anything in the head that depends on the Custom Type
instance (meta title, meta description, canonical, robots) has to repeat the
path build and the lookup inside `html.head` itself.

This is the usual reason meta title and description come out blank on Custom
Type pages while the visible page content renders correctly.

The head-side lookup rebuilds the path from the request params, appending
levels 2 and 3 only when they are present, then queries the same collection:

```liquid
{% assign cp_path = request.path_params['page-path-1'] %}
{% if request.path_params['page-path-2'] != blank %}
	{% assign cp_path = cp_path | append: '/' | append: request.path_params['page-path-2'] %}
{% endif %}
{% if request.path_params['page-path-3'] != blank %}
	{% assign cp_path = cp_path | append: '/' | append: request.path_params['page-path-3'] %}
{% endif %}
{% if cp_path != blank %}
	{% assign cp_page = website.custom_types.pages | where: 'custom.page_path', cp_path | first %}
{% endif %}
```

The `!= blank` guard matters. Without it the lookup runs on every page on the
site, not only on the catch-all pages.

**Suppressing indexing per page.** Add a boolean custom field to the `pages`
Custom Type and emit the robots tag from the head block above. `hide_from_index`
matches the naming already used for products and designs. Two platform rules
apply:

- Boolean custom fields are real booleans. Test with
  `{% if cp_page.custom.hide_from_index %}`, never against the strings
  `'true'` or `'false'`.
- Custom fields do not inherit from parent to child, so the field has to be
  created on every site that needs it.

The same head block is what fixes meta title and description on Custom Type
pages, so it is worth doing all of it in one pass rather than only the robots
tag.

## 15. Custom Admin — Shopper v2 (`/site/manage/`)

The Shopper v2 custom admin is a replacement for the legacy `setup/` wizard pages. It uses the `shopper-admin` layout and provides a sidebar navigation to all configuration pages.

### Access control

The `shopper-admin` layout includes a `{% if user.is_admin %}` gate. Only admin users can access `/site/manage/*` pages. Non-admin users see nothing.

### Critical dependency

The `shopper-admin` layout must include `cms.js` and `cms.css` (loaded via `pixfizz_asset_url`). Without `cms.js`, all `{% form %}` tags with `async: true` / `autosubmit: true` render as static HTML — checkboxes and snippet-saving forms will not submit.

### Page inventory

| Path | Content |
|---|---|
| `manage/dashboard` | Overview / home page |
| `manage/branding` | Colors, fonts, logo, brand identity |
| `manage/store` | Storefront settings, collection layout, product page options |
| `manage/homepage` | Homepage content configuration |
| `manage/navigation` | Nav links, megamenu, footer links |
| `manage/collections` | Collection display options |
| `manage/products` | Product page configuration |
| `manage/gallery` | Gallery feature settings |
| `manage/cart` | Cart page options |
| `manage/checkout` | Checkout flow, rush fees, shipping display |
| `manage/payments` | Payment gateway settings |
| `manage/account` | Customer account area configuration |
| `manage/seo` | SEO settings, meta defaults, llms.txt management |
| `manage/emails` | Email template configuration |
| `manage/integrations` | Third-party integrations (GTM, Klaviyo, etc.) |
| `manage/advanced` | Advanced settings |

### Tools pages

| Path | Content |
|---|---|
| `manage/tools/product-importer` | CSV-based static product importer |
| `manage/tools/download-images` | Live preview image downloader (per-collection ZIP download) |

#### Static Product Importer — CSV format

The Static Product Importer (`manage/tools/product-importer`) is a Shopper template-level
tool, not a Pixfizz Core feature. It is gated by the `shopper-admin` layout's admin check,
so only admin users can reach it. On upload, it creates products via the Pixfizz API and
assigns them to a selected collection. A blank CSV template can be downloaded from the tool
page itself. Recommended for stores with large static catalogues (standard print sizes,
fixed products without personalization).

CSV column order (the tool reads columns positionally):

`name, code, price, description, category, asset_image_name, fulfillment_code, track_inventory, current_inventory, tax_exempt, min_quantity, max_quantity`

| Column | Type | Notes |
|---|---|---|
| `name` | string | **Required** |
| `code` | string | **Required** — the product code/SKU |
| `price` | number/formula | **Required** |
| `description` | string | Optional |
| `category` | string | Optional |
| `asset_image_name` | string | Optional — filename of an asset already uploaded under Website > Assets |
| `fulfillment_code` | string | Optional |
| `track_inventory` | boolean | `"true"` / `"false"` |
| `current_inventory` | integer | Whole number |
| `tax_exempt` | boolean | `"true"` / `"false"` |
| `min_quantity` | integer | Whole number |
| `max_quantity` | integer | Whole number |

- A header row is optional and is skipped if present.
- This tool drives the same product-creation path as the Pixfizz API; it does not create
  personalization templates or designs, only static products.

### Sidebar navigation

The sidebar is defined in a shared snippet. When adding new pages, update the sidebar snippet with the new nav item. The sidebar uses the `shopper-admin` design system CSS classes (`s-card`, `s-field`, `s-field-label`, etc.).

---

## 16. Kiosk Touchscreen Mode

**Status:** Partially implemented. Login gate, idle screen, and cart/checkout overlays not yet built. Parked as of May 2026.

Kiosk mode is a checklist-gated feature designed for in-store photo lab kiosks. When enabled, it transforms the Shopper storefront into a touch-friendly, simplified UI for two primary use cases: ordering photo prints and submitting film processing orders.

### Architecture

- **Gate:** The `admin/checklist/kiosk-touchscreen-mode` snippet controls activation. When set to `TRUE`, the `index` layout adds the class `kiosk-touchscreen` to the `<body>` tag.
- **9 checklist snippets** created on the parent template:
  - `admin/checklist/kiosk-touchscreen-mode` — master toggle
  - `admin/checklist/kiosk-prints-collection` — collection path for the prints workflow
  - `admin/checklist/kiosk-film-collection` — collection path for film processing
  - `admin/checklist/kiosk-other-collection` — collection path for secondary products
  - `admin/checklist/kiosk-prints-image` — hero tile image for prints
  - `admin/checklist/kiosk-film-image` — hero tile image for film
  - `admin/checklist/kiosk-other-image` — hero tile image for other products
  - `admin/checklist/kiosk-prints-label` — tile label for prints
  - `admin/checklist/kiosk-film-label` — tile label for film

- **Content snippets:** `kiosk/top-rail` (simplified header bar with logo + Start Over button) and `kiosk/home` (tile-based landing page).

### CSS scoping

All kiosk CSS is scoped under `.kiosk-touchscreen` so it has zero impact when the mode is off. CSS lives in the child site's `style/custom.css`.

### UX constraints

- No navigation bar (hidden via CSS)
- Large buttons, large tiles — designed for touch
- Minimal scrolling
- Login gate required (not yet implemented)
- Idle timeout with attractor screen (not yet implemented)

### Remaining work

1. Login gate + login page styling
2. Idle screen / attractor
3. Cart/checkout CSS overlays
4. PDP CSS overlay
5. Start Over + idle timer JS
6. Custom admin section for kiosk settings

---

## 17. Known Gotchas

These are recurring issues worth warning yourself about. Not fix recipes — the fix
is in the code or the commit history. These are "things to watch for when you are
debugging a symptom that matches one of these patterns".

### Image slider not refreshing after gallery updates (2026-02-23)
**Symptom:** Customer uploads / changes images in a gallery, the gallery data
updates, but the image slider on the product or cart page does not reflect the
change until the user does a hard page reload.
**Cause:** Slider initialization happens once on page load and does not observe
gallery data changes.
**Workaround:** Trigger a slider re-init hook after any gallery mutation, or
force a reload as a last resort on pages where the issue is visible.

### Date input failure in some Chrome / older Safari (2026-03-16)
**Symptom:** Customer cannot select a date on a form date-picker input in certain
Chrome versions and older Safari builds.
**Cause:** Browser-level `<input type="date">` implementation bug — not a Pixfizz
bug, but it affects Pixfizz forms.
**Workaround:** Where date selection is critical, use a JS-based date picker
rather than relying on the native `<input type="date">`. Document the affected
browsers in customer-facing support docs so they know to update their browser.

### Worker JS impacting site speed / SEO (2026-03-31)
**Status:** Under investigation as of 2026-03-31. No fix documented yet.
**Symptom:** Worker JS loading is impacting Core Web Vitals and Lighthouse SEO
scores on at least one site.
**Action:** Track separately — do not assume a fix is available when scoping SEO
work on a site that depends on Worker JS. Confirm current status before
committing to a performance target.

### CSV export filter excludes anonymous projects (Rapid)
**Status:** Known bug, 2026-03-16. To be fixed.
**Symptom:** CSV export of projects, when filtered, does not include anonymous
projects on the Rapid site.
**Workaround:** Export without filters and filter in a spreadsheet, or wait for
the platform fix.

### Logged-out app error on custom-admin pages (2026-06-08)
**Status:** Platform rendering-order behaviour. Confirmed, with a working fix.
**Symptom:** Visiting a `shopper-admin` custom-admin page (e.g. `manage/dashboard`) while
logged out throws an app error.
**Cause:** The page body document renders **before** the `shopper-admin` layout's
`{% if user.is_admin %}` gate is evaluated. Admin-only snippet calls placed in the page
body (for example `{% snippet 'admin/forms/checkbox' %}`) therefore execute for
unauthenticated visitors and throw, because the layout gate has not run yet.
**Fix:** Wrap the entire page-body content in its own `{% if user.is_admin %} ... {% endif %}`
guard rather than relying on the layout gate. Alternatively, pass `fallback_content` to any
admin-only snippet call so a missing/blocked snippet degrades gracefully instead of erroring.
**Related:** Bootstrap modal CSS/JS is **not** active on the `shopper-admin` instance, so a
Bootstrap `.modal` renders as unstyled inline content. For modals inside custom admin, use a
pure-CSS `:target` toggle (or another no-JS pattern) scoped under a `pf-` prefix to avoid
clashing with the `s-` design-system classes.

### Add to Cart button carries no `type` attribute (2026-08-10)
**Status:** Confirmed on live product pages.
**Symptom:** Custom JavaScript that resolves the cart button as
`form.querySelector('button[type="submit"], input[type="submit"]')` gets `null`, so any
programmatic enable or `.click()` does nothing. A `.click()` on a still-disabled button is
silently swallowed — no error, no navigation, and the user simply stays on the page.
**Cause:** The real control is
`<button class="btn btn-block btn-primary add-to-cart-button">ADD TO CART · $20.00</button>`.
A `<button>` inside a form submits by default, but an attribute selector matches only the
*literal* attribute, and `type` is absent.
**Fix:** Resolve by class and visible text — `.add-to-cart-button`, or
`/add[\s_-]*to[\s_-]*(cart|basket|bag)/i` — never by `button[type="submit"]` alone.

## The Add to Cart control

Measured on a live Shopper product page, 10 Aug 2026.

```html
<form class="project_create">
	…
	<px-option code="…">…</px-option>
	…
	<button class="btn btn-block btn-primary add-to-cart-button">ADD TO CART · $20.00</button>
</form>
```

Three things matter to anything that scripts against it:

- **The form is `.project_create`**, not `.product-form`. The artwork/option elements are
  inside it, so `closest('form')` from a `px-option` reaches it reliably.
- **The button carries NO `type` attribute.** A `<button>` inside a form submits by
  default, but `form.querySelector('button[type="submit"]')` matches only the *literal*
  attribute and therefore **returns null**. This is a silent failure: the selector finds
  nothing, whatever depended on it quietly does not happen, and no error is raised.
- **Resolve it by text and class instead:**

```js
	/add[\s_-]*to[\s_-]*(cart|basket|bag)/i
```

  tested against the element's `textContent`, `value`, `aria-label`, `title`, `id` and
  `className`. That matches both the visible label and the `add-to-cart-button` class.

**A disabled button ignores `.click()` silently.** Anything that programmatically submits
the product form must re-enable the button first, and must therefore be able to find it.
The two failures compound: a resolver that returns null cannot re-enable anything, so the
click is swallowed and the customer stays on the product page with no error shown.

### Reading the product price from JavaScript (2026-08-10)
**Status:** Confirmed. Applies to any custom tool or snippet mirroring the live price.
- The price is rendered by a `px-product-price` web component, not by static markup.
  Read that element, and fall back to its `initial` attribute.
- **Do not scrape `.product-price` text.** On any product with
  `product.custom.regular_pricing` set, `product/product-details` renders the struck-through
  pre-discount price **first** inside the same block, so a non-global regex returns the wrong
  number. Verified: a product selling at $29.00 reported $45.00. If a text scrape is
  unavoidable, strip `<s>` and `<del>` content first.
- **A `MutationObserver` must sit on the `px-product-price` element itself.** It replaces its
  own contents, so watching a parent leaves the reader stale after a variant change.
- **Do not compute per-unit price by dividing total by quantity in JavaScript.** It disagrees
  with the platform on rounding, and on any ladder with a fixed component it is a different
  number. When `display_each_pricing: true` is set on the product, the page already renders
  the platform's own per-unit figure in a second `px-product-price` instance carrying
  `unit-price="true"` — read that.

### A child's CMS backup can hold a stale copy of a parent snippet (2026-08-10)
**Status:** Confirmed on a live child site.
**Symptom:** A snippet present in the child's CMS backup is **not** what the site renders.
Observed on `collection/collection-filters-static`, where the backup's version built product
cards one way and the live render (inherited from the parent) built them another, with markup
and scripts in the live version that appear nowhere in the backup.
**Cause:** The child had no active override. The parent's newer snippet was rendering, and the
backup carried an inherited copy from whenever it was last synced.
**Two consequences:**
1. The backup is not a reliable picture of the parent. A snippet being *present* in the
   backup does not mean its *content* matches the parent's.
2. "Copy it down and edit it" silently reverts parent improvements — filter fixes,
   accessibility work, new card fields — with nothing in the diff to notice, because the diff
   is against the stale copy.
**Rule:** Before overriding any snippet, get its current source from the parent admin or from
the rendered page, never from the child's backup. Then change only what must change and leave
the rest verbatim.
**Prefer not to override at all when the change is presentational.** Scoped CSS on the
section wrapper does the job without freezing hundreds of lines of parent logic, and stays
correct when the parent snippet moves on.
**Corollary:** the parent-first rule (a child can only override a snippet the parent already
has) still holds, but the inverse does not — a snippet **absent** from the child's backup may
well exist on the parent and be perfectly legal to override for the first time. Confirm from
the parent admin rather than treating absence as proof it does not exist.

## Changelog
- 2026-03-14: Added website/homepage snippet pattern and Custom Admin checkbox requirement to Section 13.
- 2026-03-19: Added how to create pages with Custom Types to Section 14.
- 2026-04-08: Updated how to create pages with Custom Types to Section 14.
- 2026-04-10: Added Section 15 — Known Gotchas (image slider refresh, date input browser bug, Worker JS SEO, CSV export anonymous projects).
- 2026-04-20: Section 13 — clarified nav link editing must target the active nav style snippet based on header-logo-position checklist value.
- 2026-05-19: Added `shopper-admin` layout to layouts table. Added Section 15 — Custom Admin manage/ page inventory including tools pages. Added Section 16 — Kiosk Touchscreen Mode architecture and status. Renumbered Known Gotchas to Section 17. Source: Claude chats (admin v2 work, kiosk mode design).
- 2026-06-01: Added Shop All page feature note. Source: fireflies-call.
- 2026-06-15: Added Static Product Importer CSV column spec under Tools pages. Added Google Ads conversion-tracking note (no built-in preset; deploy via GTM). Added Known Gotcha: logged-out app error on custom-admin pages (page body renders before the layout user.is_admin gate; Bootstrap modal CSS inactive in custom admin). Source: claude-chat.
- 2026-07-20: Added kiosk captcha per-subdomain note — captcha config does not carry from the main storefront to the kiosk subdomain and must be set on the kiosk site. Source: support-ticket.
- 2026-08-05: Corrected `page_path` to store the full slash-joined path at levels 2 and 3, not only the final segment, and corrected the constraint that wrongly stated level 1 only. Added Head-level dependencies must repeat the lookup, covering the `html.head` before `page.content` render order, the paste-ready head lookup block, the `!= blank` guard, and the per-page noindex pattern via a boolean `hide_from_index` custom field. Source: claude-chat.
- 2026-08-11: Added Value-bearing checklists to Section 5 — many `admin/checklist/*` keys hold interpolated values, and overwriting one with a boolean takes every page down; includes the known value-bearing key list, the case-sensitivity and `| strip` capture rules, and the `custom-X-page` flag against an empty target snippet. Added three Known Gotchas: the Add to Cart button carries no `type` attribute; reading the product price from JavaScript (`px-product-price`, the `regular_pricing` strikethrough trap, observer placement, and `unit-price="true"` instead of JS division); and a child's CMS backup can hold a stale inherited copy of a parent snippet. Source: claude-chat.


=================================================================
FILE: 51_CUSTOM_FIELDS_REFERENCE.md
=================================================================

# Pixfizz Shopper v2 Custom Fields Master Reference

**Last Updated:** 2026-06-30

---

## Table of Contents

1. [Summary & Statistics](#summary--statistics)
2. [Key Notes](#key-notes)
3. [Object Type Reference](#object-type-reference)
   - [Product (81 total)](#product-81-total)
   - [Collection (53 fields)](#collection-53-fields)
   - [Design (25 fields)](#design-25-fields)
   - [Post (36 fields across groups)](#post-36-fields-across-groups)
   - [Option (13 fields)](#option-13-fields)
   - [Cart (11 fields)](#cart-11-fields)
   - [User (7 fields)](#user-7-fields)
   - [Order (4 fields)](#order-4-fields)
   - [Address (3 fields)](#address-3-fields)
   - [Subcollection (6 fields)](#subcollection-6-fields)
   - [Promotion (6 fields)](#promotion-6-fields)
   - [Custom_Page (5 fields)](#custom_page-5-fields)
   - [Blog_Post Detail (8 fields)](#blog_post-detail-8-fields)
   - [Service Detail (9 fields)](#service-detail-9-fields)
   - [Business Detail (2 fields)](#business-detail-2-fields)
   - [Value (3 fields)](#value-3-fields)
   - [Variant (3 fields)](#variant-3-fields)
   - [Template_Option (2 fields)](#template_option-2-fields)
   - [Static_Product (2 fields)](#static_product-2-fields)
   - [Promo (2 fields)](#promo-2-fields)
   - [Google_Summary (2 fields)](#google_summary-2-fields)
   - [Top_Collection (1 field)](#top_collection-1-field)
   - [Project (1 field)](#project-1-field)
   - [Page (1 field)](#page-1-field)
   - [School (1 field)](#school-1-field)
   - [Club (1 field)](#club-1-field)
   - [Charity (1 field)](#charity-1-field)
   - [First_Static_Product (1 field)](#first_static_product-1-field)
   - [Next_Option (1 field)](#next_option-1-field)
   - [Previous_Option (1 field)](#previous_option-1-field)
4. [Next Steps](#next-steps)

---

## Summary & Statistics

This reference documents **30 object access patterns** mapping to approximately **15 actual CMS object types** discovered during comprehensive v2 codebase analysis.

| Object Type | Field Count | Notes |
|-------------|-------------|-------|
| Product | 81 (67 in code + 14 export-only) | Largest object type |
| Collection | 53 | Second largest |
| Post (all groups) | 36 | Shared across 7 post group types |
| Design | 25 | Theme/design configuration |
| Option | 13 | Product option handling |
| Cart | 11 | Checkout form inputs |
| Blog_Post | 8 | Detail fields |
| Service | 9 | Detail fields |
| User | 7 | Checkout form inputs |
| Subcollection | 6 | Collection child object |
| Promotion | 6 | Post group type |
| Custom_Page | 5 | Custom page fields |
| Order | 4 | Order data fields |
| Value | 3 | Option value representation |
| Address | 3 | Shipping/billing address |
| Variant | 3 | Cart item variant (via option.variant) |
| Template_Option | 2 | Cart item template option (via option.template_option) |
| Static_Product | 2 | Accessed in specific contexts |
| Promo | 2 | Promotion context variant |
| Google_Summary | 2 | Google review summary |
| Business | 2 | Detail fields |
| Single-field objects | 11 | Top_Collection, Project, Page, School, Club, Charity, First_Static_Product, Next_Option, Previous_Option |

**Total: 30 access patterns across ~15 actual CMS object types**

---

## Key Notes

### Object Type Relationships

- **Post Object Family**: `post`, `blog_post`, `service`, and `business` are all **the same CMS Post object** accessed in different page contexts (blog listing, service listing, business directory, or promotions modal). Custom field definitions are shared across all post groups.

- **Post Groups**: The Post object stores data using different field naming conventions depending on context:
  - Blog posts: `blog_*` fields
  - Service posts: `service_*` fields
  - Business posts: `business_*` fields
  - FAQ posts: `faq_*` fields
  - Promotion posts: `promo_*` fields
  - Our Work posts: `work_*` fields
  - Google Review posts: `google_*` fields

- **Promo Variants**: `promo` and `promotion` are different variable names for **Post objects** in the promotions modal context. `promo` accesses minimal fields; `promotion` accesses full post context.

- **Option Nesting**: `template_option` and `variant` are accessed via `option.template_option` and `option.variant` respectively in cart item loops.

- **Collection Nesting**: `subcollection` and `top_collection` are Collection objects accessed as children and parents of a parent collection.

- **Product Context**: `first_static_product` is a Product object accessed as the first static product in a collection.

- **Checkout Form Input**: Cart and User custom fields are **set via HTML form inputs** during the checkout process, not via CMS admin interface. These fields store shopper-provided data.

- **There is no `html` field type.** Confirmed twice (custom type schema editor, and the Product Attributes → Custom Field Schema → New Field dropdown). The type list is exactly **text · multitext · boolean · number · asset · snippet**. Entries in this file previously typed as `html` have been corrected to `snippet`.

- **What each non-string type returns in Liquid**: `snippet` returns the **rendered snippet**; `asset` returns an **`Asset` object** (pass it through the `asset_url` filter); everything else returns a plain string, or nil. A filename held in an `asset`-type field and the same filename held in a `text`-type field both work through `asset_url`, but they are not the same value — one is an object, one is a string. Pick one type per field and keep every product consistent, or fallback logic such as `{% if x != blank %}` behaves differently product to product.

- **The `Public` flag controls non-admin EDIT rights, not storefront visibility.** `product.custom.<field>` renders from Liquid whether or not Public is ticked. Leave it unticked for any field that exists purely to feed a template, which is nearly all of them. Tick it only where a non-admin user is meant to change the value themselves. The name reads like a visibility switch, and the cautious default of ticking it "so the storefront can see it" silently hands edit rights on template-critical fields to non-admin users.

- **Large content: snippet-type only.** A snippet-type custom field holds 20,000+ characters, returned byte-clean via `item.custom.<field>` with no entity encoding and round-tripping through `parse_json` intact. A **text-type field does not** — the practical cap is small, on the order of ~1KB. (Exact text-type limit still to be pinned; treat anything over ~1KB as snippet-type.) Snippet-type fields are conventionally used to *reference* a snippet but can also hold content directly. This is a separate limit from the ~2KB cap on **template options**.

- **Field type can differ by object for the same field name**: the product tab fields `details`, `features` and `production` render as markup at the Collection level but not at the Product level. Tab content authored as HTML belongs on the Collection, not on the individual product export.

- **New products start with blank custom field values**: field *definitions* exist on the site, but values default to blank (and boolean fields to false) on every newly created product. An export showing empty custom fields is expected behaviour, not a failed export.

- **`manage/custom-fields` is the in-CMS field authority**: the CMS carries a maintained reference page listing every custom field with its object type, field type and description, including fields that are missing from import tars. Where a field's type or purpose is ambiguous, that page outranks any export file.

- **Export-Only Fields**: Product has 14 fields defined in the CMS export but **not referenced in any template**. These are reserved for platform-level features including:
  - Gift Finder (budget_tier, gift_occasion, gift_type)
  - Lab production routing (lab_printer, lab_size, oversize)
  - Variant conversion (quantity_from_variants)
  - Future filtering and production workflows

---
- **Snippet-type custom field rendering requires a non-empty product Description**
  Custom fields of type `snippet` (features, details, options, pricing, production, additional tabs)
  are silently suppressed on the storefront if the product `description` field is empty — even if
  the snippet content is fully populated and saved in admin. Simple-type fields (production_time,
  promotion_badge, shipping flags) are unaffected. The Public toggle has no bearing on this.

  **Fix:** Add any text to the product Description before filling snippet custom field content.
  If the Description must appear blank to shoppers, use a non-breaking space or invisible character.

  RATIONALE: Silent failure with no admin warning. Repeat signal — #kb-sync message + Fireflies call (Xenia, Aug 18).
  SOURCE TYPE: slack-message + fireflies-call

  ---
## Object Type Reference

### Product (81 total)

**67 fields in code + 14 export-only fields**

#### Template-Referenced Product Fields (67)

| Field | Type | Description |
|-------|------|-------------|
| add_to_cart | boolean | Legacy flag for add-to-cart flow in saved projects |
| hide_from_search | boolean | Exclude this product from the storefront search flyout. Only affects the storefront search flyout — the product stays active elsewhere, including in POS. Per-design hiding is also supported via design.custom.hide_from_search |
| additional | snippet | 'Additional Info' tab content. Falls back to collection if not set |
| btn_add_to_cart | boolean | Show 'Add to Cart' button, skip Design Tool |
| btn_buy_now_design_later | boolean | Show 'Buy Now, Design Later' button |
| btn_design_tool | boolean | Show 'Design More' button (displayed with btn_add_to_cart) |
| cart_product_name_format | text | Format for cart line item name: 'design', 'both', or blank (default) |
| class | text | Display class for static collection filters and grouping |
| custom_pricing | text | Custom pricing text display, e.g. 'From 5 for $10' |
| details | snippet | 'Details' tab content on product page |
| disable_required_form | boolean | Bypass HTML5 form validation on design-now form |
| display_each_pricing | boolean | Show per-unit pricing on quantity selector |
| do_not_save | boolean | Add to cart only, do not create saved project |
| dynamic_production_time | boolean | Enable dynamic production time calculation |
| envelope_imprinting | text | Envelope design code (must be in 'envelopes-with-cards' collection) |
| extra_charge | number | Extra shipping or handling fee in cents |
| features | snippet | 'Features' tab content on product page |
| film_processing | boolean | Show film drop-in vs mail-in option selector |
| from_pricing | number | Minimum price for tiered pricing display |
| gallery_stacked | boolean | Stack preview images vertically instead of grid |
| google_category | text | Google Product Taxonomy code for feed |
| gtin | text | Product GTIN (Global Trade Item Number) |
| hide_design_options | boolean | Hide design options selector panel |
| hide_edit_in_cart | boolean | Hide 'Edit' button in cart for this product |
| hide_select_month | boolean | Year-only selection on calendar products, hide month selector |
| hide_tooltip | boolean | Disable option tooltips on design-now form |
| meta_description | text | SEO meta description |
| meta_title | text | SEO meta title |
| mpn | text | Manufacturer product number |
| name_alt | text | Superscript alternate name on Shop page listings |
| options | snippet | 'Options' tab content on product page |
| orientation | text | Product orientation filter value (e.g., 'landscape', 'portrait') |
| pickup_unavailable | boolean | Disable local pickup option |
| prepay_only | boolean | Force prepay checkout, disable pay-on-delivery |
| preview_alt_tag | text | Alt tag for live preview images |
| pricing | snippet | 'Pricing' tab content on product page |
| product_filters | snippet | Wildcard filters applied on product page |
| product_footer | snippet | Content footer at bottom of product page |
| product_type_custom | text | Custom product type for feed generation |
| production | snippet | 'Production & Shipping' tab content on product page |
| production_time | number | Days for production (0 = same day production) |
| promotion | snippet | Campaign promotional message displayed on product |
| promotion_badge | text | Badge text: 'On Sale', 'Best Seller', etc. |
| promotion_message | text | Custom promotional message |
| regular_pricing | text | Regular price displayed with strikethrough + SALE badge |
| remove_live_preview_img_schema | boolean | Exclude live preview image from schema.org markup |
| select_date | boolean | Show month/year selection on product page |
| select_page_count | boolean | Page count dropdown (photo books) |
| select_page_count_label | text | Custom page count label on selector |
| select_quantity | boolean | Quantity input field on product page |
| shipping | text | Legacy: 'false' disables shipping for this product |
| shipping_unavailable | boolean | Disable shipping option |
| size | text | Product size filter value (e.g., '4x6', '8x10') |
| skip_cart_redirect | boolean | Stay on product page after add-to-cart instead of redirect |
| sold_out | boolean | Disable purchase buttons, mark as sold out |
| special_promo | text | Promo message displayed below cart line item |
| spreads_as_pages | boolean | TRUE doubles the number shown in the page-count selector (display only). The submitted `book[pages]` value, pricing, and fulfillment are unchanged. |
| starting_at | number | Starting price for tiered pricing |
| strikethrough | number | Original price displayed as strikethrough |
| to_pricing | number | Maximum price for price range comparison |
| upsell_img | asset | Cart upsell banner image |
| upsell_label | text | Cart upsell call-to-action label |
| upsell_link | text | Cart upsell URL |
| url | text | Custom URL for structured data |
| url_parameter | text | URL parameter appended to product URL |
| url_path | text | Custom path for static product URL slug |
| video | asset | Product video (webm format) for gallery display |
| requires_design | boolean | Platform-level gate: holds the Add to Cart button until the orderline carries a design. Correct and safe to leave TRUE on products driven by a custom design tool — see the note below |

### `requires_design` on custom-design-tool products

`requires_design: true` makes the platform disable the Add to Cart button. That is
correct and is usually what you want, including on products driven by a custom design
tool.

A tool that submits the product form itself must **release the button before clicking
it** — a disabled button ignores `.click()` silently. If the tool's button resolver is
wrong, the release never happens and the order dies at the last step with no error.

**The symptom is: tool completes, Add to Cart does nothing, customer stays on the product
page, nothing in the console.** That symptom has been misattributed to `requires_design`
itself at least three times. It is not the flag. Verified 10 Aug 2026 by exporting two
products on the same site — one shipping orders, one failing — and diffing them field by
field: **zero differing fields, both `requires_design: true`.** The difference was
entirely in the tool's JavaScript.

Do not set `requires_design: false` as a workaround. Fix the resolver.

#### Export-Only Product Fields (14 — not referenced in templates)

Reserved for platform-level features, production routing, and future functionality.

| Field | Type | Description |
|-------|------|-------------|
| budget_tier | text | Gift Finder budget tier classification |
| category | multitext | Product categories for filtering and organization |
| gift_occasion | multitext | Gift Finder occasion tags (e.g., 'Birthday', 'Wedding') |
| gift_type | text | Gift Finder gift type classification |
| invoice_preview_page | number | Invoice preview page number for product |
| lab_printer | boolean | Lab printer workflow flag for production routing |
| lab_size | text | Lab size code for production routing decisions |
| orientation_img | asset | Custom orientation illustration image |
| oversize | text | Oversize shipping extra fee code |
| product_name | text | Override product name on collection listings |
| quantity_from_variants | boolean | Convert variants to quantity inputs for checkout |
| ratio | text | Product image aspect ratio (e.g., '4x6', '1x1') |
| shop_by_price | text | Price tier for shop price-based filtering |
| style | multitext | Left-hand side filter category values |

---

### Collection (53 fields)

| Field | Type | Description |
|-------|------|-------------|
| additional | snippet | Additional collapsible details section on product page |
| banner | asset | Banner image displayed at top of collection |
| banner_html | snippet | HTML content displayed below banner image |
| breadcrumb | string | Breadcrumb format: 'product' or design name |
| btn_add_to_cart | boolean | Show 'Add to Cart' on product detail page |
| btn_buy_now_design_later | boolean | Show 'Buy Now & Design Later' button on PDP |
| btn_design_tool | boolean | Show 'Design Tool' button on PDP |
| collection_filters | text | Filter configuration, one per line: "Label \| URL \| Attribute" |
| collection_footer | snippet | Footer content on collection and product pages |
| collection_pdp_filters | text | Product page-specific filter configuration |
| combine_design_static | boolean | Combine design and static products in same listing |
| contact_label | string | Label for contact section on design-now page |
| custom_filters | snippet | Custom filter UI rendered alongside standard filters |
| design_now_disable_required | boolean | Skip required field validation on design-now |
| details | snippet | 'Details' tab content on product page |
| disable_live_preview_on_shop | boolean | Hide live preview on collection listing page |
| disable_required_form | boolean | Skip HTML5 required field validation |
| dynamic_production_time | boolean | Calculate production time dynamically |
| features | snippet | 'Features' section content on product page |
| gallery_stacked | boolean | Display gallery images vertically instead of grid |
| google_category | string | Google Product Taxonomy category for schema.org |
| help_video | snippet | Embedded help video on product detail page |
| hide_collection_filters | boolean | Hide sidebar filter panel entirely |
| hide_delivery_options | boolean | Hide delivery/shipping option section on PDP |
| hide_design_options | boolean | Hide design options selector panel |
| hide_pricing | boolean | Hide pricing information on collection listings |
| hide_product_name | boolean | Hide product name on collection listings |
| load_more | boolean | Enable infinite scroll pagination |
| meta_description | string | SEO meta description for collection page |
| meta_title | string | SEO meta title for collection page |
| options | snippet | 'Options' tab content on product page |
| pdp_layout | boolean | Dual-mode product detail page layout |
| pricing | snippet | 'Pricing' tab content on product page |
| print_ux | boolean | Special photo prints product detail layout |
| production | snippet | 'Production & Shipping' tab on product page |
| production_time | number | Production time in days (0 = same-day) |
| production_time_custom | string | Custom production time text display |
| promotion | snippet | Promotional content displayed on cart page |
| promotion_message | string | Limited-time offer message |
| quickview | boolean | Enable quickview modal on collection listings |
| remove_live_preview_img_schema | boolean | Exclude preview images from schema.org markup |
| reverse | boolean | Reverse product sort order in collection |
| select_quantity | boolean | Enable quantity selector on checkout form |
| show_sub_collections | boolean | Display subcollections on collection page |
| starting_at | number | Starting price display |
| sticker_1 | asset | Badge/sticker image 1 |
| sticker_2 | asset | Badge/sticker image 2 |
| sub_collections | boolean | Enable subcollections with detail view |
| sub_collections_position | text | Subcollection render order relative to products on a collection page: ABOVE = before products, blank or BELOW = after products (default). Only applies when show_sub_collections is also enabled. Created on the parent, overridden per child site/collection. |
| subtitle | string | Subtitle displayed under collection heading |
| title_format | string | Title display format: 'design', 'both', 'collection', or blank |
| url_path_parameter | string | URL parameter added to product links |
| wildcard | boolean | Enable wildcard carousel display |
| wildcard_filter | boolean | Enable wildcard filtering on product page |

---

### Design (26 fields)

| Field | Type | Description |
|-------|------|-------------|
| badge | string | Badge text on product cards (e.g., 'Limited Edition') |
| cart_edit_url | string | Cart edit link target: 'editor' or 'project-edit' |
| disable_live_preview_on_shop | boolean | Disable preview on collection listing page |
| hide_from_search | boolean | Exclude this specific design from the storefront search flyout (per-design override) |
| disable_required_form | boolean | Bypass form field validation |
| display_name | string | Custom name overriding design code name |
| envelope_imprinting | string | Envelope design code reference for cart display |
| extra_info | string | Extra metadata for prints JSON output |
| features | snippet | Features content (cascade: design > product > collection) |
| holiday_dates | boolean | Enable holiday date picker on design-now |
| img_alt | string | Alt text for shop gallery images |
| meta_description | string | SEO meta description for design page |
| meta_title | string | SEO meta title for design page |
| msp_code | string | Marketplace SKU code for search/discovery |
| only_add_to_cart | boolean | Show only 'Add to Cart', hide 'Design Now' option |
| only_design_now | boolean | Show only 'Design Now', hide 'Add to Cart' option |
| personal_dates | boolean | Enable personal date picker on design-now |
| preview_alt_tag | string | Alt text for gallery preview images |
| promotion | snippet | Promotional content on cart page |
| promotion_badge | string | Badge on product cards and PDP (e.g., 'On Sale') |
| reverse_live_preview_on_shop | boolean | Swap front/back preview orientation on shop |
| shop_label | string | Custom label on product cards in shop |
| style | string | Style category for sidebar filter matching |
| url_parameter | string | Additional URL parameters for tracking |
| url_path | string | Primary custom URL slug |
| url_slug | string | Fallback URL slug when url_path not set |

---

### Post (36 fields across groups)

Post is a single CMS object type with context-dependent field naming. Fields are organized by post group type.

#### Blog Posts (blog_*)

| Field | Type | Description |
|-------|------|-------------|
| blog_date | text | Publication date |
| blog_description | text | Excerpt for blog listing |
| blog_path | text | URL slug for blog post |
| blog_thumbnail | asset | Thumbnail image for blog listing |
| blog_thumbnail_alt | text | Alt text for thumbnail image |
| blog_title | text | Blog post title |
| blog_unpublished | text | 'true' to hide from blog listing |

#### Service Posts (service_*)

| Field | Type | Description |
|-------|------|-------------|
| service_meta_description | text | SEO meta description / listing excerpt |
| service_path | text | URL slug for service page |
| service_thumbnail | asset | Thumbnail image for service listing |
| service_thumbnail_alt | text | Alt text for service thumbnail |
| service_title | text | Service name/title |
| service_unpublish | boolean | Hide service from listing |

#### FAQ Posts (faq_*)

| Field | Type | Description |
|-------|------|-------------|
| faq_content | snippet | Answer/content for FAQ item |
| faq_header | text | Question heading |
| faq_order | number | Sort order in FAQ listing |

#### Business Posts (business_*)

| Field | Type | Description |
|-------|------|-------------|
| business_meta_description | text | Business description for listing |
| business_path | text | URL slug for business directory listing |
| business_thumbnail | asset | Business photo/logo thumbnail |
| business_thumbnail_alt | text | Alt text for thumbnail |
| business_title | text | Business name |

#### Promotion Posts (promo_*)

| Field | Type | Description |
|-------|------|-------------|
| promo_end_date | text | Promotion end date |
| promo_img | asset | Promotion card background image |
| promo_message | text | Description text |
| promo_name | text | Promotion title |

#### Our Work Posts (work_*)

| Field | Type | Description |
|-------|------|-------------|
| work_description | text | Project description |
| work_image | asset | Portfolio gallery image |
| work_image_alt | text | Alt text for image |
| work_title | text | Project title |

#### Google Review Posts (google_*)

| Field | Type | Description |
|-------|------|-------------|
| google_name | text | Reviewer name |
| google_publish_time | text | Review publication date |
| google_rating | number | Star rating (1-5) |
| google_text | text | Review content/text |

#### Generic/Top Picks Posts

| Field | Type | Description |
|-------|------|-------------|
| image | asset | Post image |
| path | text | Link path/URL |
| title | text | Title |

---

### Option (13 fields)

| Field | Type | Description |
|-------|------|-------------|
| description | text | Option description shown to shopper |
| display | boolean | Show option on product page |
| snippet | snippet | Custom HTML for option rendering |
| img | asset | Option preview image |
| img_alt | text | Alt text for preview image |
| label | text | Display label for option |
| name | text | Internal option name |
| order | number | Sort order in option list |
| required | boolean | Mark option as required |
| selected | boolean | Default selected state |
| template_option | object | Template option metadata (accessed via option.template_option) |
| type | text | Option type (select, checkbox, radio, etc.) |
| variant | object | Variant data (accessed via option.variant) |

---

### Cart (16 fields)

**Note:** Cart custom fields are set via HTML form inputs during checkout, not CMS admin.

| Field | Type | Description |
|-------|------|-------------|
| expedited_shipping | text | Expedited shipping selection from checkout form |
| gift_message | text | Gift message entered by shopper |
| gift_wrap | boolean | Gift wrap option selected |
| locale | text | Shopper's locale preference |
| note | text | Order note from checkout form |
| option1 | boolean | Generic white-labelled order flag, wording configured per client. Reaches OrderHub once whitelisted. Rolling out from 2026-08. |
| option2 | boolean | Generic white-labelled order flag. Rolling out from 2026-08. |
| option3 | boolean | Generic white-labelled order flag. Rolling out from 2026-08. |
| phone | text | Shopper phone number from form |
| preferred_delivery | text | Preferred delivery method |
| rush | boolean | Standard rush delivery tier. Mutually exclusive with `urgent`. Rolling out from 2026-08. |
| rush_production | boolean | Rush production option selected (older single-purpose flag; superseded by `rush` on sites moved to the delivery-speed radio group) |
| scheduled_delivery_date | text | Scheduled delivery date if applicable |
| special_instructions | text | Special handling instructions |
| timezone | text | Shopper's timezone for scheduling |
| urgent | boolean | Faster-than-rush delivery tier (typically same day). Mutually exclusive with `rush`. Rolling out from 2026-08. |

**Naming.** `option1`–`option3` carry no underscore. All five flags must be
lowercase and whitelisted in OrderHub before they route — see
`45_ORDERHUB.md` § The five order-level boolean slots.

**Value convention.** Write the explicit strings `true` and `false`, never blank.
Blank is not confirmed to clear a cart custom field, and a flag that sticks on
`true` keeps charging the customer after they switch back. Because `false` is a
non-empty string, every Liquid test must be `== 'true'`; a blank value means the
field was never set, which reads as off.

---

### User (7 fields)

**Note:** User custom fields are set via HTML form inputs during checkout, not CMS admin.

| Field | Type | Description |
|-------|------|-------------|
| company | text | Shopper company name from checkout |
| email | text | Shopper email address |
| first_name | text | Shopper first name |
| last_name | text | Shopper last name |
| phone | text | Shopper phone number |
| preferences | text | Shopper communication preferences |
| title | text | Shopper job title |

---

### Order (5 fields)

| Field | Type | Description |
|-------|------|-------------|
| gift_message | text | Order-level gift message |
| order_notes | text | Internal order notes |
| special_handling | text | Special handling requirements |
| store_id | text | Per-store order tracking identifier (set at order time; used for multi-location fulfillment routing). Added 2026-02-27. |
| tracking_update_preference | text | Shopper's tracking notification preference |

---

### Address (4 fields)

| Field | Type | Description |
|-------|------|-------------|
| apartment | text | Apartment/unit/suite number |
| hide_address | boolean | Hide the address lines from the customer-facing UI while keeping the address data in the backend. Used for named pickup locations where the shopper should not see the physical address, but the address is still needed for order routing/fulfillment. |
| instructions | text | Delivery instructions |
| phone | text | Address-specific phone number |

---

### Subcollection (6 fields)

**Subcollection is a Collection object accessed as a child of a parent collection.**

| Field | Type | Description |
|-------|------|-------------|
| description | text | Subcollection description |
| image | asset | Subcollection thumbnail image |
| meta_description | string | SEO meta description |
| meta_title | string | SEO meta title |
| title | text | Subcollection display name |
| url_path | text | URL slug for subcollection |

---

### Promotion (6 fields)

**Promotion is a Post object accessed in promotions modal context.**

| Field | Type | Description |
|-------|------|-------------|
| active | boolean | Promotion is active/displayed |
| discount_code | text | Discount code if applicable |
| end_date | text | Promotion end date |
| promotion_message | text | Promotion description |
| promotion_name | text | Promotion title |
| start_date | text | Promotion start date |

---

### Custom_Page (5 fields)

| Field | Type | Description |
|-------|------|-------------|
| content | snippet | Page content |
| meta_description | text | SEO meta description |
| meta_title | text | SEO meta title |
| title | text | Page title |
| url_path | text | Custom page URL slug |

---

### Blog_Post Detail (8 fields)

**Blog_Post accessed as a detail/item object (vs. the Post listing context).**

| Field | Type | Description |
|-------|------|-------------|
| author | text | Blog post author name |
| blog_date | text | Publication date |
| blog_description | text | Post excerpt |
| blog_path | text | URL slug |
| blog_thumbnail | asset | Feature image |
| blog_thumbnail_alt | text | Image alt text |
| blog_title | text | Post title |
| content | snippet | Full post content |

---

### Service Detail (9 fields)

**Service accessed as a detail/item object (vs. the Post listing context).**

| Field | Type | Description |
|-------|------|-------------|
| content | snippet | Service page full content |
| service_description | text | Service excerpt |
| service_meta_description | text | SEO meta description |
| service_path | text | URL slug |
| service_thumbnail | asset | Service image/icon |
| service_thumbnail_alt | text | Image alt text |
| service_title | text | Service name |
| contact_info | text | Service contact information |
| related_services | text | Links to related services |

---

### Business Detail (2 fields)

**Business accessed as a detail/item object (vs. the Post listing context).**

| Field | Type | Description |
|-------|------|-------------|
| business_title | text | Business name |
| business_description | text | Business description/info |

---

### Value (3 fields)

**Value is typically accessed within option value iteration.**

| Field | Type | Description |
|-------|------|-------------|
| label | text | Display label for value |
| name | text | Internal value name |
| price | number | Price adjustment if applicable |

---

### Variant (3 fields)

**Variant is accessed via `option.variant` in cart loops.**

| Field | Type | Description |
|-------|------|-------------|
| id | text | Variant identifier |
| label | text | Display label for variant |
| price_adjustment | number | Price modification from base product |

---

### Template_Option (2 fields)

**Template_Option is accessed via `option.template_option` in cart loops.**

| Field | Type | Description |
|-------|------|-------------|
| label | text | Template option display name |
| value | text | Selected template option value |

---

### Static_Product (2 fields)

| Field | Type | Description |
|-------|------|-------------|
| name | text | Static product name |
| price | number | Static product price |

---

### Promo (2 fields)

**Promo is a Post object in minimal promotions modal context (vs. full Promotion).**

| Field | Type | Description |
|-------|------|-------------|
| promo_name | text | Promotion title |
| promo_message | text | Promotion description |

---

### Google_Summary (2 fields)

**Google_Summary for Google reviews/ratings display.**

| Field | Type | Description |
|-------|------|-------------|
| rating | number | Average star rating |
| review_count | number | Number of reviews |

---

### Top_Collection (1 field)

**Top_Collection is a Collection object accessed as a parent collection.**

| Field | Type | Description |
|-------|------|-------------|
| name | text | Parent collection name |

---

### Project (1 field)

| Field | Type | Description |
|-------|------|-------------|
| name | text | Project name/title |

---

### Page (1 field)

| Field | Type | Description |
|-------|------|-------------|
| title | text | Page title |

---

### School (1 field)

| Field | Type | Description |
|-------|------|-------------|
| name | text | School name |

---

### Club (1 field)

| Field | Type | Description |
|-------|------|-------------|
| name | text | Club name |

---

### Charity (1 field)

| Field | Type | Description |
|-------|------|-------------|
| name | text | Charity organization name |

---

### First_Static_Product (1 field)

**First_Static_Product is a Product object accessed as the first static product in a collection.**

| Field | Type | Description |
|-------|------|-------------|
| name | text | First static product name |

---

### Next_Option (1 field)

| Field | Type | Description |
|-------|------|-------------|
| label | text | Next option/button label |

---

### Previous_Option (1 field)

| Field | Type | Description |
|-------|------|-------------|
| label | text | Previous option/button label |

---

## Next Steps

### Phase 1: Export Current Definitions

Export custom field definitions from CMS admin for each object type:

1. **Collection** — Export current custom field definitions
   - Target file: `__custom_field_definitions_collection.yml`
   
2. **Design** — Export current custom field definitions
   - Target file: `__custom_field_definitions_design.yml`
   
3. **Post (all groups)** — Export current custom field definitions
   - Target file: `__custom_field_definitions_post.yml`
   - Includes: blog_*, service_*, business_*, faq_*, promo_*, work_*, google_*
   
4. **Option** — Export current custom field definitions
   - Target file: `__custom_field_definitions_option.yml`
   
5. **User** — Export current custom field definitions
   - Target file: `__custom_field_definitions_user.yml`
   
6. **Order** — Export current custom field definitions
   - Target file: `__custom_field_definitions_order.yml`
   
7. **Address** — Export current custom field definitions
   - Target file: `__custom_field_definitions_address.yml`

### Phase 2: Diff & Identify Gaps

1. Compare exported definitions against this codebase reference
2. Identify missing field definitions
3. Document missing descriptions for fields lacking proper labeling
4. Flag fields with incorrect type classifications

### Phase 3: Add Missing Definitions

1. For each missing field, create definition with:
   - Proper field name (lowercase, snake_case)
   - Type (text, multitext, boolean, number, asset, snippet — there is no `html` type)
   - Description from this reference
   - Any relevant UI hints or validation rules

2. Prioritize by impact:
   - Product (65 in code)
   - Collection (53)
   - Post groups (37)
   - Design (25)
   - Option (13)

### Phase 4: Generate Import Packages

1. Create `__custom_field_definitions.yml` files for each object type
2. Package each as individual tar.gz:
   - `custom-field-definitions-collection.tar.gz`
   - `custom-field-definitions-design.tar.gz`
   - `custom-field-definitions-post.tar.gz`
   - `custom-field-definitions-option.tar.gz`
   - `custom-field-definitions-user.tar.gz`
   - `custom-field-definitions-order.tar.gz`
   - `custom-field-definitions-address.tar.gz`

3. Root-level tar structure (one file per archive):
   ```
   __custom_field_definitions_[object_type].yml
   ```

4. Import via CMS admin: Settings > Custom Fields > Import

---

**Reference Document Version:** 1.1  
**Generated From:** Comprehensive Pixfizz Shopper v2 codebase analysis  
**Maintenance:** Update this document when new custom fields are added or existing fields are modified.

---

## Site-Specific Custom Field Patterns

These are custom field patterns seen on individual client sites. They are **not
platform-level** and should not be added to the master Shopper v2 baseline — but
they are worth documenting as real-world examples for similar use cases.

### Wholesale / Retail / Margin Pattern (2026-02-10)

A wholesale/retail client site uses Product-level custom fields to store pricing inputs that
feed into the pricing formulas:

| Field | Type | Purpose |
|-------|------|---------|
| wholesale_price | number | Base wholesale cost (source of truth for margin calcs) |
| retail_price | number | Retail price for the product |
| margin | number | Margin % or fixed-margin adjustment used by the pricing formula |

These fields are **exposed in the template-import flow** so they can be set in bulk
when importing product templates, and they are **referenced directly from the Ruby
pricing formula** on the product (e.g. `retail_price` or `wholesale_price * (1 + margin/100.0)`).

This is a useful reference pattern for any wholesale/retail dual-pricing scenario
where the same product is sold under both models — the pricing variable lives on
the product as a custom field rather than as a global Price Variable, keeping
per-SKU variation local to the product.

## The per-product export archive is also an import format

Manage Products → Product Attributes → *product* → **Export** produces an
archive that **imports** back through Product Attributes → **Import**, one
product per archive. Unlike the Static Product Importer CSV — which creates flat
products with a single price and cannot create variants — this format carries
`variant_types`, each with a full `variant_values` list. A size ladder is
therefore expressible, which makes a large catalogue a generated artifact rather
than a five-minute-per-product hand build.

**Archive shape** (gzipped `.tar.gz`, same five-empty-media-directory convention
as the Custom Type instance archive):

```
./assets/  ./fonts/  ./glb_files/  ./images/  ./pdfs/     (all empty)
./__product.yml
```

`__product.yml` holds the product row, its `custom:` hash, `linked_assets`, and
`variant_types`, then the four `__*_map: {}` keys.

**Verified behaviour:**

- **A blank `id:` is accepted** at product, variant-type and variant-value
  level; the platform assigns its own. No id reservation, no collision handling.
- **An asset-type custom field is set from a plain filename string**, with
  `__asset_map: {}` left empty. The asset must already exist in Website →
  Assets, so import the CMS backup (which carries `asset_files/`) **before** the
  products.
- **Unset custom fields simply do not appear.** The `custom:` hash of a fresh
  product contains only the boolean schema fields at `false`. Absence is not an
  error and is not the same as an empty string.

**Untested — flag before relying on:** whether re-importing an archive whose
`code` already exists updates in place or creates a duplicate; whether
`linked_assets` drives the Preview Images panel; whether `image:` accepts a bare
filename the way an asset-type field does.

**If generating the YAML, reproduce Ruby Psych's whitespace exactly.** Psych
writes a nil as `key: ` (key, colon, one **trailing space**) and a mapping or
sequence key as `key:` with no trailing space. Python's `yaml.dump` gets both
wrong. Parse a real export, re-emit it, assert byte-identical, and refuse to run
otherwise — and keep that reference export next to the generator, or the
assertion has nothing to assert against.

**Naming rule that pairs with this:** put print dimensions in the variant name
(`8x10`, `16 x 20 in`, `50 x 70 cm`) so size-aware storefront features can read
them off the platform's own rendered controls rather than needing injected data
attributes.

---

## Changelog
- 2026-06-01: Added Collection field sub_collections_position (subcollection render order). Source: chat/slack/call.
- 2026-06-30: Added hide_from_search boolean (Product + Design) — excludes a product/design from the storefront search flyout. Deployed platform-wide on Shopper. Source: claude-chat, slack-message (#development).
- 2026-07-04: Clarified hide_from_search scope — affects the storefront search flyout only; the product remains available in POS. Source: Fireflies (2026-07-01).
- 2026-07-11: Added Address field hide_address (boolean) — suppresses address display in the customer-facing UI for pickup locations while keeping the backend address for order routing (Address count 3 → 4). Source: slack-message (#development, 2026-07-10).
- 2026-07-25: Added Product field spreads_as_pages (boolean, display-only page-count doubling on the page selector). Removed blog_meta_description from both the Blog Posts group and Blog_Post Detail tables — the field does not exist; Shopper SEO Settings exposes blog_post.custom.description and blog_post.custom.blog_description instead. Normalised Product counts to 81 total / 67 template-referenced (summary stats row had drifted to 79/65), Post to 36, Blog_Post Detail to 8. Source: claude-chat, fireflies-call.
- 2026-07-28: Added Key Notes entries — product tab fields (`details`, `features`, `production`) are snippet-type at Product level and html-type at Collection level; new products start with blank custom field values by design; `manage/custom-fields` is the in-CMS authority for field types and descriptions. Source: claude-chat.
- 2026-08-14: Added the checkout delivery-speed and generic order flags to the Cart table (`rush`, `urgent`, `option1`, `option2`, `option3`; count 11 → 16), with the no-underscore naming rule, the OrderHub lowercase/whitelist dependency, and the explicit `true`/`false` value convention. Noted `rush_production` as the older single-purpose flag it supersedes. Source: fireflies-call (2026-08-13), claude-chat.
- 2026-08-11: Corrected the field type list — there is no `html` type; all 18 table rows typed `html` changed to `snippet`, and the Phase 3 type list corrected to text/multitext/boolean/number/asset/snippet. Added Key Notes for what each non-string type returns in Liquid, the `Public` flag controlling non-admin edit rights rather than storefront visibility, and large-content capacity being snippet-type only (text-type caps around 1KB). Added Section — the per-product export archive as a bulk-creation format carrying variant types and values. Source: claude-chat (Shopper v2 verification kit, art-archive build).
- 2026-08-21: Added snippet-type custom field rendering gotcha (requires non-empty Description). Source: slack-message + fireflies-call.


=================================================================
FILE: 52_SNIPPET_INVENTORY.md
=================================================================

# 52 — Shopper Snippet Inventory

**Authority Scope:** Complete inventory of every snippet on the Shopper parent template (`shopper24.pixfizz.com`). Use this to identify snippet names, understand what renders what, and determine which snippet a child site needs to override.

_Source: CMS backup 2026-05-27. Total: 926 snippets across 23 namespaces._

_Last updated: 2026-05-27_

---

## How to Use This File

- To find a snippet by what it does: scan the namespace that matches (e.g. collection page issue → `collection/`, cart issue → `checkout/`).
- To identify which snippet renders a specific piece of UI: check the rendering chain notes at the top of each namespace section.
- To override a snippet on a child site: the snippet must exist on the parent (this list). Child sites can only override existing parent snippets — they cannot create new ones.
- Snippet paths use `/` notation. In the CMS filesystem, `/` is stored as `__` (e.g. `collection/banner` → `collection__banner`).

---

## Critical Rendering Chains

These are the most commonly needed "what renders what" chains. Start here before scanning individual snippets.

### Collection page → product cards

```
Page: shop/:collection-level-1
  → Routes to one of:
    - collection/collection-filters        (standard — most common)
    - collection/collection-load-more      (if collection.custom.load_more)
    - product/details-filter-dual-mode     (if collection.custom.pdp_layout)
    - product/product-details-prints       (if collection.custom.print_ux)
```

**Product cards on collection grids are rendered INLINE within `collection/collection-filters` and `collection/collection-load-more` — NOT as separate snippets.** The `product/cards/*` snippets exist but are not called from the main collection grid. They are standalone card components available for use in custom sections or dynamic blocks.

### "As low as" pricing on collection cards

Already built into the parent `collection/collection-filters` snippet. The pricing block checks `product.custom.from_pricing` — if populated, renders "As low as {price}" automatically. No override needed; just populate the `from_pricing` custom field on the product.

Pricing display priority on design product cards:
1. `product.custom.from_pricing` → "As low as {price}"
2. `product.custom.regular_pricing` → strikethrough + SALE badge + current price
3. `product.custom.custom_pricing` → free-text pricing string
4. Fallback → `product.price | currency`

### Navigation

```
Layout: index
  → admin/checklist/header-logo-position value determines nav style:
    - "LEFT"   → navigation/style1
    - "CENTER" → navigation/style3  (default)
```

### Product detail page

```
Page: product/:collection/:url-path
  → product/product-details              (standard)
  → product/product-details-filter       (with sidebar filters)
  → product/product-details-prints       (photo prints UX)
  → product/details-filter-dual-mode     (dual design+static)
```

---

## Root-Level Snippets (5)

These sit outside any namespace.

| Snippet | Description |
|---|---|
| `back-to-top` | Back-to-top scroll button |
| `calendar-date-form` | Calendar product date selection form |
| `footer` | Full site footer — newsletter signup, logo, social links, 4 columns |
| `gdpr-banner` | GDPR/cookie consent banner overlay |
| `html.head` | Global `<head>` content — analytics scripts, meta defaults, favicon, fonts, CSS includes |

---

## `account/` — Customer Account Pages (32 snippets)

Components for the customer account area (`/site/account-*` pages).

| Snippet | Description |
|---|---|
| `account/address-form` | Address form with US & Canadian state dropdowns |
| `account/custom-order-details` | Custom order details display |
| `account/custom-registration` | Custom registration form fields |
| `account/extra-group-project-code` | Extra code injection for group projects |
| `account/extra-order-code` | Extra code injection in order context |
| `account/extra-orderline-code` | Extra code injection per orderline |
| `account/galleries-index-v1` | Gallery listing page (v1) |
| `account/galleries-index-v2` | Gallery listing page (v2) |
| `account/gallery-item-modal` | Gallery item detail modal |
| `account/gallery-preview` | Gallery preview card (v1) |
| `account/gallery-preview-v2` | Gallery preview card (v2) |
| `account/gallery-shop-preview` | Gallery shop preview card |
| `account/gallery-show-v1` | Gallery detail page (v1) |
| `account/gallery-show-v2` | Gallery detail page (v2) |
| `account/home-login-form` | Home page login gate form |
| `account/login` | Login page content |
| `account/login-form` | Reusable login form component |
| `account/mailer_label` | Film mailer label for print |
| `account/navigation` | Account sidebar navigation |
| `account/password-reset-sent` | Password reset confirmation message |
| `account/personal-info` | Personal info edit form |
| `account/v2/addresses` | Account v2 — saved addresses |
| `account/v2/carts` | Account v2 — saved carts |
| `account/v2/dashboard` | Account v2 — dashboard overview |
| `account/v2/dates` | Account v2 — personal dates |
| `account/v2/empty-state` | Account v2 — empty state placeholder |
| `account/v2/galleries` | Account v2 — galleries listing |
| `account/v2/info` | Account v2 — personal info |
| `account/v2/order-details` | Account v2 — order detail view |
| `account/v2/orders` | Account v2 — order history |
| `account/v2/projects` | Account v2 — saved projects |
| `account/v2/sidebar` | Account v2 — sidebar navigation |

**Version note:** Account v2 snippets are gated by the `admin/checklist/account-v2` flag. Both v1 and v2 coexist on the parent.

---

## `admin/` — Admin Components (246 snippets)

### `admin/checklist/` — Feature Flags & Config (198 snippets)

Fully documented in `50_SHOPPER_TEMPLATE_REFERENCE.md` Section 5. Not repeated here. Key groups: account settings, cart/checkout, delivery, galleries, icons, kiosk mode, payment gateways, photo print sizes, product display, SEO, content pages.

### Other `admin/` Snippets (48 snippets)

| Snippet | Description |
|---|---|
| `admin/checklist-confirm-start-date` | Confirm start date form component |
| `admin/contribution-label` | Contribution/donation label text |
| `admin/forms/asset` | Admin form — asset upload field |
| `admin/forms/checkbox` | Admin form — checkbox field |
| `admin/forms/radio-button` | Admin form — single radio button |
| `admin/forms/radio-buttons` | Admin form — radio button group |
| `admin/forms/snippet` | Admin form — snippet editor field |
| `admin/guest-or-create-account-label` | Guest vs create account label text |
| `admin/header` | Admin page header bar |
| `admin/modals/favicon` | Favicon upload modal |
| `admin/modals/logo` | Logo upload modal |
| `admin/modals/ogimage` | OG image upload modal |
| `admin/order-filters` | Order management filter controls |
| `admin/orders/custom-filters` | Custom order filter definitions |
| `admin/orders/date-format` | Order date display format |
| `admin/orders/table-column4` | Custom 4th column in order table |
| `admin/orders/table-header` | Order table header row |
| `admin/pagination` | Admin list pagination |
| `admin/sidenav` | Admin sidebar navigation (full) |
| `admin/sidenav-clean` | Admin sidebar navigation (minimal) |
| `admin/starting-at-label` | "Starting at" label text |
| `admin/urlroot` | Admin URL root path |
| `admin/vat-rate` | VAT rate value |

---

## `checkout/` — Checkout Components (47 snippets)

| Snippet | Description |
|---|---|
| `checkout/authorizedotnet-checkout` | Authorize.net on-page credit card form |
| `checkout/available-payment-methods` | Payment method selector |
| `checkout/braintree-checkout` | Braintree on-page credit card form |
| `checkout/bridgepay-checkout` | BridgePay on-page credit card form |
| `checkout/cart-extra-product-details` | Additional content below product name in cart |
| `checkout/cart-note` | Cart notes text area |
| `checkout/cashondelivery` | Cash on delivery / pay-in-store option |
| `checkout/checkout-list-addresses` | List of user's saved addresses |
| `checkout/checkout-standard-addressform` | Single address form for checkout |
| `checkout/confirm_order_invoice` | Order confirmation with invoice |
| `checkout/cross-sell-collection` | Cross-sell product suggestions at checkout |
| `checkout/custom-cart-option` | Custom cart option field |
| `checkout/disclaimer` | Checkout disclaimer (forced acknowledgment) |
| `checkout/extra-cart-code` | Extra code injection in cart |
| `checkout/extra-cart-orderline-code` | Extra code injection per cart orderline |
| `checkout/extra-checkout-code` | Extra code injection at checkout |
| `checkout/extra-orderline-code` | Extra code injection per orderline |
| `checkout/film-processing-note` | Film processing drop-off/mail-in note |
| `checkout/guest-phone` | Guest checkout phone number field |
| `checkout/login-form` | Checkout login form |
| `checkout/min-charge-note` | Minimum charge notice |
| `checkout/no-rush-button` | Standard processing button (non-rush) |
| `checkout/only-group-redirect-message` | Group order redirect message |
| `checkout/payment-failed` | Payment failed page content |
| `checkout/payment-success` | Payment success page content |
| `checkout/pickup-contact-details` | Pickup contact details form |
| `checkout/prepay-only-message` | Prepay required message |
| `checkout/proceed-to-checkout` | Proceed to checkout button/section |
| `checkout/public-list-addresses` | Public address list (guest checkout) |
| `checkout/rush-button` | Rush processing button (primary) |
| `checkout/rush-button2` | Rush processing button (secondary/special) |
| `checkout/rush-disclaimer` | Rush processing disclaimer text |
| `checkout/rush-label` | Rush processing label |
| `checkout/rush-title` | Rush processing section title |
| `checkout/shipping-details` | Shipping address review section |
| `checkout/shipping-film-order-note` | Film order shipping note |
| `checkout/shipping-options` | Shipping method options |
| `checkout/shipping-service-extra-note` | Additional shipping service note |
| `checkout/shipping-unavailable-text` | Shipping unavailable message |
| `checkout/square-checkout` | Square on-page credit card form |
| `checkout/states-canada` | Canadian province dropdown options |
| `checkout/states-india` | Indian state dropdown options |
| `checkout/states-usa` | US state dropdown options |
| `checkout/states-usa-custom` | Custom US state dropdown |
| `checkout/stripe-checkout` | Stripe on-page credit card form |
| `checkout/trust-badge` | Checkout trust badge |
| `checkout/utm-cart-code` | UTM parameter capture for cart |

---

## `collection/` — Collection Page Components (10 snippets)

| Snippet | Description |
|---|---|
| `collection/banner` | Collection banner image display |
| `collection/collection-filters` | **Main collection grid with sidebar filters — contains inline product card rendering** |
| `collection/collection-filters-collapse` | Collection filters with collapsible panels |
| `collection/collection-filters-static` | Static (non-interactive) filter display |
| `collection/collection-filters-top` | Horizontal filter bar above grid |
| `collection/collection-load-more` | Collection grid with "load more" pagination (contains inline card rendering) |
| `collection/description` | Collection description display |
| `collection/shop-all` | "Shop all" collection page variant |
| `collection/sub-collection` | Subcollection card display |
| `collection/top-banner` | Full-width banner above collection grid |

**Key pattern:** `collection/collection-filters` is the default collection page renderer. It's called from `shop/:collection-level-1` (and level 2/3 variants). Product cards are rendered inline — not as separate snippet calls.

---

## `cookie-consent/` — Cookie Consent (2 snippets)

| Snippet | Description |
|---|---|
| `cookie-consent/banner` | Cookie consent banner content |
| `cookie-consent/show-hide` | Set to TRUE to enable cookie banner |

---

## `editor/` — Design Tool (1 snippet)

| Snippet | Description |
|---|---|
| `editor/scripts.js` | Custom JS injected into the design tool editor iframe |

---

## `email-notifications/` — Legacy Email System (13 snippets)

| Snippet | Description |
|---|---|
| `email-notifications/button` | Email CTA button component |
| `email-notifications/content/abandoned-cart` | Abandoned cart email content |
| `email-notifications/content/order-confirmed` | Order confirmation email content |
| `email-notifications/content/order-pending` | Order pending email content |
| `email-notifications/content/order-shipped` | Order shipped email content |
| `email-notifications/content/orderline-fulfilled` | Orderline fulfilled (gift card) email content |
| `email-notifications/content/password-reset` | Password reset email content |
| `email-notifications/content/user-signup` | New user signup email content |
| `email-notifications/header` | Email header component |
| `email-notifications/layout` | Email layout wrapper (legacy) |
| `email-notifications/layout-new` | Email layout wrapper (updated) |
| `email-notifications/logo` | Email logo component |
| `email-notifications/social-links` | Email social links footer |

---

## `email-shopper/` — Current Email System (16 snippets)

| Snippet | Description |
|---|---|
| `email-shopper/banner` | Email banner image |
| `email-shopper/header` | Email header component |
| `email-shopper/layout` | Email layout wrapper |
| `email-shopper/logo` | Email logo component |
| `email-shopper/style/background-light` | Email background color token |
| `email-shopper/style/brand-color` | Email brand color token |
| `email-shopper/style/muted-color` | Email muted text color token |
| `email-shopper/style/text-color` | Email body text color token |
| `email-shopper/templates/abandoned-cart` | Abandoned cart email template |
| `email-shopper/templates/gift-voucher` | Gift voucher email template |
| `email-shopper/templates/order-confirmed` | Order confirmation email template |
| `email-shopper/templates/order-draft` | Draft order email template |
| `email-shopper/templates/order-shipped` | Order shipped email template |
| `email-shopper/templates/password-reset` | Password reset email template |
| `email-shopper/templates/user-signup` | User signup email template |
| `email-shopper/trust-block` | Email trust/credibility block |

---

## `footer/` — Footer (1 snippet beyond root `footer`)

| Snippet | Description |
|---|---|
| `footer/logo` | Footer logo (usually inverted/white for dark background) |

---

## `header/` — Header Components (7 snippets)

| Snippet | Description |
|---|---|
| `header/bottom-promotion` | Bottom promotion content (below nav) |
| `header/bottom-promotion-bar` | Bottom promotion bar HTML |
| `header/logo` | Site logo image and alt tag |
| `header/logo-height-desktop` | Logo height for desktop |
| `header/logo-height-mobile` | Logo height for mobile |
| `header/promotion` | Top promotion bar message text |
| `header/promotion-bar` | Top promotion bar HTML wrapper |

---

## `helpers/` — Utility Snippets (1 snippet)

| Snippet | Description |
|---|---|
| `helpers/is-kiosk-mode` | Renders `TRUE` if the current session is in kiosk mode |

---

## `icons/` — SVG Icon Library (182 snippets)

Full icon library. Each snippet contains a single SVG icon. Called via `{% snippet 'icons/name.svg' %}` with optional `class` parameter.

Icons include: account, arrow, bag, basket, calendar, camera, cart, check, chevron, close, edit, eye, social media (facebook, instagram, linkedin, pinterest, tiktok, twitter, youtube), file types, navigation, e-commerce (credit-card, gift, ship, store, tag, truck), and miscellaneous (award, badge-check, brush, building-shield, crop, download, flag-usa, frame, graduation, golf, etc.).

Not listed individually — 182 SVG icons. Browse the namespace or search by icon name.

---

## `integrations/` — Third-Party Integrations (27 snippets)

| Snippet | Description |
|---|---|
| `integrations/ahrefs/script` | Ahrefs analytics script |
| `integrations/chatbot/script` | AI chatbot script injection |
| `integrations/constant-contact/universal-code` | Constant Contact embed code |
| `integrations/custom-body-scripts` | Custom scripts injected at end of `<body>` |
| `integrations/custom-links` | Custom `<link>` tags in `<head>` |
| `integrations/google/ai-widget` | Google AI widget |
| `integrations/google/event/add-to-cart` | GA4 add-to-cart event |
| `integrations/google/event/begin-checkout` | GA4 begin-checkout event |
| `integrations/google/event/purchase` | GA4 purchase event |
| `integrations/google/event/view-item` | GA4 view-item event |
| `integrations/google/gtag` | Google Analytics tag (measurement ID) |
| `integrations/google/rating-widget` | Google rating widget (small) |
| `integrations/google/reviews-widget` | Google reviews widget (full) |
| `integrations/google/tag-manager` | GTM container ID |
| `integrations/google/utm-capture` | UTM parameter capture script |
| `integrations/klaviyo/added-to-cart` | Klaviyo add-to-cart event |
| `integrations/klaviyo/api-key` | Klaviyo public API key |
| `integrations/klaviyo/viewed-product` | Klaviyo viewed-product event |
| `integrations/la_cameras_dealercode` | LA Cameras dealer code |
| `integrations/sharemechat/script` | ShareMe.Chat widget script |
| `integrations/stamped.io/api-key` | Stamped.io public API key |
| `integrations/stamped.io/collection-review-stars` | Stamped review stars on collection cards |
| `integrations/stamped.io/collection-review-widget` | Stamped review widget on collection pages |
| `integrations/stamped.io/product-review-stars` | Stamped review stars on product pages |
| `integrations/stamped.io/product-review-widget` | Stamped review widget on product pages |
| `integrations/stamped.io/script` | Stamped.io base script |
| `integrations/stripe/apple-pay-verification` | Apple Pay domain verification file content |

---

## `kiosk/` — Kiosk Mode Components (3 snippets)

| Snippet | Description |
|---|---|
| `kiosk/home` | Kiosk home screen (product tiles) |
| `kiosk/idle-screen` | Kiosk idle/attractor screen |
| `kiosk/top-rail` | Kiosk top navigation rail |

---

## `modals/` — Modal Dialogs (18 snippets)

| Snippet | Description |
|---|---|
| `modals/cart-notification` | Cart add notification toast |
| `modals/editor-help-guide` | Design tool help guide modal |
| `modals/login` | Login modal |
| `modals/modal-add-gallery` | Add gallery modal (v1) |
| `modals/modal-add-gallery-v2` | Add gallery modal (v2) |
| `modals/modal-edit-gallery` | Edit gallery modal (v1) |
| `modals/modal-edit-gallery-v2` | Edit gallery modal (v2) |
| `modals/orderline` | Orderline detail modal |
| `modals/password-reset` | Password reset modal |
| `modals/product` | Product quickview modal (TBC) |
| `modals/promotions` | Promotions fly-out sidebar |
| `modals/proof` | Proof approval modal |
| `modals/search` | Site search modal |
| `modals/ship-order` | Ship order admin modal |
| `modals/shopping-cart` | Shopping cart fly-out sidebar |
| `modals/skip-proof` | Skip proof confirmation modal |
| `modals/upload` | File upload modal |
| `modals/warning` | Generic warning modal |

**Note:** Modals listed in the `index` layout (password-reset, shopping-cart, cart-notification, search, upload, login, warning, promotions, proof, skip-proof) are rendered globally on every page. Do not remove them.

---

## `navigation/` — Navigation Components (43 snippets)

### Core Navigation

| Snippet | Description |
|---|---|
| `navigation/back-to-top` | Back-to-top button |
| `navigation/beta` | Beta navigation variant |
| `navigation/blog-breadcrumbs` | Blog-specific breadcrumbs |
| `navigation/breadcrumbs` | Non-product page breadcrumbs |
| `navigation/cart-icon` | Cart icon with item count badge |
| `navigation/dropdown` | Standard single-column dropdown menu |
| `navigation/logo-left` | Logo-left navigation variant |
| `navigation/megamenu` | Megamenu wrapper/shell |
| `navigation/pagination` | Page pagination controls |
| `navigation/style1` | Nav style 1: logo left, center menu, icons right |
| `navigation/style2` | Nav style 2: menu left, center logo, icons right |
| `navigation/style3` | Nav style 3 (default): 2 rows, center logo, main nav bottom row |
| `navigation/style4` | Nav style 4: 2 rows, left logo, main nav bottom row |
| `navigation/style5` | Nav style 5 |
| `navigation/user-icon` | User account icon |

### Megamenu Panels (28 snippets)

Each panel is a full-width dropdown for a specific product category. Registered in the `navigation_links` capture block of the active nav style snippet.

`navigation/megamenu/all-products`, `archiving`, `art-services`, `bound-products`, `business`, `calendars`, `cards`, `cards-calendars`, `create`, `custom`, `digitize-media`, `digitizing`, `education`, `film`, `film-cameras`, `gifts`, `leaflets`, `photo-books`, `press-printing`, `print-services`, `prints`, `services`, `sports-events`, `stationery`, `studio`, `wall-art`, `wall-decor`, `wide-format-simple`

---

## `product/` — Product Page Components (65 snippets)

### Product Cards (8 snippets)

Standalone card components. **Not used by the main collection grid** (which renders cards inline). Available for custom sections, dynamic blocks, or future use.

| Snippet | Description |
|---|---|
| `product/cards/double-image` | Two-image card with hover swap |
| `product/cards/double-image-hover-actions` | Two-image card with hover action icons |
| `product/cards/double-image-hover-button` | Two-image card with hover button |
| `product/cards/dynamic-collection` | Dynamic collection card (for carousel/block sections) |
| `product/cards/dynamic-design-product` | Dynamic design product card (for carousel/block sections) |
| `product/cards/single-image` | Single-image card |
| `product/cards/single-image-hover-actions` | Single-image card with quickview icon on hover |
| `product/cards/single-image-hover-button` | Single-image card with hover button |

### Product Detail Pages (5 snippets)

| Snippet | Description |
|---|---|
| `product/product-details` | Master product detail page (standard PDP) |
| `product/product-details-filter` | Product page with sidebar filter controls |
| `product/product-details-prints` | Product page for photo prints UX |
| `product/details-filter-dual-mode` | Dual-mode PDP supporting both design & static products |
| `product/product-quickview` | Product quickview modal content |

### Product Sub-Components

| Snippet | Description |
|---|---|
| `product/approve-selection-checkbox` | Photo selection approval checkbox |
| `product/badges` | Product badge rendering |
| `product/breadcrumb-schema` | Breadcrumb structured data |
| `product/breadcrumbs` | Product page breadcrumbs |
| `product/canonical/collection-name` | Canonical collection name |
| `product/canonical/collection-path` | Canonical collection path |
| `product/canonical/image-url` | Canonical image URL |
| `product/canonical/review-id` | Canonical review ID |
| `product/canonical/review-image-url` | Canonical review image URL |
| `product/canonical/review-url` | Canonical review URL |
| `product/canonical/sku` | Canonical SKU |
| `product/canonical/url` | Canonical product URL |
| `product/custom-breadcrumb` | Custom breadcrumb override |
| `product/custom-prints-code` | Custom photo prints code injection |
| `product/custom-scripts` | Custom scripts on product page |
| `product/description` | Product description display |
| `product/design-now` | Design Now form (launches Design Tool) |
| `product/extra-prints-code` | Extra photo prints code injection |
| `product/features` | Product features tab content |
| `product/filter-controls` | Product filter dropdown/radio rendering |
| `product/gallery/standard` | Standard product gallery (preview images + live previews) |
| `product/gallery/testing` | Gallery testing variant |
| `product/header_instructions_photo_selection` | Photo selection page instructions |
| `product/inventory` | Product inventory display |
| `product/name` | Product name rendering |
| `product/photo-prints` | Photo prints UI |
| `product/photo-prints-design-product` | Photo prints for design products |
| `product/photo-prints-option` | Photo prints option selector |
| `product/photo-prints-path` | Photo prints URL path logic |
| `product/pricing` | Product pricing display |
| `product/pricing-prefix` | Pricing prefix text ("Starting at", "From") |
| `product/production-time` | Production time display |
| `product/production_time` | Production time display (alternate) |
| `product/px-option` | Option/variant renderer (dropdowns & radio buttons) |
| `product/px-option-cart` | Option/variant renderer for cart context |
| `product/px-option-selector` | Cart option selector (editable variants) |
| `product/px-option-selector-alt` | Cart option selector (specific options) |
| `product/px-options` | Master options renderer (loops all product options) |
| `product/px-options-shop` | Color swatch display on collection cards |
| `product/px-pdf-detection` | PDF upload detection logic |
| `product/schema` | Product structured data (JSON-LD) |
| `product/select-date` | Date selection for calendar products |
| `product/select-website-calendars` | Website calendar selection |
| `product/service-schema` | Service structured data |
| `product/shipping-available` | Shipping availability display |
| `product/subheader` | Product page subheader |

---

## `sections/` — Page Sections (73 snippets)

### Dynamic Sections (7 snippets)

Re-inject into DOM on AJAX updates. Use `style onload` pattern for JS.

| Snippet | Description |
|---|---|
| `sections/dynamic/blog` | Blog post listing (from Custom Types) |
| `sections/dynamic/carousel-products` | Product carousel (AJAX-safe) |
| `sections/dynamic/collection-block` | Collection product block (first 8 designs by default) |
| `sections/dynamic/free_shipping_progress_bar` | Free shipping progress bar |
| `sections/dynamic/product-carousel` | Product carousel variant |
| `sections/dynamic/product-description-tabs` | Tabbed product description section |
| `sections/dynamic/services` | Services listing (from Custom Types) |

### Static Sections (52 snippets)

Reusable homepage/content sections. Include in `pages/__home` or relevant pages.

`sections/static/2-column-cards`, `2-column-collection-spotlight` (+ `-2`), `2-photo-feature` (+ `-2`), `3-column-cards` (+ `-1`, `-2`, `-2nd-row`), `3-features`, `3-steps`, `4-block-best-sellers`, `4-company-values`, `6-across`, `6-categories`, `7-across`, `8-feature-spotlights`, `8-photo-feature`, `about`, `banner-alt`, `carousel`, `carousel-features`, `carousel-products`, `checklist-feature`, `coming-soon`, `countdown-promo-fullwidth`, `custom`, `discover-more-ways`, `features`, `hero-3-columns-fullwidth` (+ `-2`), `hero-4-features`, `hero-carousel`, `home-page/3-columns-fullwidth`, `home-page/digitize-services-cards`, `home-page/hero-banner-fullwidth`, `home-page/trusted-brands`, `image-slider-comparison`, `newsletter`, `our-blog`, `parallax-banner`, `parallax-text-block`, `project-gallery`, `projects`, `promo-banner-color`, `promo-countdown`, `reviews`, `reviews-1-across`, `services-contact-footer`, `services/cards/audio-transfers`, `services/cards/movie-film`, `services/cards/photo-scanning`, `services/cards/vhs-tapes`, `shop-by-brand`, `top-item-feature`, `top-picks`

### Product-Specific Sections (5 snippets)

| Snippet | Description |
|---|---|
| `sections/product/canvas/details` | Canvas prints detail section |
| `sections/product/canvas/wrap-description` | Canvas wrap description |
| `sections/product/canvas/wrap-options` | Canvas wrap options |
| `sections/product/photo-gifts/landing-page` | Photo gifts landing page section |
| `sections/product/prints/details` | Photo prints detail section |
| `sections/product/prints/image-slider-comparison` | Before/after image comparison for prints |

### Other Sections (2 snippets)

| Snippet | Description |
|---|---|
| `sections/holiday` | Holiday/seasonal promotional section |
| `sections/homepage` | Homepage section wrapper |
| `sections/custom/cattle-catalogs-fall-2024` | Client-specific section (cattle catalogs fall 2024) |
| `sections/custom/cattle-catalogs-fall-2025` | Client-specific section (cattle catalogs fall 2025) |

---

## `services/` — Services Page Components (23 snippets)

### Service Cards (16 snippets)

Individual service cards for the services listing page.

`services/cards/audio`, `commercial-archiving`, `design`, `image-recovery`, `movie-film`, `negative-scanning`, `passport-photos`, `photo-classes`, `photo-restorations`, `photo-scanning`, `pre-owned`, `rentals`, `repairs`, `slide-scanning`, `specialty-scanning`, `vhs-tapes`

### Service Page Components (7 snippets)

| Snippet | Description |
|---|---|
| `services/copyright` | Copyright/licensing notice |
| `services/film-services` | Film processing services section |
| `services/gather-box` | Gather box CTA section |
| `services/hero-banner` | Services hero banner (4 images with text overlay) |
| `services/list` | Full services listing |
| `services/portrait-studio` | Portrait studio section |
| `services/safe-local` | Local/safe messaging section |

---

## `social-media/` — Social Media Links (9 snippets)

Each contains the URL to the site's social profile. Used in footer and OG tags.

`social-media/etsy`, `facebook`, `instagram`, `linkedin`, `open-graph`, `pinterest`, `tiktok`, `twitter`, `youtube`

---

## `staging/` — Staging (1 snippet)

| Snippet | Description |
|---|---|
| `staging/homepage` | Staging homepage content |

---

## `style/` — Theme Tokens (53 snippets)

Fully documented in `50_SHOPPER_TEMPLATE_REFERENCE.md` Section 4. Each snippet contains a single CSS value. Referenced inline by `pages/custom.css`.

**Colors (28):** `badge-bkg-color`, `badge-text-color`, `color-announcement-bar`, `color-background`, `color-bg-v-light`, `color-bullet-point`, `color-button-bkg`, `color-button-bkg-hover`, `color-button-label`, `color-button-label-hover`, `color-button-outline`, `color-button-outline-hover`, `color-font`, `color-font-hover`, `color-font-secondary`, `color-footer`, `color-highlight`, `color-highlight-light`, `color-pill-btn-outline`, `color-primary`, `color-promotion-bar`, `color-promotion-text`, `color-secondary`, `color-secondary-50`, `link-color`, `link-hover-color`, `nav-hover-bkg-color`, `nav-hover-text-color`, `navbar-hover-text-color`

**Fonts (5):** `body-font-size`, `body-font-weight`, `custom-body-font`, `fonts`, `nav-font-size`, `nav-font-weight`

**Borders/Radius (8):** `btn-border-radius`, `btn-pill-bkg-color`, `btn-pill-bkg-color-hover`, `btn-pill-border-radius`, `btn-pill-text-color`, `btn-pill-text-color-hover`, `color-swatch-border-radius`, `color-swatch-width`, `content-border-radius`, `pill-img-outline-width`, `text-input-border-radius`, `var-img-border-radius`

**Layout (3):** `footer-color-background`, `footer-color-font`, `header-class`, `scroll-to-top-color-background`

**Custom CSS (2):** `custom.css` (per-site custom CSS — blank on parent), `editor.css` (custom CSS for design tool)

---

## `website/` — Site-Level Data (48 snippets)

### Contact Information (16 snippets)

`website/contact/address`, `city`, `city-state`, `faq_path`, `form`, `geo-location`, `geo-map`, `location`, `meta_description`, `meta_title`, `services-telephone`, `services-telephone-label`, `sla-note`, `state`, `support-email-address`, `support-hours`, `support-telephone`, `support-telephone-label`, `support-text`, `title`, `zip-code`

### Content Pages (7 snippets)

| Snippet | Description |
|---|---|
| `website/404` | 404 page content |
| `website/blog_page` | Blog listing page content |
| `website/blog_post` | Individual blog post template |
| `website/contact_page` | Contact page content |
| `website/contact_thank_you` | Contact form thank you page |
| `website/faq` | FAQ page supplementary content |
| `website/faq_page` | FAQ page template |

### Site Settings (15 snippets)

| Snippet | Description |
|---|---|
| `website/brand` | Brand name/identity |
| `website/current-promotions` | Currently active promotions |
| `website/custom-css` | Legacy custom CSS (use `style/custom.css` instead) |
| `website/description` | Site meta description |
| `website/film-delivery-address` | Film mail-in delivery address |
| `website/gallery-preview` | Gallery preview configuration |
| `website/google-review-link` | Google review page link |
| `website/gtag` | Google Analytics measurement ID |
| `website/homepage` | Custom homepage snippet content |
| `website/llms.txt` | LLMs.txt content for AI crawlers |
| `website/meta-pixel` | Meta (Facebook) Pixel ID |
| `website/px-subdomain` | Pixfizz subdomain (required for AI chatbot) |
| `website/robots.txt` | Custom robots.txt content |
| `website/schema` | Homepage structured data (JSON-LD) |
| `website/shipping-returns-policy` | Shipping & returns policy content |
| `website/sitemap` | HTML sitemap content |
| `website/sitewide-promotion` | Sitewide promotion message |
| `website/terms_page` | Terms & conditions page content |
| `website/title` | Website title (suffix on all page titles) |
| `website/trust-badges` | Trust badges below Add to Cart |

---

## Changelog

- 2026-05-27: Created from full CMS backup of `shopper24.pixfizz.com`. 926 snippets inventoried across 23 namespaces.


=================================================================
FILE: 60_SHOPIFY_INTEGRATION.md
=================================================================

# 60 — Shopify Integration

**Authority Scope:** Shopify + Pixfizz integration architecture, snippets, metafields, cart page, and order sync.

_Last updated: 2026-06-30_

---

## 1. Architecture Overview

In the Shopify deployment path:

- **Shopify** handles the storefront, cart, checkout, and payment.
- **Pixfizz** handles personalization (editor, options, photo prints), project storage, and production/order orchestration.
- Pixfizz runs on a **subdomain of the Shopify store's domain** (e.g. `create.myprintshop.com`). It must be a custom hostname — not the default `*.pixfizz.com` subdomain.
- A JavaScript API (`api.js`) is loaded from the Pixfizz host into every Shopify page. It provides the `Pixfizz.Shopify.*` methods used by all integration snippets.
- Customer identity is passed from Shopify to Pixfizz via an MD5 hash of the customer ID and email, signed with a shared secret.
- When a Shopify order is paid, a webhook fires to Pixfizz, confirming the order. Pixfizz changes the order status to "Confirmed".
- When an order is fulfilled in Shopify, a second webhook can fire to mark the Pixfizz order as "Shipped".

### Master Shopify CMS: staging vs production

Internally there are two master Shopify CMS sites: a **staging** master used to develop and test integration code, and a **production** master that live client sites inherit from. Only the actively synced pages/snippets (in practice `shopify/api.js` and `shopify/product`) are kept in step between them. Other pages and snippets on the staging master are likely outdated and should **not** be treated as canonical or copied from — always take known-good code from the production master. Workflow: build/test on staging, then copy the finished code across to production.

> Internal-infrastructure note: the specific master-site hostnames are internal; keep them out of the public KB and reference them from the private repo if needed.

---

## 2. Shopify Metafields

### Product Metafields (`pixfizz.*` namespace)

| Name | Namespace + Key | Type | Notes |
|---|---|---|---|
| Pixfizz Integration Type | `pixfizz.integration_type` | Single line text | One of: `editor`, `options-to-editor`, `options-to-cart`, `photo-prints`. Defaults to `editor` if blank. |
| Pixfizz Product SKU | `pixfizz.product_sku` | Single line text | `theme-code:product-code` from Pixfizz. Used for products without Shopify variations. |
| Pixfizz Extra Page Addon | `pixfizz.page_addon_product` | Product Variant | Links to the variant used to charge for extra pages (photo books). Price is per single page. |
| Pixfizz Option Addons | `pixfizz.option_addon_products` | Product | Products added to cart when a linked Pixfizz option is selected. |
| Pixfizz Option Type Code | `pixfizz.option_type_code` | Single line text | On addon products — links the addon to a Pixfizz option type code. |
| Pixfizz Adjust Cart QTY | `pixfizz.adjust_cart_qty` | True or False | Set to `false` to prevent customers changing cart quantity. Used for photo prints. |
| Pixfizz Photo Prints Collection Path | `pixfizz.photo_prints_collection_path` | Single line text | Collection path used when launching the photo prints flow. |
| SEO Hidden | `seo.hidden` | Integer | Set to `1` to hide addon products from search/Google. |
| Pixfizz Static Product Code | `pixfizz.static_product_code` | Single line text | Maps a non-personalized Shopify product to a Pixfizz static product by its `code`. Used for ingesting non-Pixfizz line items into Pixfizz orders via the webhook. |

### Variant Metafields (`pixfizz.*` namespace)

| Name | Namespace + Key | Type | Notes |
|---|---|---|---|
| Pixfizz Product SKU | `pixfizz.product_sku` | Single line text | Used for products WITH Shopify variations. Takes precedence over product-level SKU. |
| Pixfizz Extra Page Addon | `pixfizz.page_addon_product` | Product Variant | Variant-level override for the extra page addon. |
| Pixfizz Option Value Code | `pixfizz.option_value_code` | Single line text | Links this Shopify variant to a specific Pixfizz option value code. |
| Pixfizz Option Addons | `pixfizz.option_addon_products` | Product | Variant-level option addon override. |
| Pixfizz Static Product Code | `pixfizz.static_product_code` | Single line text | Variant-level override. Takes precedence over product-level value, same pattern as `product_sku`. |

### SKU Precedence Rule

For products WITH Shopify variations: the SKU **must** be set on the variant metafield. The product-level SKU is the fallback only.

---

## 3. Integration Types

Set via `pixfizz.integration_type` product metafield. 

| Value | Behaviour |
|---|---|
| `editor` | Launches the Pixfizz editor directly. Default. Used for photo books and products where full customization is expected. |
| `options-to-editor` | Customer selects options first (e.g. calendar start month), then enters the editor. |
| `options-to-cart` | Launches a live preview modal. Customer selects options and sees a preview before adding to cart. No full editor. Used for notebooks and simpler products. |
| `photo-prints` | Launches the photo prints flow. Quantity is set inside Pixfizz, not in the Shopify cart. |

---

## 4. Cart Item Properties (Pixfizz-managed)

Pixfizz sets these line item properties on Shopify cart items. They drive all cart-page behaviour.

| Property | Meaning |
|---|---|
| `_pixfizz_project_id` | ID of the associated Pixfizz project. Present on all Pixfizz personalized items. Private (underscore prefix = hidden from customer display). |
| `_pixfizz_addon` | Present on addon orderlines. Value equals the `_pixfizz_project_id` of the parent orderline. Used to link addons to their parent and to suppress their independent display. |
| `_pixfizz_unit_quantity` | The per-unit quantity of an addon. Used by the quantity handler to scale addon quantities correctly when the parent quantity changes. |

Properties with underscore prefix are not displayed to customers in Shopify's default rendering. The cart template already filters these out via `property_first_char != '_'`.

---

## 5. Core Shopify Snippets

Five snippets must be created in the Shopify theme. All are referenced by `{% render %}` calls in the product template and cart page.

---

### `pixfizz-setup.liquid`

**Purpose:** Loads `api.js` from the Pixfizz host and initialises the integration with customer identity.

**Rendered in:** `theme.liquid` `<head>`, via `{% render 'pixfizz-setup' %}`.

**Key configuration:**
- `pixfizz_host`: the Pixfizz custom subdomain (e.g. `create.myprintshop.com`)
- `shared_secret`: from Pixfizz superadmin API Settings

**How identity works:** If a Shopify customer is logged in, their ID and email are MD5-hashed twice (with the shared secret) and passed to `Pixfizz.Shopify.setup()`. If no customer, `null` is passed and Pixfizz uses an anonymous session.

---

### `pixfizz-launch-product-handler.liquid`

**Purpose:** Returns an inline JS handler (as escaped string) used in the product page "Personalize" button `onclick`.

**Usage in product template:**
```liquid
<button onclick="{% render 'pixfizz-launch-product-handler' %}">Personalize</button>
```

**What it does:**
- Builds a map of Shopify variant IDs → Pixfizz SKUs
- Builds an addon map (page addons + option addons) per variant
- Reads the selected variant from the product form
- Calls `Pixfizz.Shopify.launchProduct(sku, integration_type, product_id, variant_id, parameters)`

**Note:** This snippet may need adjustment for themes other than Dawn, specifically the `form = document.querySelector('form[action*="/cart/add"]')` selector.

---

### `pixfizz-orderline-preview-handler.liquid`

**Purpose:** Replaces the static product image in the cart with a live Pixfizz project preview, using a `<style onload>` pattern so it fires on DOM injection.

**Usage in cart template** (placed immediately after the `<img>` element inside `cart-item__image-container`):
```liquid
{% render 'pixfizz-orderline-preview-handler', item: item, width: 300 %}
```

**Parameters:**
- `item`: the cart line item object
- `width` or `height`: desired preview dimension (use one or both)

**How it works:** Uses `<style onload>` so the preview replacement fires every time the element is injected into the DOM (including after AJAX cart updates). Only acts if `item.properties._pixfizz_project_id` is present — safe to include unconditionally for all items.

**Resolution:** the underlying preview renders small by default, so omitting a size produces a blurry, upscaled thumbnail. This is the common cause of low-res cart and project previews. Pass a `width` of roughly **2x the rendered display size**, about **600** for a typical cart thumbnail. Apply the same fix to any Shopify saved-projects or my-projects page that calls the preview handler.

---

### `pixfizz-edit-orderline-handler.liquid`

**Purpose:** Returns an inline JS handler used on the cart item image link `onclick`, allowing the customer to re-enter the Pixfizz editor from the cart.

**Usage in cart template** (on the `<a>` element wrapping the cart item image):
```liquid
onclick="{% render 'pixfizz-edit-orderline-handler', item: item %}"
```

**Only rendered when** `item.properties._pixfizz_project_id` is present.

**What it does:**
- For `photo-prints` integration type: calls `Pixfizz.Shopify.launchPhotoPrints(parameters)`
- For all other types: calls `Pixfizz.Shopify.launchProduct(...)` — re-opens the same editor flow with the existing project

---

### `pixfizz-orderline-quantity-handler.liquid`

**Purpose:** Intercepts quantity changes and item deletions in the cart, ensuring addon orderlines are updated or removed in sync with their parent.

**Usage in cart template:**

On the quantity `<input>`:
```liquid
onchange="{% render 'pixfizz-orderline-quantity-handler', item: item, cart: cart %}"
onkeydown="if(event.keyCode===13)event.preventDefault()"
```

On the remove `<a>` button:
```liquid
onclick="{% render 'pixfizz-orderline-quantity-handler', item: item, cart: cart, delete_item: true %}"
```

**What it does:**
- When quantity changes: finds all addon items whose `_pixfizz_addon` matches this item's `_pixfizz_project_id`, and updates their quantities using `_pixfizz_unit_quantity * new_quantity`
- Uses `event.stopImmediatePropagation()` + `__pixfizz_addons_handled` flag to prevent double-firing
- When deleting: updates all quantities to 0 then reloads the page (native handler not reliable for deletions)
- Uses `fetch('/cart/update.js')` to apply all quantity changes in a single request

**Critical:** This handler must be present on every part of the UI that can modify or delete orderlines — not just the main cart page. In the drawer cart, on quick-buy, etc., the same handlers need to be applied.

---

## 6. Product Page Template

For Pixfizz products, the standard Shopify buy button block is replaced with a "Personalize" button.

**Recommended approach (Shopify 2.0 themes):**
1. Create a new product template called "Pixfizz Product" based on the default product template
2. Remove the buy button block
3. Add a Custom Liquid block with the personalize button

**Button code:**
```liquid
<button onclick="{% render 'pixfizz-launch-product-handler' %}" class="button button--full-width">
  Personalize
</button>
```

The button text can be changed freely. The `onclick` handler is what matters.

### Editor `domready` message (integration mode)

When the Pixfizz editor loads inside the Shopify integration, it posts a `domready` message once it is ready to receive instructions. Any code that posts messages **into** the editor iframe must wait for `domready` before sending — messages posted before the editor signals `domready` are lost. If a launch/config message appears to be ignored, confirm it is being sent only after `domready` is received. Added 2026-07-05.

---

## 7. Cart Page — Working Version (Dawn, inline_asset_content)

This is the production-tested cart page. It is based on Dawn theme using the `inline_asset_content` filter for SVG icons (a pattern introduced in later Dawn versions). It differs from the v13/v11 examples in the docs.

### Key differences from earlier documented Dawn versions

- SVG icons use `'icon-*.svg' | inline_asset_content` instead of `{% render 'icon-*' %}`
- Quantity lock uses `item.product.metafields.pixfizz.adjust_cart_qty.value == false` (metafield boolean check) instead of product type string comparison
- Edit handler does not have the `options-to-cart` exclusion present in v11 docs — simpler: any item with `_pixfizz_project_id` gets the handler
- Info button uses `small-hide medium-hide` classes

### Pixfizz modifications summary (what was added and where)

**1. Skip addon items at the top of the loop**
```liquid
{%- if item.properties._pixfizz_addon -%}
  {% continue %}
{%- endif %}
```
Addon items are rendered separately as sub-rows under their parent. They must not get a full independent row.

**2. Edit handler on the image link**
```liquid
style="z-index:1;"
{%- if item.properties._pixfizz_project_id -%}
  onclick="{% render 'pixfizz-edit-orderline-handler', item: item %}"
{%- endif -%}
```
`z-index:1` is required because the image container sits above the link in stacking order; without it the click doesn't reach the `<a>`.

**3. Project preview replacement**
```liquid
{% render 'pixfizz-orderline-preview-handler', item: item, width: 300 %}
```
Placed immediately after the `<img>` tag, inside `cart-item__image-container`.

**4. Quantity lock via metafield**
```liquid
{%- if item.product.metafields.pixfizz.adjust_cart_qty.value == false -%}
  <span>{{ item.quantity }}</span>
{%- else -%}
  <quantity-input ...>...</quantity-input>
{%- endif -%}
```
When `adjust_cart_qty` is `false`, renders a plain quantity display instead of the input controls. Used for photo prints.

**5. Quantity change handler on input**
```liquid
onchange="{% render 'pixfizz-orderline-quantity-handler', item: item, cart: cart %}"
onkeydown="if(event.keyCode===13)event.preventDefault()"
```

**6. Delete handler on remove button**
```liquid
onclick="{% render 'pixfizz-orderline-quantity-handler', item: item, cart: cart, delete_item: true %}"
```

**7. Addon sub-rows after each main item row**
After the closing `</tr>` of the main item, a second loop renders addon items as separate `<tr>` rows:
```liquid
{%- if item.properties._pixfizz_project_id -%}
  {%- for addon_item in cart.items -%}
    {%- if addon_item.properties._pixfizz_addon == item.properties._pixfizz_project_id -%}
      <tr class="cart-item">
        <td class="cart-item__media"></td>
        <td class="cart-item__details">
          {{ addon_item.product.title | escape }}
          {{ addon_item.final_price | money }}
          ...variants...
        </td>
        <td>{{ addon_item.final_line_price | money }}</td>
        <td>{{ addon_item.quantity }}</td>
        <td>{{ addon_item.final_line_price | money }}</td>
      </tr>
    {%- endif -%}
  {%- endfor -%}
{%- endif -%}
```
Addon rows: no media cell, no quantity controls, no remove button — display only.

---

## 7a. Cart Page — Focal / Maestrooo Theme Differences

Section 7 is written against Dawn. The Focal theme family (Maestrooo) structures
cart markup differently. The Pixfizz hooks are the same in intent but differ in
four ways:

1. Edit-design handler. On Focal, re-entering an existing project opens the
   project-edit modal via `openModal` with a `?project_id=` query parameter.
   Calling `launchProduct` on this theme starts a NEW project instead of editing
   the existing one. (Observed on a live Focal site. Confirm with Matjaz whether
   this should also replace `launchProduct` in the standard edit-orderline
   snippet, or stays Focal-only.)
2. Preview image CSS. Focal's `.aspect-ratio img` rule uses absolute positioning,
   which hides the swapped-in preview. Scope an override under the line item image
   wrapper (`.line-item__image-wrapper`) to restore it.
3. Preview rendering. In the Shopify cart the preview is always a plain `<img>`
   src swap. It is never the `px-project-preview` web component (that component is
   CMS / Shopper only).
4. Variant source. The Focal cart has no add-to-cart form, so the variant must be
   read from `item.variant.id`, not from a form lookup.

The third-party SpurIt Product Options 2 include lines, where present, are app
glue and must not be removed.

---

## 8. Addon Products — Setup and Behaviour

### Extra Pages (photo books)

- Create a separate Shopify product representing "extra page" — price is per single page
- Link it to the main book product via `pixfizz.page_addon_product` metafield (Product Variant type)
- Pixfizz adds this product to the cart automatically when the user adds pages in the editor
- Pages are always added in pairs (or multiples) as set by the template; Shopify price is per page
- Hide addon product from search using `seo.hidden = 1`
- Recommended: disable pricing display in the Pixfizz editor for these products to avoid confusion

### Option Addons (variant-linked pricing)

- Create an addon product in Shopify. Set `pixfizz.option_type_code` on it to the code of the Pixfizz option type
- Create variants on the addon product. Set `pixfizz.option_value_code` on each variant to the corresponding Pixfizz option value code
- Link the addon product to the base product via `pixfizz.option_addon_products`
- Pixfizz adds the correct addon variant to the cart when the customer selects the linked option

---

## 9. Order Sync — Webhooks

### Payment webhook (required)
- **Event:** Order payment
- **Format:** JSON
- **URL:** `https://<pixfizz-host>/webhooks/shopify/orders`
- **Effect:** Pixfizz marks the order as "Confirmed"

### Fulfillment webhook (optional)
- **Event:** Fulfillment Creation
- **Format:** JSON
- **URL:** Same endpoint pattern
- **Effect:** Pixfizz marks the order as "Shipped"

### Webhook signing secret
After creating the webhook in Shopify (Settings → Notifications), copy the signing secret shown under the callback URL and paste it into the Pixfizz superadmin panel under Website → API Settings → Shopify Signing Secret.

**Updating Pixfizz order status from Shopify Flow.** When using Shopify Flow (or any external automation) to push an order status change into Pixfizz, for example moving an order from "ready for pickup" to a Pixfizz shipped status, use an HTTP **PUT** request, not POST.

**Project ownership on order sync.** When a Shopify order is synced and the created Pixfizz project has no owner, Pixfizz assigns the project to the user account that placed the order. This means Shopify-originated projects are owned by the ordering customer by default rather than being left unowned. Added 2026-07-07.

**All line item properties captured into orderline options.** Shopify order line item properties are now captured wholesale into the corresponding Pixfizz orderline's `options`, not only the Pixfizz-specific ones. Custom line properties set by the theme or by third-party apps flow through to the Pixfizz order and are available downstream (admin, fulfillment templates). For static-product routing to work, the line item property that carries the static-product code must **exactly match** the name the webhook expects (`_pixfizz_static_product`, see §9a) — a mismatched property name is the usual reason a static line item fails to route. Confirm the exact expected property string with the platform team if routing fails. Added 2026-07-20.

---

## 9a. Static Product Ingestion (Non-Personalized Shopify Products)

Shopify stores that sell a mix of personalized and non-personalized products (frames, film, accessories, etc.) can route **all** line items into the Pixfizz order — not just those with a `_pixfizz_project_id`.

### How it works

1. **Define the metafield** — create a `pixfizz.static_product_code` text metafield on Products in Shopify (Settings → Custom Data → Products → Metafields). Optionally define the same metafield on Product Variants if different Shopify variants should map to different Pixfizz products.

2. **Create matching static products in Pixfizz** — the product `code` in Pixfizz must match the value set in the Shopify metafield.

3. **Inject the line item property** — modify the Shopify theme's product form so that products with the metafield get a hidden `_pixfizz_static_product` property added to the cart line item. The variant-level metafield takes precedence over product-level (same pattern as `product_sku`).

4. **Webhook processing** — when the order payment webhook fires, Pixfizz reads `_pixfizz_static_product` on each line item and maps it to the corresponding Pixfizz static product, creating an orderline for it on the Pixfizz order alongside any personalized orderlines.

### Theme implementation (Dawn v14 example)

In `snippets/buy-buttons.liquid`, add inside the product form:

```liquid
{%- assign pixfizz_static_product = product.selected_or_first_available_variant.metafields.pixfizz.static_product_code %}
{%- if pixfizz_static_product == blank %}
  {%- assign pixfizz_static_product = product.metafields.pixfizz.static_product_code %}
{%- endif %}
{%- if pixfizz_static_product != blank %}
  <input type="hidden" name="properties[_pixfizz_static_product]" value="{{ pixfizz_static_product | escape }}" />
{%- endif %}
```

### Key points

- The variant-level metafield is checked first; falls back to product-level. This is consistent with how `pixfizz.product_sku` works.
- The property is underscore-prefixed (`_pixfizz_static_product`), so it is hidden from the customer in the Shopify cart and checkout.
- The exact placement of the hidden input depends on the Shopify theme — `buy-buttons.liquid` is correct for Dawn but other themes may differ.
- Static products in Pixfizz must exist and have a `code` matching the metafield value, otherwise the line item will not be mapped.

#### Known limitation: options/bundle apps and static product ingestion

If a Shopify store runs a product-options or bundle app on a product that also carries `_pixfizz_static_product`, the app can expand a single cart line into a **bundle parent line plus component lines** at checkout, all stamped with the same `_pixfizz_static_product` property. The Pixfizz order webhook does **not** create an order from this grouped/bundled shape, so the static orderline silently fails to ingest.

- A plain single-variant product (no options/bundle app involvement) ingests correctly.
- If static products are not landing in the Pixfizz admin despite the property being present on the line item, check whether an options/bundle app is restructuring the line at checkout.

**JS gotcha in the injection snippet:** if the snippet builds its cart-property payload by looping qualifying lines in Liquid and declares `const line` once per line in the same script scope, two or more static-product lines in one cart produce a duplicate-declaration syntax error. Inline `line.key` / `line.quantity` into the pushed object (or use a block-scoped variable per iteration) so only one declaration exists.

---

## 10. Product Linking Rules

### Products WITHOUT Shopify variations
Required metafields on the product:
- `pixfizz.integration_type`
- `pixfizz.product_sku` (on the product, not variant)

### Products WITH Shopify variations
Required metafields:
- `pixfizz.integration_type` (on the product)
- `pixfizz.product_sku` (on **each variant**, not the product)

### General rules
- Shopify product must have "Track QTY" disabled
- For photo prints: set `pixfizz.adjust_cart_qty` to `false` on the product
- SKU format: `theme-code:product-code` (no spaces, no typos)
- **Edit product/variant ID mapping CSVs in a plain-text editor, not Excel.** When preparing a CSV that maps Shopify product/variant IDs to Pixfizz codes, do not open or save it in Excel — Excel silently reformats long numeric IDs (scientific notation, dropped leading characters) and corrupts the mapping. Use a plain-text/code editor (e.g. VS Code) so the IDs stay intact. Source: fireflies-call (2026-07-10).

---

## 10a. Third-Party Variant / Option Apps (Globo)

Some Shopify stores use a third-party options app such as **Globo Product Options** to add variant/option choices beyond Shopify's native limits. Globo has its own variant-count limit that can block complex setups (many options × many values). When a setup exceeds that limit, fall back to either **Shopify core variants** (within Shopify's own limits) or move option selection to the **Pixfizz Shopper frontend** instead of the Shopify product page. Source: Fireflies (2026-06-29, 2026-07-03).

---

## 11. Troubleshooting Guide

### Project preview not showing in cart
- Check that `pixfizz-orderline-preview-handler` is placed immediately after the `<img>` element inside `cart-item__image-container`
- Check that `item.properties._pixfizz_project_id` is actually present (inspect cart item properties)
- The `<style onload>` pattern fires on DOM injection — if the preview worked once but stopped after a cart AJAX update, check that the snippet is not inside a `<script>` tag

### Edit link not firing on image click
- The `<a>` element needs `style="z-index:1;"` — without it, the image container sits above the link in the stacking order and swallows the click
- Confirm `_pixfizz_project_id` is present on the item

### Addon quantities not updating when parent quantity changes
- `pixfizz-orderline-quantity-handler` must be on the quantity `<input>` onchange
- If the theme has a drawer cart or other quantity controls elsewhere, the handler must be added there too — not just on the main cart page
- Check `_pixfizz_unit_quantity` is set on the addon item (Pixfizz sets this when adding to cart)

### Quantity selector showing for photo prints when it should be hidden
- `pixfizz.adjust_cart_qty` metafield must be set to `false` (boolean False, not the string "false")
- The check in the template is `.value == false` — if the metafield is missing or is the string "false", it will not match

### Addon items appearing as independent rows
- The `{%- if item.properties._pixfizz_addon -%} {% continue %} {%- endif %}` block must be at the very top of the `for item in cart.items` loop, before the `<tr>` is opened
- If this block is missing or misplaced, addon items render as full rows

### Orders not confirming in Pixfizz after payment
- Check the Shopify webhook is set to "Order payment" event (not "Order creation")
- Verify the Shopify signing secret matches what's in Pixfizz superadmin
- Check that the webhook URL matches the Pixfizz host exactly

### "Personalize" button text overwritten with "Add to cart" after variant change
- Cause: Dawn's Section Rendering API re-renders the product section on variant change and overwrites the `innerHTML` of any element with class `product-form__add-button` — including the Pixfizz personalize button — replacing it with the "Add to cart" translation string.
- **Recommended fix**: change the button's class away from `product-form__add-button` to a custom class (e.g. `pixfizz-launch-btn`) so Dawn's variant update logic does not target it. Add a small script to hide the native submit button if needed.
- **Simpler fix**: wrap the button text in a structured inner `<span>` rather than a plain text node. Dawn targets plain text nodes specifically; a structured span may survive the replacement depending on the Dawn version.
- This affects any Dawn theme using the Section Rendering API — confirm which fix is appropriate by fetching a section render response and inspecting the returned button HTML.

### `Pixfizz is not defined` JS error
- `pixfizz-setup` snippet must be rendered in `theme.liquid` before `</head>`
- The Pixfizz host in `pixfizz-setup.liquid` must use the custom subdomain, not `*.pixfizz.com`

### Customer receives duplicate order emails
When a store runs Shopify alongside Pixfizz, both systems can send order notifications, so the customer receives two of each. Disable or blank out the redundant Pixfizz email templates for the lifecycle events Shopify already covers, so only one system notifies the customer.

---

## 12. Retrieval Pointer

For questions about:
- Cart item properties and addon row rendering → sections 5, 7 of this file
- Metafield setup → section 2
- Integration types → section 3
- Product linking → section 10
- Webhook/order sync → section 9
- Troubleshooting → section 11
- Pixfizz platform objects (order, orderline) → `50_LIQUID_REFERENCE.md`
- Order external source/reference fields → `50_LIQUID_REFERENCE.md` (Order object)
- Local pickup address mapping → section 13
- Variable pages (saved projects, galleries, Pixfizz.Shopify methods) → section 15

---

## 13. Pickup Orders — Webhook Address Handling

When a Shopify order is placed using a local pickup option, Shopify sends no shipping address in the order webhook. Pixfizz handles this by attempting to match the **shipping method name** from the webhook to a **public address** in Pixfizz.

### Matching logic

- Pixfizz reads the shipping method name from the Shopify webhook
- It searches for a public address in Pixfizz where the **Company** field matches that name exactly (case-sensitive)
- If a match is found, that address is assigned to the order automatically
- If no match is found, the order arrives in Pixfizz with no address and must be assigned manually

### Setup

1. In Shopify admin → **Settings → Locations**, note the exact **Name** of the pickup location
2. In Pixfizz admin → **Settings → Addresses**, create or edit a public address and set the **Company** field to exactly that name
3. Test with a pickup order and confirm the address is assigned in Pixfizz

### Notes

- The match is case-sensitive — `MyStore` and `mystore` are treated as different values
- If you have multiple pickup locations, create a separate public address in Pixfizz for each one
- Cart-page snippets are unaffected — this is purely webhook ingestion behaviour

---

## 14. Multi-Site Product Inheritance

**Pattern, 2026-03-05.**

This pattern applies when a customer operates multiple brand storefronts that
share a **single underlying product database** across all brands, while each
brand has its own Shopify storefront + Pixfizz personalization configuration.

### How it works

- Products, templates, and fulfillment logic are defined once at the parent
  organization level in Pixfizz.
- Each brand's Shopify store references the same Pixfizz products via the
  usual `pixfizz.product_sku` metafield, but the Shopify-level surfaces
  (titles, prices, collection structure, theme) are independently managed
  per brand.
- Pixfizz Super Admin's **website inheritance** (see `18_ADMIN_NAVIGATION.md`)
  is how the shared product/design/tax/email configurations are exposed to
  each brand site.

### When to reach for this pattern

- A customer operates multiple consumer-facing brands that sell the same
  underlying photo products with different branding and pricing.
- Each brand needs its own storefront identity and checkout, but the
  production pipeline is centralised.
- The alternative — copying products into each site separately — creates
  unmanageable drift over time.

### Gotchas

- Price differences between brands need to be handled at the Shopify layer
  (different prices per brand on the same underlying Pixfizz product), not
  in the Pixfizz pricing formula.
- Brand-specific production overrides (e.g. different fulfillment destinations
  per brand) need per-site fulfillment configuration, set at the Pixfizz
  website level rather than on the product itself.

---

## 15. Variable Pages — Saved Projects and Galleries

Shopify stores can host customer-facing pages that display Pixfizz data (saved projects, galleries) by calling the Pixfizz REST API directly from JavaScript. This works because `api.js` establishes a Pixfizz session for the customer on every page load, and `fetch()` calls with `credentials: 'include'` carry that session to the Pixfizz subdomain.

No additional authentication is required beyond the standard `pixfizz-setup.liquid` integration.

### Confirmed `Pixfizz.Shopify.*` method list (as of 2026-04-12)

```
pixfizz_origin         — the Pixfizz host URL (use this instead of hardcoding the subdomain)
locale
cart_target
skip_draft_redirect
product_data_loader    — loads and caches Shopify product data from the pixfizz-product-api page
_user                  — object: { uid, email } for the logged-in customer. null if not logged in.
_hash
_modal
_session_established
setup
establishSession
openModal
closeModal
launchProduct
launchPhotoPrints
launchProject
launchSavedProject
loadProjectPreview
replaceImageWithProjectPreview
getSavedProjects
```

### Key values for variable pages

- `Pixfizz.Shopify.pixfizz_origin` — use this as the base URL for all API calls. Never hardcode the subdomain.
- `Pixfizz.Shopify._user.uid` — the **Shopify customer ID** for the logged-in customer (not the Pixfizz user ID). Sufficient for read-only API calls (GET endpoints).

### Pixfizz User ID vs Shopify Customer ID

**Critical distinction for user-scoped POST endpoints** (e.g. creating galleries, projects):

- `Pixfizz.Shopify._user.uid` returns the **Shopify customer ID** — this works for Shopify-side operations and for constructing GET API calls.
- The **Pixfizz user ID** (needed for POST endpoints like `POST /v1/users/{uid}/galleries.json`) must be extracted from the `_mine.json` redirect response.

To get the Pixfizz user ID:

```javascript
// Fetch _mine.json — the response URL contains the Pixfizz user ID
var resp = await fetch(Pixfizz.Shopify.pixfizz_origin + '/v1/galleries/_mine.json', {
    credentials: 'include'
});
// Parse the user ID from the response URL (redirected to /v1/users/{pixfizzUserId}/galleries.json)
var pixfizzUserId = resp.url.match(/users\/(\d+)/)[1];
```

**If you use the Shopify customer ID in a Pixfizz POST endpoint, the call will fail or create resources under the wrong user.** This applies to any variable page that creates user-scoped resources (galleries, projects), not just reads them.

**UX: drop the customer straight into the new gallery.** After a successful gallery-create call, immediately call the existing `openGallery(gallery)` handler inside the create success callback — place it *after* `addGalleryCard(gallery)` so the new gallery still appears on the index when the customer navigates back. This takes the customer into the upload flow instead of leaving them on the list view. Source: claude-chat, Fireflies (2026-07-01, 2026-07-03).

### Gallery API endpoints (authenticated via session cookie)

```
GET  {origin}/v1/galleries/_mine.json                  — list customer's galleries
POST {origin}/v1/users/{uid}/galleries.json            — create a gallery (FormData: gallery[name])
GET  {origin}/v1/galleries/{id}/images.json            — list images in a gallery
POST {origin}/v1/galleries/{id}/images.json            — upload an image (FormData: data = file)
```

All calls require `credentials: 'include'` (fetch) or `xhr.withCredentials = true` (XHR).

### Image thumbnail URLs

Gallery image `preview` URLs contain a path segment `thumbnail/250` where `250` is the rendered size. This is a path parameter, not a query parameter — `?height=N` has no effect. To get different sizes, replace the number in the path:

```javascript
url.replace(/thumbnail\/\d+/, 'thumbnail/800')   // grid thumbnails
url.replace(/thumbnail\/\d+/, 'thumbnail/1600')  // lightbox
```

### Page template pattern

Variable pages are created as Shopify page templates (`page.{name}.liquid`) — not snippets, and not sections. The `{% schema %}` tag is not valid in page templates. Padding must be handled with plain CSS, not `section.settings`.

This describes **Dawn-era themes**, where the page template is a single `.liquid` file that holds all markup, CSS, and JS. It does **not** hold on newer block-based themes (Horizon and similar) — see "Theme architecture" immediately below.

### Theme architecture: Liquid template vs JSON template (Dawn vs Horizon)

On Horizon (Shopify's newer theme architecture, and others like it), page templates are **JSON** files (`page.{name}.json`) that reference sections and blocks, not standalone `.liquid` files. If both `page.{name}.liquid` and `page.{name}.json` exist with the same name, the **JSON template wins** and the `.liquid` file silently renders nothing. Edits to the `.liquid` file then have no visible effect on the page.

**Symptom:** the page shows only the theme's default page section (page title as a heading plus an empty rich-text block) and nothing else. The custom markup (e.g. the gallery wrapper) is absent from the rendered page source.

**How to confirm:** view the live page source.
- `<main>` carries `data-template="page.{name}"`, so the template IS assigned and the session IS established (`Pixfizz.Shopify.setup(...)` runs in the theme `<head>` on every page). Those are not the problem.
- The `<main>` body contains only the default heading + empty rich-text block. The custom markup is nowhere in the source. That confirms the `.liquid` file is dead code being overridden by the JSON template.

**Reliable delivery on Horizon:** open the theme Customizer on the page's template, add a **Custom Liquid block** (or a proper section), and paste the entire variable-page file into it — the `{{ ... }}` line, the `<style>` block, the HTML, and the `<script>`. Because `pixfizz-setup` already runs in the theme `<head>`, the pasted script initialises on direct load. Delete the default heading block so the page does not show two titles.

**Notes:**
- Remove any `{{ 'customer.css' | asset_url | stylesheet_tag }}` line unless the theme actually ships that asset. Horizon does not, and it 404s in the console. Keep all CSS inline in the block.
- Horizon uses view transitions and section hydration for in-site navigation. Inline scripts run on direct load and refresh, but may not re-execute when the page is reached via an internal link. If that surfaces, it is the standard AJAX re-injection issue and needs a JS re-init pattern (escalate to the platform devs).

### Implementation reference

- Saved projects page: `pixfizz-saved-projects.liquid` snippet, also requires the `page.pixfizz-product-api.liquid` template and a corresponding Shopify page
- Gallery page: `page.pixfizz-galleries.liquid` — standalone template, no additional dependencies beyond `pixfizz-setup`

---

## 16. Non-Pixfizz Product Passthrough (Static Products via Webhook)

Shopify stores selling a mix of personalized and non-personalized products (frames, film, accessories) can ingest non-Pixfizz line items into Pixfizz orders via the existing order webhook.

### How it works

1. Set the `pixfizz.static_product_code` metafield on the Shopify product (or variant) — this maps it to a Pixfizz static product by its `code`.
2. Add a hidden input to the product form in the Shopify theme so the code is passed as a line item property:

```liquid
{%- assign pixfizz_static_product = product.metafields.pixfizz.static_product_code -%}
{%- if pixfizz_static_product == blank -%}
  {%- assign pixfizz_static_product = product.metafields.pixfizz.static_product_code -%}
{%- endif -%}
{%- if pixfizz_static_product != blank -%}
  <input type="hidden" name="properties[_pixfizz_static_product]" value="{{ pixfizz_static_product | escape }}" />
{%- endif -%}
```

3. When the order webhook fires, Pixfizz reads the `_pixfizz_static_product` property on each line item. If it finds a match to a static product by code, it creates an orderline in Pixfizz for that item.

### Key points

- The `_pixfizz_static_product` property is underscore-prefixed, so it is hidden from the customer in Shopify cart and checkout.
- Static product orderlines in Pixfizz have no project, template, or production files — they are informational records for order tracking and fulfillment communication.
- If the static product code doesn't match any product in Pixfizz, the line item is silently skipped.
- Variant-level `pixfizz.static_product_code` takes precedence over product-level, same as `product_sku`.
- This is the mechanism for getting a single unified order view in Pixfizz admin for mixed orders.

---

## 17. Classic vs New Customer Accounts

Shopify has two customer account systems. The integration approach differs:

| Account System | Auth Method | `/account` Customizable? | Integration Approach |
|---|---|---|---|
| **Classic Customer Accounts** | Email + password | Yes (theme Liquid) | Embed Pixfizz snippets directly in `main-account.liquid` or create sections |
| **New Customer Accounts** | Passwordless (one-time code) | No (hosted by Shopify) | Use standalone Shopify pages (`/pages/my-projects`, `/pages/my-galleries`) |

### Implications for variable pages

- **Saved Projects / My Galleries / My Projects** built as standalone Shopify page templates (`page.*.liquid`) work with **both** account systems because they don't depend on the `/account` template.
- The legacy table-based saved projects approach (embedded in `main-account.liquid`) only works with **Classic Customer Accounts**.
- When documenting for customers, always specify which account system the setup targets.

---

## Changelog
- 2026-03-13: Initial version. Compiled from public docs + working cart page (Dawn, inline_asset_content variant).
- 2026-03-21: Added Dawn button innerHTML overwrite troubleshooting entry (§11).
- 2026-04-10: Added pickup order webhook address handling (§13) and multi-site product inheritance pattern (§14).
- 2026-04-12: Updated §13 with specific pickup matching logic (Shopify location Name → Pixfizz address Company field). Added §15 Variable Pages — confirmed Pixfizz.Shopify method list, gallery API endpoints, thumbnail URL path pattern, page template conventions.
- 2026-05-19: CORRECTED §15 — `_user.uid` is Shopify customer ID, not Pixfizz user ID; added Pixfizz user ID extraction pattern via `_mine.json` redirect. Added §16 Non-Pixfizz Product Passthrough (static products via webhook). Added §17 Classic vs New Customer Accounts terminology and integration implications. Source: Claude chats (gallery create fix, static product linking, Shopify projects page).
- 2026-06-01: Added Section 7a, Focal/Maestrooo cart differences. Source: claude-chat.

- 2026-06-15: Added preview-resolution guidance to the orderline preview handler (pass ~600px; small default causes low-res thumbnails). Added §9 note: update Pixfizz order status from Shopify Flow with PUT, not POST. Added §11 troubleshooting entry for duplicate order emails when running Shopify alongside Pixfizz. Source: slack-kb-sync (LisPhoto calls).
- 2026-06-29: Added §15 "Theme architecture" subsection — on Horizon and other block-based themes a same-named `page.{name}.json` template takes precedence over `page.{name}.liquid`, which then renders nothing; deliver variable pages via a Custom Liquid block on the template instead. Qualified the existing "Page template pattern" note as Dawn-era. Source: claude-chat (Shopify photo-lab galleries debug).
- 2026-06-30: Documented static-product ingestion limitation with options/bundle apps (bundle expansion stamps duplicate _pixfizz_static_product; webhook skips grouped shape) and the const-redeclaration JS gotcha in the injection snippet. Source: claude-chat (Shopify static product debugging).
- 2026-07-04: Added §15 gallery auto-open (`openGallery` after `addGalleryCard` in create callback) and §10a Globo variant-limit fallback (core variants or Shopper frontend). Source: claude-chat, Fireflies.
- 2026-07-11: Added §1 master-CMS staging-vs-production workflow note (only api.js/product kept in sync; do not copy from staging); §9 project-ownership auto-assignment on order sync (unowned project → ordering user); §6 editor `domready` message (wait before posting into the editor iframe); §10 plain-text-editor gotcha for product/variant ID mapping CSVs (Excel corrupts IDs). Source: slack-message (#development, commits 2026-07-05/07), fireflies-call (2026-07-10).
- 2026-07-20: Noted all Shopify line item properties are now captured into Pixfizz orderline options, and that static-product routing depends on the exact expected property name. Source: #development (commit 2026-07-13), Weekly Tech call.


=================================================================
FILE: 61_PIXFIZZ_API.md
=================================================================

# 61 — Pixfizz API Reference

**Authority Scope:** Pixfizz REST API (v1), JS API, user handoff, project/fulfillment endpoints, dynamic previews, and custom eCommerce integration.

_Last updated: 2026-07-01. Compiled from Pixfizz Notion wiki (API section)._

---

## 1. Overview

The Pixfizz API is a read/write/delete REST API over HTTPS. All responses are JSON.

- **Base URL:** `https://<subdomain>.pixfizz.com/v1/`
- **Version:** v1 (breaking changes will be released as v2 — v1 will not have breaking changes)
- **Format:** JSON only. Send `Content-Type: application/json` for POST/PUT requests with JSON bodies.
- **Timestamps:** ISO 8601 (`YYYY-MM-DDTHH:MM:SSZ`)
- **Pagination:** Append `?page=N` to index endpoints. `page=3` fetches the third page of results.
- **Rate limits:** No hard limits. Best practice: add a 30-second delay after every 100 requests. Notify support before large bulk uploads.
- **ETag headers:** Present on every response. Use to detect unchanged data.
- **User-Agent header:** Required on every request. Automatically included by the JS API and browsers. Custom scripts must supply a descriptive string.

### API Objects

| Object | Description |
|---|---|
| Book (Project) | A user's personalized project created in the editor |
| Book File | A production PDF generated from a project |
| Gallery | A collection of images belonging to a user |
| Order | A customer payment and fulfillment record |
| Product | A product definition with pricing and variants |
| Promocode | A discount code for use during checkout |
| User | An authenticated user in the system |
| Calendar | A template for user-generated calendar products |
| Font | A font palette available in the editor |
| Color | A color palette available in the editor |

---

## 2. Authentication

### HTTP Basic Auth (server-to-server, recommended)
Use an admin user's credentials. Required for all admin endpoints.

```
curl --user admin@example.com:password https://<subdomain>.pixfizz.com/v1/admin/orders.json
```

### Cookie-based (browser/testing only)
```
curl -X POST \
     -d "email=user@example.com" \
     -d "password=mypassword" \
     https://<subdomain>.pixfizz.com/v1/session
```
Not recommended for production.

### OAuth 2.0 (client-side / mobile apps)
- Create an OAuth application in Pixfizz admin under **Site → OAuth**.
- Authorization URL: `https://<subdomain>.pixfizz.com/v1/oauth/authorize?client_id=YOUR_APP_ID&response_type=token&redirect_to=http://localhost/callback&state=`
- For mobile apps, `redirect_to` must be `http://localhost/callback`.
- Verify access token: `POST https://<subdomain>.pixfizz.com/v1/oauth/debug` with `client_id`, `client_secret`, and `token`.
- OAuth tokens do not expire unless revoked by the user or admin.
- Use `Authorization: OAuth YOUR_ACCESS_TOKEN` header for requests.
- Get current user ID after auth: `GET /v1/users/me.json`

---

## 3. Users

### Get current user
```
GET /v1/users/me.json
```
Response includes `id`, `email`, `first_name`, `last_name`, and `links` (self, galleries, books, orders, addresses, groups, promocodes).

### List all users (admin)
```
GET /v1/admin/users.json
```

### User handoff from external system
See section 7 (Custom eCommerce Integration).

---

## 4. Projects (Books)

**Note:** The Pixfizz API calls projects "books" — this is legacy terminology. In the UI and Liquid they are called "projects". The REST API uses `books` throughout.

### List user's projects
```
GET /v1/users/<user-id>/books.json
GET /v1/books/_mine.json           # convenience redirect for logged-in user
```

### Read a project
```
GET /v1/books/<id>
```

### Create a project
```
POST /v1/books.json
```
Required parameters (use one pair):
- `theme_code` + `product_code`  — OR —  `theme_id` + `product_id`

Optional parameters:
- `book[name]` — project name
- `variants[<variant-code>]` — product variant values (multi-choice: option code; text: value string)
- `template_options[<option-code>]` — same pattern as variants but for template options
- `book[custom][<field-name>]` — custom fields configured in Pixfizz admin
- `book[start_year]`, `book[start_month]`, `book[start_day]` — calendar products only

Example response fields: `id`, `name`, `saved`, `ordered`, `preview`, `theme_id`, `theme_code`, `product_id`, `product_code`, `template_id`, `template_code`, `options`, `template_options`, `links`.

### Update a project
```
PUT /v1/books/<id>
```
- Rename: `book[name]=new+name`
- Mark saved: `book[saved]=1`
- Mark ordered: `book[ordered]=1`

Do not directly edit the XML structure via API — this will cause editor issues.

### Copy a project
```
POST /v1/books/<id>/copy
```
Returns a 302 redirect to the new project resource. The copy is fully independent.

**The copy is created unsaved.** `POST /v1/books/<id>/copy` creates the new project with `book[saved]=0`, so it will not appear in `getSavedProjects` until you explicitly set `book[saved]=1` on the new project ID (extract that ID from the copy response's redirect URL).

**Cross-origin `PUT` silently fails; use `POST` with `_method=put`.** A browser cross-origin `PUT` to `/v1/` triggers a CORS preflight that the endpoint does not answer, so the request never completes and no error is surfaced. This is why a `POST` copy works from a Shopify page while a `PUT` unsave/rename appears to do nothing. Send `POST` with a `_method=put` field so Rails routes it as a `PUT` without triggering the preflight. Source: claude-chat (Shopify projects page).

### Preview a project (JPEG, free)
```
GET /v1/books/<id>/preview
GET /v1/books/<id>/preview?share=<share-code>   # bypass auth using share code
```
Returns a JPEG, max 300px wide. Response is cached.

### Retrieve embedded text
```
GET /v1/books/<id>/text_elements
```

### Data retention
| State | Retained for |
|---|---|
| Unsaved | 1 year from last activity |
| Saved | 2 years from last save |
| Ordered | Indefinitely |

---

## 5. Book Files (Production PDFs)

### Create (initiate PDF generation)
```
POST /v1/books/<id>/files.json
```
This is a paid-per-call endpoint depending on your contract. Only one file per project is permitted. Delete before retrying.

For cut-print billing, supply a unique `order_uid` (64–255 characters):
```
POST /v1/books/<id>/files.json
-d "order_uid=<unique-order-identifier>"
```
`order_uid` must be unique per user+order combination and consistent across all projects in the same order.

### Check status
```
GET /v1/books/<id>/files.json
```
| Status | HTTP | Meaning |
|---|---|---|
| Queued for processing | 202 | Job in queue, not started |
| Started | 202 | Job running |
| Completed | 200 | File ready at `http_links` |
| Error occurred | — | See `error_message`; delete and retry |
| Deleted | — | File has been removed |

### Delete a file
```
DELETE /v1/files/<file-id>.json
```
Deletion is only permitted when: the job errored; the job was requested more than 2 hours ago and is still not complete; or the file was successfully generated more than 6 hours ago.

### File response object structure

The response from both `POST /v1/books/<id>/files.json` and the polling `GET` contains:

| Field | Description |
|---|---|
| `status` | Status string — see table above |
| `links.self` | URL to poll for status updates. Append `.json` when fetching. |
| `files` | Array of generated file objects (present when status is `Completed`) |

Each object in `files`:

| Field | Description |
|---|---|
| `link` | Direct download URL for the file |
| `type` | File type string, e.g. `cover`, `pages`, `pages1`, `pages2` |
| `format` | File extension string, e.g. `pdf` |
| `layer_name` | Layer name string if output is split by layer; `null` otherwise |

### Polling gotcha — error status typo

The actual API response string for a failed job is `"Error ocurred"` (single 'r') — not `"Error occurred"`. Polling code must match this exact string or the loop will never exit on error.

```javascript
while (!(file.status === 'Completed' || file.status === 'Error ocurred')) { ... }
```

---

## 6. Galleries and Images

### List user's galleries
```
GET /v1/users/<user-id>/galleries.json
GET /v1/galleries/_mine.json
```

### Create a gallery
```
POST /v1/users/<user-id>/galleries.json
-d "gallery[name]=my-gallery"
```
All galleries must be associated to a user.

### Read / update a gallery
```
GET  /v1/galleries/<id>.json
PUT  /v1/galleries/<id>.json  -d "gallery[name]=Updated+Name"
```

### Add an image
```
POST /v1/galleries/<id>/images.json
-F Filedata=@image.jpg
-d "tags=tag1,tag2"
```

### Read a single image
```
GET /v1/images/<id>.json
```

### Filter images by tags
```
GET /v1/galleries/<id>/images.json?tags[]=tag1&tags[]=tag2
```

---

## 7. Custom eCommerce Integration

This section covers integrating Pixfizz personalization and fulfillment into an external storefront (not Shopify — for Shopify see `60_SHOPIFY_INTEGRATION.md`).

### Prerequisites
- Pixfizz site must share the same base domain as the external site (e.g. `design.mystore.com` if main site is `www.mystore.com`).
- Add the custom hostname under **Settings → General → Domain Hosting** and point a CNAME to `hosting.pixfizz.com`.
- Add the external site hostname under **Settings → General → External Hosts** to authorize CORS requests.

### User Handoff

Endpoint: `POST /v1/users/_uid/<external-source>/<external-user-id>.json`

- `<external-source>` — string identifying your application
- `<external-user-id>` — unique user ID in your system

This endpoint creates the Pixfizz user if they don't exist, updates their data if it has changed, and logs them in. It is idempotent.

> **Login-capable vs external users.** Any user created with `user[external_id]` or
> `user[external_source]` in the POST body becomes an **external user**: they cannot log in
> with email/password and can only be reached from the integrated external site. This applies
> even when posting to `/v1/users` (not only to the `_uid` handoff endpoint). For regular
> login-capable accounts (OrderHub operators, normal storefront customers), **omit those
> fields entirely**. To repair an account created external by mistake, create a fresh user and
> merge the old one into it: `POST /v1/users/<id>#merge`.

POST body:
```
user[email]=...
user[first_name]=...
user[last_name]=...
hash=...
v=3
```

The `hash` is a server-side security calculation (never expose to browser):
```
md5_hexdigest(md5_hexdigest("<external-user-id>|<email>|<external-source>|<first-name>|<last-name>") + "<secret-key>")
```

Find `<secret-key>` in Pixfizz superadmin under **Website → API Settings → Shared Secret**.

Call this endpoint: on every page load if the user is logged in; always before any Pixfizz API interaction; immediately after login.

### Set session locale
```
POST /session/set_locale
-d "locale=fr"
```

### Log out
```
DELETE /v1/session.json
```

### Project workflow (custom integration)
1. POST to `/v1/books.json` with `product_code` and `theme_code`
2. Take the `id` from the response
3. Redirect user to: `https://design.mysite.com/v1/editor?book=<id>&cart_target=<cart-url>`
   - `<cart-url>` may contain `{{book_id}}` placeholder
4. On cart add, store the Pixfizz project ID to the orderline

To re-open for editing: `/v1/editor?book=<id>&cart=t`

---

## 8. Order Fulfillment (External Orders)

Endpoint: `POST /v1/admin/orders/_external/<external-source>.json`

Requires admin HTTP basic auth. Creates the order in Pixfizz, moves it to "Confirmed" status, and initiates fulfillment.

```json
{
  "order": {
    "external_reference": "myref-12345",
    "paid": true,
    "user_notes": "I need this by Thursday",
    "user": {
      "external_id": "1234321",
      "email": "my@email.com",
      "first_name": "John",
      "last_name": "Johnson",
      "telephone": "+12 333 2123"
    },
    "address": {
      "first_name": "Johnny",
      "last_name": "Johnson",
      "telephone": "123456",
      "company": "Company Ltd",
      "street": "Mystreet 42",
      "street2": "appartment 123",
      "city": "Mycity",
      "postcode": "12345",
      "region": "Some Region",
      "country_code": "FR"
    },
    "orderlines": [{
      "print_book_id": 48665,
      "product_code": "frame-24x24",
      "quantity": 2,
      "unit_price": "12.99",
      "discount": "0.49",
      "custom": {
        "external_line_id": "123456"
      }
    }],
    "custom": {
      "shipping_service": "Fedex"
    }
  }
}
```

Field notes:
- `print_book_id` — required for design products (personalized projects); omit for static products
- `product_code` — required for static products; omit for design products
- `discount` — total discount applied to the orderline, optional
- All `custom` blocks are optional and accept arbitrary key/value data
- Projects supplied in `orderlines` must be anonymous (not logged-in user's projects). Test in incognito.

### Fulfillment types

**Order-based:** Use the external order endpoint above. Pixfizz generates production files and routes them via job tickets to the configured fulfillment destination (HTTP endpoint or FTP).

**Project-based (pull):** Use the Book Files API (section 5). Your system polls for completed files and downloads them directly.

---

## 8a. Individual Orders — Read, Update, and Shipping

This section covers the single-order endpoint (`/v1/orders`), distinct from the external order creation endpoint in section 8.

### Listing (admin only)

```
GET /v1/admin/orders.json
```

Special admin namespace — lists orders across the whole website. Only accessible to users with admin privileges for the website. This differs from the single-order endpoint below.

### Creating (from a cart)

Given a Cart object with orderlines already added, create an order via:

```
POST /v1/orders
-d "cart_id=2112"
-d "user_id=12345"
```

The `cart_id` for a given session is obtained via the `<px:cart:id>` CMS tag.

### Reading

```
GET /v1/orders/<id>.json
```

### Updating — admin user

```
PUT /v1/orders/<id>
-d "order[status]=S"
```

Admin-editable fields: `notes`, `status`, `custom`, `paid`. Any other field sent will be ignored.

### Marking an order as Shipped

Set the order's status code to `S`:

```
PUT /v1/orders/<id>
-d "order[status]=S"
```

This triggers Pixfizz's internal order-completion processes (see `32_ORDER_LIFECYCLE.md` for what fires — notification emails, etc., subject to Settings > Email Notifications and any OrderHub suppression).

### Updating — regular (non-admin) user

A logged-in user can update their own order's public custom fields client-side, e.g. to track page views:

```
PUT /v1/orders/<id>
-d "order[custom][thankyou_page_viewed]=true"
```

Only `custom` can be set this way. Any non-public field a regular user attempts to set is silently ignored.

---

## 9. Dynamic Design Previews

Preview a design (theme) without creating a project.

**URL pattern:**
```
/v1/themes/<theme-id>/preview.<ext>?<query-params>
```

Supported extensions: `jpg`, `webp`, `svg`

SVG is recommended for performance and sharpness at small sizes, but cannot be used in `<img src>`. Use `<object type="image/svg+xml" data="...">` instead.

### Query parameters

| Parameter | Description |
|---|---|
| `width` | Output width in px (default 100) |
| `height` | Output height in px |
| `variants[<variant-code>]` | Set a product variant value |
| `template_options[<option-code>]` | Set a template option value |
| `template_name` | Name of the page to preview (default: first page) |
| `page` | Page number (0-indexed). `page=0` = first page |
| `fulfillment` | `false` (default) uses the preview pipeline and renders "show on preview" placeholders but caps width at 1200px; `true` renders at the full requested width but strips those placeholders. Not officially documented on this endpoint. |

Multi-choice variants/options expect the option code. Text variants expect the text string.

Example:
```
/v1/themes/132496/preview.jpg?width=800&template_options[name]=Smith&template_options[base-colour]=ivory
```

### Project previews
```
https://<subdomain>.pixfizz.com/v1/books/<project-id>/preview.webp?width=800
```
Requires admin access.

### Preview resolution and production-quality output

- The theme and project preview endpoints above are optimised for on-page previews, not
  print. Output is **capped at `width=1200`** and rendered at a lower JPEG quality.
- For higher-quality, production-resolution output, render the page directly:
  `/v1/pages/<page-id>.jpg?width=X&fulfillment=true`. This uses the production render
  settings rather than the preview pipeline.
- The `fulfillment=true` page endpoint is currently **superadmin (omnipotent) only**;
  opening it to all admins is under consideration. Confirm current access before relying on it.
- **`fulfillment` on the theme/project preview endpoint trades placeholders against
  resolution.** With `fulfillment=false` (preview pipeline) elements set to "show on
  preview" ARE rendered, but output is still capped at `width=1200`. With
  `fulfillment=true` the request renders at the full requested width but "show on
  preview" placeholder elements are **stripped**. There is currently no way to get
  both full resolution and "show on preview" placeholders from this endpoint — they
  are architecturally coupled. This parameter is not officially documented on the
  preview endpoint; confirm with the platform team before building on it, and treat a
  raised/removed 1200px cap as a feature request.

---

## 10. JS API

The JS API is a client-side library for integrating Pixfizz into an external storefront page. For the Shopify-specific wrapper (`Pixfizz.Shopify.*`) see `60_SHOPIFY_INTEGRATION.md`.

### Setup

The JS API script is loaded from the Pixfizz subdomain. The subdomain must share the same base domain as the external site to avoid third-party cookie restrictions.

Initialize before calling any other function:
```javascript
Pixfizz.setup('create.myshop.com', {uid: '12345', email: 'user@server.com', first_name: 'Bob'}, 'security-hash');
```

Security hash calculation:
1. Take `uid` and all other user fields present, ordered alphabetically. Join values with `|`. MD5 hash the result.
2. Concatenate that MD5 with the shared secret. MD5 hash again.

Find the shared secret in Pixfizz admin under **Site → General → API Settings**.

Add the external hostname under **Site → General → Domain Hosting** for CORS to work.

### Functions

**`Pixfizz.setup(website_url, [user_data], [security_hash])`**
Initializes the API. Always call first.

**`Pixfizz.createProject(pxid, [params])`**
Creates a new project and opens it in the editor.
- `pxid` — `theme-code:product-code` string from Pixfizz admin
- Common params: `cart_target`, `exit_target`, `save_target`

```javascript
Pixfizz.createProject('layflat-11x7.5-silver:layflat-11x7.5', {
  cart_target: 'http://mysite.com/cart?add={{book_id}}'
});
```

**`Pixfizz.openProject(project_id, [params])`**
Opens an existing project in the editor.
- Common params: `target`, `exit_target`, `cart_target`, `save_target`

**`Pixfizz.openPage(path, [params])`**
Opens a page on the Pixfizz site.

**`Pixfizz.api(url, options)`**
XHR wrapper around JS `fetch`. Handles token automatically. Use relative URLs. Supports `:user_id` placeholder.

```javascript
Pixfizz.api('/v1/books/_mine.json').then(r => r.json()).then(response => {
  console.log(response);
});
```

**`Pixfizz.getUserToken(callback)`**
Returns a short-lived login token. Token is cached in a cookie. Used internally by `createProject`, `openProject`, `openPage`, and `api`.

**`Pixfizz.getUserId(callback)`**
Returns the current Pixfizz user ID, or `null` for anonymous users.

---

## 11. Calendars

### List / create user calendars
```
GET  /v1/users/<id>/calendars.json
GET  /v1/calendars/_mine.json
POST /v1/users/<id>/calendars.json  -d "calendar[name]=Family 2026"
```

### Admin calendars (site-wide, requires admin)
```
POST /v1/calendars.json  -d "calendar[name]=Holidays"  -d "calendar[code]=HOL2026"
```
`code` is required for admin calendars; not required for user calendars.

### CRUD
```
GET    /v1/calendars/<id>.json
PUT    /v1/calendars/<id>.json  -d "calendar[name]=Updated"
DELETE /v1/calendars/<id>.json
```
Deleting a calendar also deletes its dates.

### Calendar dates
```
GET    /v1/calendars/<id>/dates.json
GET    /v1/calendars/<id>/dates/<date-id>.json
POST   /v1/calendars/<id>/dates.json  -d "date[caption]=Birthday"  -d "date[start]=2026-04-23"
DELETE /v1/calendars/<id>/dates/<date-id>.json
```
Start dates must use ISO 8601. Time values are ignored.

---

## 12. Colors and Fonts (read-only)

```
GET /v1/colors.json
GET /v1/fonts.json
```

Colors and fonts are organized into palettes. Individual colors/fonts cannot be added, updated, or removed via the API — only read. Each palette has an `id`, `name`, and an array of items.

---

## 13. Fulfillment Partner Callbacks

Pixfizz accepts inbound shipping/status callbacks from fulfillment partners at:
```
POST https://login.pixfizz.com/custom/<partner>/order_callback
```
Requires HTTP basic auth. Supported partners: Advertek, Navitor, Gooten, PRNTMSTR, Siteflow.

Callback payloads carry shipment status, tracking name, tracking code, tracking URL, and package IDs. The payload schema varies by partner.

**Note:** Callback endpoint credentials are operational secrets and are not documented here. Retrieve from the Pixfizz Notion wiki (Callbacks from Fulfillment Partners page) or contact support.

---

## 13a. Networking: No Fixed Outbound IP

Pixfizz runs on a pool of workers and does **not** have a fixed outbound IP address —
traffic from Pixfizz (FTP pushes, outbound API calls, fulfillment posts) originates
from whichever worker handles the job. Do not ask a partner or customer to
IP-whitelist Pixfizz, and do not build an integration that depends on a stable Pixfizz
source IP. Authenticate integrations by another method (basic auth, tokens, signed
requests) instead of IP allow-listing.

---

## 13b. Order IDs on Reprints (Fulfillment JSON)

When an order is reprinted, the reprint can carry the **same order ID** as the original
in the fulfillment JSON/job ticket, which some downstream systems reject as a duplicate.
To keep the emitted order ID unique across reprints, append a letter suffix per reprint
(e.g. `A`, `B`, ...). Confirm the exact suffixing rule with the platform team when wiring
a new fulfillment integration.

---

## 13c. Admin Content API — Custom Types, Assets, and Custom Fields

> **Not publicly announced.** These endpoints work but are not in the published API documentation. Treat them as subject to change and confirm behaviour against the target site before building on them.

Authentication is HTTP Basic with an admin account, as in § 2.

**Path prefix inconsistency:** some of these endpoints sit under `/admin/...` and others under `/v1/admin/...`, as listed below. This is not a transcription error — the prefixes genuinely differ per endpoint. Do not assume a uniform prefix; test each one.

### Custom types

```
GET  /admin/custom_types.json                                    # list all custom types
GET  /admin/custom_types/<id>.json                               # single custom type
GET  /admin/custom_types/<id>/custom_type_instances.json         # list instances
POST /admin/custom_types/<id>/custom_type_instances.json         # create an instance
```

Create parameters, one per custom field on the type:

```
custom_type_instance[custom][<custom-field-name>]
```

The custom type `<id>` is numeric and site-specific. Fetch the list endpoint on the target site to find it rather than reusing an ID from another site.

### Assets

```
POST /admin/assets.json     # multipart encoded
GET  /admin/assets.json     # list all assets
```

Upload parameters:

```
asset[name]
asset[description]
asset[file]        # multipart-encoded file
```

The upload response returns the created asset IDs. Where a custom type instance needs to reference an image, upload the asset first, take the ID from the response, then create the instance referencing it.

### Updating custom fields on existing objects

Custom fields are written through the parent object's update endpoint, using the object's own parameter key. The key is not always the name you would expect from the admin UI — a Design is `theme`, a Collection is `theme_category`.

**Product attributes**

```
PUT /v1/admin/products/<product-id>.json

product[custom][custom_field_1]=value1
product[custom][custom_field_2]=value2
```

**Designs**

```
PUT /v1/admin/themes/<design-id>.json

theme[custom][custom_field_1]=value1
theme[custom][custom_field_2]=value2
```

**Collections**

```
PUT /admin/theme_categories/<collection-id>.json

theme_category[custom][custom_field_1]=value1
theme_category[custom][custom_field_2]=value2
```

Two things follow from § 13's CORS note and from `13_TEMPLATE_BOUNDARIES.md`:

- A cross-origin `PUT` triggers a CORS preflight. Use `POST` with `_method=put` from a browser context.
- Custom fields are site-specific and are not inherited parent to child. The field must already exist on the site being written to, or the value is silently dropped.

---

## 14. Retrieval Pointer

| Topic | File |
|---|---|
| Shopify JS API (`Pixfizz.Shopify.*`) | `60_SHOPIFY_INTEGRATION.md` |
| Fulfillment job ticket schema | `31_FULFILLMENT_ENGINE.md` |
| Pixfizz Liquid objects (user, order, etc.) | `50_LIQUID_REFERENCE.md` |
| Shopper template cart/checkout | `20_SHOPPER_CART_RULES.md`, `21_SHOPPER_CHECKOUT_POLICY.md` |
| Template responsibility boundaries | `13_TEMPLATE_BOUNDARIES.md` |

---

## Changelog
- 2026-03-30: Initial version. Compiled from Pixfizz Notion wiki: Pixfizz API Documentation, Create Order API Endpoint, Callbacks from Fulfillment Partners, Creating a Project, Dynamic Design Previews, Custom eCommerce CMS Integration Notes.
- 2026-06-15: Documented login-capable vs external user creation (external_id / external_source param makes a user external; omit for login-capable accounts; merge to repair). Documented preview resolution cap (width 1200, lower quality) and the production-quality /v1/pages/<id>.jpg?fulfillment=true endpoint (superadmin-only). Source: slack-kb-sync (Matjaz, #development).
- 2026-07-01: Added § 8a — Individual Orders (list/read/create/update via `/v1/orders` and `/v1/admin/orders`), including the PUT `order[status]=S` pattern to mark an order Shipped and the custom-field-only update path for regular users. Source: claude-chat.
- 2026-07-04: Documented that `/copy` creates an unsaved project (set `book[saved]=1` on the new ID), and the cross-origin `PUT` CORS-preflight trap (use `POST` + `_method=put`). Source: claude-chat.
- 2026-07-20: Documented the `fulfillment` param on the theme/project preview endpoint (placeholder-vs-resolution trade-off, not officially documented); added §13a no-fixed-outbound-IP networking note; added §13b reprint order-ID uniqueness (append a letter suffix). Source: support-ticket, fulfillment-integration call, #development.
- 2026-07-25: Added § 13c Admin Content API — custom type list/read/instance-create, asset upload and list, and the custom-field update endpoints for products, designs (`theme`), and collections (`theme_category`). Marked not publicly announced; documented the genuine `/admin` vs `/v1/admin` prefix inconsistency. Source: internal notes (Matjaz).


=================================================================
FILE: 62_SHOPIFY_BUY_NOW_PERSONALIZE_LATER.md
=================================================================

# 62 — Shopify: Buy Now, Personalize Later (Draft Order Flow)

**Authority Scope:** Shopify + Pixfizz deployment path only. Covers the full end-to-end flow for the "Buy Now, Personalize Later" feature — from Shopify product page through Pixfizz CMS draft order completion. Also covers the `skip_draft_redirect` site-level option and the `pixfizz-setup.liquid` snippet. Grounded in the master Shopify CMS backup (2026-04-09) and the `pixfizz-launch-product-handler.liquid` snippet.

_Last updated: 2026-04-09_

---

## Overview

Buy Now, Personalize Later (internally: the draft order flow) allows a customer to purchase a personalized product without designing it at the time of checkout. A Pixfizz project is created immediately at point of purchase, but it is placed in draft status. The order sits in a waiting state in Pixfizz until the customer returns via a magic-link email, designs their product, and explicitly approves each orderline. Fulfillment is only triggered once all orderlines are approved.

---

## 1. Compatibility

The draft button is only available for these integration types:
- `editor`
- `options-to-editor`

It is **not available** for:
- `photo-prints`
- `options-to-cart`

This restriction is enforced in the Shopify product template itself:

```liquid
{% assign integration_type = product.metafields.pixfizz.integration_type %}
{% if integration_type == 'editor' or integration_type == 'options-to-editor' %}
  <button onclick="{% render 'pixfizz-launch-product-handler', draft: true %}" class="button button--full-width">
    Buy Now, Personalize Later
  </button>
{% endif %}
```

---

## 2. Shopify Product Page

### Button rendering

The "Buy Now, Personalize Later" button sits below the standard "Personalize" button in the product template Custom Liquid block. Both buttons use `pixfizz-launch-product-handler`, but the draft button passes `draft: true`.

### Handler behavior with `draft: true`

Inside `pixfizz-launch-product-handler.liquid`, when `draft` is truthy:

```liquid
{% if draft %}
  parameters.draft = true;
{% endif %}
```

This adds `draft=true` as a query parameter when calling `Pixfizz.Shopify.launchProduct(...)`, which opens the Pixfizz modal at `/site/shopify/product?...&draft=true`.

The rest of the handler is identical to the standard Personalize flow — SKU map, addons map, variant resolution, quantity, and `page_addon`/`option_addons` parameters are all passed through in the same way.

---

## 3. Pixfizz CMS — `shopify/product` Page

URL: `/site/shopify/product`

This page handles both the normal and draft flows. Draft is detected via:

```liquid
{% if request.params.draft == 'true' %}
  {% assign is_draft = true %}
{% else %}
  {% assign is_draft = false %}
{% endif %}
```

### Project creation

Regardless of draft or not, a project is created immediately via `POST /v1/books.json` with the product and design IDs, variant/option selections, and the Pixfizz token. The project exists in Pixfizz from this point forward.

### Branch: `editor` integration type with `is_draft = true`

Normally for `editor` type, the page auto-submits the form and then fires `postMessage('launch_project', ...)` to open the design tool. When `is_draft` is true, this branch is bypassed. Instead, the page behaves like `options-to-cart`: it fires `postMessage('add_to_cart', ...)` with the project already created.

### Branch: `options-to-editor` integration type with `is_draft = true`

The options modal is shown as normal. The submit button label changes from "Design Now" to "Add to Cart". On submit, a project is created and `postMessage('add_to_cart', ...)` is fired — same as the options-to-cart path.

### `_pixfizz_draft` cart property

When `is_draft` is true, the properties object sent in `add_to_cart` includes:

```javascript
properties._pixfizz_draft = true;
```

This is alongside the standard `_pixfizz_project_id`. The Shopify line item will therefore carry both properties after checkout.

---

## 4. Shopify Cart and Checkout

The customer proceeds through the normal Shopify cart and checkout. The line item properties visible on the order are:

- `_pixfizz_project_id` — the Pixfizz project ID
- `_pixfizz_draft` — `true`
- Any option/variant name properties set during creation

No special cart handling is required beyond what is already in place for the standard Pixfizz Shopify cart setup. The draft flag is informational at the Shopify level — Pixfizz reads it when processing the order webhook.

---

## 5. Order Webhook and Pixfizz Order Status

When Shopify fires the order payment webhook, Pixfizz processes the line items and creates the Pixfizz order. Orderlines where `_pixfizz_draft` is `true` are created in draft status (`is_draft: true`).

The Pixfizz order is placed in status `W` (waiting/draft). Fulfillment is **not** triggered at this stage.

---

## 6. Draft Order Email

Pixfizz sends the customer a transactional email containing a magic link to the draft order page. The link is generated using `signin_token` and expires after a configurable period (set in the email template, not at platform level).

### Key template constructs

```liquid
{% assign name = 'Personalize order ' | append: order.external_reference %}
{% assign target_url = '/site/shopify/draft-order/' | append: order.id %}
{% capture token_code %}
  {% signin_token user, name, target_url: target_url, expires_in: '6 months' %}
{% endcapture %}
<a href="{{ website.signin_tokens[token_code].signin_url | escape }}">
  Click here to start personalizing
</a>
```

- `signin_token` generates a single-use magic link that authenticates the customer into Pixfizz and redirects them to the draft order page.
- `expires_in: '6 months'` — the expiry is controlled here. Each client's email template may set a different value.
- `order.external_reference` is the Shopify order number.
- `order.id` is the Pixfizz order ID (not the Shopify order ID).
- The `name` parameter is a human-readable label for the token — no functional effect.

### Authentication flow

The magic link takes the customer to the Pixfizz CMS, logs them in automatically (no password required), and lands them directly on their draft order page. If they are already logged in, the session token is refreshed.

---

## 7. Pixfizz CMS — `shopify/draft-order/:order_id` Page

URL: `/site/shopify/draft-order/:order_id`

This is a standalone Pixfizz CMS page — it is **not** part of the Shopper template. It has its own `<html>` shell, loads Pixfizz CMS assets directly, and uses Bootstrap 4.6 layout classes.

### Order and orderline resolution

```liquid
{% assign order = user.orders | where: 'id', request.path_params.order_id | first %}
{% assign draft_orderlines = order.orderlines | where: 'is_draft', true %}
{% assign approved_orderlines = order.orderlines | where: 'is_draft', false %}
```

### Draft orderline UI

For each draft orderline, the page renders:

**Preview**
```liquid
{% if orderline.project %}
  {% assign mapped_preview = orderline.product.template.mapped_previews | first %}
  <px-project-preview
    project-id="{{ orderline.project.id }}"
    title="{{ orderline.product.name | escape }}"
    width="100"
    height="100"
    mapped-preview-settings="{{ mapped_preview.preview_settings | escape }}"
    timestamp="{{ orderline.project.updated_at | date: '%s' }}"
    class="img-fluid">
  </px-project-preview>
{% else %}
  <img src="{{ orderline.product.image | asset_url | escape }}" width="100" height="100" alt="product">
{% endif %}
```

If no project exists on the orderline (edge case), falls back to the product image.

**Edit Project button**
```liquid
{% assign edit_url = orderline.project | design_tool_url: cart: 't', cart_target: request.path %}
<a class="btn btn-sm btn-block btn-outline-dark mb-3" href="{{ edit_url | escape }}">
  Edit Project
</a>
```

Opens the Pixfizz design tool. `cart_target: request.path` causes the design tool to return the customer to the draft order page (not the Shopify cart) when they are finished.

**Approve Now button**
```liquid
{% form 'orderline_commit', orderline: orderline %}
  <button class="btn btn-sm btn-block btn-outline-dark"
    {% if orderline.project and orderline.project.updated_at <= order.created_at %}
      disabled
      title="You have to edit the project before you can approve it"
    {% endif %}
  >
    Approve Now
  </button>
{% endform %}
```

- Uses the `orderline_commit` Pixfizz form tag, which commits the draft orderline to confirmed status.
- The button is **disabled** if the project has not been modified since the order was created (`updated_at <= order.created_at`). The customer must edit first.
- Each orderline has its own independent Approve button — approval is per-orderline, not per-order.

### Approved orderline UI

Approved orderlines are shown in a read-only section with preview and product name/price only — no edit or approve controls.

### Order status guard

```liquid
{% if order.status != 'W' %}
  <p>Order {{ order.external_reference }} is no longer in draft status.</p>
{% endif %}
```

If the order has moved out of `W` (waiting) status, the page informs the customer rather than showing the approval UI.

### Fulfillment trigger

When all orderlines on the order are committed (all `is_draft` flags become `false`), Pixfizz automatically moves the order from `W` to confirmed status and kicks off the fulfillment pipeline. There is no manual step required on the admin side.

---

## 8. Known Bugs

### `draft_orderline.total_items` typo

On the draft order page, the count of waiting orderlines references the wrong variable:

```liquid
{# BUG — should be draft_orderlines.total_items (plural) #}
<p>Number of order lines waiting to be designed and confirmed: {{ draft_orderline.total_items }}</p>
```

`draft_orderline` (singular) is not defined — `draft_orderlines` (plural) is the array. This line will render blank. Fix:

```liquid
<p>Number of order lines waiting to be designed and confirmed: {{ draft_orderlines.total_items }}</p>
```

---

## 9. CMS Pages and Snippets Reference

All of the following live in the master Shopify CMS site (the standalone Pixfizz CMS that backs the Shopify integration — not a Shopper site):

| Page/Snippet | URL / Name | Purpose |
|---|---|---|
| Product page | `shopify/product` | Creates project, handles draft vs normal branch, fires postMessage |
| Draft order page | `shopify/draft-order/:order_id` | Customer-facing design + approval UI |
| Project edit page | `shopify/project-edit` | Options/variant editor for existing projects (used from cart edit, not draft flow) |
| Token page | `shopify/token` | Magic link authentication — logs customer into Pixfizz via iframe |
| API JS | `shopify/api.js` | `Pixfizz.Shopify.*` client-side integration library |
| Launch handler snippet | `pixfizz-launch-product-handler` | Shopify theme snippet — builds and fires `launchProduct()` |

---

## 10. Email Template Notes

- The draft order notification email is a transactional template managed in Pixfizz admin under email notifications.
- Each client site has its own branded version — logo, body copy, and footer are client-specific.
- The `expires_in` value on the `signin_token` tag controls how long the magic link remains valid. The Canvas and More template sets this to `6 months`.
- The email does not include a preview of the ordered items — the customer sees the items only after clicking through to the draft order page.
- There is no built-in copy for what happens if the link expires. Client templates should include guidance.

---

## 11. Relationship to Other Files

- **60_SHOPIFY_INTEGRATION.md** — covers the standard (non-draft) Shopify setup: metafields, snippets, cart page, webhook, product linking, troubleshooting.
- **32_ORDER_LIFECYCLE.md** — covers Pixfizz order statuses including `W` (waiting/draft).
- **11_USER_IDENTITY_MODEL.md** — covers how anonymous/registered sessions are handled; relevant because the magic link may authenticate a user who was previously anonymous.

---

## 12. `skip_draft_redirect` — Suppress Cart Redirect After Draft Add

By default, when a customer clicks "Buy Now, Personalize Later" and the project is added to the Shopify cart, `shopify/api.js` redirects the customer to the cart page. This is the same redirect behaviour as all other add-to-cart flows.

Some clients prefer to keep the customer on the product page after a draft add — particularly where the product page itself acts as a landing page or where the cart redirect disrupts the UX flow. The `skip_draft_redirect` option enables this per site.

When enabled, the following happens instead of a redirect:
- The modal closes immediately.
- The cart icon bubble count updates in place without a page reload.
- A confirmation toast appears below the cart icon (desktop) or bottom-center (mobile), then fades out after 5 seconds.

### Scope

This option only suppresses the redirect for draft add-to-cart operations — identified by `properties._pixfizz_draft === true` on the postMessage payload. It has no effect on:
- The standard "Personalize" button (which opens the design tool fullscreen — no redirect is involved)
- `options-to-cart` products (non-draft)
- Any other postMessage type

### Third-party script compatibility note

Some Shopify themes include third-party scripts (e.g. `visually-a.js`) that monkey-patch `window.fetch` and may throw errors when processing the `cart/add.js` response. The `add_to_cart` handler includes a `.catch()` block that fires the same close/toast/bubble-update logic if the promise rejects for this reason. The item will already be in the cart — the catch block ensures the UI recovers correctly regardless.

### CMS changes — `shopify/api.js`

The following changes must be applied to `shopify/api.js` in the Pixfizz CMS. They are fully backwards-compatible — sites that do not set `skip_draft_redirect` in `setup()` are unaffected.

#### Globals block

**Find:**
```javascript
  Pixfizz.Shopify.cart_target = null;
```

**Replace with:**
```javascript
  Pixfizz.Shopify.cart_target = null;
  Pixfizz.Shopify.skip_draft_redirect = false;
  Pixfizz.Shopify.draft_toast_message = null;
```

#### `setup()` function

**Find:**
```javascript
    if (options.cart_target) {
        Pixfizz.Shopify.cart_target = options.cart_target;
    }
```

**Replace with:**
```javascript
    if (options.cart_target) {
        Pixfizz.Shopify.cart_target = options.cart_target;
    }
    if (options.skip_draft_redirect) {
        Pixfizz.Shopify.skip_draft_redirect = true;
    }
    if (options.draft_toast_message) {
        Pixfizz.Shopify.draft_toast_message = options.draft_toast_message;
    }
```

#### `_showDraftToast` function — add after `closeModal`

**Find:**
```javascript
  Pixfizz.Shopify.closeModal = function() {
    if (Pixfizz.Shopify._modal) {
      Pixfizz.Shopify._modal.remove();
      Pixfizz.Shopify._modal = null;
    }
  };
```

**Replace with:**
```javascript
  Pixfizz.Shopify.closeModal = function() {
    if (Pixfizz.Shopify._modal) {
      Pixfizz.Shopify._modal.remove();
      Pixfizz.Shopify._modal = null;
    }
  };

  Pixfizz.Shopify._showDraftToast = function() {
    const message = Pixfizz.Shopify.draft_toast_message || 'Added to cart';
    const toast = document.createElement('div');
    toast.setAttribute('role', 'status');

    const cartIcon = document.getElementById('cart-icon-bubble');
    const isDesktop = window.innerWidth >= 768;

    const baseStyles = [
      'position:fixed',
      'background:#333',
      'color:#fff',
      'padding:14px 20px',
      'border-radius:6px',
      'z-index:9999999',
      'font-size:14px',
      'line-height:1.4',
      'max-width:320px',
      'box-shadow:0 4px 16px rgba(0,0,0,0.25)',
      'opacity:0',
      'transition:opacity 0.3s ease'
    ];

    if (isDesktop && cartIcon) {
      const rect = cartIcon.getBoundingClientRect();
      baseStyles.push(`top:${Math.round(rect.bottom + 12)}px`);
      baseStyles.push(`right:${Math.round(window.innerWidth - rect.right)}px`);
    } else {
      baseStyles.push('bottom:24px');
      baseStyles.push('left:50%');
      baseStyles.push('transform:translateX(-50%)');
      baseStyles.push('text-align:center');
    }

    toast.style.cssText = baseStyles.join(';');
    toast.textContent = message;
    document.body.appendChild(toast);
    requestAnimationFrame(() => { toast.style.opacity = '1'; });
    setTimeout(() => {
      toast.style.opacity = '0';
      setTimeout(() => toast.remove(), 300);
    }, 5000);
  };
```

#### `add_to_cart` message handler

**Find:**
```javascript
          if (evt.data.data.redirect !== false) {
            window.location = Pixfizz.Shopify.cart_target || `${Shopify.routes.root}cart`;
          }
        });
        break;
```

**Replace with:**
```javascript
          const is_draft_add = evt.data.data.properties && evt.data.data.properties._pixfizz_draft === true;
          if (evt.data.data.redirect !== false && !(is_draft_add && Pixfizz.Shopify.skip_draft_redirect)) {
            window.location = Pixfizz.Shopify.cart_target || `${Shopify.routes.root}cart`;
          } else if (is_draft_add && Pixfizz.Shopify.skip_draft_redirect) {
            Pixfizz.Shopify.closeModal();
            Pixfizz.Shopify._showDraftToast();
            fetch(`${Shopify.routes.root}cart.js`)
              .then(r => r.json())
              .then(cart => {
                const bubble = document.querySelector('.cart-count-bubble');
                if (bubble) {
                  bubble.querySelector('[aria-hidden]').textContent = cart.item_count;
                  bubble.querySelector('.visually-hidden').textContent = cart.item_count + ' items';
                }
              })
              .catch(() => {});
          }
        }).catch(() => {
          const is_draft_add = evt.data.data.properties && evt.data.data.properties._pixfizz_draft === true;
          if (is_draft_add && Pixfizz.Shopify.skip_draft_redirect) {
            Pixfizz.Shopify.closeModal();
            Pixfizz.Shopify._showDraftToast();
            fetch(`${Shopify.routes.root}cart.js`)
              .then(r => r.json())
              .then(cart => {
                const bubble = document.querySelector('.cart-count-bubble');
                if (bubble) {
                  bubble.querySelector('[aria-hidden]').textContent = cart.item_count;
                  bubble.querySelector('.visually-hidden').textContent = cart.item_count + ' items';
                }
              })
              .catch(() => {});
          }
        });
        break;
```

No changes are needed to `shopify/product`, `pixfizz-launch-product-handler`, or any other CMS page or Shopify theme snippet.

### Shopify Theme — `pixfizz-setup.liquid`

`pixfizz-setup.liquid` is the canonical Shopify theme snippet that loads `shopify/api.js` and calls `Pixfizz.Shopify.setup()`. It is site-specific — each client has their own copy with their own `pixfizz_host` and `shared_secret` values.

Standard structure:

```liquid
{%- assign pixfizz_host = 'shopper.pixfizz.com:5748' -%}
{%- assign shared_secret = 'AXDQ6GdzX0iUwu2K1FRKIw' -%}

<script type="text/javascript" src="https://{{ pixfizz_host }}/site/shopify/api.js"></script>
<script type="text/javascript">
  (() => {
    {%- if customer %}
      {%- capture hash_str %}{{ customer.id }}|{{ customer.email }}{% endcapture -%}
      {%- capture hash %}{{ hash_str | md5 }}{{ shared_secret }}{% endcapture -%}
      const user = {
        uid: "{{ customer.id }}",
        email: "{{ customer.email }}"
      };
      const hash = "{{ hash | md5 }}";
    {%- else %}
      const user = null;
      const hash = null;
    {%- endif %}
    Pixfizz.Shopify.setup({{ pixfizz_host | json }}, user, hash);
  })();
</script>
```

To enable `skip_draft_redirect` for a client, find the `setup()` call and add the options object:

```javascript
    Pixfizz.Shopify.setup({{ pixfizz_host | json }}, user, hash, {
      skip_draft_redirect: true
    });
```

To also customise the confirmation toast message:

```javascript
    Pixfizz.Shopify.setup({{ pixfizz_host | json }}, user, hash, {
      skip_draft_redirect: true,
      draft_toast_message: "Added to cart! We'll email you a link to personalize your order."
    });
```

If the client already has other options set (e.g. `cart_target`, `locale`), add the new keys to the existing options object — do not create a second one.

### Cart bubble compatibility

The cart count bubble update targets `.cart-count-bubble [aria-hidden]` and `.cart-count-bubble .visually-hidden` — the standard Dawn structure. If a client is using a heavily customised theme where these selectors do not exist, the bubble update will silently no-op (the `.catch(() => {})` prevents any error). The toast and modal close will still work correctly.

### All available `setup()` options

| Option | Type | Default | Effect |
|---|---|---|---|
| `locale` | string | — | Sets the Pixfizz session locale |
| `cart_target` | string | — | Overrides the redirect URL after add-to-cart (all non-draft types) |
| `skip_draft_redirect` | boolean | `false` | Suppresses cart redirect after draft add-to-cart; closes modal, shows toast, updates cart bubble instead |
| `draft_toast_message` | string | `'Added to cart'` | Custom confirmation message shown in the toast when `skip_draft_redirect` is true |

---

## Changelog
- 2026-04-09: Created from CMS backup (2026-04-09), launch handler snippet, and email template review.
- 2026-04-09: Added § 12 — `skip_draft_redirect` option, CMS changes, and `pixfizz-setup.liquid` reference.
- 2026-04-15: Rewrote § 12 — added full implementation including `_showDraftToast`, cart bubble update, `.catch()` handler for third-party fetch interceptors, toast positioning (desktop below cart icon / mobile bottom-center), and `draft_toast_message` option. Validated on demo site.


=================================================================
FILE: 70_MYPIXFIZZ_OVERVIEW.md
=================================================================

# 70 — MyPixfizz Overview

**Authority Scope:** System identity, architecture, portals, tech stack, and integrations for my.pixfizz.com.

_Last updated: 2026-03-26_

---

## What MyPixfizz Is

MyPixfizz (`my.pixfizz.com`) is the internal ERP/CRM and customer self-service portal for Pixfizz. It is a separate product from the Pixfizz personalization platform — it does not handle photo product creation, ordering, or fulfillment. Its role is operational management.

Two audiences:
- **Pixfizz staff** — sales, ops, leadership. Full operational control: pipeline, CRM, tasks, finance, support, onboarding, roadmap.
- **Pixfizz customers** — photo labs and print businesses that subscribe to Pixfizz. Self-service access to their account, projects, support, and roadmap.

---

## Tech Stack

- **Frontend:** React + Vite + Tailwind CSS
- **Backend:** Lovable Cloud (Supabase under the hood — Postgres + Edge Functions + RLS)
- **Design style:** Dark-mode-first, Apple-inspired minimal SaaS UX
- **Auth:** OTP and password-based sign-in via Supabase Auth
- **Key external integrations:** Fireflies, QuickBooks, GA4, Pixfizz Order Webhook, Calendly

---

## Portal Structure

### Admin Portal
Routes: `/dashboard`, `/pipeline`, `/leads`, `/organizations`, `/contacts`, `/brands`, `/projects`, `/tasks`, `/onboarding`, `/invoices`, `/costs`, `/suppliers`, `/ideas`, `/roadmap`, `/product-intelligence`, `/call-log`, `/executive`, `/infrastructure`, `/admin/support`, `/admin/events`, `/admin/portal-preview/:orgId`

Full access for Pixfizz staff. No org-scoping — sees all data.

### Customer Portal
Routes: `/portal/*`

Scoped strictly to the user's own organization. Customers cannot see other orgs' data. RLS enforced at DB level.

### Admin Portal Preview
Route: `/admin/portal-preview/:orgId`

Lets a Pixfizz admin impersonate a customer's portal view — see exactly what that customer sees. Used for support and onboarding.

---

## Role & Access Model

- Roles stored in `user_roles` table
- Two primary roles: `admin` (Pixfizz staff), `customer` (subscriber org users)
- Protected routes with role checking at the router level
- Organization-scoped data isolation via RLS policies
- All customer portal queries are filtered to the user's `org_id`

---

## Key Concepts

### Organization
The master record for a Pixfizz customer (a photo lab or print business). Has a lifecycle status: Lead → Onboarding → Active → On Hold. All customer data (brands, projects, contacts, tasks, invoices, support cases) belongs to an organization.

### Brand
A brand within an organization. An org can have multiple brands. Each brand has its own logo, brand guide assets, GA4 config, and billing currency. Revenue data rolls up to brand level.

### Project
An internal Pixfizz project record for work being done for an org — e.g. a site build, integration, or feature rollout. Has notes, meetings, links, Loom videos, and a client-visibility toggle per item.

### Contact
A person at a customer organization. Bidirectional sync with platform `user` records. Contacts can be converted to users and vice versa.

### Task
The core execution unit. Tasks can belong to an org, brand, project, or be internal. Assignable to admin users, customer users, or contacts. Supports 27 standardized categories, recurring patterns, and a daily focus/priority system.

---

## Integrations Summary

| Integration | Purpose |
|---|---|
| **Fireflies** | Sync meeting transcripts, AI summaries, action item extraction, auto-match to org/project. Webhook ingestion. AI idea candidate creation (with mandatory human review before publishing). |
| **QuickBooks** | Invoice and payment sync. Token refresh. Webhooks for payment events. |
| **GA4** | Per-brand measurement IDs. Event outbox with retry logic. UTM attribution. Pixfizz order webhook feeds GA4 Measurement Protocol. |
| **Pixfizz Order Webhook** | Receives orders from the Pixfizz platform, processes for GA4 and reporting. |
| **Calendly** | Webhook ingestion of meeting events into the call log / CRM. |
| **Email (Edge Functions)** | Transactional emails: forgot password, onboarding invites, support agent notifications, idea promotion alerts. |

---

## Infrastructure Notes

- Edge Functions handle: SLA checking, GA4 event dispatch, QuickBooks token refresh, Fireflies sync, email sending, webhook processing.
- Infrastructure health check dashboard at `/infrastructure` — automated service monitoring with status strip visible in both portals.
- GA4 Event Log available for debugging the analytics pipeline.

---

## Relationship to Pixfizz Platform

MyPixfizz is **operationally connected** to the Pixfizz platform but is a separate codebase:
- Pixfizz orders flow into MyPixfizz via webhook for GA4 tracking and revenue reporting.
- Customer organizations in MyPixfizz correspond to Pixfizz subscriber accounts.
- MyPixfizz does not control the Pixfizz personalization engine, storefront, or fulfillment — those are handled by the Pixfizz CMS platform (see files 10–60).

---

## Changelog
- 2026-03-26: Initial version. Compiled from Lovable project summary.


=================================================================
FILE: 71_MYPIXFIZZ_FEATURES_ROUTES.md
=================================================================

# 71 — MyPixfizz Features & Routes

**Authority Scope:** Feature inventory and route map for my.pixfizz.com.

_Last updated: 2026-03-26_

---

## Admin Portal — Feature Map

### Dashboard & Executive

| Route | Feature | Notes |
|---|---|---|
| `/dashboard` | Admin Dashboard | Daily planning, active projects, task focus, smart nudges |
| `/executive` | Executive Dashboard | YTD revenue, YoY comparisons, profitability, IPI market penetration, LTV tracking, cost/revenue charts, marketing health telemetry |

### Sales & Pipeline

| Route | Feature | Notes |
|---|---|---|
| `/pipeline` | Pipeline Kanban | Drag-and-drop deal stages, MRR aggregates in GBP, days-in-stage badges, momentum arrows, stale deal detection (14+ days) |
| `/leads` | Lead Inbox | Inbound leads, AI analysis, manual lead creation, auto-response target within 15 min |
| `/opportunities/:id` | Deal Detail | Tabbed: Overview, Activity, Notes, Emails, Meetings, Files |

### CRM

| Route | Feature | Notes |
|---|---|---|
| `/organizations` | Organizations | Master company records. Lifecycle: Lead → Onboarding → Active → On Hold. Tabs: Users, Tasks, Brands, Meetings, Documents, Links, Activity, Kickoff |
| `/contacts` | Contacts | CRM directory. Bidirectional user↔contact conversion. Auto-sync from platform users. |
| `/brands` | Brands | Per-org brand management. Logos, brand guides, social assets, GA4 config, billing currency, revenue sparklines. |
| `/projects` | Projects | Internal project hub. Notes, meetings, links, Loom videos, documents. Per-item client visibility toggles. |
| `/call-log` | Call Log | Fireflies-synced transcripts with AI summaries, action items, keyword extraction, org/project matching. |

### Task System

| Route | Feature | Notes |
|---|---|---|
| `/tasks` | Global Task List | Unified tasks across sales, onboarding, internal dev |
| (Dashboard panel) | Daily Planning Flow | Multi-step triage: Must Win Today, Weekly Wins, Smart Nudges |
| (Dashboard panel) | Today Focus Panel | Strict daily focus: overdue / due-today / must-win sections |

**Task rules:**
- 27 standardized categories (synced between DB and UI — do not add categories in only one place)
- Recurring patterns: daily, weekdays, weekly, biweekly, monthly, custom
- Auto-clone on completion for recurring tasks
- Midnight Mercy Rule: hourly reset of incomplete focus tasks
- Task Detail Drawer: inline-editable title, delegation context, comments, recurrence, project/org/brand/contact/user linking

### Onboarding

| Route | Feature | Notes |
|---|---|---|
| `/onboarding` | Onboarding Admin | Phase-based accordion, task tracking, stall detection, overdue checks |
| `/kickoff/:orgId` | Kickoff Wizard | Multi-section form: Vision, Branding, Products, Fulfillment, Homepage, Tax/Payment |
| (from Kickoff) | Onboarding Architect | AI-powered project scaffolding generated from kickoff form data |

### Product & Roadmap

| Route | Feature | Notes |
|---|---|---|
| `/ideas` | Ideas | Customer + internal submissions. Fireflies auto-creation with mandatory human review. Voting weighted by org revenue tier. Bulk actions, pagination. |
| `/roadmap` | Roadmap Kanban | Feature cards with priority scoring: revenue impact + votes + effort + complexity + strategic fit + sales blocker bonus. Target windows. Public visibility toggle. |
| `/product-intelligence` | Product Intelligence | IPI penetration tracking, ROI analysis |

### Finance & Billing

| Route | Feature | Notes |
|---|---|---|
| `/invoices` | Invoices | QuickBooks-synced. Line items, payment tracking, FX rates. |
| (from invoices) | Billing Runs | Monthly automated billing. CSV processing, loyalty discounts, VAT, multi-currency (EUR/USD → GBP conversion). |
| `/costs` | Costs | Company expense tracking. Suppliers, categories, receipt uploads. |
| `/suppliers` | Suppliers | Vendor management linked to costs. |
| (inline) | FX Tracker | Real-time exchange rate monitoring with historical charts. |

### Support (Admin)

| Route | Feature | Notes |
|---|---|---|
| `/admin/support` | Support Inbox | Case management. Metrics, assignee filtering, severity/type filters, conversation panel. |
| (inline) | SLA Checking | Automated SLA monitoring via edge function. |
| (settings) | Support Contacts | Managed staff directory. Drag-and-drop ordering, per-org visibility, avatar uploads. |

### Events

| Route | Feature | Notes |
|---|---|---|
| `/admin/events` | Admin Events | CRUD for webinars and workshops. |

### Infrastructure & Tools

| Route | Feature | Notes |
|---|---|---|
| `/infrastructure` | Infra Monitoring | Service health dashboard. |
| (inline) | GA4 Event Log | Debug and monitor the GA4 analytics pipeline. |
| (global) | Command Palette | Keyboard-driven navigation. |
| (global) | Keyboard Shortcuts | Task management hotkeys. |
| `/admin/portal-preview/:orgId` | Portal Preview | Admin view of a specific customer's portal. |

---

## Customer Portal — Feature Map

All routes prefixed `/portal/*`. All data scoped to the user's organization.

| Feature | Notes |
|---|---|
| Customer Dashboard | Brands snapshot, active projects, upcoming events, What's New feed, support hub |
| Brands | Subscription/loyalty details with bidirectional sync to admin side |
| Projects | View projects with client-visible items only (visibility toggle controlled by admin) |
| Assets | Document uploads + link management. Admin controls delete. Client-visible toggles. |
| Roadmap View | Public roadmap features. Customer reactions and feedback on items. |
| Tasks | External-visible tasks only. Customer can update via RPC. |
| Events | Upcoming events (live indicators, Zoom links, .ics export) and past events (YouTube embeds + AI summaries) |
| Support Hub | Submit and track support cases. View support contacts with avatars and Calendly links. |
| What's New | Announcements feed |
| Profile | Account management |

---

## Prompt-Writing Notes (for Lovable)

When asking Lovable to modify or build features, always specify:
- Which portal (admin or customer) the change is in
- The route or component name if known
- Whether it involves an edge function, RLS policy, or DB change
- If it touches a Fireflies/QuickBooks/GA4 integration, name the integration explicitly
- For task-related changes: note whether it affects recurring logic, the daily focus system, or the 27-category enum — these are tightly coupled

---

## Changelog
- 2026-03-26: Initial version. Compiled from Lovable project summary.


=================================================================
FILE: 72_MYPIXFIZZ_DATA_MODEL.md
=================================================================

# 72 — MyPixfizz Data Model

**Authority Scope:** Supabase database schema for my.pixfizz.com.

_Last updated: 2026-03-26_

---

## Enums

| Enum | Values |
|---|---|
| `app_role` | `admin`, `customer` |
| `invoice_line_category` | `Subscription`, `TransactionFees`, `ProServices`, `Discount`, `Other` |

---

## Views

| View | Purpose |
|---|---|
| `organizations_safe` | Restricted view of `organizations` — non-sensitive columns only (id, name, status, type, country, timezone, onboarding_status, customer_since, created_at, updated_at). Used for customer-facing queries. |

---

## Core Entity Map

Everything in MyPixfizz ultimately connects to `organizations`. Key relationships:

- `organizations` → `brands` (one-to-many)
- `organizations` → `contacts` (one-to-many)
- `organizations` → `projects` (one-to-many)
- `organizations` → `opportunities` (one-to-many)
- `organizations` → `tasks` (many — via organization_id)
- `organizations` → `support_cases` (one-to-many)
- `organizations` → `financial_invoices` (one-to-many)
- `organizations` → `onboarding_projects` (one-to-many)
- `organization_members` — maps auth.users to organizations
- `user_roles` — maps auth.users to role (admin / customer)

---

## Table Reference

### announcements
Stores system-wide or org-specific announcements shown in the customer portal.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| body | text | |
| category | text | default value |
| priority | text | default value |
| system_scope | text | |
| is_published | boolean, nullable | |
| published_at | timestamptz, nullable | |
| requires_acknowledgement | boolean | |
| image_url | text, nullable | |
| video_url | text, nullable | |
| link_url | text, nullable | |
| link_label | text, nullable | |
| organization_id | uuid, nullable | FK → organizations (null = system-wide) |
| created_by | uuid, nullable | |
| created_at | timestamptz, nullable | |

---

### billing_invoices
Monthly billing invoices per brand, linked to billing runs.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| billing_run_id | uuid | FK → billing_runs |
| brand_id | uuid | FK → brands |
| organization_id | uuid | FK → organizations |
| currency | text | |
| gross_revenue | numeric | |
| gross_revenue_currency | text, nullable | |
| transaction_fees | numeric | |
| subscription_fees | numeric | |
| sms_fees | numeric | |
| perfectly_clear_fees | numeric | |
| promo_credits | numeric | |
| loyalty_discount_pct | numeric | |
| loyalty_discount_amount | numeric | |
| net_before_vat | numeric | |
| vat_rate | numeric | |
| vat_amount | numeric | |
| total_due | numeric | |
| total_gbp | numeric | Normalized to GBP for reporting |
| csv_filename | text, nullable | |
| csv_row_count | integer | |
| qb_invoice_id | text, nullable | QuickBooks invoice ID |
| qb_status | text | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### billing_line_items
Individual line items within a billing invoice.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| invoice_id | uuid | FK → billing_invoices |
| category | text | |
| row_count | integer | |
| subtotal | numeric | |
| created_at | timestamptz | |

---

### billing_payment_lines
Links payments to specific QuickBooks invoices with amounts applied.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| payment_id | uuid | FK → billing_payments |
| invoice_qb_id | text | |
| amount_applied | numeric | |
| created_at | timestamptz | |

---

### billing_payments
Payments received, synced from QuickBooks.

| Column | Type | Notes |
|---|---|---|
| id | text, PK | QuickBooks payment ID |
| customer_qbo_id | text | |
| customer_display_name | text, nullable | |
| total_amount | numeric | |
| currency | text | |
| txn_date | text, nullable | |
| payment_method | text, nullable | |
| deposit_to_account | text, nullable | |
| organization_id | uuid, nullable | FK → organizations |
| realm_id | text, nullable | |
| raw_json | jsonb, nullable | |
| last_synced_at | timestamptz | |
| created_at | timestamptz | |

---

### billing_run_fx_rates
FX rates used for a specific billing run.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| billing_run_id | uuid | FK → billing_runs |
| source_currency | text | |
| target_currency | text | |
| rate | numeric | |
| created_at | timestamptz | |

---

### billing_runs
Monthly billing run records with period and status.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| period_year | integer | |
| period_month | integer | |
| status | text | |
| fx_rate_usd_gbp | numeric | |
| fx_rate_eur_gbp | numeric | |
| notes | text, nullable | |
| finalized_at | timestamptz, nullable | |
| created_by | uuid | |
| created_at | timestamptz | |

---

### brands
Customer brands/storefronts, each linked to an organization.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| organization_id | uuid | FK → organizations |
| subdomain | text, nullable | Pixfizz subdomain |
| website_code | text, nullable | |
| logo_url | text, nullable | |
| brand_guide | jsonb, nullable | |
| storefront_type | text, nullable | |
| billing_currency | text, nullable | |
| ga4_enabled | boolean | |
| ga4_measurement_id | text, nullable | |
| ga4_api_secret | text, nullable | |
| ga4_send_debug | boolean | |
| webhook_secret | text, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### call_log
Fireflies-synced meeting transcripts and call records.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| fireflies_transcript_id | text, unique | |
| title | text, nullable | |
| summary | text, nullable | |
| overview | text, nullable | |
| shorthand_bullets | text, nullable | |
| detailed_notes | text, nullable | |
| action_items | jsonb, nullable | |
| attendees | jsonb, nullable | |
| keywords | text[], nullable | |
| meeting_date | timestamptz, nullable | |
| duration_minutes | integer, nullable | |
| transcript_url | text, nullable | |
| organization_id | uuid, nullable | FK → organizations |
| contact_id | uuid, nullable | FK → contacts |
| opportunity_id | uuid, nullable | FK → opportunities |
| project_id | uuid, nullable | FK → projects |
| match_status | text | Matching state to org/project |
| customer_title | text, nullable | |
| customer_notes | text, nullable | |
| synced_at | timestamptz | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### company_costs
Internal company operating costs and expenses.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| category | text | |
| vendor | text, nullable | |
| amount | numeric | |
| currency | text | |
| frequency | text | |
| start_date | date | |
| end_date | date, nullable | |
| billing_day | integer, nullable | |
| invoice_date | date, nullable | |
| invoice_ref | text, nullable | |
| is_paid | boolean | |
| notes | text, nullable | |
| receipt_url | text, nullable | |
| supplier_id | uuid, nullable | FK → suppliers |
| created_by | uuid | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### contacts
CRM contacts — people linked to organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| first_name | text | |
| last_name | text | |
| email | text | |
| phone | text, nullable | |
| job_title | text, nullable | |
| department | text, nullable | |
| organization_id | uuid | FK → organizations |
| is_primary | boolean | |
| status | text | |
| preferred_contact_method | text | |
| linkedin_url | text, nullable | |
| notes | text, nullable | |
| user_id | uuid, nullable | Implicit FK → auth.users (bidirectional sync) |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### daily_goals
Per-user daily planning goals with sub-goals.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| user_id | uuid | |
| goal_date | date | |
| goal_text | text, nullable | |
| sub_goals | jsonb | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### email_invoice_inbox
Inbound invoice emails parsed and linked to costs/suppliers.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| sender_email | text | |
| subject | text, nullable | |
| received_at | timestamptz | |
| status | text | pending/processing/completed/failed/duplicate/skipped |
| pdf_filename | text, nullable | |
| extracted_data | jsonb, nullable | |
| error_message | text, nullable | |
| idempotency_key | text, nullable | |
| linked_cost_id | uuid, nullable | FK → company_costs |
| linked_supplier_id | uuid, nullable | FK → suppliers |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### events
Public/internal events (webinars, workshops) shown to customers.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| description | text, nullable | |
| event_date | timestamptz | |
| event_type | text, nullable | |
| status | text | upcoming/live/completed |
| duration_minutes | integer, nullable | |
| zoom_link | text, nullable | |
| youtube_id | text, nullable | For past events |
| thumbnail_url | text, nullable | |
| ai_summary | text, nullable | |
| is_visible | boolean, nullable | |
| created_at | timestamptz, nullable | |
| updated_at | timestamptz, nullable | |

---

### feature_feedback
Customer feedback messages on specific roadmap features.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| feature_id | uuid | FK → features |
| user_id | uuid, nullable | |
| organization_id | uuid, nullable | |
| contact_email | text | |
| company_name | text, nullable | |
| subject | text | |
| body | text | |
| created_at | timestamptz | |

---

### feature_ideas
Join table linking features to ideas they originated from.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| feature_id | uuid | FK → features |
| idea_id | uuid | FK → ideas |
| created_at | timestamptz | |

---

### feature_opportunities
Links features to sales opportunities blocked by them. Feeds sales_blocker_bonus scoring.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| feature_id | uuid | FK → features |
| opportunity_id | uuid | FK → opportunities |
| estimated_subscription_mrr | numeric | |
| created_at | timestamptz | |

---

### feature_organizations
Links features to requesting/interested organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| feature_id | uuid | FK → features |
| organization_id | uuid | FK → organizations |
| created_at | timestamptz | |

---

### feature_reactions
Customer sentiment reactions on roadmap features.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| feature_id | uuid | FK → features |
| user_id | uuid | |
| organization_id | uuid, nullable | |
| reaction | text | positive/neutral/negative — validated by trigger |
| created_at | timestamptz | |

---

### features
Product roadmap features with priority scoring.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| description | text, nullable | |
| status | text | |
| target_window | text, nullable | |
| effort_days | numeric | |
| complexity_score | integer | |
| strategic_fit_score | numeric | |
| competitive_pressure_score | numeric | |
| weighted_vote_score | integer | Aggregated from idea_votes |
| revenue_impact_score | numeric | |
| sales_blocker_bonus | integer | From feature_opportunities |
| effort_penalty | numeric | |
| priority_score | numeric | Composite scoring — see 71 for formula |
| manual_priority_override | numeric, nullable | |
| public_visible | boolean | Controls customer portal visibility |
| owner_user_id | uuid, nullable | |
| shipped_at | timestamptz, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### financial_invoice_line_items
QuickBooks-synced invoice line items with categorization.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| invoice_id | uuid | FK → financial_invoices |
| description | text, nullable | |
| qty | numeric | |
| unit_price | numeric | |
| line_total | numeric | |
| category | enum: invoice_line_category | Subscription/TransactionFees/ProServices/Discount/Other |
| qb_line_id | text, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### financial_invoice_sync_log
Audit log for QuickBooks invoice sync events.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| event_type | text | |
| status | text | |
| qb_invoice_id | text, nullable | |
| organization_id | uuid, nullable | |
| details | jsonb, nullable | |
| created_at | timestamptz | |

---

### financial_invoices
QuickBooks-synced invoices with payment status.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| qb_invoice_id | text | |
| qb_customer_id | text | |
| invoice_number | text, nullable | |
| customer_display_name | text, nullable | |
| organization_id | uuid, nullable | FK → organizations |
| status | text | Draft/Sent/Open/Overdue/Paid/Void/Deleted — validated by trigger |
| invoice_date | date, nullable | |
| due_date | date, nullable | |
| currency | text | |
| subtotal_amount | numeric | |
| tax_amount | numeric | |
| discount_amount | numeric | |
| total_amount | numeric | |
| balance_due | numeric | |
| email_status | text, nullable | |
| pdf_url | text, nullable | |
| realm_id | text, nullable | |
| raw_json | jsonb, nullable | |
| qb_last_updated_at | timestamptz, nullable | |
| synced_at | timestamptz | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### financial_settings
Key-value store for financial configuration (e.g. tax rates).

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| key | text, unique | |
| value | numeric | |
| description | text, nullable | |
| updated_at | timestamptz | |

---

### fulfillment_map
Maps product categories to fulfillment methods per organization.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| product_category | text | |
| method | text | |
| equipment | text, nullable | |
| created_at | timestamptz | |

---

### fx_rate_history
Historical foreign exchange rate snapshots.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| source_currency | text | |
| target_currency | text | |
| rate | numeric | |
| fetched_at | timestamptz | |

---

### ga4_event_attempts
Individual GA4 Measurement Protocol send attempts.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| outbox_id | uuid | FK → ga4_event_outbox |
| organization_id | uuid | FK → organizations |
| endpoint | text | |
| http_status | integer, nullable | |
| response_body | text, nullable | |
| error | text, nullable | |
| attempted_at | timestamptz | |

---

### ga4_event_outbox
Outbox queue for GA4 events pending delivery via Measurement Protocol.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| brand_id | uuid | FK → brands |
| organization_id | uuid | FK → organizations |
| source_system | text | |
| source_event | text | |
| source_event_id | text | |
| idempotency_key | text | |
| client_id_used | text | |
| ga4_payload | jsonb | |
| raw_payload | jsonb | |
| status | text | pending/sent/failed/disabled/skipped — validated by trigger |
| skip_reason | text, nullable | |
| attempt_count | integer | |
| last_attempt_at | timestamptz, nullable | |
| next_attempt_at | timestamptz | |
| last_http_status | integer, nullable | |
| last_error | text, nullable | |
| ga4_purchase_sent_at | timestamptz, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### idea_comments
User comments on ideas.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| idea_id | uuid | FK → ideas |
| user_id | uuid | |
| content | text | |
| is_deleted | boolean | |
| created_at | timestamptz | |

---

### idea_votes
Weighted votes on ideas, capped per organization.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| idea_id | uuid | FK → ideas |
| user_id | uuid | |
| organization_id | uuid | |
| vote_weight | integer | Auto-calculated by trigger based on org revenue tier |
| created_at | timestamptz | |

---

### ideas
Product ideas from customers, meetings, or internal submissions.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| description | text, nullable | |
| status | text | |
| source | text | |
| organization_id | uuid, nullable | FK → organizations |
| contact_id | uuid, nullable | FK → contacts |
| created_by_user_id | uuid | |
| reviewed_by_user_id | uuid, nullable | Fireflies-created ideas require human review before publishing |
| review_notes | text, nullable | |
| transcript_reference | text, nullable | |
| weighted_vote_score | integer | |
| raw_vote_count | integer | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### infrastructure_checks
Health check results for monitored infrastructure services.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| service_id | uuid | FK → infrastructure_services |
| status | text | |
| status_code | integer, nullable | |
| response_time_ms | integer, nullable | |
| error_message | text, nullable | |
| metadata | jsonb, nullable | |
| checked_at | timestamptz | |

---

### infrastructure_services
Definitions of monitored infrastructure services.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| description | text, nullable | |
| category | text | |
| check_type | text | |
| check_url | text, nullable | |
| expected_status | integer, nullable | |
| check_config | jsonb, nullable | |
| display_order | integer | |
| public_visible | boolean | Controls customer portal visibility |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### lead_inbox
Inbound sales leads from forms, Calendly, and manual entry.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| sender_email | text | |
| sender_name | text, nullable | |
| company_name | text, nullable | |
| phone | text, nullable | |
| message | text, nullable | |
| source | text | |
| status | text | |
| subject | text, nullable | |
| body_preview | text, nullable | |
| website_url | text, nullable | |
| assigned_to | uuid, nullable | |
| qualified_at | timestamptz, nullable | |
| converted_at | timestamptz, nullable | |
| converted_org_id | uuid, nullable | FK → organizations |
| converted_opp_id | uuid, nullable | FK → opportunities |
| deal_id | uuid, nullable | FK → opportunities |
| ai_analysis | jsonb, nullable | |
| ai_analysis_generated_at | timestamptz, nullable | |
| created_at | timestamptz | |

---

### marketing_channels
Marketing channel integrations (GA4, LinkedIn, YouTube).

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| display_name | text | |
| channel_type | text | ga4/linkedin/youtube — validated by trigger |
| sync_status | text | active/paused/error |
| external_account_id | text, nullable | |
| last_synced_at | timestamptz, nullable | |
| last_sync_error | text, nullable | |
| created_at | timestamptz | |

---

### marketing_metric_snapshots
Daily metric snapshots for marketing channels.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| channel_id | uuid | FK → marketing_channels |
| metric_date | date | |
| metric_key | text | |
| metric_value_numeric | numeric, nullable | |
| metric_value_text | text, nullable | |
| raw_payload | jsonb, nullable | |
| created_at | timestamptz | |

---

### notifications
In-app notifications delivered to users.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| user_id | uuid | |
| type | text | |
| title | text | |
| body | text, nullable | |
| read | boolean | |
| project_id | uuid, nullable | FK → projects |
| metadata | jsonb, nullable | |
| created_at | timestamptz | |

---

### onboarding_phases
Phases within an onboarding project.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| onboarding_project_id | uuid | FK → onboarding_projects |
| template_phase_id | uuid, nullable | FK → onboarding_template_phases |
| name | text | |
| order_index | integer | |
| status | text | pending/in_progress/completed/skipped — validated by trigger |
| completed_at | timestamptz, nullable | |
| created_at | timestamptz | |

---

### onboarding_projects
Customer onboarding projects with status tracking.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| organization_id | uuid | FK → organizations |
| template_id | uuid, nullable | FK → onboarding_templates |
| status | text | active/paused/completed/cancelled — validated by trigger |
| started_at | date | |
| target_date | date, nullable | |
| completed_at | timestamptz, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### onboarding_task_files
File attachments on onboarding tasks.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| task_id | uuid | FK → onboarding_tasks |
| file_name | text | |
| file_url | text | |
| file_size | integer, nullable | |
| uploaded_by | uuid, nullable | |
| created_at | timestamptz | |

---

### onboarding_tasks
Individual tasks within onboarding phases.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| phase_id | uuid | FK → onboarding_phases |
| template_task_id | uuid, nullable | FK → onboarding_template_tasks |
| title | text | |
| description | text, nullable | |
| task_type | text | |
| assignee | text | |
| status | text | pending/in_progress/completed/skipped/blocked — validated by trigger |
| order_index | integer | |
| due_date | date, nullable | |
| visibility | text | |
| notes | text, nullable | |
| link_url | text, nullable | |
| link_instructions | text, nullable | |
| form_schema | jsonb, nullable | |
| form_response | jsonb, nullable | |
| approval_content | text, nullable | |
| approval_status | text, nullable | |
| completed_at | timestamptz, nullable | |
| completed_by | uuid, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### onboarding_template_phases
Reusable phase definitions for onboarding templates.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| template_id | uuid | FK → onboarding_templates |
| name | text | |
| description | text, nullable | |
| order_index | integer | |
| is_milestone | boolean | |
| created_at | timestamptz | |

---

### onboarding_template_tasks
Reusable task definitions within onboarding template phases.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| phase_id | uuid | FK → onboarding_template_phases |
| title | text | |
| description | text, nullable | |
| task_type | text | |
| default_assignee | text | |
| is_required | boolean | |
| offset_days | integer | |
| order_index | integer | |
| link_url | text, nullable | |
| link_instructions | text, nullable | |
| form_schema | jsonb, nullable | |
| approval_content | text, nullable | |
| created_at | timestamptz | |

---

### onboarding_templates
Named onboarding templates optionally tied to org types.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| description | text, nullable | |
| org_type | text, nullable | |
| is_active | boolean | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### opportunities
Sales pipeline deals linked to organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| primary_contact_id | uuid, nullable | FK → contacts |
| owner_user_id | uuid, nullable | |
| stage | text | Pipeline stage |
| estimated_subscription_mrr | numeric | |
| expected_monthly_transaction_revenue | numeric | |
| probability_pct | integer | |
| expected_close_date | date, nullable | |
| next_step | text, nullable | |
| lost_reason | text, nullable | |
| momentum_level | text | |
| risk_level | text | |
| stage_entered_at | timestamptz, nullable | Used for days-in-stage calculation |
| last_activity_at | timestamptz, nullable | Used for stale deal detection (14+ days) |
| closed_at | timestamptz, nullable | |
| ai_summary | jsonb, nullable | |
| ai_summary_generated_at | timestamptz, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### opportunity_activities
Activity log entries for sales opportunities.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| opportunity_id | uuid | FK → opportunities |
| organization_id | uuid, nullable | FK → organizations |
| contact_id | uuid, nullable | FK → contacts |
| type | text | |
| summary | text, nullable | |
| notes | text, nullable | |
| occurred_at | timestamptz | |
| external_ref | text, nullable | |
| fireflies_id | text, nullable | |
| is_pinned | boolean, nullable | |
| created_by | uuid | |
| created_at | timestamptz | |

---

### org_activity_events
Organization-level activity feed events. Auto-generated by DB triggers.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| event_type | text | |
| source_table | text, nullable | |
| source_id | uuid, nullable | |
| actor_id | uuid, nullable | |
| summary | text, nullable | |
| body | text, nullable | |
| visibility | text | |
| metadata | jsonb, nullable | |
| created_at | timestamptz | |

---

### org_kickoff_* tables (Kickoff Wizard)

Data collected in the Kickoff Wizard (`/kickoff/:orgId`). Most are one-to-one with `organizations` (PK = organization_id). Some are one-to-many.

| Table | Cardinality | Key columns / notes |
|---|---|---|
| org_kickoff_branding | one-to-one | primary_color, accent_color, font_preference, needs_font_help, logo/favicon URLs |
| org_kickoff_competitors | one-to-many | competitor_name, order_index |
| org_kickoff_featured_products | one-to-many | product_name, priority_order |
| org_kickoff_fulfillment | one-to-one | production_model, shipping_provider, pickup_in_store_enabled |
| org_kickoff_homepage | one-to-one | homepage_layout_style, pro_portal_required |
| org_kickoff_inspiration_links | one-to-many | url, order_index |
| org_kickoff_navigation_items | one-to-many | label, order_index |
| org_kickoff_product_categories | one-to-many | category_name, priority_order, produced_in_house |
| org_kickoff_production_software | one-to-many | software_name, order_index |
| org_kickoff_products | one-to-one | pricing_model, volume_pricing_enabled, tax_exempt_required, pro_discount_percent |
| org_kickoff_profiles | one-to-one | Existence marker only |
| org_kickoff_tax_payment | one-to-one | tax_region, payment_gateway, stripe_connected, stripe_webhook_ready |
| org_kickoff_vision | one-to-one | brand_positioning, primary_customer_type, launch_focus, target_launch_date |

---

### org_links
Bookmarked links for an organization with client visibility toggle.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| name | text | |
| url | text | |
| category | text | |
| client_visible | boolean | Controls customer portal visibility |
| created_by | uuid | |
| created_at | timestamptz | |

---

### org_meeting_action_items
Action items from organization meetings.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| meeting_id | uuid | FK → org_meetings |
| title | text | |
| owner_id | uuid, nullable | |
| due_date | date, nullable | |
| status | text | |
| linked_task_id | uuid, nullable | FK → tasks |
| created_at | timestamptz | |

---

### org_meeting_agenda_items
Agenda items for organization meetings.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| meeting_id | uuid | FK → org_meetings |
| topic | text | |
| details | text, nullable | |
| priority | text | |
| order_index | integer | |
| created_by | uuid, nullable | |
| created_at | timestamptz | |

---

### org_meeting_participants
Participants in organization meetings.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| meeting_id | uuid | FK → org_meetings |
| contact_id | uuid, nullable | FK → contacts |
| name | text, nullable | |
| email | text, nullable | |
| created_at | timestamptz | |

---

### org_meetings
Organization-level meetings with notes and customer visibility.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| title | text | |
| meeting_date | timestamptz, nullable | |
| status | text | |
| internal_summary | text, nullable | |
| decisions | text, nullable | |
| attachments | jsonb, nullable | |
| fireflies_url | text, nullable | |
| call_log_id | uuid, nullable | FK → call_log |
| visible_to_customer | boolean | |
| created_by | uuid, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### organization_members
Maps users to organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| user_id | uuid | FK → auth.users |
| organization_id | uuid | FK → organizations |
| role | text, nullable | |
| created_at | timestamptz | |

---

### organizations
Master company records — the core entity everything rolls up to.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| status | text | Lead/Onboarding/Active/On Hold |
| type | text | |
| country | text, nullable | |
| timezone | text, nullable | |
| onboarding_status | text | |
| pipeline_status | text | |
| integration_type | text | |
| customer_since | date, nullable | |
| subscription_rate | numeric | |
| billing_currency | text | |
| billing_account_id | text, nullable | |
| billing_email | text, nullable | |
| billing_link | text, nullable | |
| billing_notes | text, nullable | |
| billing_details | jsonb, nullable | |
| payment_terms_days | integer | |
| auto_charge_approved | boolean | |
| contract_json | jsonb | |
| loyalty_eligible | boolean | |
| is_ipi_member | boolean | |
| is_msp_subscriber | boolean | |
| quickbooks_customer_id | text, nullable | |
| quickbooks_customer_name | text, nullable | |
| expected_transaction_baseline | numeric, nullable | |
| has_physical_location | text, nullable | |
| location_count | text, nullable | |
| lead_source | text, nullable | |
| sales_owner_id | uuid, nullable | |
| sales_notes | text, nullable | |
| last_sales_activity_at | timestamptz, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### project_documents
Files uploaded to projects or organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| file_url | text | |
| project_id | uuid, nullable | FK → projects |
| organization_id | uuid, nullable | FK → organizations |
| uploaded_by | uuid | |
| client_visible | boolean | |
| created_at | timestamptz | |

---

### project_links
Bookmarked URLs attached to projects.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| project_id | uuid | FK → projects |
| name | text | |
| url | text | |
| description | text, nullable | |
| created_by | uuid | |
| created_at | timestamptz | |

---

### project_looms
Loom video recordings attached to projects.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| project_id | uuid | FK → projects |
| title | text | |
| loom_url | text | |
| video_id | text | |
| visible_to_customer | boolean, nullable | |
| created_by | uuid | |
| created_at | timestamptz | |

---

### project_meetings
Fireflies meeting transcripts linked to projects.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| project_id | uuid | FK → projects |
| fireflies_transcript_id | text | |
| title | text, nullable | |
| summary | text, nullable | |
| meeting_date | timestamptz, nullable | |
| duration_minutes | integer, nullable | |
| transcript_url | text, nullable | |
| opportunity_id | uuid, nullable | FK → opportunities |
| visible_to_customer | boolean | |
| created_at | timestamptz | |

---

### project_notes
Free-text notes on projects or organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| project_id | uuid, nullable | FK → projects |
| organization_id | uuid, nullable | FK → organizations |
| author_id | uuid | |
| content | text | |
| created_at | timestamptz | |

---

### project_templates
Reusable project templates.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| description | text, nullable | |
| org_type | text, nullable | |
| created_at | timestamptz | |

---

### projects
Active internal projects linked to organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| description | text, nullable | |
| organization_id | uuid, nullable | FK → organizations |
| template_id | uuid, nullable | FK → project_templates |
| status | text | |
| visibility | text | |
| icon | text, nullable | |
| color | text, nullable | |
| started_at | date, nullable | |
| target_launch_date | date, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### qbo_tokens
QuickBooks OAuth token storage (single-row pattern).

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| refresh_token | text | |
| access_token | text, nullable | |
| access_token_expires_at | timestamptz, nullable | |
| updated_at | timestamptz | |

---

### service_cost_entries
Monthly revenue vs cost tracking per service type.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| service_type | text | |
| period_year | integer | |
| period_month | integer | |
| revenue_amount | numeric | |
| cost_amount | numeric | |
| currency | text | |
| notes | text, nullable | |
| created_by | uuid | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### suppliers
Vendor records linked to cost entries.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| contact_name | text, nullable | |
| contact_email | text, nullable | |
| website | text, nullable | |
| default_currency | text | |
| default_category | text | |
| notes | text, nullable | |
| quickbooks_vendor_id | text, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### support_case_satisfaction
CSAT ratings on resolved support cases.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| support_case_id | uuid | FK → support_cases (one-to-one) |
| user_id | uuid | |
| rating | integer | 1–5, validated by trigger |
| comment | text, nullable | |
| created_at | timestamptz, nullable | |

---

### support_cases
Customer support tickets and bug reports with SLA tracking.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| created_by_user_id | uuid | |
| type | text | bug/ticket — validated by trigger |
| status | text | new/awaiting_triage/awaiting_customer/in_progress/resolved/closed |
| severity | text, nullable | blocking/major/minor |
| category | text, nullable | |
| subject | text | |
| app_error_id | text, nullable | |
| domain | text, nullable | |
| app_context | jsonb, nullable | |
| priority | text, nullable | |
| assignee_contact_id | uuid, nullable | FK → support_contacts |
| case_number | integer | Auto-assigned by trigger |
| first_response_at | timestamptz, nullable | SLA tracking |
| resolved_at | timestamptz, nullable | SLA tracking |
| sla_first_response_hours | numeric, nullable | |
| sla_resolution_hours | numeric, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### support_contacts
Pixfizz staff displayed in the customer support hub.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| email | text | |
| title | text, nullable | |
| contact_type | text | Pixfizz/Expert |
| calendly_url | text, nullable | |
| public_notes | text, nullable | |
| avatar_url | text, nullable | |
| is_active | boolean | |
| receive_ticket_notifications | boolean | |
| sort_order | integer | Drag-and-drop ordering |
| excluded_org_ids | text[] | Orgs that should not see this contact |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### support_message_attachments
File attachments on support messages.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| support_message_id | uuid | FK → support_messages |
| storage_path | text | |
| file_name | text | |
| mime_type | text | |
| size_bytes | integer | |
| created_at | timestamptz | |

---

### support_message_links
URL links attached to support messages.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| support_message_id | uuid | FK → support_messages |
| url | text | |
| label | text, nullable | |
| created_at | timestamptz | |

---

### support_messages
Messages within support case conversations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| support_case_id | uuid | FK → support_cases |
| author_user_id | uuid | |
| visibility | text | external/internal — validated by trigger |
| body | text | |
| is_system | boolean | |
| created_at | timestamptz | |

---

### task_comments
Comments on tasks.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| task_id | uuid | FK → tasks |
| author_id | uuid | |
| content | text | |
| created_at | timestamptz | |

---

### tasks
Unified task system for all work items (sales, onboarding, internal, customer).

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| description | text, nullable | |
| status | text | |
| priority | text | |
| category | text | One of 27 standardized categories — synced between DB and UI |
| task_type | text | |
| visibility | text | |
| execution_state | text | |
| assignee_id | uuid, nullable | Admin user |
| assignee_contact_id | uuid, nullable | FK → contacts |
| organization_id | uuid, nullable | FK → organizations |
| project_id | uuid, nullable | FK → projects |
| brand_id | uuid, nullable | FK → brands |
| opportunity_id | uuid, nullable | FK → opportunities |
| source_meeting_id | uuid, nullable | FK → org_meetings |
| weekly_win_id | uuid, nullable | FK → weekly_wins |
| due_date | timestamptz, nullable | |
| completed_at | timestamptz, nullable | |
| order_index | integer | |
| manual_rank | integer | |
| today_flag | boolean | Daily focus flag |
| is_must_win_today | boolean | Must Win Today flag |
| must_win_date | date, nullable | |
| waiting_on | text, nullable | |
| reference_link | text, nullable | |
| recurrence_rule | jsonb, nullable | Recurring pattern config |
| delegation_context | jsonb, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### template_tasks
Task definitions within project templates.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| template_id | uuid | FK → project_templates |
| title | text | |
| description | text, nullable | |
| default_category | text | |
| default_visibility | text | |
| default_waiting_on | text | |
| offset_days | integer | |
| order_index | integer | |
| reference_link | text, nullable | |
| created_at | timestamptz | |

---

### user_roles
Maps users to application roles.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| user_id | uuid | unique with role — FK → auth.users |
| role | enum: app_role | admin/customer |
| created_at | timestamptz | |

---

### weekly_wins
Weekly win/goal items per user.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| description | text, nullable | |
| status | text | |
| week_start_date | date | |
| owner_user_id | uuid | |
| organization_id | uuid, nullable | FK → organizations |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

## Changelog
- 2026-03-26: Full schema populated from Lovable export. 88 tables documented.


=================================================================
FILE: 80_ONBOARDING.md
=================================================================

# 80 — Onboarding

**Authority Scope:** Onboarding process, deployment path phase sequences, pre-onboarding preparation, launch readiness, and email notification templates.

_Last updated: 2026-05-27_

---

## How to Use This Guide

This document covers the full onboarding process for getting a Pixfizz site live. It applies to all deployment paths.

- **If you're a new Pixfizz customer:** Start with "Preparing for Onboarding" — it tells you exactly what to gather before work begins.
- **If you're a Pixfizz team member:** Use the phase sequences for your deployment path to track progress and identify blockers.
- **If you're a prospect evaluating Pixfizz:** The deployment path overview and preparation checklist will give you a realistic picture of what's involved.

---

## Deployment Paths

Before anything else, confirm which deployment path applies. This determines which phases are in scope and what the customer owns vs what Pixfizz owns.

| Path | Storefront | Cart & Checkout | Personalization | Production |
|------|-----------|-----------------|-----------------|------------|
| Full Pixfizz (Shopper) | Pixfizz (Shopper template) | Pixfizz | Pixfizz | Pixfizz |
| Full Pixfizz (Custom) | Pixfizz (custom template) | Pixfizz | Pixfizz | Pixfizz |
| Shopify + Pixfizz | Shopify | Shopify | Pixfizz | Pixfizz |
| Custom API Integration | Customer's platform | Customer's platform | Pixfizz | Pixfizz |
| Marketplace (Etsy, etc.) | Marketplace | Marketplace | Pixfizz | Pixfizz |

**Full Pixfizz (Shopper)** is the most common path. The Shopper template provides a fully managed storefront with built-in checkout, cart, and navigation. Most photo labs and print businesses use this path.

**Full Pixfizz (Custom)** is for customers who need a completely bespoke storefront within the Pixfizz CMS — different layout, navigation, or UX than Shopper provides. This path inherits the Pixfizz CMS and eCommerce engine but builds the frontend from scratch. Higher cost, longer timeline.

**Shopify + Pixfizz** is for customers who already have or want a Shopify store. Shopify handles the storefront and checkout; Pixfizz handles personalization and production.

**Custom API Integration** is for customers who have an existing eCommerce platform (not Shopify) and want to add Pixfizz personalization and fulfillment. Requires developer resources on the customer's side.

**Marketplace** is for customers who sell through Etsy or similar marketplaces. Pixfizz handles personalization and production; the marketplace handles the storefront and payment.

---

## Preparing for Onboarding — What You Can Do Before We Start

This section is for customers who have committed to Pixfizz and want to get a head start. Gathering these items before the kickoff call significantly reduces onboarding time.

### Required for all deployment paths

**Brand assets:**
- Logo in SVG or high-resolution PNG (minimum 1000px wide)
- Brand colours as hex codes (primary, secondary, accent)
- Brand fonts — if you use custom fonts, provide the font files (OTF or TTF) or the Google Fonts name
- Brand guidelines document if you have one (not required, but helpful)

**Product information:**
- Complete product list — every product you want to sell, with names, descriptions, and categories
- Pricing for each product, including any variant-based pricing (e.g. different sizes at different prices)
- Variant structure for each product: sizes, finishes, paper types, quantity tiers — whatever options customers choose from
- Product images — at least one hero image per product. Lifestyle or in-context photography is ideal but not required at launch

**Production specifications:**
- File format requirements from your lab or print supplier (PDF, JPEG, TIFF, colour profile)
- Bleed and trim specifications per product (your print supplier will know these)
- Any special production requirements (e.g. spine width calculations for books, packaging inserts)

### Required for Full Pixfizz (Shopper or Custom)

Everything above, plus:
- **Domain name** — the domain your store will live on (e.g. shop.yourbusiness.com). You'll need DNS access to point it to Pixfizz.
- **Payment gateway account** — Stripe is the most common. Have your Stripe account created and verified before kickoff. Pixfizz will connect it during onboarding.
- **Shipping rate structure** — flat rate, weight-based, zone-based, or free shipping. Include any free shipping thresholds. If you offer in-store collection, note the pickup location(s).
- **Tax registration** — VAT/sales tax registration details if applicable.
- **Navigation structure** — a rough outline of how you want your site organized. Categories, subcategories, service pages. A simple list or sketch is fine.
- **Content** — About Us text, contact information, FAQs, terms and conditions, privacy policy. These can be drafted during onboarding but having them ready saves time.

### Required for Shopify + Pixfizz

Everything in the "all deployment paths" section, plus:
- **Shopify store** — your Shopify store must be on at least the Basic Shopify plan
- **Admin access** — Pixfizz needs collaborator access to your Shopify admin
- **Existing products** — if you already have products in Shopify, provide a mapping of which ones should have personalization enabled

### Required for Custom API Integration

Everything in the "all deployment paths" section, plus:
- **Developer availability** — this path requires a developer on your side who can integrate with the Pixfizz API. They should be available for the full onboarding period.
- **Platform details** — what eCommerce platform you're running, what language/framework, and how orders currently flow
- **User identity approach** — how your users log in and how you'll pass their identity to Pixfizz (see `61_PIXFIZZ_API.md` § 7 for the user handoff mechanism)
- **Domain setup** — Pixfizz needs a subdomain on your domain (e.g. `design.yoursite.com`) pointing to Pixfizz hosting. This is required for the design tool to work within your site's domain.

### Required for Marketplace (Etsy, etc.)

Everything in the "all deployment paths" section, plus:
- **Active marketplace store** — your Etsy (or other marketplace) store must be set up and active
- **Listing structure** — how your personalized products are listed on the marketplace
- **Order notification setup** — how you'll receive orders from the marketplace (email, API, CSV export)

### Things you do NOT need to have ready

- You don't need final product photography — we can use placeholder images during setup and replace them before launch
- You don't need every product priced to the penny — pricing can be refined during Phase 2, but the structure (flat, tiered, volume-based) should be decided
- You don't need your domain pointed to Pixfizz yet — that happens near the end of onboarding
- You don't need to know how Pixfizz works — that's what onboarding is for

---

## Pre-Onboarding (Pixfizz Internal — Before Kickoff Call)

Before the customer kickoff call, Pixfizz handles these internally:
- Organisation created in myPixfizz (contacts, billing, subscription)
- Pixfizz environment provisioned
- Asset storage and SFTP access configured (if needed)
- Base product templates prepared based on customer's product list
- Production pipeline validated end-to-end

Do not schedule the kickoff call until these are confirmed complete.

---

## Full Pixfizz (Shopper) — Onboarding Phases

A typical Shopper onboarding runs 25–40 tasks across 5 phases, managed in myPixfizz.

### Phase 1: Store Foundations

_Pixfizz completes internally; customer provides brand assets._

- Select and configure Shopper theme
- Upload logo and configure brand colours in style snippets
- Set up custom domain and SSL
- Build primary navigation structure
- Configure homepage (sections, hero, featured products)
- Set announcement bar, footer content, contact information

**Blockers:** Logo, colours, domain/DNS access, and a rough navigation structure from customer.

#### Custom domain and SSL sequence

Domain setup is a sequence with waiting built into it, not a single step to be
left until launch week. Start it early in Phase 1.

1. Customer creates a CNAME for the storefront domain or subdomain pointing at
   `hosting.pixfizz.com`.
2. Register the domain in Pixfizz admin under **Settings → General → Domain
   Hosting**.
3. Wait for DNS propagation. Allow up to **48 hours**. Tell the customer not to
   change DNS records again while propagation is in flight, since further edits
   restart the wait and make the problem much harder to diagnose.
4. Once the domain resolves to Pixfizz, **request the SSL certificate**. SSL is
   not provisioned automatically when DNS changes, it is requested after DNS is
   confirmed correct.
5. Certificate issuance takes roughly **40 minutes** from the request.

If propagation is still failing well past 48 hours, the problem is normally at
the registrar or the previous host rather than at Pixfizz. Have the customer
raise it with whoever manages the domain.

The same sequence applies to every additional domain on a site, including kiosk
domains and the personalization subdomain on a Shopify plus Pixfizz build.

### Phase 2: Product Setup

_Longest phase. Can run in parallel with Phase 1 if assets are ready._

- Define product categories and collection structure
- Create product templates (XML) for each product type
- Create design templates and configure personalization options
- Set up product variants (size, finish, quantity tiers, etc.)
- Configure pricing rules per product (Ruby pricing formulas)
- Upload product imagery and preview designs
- For stores with large static product catalogs (standard print sizes, fixed products without personalization), use the bulk static product CSV importer at **Custom Admin → manage/tools/product-importer**. Download the CSV template directly from that page.

**Blockers:** Complete product catalog, pricing structure, and product images from customer. Pricing must be fully confirmed before launch — changes after go-live cause friction.

### Phase 3: Checkout Configuration

- Connect payment gateway (Stripe typical)
- Configure shipping rules and rates
- Set up tax handling (VAT, sales tax)
- Configure order confirmation and notification emails (14 templates available — see Email Notifications section below)
- Set minimum order amounts if required

**Email delivery:** Shopper email notifications are sent via SendGrid. Deliverability problems are almost always caused by missing or misconfigured SPF/DKIM DNS records on the customer's sending domain. Verify email authentication DNS records during this phase and send test notifications before launch.

**Blockers:** Payment gateway credentials, shipping rate structure, tax registration info.

### Phase 4: Production Setup

- Verify artwork generation for each product template
- Test production file output and confirm format matches lab/production requirements
- Confirm OrderHub job creation per product type
- Validate production file downloads
- Configure fulfillment template (job ticket) if sending to external lab

**Blockers:** Production specs from the lab or print supplier. Must know file format, colour profile, bleed/trim requirements per product before this phase can complete.

### Phase 5: Launch Readiness

- Place internal test orders across all product types
- Validate full order lifecycle end-to-end (order → payment → production → fulfillment)
- Confirm payment capture and production routing
- Verify all email notifications fire correctly
- Content completeness check — all products and pages have descriptions (see Content Completeness section below)
- SEO setup — redirects configured if migrating from an existing site (see SEO Migration section below)
- Soft launch (restricted access or internal only)
- Fix any issues found in soft launch
- Full launch

**Do not skip internal test orders.** Issues found here are far cheaper than issues found by live customers.

---

## Full Pixfizz (Custom) — Onboarding Phases

Custom template builds follow the same phase structure as Shopper but with additional time and scope in Phase 1.

### Phase 1: Store Design & Build

_Significantly longer than Shopper Phase 1. Requires design approval cycles._

- Design review and approval (wireframes, mockups, or reference sites)
- Custom template development (layouts, snippets, CSS, navigation)
- Homepage and key page builds
- Responsive testing across devices
- Custom admin configuration (if applicable)

**Blockers:** Design approval from customer. This phase cannot proceed without signed-off designs. Budget 2–4 weeks for design iteration on top of the build time.

### Phases 2–5

Same as Shopper (Product Setup, Checkout Configuration, Production Setup, Launch Readiness).

---

## Shopify + Pixfizz — Onboarding Phases

### Phase 1: Shopify Store Access

- Customer grants Pixfizz collaborator admin access
- Confirm Shopify plan and theme compatibility
- Review existing product catalog structure
- Identify which products will have personalization

### Phase 2: Pixfizz Integration Setup

- Install Pixfizz Shopify integration code (snippets, scripts, metafields)
- Generate API credentials and connect systems
- Configure webhooks for order sync
- Set up the Pixfizz subdomain on the customer's domain

### Phase 3: Product Synchronization

- Map Shopify products to Pixfizz templates via product metafields
- Configure personalization options per product
- Map variants between Shopify and Pixfizz
- Validate pricing alignment between platforms
- Test the "Personalize" button and design tool launch flow

### Phase 4: Order Flow Validation

- Validate cart integration and personalization data transfer
- Verify orders flow correctly from Shopify to Pixfizz on payment
- Confirm order line item data is complete for production routing
- Test the full checkout flow end-to-end

### Phase 5: Production Verification

- Place test orders through Shopify
- Confirm artwork generation and OrderHub job creation
- Validate production routing and file output for each product type

### Phase 6: Launch

- Validate full personalization and checkout UX on the live theme
- Confirm production output quality
- Enable products for public purchase

---

## Custom API Integration — Onboarding Phases

This path requires developer resources on the customer's side. Pixfizz provides the API, documentation, and support; the customer's developer builds the integration.

### Phase 1: Environment Setup

- Pixfizz environment provisioned
- Customer's subdomain configured (e.g. `design.customersite.com` → `hosting.pixfizz.com`)
- External host authorized for CORS in Pixfizz admin
- API credentials generated and shared securely
- Customer's developer reviews `61_PIXFIZZ_API.md`

### Phase 2: User Handoff Integration

- Implement the user handoff endpoint (POST `/v1/users/_uid/...`) — see `61_PIXFIZZ_API.md` § 7
- Generate the security hash server-side (never expose the shared secret to the browser)
- Call user handoff on every page load where the user is logged in, and before any Pixfizz API interaction
- Test: verify Pixfizz user is created and logged in when handoff is called

**Important:** Users created via the `_uid` handoff endpoint are classified as **external users** — they log in via the customer's platform, not directly into Pixfizz. Do not use the `_uid` endpoint to create users who need direct Pixfizz login access (e.g. admin users or OrderHub operators). Those users must be created via the standard `/v1/users` endpoint.

**Blocker:** This must work before any other integration step. If user handoff fails, nothing else will work.

### Phase 3: Product & Template Setup

- Create product templates (XML) and design templates in Pixfizz
- Configure personalization options and pricing
- Map customer's product IDs to Pixfizz product/template codes
- Test project creation via the API (POST `/v1/books.json`)

### Phase 4: Design Tool Integration

- Implement the design tool launch flow: create project → redirect to editor → handle cart callback
- Configure the `cart_target` URL to receive the completed project ID
- Test the full flow: launch editor → personalize → add to cart on customer's platform
- Test re-editing: reopen a saved project in the editor

### Phase 5: Order & Fulfillment Integration

- Implement order creation via the API (POST `/v1/admin/orders/_external/...`) or
  configure webhook-based order sync
- Test: place an order on the customer's platform → verify it appears in Pixfizz → verify artwork generation → verify production file output
- Configure fulfillment template if needed

### Phase 6: Launch

- End-to-end test of the full flow: browse → personalize → cart → checkout → order → production
- Validate production output quality
- Go live

**Note:** Custom API integrations typically take longer than Shopper or Shopify paths because of the development work on the customer's side. Budget 4–8 weeks depending on the customer's developer availability and platform complexity.

---

## Marketplace (Etsy, etc.) — Onboarding Phases

Marketplace integrations are the simplest from a storefront perspective — the marketplace handles the customer-facing side. Pixfizz handles personalization and production.

### Phase 1: Workflow Design

- Define how customers will submit personalization requests (e.g. upload photos via Etsy message, use a linked Pixfizz design tool page, provide details in order notes)
- Decide whether the design tool will be linked from marketplace listings or managed via a separate Pixfizz page
- Set up the Pixfizz environment and product templates

### Phase 2: Product & Template Setup

- Create product templates matching the marketplace listings
- Configure personalization options
- Set up pricing (the marketplace handles payment; Pixfizz pricing is for internal cost tracking and production routing)

### Phase 3: Order Workflow Setup

- Define how marketplace orders enter Pixfizz:
  - **Manual entry:** operator creates orders in Pixfizz admin from marketplace notifications
  - **CSV import:** batch import orders from marketplace export files
  - **API automation:** use the external order API to push marketplace orders into Pixfizz programmatically (requires development)
- Configure production routing and fulfillment

### Phase 4: Production Verification

- Place test orders through the marketplace workflow
- Verify artwork generation and production file output
- Test the fulfillment and shipping notification flow

### Phase 5: Launch

- Validate the full workflow end-to-end
- Update marketplace listings with personalization instructions or design tool links
- Go live

**Note:** Marketplace integrations vary significantly depending on the marketplace's API capabilities. Etsy's API supports order retrieval but not real-time webhooks for all events, so some manual steps may be required.

---

## Common Blockers by Phase

These are the most common reasons a phase stalls. Flagging them early saves weeks.

| Phase | Typical blocker |
|-------|-----------------|
| Phase 1 | Customer slow to provide logo / brand assets |
| Phase 1 | Domain/DNS access not available (IT department delays) |
| Phase 2 | Product catalog incomplete or pricing not finalized |
| Phase 2 | Customer hasn't decided on variant structure |
| Phase 3 | Payment gateway credentials missing or gateway not yet set up |
| Phase 3 | Shipping rate structure not defined |
| Phase 4 | Production specs not confirmed with lab/supplier |
| Phase 4 | Lab requires a specific file format Pixfizz doesn't output by default (confirm early) |
| Phase 5 | Customer unavailable to approve test orders |
| Phase 5 | Missing content — product descriptions, about page, terms and conditions |

---

## Vertical-Specific Notes

### Photo Labs

- **Kiosk mode:** If the lab needs an in-store kiosk experience, plan for this in Phase 1 — it affects storefront structure. Basic kiosk mode uses a dedicated alternate domain (separate CNAME pointing to `hosting.pixfizz.com` with SSL). The kiosk domain is registered under **Settings → General → Domain Hosting** in admin. Checklist keys to set: `kiosk-mode-enabled: TRUE`, `kiosk-mode-domain: <kiosk-domain>`, `kiosk-pay-in-store-only: TRUE` (restricts pay-in-store to kiosk sessions only). Pay-in-store auto-confirmation is configured separately in admin. If the kiosk offers products at different pricing than the web store, unpublish those collections from the public storefront or access-gate them.
- Establish whether the lab offers in-store collection (affects checkout/shipping config)
- **Film processing:** If offered, needs separate product template setup. 120 and 220 film formats have different frame counts and scan rates — set them up as separate products, not as variants of the same product.
- **Same-day collection:** Same-day or next-day collection messaging is a strong commercial differentiator — plan for it in Phase 1. A JavaScript-based time cutoff (typically after 1pm Mon–Fri) can be configured in Shopper to switch checkout messaging once the same-day window has passed. This is implemented at the snippet level.
- **OrderHub Desktop (OHD):** OHD is a single-location install only. Installing it at multiple workstations in the same location without proper single-instance setup causes order conflicts. Verify Mac compatibility against the current OHD release before recommending it.
- Photo labs often have existing customers who expect a specific workflow — discuss migration messaging

### School / Sports Photography

- Access code or class lookup flow must be planned before navigation is built (Phase 1)
- Package tiers (print bundles vs single prints) must be fully defined before Phase 2
- Seasonal campaigns mean launch timing is critical — delay past peak season has real revenue impact
- High-volume gallery management is a core requirement — plan gallery structure early

### Photo Gifting Brands

- Occasion-based navigation structure should be confirmed in Phase 1 before building nav
- Gift wrapping and delivery date options add complexity to Phase 3 — flag early
- Shopify + Pixfizz is the most common path for gifting brands

### Print-on-Demand / Trade

- Fulfillment routing is the core complexity — multiple production partners, each with different file format and delivery requirements
- Pricing formulas often need wholesale/retail logic — confirm pricing model in Phase 2
- Template auto-import may be relevant if the catalog is large

---

## Content Completeness Before Launch

Before going live, verify that all products and pages have descriptions populated. Missing descriptions are a common missed step during setup.

Missing descriptions cause:
- **Sitemap quality degradation** — blank description entries reduce the usefulness of the XML sitemap submitted to search engines
- **Product feed issues** — Google Shopping and Meta catalogue feeds require descriptions; missing ones cause feed errors or rejected products
- **Google Search Console warnings** — structured data validation flags missing or empty description fields
- **Crawler / feed generation errors** - when the built-in crawler runs (Admin > Website Crawls), products or pages with missing descriptions can throw JSON or crawl errors that interrupt `sitemap.xml` / `product-feed.json` generation. The fix is to populate the missing descriptions in admin. Seen across multiple client sites.

Add a description check to the final pre-launch validation step for all deployment paths. Setting the crawler to run automatically on a daily schedule catches newly added products with blank descriptions before they cause feed errors.

---

## SEO Migration: Sitemap and 301 Redirects

When migrating a customer from an existing website to Pixfizz, SEO continuity requires three things:

1. **URL mapping** — the customer provides a spreadsheet of their current live URLs, categorized by content type (products, pages, blog posts). Map each to the corresponding new Pixfizz URL structure.
2. **301 redirects** — configure redirects in Pixfizz using the JSON config format. The config is an **array of pairs** (array of arrays), even for a single redirect: `[["^/old-path/?$", "/new-path"]]`. Each inner pair is `[regex_pattern, destination]`. A bare single pair (`["^/old-path/?$", "/new-path"]`) **silently fails** with no error and no redirect, so always wrap pairs in the outer array. On sites that use the `/site/` prefix it is included in the pattern. This preserves Google rankings and prevents 404s for bookmarked or indexed URLs.
3. **Sitemap submission** — once the new site is live, submit the new sitemap to Google Search Console. Monitor indexing for the first 2–4 weeks to catch any missed redirects.

System paths (cart, checkout, account, order-confirmation) do not need redirects — they are handled by the platform.

Start the URL mapping exercise early in the onboarding process. It is the customer's responsibility to provide the existing URL list, but Pixfizz should prompt for it and explain why it matters.

---

## Pre-Launch Handoff Checklist

Use this checklist before declaring a site ready for launch.

- [ ] All product types have been test-ordered internally
- [ ] Payment capture confirmed (real test transaction, not test mode only)
- [ ] Production files generated and approved for each product type
- [ ] All email notification templates reviewed — active ones customized for customer brand
- [ ] Email delivery verified — SPF/DKIM DNS records confirmed for SendGrid deliverability
- [ ] Custom domain live with SSL confirmed
- [ ] Navigation and homepage signed off by customer
- [ ] Pricing confirmed and validated against customer's agreed rate card
- [ ] Shipping rules tested with real address(es)
- [ ] All products and pages have descriptions populated
- [ ] 301 redirects configured and tested (if migrating from existing site)
- [ ] Sitemap submitted to Google Search Console (if applicable)
- [ ] Kiosk mode tested on intended hardware (if applicable)
- [ ] Mobile experience reviewed on actual devices
- [ ] Legal pages present (terms, privacy policy, returns policy)

---

## Email Notification Templates

Pixfizz includes 14 email templates mapped to the order lifecycle. Configured in Admin → Settings → Email Notifications. Each can be enabled or disabled individually.

**Order lifecycle emails:**
- Order Pending
- Order Draft
- Order Confirmed
- Order Downloaded
- Order Manufactured
- Order Shipped
- Order Fulfilled
- Order Error
- Order Canceled
- Orderline Fulfilled

**Other emails:**
- Cart Abandoned
- User Signup
- Password Reset
- Sign-in Token Renewed

Review and customize all active templates before launch — default content references generic Pixfizz copy that should be replaced with the customer's brand voice.

---

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-23: Added content completeness (descriptions) pre-launch checklist item.
- 2026-04-27: Added SEO migration workflow (sitemap + 301 redirects for domain moves).
- 2026-06-26: Corrected 301 redirect config to outer-array format (bare single pair fails silently). Added crawler/feed JSON-error failure mode for missing product descriptions and daily auto-crawl note. Source: claude-chat/fireflies-call.
- 2026-05-21: Major rewrite. Added all deployment paths (Custom API, Marketplace/Etsy). Added "Preparing for Onboarding" customer preparation section. Added Full Pixfizz Custom path. Expanded phase sequences with blockers. Merged content from onboarding skill. Added pre-launch handoff checklist. Added vertical-specific notes.
- 2026-05-27: Photo Labs vertical notes: added kiosk mode setup procedure (CNAME, checklist keys, pay-in-store config), OHD single-location install rule, film 120/220 as separate products, same-day JS cutoff pattern. Phase 2: added static product CSV importer note (manage/tools/product-importer). Phase 3: added SendGrid deliverability and DNS authentication note. Pre-launch checklist: added email delivery DNS check. Custom API Phase 2: added external user warning (_uid creates non-login users; OrderHub operators must use /v1/users). Source: Fireflies calls, Slack #dev, support tickets.
- 2026-08-05: Added the custom domain and SSL sequence to Phase 1 (CNAME to hosting.pixfizz.com, register under Settings > General > Domain Hosting, up to 48 hours propagation, SSL requested manually after DNS confirms, roughly 40 minutes to issue). Confirms SSL is not auto-provisioned. Source: fireflies-call.


=================================================================
FILE: 81_SEO_AND_GEO_REFERENCE.md
=================================================================

# 81 — SEO and GEO Reference

**Authority Scope:** How AI-powered search works, Generative Engine Optimization (GEO) practice, and what the Pixfizz platform and Shopper template currently do for AI search visibility. Industry concepts are general knowledge. Platform and template capabilities are tagged explicitly.

_Last updated: 2026-06-18_

---

## Purpose

Reference for advising customers and staff on search visibility in the AI era. Covers three things:

1. How AI search works and how it differs from traditional keyword search (general).
2. The GEO playbook — how to get cited by AI answer engines (general).
3. What Pixfizz and Shopper actually provide for this today, and the current gaps (platform truth).

For onboarding-time SEO setup (URL mapping, 301 redirects, sitemap submission, pre-launch description checks) see `80_ONBOARDING.md` § SEO Migration and § Content Completeness Before Launch. This file does not repeat that material.

**Layer tags used below:** `[PLATFORM]` = always true of Pixfizz CMS. `[SHOPPER]` = Shopper template behavior, may not apply to Custom or Shopify paths. `[GENERAL]` = industry knowledge, not Pixfizz-specific. `[GAP]` = not currently provided. `[CONFIRM]` = needs Matjaz/team confirmation.

**On statistics:** Figures in this file are directional and drawn from third-party 2025–2026 research. The precise numbers vary widely between sources. Verify any specific figure against its named source before using it in a customer-facing context (webinar, slide, email).

---

## Part A — How AI Search Works (General)

### The shift from links to answers

Search has moved from returning a list of ten links to returning a single AI-written answer at the top of the page. The AI does the reading and shortlisting for the customer. Being seen no longer means being clicked.

- **AI Overviews** are AI-generated summary blocks at the top of Google results, condensing multiple sources into one answer with citation links.
- **AI Mode** is Google's conversational search experience, where users ask complex, follow-up questions and can input images or voice without leaving Google.
- Third-party answer engines that matter: ChatGPT, Perplexity, Gemini, Claude, Copilot.

### How an answer is assembled

- **Query fan-out:** the engine breaks one customer question into multiple background sub-queries and searches each separately. (e.g. "best paper for wedding photos" fans out into "archival photo paper," "matte vs glossy," "local high-resolution photo labs").
- **Synthesis (RAG):** it retrieves passages from many sources and stitches them into one custom answer. It is not matching your keyword, it is assembling a recommendation.
- Implication: you are no longer competing for one ranking, you are competing to be one of the trusted sources the answer is built from.

### Ranking vs being cited (important nuance)

The old goal was to rank in the list of links. The new goal is to be cited or recommended inside the AI answer. The new metric is **citation rate** (how often AI names you), not just rank. But the relationship to classic ranking differs by engine:

- **Google AI Overviews still lean heavily on top-ranked pages.** The majority of URLs cited in AI Overviews also rank in Google's top 10 (Ahrefs, 2025). Classic SEO still feeds Google's AI answers.
- **ChatGPT and Perplexity play by different rules.** A large share of their citations come from pages that do not rank in Google's top 10 (Ahrefs, 2025).
- Correct framing: strong SEO fundamentals plus AI-specific signals. Not one or the other. The blanket claim "ranking number one no longer matters" is overstated for Google and only broadly true for the chat engines.

### Scale (directional, verify before quoting)

- AI Overviews now appear on roughly a quarter of US searches, up from roughly half that a year earlier (Conductor, 2026).
- With an AI Overview present, click-through to the top result drops sharply (roughly 8% vs 15% without, Pew Research, 2025).
- Zero-click: more than 68% of US Google searches ended without a click in early 2026 (Jan–April), up from about 60% in 2024 (Adsroid, June 2026, citing market-intelligence panel data). Note this counts clicks to Google's own properties (Maps, YouTube) as non-clicks. The older "58.5%" SparkToro figure is 2024 and now outdated; use the 68% early-2026 figure, framed as "reported."
- Bots now generate the majority of web traffic: 57.5% of HTTP requests versus 42.5% human, the first machine majority, driven mainly by agentic AI (Cloudflare Radar, Matthew Prince, June 3, 2026). The through-line is "crawl without credit" — bots crawl heavily but send little referral traffic. Strong wake-up statistic. Caveat: do not pair it with the "ChatGPT-User/ClaudeBot crawl 3.6x more than Googlebot" claim — Cloudflare's own data still has Googlebot as the largest crawler by reach.
- AI-referred visitors tend to convert better than standard organic visitors because they arrive pre-qualified by the AI's recommendation. Reported multipliers vary too widely across sources to state a single figure; if a number is used, attribute it (e.g. Seer/Semrush) and frame as "reported," and do not show multiple conflicting figures together.

---

## Part B — Glossary (General)

### AEO (Answer Engine Optimization)
Structuring content so AI answer platforms select and cite it. Focuses on clear, extractable answers rather than keyword rankings. Often used interchangeably with GEO and AEO/LLMO.

### AI Mode
Google's conversational search experience powered by Gemini, allowing complex questions and follow-ups in the search interface, assembling custom answers from multiple background searches.

### AI Overviews
AI-generated summary blocks at the top of Google results that condense multiple sources into one answer with citation links.

### Citation rate
How frequently an AI engine references or links a specific brand as a source. Because AI pulls from different sources than classic search, tracking citation rate across platforms is the core AI-visibility metric.

### E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness)
Credibility signals Google and AI systems use to judge whether a source is safe to recommend. AI leans on external citations, established reputation, and expert authorship, especially for sensitive topics.

### GEO (Generative Engine Optimization)
The broad practice of structuring content and online presence so LLM-based engines cite or recommend you. Encompasses statistics, quotations, clear formatting, structured data, and authority signals.

### GPTBot
OpenAI's web crawler that gathers content for its models. Frequently blocked by publishers via robots.txt. For a business that wants to be recommended, it should be allowed, not blocked.

### llms.txt
A markdown file in the site root that describes site structure to AI agents. `[GENERAL]` Google has stated it does **not** directly affect visibility in AI search features. Treat as optional and low priority, not a required tactic.

### LLMO (Large Language Model Optimization)
Synonym for GEO. Managing content so LLMs understand and reference your brand.

### OKF (Open Knowledge Format)
`[GENERAL]` An open spec from Google Cloud (v0.1, published June 12, 2026) for representing organizational knowledge as a directory of markdown files with YAML frontmatter, one concept per file, linked as a graph. **Important: OKF is NOT a web-visibility or SEO tactic.** It is an internal knowledge bundle that your own AI agents read, not a public page format you publish for the web to crawl, and Google has stated plainly it is not a ranking or visibility mechanism. Do not present OKF to customers as a way to get cited by AI search. It is, however, relevant to Pixfizz internally — our own knowledge repo is essentially this pattern.

### Query fan-out
An engine breaking one prompt into multiple simultaneous sub-queries, then synthesizing the retrieved passages into one answer.

### SEO (Search Engine Optimization)
Traditional practice of optimizing to rank higher in standard results via on-page optimization, backlinks, and keywords. Wins clicks from a list of links rather than being the generated answer.

### Structured data / schema
Machine-readable markup (e.g. JSON-LD) that tells engines your entities explicitly (product name, price, availability, brand, business identity). Not strictly required to appear in AI search, but it helps AI extract facts accurately and keeps you eligible for rich results.

### WebMCP (Web Model Context Protocol)
`[GENERAL]` An emerging browser standard co-developed by Google and Microsoft via the W3C, first announced February 10, 2026, and in a public origin trial in Chrome as of June 2026. It lets a website expose its features to AI agents as structured, callable tools (a "Tool Contract") rather than agents guessing via screenshots. Two paths: a Declarative API (HTML attributes on forms) and an Imperative API (JavaScript). Model-agnostic. The "action layer" of AI readiness: schema gets a product understood, GEO gets a brand cited, WebMCP gets a product acted on and purchased by an agent. Status: early and Chrome-only; treat as "watch and early-adopt," not a do-it-now requirement.

### Zero-click search
When a user gets a complete answer on the results page without clicking through. Dominant in AI search, which is why citations and brand mentions matter more than raw click traffic.

---

## Part C — The GEO Playbook: The Five Signals (General)

A memorable framing for advising customers: AI looks for five signals. Be crawlable, be structured, be quotable, be trusted, be present locally.

### Signal 1 — Be crawlable
- If AI bots cannot read your site, you cannot be cited.
- Keep standard, clean HTML. Do not block AI crawlers (GPTBot, ClaudeBot, etc.) in robots.txt if you want to be recommended.
- Fast-loading, error-free pages are retrieved more reliably.

### Signal 2 — Be structured
- Schema gives AI clean facts without it having to interpret your layout: product name, price, availability, brand.
- Deploy where supported: Product, FAQPage, Article, LocalBusiness JSON-LD. Ensure structured data matches the visible text.
- Every product and page needs a populated description. Blank descriptions degrade feeds and trip structured-data validation.

### Signal 3 — Be quotable
- Open each section with a direct 40–60 word answer to a real question before the supporting detail. AI pulls heavily from the top of a page.
- Use FAQ-style question headings with short, factual answers that mirror how people prompt AI.
- Add specific numbers and cite sources. The original Princeton GEO research (KDD '24) found that adding statistics, citing sources, and adding quotations measurably lift AI visibility, while keyword stuffing performs worse than baseline (roughly 10% worse).
- Write for a human reading the answer aloud, not for an algorithm.

### Signal 4 — Be trusted
- AI treats reviews as a gate, not just a ranking factor. Locations ChatGPT recommends average above four stars (around 4.3, SOCi 2026); very low review response rates correlate with being effectively invisible to AI recommendations.
- Ask customers to describe the specific problem you solved, not just leave a star rating. AI lifts specific phrases from review text to justify a recommendation. Respond to reviews.
- Keep Name, Address, Phone (NAP) identical across your site, Google, and directories. Inconsistency erodes the AI's confidence to recommend you.
- Third-party mentions and earned media measurably lift citations. A meaningful share of AI citations come from sources other than your own site.

### Signal 5 — Be present locally
- Treat the Google Business Profile (GBP) as a data feed for AI. Fill every field, use the full business description, list every service and attribute.
- Add fresh photos and posts regularly. Stale profiles lose visibility.
- Never keyword-stuff the business name; it gets penalized and flagged as untrustworthy.

### Local nuance worth knowing
- Local-intent queries trigger an AI Overview far less often than general queries (around 8%, Ahrefs 2025). For local, the AI Overview on the results page is a smaller threat than whether you appear in the AI's *recommendation* at all.
- AI is currently far stingier than the classic map pack: ChatGPT names only ~1.2% of local business locations vs ~36% surfaced in Google's local 3-pack (SOCi 2026). For local discovery, a complete GBP and the map pack still do most of the heavy lifting; the chat engines and Google Ask Maps are the emerging battleground.

---

## Part D — Pixfizz and Shopper: What's Provided Today

This is the platform-truth section. Map each signal to what the platform and template actually do, and flag the gaps.

### Crawlable
- `[PLATFORM]` Built-in crawler at **Admin → Website Crawls** (`/admin/website_crawls`) generates `sitemap.xml` and `product-feed.json`. The `/site/sitemap` path does not exist; do not assume it. The crawler can be set to run automatically every 24 hours; once existing crawl errors are cleared, leaving it on a daily schedule keeps SEO health maintained without manual re-runs.
- `[SHOPPER]` `no-index` checklist key set to `TRUE` adds a site-wide noindex. Confirm it is OFF on a live store.
- `[SHOPPER]` **Key-name mismatch in the v2 manage admin.** The manage SEO page writes the hide-from-search toggle to `admin/checklist/launch-no-index`, but `html.head` reads `admin/checklist/no-index`. The toggle therefore appears to work in admin and silently does nothing on the storefront, so a pre-launch site can be indexed while the operator believes it is hidden. Until the template is corrected, verify the noindex is actually present in the rendered page source rather than trusting the admin toggle, or set `admin/checklist/no-index` directly. Source: claude-chat (shopper24 admin audit).
- `[SHOPPER]` **Staging / pre-launch pages can get indexed by Google.** If a site was live on a staging URL before launch, those staging pages may already be in Google's index and compete with production. Fix by redirecting the old staging URLs to their production equivalents. The Shopper redirect config file is a JSON **array of `[regex, destination]` pairs** (each entry is a two-element array: a path-matching regex and the destination URL); anchor patterns as `^/site/<path>/?$` to tolerate a trailing slash. See `80_ONBOARDING.md` § SEO Migration for where this file lives and how it is applied.
- For sitemap submission to Google Search Console and 301 redirect configuration during migration, see `80_ONBOARDING.md` § SEO Migration.

### Structured data
- `[PLATFORM]/[SHOPPER]` **Product schema** is supported. Enable site-wide via the `schema_loop_all_products` checklist key (`TRUE` = include all products in schema). See `50_SHOPPER_TEMPLATE_REFERENCE.md` § SEO & Metadata.
- `[SHOPPER]` **LocalBusiness JSON-LD: placeholders exist** in the Shopper template. They are present but should not be assumed populated or emitted correctly by default. `[CONFIRM]` whether they are active out of the box, whether they include geo-coordinates and `sameAs`, and what configuration is required.
- `[GAP]` **Review / AggregateRating schema: not currently emitted.** There is on-page review display (Google rating/review fields, `Google_Summary` custom field with `rating` and `review_count`) but no review structured-data output. This is a build opportunity (see Pending Confirmation).
- `[CONFIRM]` **FAQPage, HowTo, Article JSON-LD**: output status unconfirmed. Do not tell customers Pixfizz emits these until confirmed.
- `[SHOPPER]` Per-object SEO meta fields exist: `meta_title` and `meta_description` on products, collections, subcollections, custom pages, blog posts, and services (see `51_CUSTOM_FIELDS_REFERENCE.md`). Site-level title/description via `update-website-title`, `update-website-description`, and the `seo-tdks` checklist key.
- `[SHOPPER]`/`[GENERAL]` **High-fidelity product attributes**: the current Product schema is a sound baseline but emits only a fraction of what Google now recommends. See **Part F** for the current baseline, Google's required-vs-recommended set, and prioritized Shopper upgrades. A standalone **Schema Builder** tool (`pixfizz-schema-builder.html`) generates paste-ready JSON-LD for a site-wide Organization/policies block and for a full single Product block.

### Quotable content
- `[SHOPPER]` Content surfaces available: blog (`blog`, `custom-blog-page`, `custom-blog-post` keys), custom pages, and a custom FAQ page (`custom-faq-page` key). These are where answer-first and FAQ-style content lives.
- Note the distinction: a custom FAQ *page* exists as a content surface; whether FAQPage *schema* is emitted from it is `[CONFIRM]` / likely a `[GAP]`.
- `[PLATFORM]` Populated product and page descriptions are required for feed and structured-data quality. See `80_ONBOARDING.md` § Content Completeness Before Launch.

### Trust and local
- `[SHOPPER]` Review display exists via Google rating fields. Review *generation* programs and GBP optimization are operational/marketing tasks for the client, not platform features.
- Product feed for Google Shopping / Meta is produced by the built-in crawler (`product-feed.json`). Google Merchant Center connection is a client-side setup task.
- `[GENERAL]` **Serve the store on the primary domain, not a `shop.` subdomain.** Where a business runs its store on `shop.domain.com` while the main site is `domain.com`, consolidating the store onto the primary domain (`domain.com`) generally improves indexing, keeps analytics attribution accurate (traffic not split across hostnames), and keeps local/`LocalBusiness` schema consistent with the brand's main entity. Treat a `shop.` subdomain as a migration candidate, handled with 301 redirects from the subdomain URLs to their primary-domain equivalents.

### Quick capability summary

| Signal | Pixfizz/Shopper status |
|---|---|
| Crawlable (sitemap, product feed) | Provided `[PLATFORM]` |
| Product schema | Provided, enable via `schema_loop_all_products` `[SHOPPER]` |
| Per-object SEO meta fields | Provided `[SHOPPER]` |
| LocalBusiness schema | Placeholders exist, not assumed active `[SHOPPER]` `[CONFIRM]` |
| Review / AggregateRating schema | Not emitted `[GAP]` |
| Return policy / shipping schema | Not emitted; best authored at Organization level, see Part F `[GAP]` |
| Organization / Brand / `sameAs` schema | Status unconfirmed; Schema Builder can author it `[CONFIRM]` |
| Product attribute fidelity (material, specs, variants) | Baseline only; upgrade tiers in Part F `[SHOPPER]` |
| FAQPage / HowTo / Article schema | Unconfirmed `[CONFIRM]` |
| Blog / FAQ / custom page surfaces | Provided `[SHOPPER]` |
| 301 redirects, Search Console submission | Provided, see `80_ONBOARDING.md` |
| llms.txt | Not provided; low priority per Google `[GAP]` |

---

## Part E — Measurement (General)

- Shift the scoreboard from rank and clicks to **citation rate** and **brand mention volume** across ChatGPT, Perplexity, Gemini, and Google AI surfaces.
- **Baseline test (repeatable):** run your 10–20 most important buyer queries through ChatGPT, Perplexity, and Google AI Mode. Note where you appear, how you are described, and which sources are cited. Re-run periodically; citation behavior shifts quickly and varies wildly between engines.
- **What AI already knows about you:** prompt an engine directly with "Tell me what you know about [business name]" to see its current understanding of your services, area, and reputation.
- Filter analytics referral traffic by AI sources (chatgpt.com, perplexity.ai, etc.) for the click-through slice, but expect most AI value to be zero-click.

### Platform measurement tools (current state, 2026)

- **Google Search Console — Search Generative AI performance reports** (launched June 3, 2026): a dedicated view showing how often your URLs appear in AI Overviews and AI Mode (and generative Discover), broken down by page, country, device, and date. Limitations: impressions only, no click or query data, not in the API yet (UI/CSV export only), and rolling out to a subset of properties first (initially UK). Use it to measure AI visibility, not traffic. This is the first time AI-surface visibility is separable from standard organic.
- **Google Search Console — AI controls toggle** (effective from June 17, 2026): lets a site opt its content out of AI Overviews, AI Mode, and generative Discover. Opting out forfeits impressions and any clicks from those features. Google states it is not a ranking signal for normal organic results. Almost no one should use this; relevant only as a thing that exists.
- **Bing Webmaster Tools — AI Performance report** (launched February 2026): tracks citations, the number of times your content is used to build a Copilot or Bing AI answer. Arguably more useful than GSC's impressions-only view because it counts actual citations. Worth pairing, since serious GEO measurement is cross-engine, not Google-only.

---

## Part F — Product Schema: High-Fidelity Attributes (Reference)

"High-fidelity" schema means the richest, most complete, and verifiably accurate Product markup the catalogue can support, rather than the minimum that triggers a rich result. It matters more now because AI shopping surfaces extract specific attributes (materials, dimensions, ratings, return and shipping terms) to answer specific buyer questions, and because richer Product, Brand, and Organization markup raises entity-confidence signals over time, not just snippet appearance. The catch: markup emits whatever the catalogue contains, including invalid identifiers and stale prices. Fidelity means complete **and** correct. Emitting a malformed attribute is worse than omitting it. Source: Google Search Central, Product / merchant-listing / product-snippet structured-data docs (2026); supporting industry analysis (2026), all directional.

### What Shopper emits today (baseline) `[SHOPPER]`

The auto-generated product JSON-LD already covers the Google-required trio and a good recommended set:

- `@type` Product with `name`, `description`, `sku`, `mpn` (falls back to `sku`), and a conditional `gtin` (emitted only when present).
- `image` array: live-preview WebP plus a fallback image.
- `brand` (Brand), `category` (from `google_category`), and `product_type` (from a custom field).
- `offers` (Offer) with `price` (or `from_pricing`), `priceCurrency`, `url`, `itemCondition` (`NewCondition`), `priceValidUntil`, and an `availability` mapping (`InStock` / `OutOfStock` / `InStoreOnly` when shipping is unavailable / `OnlineOnly` when pickup is unavailable).

This meets Google's required fields for a merchant listing (`name`, `image`, and an `offer` with `price` + `priceCurrency`). The opportunity is in the recommended layer below.

### Google's required vs recommended set `[GENERAL]`

- **Required (merchant listing):** `name`, `image`, and `offers` with `price` and `priceCurrency`. Merchant listings require an `Offer` (not `AggregateOffer`).
- **Recommended on Product:** `aggregateRating`, `review`, `brand.name`, `description`, the most specific `gtin8|gtin12|gtin13|gtin14` plus `mpn`/`sku`, `color`, `material`, `pattern`, `size` (or `SizeSpecification`), `audience` (`PeopleAudience`: suggested gender/age), `hasCertification` (energy labels etc., mainly EU), `isVariantOf` / `inProductGroupWithID` (variants), and `subjectOf` (a `3DModel`).
- **Recommended on Offer:** `availability`, `itemCondition`, `url`, `hasMerchantReturnPolicy`, `shippingDetails` (`OfferShippingDetails`), and `priceValidUntil`. Complex pricing uses `priceSpecification` with `UnitPriceSpecification` — `referenceQuantity` for per-unit pricing, `priceType: StrikethroughPrice` for sale prices, and `validForMemberTier` for loyalty/member prices.
- **Hard rules:** structured data must be present in the server HTML (not added by JavaScript after load), must match what the shopper sees, and identifiers must be valid. Each offer on a page needs a `sku` or `gtin` that matches the corresponding `product-feed.json` entry.

### Upgrade priorities for Shopper

**Tier 1 — high value, broadly applicable**
- `[GAP]` **`aggregateRating` + `review`.** Not emitted today, though `Google_Summary` already holds `rating` and `review_count`. This is the single biggest visible win (star ratings) and the "be trusted" GEO signal in concrete form. Only publish ratings that are visible on the page.
- `[GAP]` **`hasMerchantReturnPolicy` + `shippingDetails`.** Google expanded merchant-listing eligibility specifically around these. Best authored once at the Organization level, with per-offer overrides only where a product differs. Required return fields: `applicableCountry` and `returnPolicyCategory` (and `merchantReturnDays` when the window is finite).
- `[SHOPPER]` **`material`, `color`, `pattern`, `size`.** Cheap, factual attributes that AI extracts to match queries; relevant to apparel, gifting, and print verticals.

**Tier 2 — fit for the Pixfizz product model**
- `[GENERAL]`/`[SHOPPER]` **Product variants** (`ProductGroup` / `isVariantOf` / `inProductGroupWithID`) map onto Pixfizz option types, but this is a real design decision given personalization (see cautions).
- `[GENERAL]`/`[SHOPPER]` **`UnitPriceSpecification` + `referenceQuantity`** is the correct home for per-unit / tiered print pricing, rather than a single flat `price`.
- `[SHOPPER]` **`additionalProperty` (`PropertyValue`)** is the catch-all for specs with no dedicated field (paper stock, GSM, finish, page count, turnaround). This is where AI answers find the specifics.

**Tier 3 — entity grounding and GEO**
- `[CONFIRM]` **Organization + Brand + `sameAs`** ground the brand as a known entity; pairs with the LocalBusiness work already flagged.
- `[GAP]` **FAQPage** on product and Q&A pages: the "be quotable" signal in schema form.

### Pixfizz-specific cautions

- `[CONFIRM]` **Server-side rendering is mandatory for merchant listings.** Confirm the Shopper schema snippet renders in the initial server HTML, not via JavaScript after load. The same constraint forbids per-customer page changes (e.g. IP-based pricing).
- `[PLATFORM]` **Match visible content and the feed.** Schema values must match what the shopper sees, and each offer's `sku`/`gtin` must match `product-feed.json`.
- `[SHOPPER]` **GTIN validity.** Keep the conditional emission; an invalid GTIN actively hurts, and personalized/bespoke items legitimately have none. Decide how `identifier_exists`-style logic should behave for items with no GTIN.
- `[SHOPPER]` **`from_pricing` under-represents ranges.** A single `price` from a "from" value reads as a fixed price. For option-driven price ranges, decide deliberately between variant markup, `UnitPriceSpecification`, or an explicit starting-price framing. Note `AggregateOffer` is not for variants.
- `[GENERAL]` **Loyalty alignment.** The Offer pricing model now supports member pricing (`validForMemberTier`) and loyalty-program markup. Given the loyalty program on the roadmap, design the pricing schema so it does not need reworking later.

### Two correctness flags in the current snippet `[SHOPPER]`

- **`priceValidUntil` is computed inside a `{% dynamic %}` block.** Given the known platform behaviour where dynamic blocks can fail and render nothing, a failure here would emit `"priceValidUntil":` with no value and break the JSON for the entire product. Harden it or move the calculation out of the dynamic block.
- **`@context` uses `http://schema.org`.** Google accepts it, but `https://` is the current convention and matches Google's own examples.

### The Schema Builder tool

- A standalone HTML generator (`pixfizz-schema-builder.html`) produces paste-ready JSON-LD for (a) a site-wide **Organization + Brand + `sameAs` + return policy** block and (b) a full single **Product** block with the high-fidelity attributes above. It includes live validation hints and a copy button.
- Intended uses: author the Organization/policies block once per site (this is static data Shopper does not auto-build); test the target Product shape in the Rich Results Test; and hand-build product schema for the Shopify path or static products.
- It is an authoring aid, not a platform feature. The in-platform auto-generation of these formats (below) remains the larger opportunity.

---

## Pending Confirmation (for Matjaz / team)

- Shopper LocalBusiness JSON-LD: are the placeholders populated and emitted by default, and do they include geo-coordinates and `sameAs`? What configuration is required?
- FAQPage / HowTo / Article JSON-LD: does the platform or Shopper emit any of these today?
- Review / AggregateRating schema: confirmed not emitted; candidate build (auto-generate from existing review/`Google_Summary` data).
- llms.txt: low priority (Google states no direct AI-search visibility impact); decide whether to support at platform level as an agent-navigation aid only.
- Product schema rendering: does the Shopper product schema snippet render server-side (in the initial HTML), or could any of it depend on JavaScript? Merchant listings require server-side markup.
- `priceValidUntil` is built inside a `{% dynamic %}` block — confirm it is hardened so a silent dynamic-block failure cannot emit broken Product JSON.
- Return-policy and shipping values: where do they live (existing custom fields or new), and should they be Organization-level defaults or per-product overrides?
- Organization / Brand / `sameAs` schema: is any of this emitted today, or only the LocalBusiness placeholders?

## Knowledge Gaps / Build Opportunities (not yet built — do not present as live)

- Auto-generation of ideal structured-data formats (Product, LocalBusiness, Review/AggregateRating, FAQPage) from existing Shopper data, as a Shopper feature to improve AI visibility. Should extend to the high-fidelity Product layer in Part F (ratings, return/shipping policy, `additionalProperty`, variants, unit pricing). Under discussion; document only once live.
- Schema Builder tool (`pixfizz-schema-builder.html`): a built, standalone authoring aid that generates the Organization/policies block and a full Product block for paste and testing. Not a platform feature and not wired into Shopper; treat as an internal/merchant utility, distinct from the in-platform auto-generation opportunity above.
- WebMCP in Shopper: exposing storefront actions (search, configure product, add to cart) to browser AI agents via the Web Model Context Protocol. Emerging standard, Chrome origin trial only as of mid-2026. On the radar for Shopper; document only once built.

---

## Changelog

- 2026-06-15: Initial version. Created from webinar prep research (how AI search works, GEO playbook, glossary) and platform facts confirmed in-session: Shopper has LocalBusiness JSON-LD placeholders, review schema is not currently emitted, product schema via `schema_loop_all_products`. Cross-references 80 (SEO migration), 50 (checklist keys), 51 (custom fields). Source: claude-chat.
- 2026-06-18: Added verified 2026 stats (68% zero-click early 2026 per Adsroid; Cloudflare 57.5% bot-majority traffic, June 3 2026), with sourcing and caveats. Added glossary entries for WebMCP (real, emerging, Chrome origin trial) and OKF (real but internal-knowledge format, explicitly not an AI-visibility tactic — corrects an earlier framing). Added platform measurement tools: GSC Search Generative AI performance reports and AI controls toggle (June 2026), and Bing Webmaster Tools AI Performance/citation report (Feb 2026). Added WebMCP-in-Shopper to build opportunities. Source: claude-chat (web-verified).
- 2026-07-11: Part D — noted the built-in crawler can run on a 24-hour schedule to maintain SEO health; added staging/pre-launch indexing gotcha with the Shopper redirect-config format (JSON array of `[regex, destination]` pairs, anchored `^/site/<path>/?$`); added the primary-domain-vs-`shop.`-subdomain consolidation recommendation under Trust and local. Source: fireflies-call (2026-07-07, 2026-07-10), claude-chat (redirect JSON build).
- 2026-06-18: Added Part F (Product Schema: High-Fidelity Attributes), researched against Google Search Central's product / merchant-listing / product-snippet structured-data docs. Documents the current Shopper baseline, Google's required-vs-recommended set, prioritized Shopper upgrade tiers (Tier 1 ratings + return/shipping + attributes; Tier 2 variants, unit pricing, `additionalProperty`; Tier 3 Organization/Brand/`sameAs`, FAQPage), Pixfizz cautions (server-side rendering, match-visible-content, GTIN validity, `from_pricing`, loyalty/`validForMemberTier`), and two correctness flags on the current snippet (`priceValidUntil` inside a `{% dynamic %}` block; `http` vs `https` context). Added the standalone Schema Builder authoring tool. Updated the Quick capability summary, Pending Confirmation, and Build Opportunities to match. Source: claude-chat (web-verified against developers.google.com).
- 2026-07-25: Documented the v2 manage-admin noindex key mismatch (writes `launch-no-index`, `html.head` reads `no-index`) — the hide-from-search toggle silently does nothing. Source: claude-chat.


=================================================================
FILE: 83_AI_IMAGERY_PRODUCTION.md
=================================================================

# 83 — AI Imagery and Video Production

**Authority Scope:** Guidance for producing AI-generated marketing imagery and video to support a Pixfizz storefront, using Higgsfield AI. This file documents a third-party tool and general production practice. Nothing in it changes Pixfizz CMS, Shopper, or Shopify integration behavior. Vendor capabilities change frequently and are tagged as such.

_Last updated: 2026-07-26_

---

## Purpose

Print labs, photo gifting brands, and school/sports photography businesses need a constant supply of marketing imagery: lifestyle shots for category pages, hero banners, social posts, seasonal campaigns. Traditional product photography is slow and expensive, and most Pixfizz customers do not have a studio budget for it.

AI generation closes that gap for *marketing context* imagery. It does not replace product photography for *the product itself*. That distinction is the single most important rule in this file and it is covered in the next section.

This file covers three things:

1. The rules that must not be broken when using AI imagery on a commerce site.
2. The durable production fundamentals, which stay true regardless of which model is current.
3. The Higgsfield tool landscape as of the date above, quarantined into one clearly dated section so it can be updated without touching the rest.

For storefront SEO and AI-search visibility see `81_SEO_AND_GEO_REFERENCE.md`. For where images are uploaded and referenced in a Shopper site see `50_SHOPPER_TEMPLATE_REFERENCE.md` and `18_ADMIN_NAVIGATION.md`.

**Layer tags used below:**
`[RULE]` = non-negotiable, brand or compliance risk if ignored.
`[GENERAL]` = production practice, not vendor-specific, stable over time.
`[VENDOR]` = a claim about a third-party product. Verify against the current product before relying on it.
`[PIXFIZZ]` = touches Pixfizz platform or Shopper behavior.
`[CONFIRM]` = not yet verified by Pixfizz.

---

## Part A — Non-Negotiable Rules

### A1. Never let AI generate the product itself `[RULE]`

Generate the *scene*. Composite the *real product*.

A generative model does not know what your 210gsm lustre paper looks like, how your canvas edge wrap folds, what your photo book hinge does when it lies flat, or how your foil finish catches light. It will produce something plausible and wrong.

If a customer buys a canvas based on an AI-rendered image that shows a finish, thickness, edge treatment, or colour the lab does not actually produce, that is a misrepresented product. Depending on the market this is an advertising-standards and consumer-protection problem, not just a quality problem. It is also the fastest route to refund requests and chargebacks.

**The correct workflow in every case:**

1. Generate the background, room, table, hands, or lifestyle context with AI.
2. Composite a real photograph of the real product into that scene.
3. Never use image-to-image or video generation to "clean up" or re-render the product region.

**Where AI-only imagery is acceptable:**

- Abstract or atmospheric hero banners with no product visible.
- Seasonal and mood backgrounds behind overlaid text.
- Category tile backgrounds where the product is represented generically rather than as a specific SKU.
- Social content where the product is out of frame or clearly incidental.

**Where it is not:**

- Any image on a product page.
- Any image a customer could reasonably read as "this is what I will receive".
- Any image showing paper stock, finish, binding, frame moulding, mounting, or edge treatment.

### A2. Do not generate identifiable real people without consent `[RULE]`

Trained character models and likeness tools make it trivially easy to produce imagery of a real person. Do not do this with customers, staff, or members of the public without written consent. This applies with particular force in school and sports photography, where the subjects are minors.

Use synthetic characters for lifestyle work, or licensed stock talent, or your own staff with a signed release.

### A3. Do not generate children for school or sports marketing `[RULE]`

Even synthetic. The reputational risk to a school photography business of being seen to use AI-generated children in its marketing outweighs any production saving. Use real, released photography, or frame the shot to avoid faces (hands holding a book, a desk scene, a wall display).

### A4. Check disclosure requirements in your market `[CONFIRM]`

Some AI image models embed invisible provenance watermarks in their output, and disclosure obligations for AI-generated advertising content vary by jurisdiction and are actively changing. Pixfizz does not provide legal advice on this. Confirm your own position before running AI imagery in paid advertising.

### A5. Keep AI imagery out of production files `[PIXFIZZ]` `[RULE]`

This file is about *marketing* assets only. Nothing generated by these tools should ever enter a product template, a design tool asset library that customers can place in their own projects, or a production output file. Licensing of AI-generated content for downstream commercial resale by a third party (your customer) is a separate and unresolved question.

---

## Part B — Durable Fundamentals

These hold regardless of which model or platform is current.

### B1. Still first, then animate `[GENERAL]`

Image-to-video beats text-to-video for commercial work, every time. Generate a strong still frame, approve it, then animate it. Text-to-video gives the model control over composition, product placement, and lighting all at once, and it will get at least one of them wrong.

Practical consequence: never animate from a low-quality or "close enough" starting frame. Everything downstream inherits its faults.

### B2. Prompt in short focused sections, not paragraphs `[GENERAL]`

Break the prompt into segments of roughly six to ten words, each describing one element. Dense paragraph prompts cause models to drop or blend instructions.

**Text-to-video** (no starting frame) needs all five elements:

```
[subject description], [setting], [lighting], [action], [dialogue or "no dialogue"]
```

**Image-to-video** (starting frame supplied) needs only three. The model already knows the subject and the environment from the frame. Re-describing them creates conflicts and degrades the result:

```
[action], [camera movement], [audio, or "no dialogue"]
```

This is the most commonly ignored rule and the most reliable single improvement to output quality.

### B3. Always state audio intent `[GENERAL]`

Audio-capable video models will invent speech, music, or ambient noise if the prompt is silent on the subject. Write `no dialogue` explicitly when you do not want a voice. Write the specific sound when you do (`soft ambient room tone`, `paper rustling`).

### B4. Describe subject motion in any moving-camera shot `[GENERAL]`

A tracking or following shot with no described subject action produces a frozen subject and a moving camera. If the camera follows someone walking, say that they are walking.

### B5. Keep clips short and cut them together `[GENERAL]`

Five to eight seconds per clip holds quality. Lip-sync and physical coherence degrade past roughly ten to twelve seconds in most models. Build a thirty-second ad from four to six clips rather than attempting one long generation. Stitching is cheaper than regenerating.

### B6. Never regenerate the whole image for a small change `[GENERAL]`

Regeneration changes the parts you wanted to keep. Use targeted editing, region-level tools, or dedicated single-purpose apps. This is both a quality rule and a credit-cost rule.

### B7. Capability classes, not model names `[GENERAL]`

Model names and version numbers churn every few weeks. Think in capability classes and map them to whatever is current. The mapping table lives in Part D and is the only part of this file that should need regular updating.

| Class | What it is for |
|-------|----------------|
| **Still — photoreal** | Hero frames, product context stills, base frames for animation |
| **Video — realistic physics** | Natural object and hand movement, page turns, pouring, lip-sync |
| **Video — emotional performance** | Character acting, dialogue, warmth, reaction shots |
| **Video — candid / organic** | UGC and social-native content, smartphone aesthetic |
| **Video — heavy motion / VFX** | Action, dynamic reveals, multi-reference composites |
| **Character consistency** | Keeping the same face across an entire campaign |

### B8. Aspect ratios by destination `[GENERAL]`

| Destination | Ratio |
|-------------|-------|
| Instagram feed | 1:1 |
| Instagram portrait, Pinterest | 4:5 |
| Stories, Reels, TikTok | 9:16 |
| Website hero banner | 16:9 |
| Shopper category tile | Match the tile aspect in your theme `[PIXFIZZ]` |

Generate at the destination ratio where the tool supports it. Outpainting from a square master is a fallback, not a default, because it invents content at the edges.

### B9. Consistency across a campaign `[GENERAL]`

Three levers, in order of effectiveness:

1. **Trained character model** for a recurring person. Requires a set of reference images across varied angles.
2. **Reference image conditioning** for a recurring product, environment, or prop.
3. **Consistent prompt scaffolding**: same lighting vocabulary, same lens language, same colour direction across every prompt in the set.

Without at least one of these, a batch will drift noticeably in tone and palette across ten images.

---

## Part C — Pixfizz Product Type Routing

Capability classes from B7. Model names deliberately omitted so this table does not go stale.

| Product | Still class | Animate class | Shot types that work | Notes |
|---------|-------------|---------------|----------------------|-------|
| Photo book | Photoreal | Realistic physics | Cover reveal, pages opening to a spread, gifting hands | Composite the real cover. Page-turn physics is the hardest thing to fake, so a slow dolly-in on a static open book is safer than an animated turn. |
| Canvas print | Photoreal | Realistic physics or candid | Wall placement, room context, slow pull-back to reveal scale | Edge wrap and depth must come from a real photo. Generate the room only. |
| Framed print | Photoreal | Realistic physics | Gallery wall, hands holding, gift handover | Frame moulding profile is product-specific. Never generate the frame. |
| Personalised mug | Photoreal | Realistic physics or heavy motion | Steam, hands wrapping, morning table, extreme macro | Print wrap and handle geometry must be real. Steam and warmth animate well. |
| Phone case | Photoreal | Candid / organic | Hand-held, pocket pull, casual context | Case cutouts and camera bump are device-specific. Composite. |
| Calendar | Photoreal | Realistic physics | Desk placement, wall context, seasonal setting | Page turn animates acceptably. Binding must be real. |
| Photo gifts, general | Photoreal | Emotional performance | Unwrapping, reaction, handover | Strongest emotional category. Keep the product mostly out of focus during the reaction beat. |
| Sports photography | Photoreal | Heavy motion | Locker room, field-side, team display context | Do not generate athletes who resemble real clients. See A2. |
| School photography | Photoreal | Emotional performance | Parent viewing a print at home, desk display, wall arrangement | See A3. No generated children. Frame around them. |

---

## Part D — Higgsfield Tool Landscape

> **This section is dated and volatile.** Everything below is `[VENDOR]`. Model lineups, app names, and subscription structure change frequently. Verify against the current product before relying on any specific claim. Last reviewed 2026-07-26.

### D1. What Higgsfield is

Higgsfield AI is an **aggregator**, not a model developer. It provides access to multiple third-party frontier models under a single subscription and credit balance, alongside its own workflow tooling and a Model Context Protocol connector.

This is the actual reason to recommend it to a Pixfizz customer: one subscription, one credit pool, one interface, instead of five separate accounts. It is not because it has better models than the model developers themselves.

**Correction to earlier internal notes:** Higgsfield does not own Nano Banana. Nano Banana is Google DeepMind's image model family, marketed under the Gemini Image name (Gemini 2.5 Flash Image, then Gemini 3 Pro Image as "Nano Banana Pro", then Gemini 3.1 Flash Image as "Nano Banana 2"). Higgsfield resells access to it. Similarly, Kling is Kuaishou's, Veo is Google's, Sora is OpenAI's, Seedance is ByteDance's.

**Spelling note:** the video model is **Kling**, not "Cling". Earlier Pixfizz internal drafts carried this error throughout, introduced by audio transcription of video tutorials.

### D2. Capability class to model mapping, July 2026 `[VENDOR]`

| Class (Part B7) | Current model on Higgsfield |
|-----------------|------------------------------|
| Still — photoreal | Nano Banana family (Google Gemini Image) |
| Video — realistic physics | Kling (Kuaishou) |
| Video — emotional performance | Veo (Google) |
| Video — candid / organic | Sora (OpenAI) |
| Video — heavy motion / VFX | Seedance (ByteDance) |
| Character consistency | Soul ID / Soul Characters (Higgsfield) |

Model version numbers move too fast to pin usefully here. Check the current lineup in the product, or via the connector's model discovery tool.

### D3. Higgsfield's own tooling `[VENDOR]`

The aggregated models are commodity. Higgsfield's own layer is the differentiator and is worth understanding:

- **Cinema Studio** — camera control expressed as film language rather than prompt text: format, lens type, aperture and resulting depth of field, plus movement presets (dolly in, dolly out, orbit, crash zoom, tracking, static) and custom camera commands.
- **Soul ID** — trains a persistent character from a set of reference images so the same face survives across an entire campaign.
- **Shots** — generates multiple camera angles from a single image in one pass. Useful both for building a Soul ID training set and for producing product b-roll.
- **Transitions** — morphs between a start frame and an end frame with cinematic transition styles.
- **Outfit Swap** — changes clothing while holding pose, face, and background.
- **Popcorn** — multi-shot storyboarding. Write several scene actions, get the starting frames for a sequence.
- **What's Next** — takes one image and proposes several onward scene directions. Useful when a base frame is strong but the follow-up shot is not obvious.
- **Color Transfer** — applies a reference image's palette to the output. Remove all colour and lighting language from the text prompt when using it, or the prompt overrides the tool.
- **Reference elements** — reusable characters, environments, and props saved per workspace.

### D4. Access paths: web app versus Claude connector `[VENDOR]` `[CONFIRM]`

This distinction matters and is easy to get wrong. Higgsfield can be driven from its own web application or from Claude via a Model Context Protocol connector. **They do not expose the same feature set.**

Observed connector surface as of the date above:

**Available through the connector**

- Image generation, video generation, 3D generation, speech generation
- Model discovery and preset browsing
- Motion control (apply motion and camera movement from a reference clip to a still)
- Image and video upscaling, background removal, outpainting, reframing to a new aspect ratio
- Saved Soul Characters and reference elements (selection and use, not training)
- Media upload and import, browsing past generations
- Dubbing, voice replacement, voice library
- Shorts Studio, clip generation, video analysis
- Marketing Studio (DTC ad generation)
- TikTok publishing chain
- Credit balance and transaction history
- A generic marketplace app search / describe / invoke path

**Web app only, or not confirmed on the connector**

- Cinema Studio
- Soul ID *training* (assembling and training a new character)
- Shots, Popcorn, What's Next, Outfit Swap, Transitions, Color Transfer as named tools

The marketplace invoke path may cover some of the named apps. This has not been verified by Pixfizz. `[CONFIRM]`

**Practical recommendation for customers:** set up both. Use the connector inside Claude for prompt construction, generation, iteration, upscaling, and reframing, which is where the conversational loop genuinely helps. Use the web app for Cinema Studio and for training a Soul ID. Do not tell a customer they can run the full campaign workflow from Claude alone, because at present they cannot.

**Naming quirk:** the connector may appear under a misspelled namespace (`Higgsield` rather than `Higgsfield`) depending on how it was registered. Not an error on the customer's part.

### D5. Credits and cost discipline `[VENDOR]`

Higgsfield runs on a paid subscription plus a credit balance. Some capabilities are gated to higher tiers. Pixfizz does not quote, resell, or guarantee Higgsfield pricing. Customers should check current plans directly.

Cost discipline that holds regardless of the rate card:

- Explore composition with the cheapest available image generation before committing credits to video.
- Use single-purpose editing tools instead of full regeneration for small changes.
- Generate one strong still and animate it, rather than running text-to-video repeatedly.
- Run the same image-to-video prompt across two or three models only when the shot matters. They produce noticeably different results, but testing three models on every clip triples the spend.

---

## Part E — Prompt Templates

### E1. Product still `[GENERAL]`

```
[product type] on [surface], [environment], [lighting],
[lens or shot descriptor], [mood]
```

Example, for a scene into which a real photo book will be composited:

```
Empty light oak dining table, soft morning kitchen behind,
diffused window light from the left, 35mm shallow depth of field,
calm domestic mood, no products on the table
```

Explicitly excluding products from the generated scene is what makes the composite clean.

### E2. Image-to-video, product `[GENERAL]`

```
[action, six to ten words], [camera movement], [audio state]
```

Examples:

```
The photo book opens slowly to a family portrait spread, dolly in, no dialogue
Steam rises gently from the ceramic mug, extreme macro, soft ambient room tone
Hands lift the framed print from the wrapping, slow tracking, no dialogue
```

### E3. Lifestyle and UGC `[GENERAL]`

```
[action] + [setting] + [lighting] + [vibe]
```

Examples:

```
Unwrapping a photo book at a kitchen table, warm lamp light, delighted expression, cosy home
Holding an open photo book showing a team portrait, locker room behind, natural light, quiet pride
```

### E4. Cinematic vocabulary that works `[GENERAL]`

| Element | Terms |
|---------|-------|
| Lens and focus | `extreme macro`, `shallow depth of field`, `anamorphic`, `35mm prime`, `70mm`, `bokeh background` |
| Lighting | `golden hour`, `diffused overcast`, `practical lamp light`, `studio softbox`, `rim light`, `window light` |
| Shot type | `close-up`, `medium shot`, `wide establishing`, `low angle`, `overhead flat lay` |
| Movement | `dolly in`, `dolly out`, `orbit`, `slow tracking`, `static` |

---

## Part F — Common Mistakes

| Mistake | Why it fails | Fix |
|---------|-------------|-----|
| Letting AI render the product | Wrong finish, stock, binding, edge treatment. Misrepresents what the customer receives. | Generate the scene, composite the real product. See A1. |
| Paragraph-length prompts | Models drop or blend instructions | Six to ten word sections, one element each |
| Re-describing subject and setting in image-to-video | The frame already carries that information, so re-description creates conflicts | Action, camera, audio only |
| No stated audio intent | Model invents speech or music | Always write `no dialogue` or the specific sound |
| Moving camera with no described subject action | Subject freezes while camera moves | Describe the subject's motion in the same prompt |
| Clips over ten to twelve seconds | Lip-sync and physical coherence break down | Five to eight second clips, stitched |
| Regenerating everything for a small fix | Loses the parts that were working | Targeted edit tools |
| Colour terms used with a colour-transfer tool | Prompt overrides the tool | Strip all colour and lighting language |
| Animating from a weak still | Every fault is inherited and amplified | Approve the still first |
| No consistency mechanism across a batch | Palette and tone drift visibly across ten images | Trained character, reference conditioning, or fixed prompt scaffolding |

---

## Part G — Recommending This to Customers

Positioning that is accurate and does not overpromise:

- This is a marketing content tool, not a product photography replacement. Lead with that, because it manages expectations and it is also the honest framing.
- The value of the aggregator is one subscription instead of five, plus the conversational loop when the connector is set up in Claude.
- The compositing rule (Part A1) is the difference between this being a useful capability and a refund generator. It should be the first thing any customer is told, not a footnote.
- Pixfizz does not resell, endorse commercially, or provide support for Higgsfield. Customers subscribe directly. Any commercial arrangement between Pixfizz and Higgsfield would need to be stated explicitly and is not in scope for this file. `[CONFIRM]`

Suggested delivery: a short setup walkthrough on `help.pixfizz.com` covering subscription and connector setup, linking here for the production practice. This file is written as AI-assistant context; the help article is the human-facing front door.

---

## Pending Confirmation

- Whether the Higgsfield marketplace app invoke path exposes Shots, Popcorn, Outfit Swap, Transitions, What's Next, and Color Transfer, or whether those remain web-app only.
- Whether Soul ID training can be initiated through the connector or only in the web app.
- Whether Pixfizz has or wants any commercial relationship with Higgsfield, and whether this file should carry a referral link.
- Disclosure and provenance-watermark obligations for AI-generated advertising imagery in the markets where Pixfizz customers operate (US, UK, EU, CA, ZA).
- Whether an equivalent file is wanted for still-image-only tooling, given several Pixfizz customers will not need video at all.

---

## Changelog

- 2026-07-26: Initial version. Consolidated from two internal drafts (`HIGGSFIELD_EXPERT_REFERENCE.md`, `AI_IMAGERY_MASTER_INDEX.md`, both April 2026) into a single Higgsfield-focused file. Corrections applied: "Cling" renamed to **Kling** throughout (Kuaishou; transcription error in the source drafts); Nano Banana attribution corrected from "Higgsfield's own model" to **Google DeepMind Gemini Image** (verified against Google DeepMind and Kuaishou sources). Weavy and Midjourney content dropped, since only Higgsfield is being recommended and the source index referenced two files that do not exist in this repo. Restructured so volatile vendor detail is isolated in Part D and the durable production practice in Parts A to C and E to F does not need touching when model lineups change. Added Part A (product misrepresentation, consent, minors, disclosure, production-file boundary), which was implicit at best in the source drafts. Added Part D4 mapping the connector surface against web-app-only features. Removed the source YouTube link list (unverified URLs, low durability). Source: claude-chat, web-verified.


=================================================================
FILE: 90_FAQ.md
=================================================================

# 90 — Frequently Asked Questions

**Authority Scope:** Customer-facing Q&A grounded in platform truth (files 10–32, 60). Covers Full Pixfizz / Shopper and Shopify + Pixfizz deployments. Not a developer reference — answers are written for store owners and operators.

_Last updated: 2026-07-31_

---

## Scope tags used in this file

- **All** — applies to every Pixfizz deployment
- **Full Pixfizz** — applies to sites using the Pixfizz storefront and checkout
- **Shopper** — applies specifically to Shopper-template sites (subset of Full Pixfizz)
- **Shopify** — applies to Shopify + Pixfizz deployments only
- **Film Labs** — applies to film processing / photo lab businesses specifically

---

## Section 1 — Getting Started & Account Basics

**Q: What is Pixfizz and how does it work?**
_Applies to: All_

Pixfizz is a personalized commerce platform for print and photo product businesses. It handles product personalization (photo books, prints, canvas, calendars, etc.) and the full journey from customization through to production and fulfillment. Depending on your setup, Pixfizz may also run your storefront and checkout, or it may work alongside an existing platform like Shopify.

---

**Q: What's the difference between Full Pixfizz and Shopify + Pixfizz?**
_Applies to: All_

In a **Full Pixfizz** deployment, Pixfizz runs everything — your storefront, checkout, product catalog, payments, and production workflow. In a **Shopify + Pixfizz** deployment, Shopify handles the storefront, catalog, checkout, and payments, while Pixfizz handles product personalization and production. The right setup depends on whether you already have a Shopify store or are starting fresh.

---

**Q: What is a guest user vs a registered user?**
_Applies to: All_

A **registered user** has created an account with login credentials and can access saved projects and order history. A **guest user** completes checkout without registering — they can receive order confirmation emails and their order is tracked, but they cannot log in or access saved projects later. Admins can merge a guest user into a registered account via the admin, which transfers their order history and projects.

---

**Q: Can a customer use the site without being logged in?**
_Applies to: All_

Yes. Pixfizz creates a temporary anonymous identity for every visitor automatically. Customers can browse, personalize products, and add to cart without logging in. If they then register or log in, their cart and any saved projects transfer to their account automatically.

---

**Q: How do I raise a support ticket, and where do I track it?**
_Applies to: All_

Support runs through the **myPixfizz portal** (`my.pixfizz.com`). Raise a ticket there, track its status, and see the full conversation thread in one place. The portal is also where you find training videos, what's new, the roadmap, and your onboarding tasks.

The previous third-party helpdesk is being retired — from **1 September 2026** the myPixfizz portal is the support channel. Tickets raised through the old system before that date are being carried across; if you have an open ticket, it will continue to be worked. Email to the support address still reaches the team either way.

---

**Q: Is there regular training on new platform features?**
_Applies to: All_

Yes. Pixfizz runs a **quarterly review webinar** covering platform updates and new capabilities — the next one is **1 September 2026**. Recordings and walkthrough videos are published in the myPixfizz portal afterwards, so missing the live session is not a problem.

---

## Section 2 — Products, Templates & Collections

**Q: What's the difference between a Product Attribute, a Template, and a Design?**
_Applies to: All_

These three objects work together to define a product:

- A **Template** is the production blueprint — it defines dimensions, DPI, and how artwork is generated for manufacturing.
- A **Design** sits under a Template and is the customer's starting point when they open the editor — a pre-built layout they can personalize.
- A **Product Attribute** is the commercial side — pricing, variants, name, and description — linked to a Template.

Multiple Product Attributes can share the same Template (e.g. metal prints and acrylic prints with identical production specs but different prices).

---

**Q: What's the difference between a design product and a static product?**
_Applies to: All_

A **design product** is a Product Attribute linked to a Template. It requires personalization — customers go through the editor before adding to cart. A **static product** has no Template link — it's a standard item like a frame, gift voucher, or accessory that goes straight to the cart without any customization step.

---

**Q: What is a Collection and how does it relate to products?**
_Applies to: Full Pixfizz_

A Collection is how products are published to your storefront. A product does not appear in your store until it's added to at least one Collection. Collections also determine the URL structure — customers browse by collection path (e.g. `/shop/photo-books`). A product can appear in more than one Collection.

---

**Q: Why can't my customers see a product I've just set up?**
_Applies to: Full Pixfizz_

The most common reason is that the product hasn't been added to a Collection yet. Check that it's published to at least one active Collection. If it is, also check that the Collection itself is active and accessible from your storefront navigation. For Shopify deployments, check that the product metafields are correctly configured — see the Shopify section below.

---

**Q: What are Variants and Options, and what's the difference?**
_Applies to: All_

**Variants** live on Product Attributes. They're the commercial choices customers make (size, finish, material) and affect pricing. **Options** live on Templates or Designs. They're production-level choices. This split means multiple products sharing the same Template don't have to duplicate their production options.

---

**Q: Can I track inventory on my products?**
_Applies to: All_

Yes. Inventory tracking can be enabled per product at the product attribute level. Once enabled, set the current stock count in admin. Pixfizz automatically reduces stock the first time an order reaches Confirmed or Draft status. Stock is only deducted once per order. Note that out-of-stock does not automatically block purchases — the count can go negative. You can add storefront logic to show "Out of stock" messaging or hide the Add to Cart button using the `product.tracks_inventory` and `product.current_inventory` Liquid properties. See `18_ADMIN_NAVIGATION.md` for the full setup guide.

---

## Section 3 — The Design Tool & Personalisation

**Q: How do customers start personalizing a product?**
_Applies to: All_

From the storefront, customers browse to a product, select a Design (their starting point), and enter the design editor. Inside the editor they can upload images, add text, apply layouts, and preview their product. When satisfied, they add it to cart and proceed to checkout. On Shopify, this flow is triggered by a Customize or Personalize button on the product page.

---

**Q: Can I control which features appear in the editor?**
_Applies to: All_

Yes. Design Tool Configurations (found in admin under Settings > Design Tool) let you toggle 30+ individual features on or off — image rotation, crop, filters, autofill, text tools, shape buttons, and more. Different product types can use different configurations. Some configuration settings are managed by Pixfizz staff at the platform level.

---

**Q: I only want image upload available on some templates, not all of them — how do I do that?**
_Applies to: All_

A single global `editor.css` rule (or Custom CSS field) that hides image upload applies to every template using that configuration, so it can't give you per-template control on its own. Because a Design Tool Configuration is assigned per Template or per Design (not per product or category), the way to get per-template control is to create a separate configuration for the templates that should hide image upload, with its own CSS, and assign only those templates to it — leaving the rest on a configuration where upload stays visible. See "Can I control which features appear in the editor?" above and 17_DESIGN_TOOL.md § Design Tool Configurations / Editor CSS Customization. Source: support ticket #18337.

---

**Q: Can customers save their designs and come back to them later?**
_Applies to: All_

Yes. Personalization projects are saved to the customer's account (or to their browser session if not logged in). Registered users can access saved projects by logging in. Admins can view and manage all projects in the admin under Orders > Projects.

---

**Q: What happens if a customer uploads a low-resolution image?**
_Applies to: All_

The design tool can display a resolution warning when an uploaded image is below the minimum DPI required for quality output. Whether this warning appears depends on your Design Tool Configuration. Customers can still proceed with a low-resolution image, so it's worth communicating minimum image requirements in your product descriptions or FAQs.

Relatedly, uploading **CMYK JPEGs** (for example to business-card products) can cause print-quality problems, because the design tool and previews are built around RGB/sRGB. Advise customers to upload **sRGB** images and reserve any CMYK conversion for the fulfillment transformation step. Source: Fireflies (2026-07-03).

---

## Section 4 — Pricing & Options

**Q: How is product pricing set up?**
_Applies to: Full Pixfizz_

Pricing is set on Product Attributes using Ruby formula expressions. Formulas can be simple fixed prices, linear (per item), tiered (volume price breaks), or page-based for multi-page products like photo books. You can also use Price Variables — named numeric values defined once in admin and referenced across multiple formulas — to make bulk pricing updates easier.

---

**Q: What are Price Variables?**
_Applies to: Full Pixfizz_

Price Variables are named numeric values you define once in admin and reference by name in pricing formulas. For example, you could define a variable called `base_rate` and use it across multiple product formulas. When you need to change the rate, you update it in one place and all products using that variable reflect the change automatically.

---

**Q: My pricing isn't displaying correctly — what should I check?**
_Applies to: Full Pixfizz_

Start with the pricing formula on the Product Attribute. Check that it uses the correct variable for the product type — `cut_print_quantity` is only valid for photo prints; all other products should use `units` or `quantity`. Check that any Price Variables referenced actually exist in admin with the right values. If tiered pricing is set up, make sure all quantity ranges are correctly defined and cover the full expected range.

---

**Q: Can I edit product prices without opening each product individually?**
_Applies to: Full Pixfizz_

Yes. Prices are editable inline directly from the product attributes list page. Click on any price field to switch to edit mode, type the new price or formula, and press Enter for simple prices or click OK for multi-line formulas. Press Esc to cancel without saving.

---

## Section 5 — Cart & Checkout (Full Pixfizz / Shopper)

**Q: Can customers edit their personalized product from the cart?**
_Applies to: Full Pixfizz / Shopper_

Yes, depending on your site's cart settings. If the cart edit setting is enabled, customers can re-enter the editor directly from the cart. If not, they would need to return to the product and start a new project. Photo print quantities are not editable in the cart — they are managed inside the Photo Prints UI.

---

**Q: Why is a delivery option missing at checkout?**
_Applies to: Full Pixfizz / Shopper_

If a product in the cart has shipping or pickup marked as unavailable via its custom fields (`shipping_unavailable` or `pickup_unavailable`), those delivery methods are hidden at checkout for that cart. Check the product's custom fields in admin. Film processing products also affect checkout delivery options if the film processing flag is set.

---

**Q: How does digital-only delivery work?**
_Applies to: Full Pixfizz / Shopper_

For digital products — those with the variant version set to `digital-only` — Pixfizz bypasses shipping entirely at checkout. A system address is applied in the background and the customer sees a digital delivery state rather than shipping options. The `digital-only-delivery` admin checklist toggle must be enabled for this to activate.

---

## Section 6 — Shopify + Pixfizz

**Q: How does the Pixfizz integration work on my Shopify store?**
_Applies to: Shopify_

Pixfizz runs on a custom subdomain of your store (e.g. `create.yourstore.com`). A Pixfizz JavaScript API loads into your Shopify theme. When a customer clicks Personalize on a product page, they're taken into the Pixfizz editor. On completion, their project is linked to a Shopify cart item. When the order is paid, Shopify sends a webhook to Pixfizz, which confirms the order and triggers the production pipeline.

---

**Q: My Shopify product isn't launching the Pixfizz editor — what should I check?**
_Applies to: Shopify_

The most common causes are:
1. The `pixfizz.product_sku` metafield isn't set on the product (or on the variant if the product has Shopify variants — the variant-level SKU takes precedence).
2. The `pixfizz.integration_type` metafield is missing or set to an unrecognised value.
3. The Pixfizz JS snippet isn't included in the Shopify theme on that product page.

Check all three before raising a support ticket.

---

**Q: What integration types are available for Shopify products?**
_Applies to: Shopify_

There are four, set via the `pixfizz.integration_type` metafield:

- `editor` — launches the full Pixfizz editor. Default. Used for photo books and products requiring full customization.
- `options-to-editor` — customer selects options first (e.g. calendar start month), then enters the editor.
- `options-to-cart` — shows a live preview modal. Customer selects options and previews before adding to cart, with no full editor. Used for simpler products like notebooks.
- `photo-prints` — launches the photo prints flow. Quantity is managed inside Pixfizz, not in the Shopify cart.

---

**Q: Why isn't my Shopify order being confirmed in Pixfizz?**
_Applies to: Shopify_

This is almost always a webhook configuration issue. Shopify must be set up to send order payment webhooks to Pixfizz. Check that the webhook is active in Shopify (Settings > Notifications > Webhooks) and that the endpoint URL is pointing to your Pixfizz instance. Also verify that the shared secret configured in both systems matches.

---

**Q: Why are extra pages showing as a separate line item in my Shopify cart?**
_Applies to: Shopify_

This is expected for photo book products. Pixfizz adds extra page charges as a linked addon line item, using the product variant configured in the `pixfizz.page_addon_product` metafield. These addons are automatically linked to the parent item and scale with quantity changes. They should not display independently to the customer if the cart template is correctly filtering private line item properties (those prefixed with an underscore).

---

**Q: Can Pixfizz send fulfillment updates back to Shopify?**
_Applies to: Shopify_

Yes. When an order is fulfilled in Shopify, a second webhook can be configured to fire and mark the corresponding Pixfizz order as Shipped. This keeps order status in sync between both platforms.

---

## Section 7 — Orders & Production

**Q: What do the different order statuses mean?**
_Applies to: All_

| Status | Meaning |
|---|---|
| Pending | Order received, awaiting confirmation |
| Confirmed | Order confirmed, ready for production |
| Downloaded | Production assets downloaded by production team |
| Manufactured | Production complete |
| Shipped | Order dispatched to customer |
| Fulfilled | Order delivered and complete |
| Payment Failed | Payment was not successfully captured |
| Error | An error occurred — check the order history log |
| Canceled | Order canceled |
| Refunded | Order refunded |

---

**Q: Why hasn't my order moved to Confirmed?**
_Applies to: All_

For Full Pixfizz orders, confirmation happens after payment is successfully captured. For Shopify orders, confirmation is triggered by the Shopify order paid webhook — if it isn't firing or is misconfigured, the order stays in Pending indefinitely. Check your webhook configuration and review the order's history log in admin for any error messages.

---

**Q: Where do I find production files for an order?**
_Applies to: All_

Open the order in admin and look at the individual orderlines. Each orderline has a Generated Files section with downloadable production assets. You can also use the OrderHub Desktop (OHD) application to manage and download production jobs from your queue.

---

**Q: How do I update an order's status to Shipped?**
_Applies to: Full Pixfizz_

In the order detail view in admin, you can manually update the order status. Moving it to Shipped will trigger the shipping notification email to the customer if email notifications are configured for that status transition (Settings > Email Notifications).

---

**Q: A customer says they didn't receive a confirmation email — what should I check?**
_Applies to: All_

Check that email notifications are configured and active for the relevant status (Settings > Email Notifications). Verify the customer's email address is correctly recorded on the order. Also check that your email sending domain is correctly configured and not being flagged as spam — your Pixfizz account needs a verified sender domain for reliable delivery.

---

**Q: How do I cancel an order and avoid being charged transaction fees?**
_Applies to: All_

Marking an order as "Canceled" in the admin is **not sufficient** on its own to prevent Pixfizz transaction fees. You must also email the order codes to **finance@pixfizz.com** so the finance team can process the cancellation correctly. After sending, follow up to confirm the cancellations have been processed and no charges will apply. Also check that cancelled orders are not duplicated (e.g. from a customer re-ordering after a failed attempt) — duplicate charges can occur if both remain active.

---

## Section 8 — Storefront & Site Management (Full Pixfizz / Shopper)

**Q: Where do I make changes to my homepage?**
_Applies to: Shopper_

Custom homepage content lives in the `website/homepage` snippet in the CMS. This snippet only renders when the "Custom snippet (website/homepage) for home page" checkbox is enabled in Admin > Storefront Settings. Both the snippet content and the checkbox must be active at the same time for your changes to appear.

---

**Q: How do I add a new page to my site?**
_Applies to: Shopper_

On Shopper-based sites, new pages are created by adding instances of the `pages` Custom Type in your CMS admin — not by creating standard CMS pages. Each instance needs a page path, title, and content snippet. The page path must be a URL segment that doesn't conflict with any existing CMS page.

---

**Q: How do I change the logo or site fonts?**
_Applies to: Shopper_

Logo, fonts, and most branding settings are controlled through the admin checklist system in Storefront Settings. The relevant checklist keys are `update-website-logo` for the logo and `font-body` for the body font. Set the value in the checklist and save.

---

**Q: How do I add a promotion bar to the top of my site?**
_Applies to: Shopper_

Enable the `top-promotion-bar` checklist key in Storefront Settings. Once enabled, you can configure the bar content in the associated snippet or setting.

---

**Q: Can I run my store in multiple languages?**
_Applies to: All_

Yes. Pixfizz has built-in translation support for core objects including products, designs, collections, templates, variant types/values, and template option types/values. Enable multi-language support in the Super Admin, select your languages, and then use the "Translate" link that appears on supported objects in admin. Translations are applied automatically across the storefront and design tool — Liquid properties like `{{ design.name }}` resolve to the correct language automatically. Bulk translation management is also available via export/import. See `18_ADMIN_NAVIGATION.md` for the full setup guide.

---

## Section 9 — Common Troubleshooting

**Q: A customer says their project has disappeared — what happened?**
_Applies to: All_

Projects are stored against user accounts. If the customer was browsing anonymously and didn't log in, their project is tied to their browser session — it will be lost if they clear cookies, switch browsers, or use a different device. If they were logged in, the project should still be accessible under their account. Admins can search for projects in admin under Orders > Projects.

---

**Q: Customers are getting a confirmation email but the order still shows as Pending — is that normal?**
_Applies to: All_

It's possible — email notifications can be configured to fire at the Pending stage, so a confirmation-style email can go out before the order is fully confirmed. Check Settings > Email Notifications to see which status triggers which email. If an order stays in Pending for an extended period, investigate the payment or webhook flow.

---

**Q: Sorting on my site looks wrong — items aren't appearing in the order I set.**
_Applies to: Shopper_

Sort order only works correctly when every item in a set has a value in the sort field. If some items are missing a position or order value, the result is unpredictable — those items appear in a random order. Go into admin and make sure every record in that Custom Type has a position value assigned, using a simple sequential numbering like 1, 2, 3. This is a common issue with services, FAQ items, and other Custom Type records.

---

**Q: A customer is getting an error when trying to add a product to cart — what could cause it?**
_Applies to: All_

Common causes include: the product isn't correctly published to a Collection (Full Pixfizz), the product's pricing formula has an error and can't return a valid price, required metafields are missing or incorrectly configured (Shopify), or the customer's session has a conflict with an existing project. Check the admin order logs and, if the issue persists, contact Pixfizz support with the order or error details.

---

## Section 10 — Film Lab Workflows

**Q: How does the Batch Film Uploader work?**
_Applies to: Film Labs_

The Batch Film Uploader automates the process of uploading scanned film to customer galleries via FTP. The full workflow is:

1. **Upload scans to FTP** — copy your daily scanned film folders to the designated Pixfizz FTP folder (typically `Pixfizz/Film Scans`). Pixfizz sets this up for you.
2. **Automatic sync** — Syncovery automatically moves the scanned folders every 5 minutes. When the source folder is empty, your scans have uploaded successfully.
3. **Link folders to orders** — open the order in admin (scan the QR code from the 6x4 label or the barcode on the order sheet). Find the `film_scans` custom field at the bottom of the order page. Enter the film scan folder IDs (must match FTP folder names exactly) and click Add, then Save.
4. **Mark order as Shipped** — after uploading all folders to FTP and linking them, mark the order as Shipped (or Ready for Collection). You can complete this step and step 3 at the same time.
5. **Automatic gallery creation** — after step 4, the system matches the film scan folder IDs with the FTP folders and creates Image Galleries in the customer's account. Each roll of film creates a separate gallery, named "Order code - Folder ID". There is a default 1-hour processing delay to ensure all scans finish uploading before matching begins.
6. **Customer notification** — the customer receives an email notification when their scans are available in their gallery.

Configurable options include:
- Custom notification email content (can include product recommendations)
- BCC email address for record-keeping
- Folder numbering prefix accommodation (e.g. auto-adding "0000" prefix)
- Reply-to address separate from the sending address
- Processing delay adjustment (default 1 hour)

---

## Changelog
- 2026-08-14: Added Section 1 entries for the support channel (myPixfizz portal; third-party helpdesk retired 1 September 2026) and the quarterly review webinar. Source: fireflies-call (2026-08-11/12/13, 3x repeat signal).
- 2026-04-06: Initial version. 35 Q&As covering Getting Started, Products, Design Tool, Pricing, Cart/Checkout, Shopify, Orders, Storefront, and Troubleshooting.
- 2026-05-19: Added inventory tracking Q&A (Section 2), inline price editing Q&A (Section 4), order cancellation and transaction fees Q&A (Section 7), multi-language support Q&A (Section 8), and Batch Film Uploader workflow (new Section 10 — Film Lab Workflows). Source: Notion KB articles.
- 2026-07-04: Added CMYK-JPEG upload caution to the image-upload Q&A (Section 3) — upload sRGB; reserve CMYK for fulfillment transformation. Source: Fireflies (2026-07-03).
- 2026-07-31: Added per-template image upload visibility Q&A (Section 3) — use a separate Design Tool Configuration per template rather than a single global editor.css rule. Source: support ticket.

