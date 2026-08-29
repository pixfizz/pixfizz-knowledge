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

## Value Snippets in a CMS Tar Must Be Byte-Exact

**A value snippet written into a CMS tar must match the parent byte for byte,
including its trailing newline or the absence of one.** Checklist and other value
snippets on shopper24 carry **no** trailing newline: the body of
`admin/checklist/search` is exactly `TRUE`, four bytes.

The parent reads them by capture-and-compare, and Liquid's `capture` does not trim.
A generator that writes `"TRUE\n"` produces a capture of `TRUE\n`, which never
equals `'TRUE'`, so **every comparison of that flag fails silently and permanently**
while the tar imports without error and the snippet looks correct in admin.

Interpolated values are unaffected, because a trailing newline in printed output is
invisible. Only compared values break. **If the brand colours took and the flags did
not, this is the bug.**

Generator fix — carry the seed's own trailing whitespace rather than deciding:

```python
seed_tail = seed_body[len(seed_body.rstrip("\r\n")):]
body = body.rstrip("\r\n") + seed_tail
```

Any generator that writes CMS snippets should **error**, not warn, when an
overridden snippet's trailing whitespace differs from the seed's. See
50_SHOPPER_TEMPLATE_REFERENCE.md §17 for the full signature.

## Archive Emission — Use Psych, Not an Imitation of It

Verified 2026-08-24 against a full Collections → Export All (24 collections, 371
publish rows, 3,118 lines), reproduced byte for byte by a generator.

The documented nil habit is confirmed live: a nil emits as `key: ` with **one
trailing space**, an empty string emits `key: ''`, and a mapping or sequence key
emits `key:` with no space. The distinction survives inside nested maps.

**New, and it changes the advice:** the platform runs a Ruby whose Psych writes that
trailing space. **Ruby 3.3 does not.** So "just use Ruby" is no longer sufficient on
its own — a modern Psych needs a post-pass to restore the space, distinguishing a nil
from a nested structure by whether the following non-blank line is indented deeper
than the column the key starts at, or is a `- ` sequence item at or beyond it.

**Ruby is still the right emitter, and this corrects the Python approach.** Two
further Psych habits appear in this export that a Python emitter does not reproduce:

- **Psych quotes ambiguous scalars.** `product_code: '0408-pro'` and
  `print_theme_code: '0808-cut-print'` are quoted; `4x6-bordered` is not. PyYAML
  quotes none of them.
- **Psych folds long double-quoted scalars at different break points**, and without
  PyYAML's continuation marker.

Measured: a Python emitter with the documented nil fix left **87 differing lines out
of 3,118** — 50 from folding, 37 from quoting. Ruby's Psych plus the nil post-pass
produced a **byte-identical** round trip.

Keep the existing discipline of proving the emitter against a real export and
refusing to run otherwise. It is what caught both of these.

## The Collections Export Format

Fourth member of the archive family alongside the CMS backup, the Custom Type
instance archive and the per-product archive, and it follows the same
five-empty-media-directory convention.

```
./assets/  ./fonts/  ./glb_files/  ./images/  ./pdfs/
./__theme_categories.yml
```

gzipped `.tar.gz`, media directories first in the archive, YAML last.

- Export: Manage Products → Collections → Export All.
- Import: Manage Products → Collections → Import, `enctype: multipart/form-data`,
  file field `exported_file`, accepts `.gz`.

**Record shapes — the platform's key order, do not reorder.**

- **theme_category** — `id`, `product_id`, `name`, `asset_name`, `description`,
  `display_name`, `custom`, `linked_assets`, `product_themes`, `static_products`,
  `translations`. `name` is the URL path segment, `display_name` is the label.
  `product_id` on the collection itself marks a product-specific collection and was
  nil on all 24.
- **product_theme** — a design published against a product — `id`, `product_id`,
  `print_theme_id`, `order`, `published`, `product_code`, `print_theme_code`.
- **static_product** — the same minus `print_theme_code`; `print_theme_id` is nil.
- **linked_assets** — `id`, `thing_type` (`ThemeCategory`), `order`,
  `mapped_preview`, `glb_blob_hash_key`, `asset` (the filename).

**A populated media directory, first time observed in this family.** Binaries are
named by **bare numeric asset id, no extension** (`assets/390591`), and the filename
lives in `__asset_map`, keyed by that id as a quoted string, with **Ruby symbol keys**
inside:

```yaml
__asset_map:
  '390591':
    :name: Cardstock-Prints.jpg
    :description: ''
```

All four maps sit at the **end** of the YAML file, not in separate files; empty ones
emit `{}`.

**Verified on export only.** Whether the importer creates assets from a populated
`assets/` directory, or still requires them to pre-exist in Website → Assets as the
per-product patch documented, is untested. Do not build on it until someone imports
one.

**What a collections export cannot tell you.** No record carries a site or owner
field — no `site_id`, no `website_id`, no hostname, no URL anywhere in the archive,
confirmed by grep across the whole file. A collections export therefore **cannot
answer whether a product or design is owned by the site or inherited from a parent**.
It records what is published where, not where the record lives. Go to Products in
admin (inherited records show but are not editable) or compare exports.

Also unverified: whether a blank `id:` is accepted by the collections importer
(verified for the per-product importer only); whether IDs are global across sites
(the gaps in one export point that way, and if it holds, intersecting theme IDs
between two sites' exports is a definitive inheritance test); how sub-collections are
expressed (all 24 paths were single-segment yet four collections had
`sub_collections: true`); and whether re-importing an archive whose collection `name`
already exists updates in place or duplicates.

## Changelog

- 2026-03-12: Expanded Dynamic Snippet Rule with `style onload` pattern, placement rule, direct init call requirement, and `{% comment %}` block marker convention.
- 2026-03-21: Added Code Change Output Rule — scale output format to edit size.
- 2026-04-05: Added CMS Backup Tar Packaging Rule and Liquid String Quoting Rule.
- 2026-04-09: Added Checklist Snippet Creation Rule — parent template must originate all snippets before child sites can override them.
- 2026-07-28: Added Master Shopper Delivery Rule — never deliver a tar for the parent template site; parent changes ship as paste-ready blocks matching each target file's existing indentation. Source: claude-chat.
- 2026-08-11: Hardened the CMS Backup Tar Packaging Rule — front matter is a database row and a missing `renderer_type` aborts the import with a generic error naming no file; packaging must be one atomic command with a freshness assertion, because a stale tar imports cleanly and changes nothing; the seed backup is the authority for Liquid vocabulary. Recorded that `asset_files/` transfer and CDN content-hash cache-busting were both ruled out as causes. Added the Custom Type Instance Archive Packaging Rule (gzipped, five empty media directories, four `__*_map` keys, literal block scalar for `page_content`). Source: claude-chat.
- 2026-08-29: Added the byte-exact value snippet rule for CMS tars — `capture` does not trim, so a trailing newline makes every compared flag fail silently while the tar imports cleanly; includes the generator fix and the instruction to error rather than warn. Added Archive Emission — the platform's Psych writes a trailing space after a nil scalar and Ruby 3.3 does not, so a modern Psych needs a post-pass; Psych also quotes ambiguous scalars and folds long double-quoted scalars differently from PyYAML (measured 87 differing lines in 3,118 for a Python emitter, byte-identical for Psych plus the nil post-pass). Added the Collections export format — archive shape, import path, the four record shapes in platform key order, the bare-numeric-asset-id convention with `__asset_map` Ruby symbol keys, and the fact that a collections export carries no site or owner field and so cannot answer inheritance. Source: claude-chat.
