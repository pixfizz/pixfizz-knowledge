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
