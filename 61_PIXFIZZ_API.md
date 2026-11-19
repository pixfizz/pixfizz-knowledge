# 61 — Pixfizz API Reference

**Authority Scope:** Pixfizz REST API (v1), JS API, user handoff, project/fulfillment endpoints, dynamic previews, and custom eCommerce integration.

_Last updated: 2026-10-22. Compiled from Pixfizz Notion wiki (API section)._

---

## 1. Overview

The Pixfizz API is a read/write/delete REST API over HTTPS. All responses are JSON.

- **Base URL:** `https://<subdomain>.pixfizz.com/v1/`
- **Version:** v1 (breaking changes will be released as v2 — v1 will not have breaking changes)
- **Format:** JSON only. Send `Content-Type: application/json` for POST/PUT requests with JSON bodies.
- **Timestamps:** ISO 8601 (`YYYY-MM-DDTHH:MM:SSZ`)
- **Pagination:** Append `?page=N` to index endpoints. `page=3` fetches the third page of results. **Page size is not contractual and varies by endpoint**: `/v1/admin/products.json` returns 20 per page (verified by query, 2026-09-23); § 13f resources are announced at 100; the admin-host `/site/<site>/admin/assets.json?page=N` returns 50 per page with `id`, `name` and `url` (verified by query, 2026-10-15; the Assets admin screen does not print asset names in its HTML, so check new uploads through that JSON); `/v1/admin/users.json` returns 20 (§ 3). Never hardcode a page size: page until a request returns an empty list, keep a generous page-count cap as a runaway guard, and report it if the cap is hit rather than presenting a truncated result as complete. The public `/v1/products.json` also pages at 20 by default; `?per_page=500` returned all 238 products of one site in a single call (verified by query, 2026-10-02).
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
### API keys (recommended for every Basic-auth call)

Since 2026-09-23 an admin **API key** can replace the username and password on **every request that uses HTTP Basic auth**, including every `/v1/admin/...` call. Platform-level (Pixfizz CMS): the same on Shopper sites and Shopify + Pixfizz sites.

- Send the key as the Basic-auth **username**. The password is ignored and can be empty. No email or username is needed: the key identifies the user it belongs to.
  ```
  curl -u pxk_<key>: https://<subdomain>.pixfizz.com/v1/admin/orders.json
  ```
- Keys start with `pxk_` followed by a long hex string. Validate on the prefix; do not hard-code the length.
- A key bypasses the brute-force login protection that applies to username and password logins, and saves about 0.5 s per request.
- **A key belongs to one user.** Admin → Users → open the user → **API Keys** section (below Change Password and Custom Fields) → **Add API Key**. There is no site-level API keys page. A key created in a site's regular admin reaches only that site, also when the user is a super user.
- **Super user keys (from November 2026).** API keys can also be assigned to super users **in the super admin**. Such a key reaches every site that super user can access, so it is more security-sensitive than a site key: issue one only for an integration that genuinely spans sites, and prefer per-site keys otherwise. *Stated by the core developer, Notion Dashboard, November 2026.* *Corrected 2026-11-09: this section said there was no global or super-admin key and that a cross-site integration needs a key per site.*
- **Service users.** A checkbox on the user page (admin and super admin) marks a user as a **service user**: used for automation, it accesses Pixfizz only with API keys and cannot log in with a username and password. Set it on every user that exists only for API access. *Stated by the core developer, Notion Dashboard, November 2026.*
- The full key is shown **once**, when it is created. Copy it then. The Name is only a label. The table lists Name, the key masked to its first 12 characters, Created, Last Used ("never" until the first call) and Delete. Deleting a key revokes that key alone.
- Practice: create keys on a dedicated admin API user (marked as a service user, above), not on a person's own account, and issue one key per integration, named after it, so Last Used shows which integration is live and each can be revoked on its own.
- Not verified: that a key carries exactly its user's permissions, and that deleting the user or unticking Admin disables its keys. Test before relying on either.
- A key does not change the base URL. See "Admin UI host vs API host" below.

*Stated by the core developer (Notion Dashboard, "New Feature: API Keys"); admin screens verified from screenshots and a live key test by query, 2026-09-28.*


### Pixfizz MCP connector

A Pixfizz MCP connector lets an AI assistant work with a Pixfizz site through these APIs: sign in with myPixfizz, then test with a request such as "list my Pixfizz sites". Every change it makes is logged and can be reverted. *Stated in team Slack, 2 Nov 2026.* Hosting detail: `70_MYPIXFIZZ_OVERVIEW.md` § Infrastructure Notes.

Customers have connected it from the Claude desktop app on Windows; a setup in ChatGPT was not completed (November 2026). *Stated in customer calls, 2026-11-10 and 2026-11-12.*

### HTTP Basic Auth (server-to-server, recommended)
Use an admin user's **API key** (above), or, as the legacy form, the admin user's email and password. Required for all admin endpoints.

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

**The admin session cookie is `__Host-` prefixed since 2026-09-23.** Sessions were seen dropping repeatedly during long browser-driven admin runs that day; treat a run of unexpected 401s or login redirects mid-session as a possible session drop before suspecting the credentials. *Observed live, not root-caused.*
### Admin UI host vs API host: never interchangeable

Since the 2026-09-23 deploy the admin UI lives on its own host. **The API did not move.** Platform-level (Pixfizz CMS).

| Purpose | Form |
|---|---|
| Human admin UI (what the browser bar shows) | `https://admin.pixfizz.com/site/<slug>/admin/...` |
| API base (what an integration stores and calls) | `https://<slug>.pixfizz.com` + `/v1/admin/...` |

- `https://<slug>.pixfizz.com/admin` answers **301** to the same path on the admin host. `https://<slug>.pixfizz.com/v1/admin/orders.json` answers directly (401 without credentials, no redirect).
- `https://admin.pixfizz.com/site/<slug>/v1/admin/orders.json` is **404**. Appending `/v1/admin/...` to a URL copied from the browser produces a path that does not exist.
- The slug is the same in both forms on every site checked.
- People now copy the admin-host URL from their browser, and it is the wrong value for an integration's base URL. Convert a pasted admin link to `https://<slug>.pixfizz.com` before storing it; do not loosen the validation to accept it.
- **A login on `admin.pixfizz.com` is not a storefront admin session.** Signed in on the admin host, the same browser on `<slug>.pixfizz.com` got 401 *admin privileges required* from `/v1/admin/orders.json`. Storefront pages gated on `user.is_admin` (Shopper `manage/*`, `setup/*`, custom preview pages) show nothing until the admin also signs in on the storefront itself. A site with no storefront login page (an Etsy-flow site, for example) needs one added before such a page can be used. *Verified by query, 2026-10-07.* That the storefront sign-in then sets `user.is_admin` is not verified.
- **A script on an admin page cannot call the site host.** A `fetch` from a page on `admin.pixfizz.com` to `https://<slug>.pixfizz.com/v1/...` fails (the admin CSP, § 13h, blocks every host but `'self'` and `cdn.pixfizz.com`), and a `catch` that turns the failure into an empty result reads as "0 rows". Two ways round it: call the admin host itself with `?sitename=<slug>` where the route answers there (assets, products and collections were seen working; `/v1/theme_categories/<name>.json?sitename=<slug>` works same-origin, *verified by query, 2026-10-20*), or collect what you need on the admin page, navigate the same tab to the site domain and call `/v1` there (theme previews and theme JSON were read this way, *verified by query, 2026-10-19*). Platform-level.

*Verified live (unauthenticated requests and a logged-in browser), 2026-09-23, 2026-09-24 and 2026-09-28.*


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

It returns the session user, with `id: null` for a visitor who has no user yet. Creating a project creates the anonymous user. `/v1/users/current.json` and `/v1/me.json` return 404. *Verified by query on a Shopper child, 2026-10-03.*

### List all users (admin)
```
GET /v1/admin/users.json
```

- Fixed **20 users per page**. `per_page`, `page_size`, `limit`, `per` and `count` are ignored. Page with `?page=N`; an empty array is the end. There are no total or link headers. Parallel paging works (12,000+ users in under 30 s with 6 requests at a time).
- Each user carries `telephone`, `custom`, `category` and `address`, but `address` was null on every user tested. Read saved addresses with `GET /v1/users/<id>/addresses.json`, which works with an admin key for any user on the site. Address fields: `id`, `telephone`, `title`, `first_name`, `last_name`, `company`, `street`, `street2`, `zipcode`, `state`, `city`, `country`, `email`, `public`, `custom`. There is no `created_at`: "first address" means the lowest id.
- `PUT /v1/users/<id>.json` with only `user[telephone]=...` saves the phone and leaves names, email and custom unchanged. The PUT validates the whole user: a stored invalid email answers 422 `{"email":["is invalid"]}` and nothing saves. A user whose email is on pixfizz.com answered 403 `{"error":"Access not permitted"}` to a site key (probably Pixfizz staff or super admin accounts; not verified). Both fail loudly.

*Verified by query with an admin API key, 2026-10-13.*

### Create users in bulk (no CSV import)
There is no user CSV import in the admin. Create accounts one by one with `POST /v1/users.json` and an admin API key as the Basic username, JSON body `{"user":{"email","first_name","last_name","telephone","custom":{...}}}`. Custom fields are stored on create when their definitions exist on the site. Platform-level. *Verified by query on a client site, 2026-10-28 (about 2,450 accounts).*
- **Omit `external_id` and `external_source`**, or every account becomes an external, non-login user (§ 7).
- **About 4% of creates in a burst of about 250 parallel requests failed with a dropped connection and were not created.** Verify each account by email afterwards (page `GET /v1/admin/users.json`, 20 per page; it ignores `email` and `q`), then retry the missing ones.
- Each create fires `user.created` to the site's webhooks. If a webhook consumer acts on new users, give it a way to skip imported accounts before the run.
- Whether an API-created account triggers the User Signup email is not proven (none arrived in one 5-minute test). Empty the User Signup email body before a bulk run if no email may go out.

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

With a site admin API key, `GET /v1/users/<user id>/books.json` returns that user's saved projects, each with `id`, `name`, `saved`, `ordered`, `preview` (`/v1/books/<id>/preview.jpg?t=...`), `created_at`, `updated_at` (store local UTC offset), `user_id`, `theme_id`, `theme_code`, `product_id`, `product_code`, `template_id`, `template_code`, `options`, `template_options`, `build_parameters`, `custom` and `links`. *Verified by query on a client site, 2026-10-17.*

**There is no admin endpoint that lists projects across users.** Finding unfinished designs on a site means paging `GET /v1/admin/users.json` (20 per page, § 3) and reading each user's books. Platform-level. *Verified by query, 2026-10-17.*

### Read a project
```
GET /v1/books/<id>
```

`GET /v1/books/<id>.json` with an admin key returns more than the list entry: `images[]` (`id`, `width`, `height`, `filename`, thumbnails at 150, 250, 600, 900 and 1200 px, `preview`), `page_count`, `total_pages`, `source_book_id`, `product_name`, `theme_id`, `theme_code`, `template_id`, `template_code`, `template_options`, `options`, `unit`, `dpi`, `minimum_dpi`, `share_code` and `links` (`pages`, `files`). The length of `images[]` tells whether the customer uploaded any photos. *Verified by query on a client site, 2026-10-18.*

### A saved project and its ordered project are two books

At checkout the platform orders a **separate book** for the line. The customer's saved project keeps `saved: true, ordered: false`, so `ordered: false` in `/v1/users/<id>/books.json` does **not** mean the design was never bought. Platform-level. *Verified by query on a client site, 2026-10-17.*

- The two books are linked by `source_book_id` on the saved one: in the case checked, the saved book's `source_book_id` was the id of the book on the customer's order, and that ordered book had the lower id. Use `source_book_id` to tell exactly which saved designs converted. *Verified by query, 2026-10-18.* Which of the two the platform copies from which is not confirmed.
- Matching on `book_id` alone finds nothing (0 of 165 saved designs on one site). Without `source_book_id`, the fallback is to match order webhook orderlines by the same user id and product code at or after the design's `created_at` (127 of 165 matched this way). *Verified by query, 2026-10-17.*

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
- `book[start_year]`, `book[start_month]` (zero-padded), `book[start_day]` — calendar products only; the response's `build_parameters.start_date` confirms the start
- `book[calendars][]=<calendar id>` (repeat per calendar) — calendar products only; without it `website-calendar` and `user-calendar` captions do not render (`23_XML_CALENDAR_REFERENCE.md`, § 11 below). *Verified by query on a test site, 2026-10-16.*
- **A calendar project's start date and holidays are fixed at creation.** `PUT /v1/books/<id>.json` with `book[start_year]` / `book[start_month]` / `book[start_day]` or `book[calendars][]` answers 200 and changes nothing (`build_parameters.start_date` and the holiday calendars stay as created); a changed start or holiday set needs a new project. Platform-level. *Verified by query, 2026-11-05.*

More on `variants[...]` and `template_options[...]` on create (platform-level, *verified by query, 2026-10-20 and 2026-10-21*):
- `variants[<code>]=<value code>` works on a design project, and the project records it in `options` (for example `{size: <value code>}`). It is accepted for variant types that are **not published** (not on the product page form), so a tool can set a hidden variant on create.
- **Text template options are written into the page XML at create:** `template_options[name]=X` puts X into `<text name="name">`. An empty value empties the element; a missing value keeps the design's default text.

`POST /v1/books.json` with only `product_id`, `theme_id` and `book[saved]` creates a project. *Verified by query, 2026-10-30.*

Example response fields: `id`, `name`, `saved`, `ordered`, `preview`, `theme_id`, `theme_code`, `product_id`, `product_code`, `template_id`, `template_code`, `options`, `template_options`, `links`.

What a create also does (*verified by query, 2026-10-07 and 2026-10-08*):
- `book[template_options][<code>]=<value>` on create, with `book[saved]=true`, returns a project that already carries those template options (response `template_options`). A test tool can use this to open a real edit page without placing an order. `book[options][<variant code>]` on create is not verified.
- `book[pages]=N` on create gives exactly N pages (plus the cover as one more page object), and the spine width follows the count. **The platform does not clamp to the template's minimum page count:** 24 pages were accepted on a template whose minimum is 40. Any minimum or step must be enforced by whatever calls the API. To change the count later, create a new project (`PUT book[pages]` returns 500, below). Cart line, price and print file at such counts are not verified.

### Update a project
```
PUT /v1/books/<id>
```
- Rename: `book[name]=new+name`
- Mark saved: `book[saved]=1`
- Mark ordered: `book[ordered]=1`

Do not directly edit the XML structure via API — this will cause editor issues.

What else a project update does and does not change (*verified by query, 2026-10-04*):
- `book[saved]=false` on create keeps the project out of the saved list (`/v1/users/<id>/books.json`); `PUT` with `book[saved]=true` saves it later.
- `PUT book[template_options][<code>]` changes a template option. The unprefixed `template_options[...]` is ignored on update.
- `PUT book[product_id]` and `book[theme_id]` return 200 and change nothing, and `book[source_book_id]` on create is ignored: a different size or design needs a new project.
- `PUT book[pages]` returns 500.
- **Projects cannot be deleted**, including test projects a tool creates. Record test project ids instead of planning to remove them; `book[saved]=false` keeps a project out of the saved list. *Stated by the Pixfizz team, 2026-10-13; not verified by query.*

### A project's own gallery
```
GET /v1/books/<id>/gallery.json
```
Each project has its own gallery. Only images in that gallery appear in the editor's photo tray for that project. Project galleries are **not** listed in `/v1/galleries/_mine.json`. To put existing images into it, copy them server-side with `/upload/image` (§ 6). *Verified by query, 2026-10-04.*

### Read a project's pages
```
GET /v1/books/<id>/pages.json
```
Returns the project's pages, 20 per response; `?page=2` returns the next 20, and `per_page` and `limit` are ignored. Its page previews are 150 px thumbnails, not usable for a large preview. *Verified by query, 2026-10-03 and 2026-10-14.*

`GET /v1/books/<id>/pages.json?page=1` also works in a guest session and returns the page XML (`data`) plus `images[]` with `width`, `height`, `filename` and thumbnail URLs. `/v1/projects/<id>` is 404: the project endpoint is `/v1/books/`. *Verified by query, 2026-10-08.*

### Add to the cart from a script (Shopper 24 product form)

A design product's `form#project_create` posts to `/v1/books` with `product_id`, `theme_id`, a per-page `_cms_form` token, `variants[...]`, `template_options[...]`, `quantity` and `book[saved]=true`. **`editor=add-to-cart` is the name and value of the Add to cart button, not an input.** A script that builds `FormData` from the inputs must add it, or the post opens the editor (`/v1/editor?book=<id>`) and nothing reaches the cart. With it, the post answers with `/site/add-to-cart?book=<id>`, a page that auto-submits `POST /cart/add_print_product`. A `fetch` does not run that page's script, so post `/cart/add_print_product` yourself (`20_SHOPPER_CART_RULES.md` § Cart links). Called with `fetch`, `/cart/add_print_product` redirects to `/site/cart`, and a default `fetch` follows it and downloads the whole cart page: pass `redirect: 'manual'` and treat `r.type === 'opaqueredirect'` as success. *Seen in a studio speed round, 2026-11-05; not verified by query.*

- To land a line on another product, GET that product's page, take its form (a fresh `_cms_form`), fill it and post. Repeat per line, one after another: about 2 s per line plus the uploads. Each line is its own saved project, priced by its own product and variants.
- Shopper size filters switch product with `?size%5B%5D=<size>` on the collection shop path. A plain `?size=` is ignored.
- A shopper can rename a project (`PUT /v1/books/<id>.json` with `book[name]`), which works as a per-line label without a custom field. The Shopper cart does not show it.

Platform-level; the form shape is template-level (Shopper 24 `product/design-now`). *Verified by query on guest carts, 2026-10-07 and 2026-10-08.*

### A cart line from template options only (no editor)
For a design whose printed result is driven entirely by template options, a script can create the cart line without the editor. *Verified by query on a Shopper 24 child, 2026-10-26.*
- `POST /v1/books.json` with the product page form's hidden fields (`_cms_form`, `product_id`, `theme_id`), `template_options[<code>]` for every option and `book[saved]=true` returns a saved project carrying the options. Then `POST /cart/add_print_product` with `print_book_id`, `quantity`, `sum_quantities=false` and `target` adds it; the line is priced per unit by quantity and lists the options. No hidden editor step is needed.
- A File Upload template option takes the id returned by `POST /upload/file` (§ 6), written as `db:<id>`: `template_options[<code>]=db:<id>` on create stores the file on the project. A bare `<id>` is saved but the uploaded file stays empty, so the cart and order preview ignore it (see 22, file_upload; corrected 2026-10-29 from a 26 Oct test). The id survived the editor's save, `PUT book[saved]=true` and `/cart/add_print_product`; an option with `hide_from_cart` does not show on the cart line. *Verified by query, 2026-10-23.* A File Upload option on a **template provider's** template works the same on a consumer product (read back in the project's `template_options`). *Verified by query, 2026-10-23.*
- `px-image-upload` and `px-upload-dialog` are also defined on a Pages instance page (the Shopper 24 CMS bundle loads site-wide), so such a page can take uploads without being a product page. Template-level (Shopper 24).
- Remove a cart line by script: POST the line's own form to `/v1/orderlines/<id>` with `_cms_form` and `_method=delete`. Calling `click()` on its Remove button from script did not submit.
- Empty a test cart: POST the Clear Cart form (`/cart/clear`) with its own fields; this avoids the browser confirm dialog. *Verified by query, 2026-10-27.*
- **Change a line's quantity by script:** the Liquid `orderline_update` form tag accepts `id:` and `class:`, so a page can render a findable form per line; a `fetch` POST of that form with `orderline[quantity]=<n>` updates the line. A project made with `editor=add-to-cart` and a `quantity` field answers `/site/add-to-cart?book=<id>&quantity=<n>`, and `POST /cart/add_print_product` with `print_book_id`, `quantity` and `sum_quantities=false` lands the line at that quantity; this also works from a product grid tile with no product page. *Verified live on a client site and a test site, 2026-11-03.*

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

**Treat a preview request as a write.** A request for `/v1/books/<id>/preview.jpg` can render the preview on demand and set the project's `updated_at` to that moment. Fetch once and cache, and never read an `updated_at` taken after your own preview fetch as customer activity. Platform-level. *Verified by query on a client site, 2026-10-17.*

### Retrieve embedded text
```
GET /v1/books/<id>/text_elements
```

### Data retention

Project, image and cart lifetimes are set by the platform deletion policy, not by the API.

| Object | Deleted |
|---|---|
| Cart never checked out | 1 year |
| Abandoned cart | 3 months. Never deleted on Etsy-enabled sites |
| Saved cut print project | 6 months |
| Saved project, any other type | No fixed expiry. Goes with the user on inactivity |
| Project gallery, cut print | 3 months |
| Project gallery, any other type | 1 year |
| Images on an **ordered** cut print project | 6 months after the order |
| Images on any other **ordered** project | 3 years after the order |
| Images referenced nowhere | Removed by periodic cleanup once detected unused |
| Anonymous user and all their data | 3 months |
| Registered and guest users | Never |
| Website crawls | Last 5 kept regardless of age; older ones at 1 month |

PDFs and uploaded files follow the image policy exactly.

**An inactive user loses everything.** Inactive means **no login for 4 years**, guest users
included. All their galleries and all their saved projects are deleted.

**An inactive site loses every user-uploaded image.** Website and theme assets are kept. A site
is inactive when its super account is flagged inactive in super admin.

> **Correction.** An earlier version of this table said unsaved projects are kept 1 year from
> last activity, saved projects 2 years from last save, and ordered projects **indefinitely**.
> Ordered projects are not kept indefinitely: the order record persists, the images behind it do
> not. See `40_PLAYBOOK_UPDATED.md` § A Customer's Old Project Shows Broken Images for the
> support-facing version.

*Verified by reading source: Pixfizz Wiki → Legal & Compliance → Deletion Policies,
last edited 2025-08-13.*

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

**Production file from the admin, without an order:** on the admin project page `/site/<site>/admin/print_books/<id>`, POST its `files?kind=production` form with that form's `authenticity_token`; the file appears on the project page as File #0 (`/book_files/...pdf`) and is logged in Money Logs as `production_book_file` against the project. Detail: `17_DESIGN_TOOL.md` § Text, image and layout calls. *Verified by query, 2026-10-30 and 2026-11-01.*

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

`GET /v1/galleries/_mine.json` answers through a redirect to `/v1/users/<user id>/galleries.json`, so a same-origin storefront script can take the Pixfizz user id from the final response URL (`Response.url`) instead of trusting an id printed in the page. *Verified by query, 2026-10-13.*

### Create a gallery
```
POST /v1/users/<user-id>/galleries.json
-d "gallery[name]=my-gallery"
```
All galleries must be associated to a user.

A server can create a gallery for a specific end user with an admin API key (§ 2) acting for that user, with no browser session: find the user's id first (§ 7, *Look up a user by external ID*), then post `gallery[name]` to that user's galleries endpoint. The response includes the gallery id. *Stated by the core developer, 2026-10-01.*

### Read / update a gallery
```
GET  /v1/galleries/<id>.json
PUT  /v1/galleries/<id>.json  -d "gallery[name]=Updated+Name"
```

**Gallery custom fields are writable.** `PUT /v1/galleries/<id>.json` with form data `gallery[custom][favorites]=111,222`, sent as the owning customer, returned 200 and a fresh `GET` returned `"custom":{"favorites":"111,222"}`. Every row of `/v1/galleries/_mine.json` (and `/v1/users/<id>/galleries.json`) carries `custom`. Platform-level. *Verified by query, 2026-11-06.* Not verified: whether a Gallery custom field definition is needed for the value to store (the per-site definition rule may not apply to galleries), and whether `gallery.custom.<field>` resolves in Liquid.

**A subset of a customer's photos as its own gallery.** Create a gallery for the user (§ *Create a gallery*), copy the chosen images into it with `POST /upload/image?gallery_id=<new>` and `url` + `name` (§ *Add an image*), use it, then `DELETE /v1/galleries/<id>.json`. This is how a Shopper 24 prints page is opened on only some photos, since it takes a `gallery_id` and no list of image ids (`50_SHOPPER_TEMPLATE_REFERENCE.md`). *Verified by query, 2026-11-06 (two copied photos showed as "2 photos, 2 selected").*

### Add an image

**Use `POST /upload/image` on the site host.** (Corrected 2026-10-06.) The core developer stated on 2026-10-01 that the gallery images POST endpoint below "was removed some time ago". Platform-level (Pixfizz CMS).

```
POST https://<subdomain>.pixfizz.com/upload/image
```

- Multipart upload with the file parameter named `data`, **or** `url` (a publicly reachable image URL; Pixfizz downloads it) plus `name` (the filename). `name` is optional: without it the filename is the part of the URL after the last `/`. *Stated by the core developer, 2025-04.*
- Optional `gallery_id` uploads into a specific gallery.

*Stated by the core developer, 2026-10-01.*

**Customer upload by script.** On the storefront host, `POST /upload/image` with the multipart field `data` uploads an image as the current shopper (a guest works) and returns `{id, width, height, ...}`. An image upload option's value is then `db:<id>` (`22_OPTION_VARIANT_RENDERING.md`). *Verified by query on baseline.pixfizz.com, 2026-10-06.* The `url` + `name` form with `gallery_id` is verified below.

Deprecated form, kept for reference:

```
POST /v1/galleries/<id>/images.json
-F Filedata=@image.jpg
-d "tags=tag1,tag2"
```

This route still answered 200 when tested by query on 2026-09-15 and 2026-09-17, so existing code that calls it has not broken yet. Treat it as deprecated: write new code against `/upload/image`. Unconfirmed: whether and when the old route stops answering.

**Copy an existing image into another gallery, server-side:** `POST /upload/image?gallery_id=<target gallery>` with a URL-encoded body `url=<image.url>&name=<filename>`. It returns the new image JSON in about 0.2 s, same width and height. This is what the upload dialog's URL source calls. Use it to move photos into a project gallery without re-uploading. A file upload into a gallery sends the file as `data` (`fd.append('data', file, name)`). *Verified by query, 2026-10-04.*
- **The copy keeps PNG transparency** (a 640 px sticker copied by URL kept corner alpha 0 and center 255), and **the source can be another site's asset on `cdn.pixfizz.com`**. So a tool can keep its print artwork as assets on one site and copy it into each customer's project gallery, with no per-site image ids in its configuration. *Verified by query on a test site, 2026-10-29.*

**File field name: two observations.** `data` worked on 2026-10-04, 2026-10-06 and on 2026-10-23 with `gallery_id` on a test site. On 2026-10-26 `POST /upload/image?gallery_id=<project gallery>` also accepted the field **`Filedata`** (plus `name`) and kept the pixel size (1725 x 2625 PNG in and out); `file`, `image` and `upload` answered 400 "failed to parse file from the request". Neither test tried the other name. Safe rule: send `data`; `Filedata` is a known working alternative. *Verified by query, 2026-10-23 and 2026-10-26.* Use: a tool that renders artwork in a canvas creates the project (§ 4), reads the gallery id from `/v1/books/<id>/gallery.json`, uploads the canvas PNG and then fills a placeholder.

**A PDF uploaded through `/upload/image` becomes an image.** It is rasterized at 300 dpi of its page size (a 5.25 x 7.25 in page gave 1575 x 2175 px), fills an image placeholder and prints in the fulfillment render. A PDF page with no white fill is transparent, so whatever sits below it shows through. *Verified by query on a test site, 2026-10-23.*

### Upload a file (not an image)
```
POST https://<subdomain>.pixfizz.com/upload/file
```
Multipart, file in the field **`data`** (`file`, `upload`, `qqfile`, `files[]` and `image` all answer 400 "failed to parse file from the request"). Returns `{id, filename, url}`. PDF, CSV and JSON were accepted; an 8.5 MB file took under 2 s. The `id` is the value for a File Upload template option (§ 4). Platform-level. *Verified by query on a test site, 2026-10-23.*

**Copy by URL works here too.** `POST /upload/file` with a URL-encoded body `url=<an upload's url>&name=<filename>` copies the file server-side and returns `{id, filename, url}`, like a multipart upload. A 2-page PDF first uploaded through `/upload/image` came back with the same content-addressed `url` and identical bytes (SHA-256 checked). So a tool that took the customer's PDF through the upload dialog (which sends it to `/upload/image`) can fill a `file_upload` option from that URL without sending the file again: set the option value to `db:<id>` (`22_OPTION_VARIANT_RENDERING.md` § 5.2). Platform-level. *Verified by query on a test site, 2026-11-04.*

### Delete a gallery
`DELETE /v1/galleries/<id>.json` as the owning user (a guest session works) returns 200 with an empty body; the gallery leaves `/v1/galleries/_mine.json`. *Verified by query, 2026-10-04.*

### Gallery size

There is no hard limit on images per gallery or per user. Keep each gallery below 1,000 images. *Stated by the core developer, 2026-10-01.*

### Read a single image
```
GET /v1/images/<id>.json
```
Returns 403 without a session: a storefront page cannot resolve a `db:` image id through it. *Verified by query, 2026-09-29.* On a site domain it needs an **admin** login, so design artwork cannot be fetched through `/v1/images` without signing in; read it from a template export instead (`16_PRODUCT_HIERARCHY.md` § Template Export and Import Format). *Verified by query, 2026-10-19.* For a **guest's own upload**, `GET /v1/images/<id>.json` on the storefront answered **500** (and 403 for a library image); a page that needs the photo's pixel size should read `img-width` / `img-height` from the `px-image-upload` element instead (`22_OPTION_VARIANT_RENDERING.md` § 5.1). *Verified by query, 2026-10-30.*

**A third observation, not yet reconciled (2026-10-29).** Signed in on a test site, `GET /v1/images/<id>.json` answered for an image in the signed-in user's own galleries with `id`, `width`, `height`, `filename`, `preview`, `url` (full resolution, on the store's host) and `thumbnails [{size, url}]` (150, 250, 600 ...). That account was a Pixfizz staff account, so it does not show whether an ordinary shopper gets the same answer for their own images. *Verified by query, 2026-10-29.* Pending confirmation: whether the route serves a shopper's own images to that shopper. Until then a tool that stores only an image id must not rely on reading it back through `/v1/images` as a customer.

### File uploads are content addressed

The `file-url` a `px-file-upload` sets (`/fz/<hash>/0/<token>/~/<name>`) depends on the bytes: uploading the same bytes twice gives the same `file-url`, and the attribute does not change on the second upload. A script that detects "upload done" by waiting for `file-url` to **differ** from its earlier value hangs when the customer re-uploads an identical file, or when a tool rebuilds a byte-identical print file. Clear the attribute before injecting the file, then wait for it to become non-empty. Platform-level (file uploads), with `px-file-upload` on Shopper 24. *Verified by query on experience.pixfizz.com, 2026-10-22.* Tool pattern: `26_CUSTOM_DESIGN_TOOLS.md`.

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

**Merging users.** The merge call shows a confirmation warning before it merges. Through the API, add `imsure=true` to the request to skip it. *Stated by the core developer, Slack #development, 2026-10-19.* The myPixfizz Merge users tool (`71_MYPIXFIZZ_FEATURES_ROUTES.md`) calls this API but cannot pass `imsure`, so a merge that stops on the warning there needs the call made directly. **Merge direction matters:** a merge answered 403 when the user being deleted (the duplicate) held all the projects; merge so the account that keeps the projects survives, or move them first. *Observed by Pixfizz staff, 2026-10-19; cause not confirmed by the developer.* The Shopify duplicate-user case that needs this is in `60_SHOPIFY_INTEGRATION.md`.

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

### Look up a user by external ID (server-side)

```
GET /v1/users/_uid/<external-source>/<external-user-id>.json
```

The GET form of the handoff path does not log anyone in. It redirects to `/v1/users/<id>.json`, which carries the Pixfizz user id. It works with an admin API key (§ 2, HTTP Basic, key as username), so a server can find a user and then create galleries and upload images for them (§ 6) without a browser session. *Stated by the core developer, 2026-10-01; not re-tested by query.*

Unconfirmed: the response when the external user does not exist yet (expected 404). Create the user with the POST handoff above, which creates if missing.

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

### Draft orderlines

An orderline can be a draft. The API orderline carries a `draft` boolean, Liquid has `orderline.is_draft` (`50_LIQUID_REFERENCE.md`), and the admin shows "(Draft)" next to the orderline, which is easy to miss. Check it before treating a line as a normal production line. Platform-level. *Stated by the core developer, Slack #development, 2026-10-22.* Which flows create a draft orderline was not stated.

---

## 9. Dynamic Design Previews

Preview a design (theme) without creating a project.

**URL pattern:**
```
/v1/themes/<theme-id>/preview.<ext>?<query-params>
```

Supported extensions: `jpg`, `webp`, `png`, `svg`

SVG is recommended for performance and sharpness at small sizes, but cannot be used in `<img src>`. Use `<object type="image/svg+xml" data="...">` instead.

### Query parameters

| Parameter | Description |
|---|---|
| `width` | Output width in px. Without it `preview.webp` returned 150 x 150 (*verified by query, 2026-10-19 and 2026-10-21*). `size`, `max_size`, `s` and `dimension` are ignored: only `width` sizes the render. (Corrected 2026-10-22: previously "default 100", from the original API docs; other extensions without `width` not re-tested.) |
| `height` | Output height in px |
| `hide_placeholders` | `true` hides empty photo placeholders **and placeholder text elements** (a filled `name` text placeholder disappeared too). `false` shows both; use it whenever the customer's text must show. The theme JSON lists a design's storefront preview as `/v1/themes/<id>/preview.webp?hide_placeholders=false`, so listings show placeholder images, not empty frames. *Verified by query, 2026-10-18 and 2026-10-20.* |
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

What the theme preview shows (platform-level):
- **The render is the template's preview set** (often a mockup scene), not the bare page. Trim the white margin if you need just the card. *Verified by query, 2026-10-19.*
- **An empty photo placeholder renders as flat RGB 210, 208, 208 with a black picture icon in its center** with the platform's default placeholder image. *Corrected 2026-11-09: this said the color was always 210, 208, 208; the placeholder image is per site, and a site with its own placeholder (a light image with drag-and-drop text) rendered as flat 215, 215, 215 (verified by query, 2026-10-31).* A tool that finds the empty photo area by color must not hard-code one grey: fall back to the most common flat light grey in the render when 210, 208, 208 is absent. On a photo-led design the 150 px `preview.webp` is therefore almost blank white, with or without `hide_placeholders=false`, and is not a usable design tile: a design picker needs its own tile image per design (the collection image, or a per-design map in the tool's configuration). *Verified by query, 2026-10-18 and 2026-10-21.* On a design that is not photo-led (an ornament or a mug product), `/v1/themes/<design id>/preview.webp?product_id=<product id>&height=600` returned a rendered preview usable as a landing card image: the blank-tile problem is the default 150 px size plus empty photo placeholders, not the endpoint. *Verified by query, 2026-11-06.*
- **Checking that an image upload option reaches the design:** request `/v1/themes/<design id>/preview.jpg?product_id=<id>&width=600` with and without `template_options[<option code>]=db:<image id>`, in the **uploader's own browser session** (uploads are session-scoped). The same byte size both times means the option is not applied, typically because no `<image>` element carries the option's `target_element_name` (`22_OPTION_VARIANT_RENDERING.md`). *Verified by preview byte comparison, 2026-10-21.*

### A design's layouts
```
GET /v1/themes/<id>/layouts.json
```
Returns the design's layouts with the page XML in `data` (count the `<image` elements for the number of frames) and their tags, such as "1 Photo" to "6+ Photo". Anything that auto-fills a book from layouts depends on this list: a design with 0 layouts cannot be auto-filled. Platform-level. *Verified by query, 2026-10-21.*

### Project previews
```
https://<subdomain>.pixfizz.com/v1/books/<project-id>/preview.webp?width=800
```
Requires admin access.

- **`/v1/books/<id>/preview.<ext>` ignores `template_name`** and returns the first page. Select a page with `page=<0-based index>`, which also reaches the pages of a `fulfillment="false" editor="false"` preview set. *Verified by query, 2026-10-07.*
- It takes **unsaved** choices as `book[template_options][<code>]` (the keys `px-option-selector` `values()` returns), so a preview can follow the customer before Save. The theme endpoint above takes the same choices as `template_options[<code>]` and ignores the `book[...]` form. *Verified by query, 2026-10-07.*
- `px-project-preview` supports `page-number` (1-based) and `preview-section`, and fetches `/v1/books/<id>/preview.<fmt>` with the values of its `option-selector`. *Verified by reading source (cms bundle 20261006102509).*
- A transparent PNG render keeps mask holes transparent, so a shape or drilled holes can be read from the alpha channel.

### Preview resolution and production-quality output

- The theme and project preview endpoints above are optimised for on-page previews, not
  print. Output is **capped at `width=1200`** and rendered at a lower JPEG quality.
- **The cap is on the longest side, not the width.** `width=3000` on a portrait 8x10 returned 966 x 1200. *Verified by query, 2026-09-26.*
- **`template_name=<page>` renders the whole page including bleed.** An 8x10 page with `bleed="0.125"` (8.25 x 10.25 in the definition) renders at 400 x 497 for `width=400`; `crop=false` made no difference. A tool that shows the render at trim must crop it itself. *Verified by query, 2026-09-26.*
- For higher-quality, production-resolution output, render the page directly:
  `/v1/pages/<page-id>.jpg?width=X&fulfillment=true`. This uses the production render
  settings rather than the preview pipeline.
- The `fulfillment=true` page endpoint is currently **superadmin (omnipotent) only**;
  opening it to all admins is under consideration. Confirm current access before relying on it.
- **Checking every page of a design without an order** (*verified by query from an `admin.pixfizz.com` session, 2026-10-20*; whether a non-staff site admin may call it is still the open question above):
  - `/v1/pages/<print page id>.jpg?fulfillment=true&sitename=<site>&width=<px>` on the admin host returns the print render scaled to that width. `size` and `max_size` are ignored.
  - **The fulfillment render leaves placeholders out.** A text element with `placeholder="true"` does not appear until the customer edits it, and an empty photo placeholder prints nothing. To review a design with its sample photos and text, use the plain `/v1/pages/<id>.jpg` (no `fulfillment`).
  - **A stale mask id answers 500.** An image whose `mask="db:<id>"` points at an image the site does not have makes the print render answer 500 for that page, while the editor preview just drops the mask. When merging design exports, renumber `mask=` ids as well as `src=`. *Verified by query, 2026-10-20.*
- **`fulfillment` on the theme/project preview endpoint trades placeholders against
  resolution.** With `fulfillment=false` (preview pipeline) elements set to "show on
  preview" ARE rendered, but output is still capped at `width=1200`. With
  `fulfillment=true` the request renders at the full requested width but "show on
  preview" placeholder elements are **stripped**. There is currently no way to get
  both full resolution and "show on preview" placeholders from this endpoint — they
  are architecturally coupled. This parameter is not officially documented on the
  preview endpoint; confirm with the platform team before building on it, and treat a
  raised/removed 1200px cap as a feature request.

### Transparency and the page mask

- `/v1/themes/<id>/preview.webp` and `preview.png` return the page with its transparency (alpha 0 outside the design). `preview.jpg` fills it white. `preview.png?template_name=<page>&product_id=<id>&width=<w>&template_options[<code>]=<value>` takes the same parameters as `preview.jpg` and returns RGBA: everything outside the page mask is fully transparent and semi-clear artwork keeps its alpha, so a tool can read a product outline from it. The long-side cap of 1200 px applies (900 wide on a portrait page returned 869 x 1200). *Verified by query, 2026-10-18.* A tool that needs to see through a page (clear acrylic, crystal, cut shapes) must ask for WebP or PNG.
- `/v1/themes/<id>/preview.svg?template_name=<page>&product_id=<id>` returns the page as SVG. The page mask is `<mask id="page-mask-N"><image href="https://cdn.pixfizz.com/fz/.../mask_*.png">`. The CDN image loads with `crossOrigin="anonymous"` and can be read into a canvas. This is the only storefront-side way found to get a page mask as an image.
- In the SVG render, element images appear with their CDN URLs, so a full-page image (width and height equal to the viewBox) is the page background.
- On clear products the print mask can be smaller than the physical piece; see `19_XML_TEMPLATE_REFERENCE.md` § Preview Sets.

*Verified by query on a client site, 2026-09-29 (a definition with `output="png" background-transparent="true"`).*

### Render speed: WebP is the fast format

The design render answers as `preview.png`, `preview.jpg` and `preview.webp` with the same query (`template_name`, `width`, `product_id`, `template_options[...]`, `variants[...]`, `hide_placeholders`). Measured cold at 900 px on one page (*verified by query on a client site, 2026-10-31*):

| Format | Time | Size |
|---|---|---|
| PNG | 2.2 to 2.5 s | 280 to 440 KB |
| WebP | 1.3 to 1.5 s | 19 to 46 KB |
| JPEG | about 1.1 s | about 130 KB |

- PNG time barely changes with width (400 px 2.5 s, 1200 px 2.6 s): the encoding, not the size, is the cost.
- Responses are `Cache-Control: private`. The same URL from the same browser comes back in about 0.9 s; nothing is shared between shoppers.
- Use WebP for any opaque preview (a full-bleed print); keep PNG only where transparency must be exact (outlines, a split front layer). WebP also carries alpha (§ *Transparency and the page mask*).
- Under load one cold PNG took 11.9 s. Never have two identical renders in flight: two requests for the same URL both waited the full time.

Platform-level.

### Reading a design and a font

- `GET /v1/themes/<id>.json` is readable without admin from the storefront (200 for the site's own designs; `17_DESIGN_TOOL.md` notes 403 for another site's). It returns every design page's XML (`templates[].print_page.data`), the template options with their price formulas, and the XML definition (`print_product.layout`).
- `GET /v1/fonts/<id>.json` returns the font name and file URL.
- `GET /v1/font_palettes/<palette id>.json` (the palette id comes from a font option) returns each font with `name`, an `.svg` preview and the `.ttf` URL on the site host, so a tool can load the real fonts with `FontFace` for font tiles. One font in a palette failed OTS decoding in Chromium: fall back to the body font when `FontFace.load()` rejects. *Verified by query, 2026-10-26.*
- In the page XML from `/v1/themes/<id>.json`, the page size is in mm and a photo element carries its geometry (`x` or `left`, `y` or `top`, `width`, `height`, `radius`): enough to set a crop ratio for the photo window without a render. *Verified by query, 2026-10-26.*
- A preview with `template_options[<text option>]=%20` (a single space) renders that text blank; an empty value keeps the placeholder text. *Verified by query, 2026-10-26.*

*Verified by query, 2026-10-03.*

- The theme JSON's `images[]` carry `url` and thumbnails at 150, 250, 600, 900 and 1200 px, and each page's XML gives element positions, placeholders and editable text, so a design's data can be read in bulk from the storefront. Its `options[]` include `target_element_names`. In admin, `/site/<site>/admin/print_theme/product/<template id>` lists a template's designs as `px-design-preview` elements with a `design-id` attribute. *Verified by query, 2026-10-19 and 2026-10-20.*

- In `/v1/themes/<id>.json` an unpublished template option keeps `published: true` on its values (`options[].published=false`, `values[].published=true`). Check the option's flag, not only the values'. Option-level custom fields come back as `options[].custom`, unpublished options included.
- `/v1/products/<id>.json` returns the product's custom fields and variants, not its template options; template options (with their value custom fields) come only through `/v1/themes/<design id>.json`.
- `print_product.layout` carries the XML definition, so `min`, `max` and `add` on `<definition>` are readable client-side.

*Verified by query, 2026-10-14.*

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

> **Two observations disagree (2026-10-16).** In a signed-in storefront session on a test site, `POST /v1/calendars.json` with only `calendar[name]` (no `code`) created a calendar **owned by the session user**, not a site-wide one, and `GET /v1/calendars.json` listed website calendars only. Dates were then added with `POST /v1/calendars/<id>/dates.json` (`date[caption]`, `date[start]` as YYYY-MM-DD, `date[recurring]` 1 or 0, `date[date_type]`), the calendar id was passed in `book[calendars][]` on project create (§ 4), and `fill:user-calendar` captions rendered. *Verified by query, 2026-10-16; signed-out not tested.* Whether the caller's role decides between a user and a site-wide calendar on this route is not confirmed with the core developer: until it is, create user calendars on `/v1/users/<id>/calendars.json` and read back the owner.

**Admin calendar forms (alternative to `/v1/calendars.json`).** A holiday calendar can also be built with two admin forms: `POST /site/<site>/admin/calendars` with `calendar[name]` and `calendar[code]`, then one `POST /site/<site>/admin/calendars/<id>/create_date` per date with `calendar_date[caption]`, `calendar_date[start(1i)]`, `calendar_date[start(2i)]`, `calendar_date[start(3i)]` (year, month, day), `calendar_date[date_type]` (free text, empty is fine) and the `calendar_date[recurring]` 0/1 checkbox pair. 37 dates were created in one script run. To copy a calendar between sites, read each date from `/site/<source>/admin/calendars/<id>/edit_date?calendar_date_id=<n>`, create the calendar on the destination, then post each date. A tool that reads a holiday calendar on a Shopper child reads the **storefront site's** calendar, while products come from the products parent (`13_TEMPLATE_BOUNDARIES.md` § Moving a Shopper Child Onto Another Admin). Platform-level. *Verified by query, 2026-11-06.*

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

**What a user calendar date holds** (*verified by query on a test site, signed in as the calendar's owner, 2026-10-29*):
- `GET /v1/calendars/<id>.json` and `GET /v1/calendars/<id>/dates.json` work for the signed-in user's own calendar when the id is known; `GET /v1/calendars.json` still lists only website calendars (§ *Admin calendars*).
- A date is `id`, `caption`, `date_type`, `start`, `recurring`.
- **`date_type` is free text, stored as sent, at most 64 characters**; longer answers 422 "is too long (maximum is 64 characters)". Composite values such as `birthday:cake:age` or `b;s=p;i=123456789;c=5,9,205,0;a=1` are accepted, so a tool can keep its own per-date data there.
- **`start` keeps the full date including the year** (`2019-05-18` with `recurring` true stays `2019-05-18`), so a birth year can be stored in it.
- Update: `POST /v1/calendars/<id>/dates/<date id>.json` with `_method=put`; delete: the same URL with `_method=delete` (200, empty body; the date leaves `dates.json`). No CSRF token was needed from a storefront page.

**What prints** (*verified by render of a production PDF generated from the admin project page, 2026-10-29*):
- A recurring date with a past start year prints on the later calendar year (a 2019 birthday printed under the same day of 2027).
- A composite `date_type` changes nothing in the render: the caption prints as normal.
- A transparent PNG uploaded into the project gallery prints with its transparency on a day cell, and an image element added to a calendar grid page by editor script prints. Editor scripting detail: `23_XML_CALENDAR_REFERENCE.md`.

---

## 12. Colors and Fonts (read-only)

```
GET /v1/colors.json
GET /v1/fonts.json
```

Colors and fonts are organized into palettes. Individual colors/fonts cannot be added, updated, or removed via the API — only read. Each palette has an `id`, `name`, and an array of items.

`GET /v1/fonts.json` is public and returns `[{id, name, preview, url}]`, one entry per site font, not the palette shape described above (which of the two the older text describes, a palette route or an earlier response, is not confirmed). Names carry the weight and style ("Playfair Display Italic", "Josefin Sans SemiBold"), so a script can find a site font id by name. Platform-level. *Verified by query on the headline demo site, 2026-10-21.*

**On a Shopper child, `/v1/fonts.json` lists the fonts of its products parent, not its website parent.** A child whose website parent is the Shopper 24 parent (17 fonts) returned 0 fonts until fonts were added on its products parent, and then listed exactly those. A tool that resolves fonts by name needs its fonts installed on the products parent. Platform-level. *Verified by query, 2026-11-06.* Copying a font between sites: § 13h *Fonts*.

---

## 13. Fulfillment Partner Callbacks

Pixfizz accepts inbound shipping/status callbacks from fulfillment partners at:
```
POST https://login.pixfizz.com/custom/<partner>/order_callback
```
Requires HTTP basic auth. Supported partners: Advertek, Navitor, Gooten, PRNTMSTR, Siteflow.

Callback payloads carry shipment status, tracking name, tracking code, tracking URL, and package IDs. The payload schema varies by partner.

**Note:** Callback endpoint credentials are operational secrets and are not documented here. Retrieve from the Pixfizz Notion wiki (Callbacks from Fulfillment Partners page) or contact support.

### SiteFlow status callbacks

SiteFlow also has its own endpoint on the site host: `PUT https://<site-domain>/v1/orders/siteflow_update.json` (POST is also accepted), with HTTP Basic auth for an admin user. Parameters: `SourceOrderId` (the Pixfizz order code), `OrderStatus` (`received` sets D, `error` sets E, `shipped` sets S; **lowercase only**, a capital `Shipped` fails), `TrackingURL`, and `TrackingNumber` (saved to an order custom field). *Verified by reading the partner email history, 2022 to 2023; not re-tested.*

- A SiteFlow trigger cannot use a dynamic URL, so one callback URL serves every order it sends. Since February 2023 a callback can update orders on any site in the same super account. Which fixed domain is current, and whether SiteFlow uses the `login.pixfizz.com/custom/siteflow/order_callback` route above, are not verified.
- **A callback cannot update orders across different super accounts today.** A print-on-demand partner serving sites in many client accounts has no credential that covers them all; a platform change is needed. *Stated by the core developer, 2026-10-06.* Until it ships, such orders reach the partner without a postback and their Pixfizz status does not update automatically.
- A SiteFlow order can carry `orderData.postbackAddress` per order, which would let a job ticket give each site's own domain. Not verified.

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

> **Update 2026-09-23: the retirement is live.** Every unofficial `/admin/...` path now redirects cross-host to `admin.pixfizz.com/site/<site>/admin/...`, which needs an interactive session. **A cross-origin redirect drops the `Authorization` header**, so a Basic-auth client that follows it gets a 401 that looks exactly like a wrong password. Confirmed intentional and permanent by the core developer. A person clicking an `/admin` link in a browser is unaffected; the break is for server-to-server clients. Every server-to-server call must use `/v1/admin/...` and send `redirect: "manual"`, treating any redirect as its own failure. *Verified live and by query, 2026-09-23.*
>
> **`/v1/admin/...` is not a public integration surface.** External integrations use the standard `/v1` order endpoints (look up by order id); the admin namespace is for Pixfizz and site-admin tooling. *Stated by Alex, 2026-09-22.*

> **The `/admin/...` JSON endpoints are being retired.** The core developer stated on the
> Notion Dashboard (week of 2026-09-21) that every unofficial `/admin/...` endpoint **will stop
> working when the current staging code is deployed to production**. Replacements live under
> `/v1/admin/...`. Confirmed replacements so far:
>
> | Retiring | Replacement |
> |---|---|
> | `/admin/custom_types` | `/v1/admin/custom_types` |
> | `/admin/custom_types/<id>/custom_type_instances` | `/v1/admin/custom_types/<id>/custom_type_instances` |
> | `/admin/assets` | `/v1/admin/assets` |
>
> `PUT /admin/theme_categories/<id>.json` (collections, below) has **no confirmed replacement
> yet** — pending confirmation. Any script, tool or integration that calls an `/admin/...` path
> must be moved to `/v1/admin/...` before that deploy, or it breaks silently on the day. Write
> new code against `/v1/admin/...` only. Stated by the core developer; live since 2026-09-23
> (see the update above).

### Custom types

```
GET    /v1/admin/custom_types.json
GET    /v1/admin/custom_types/<id>.json
GET    /v1/admin/custom_types/<id>/custom_type_instances.json
POST   /v1/admin/custom_types/<id>/custom_type_instances.json
GET    /v1/admin/custom_types/<id>/custom_type_instances/<instance-id>.json
PUT    /v1/admin/custom_types/<id>/custom_type_instances/<instance-id>.json
DELETE /v1/admin/custom_types/<id>/custom_type_instances/<instance-id>.json
```

The `/admin/...` forms of these paths are retired (they redirect, see above). **Custom type instances are full CRUD on `/v1`** (Corrected 2026-10-06; the earlier text listed only list, read and create). Verified by query on 2026-10-01 against a `pages` custom type: an instance was created, read, updated with both `POST` + `_method=put` and a real `PUT`, deleted, and read back as gone. All calls were multipart form data with an admin browser session, `redirect: 'manual'`, no CSRF token needed.

- `GET /v1/admin/custom_types.json` returns an array of `{id, name, code}`. The instance list returns an array of `{ id, custom_type_id, custom: {...} }` (verified live, 2026-09-23).
- Create and update return the instance, with its id.
- **A real `PUT` works on `/v1`.** The "raw PUT is blocked, use `POST` with `_method=put`" rule was true on the retired `/admin` path. Both forms work on `/v1`, so code that already sends `_method=put` needs no change. The cross-origin preflight rule below still applies from a browser on another origin.
- **Update is a merge, not a replace.** Sending only `custom_type_instance[custom][page_title]` left the instance's other fields untouched, so a partial update is safe and needs no read-modify-write of the whole `custom` hash. To clear a field, send an empty string for it.
- **A JSON body must be wrapped:** `PUT .../custom_type_instances/<id>.json?sitename=<site>` with `{"custom_type_instance":{"custom":{...}}}`. A bare `{"custom":{...}}` returns 200 and changes nothing. *Verified by query, 2026-10-14.*
- **Custom field values have a size limit.** A write with a large `page_content` is refused with HTTP 422 `{"custom_fields":["are too large"]}`. A 58,208-byte value saved and a 60,852-byte value was refused, so the limit is near 60 KB. Strip comments and indentation, or move script and CSS into assets. *Verified live, 2026-10-14.*
- **Delete returns 200 with an empty body**, not 204 and not a JSON envelope. A client that parses every response as JSON throws on a successful delete. After the delete, a `GET` of the instance returns 404 `{"error":"Not Found"}` and it is gone from the list.
- After a delete, the storefront page for that instance (a `pages` instance) also returns 404. *Verified by query, 2026-10-04.*
- **Admin form route (no `/v1`):** create by POSTing the form on `/site/<site>/admin/custom_types/<type>` whose action ends in `/custom_type_instances` (`authenticity_token` only); the response URL ends with the new instance id. There is no `/new` route (404). To edit through admin, note that the instance page renders the Pages type's `page_content`, `page_description` and `page_schema` as Ace editors client-side, so a plain fetch of the page does not contain those textareas. Load the page in a hidden same-origin iframe, wait for it to render, select the instance form by a field it must contain (`custom_type_instance[custom][...]`) and check that its `_method` is `patch`, never by its action URL alone (the Save and Delete forms share the action URL, so an unchecked `new FormData(form)` can submit the Delete form: `41_IMPLEMENTATION_PATTERNS_UPDATED.md` § Scripted Admin Writes: Pick the Form by a Field, Never by Its Action URL; amended 2026-10-22), build `new FormData(form)` from that form, set `custom_type_instance[custom][<field>]`, and POST to the form action; every other field goes with its current value, so nothing is wiped. Booleans: delete both entries and append one `'1'` or `'0'`. Stored text comes back with CRLF. *Verified by query, 2026-10-05.*
- Moving custom type definitions and instances between sites: `13_TEMPLATE_BOUNDARIES.md`.
- Still manual: custom field **definitions** have no API. (Extended 2026-10-17: they can be created from a logged-in admin tab through the admin form. The custom type page's "edit schema" link opens `/site/<site>/admin/custom_field_definitions?instance_id=<type id>&type=CustomType`; the new form is `/custom_field_definitions/new` with the same query, fields `custom_field_definition[field_name]`, `[field_type]` (`text`, `multitext`, `boolean`, `number`, `asset`, `snippet`), `[description]` and `[public]` (hidden `0` plus checkbox `1`). POST `FormData` to the form action. `/custom_types/<id>/edit` returns an application error, and a definition has no `/edit` route: its show URL holds the form. *Verified by use, 2026-10-13.*) A new site needs the custom type and its fields created in admin (or imported, `18_ADMIN_NAVIGATION.md` § Custom Fields, Schema Order and Bulk Export/Import) before any instance can store a value, and a value written against a field that does not exist is silently dropped.
- **Copying definitions per object between sites by script.** `GET /site/<source>/admin/custom_field_definitions/export_all?type=<Type>` on the source returns the per-object archive; POST it to the hidden import form on `/site/<target>/admin/custom_field_definitions?type=<Type>` on the target. The object is chosen by the `type` on the import screen, not by the archive (`18_ADMIN_NAVIGATION.md` § A custom field definition archive has no object type in it). **A brand-new Shopper child started with 0 definitions on every object**, so before writing collection fields such as `pdp_layout` or `collection_filters` copy the definitions for `ThemeCategory`, `PrintTheme`, `Product`, `TemplateOptionType`, `TemplateOptionValue`, `VariantType`, `VariantValue`, `PrintProduct`, `Order`, `Orderline` and `User` from an established Shopper 24 site. Platform-level (definitions are per site). *Verified by query on a new child, 2026-11-06.*
- **Custom types export by `custom_type_ids[]`, not `ids[]`:** with `ids[]` the export is empty. Instances move the same way through `/custom_types/<id>/custom_type_instances/export_all` and `.../custom_type_instances/import` (the YAML list can be filtered before import). Full route notes: `13_TEMPLATE_BOUNDARIES.md`. *Verified by query, 2026-11-06.*

Create parameters, one per custom field on the type:

```
custom_type_instance[custom][<custom-field-name>]
```

The custom type `<id>` is numeric and site-specific. Fetch the list endpoint on the target site to find it rather than reusing an ID from another site.

### Assets

```
POST /v1/admin/assets.json     # multipart encoded
GET  /v1/admin/assets.json     # list all assets
```

`GET /v1/admin/assets.json` lists every asset with its signed `/fz/` URL (verified by query, 2026-09-26). **The multipart upload works on `/v1`** (Corrected 2026-10-06): `POST /v1/admin/assets.json?sitename=<site>` with `asset[name]`, `asset[description]`, `asset[file]` and an `X-CSRF-Token` header taken from any `authenticity_token` input on an admin page returned 200 for each of 16 files (WebP included) from a logged-in admin session. *Verified by query, 2026-10-05.* Server-to-server upload with an API key and no session is not verified.

**Asset names are unique per site.** On `/v1/admin/assets.json` a duplicate name returns **HTTP 200** with `{"error":{"name":["has already been taken"]}}`. Check the body for an `id`, not the status. *Verified by query, 2026-10-04.* The admin-host route (`POST /site/<site>/admin/assets.json`) answers a duplicate `code` with **HTTP 422** "Name has already been taken" (*verified by use, 2026-10-13*). List the site's assets by name before uploading.

**File types.** A `.glb` is refused: `POST /v1/admin/assets.json` answers **HTTP 200** with `{"error":{"name":["has incorrect filetype"]}}` (check the body, as for duplicate names). A `.json` file is accepted and `cdn.pixfizz.com` serves it gzip-compressed as `application/json` with CORS open, so binary data such as a 3D model can ship as JSON (`{ "format": "glb-base64", "data": ... }`) and be decoded with `atob`. *Verified by query, 2026-10-20 (test asset created and deleted).* A `.js` asset is served from `cdn.pixfizz.com` as `text/javascript` with CORS, so `import(<asset url>)` loads it as an ES module from a Shopper page. JS responses carry no `Content-Length` (images do; on-the-fly compression is inferred, not verified). *Verified by query of headers, 2026-10-21.* Platform-level. Model packaging for previews: `27_LIVE_FINISH_AND_3D_PREVIEWS.md`.

**An `.html` asset is refused silently.** Through the admin-host form route (`POST /site/<site>/admin/assets` with `authenticity_token`, `asset[name]`, `asset[description]`, `asset[file]`) an `.html` file answered 200 and nothing was stored; `.js`, `.webp` and `.png` uploaded the same way. List the assets after uploading. *Verified by query, 2026-11-06.* A whole HTML page (for example a clickable prototype) can still ship as a `.js` asset that sets a string (`window.X_HTML = "<!DOCTYPE html>..."`), rendered by a Pages instance into an `<iframe srcdoc>`, with its images as separate assets whose URLs are passed in with Liquid `asset_url`. Give the iframe the full viewport height below the header so the page's own modals stay on screen. *Verified by query on a demo child, 2026-11-06.*

**The CDN copy can differ in size from the uploaded file.** Two alpha WebP files uploaded as assets came back from `cdn.pixfizz.com` with an extra 5,759-byte C2PA chunk appended; the image chunks were byte-identical. Harmless, but compare image data, not file size or whole-file hash, when checking an upload. *Verified by query, 2026-10-21.*

**`?sitename=` is ignored on a storefront origin.** `POST /v1/admin/assets.json?sitename=<site>` sent from a storefront origin writes to that storefront's own site (a name collision error if the file already exists there). To write to another site, such as the parent, use the admin-host route `POST /site/<site>/admin/assets.json` (field `data`, `X-CSRF-Token`). *Verified by query, 2026-10-15.*

**The admin upload stops at 50 MB.** Uploading a file in the Pixfizz admin refuses anything over 50 MB with a browser alert, "Some files are too large. Maximum allowed file size is 50.0 MB." A custom tool's own size setting governs only the customer-side upload inside the tool on the storefront; nothing in Liquid, a template option or a custom field raises the admin ceiling. Platform-level. *Seen 2026-10-26 with an 84-page PDF.* Not verified: which admin screens apply it, and whether the server applies the same ceiling to `/v1/admin/assets.json`.
- **The 50 MB is a browser-side check in the admin bundle**, `MAX_FILE_SIZE_MB(){return Px.config.large_format?150:50}`, so a site whose admin config carries `large_format` already gets 150 MB. No admin or Super Admin screen that sets `large_format` was found (site settings, the Super Admin website form and website settings checked). *Verified by reading source (admin bundle build 20261029164200), 2026-11-02.* On a client site's admin Assets page `Px.config` was empty, so 50 MB applied. *Verified by query, 2026-11-02.* (Amends "nothing raises the admin ceiling" above: nothing a site admin can set raises it.)
- The core developer agreed to raise the admin asset upload limit (a request was made for 200 to 250 MB for large publication PDFs); not live, no date. *Stated by the core developer on a call, 2026-11-02.* Not verified: whether the server enforces its own limit above the browser check.

**Large files: `/v1/admin/assets.json` has a body size limit.** A server-to-server `POST /v1/admin/assets.json` (API key, sent to a site's custom domain) with a 12.6 MB PDF returned HTTP 413 "Request Entity Too Large" from nginx with an empty body; a 6.46 MB PDF through the same route succeeded. The same 13 MB file uploaded fine as an asset through an admin browser session on `admin.pixfizz.com`, so the limit is on the API / storefront host path, not on assets as such. *Verified by query, 2026-10-15.* Not verified: the exact limit (somewhere between 6.46 MB and 12.6 MB), and whether `/v1` on `admin.pixfizz.com` has the same one. A server-side tool that stores print-size PDFs should fall back to the orderline file URL on 413.

**`/fz/` URLs are signed for the host that issued them. Never rewrite the host.** An admin-page link `https://admin.pixfizz.com/fz/...` downloaded with no session (`credentials: 'omit'`, CORS open from the storefront origin); the same path on the storefront host returned 403 "invalid", and on `cdn.pixfizz.com` it failed. *Verified by query, 2026-10-10.* Signed `/fz/` URLs do not appear to expire: one issued on the admin host still downloaded with no session five days later (206, `application/pdf`). Observed once; not a documented guarantee. *Verified by query, 2026-10-15.* The same holds for an image asset: its full-size link on the admin host (`https://admin.pixfizz.com/fz/<hash>/0/<token>/~/<file>`) loaded with credentials omitted (200, `image/webp`, exact byte size), so it can be stored as a public image URL. The links are on the admin Assets list pages (50 per page, `td.image a[href]`). *Verified by query, 2026-10-20.*

**A `/fz/` URL issued on a storefront host loads cross-site.** (Added 2026-10-29; consistent with the rule above, because this URL was issued by the storefront host, not rewritten to it.) The admin-host upload `POST /site/<site>/admin/assets.json` returns `url` on the site's **storefront** host (`https://<site>.pixfizz.com/fz/<hash>/0/<token>/~/<name>`), and `GET /v1/admin/assets.json?sitename=<site>` lists the same URLs. Fetched from another Pixfizz site with credentials omitted, that URL answered 200 for WebP, JSON and WOFF2, `FontFace` loaded from it, and CSS background and mask images worked. The same path rewritten onto `cdn.pixfizz.com` or `admin.pixfizz.com` failed cross-origin, and from a page on `admin.pixfizz.com` the storefront-host URL also failed (fetch it from a storefront tab instead). *Verified by query, 2026-10-25.* So one site's assets can serve as a library for every site: upload the files, build an index `{name: url}` from the asset listing, upload the index as JSON and give tools the index URL. Replacing a file in place gives it a new URL, so rebuild the index after any replacement. The admin Assets Upload button takes many files at once (388 in one go, *verified by use, 2026-10-25*).

**`asset_url` is hashed per file.** `{{ 'name.webp' | asset_url }}` returns `https://cdn.pixfizz.com/fz/<hash>/0/<token>/~/name.webp` and the hash differs per file, so a tool cannot derive one asset's URL from another's. A tool with many images needs a map of `asset_url` lookups rendered in Liquid. Those `cdn.pixfizz.com` responses carry `access-control-allow-origin: *`: load with `img.crossOrigin = 'anonymous'` and a canvas stays clean for `toBlob` and `getImageData`. *Verified by query, 2026-10-26.*

The upload returns 200 with `{ id, name, url, is_image, previews: { thumb, small, medium } }`; `url` is a public `/fz/` address. *Verified by query, 2026-09-30.*

```
PUT    /v1/admin/assets/<id>.json     # asset[file]: replaces the file, same id and name, new URL
DELETE /v1/admin/assets/<id>.json     # 200, empty body; a GET then returns 404
```
*Verified by query, 2026-09-30 and 2026-10-04.*

**Resize an asset in place from a storefront tab.** Fetch the asset URL, draw it on an `OffscreenCanvas`, `convertToBlob('image/webp')`, then `PUT /v1/admin/assets/<id>.json?sitename=<site>` with `asset[file]`. The name stays the same, so `asset_url` follows the new file. The URL changes, but the old `/fz/` URL keeps answering, so cached pages do not break. *Verified by query, 2026-10-26.*

An asset-type custom field (for example a blog image field) takes the asset **name** returned by the upload, never a URL; the storefront renders it through `asset_url`.

Admin-host upload, without `/v1`: `POST /site/<site>/admin/assets.json` multipart returns `{id, name, url, previews}`. Two field sets were seen working: `asset[name]`, `asset[file]` and `authenticity_token` (verified by query, 2026-10-06), and `code` (a unique key; any uuid worked) plus `data` (the file) (verified by query, 2026-09-30). The `code` plus `data` upload also answered without an `X-CSRF-Token` header from an admin tab on 2026-10-26, while the 2026-10-25 run sent one; send it anyway. The field name `file` answers 500. *Verified by query, 2026-10-25 to 2026-10-27.*

**Replace an asset in place** (same id, same name, so every `asset_url` reference follows the new file): `PUT /v1/admin/assets/<id>.json` above (Corrected 2026-10-06: this section previously said there was no `/v1` call), or the admin form: `GET /site/<site>/admin/assets/<id>/edit`, then POST that form as multipart with `_method=patch`, `asset[name]`, `asset[description]` and `asset[file]`. The file can be built in the page as a `Blob` or `File`, so no file picker is needed. To patch an existing asset, fetch the current file from `cdn.pixfizz.com` (CORS is open there; the storefront host is not), change it, and check its hash against the tested file before posting. *Verified by query, 2026-10-05.* **Two observations disagree:** on 2026-10-10 the cdn and storefront links both failed CORS from an admin page; what worked was the signed admin-host link on the asset's edit page (`https://admin.pixfizz.com/fz/...`), fetched from that admin page. Try the cdn first and fall back to the signed link. *Verified by query, 2026-10-10.* `FormData` turns LF into CRLF in text fields, which is harmless but makes a byte compare of sent against stored differ for that reason alone. See § 13h for the other admin-form writes.

Upload parameters:

```
asset[name]
asset[description]
asset[file]        # multipart-encoded file
```

The upload response returns the created asset IDs. Where a custom type instance needs to reference an image, upload the asset first, take the ID from the response, then create the instance referencing it.

### Design previews (linked assets) and descriptions

*Verified by query, 2026-10-03 to 2026-10-04.*

- List a design's preview images: `GET /v1/admin/themes/<id>/linked_assets?sitename=<site>`.
- Set them: `PUT /v1/admin/themes/<id>?sitename=<site>&mapped_previews=false&theme[asset_ids][]=<a>&theme[asset_ids][]=<b>`. Send the full list in order: it replaces the existing links. The Shopper collection card shows linked asset 1 as the image and asset 2 on hover.
- Product image: `PUT /v1/admin/products/<id>.json` with `product[image]=<asset name>` works and changes only `image` and `image_url`.
- Product description: the API write saves nothing (§ 13g). The admin product form, submitted with only `product[description]` changed (`new FormData(form)`, empty File entries dropped), saves only the description.
- Design description: POST the design form (`/site/<site>/admin/print_theme/theme/<id>`, `theme[...]` fields) with only `theme[description]` changed. The Shopper design product page shows the design description, not the product description.

### Updating custom fields on existing objects

Custom fields are written through the parent object's update endpoint, using the object's own parameter key. The key is not always the name you would expect from the admin UI — a Design is `theme`, a Collection is `theme_category`.

**Product attributes**

```
PUT /v1/admin/products/<product-id>.json

product[custom][custom_field_1]=value1
product[custom][custom_field_2]=value2
```

**A `multitext` field must be sent in array form.** `product[custom][<multitext-field>]=Metal` returns 200 and saves an empty list, `[]`. Send `product[custom][<multitext-field>][]=Metal` (repeat the parameter for each value); the read back is `["Metal"]`. *Verified by query, 2026-10-05.* Only the keys sent change: other custom fields and the variants are kept (§ 13g).

**Designs**

```
PUT /v1/admin/themes/<design-id>.json

theme[custom][custom_field_1]=value1
theme[custom][custom_field_2]=value2
```

> **This saves nothing** (Corrected 2026-10-06). `PUT /v1/admin/themes/<id>.json` with `theme[custom][...]`, sent from the admin host, returned 200 and stored no custom field (verified by query, 2026-10-03). Write design custom fields with the admin form, `PATCH /site/<site>/admin/print_theme/update/<design id>` (§ 13h). The same endpoint does set a design's linked assets (below).

**Collections**

```
PUT /admin/theme_categories/<collection-id>.json

theme_category[custom][custom_field_1]=value1
theme_category[custom][custom_field_2]=value2
```
> **Status 2026-09-23:** this `/admin/...` path is retired with the rest; it redirects cross-host, so a server-to-server call fails. No `/v1` replacement for writing collection custom fields is confirmed yet. Do not build on this endpoint until one is. *Pending confirmation with the core developer.*
>
> **Update 2026-10-06:** there is still no `/v1` write for collections. From a logged-in admin session, collection custom fields, the collection image and the collection's product order are written through the admin's own forms on the collection page. Routes in § 13h. *Verified by query, 2026-10-05.*
>
> **Update 2026-10-12: an API key cannot put a product in a collection.** With a `pxk_` key (Basic auth, `redirect: manual`) every attempt returned 404 or was ignored: `add_products` on the storefront host, on the admin host and under `/v1/admin/theme_categories/<id>/add_products.json`; `POST /v1/admin/collections/<id>/products.json`; and `PUT /v1/admin/products/<id>.json` with `product[category_ids][]` or `product[theme_category_ids][]` (200, field ignored). The product `category` field is a free-text label, not a collection link. `add_products` works only from a logged-in admin tab (`18_ADMIN_NAVIGATION.md` § Import and publish routes). Any API-key tool must hand collection membership to a person until the core developer adds a write. *Verified by query, 2026-10-12.*


Two things follow from § 13's CORS note and from `13_TEMPLATE_BOUNDARIES.md`:

- A cross-origin `PUT` triggers a CORS preflight. Use `POST` with `_method=put` from a browser context.
- Custom fields are site-specific and are not inherited parent to child. The field must already exist on the site being written to, or the value is silently dropped.

---
## 13a. Promocodes and Gift Vouchers

Promocodes apply discounts at checkout. Gift vouchers are Promocodes with `reuse_credit: true` — they function as a balance that can be spent across multiple orders.

### Promocode fields

| Field | Description |
|---|---|
| `code` | Public reference code used at checkout |
| `name` | Friendly name (visible publicly) |
| `used` | Boolean — if `true`, the promocode cannot be used again |
| `starts_at` | Date/time the promocode becomes active (defaults to now if omitted) |
| `expires_at` | Date/time the promocode expires — **required** |
| `multiple_use` | Boolean — allow use across multiple orders |
| `number_remaining` | Limit total uses (optional) |
| `amount` | Fixed amount off total cart price |
| `percentage` | Fixed percentage off total cart price |
| `reuse_credit` | Boolean — gift voucher mode; `amount` acts as spendable balance across multiple orders |
| `discount_rules` | JSON string for complex discount behaviour (overrides simple fields) |

### Create a site-wide promocode

POST /v1/promocodes.json


```bash
curl -X POST \
     -d "promocode[name]=Summer Sale" \
     -d "promocode[code]=SUMMER25" \
     -d "promocode[expires_at]=2026-09-01T00:00:00+00:00" \
     -d "promocode[multiple_use]=true" \
     -d "promocode[percentage]=25" \
     https://yoursite.pixfizz.com/v1/promocodes.json
```

`starts_at` is optional — defaults to the current date/time (immediately active).

### Create a user-specific gift voucher

POST /v1/users/{user_id}/promocodes.json


```bash
curl -X POST \
     -d "promocode[expires_at]=2027-12-31" \
     -d "promocode[amount]=20" \
     -d "promocode[reuse_credit]=true" \
     https://yoursite.pixfizz.com/v1/users/169407050/promocodes.json
```

This creates a $20 gift voucher usable only by the specified user, spendable across multiple orders.

### Read

GET /v1/promocodes/{id}.json GET /v1/promocodes/_code/{CODE}.json GET /v1/promocodes.json # all site promocodes GET /v1/users/{user_id}/promocodes.json # user's promocodes


### Update

PUT /v1/promocodes/{id}.json


Updatable fields: `code`, `name`, `used`, `starts_at`, `expires_at`.

> **Critical:** `amount`, `percentage`, and `discount_rules` are **immutable after creation**. To correct a voucher's value, delete and recreate it. Order history of the original promocode is preserved even after deletion.

### Delete (only after expiry)

DELETE /v1/promocodes/{id}.json


Deletion is only permitted after the promocode has expired. Historical order data referencing the deleted promocode is retained.

SOURCE: Pixfizz Promocodes API documentation. Confirmed applicable: #development Slack, Alex + Matjaz, 2026-08-14.

---

## 13d. Order Webhook — Registration and Payload

The Pixfizz platform can POST an order to an external endpoint when the order changes state.
**Customers register the webhook themselves in Pixfizz admin** — it is a self-serve setup step,
not a Pixfizz engineering job, and it is the same mechanism OrderHub uses. Anyone standing up an
order consumer registers their own endpoint the same way.

**The payload carries `orderlines[].product_id`, a numeric internal product id. It does not carry
`product_code`.** *Verified by query, 2026-09-09* — `product_code` is absent from every orderline
on every payload checked.

Consequence, and it is not cosmetic: **any downstream consumer that keys items by product code
cannot join against the webhook.** The rest of the platform — the storefront, the design product,
the feed, every client-side analytics event — identifies a product by its code. A consumer built
on the webhook alone identifies it by a number, and the two sets never meet. Plan for a lookup on
the consumer side, or expect item-level reporting not to reconcile.

The live case is the server-side GA4 `purchase` pipeline, where this produces item reports with
two rows per product and funnels that never join. See `85_GA4_SERVER_SIDE_PURCHASE.md`.

**A file upload template option comes back as `db:<id>` only, everywhere outside Liquid.** `/v1/orders/<id>/orderlines.json`, `/v1/orderlines/<id>.json`, `/v1/admin/orderlines/<id>.json`, `/v1/books/<id>.json` and the webhook payload (`template_options[].value`) all return `db:<id>`; `/v1/uploaded_files/<id>`, `/v1/files/<id>` and `/v1/uploads/<id>` are 404. The file URL exists only in Liquid (`chosen_option.uploaded_file.url`) and on the admin orderline page. To hand the file to a server, stamp the URL into an orderline custom field with the orderline custom fields script. *Verified by query, 2026-10-10.*

**Generated production files (`orderlines[].files[]`, `/orderlines/w2/<id>/files/<n>?hash=`) appear on `order.status_changed` and `order.updated`, never on `order.created`.** A consumer that needs the files must listen for the later events. *Verified by query of 14 days of webhook logs, 2026-10-10.*

**Event `order.fulfilled` (November 2026).** Fires every time an order is fulfilled successfully: the first time and on every successful refulfillment. A consumer that should act once per order must dedupe on the order id. *Stated by the core developer, Notion Dashboard, November 2026.*

**Super account and global webhooks (November 2026).** Besides site webhooks:
- **Super account webhooks:** super users on a super account can create webhooks in the super admin; they fire for the event on any site of that super account.
- **Global webhooks:** omnipotent (Pixfizz staff) users can create webhooks in the global super admin; they fire for any site in the system (intended for tracking print-on-demand orders and feeding OrderHub).
- The payload is the site-scoped payload wrapped in an envelope; `data` is identical to the site-scoped payload of the same event type, and `super_account` is present only on global webhooks:
  ```json
  {"event_id": "...", "event_type": "order.fulfilled", "created_at": "2026-11-04T16:03:52+01:00",
   "website": {"id": 123456, "subdomain": "..."},
   "super_account": {"id": 123, "name": "..."},
   "data": {...}}
  ```
*Stated by the core developer, Notion Dashboard, November 2026.* Platform-level.

**Failed deliveries, retries and automatic deactivation (from 2026-10-20).** A failed delivery is retried (the worker logs *Failed to deliver webhook ... on attempt #1; will retry*). A webhook is deactivated automatically after **20 consecutive failures**, and an email (template key `WebhookDeactivated`) is sent when that happens. A webhook can also be deactivated temporarily by hand. A consumer whose endpoint was down for a long spell should check that its webhook is still active. *Observed in release commits 2026-10-20 and in worker logs; behavior not verified.* Who receives the email, and how many retries one delivery gets, is not stated.

**Other payload facts** (*verified by query on stored payloads, 2026-10-17*):
- The order channel is `custom.order_source` (values seen: `kiosk`, `online`, `shop`). There is no top-level `order_source`.
- `created_at` keeps the store's local UTC offset (for example `-05:00`). Use it for the store's calendar day rather than converting to the consumer's time zone.
- No tip or staff fields exist in the payload. `extra_fees` carried only rush, minimum order, cut print and oversize shipping fees in the payloads checked.
- No payload checked carried `custom.kiosk_id` or any store id, so per-store reporting from the webhook needs the terminal capture (`32_ORDER_LIFECYCLE.md` § Kiosk Terminal Tracking) live and verified on that site first.

---

## 13e. What Is Not Possible Today

Recorded so these stop being re-proposed as available. Each is a **current limitation, not a
roadmap commitment**, and each should be re-checked rather than quoted from here indefinitely.

### Price Variables via the API — superseded

This entry previously said Price Variables were not reachable via the API (core developer,
2026-09-07). **That is superseded:** an experimental read/write Price Variables API was
announced on 2026-09-16 — see § 13f. § 13f is staging only (verified 2026-09-16), so until it deploys,
bulk export and import **through admin** remains the safe bulk route.

**Update 2026-09-23: Price Variables read and update are confirmed on production.** `GET /v1/admin/price_variables.json` and `PUT /v1/admin/price_variables/<id>.json` work on the normal host. Fields: `id`, `name`, `description`, `value` (a string); there is no `updated_at`. Create and delete were exercised only on `:5748` (which writes to the same database, see § 13f). `cms_snippets`, `cms_pages` and `cms_layouts` were still staging-only on that date. *Verified by query.*

### There is no template import endpoint

**Stated by the core developer, 2026-09-09; not independently verified against the API.** Template tar files must
be imported **one at a time through admin**. There is no endpoint, so a bulk-generated set of
templates is a manual import per template, however the set was produced.

The named blockers are large-file handling and progress tracking, both of which are real
engineering rather than a missing route. Factor the manual import time into any project that
generates templates in bulk — it is usually the largest single item in the schedule and it is
routinely estimated as free.

---

## 13f. Experimental Admin API — Price Variables and CMS Content
> **Update 2026-09-26: `cms_snippets` and `cms_pages` answer on production**, on the normal host, with an admin session (verified by query on the Shopper parent; calls made with `?sitename=<site>`). `cms_layouts` was not checked, and no write method was tried on production that day (snippet writes were verified on 2026-09-30, see *CMS snippets* below). What the reads return:
>
> | Call | Returns |
> |---|---|
> | `GET /v1/admin/cms_snippets.json?page=N` | `[{id, name, description}]`; page until an empty array |
> | `GET /v1/admin/cms_snippets/<id>.json` | `{id, name, description, content, allow_override}` |
> | `GET /v1/admin/cms_pages.json?page=N` | `[{id, url, title, layout_id}]` |
> | `GET /v1/admin/cms_pages/<id>.json` | `{id, url, title, layout_id, body, meta_title, meta_description, meta_keywords, in_sitemap, custom}` |
>
> - The text field differs: a snippet's is `content`, a page's is `body`. The index is a summary only; read each record for its content.
> - `/v1/admin/snippets.json` and `/v1/admin/theme_snippets.json` are 404.
> - This is now the way to read one snippet or page without a CMS backup, and it shows `allow_override` per snippet, which a backup's front matter does not.
> - With an admin browser session on `admin.pixfizz.com`, `?sitename=<site>` selects any site that admin can reach (seen on assets and products). From a storefront host a call answers only for that site.
>
> The staging note below is kept for history.


> **Experimental, staging only.** Announced by the core developer on the Notion Dashboard,
> week of 2026-09-21. **Not on production:** `GET /v1/admin/cms_snippets.json` on a production
> site did not respond, and the same path on port **5748** (the staging host,
> `https://<subdomain>.pixfizz.com:5748/v1/admin/...`) did, verified by test on 2026-09-16.
> Build and test against the `:5748` host only, and re-check production after the staging deploy. Authentication is HTTP Basic with an admin account, as in § 2. Cross-origin writes
> follow the `POST` + `_method=put` rule in § 13c.

> **`:5748` is staging code against the production database, not a separate database.** A write on `:5748` is a write to live data (`80_ONBOARDING.md` § Staging and Production Share One Database). Product reads on `:5748` can return stale pre-write values, including on the paginated index, and a cache-busting query string does not help because `GET /v1/admin/products/<id>.json` redirects to `/v1/products/<id>.json` and the redirect drops the query string. Verify a product write by reading it back on the normal host. *Verified by query and confirmed by Alex, 2026-09-23.*

All four resources share one pattern. Index endpoints return **pages of 100**; use `?page=N`
as in § 1.

```
GET    /v1/admin/<resource>.json          # list (100 per page)
GET    /v1/admin/<resource>/<id>.json     # read
POST   /v1/admin/<resource>.json          # create
PUT    /v1/admin/<resource>/<id>.json     # update
DELETE /v1/admin/<resource>/<id>.json     # delete
```

### Price variables — `price_variables`

```
price_variable[name]
price_variable[description]
price_variable[value]
```

A formula that references a variable will not save until the variable exists
(`30_PRICING_ENGINE.md`), so a scripted rollout creates variables **before** it writes formulas.

### CMS pages — `cms_pages`

```
page[url]
page[title]
page[body]               # the Liquid content of the page
page[layout_id]          # layout id; -1 = site default layout; blank = no layout
page[meta_title]
page[meta_description]
page[meta_keywords]
page[in_sitemap]         # true | false
page[custom][<field-name>]
```

`page[custom][...]` writes to Page custom fields, which must already be defined on the site
(§ 13c: an undefined field's value is silently dropped).

### CMS snippets — `cms_snippets`

```
snippet[name]
snippet[description]
snippet[content]
snippet[allow_override]  # true | false
```

`snippet[description]` fills the snippet Description column. House convention: never leave it
blank — one line, sentence case, full stop. On a Shopper child site, writing a snippet that exists on
the parent creates or changes a **site override**, which pins that snippet and stops parent
inheritance — the same consequence as pressing **Override Snippet** in admin.

**Snippet writes work on production** (verified by query on baseline, 2026-09-30, from `admin.pixfizz.com` with an admin browser session and `?sitename=<site>`):

| Call | Result |
|---|---|
| `POST /v1/admin/cms_snippets/<id>.json?sitename=<site>` with `_method=put` and `snippet[content]` (multipart) | 200, returns `{id, name, description, content, allow_override}`; read back shows the new content and description |
| `POST /v1/admin/cms_snippets.json?sitename=<site>` with `snippet[name]`, `snippet[description]`, `snippet[content]` | 200, creates the snippet |

- No CSRF token was needed with an admin session.
- **Content is stored with CRLF line ends** (`\n` in, `\r\n` out). Normalize CRLF to LF before comparing or hashing.
- **A child site can create a snippet through the API whose name does not exist on its parent.** The admin UI does not offer this. Treat it as an API-only path and do not use it to create snippets the parent should own.
- **A JSON body keyed `cms_snippet` returns 200 and saves nothing.** `PUT /v1/admin/cms_snippets/<id>.json` with JSON `{"cms_snippet":{"content":...}}` answered 200 and left the content unchanged (verified by query, 2026-10-05). The parameter root is `snippet[...]`, as in the list above. A 200 is never proof of a write: read the snippet back. Fallback that always works from a session: the admin snippet form (§ 13h).
- **Child overrides by API** (verified by query on child sites, 2026-09-30 to 2026-10-05):
  - `POST /v1/admin/cms_snippets.json?sitename=<child>` with a **parent** snippet's name creates the child's override (200). Its Description is **blank unless sent**: always send the parent snippet's Description with it.
  - `POST` with `_method=put` updates an override.
  - `DELETE /v1/admin/cms_snippets/<id>.json?sitename=<child>` removes the override; `GET` on that id is then 404 and the parent value applies again. Only delete ids from the child's own snippet list (myPixfizz's "Reset to standard" works this way).
- Audit a child against its parent by content hash through this API with `?sitename=`, not only against a backup.
- Not tested: server-to-server auth with an API key and no session.

### CMS layouts — `cms_layouts`

```
layout[name]
layout[description]
layout[content]
layout[default]          # true | false
```

### What this changes

- Snippets, pages, layouts and price variables can now be read and written without a CMS
  backup tar. A script can diff, patch and verify one snippet instead of shipping a whole site
  backup.
- Renaming a snippet or page through the API breaks every reference to it exactly as a manual
  rename does. The bulk-rename ban on strings that reference platform data still applies.
- **Still not possible:** template import (§ 13e). Asset deletion is now verified on
  production (`DELETE /v1/admin/assets/<id>.json`, § 13c). (Corrected 2026-10-06.)

---

## 13g. Admin Products API — Read, Write, Create

*Verified by query on two production sites, 2026-09-23, except where noted.*

### Read
`GET /v1/admin/products.json?page=N` returns the catalogue 20 per page (§ 1): pricing (flat or formula), every variant type and value with its price, inventory state and every custom field. `GET /v1/admin/products/<id>.json` redirects to `/v1/products/<id>.json` with the same payload.

The admin product list carries no storefront URL for a product: only `links.self` (an API path) and `code`. Storefront product links read off collection pages take the form `/site/product/c/<collection>?product=<id>-<code>`. *Verified by query, 2026-09-30.*

| Field | Notes |
|---|---|
| `price` | a number **or** a Ruby formula string |
| `price_formula` | boolean, set by the platform; writing a formula string to `product[price]` flips it to `true` |
| `current_inventory` | **absent entirely (not null, not zero) when `track_inventory` is false**. Defaulting a missing key to 0 shows a made-to-order catalogue as sold out |
| `starting_price` | separate display value, nullable |
| `variants[]` | variant types (`id, type, name, code, parent_id, trigger_value_id, control_type, required, published, values[]`); values carry `id, name, code, price, default, published`. `parent_id` + `trigger_value_id` make a type conditional on a value of its parent |

### Write
`PUT /v1/admin/products/<id>.json` is a real partial update: one named parameter changes only that field. Multipart form data is accepted.

- **The inventory write is an absolute set with no compare-and-set.** Re-read immediately before writing and treat a changed baseline as a conflict.
- **A formula is not validated against the product.** `12.99 * cut_print_quantity` saved 200 OK on a static product, where that variable means nothing (`30_PRICING_ENGINE.md`). Probe the storefront after every formula write.
- **Proven writable fields:** `price`, `current_inventory`, `track_inventory`, and since 2026-10-05 also `product[code]`, `product[name]` and `product[custom][<field>]` (form encoded; all three saved together and read back with `GET /v1/admin/products/<id>.json`). Custom fields not sent were kept and variants were untouched; a write of `product[custom][from_pricing]` alone changed only that key. Sent from a logged-in admin page with its `X-CSRF-Token` and session cookie. *Verified by query on three products, 2026-10-05.* A `multitext` custom field needs the array form (§ 13c).
- **Product custom values do not need a definition to be stored.** On a site with 0 custom field definitions, product custom values written through `PUT /v1/admin/products` were stored and read back. The admin product page shows nothing for them until a definition exists (collections show no custom fields form at all without a Collection definition, `18_ADMIN_NAVIGATION.md` § Products). This differs from custom type instances (§ 13c), where a value for a missing field is dropped. Platform-level. *Verified by query, 2026-11-05.*
- **From a page on `admin.pixfizz.com`, a same-origin `PUT /v1/admin/products/<id>.json?sitename=<site>` with no `X-CSRF-Token` also returned 200 and saved.** Across 540 products only the sent key changed, and the public `/v1/products.json` showed the new value immediately. One read-back difference: a `fulfillment` that read `""` before the write reads `null` after it, so treat `""` and `null` as equal when diffing. `GET /v1/admin/products.json?sitename=<site>&per_page=100&page=N` from the same page reads the catalog 100 per page. *Verified by query, 2026-10-12 and 2026-10-13.*
- **Fulfillment destination:** the product field `fulfillment` holds the destination name. `PUT /v1/admin/products/<id>.json` with `{"product":{"fulfillment":"<name>"}}` sets it and reads back. *Verified by query, 2026-10-27.*
- Renaming `product[code]` breaks every reference to the old code (collection filters, Liquid, fulfillment, print-on-demand lookups) exactly as a manual rename does. The bulk-rename ban on strings that reference platform data applies.

### Create
`POST /v1/admin/products.json` always creates, never updates. **A design product is created the same way** (what the Publish Products wizard sends): `POST /v1/admin/products.json?sitename=<site>` from the admin host with `product[print_product_id]` (the template), `product[name]`, `product[code]`, `product[price]`, `product[min_units]`, `product[max_units]`, `product[fulfillment]` and `product[custom][...]`. Custom fields are saved on create. It works with a template provider's template on a consumer site. *Verified by query, 2026-10-10.* A product attribute on a template is created the same way in one call (form-encoded `product[code]`, `product[name]`, `product[print_product_id]`, `product[price]`, `product[custom][<key>]`, and `product[custom][<key>][]` repeated for an array field). `product[price]` accepts a formula string; the response is the new product with `price_formula: true` and `starting_price` evaluated. *Verified by query from an admin tab, 2026-10-19 (14 products).* **`product[units][min]` and `product[units][max]` are ignored on create**: the product comes out with units 1 to 1000. Set the limits afterwards with `PUT /v1/admin/products/<id>.json?sitename=<site>` and `product[min_units]`, `product[max_units]` (§ *Quantity limits*), and read back `units {min, max}`. *Verified by query, 2026-10-21.* Whether `product[min_units]` and `product[max_units]` save on create was not re-checked: read the units back after every create. `product[name]` and `product[image]` are capped at 64 characters; codes are unique case-insensitively; **`product[description]` returns 200 but is never stored**, on create or update. There is no API to delete or archive a product.

**Consumer products on a template provider (2026-10-23 and 2026-10-27).** The same create works for a template provider's consumer site with `print_product_id` set to the provider template id; use it when the provider does not appear as a source on the admin POD tab (seen for a consumer of a Pixfizz template provider). A JSON body was accepted without a CSRF header. Set units afterwards (above), and the description only through the admin product form (POST `/site/<site>/admin/products/<id>` with every field of the form holding `#product_description`). Copying the variant set: `16_PRODUCT_HIERARCHY.md` (the import is additive; the v1 product JSON lists the result under `variants`, not `variant_types`). *Verified by query.*

### Public catalog reads (no login)
Each product in the public `/v1/products.json` carries `category`, `price` and `print_product_id` (null means a static product). `/v1/theme_categories.json` lists every collection with `id`, `name` (the path segment), `display_name`, `description`, `image` and `custom.unpublished`, and each entry carries `themes` and `static_products` arrays (both empty means an empty collection). `GET /v1/admin/theme_categories.json` returns 404: the public route is the collections read. `/v1/products/<id>/variants.json` and `/v1/products/<id>/price_forecast.json` are public; a static product with base price 0 forecasts 0 until variants are applied. Public endpoints need no login. Useful for launch checks: `80_ONBOARDING.md` § Launch Check: Empty Collections and Unpriced Products. *Verified by query, 2026-10-02 and 2026-10-04.*

**The public products read exposes product custom fields and formula source.** A same-origin `GET /v1/products.json?per_page=250` on a storefront domain, with no admin session and no key, returned every product with `code`, `name`, `price` (the **full Ruby pricing formula source**), `print_product_id`, `variants` and the **whole `custom` object** (every product custom field value). Platform-level. *Verified by query on a client site, 2026-10-20 (151 products in one page).*
- Useful: a custom tool can look up other products (for example "does any product already carry this reference number") with no endpoint of its own.
- **Security consequence: anything in a product custom field or a pricing formula is readable by anyone who can reach the storefront.** Never put secrets, costs, supplier prices or internal notes in product custom fields or in formula comments.
- Not verified: whether a login-only storefront setting hides this endpoint, and the `per_page` ceiling.

More on the collections read (*verified by query, 2026-10-12 to 2026-10-15*):
- **It is not paged:** `?page=` and `per_page` return the same full list. Child collections are nested under `sub_categories`. An empty list was seen once right after a write; retry an empty list before trusting it.
- Each `static_products` item carries `product_id` and `product_code`. This is the only read that shows which static products sit in a collection: the product read has no collection field.
- Each `themes` item carries `id`, `code`, `name`, `published`, `product_id`, `product_code`, `product_custom` and `uid`, and unpublished designs are included (`published: false`).
- `/v1/theme_categories/<name>.json` returns one collection by name; by numeric id it is 404. A subcollection's name carries its parent path (`wall-art/acrylic-prints`), and both the slash and `%2F` forms answer, so anything that names a collection (a tool mount, a filter) must use the full name once the collection sits under a parent.
- **Finding designs from the storefront:** there is no product-level design list (`/v1/products/<id>/designs.json`, `/v1/products/<id>/themes.json`, `/v1/themes.json` and `/v1/print_products/...` are all 404), so the collections read is the way in. A design in no collection cannot be found this way. `/v1/themes/<id>.json` then gives its template options (§ 9, *Reading a design and a font*).
- **A link a tool builds to another design product needs the design id.** Without `&theme=<design id>` the product page renders but its template options, and any `custom_script` tool, are not on it (`50_SHOPPER_TEMPLATE_REFERENCE.md`, clean product URLs). The design id per product is only in this collections read (`themes[].id` with `themes[].product_id` and `published`); `/v1/products.json` does not carry it. One call returns the whole tree (about 1 s, and 0.8 MB on a site with 151 designs), so fetch it when the link is needed, not on page load. *Verified by query, 2026-10-20.*
- **"Sign in to see prices" hides prices in the page only:** `price_forecast.json?print_theme_id=<design>` still returns the price to a signed-out visitor. *Verified by query, 2026-10-14.*
- **`price_forecast.json` prices only what you send.** Variants and template options left out are not applied at their defaults, so a call without them returns the product formula alone; on a book, `pages` is the count of **system** pages, which differs from the printed count on a product with `spreads_as_pages`. Full rules: `30_PRICING_ENGINE.md` § Platform Price per Quantity and Cart Rounding. *Verified by query, 2026-10-19 to 2026-10-21.*
- **Sending variants:** `GET /v1/products/<id>/price_forecast.json?print_theme_id=<design>&variants[<variant code>]=<value code>` (repeat per variant) applies each value's price and answers `{price, variants_applied: {<variant code>: <amount added>}}`, so a tool can show what each choice added. *Verified by query on a test site, 2026-10-30.*

### Variant writes: experimental `/v1/admin` routes (Corrected 2026-10-09)
Until 2026-10-06 there was no write API for variants (*core developer, 2026-09-23*). **On 2026-10-06 the core developer opened the variant endpoints under `/v1/admin`, marked experimental, with the same paths and parameters as the old internal `/admin` routes.** `PUT /v1/admin/variant_values/<id>.json` with `variant_value[price]=2.51` answered 200 and read back 2.51 (then restored). *Verified by query, 2026-10-07.* `PUT /v1/admin/variant_values/<id>.json?sitename=<site>` with `variant_value[value]=<name>` renames a value; code, image, default and price are kept. Sent from an admin page with no CSRF header, it answered 200 and read back (*verified by query, 2026-10-14*). Writes of `published`, `default` and `code` on a value, and creating or deleting types, are not tested through these routes. Read every write back: a 200 is not proof. The admin forms (§ 13h) remain the proven route for creating types and values.

### Quantity limits
`product[min_units]` and `product[max_units]` on `PUT /v1/admin/products/<id>.json` both save and read back. The product read returns `units: {min, max}` only: the quantity steps (`unit_intervals`, `30_PRICING_ENGINE.md`) are neither read nor written by the API. *Verified by query, 2026-10-06 and 2026-10-07.*

---

## 13h. Admin Form Writes From a Browser Session

Some writes have no `/v1` endpoint, or the endpoint fails silently. From a logged-in admin tab on `admin.pixfizz.com` they can be scripted through the admin's own forms. Platform-level (Pixfizz CMS). *Verified by query on the Shopper parent and two client sites, 2026-10-05, except where noted.*

These are browser-session routes, not an integration surface: they need an interactive admin login and the page's CSRF token (any `authenticity_token` input, or the admin page's jQuery, which sends the header for you). Paths are relative to `admin.pixfizz.com`.

**General rules**
- **Resubmit every field of the form**, changing only the one you mean to change. Leaving a field out is not proven safe (`18_ADMIN_NAVIGATION.md` § Bulk Update Tools has an open question on whether the template option custom fields form merges).
- **Fields rendered client-side are not in the fetched HTML** (snippet-type custom fields, Ace editors; see `18_ADMIN_NAVIGATION.md` § Admin Overview). Append them to the form data yourself, or they are sent empty.
- **On the Product Attribute page, a snippet-type custom field (for example `product[custom][<field>]`) is a hidden textarea plus an Ace editor, and only shows after "Show All Fields".** Setting the textarea alone does not save. Set the editor too (`ace.edit(textarea.parentElement.querySelector('pre.ace_editor')).setValue(value, -1)`), set the textarea, then click Save, and read the value back after a reload. *Verified by query, 2026-10-13.*
- Values in snippet-type fields come back with CRLF line ends. Fold CRLF to LF before comparing.
- Re-read the object after the write. A 200 is not proof.
- **Admin pages carry a Content Security Policy of `connect-src 'self' data: blob: cdn.pixfizz.com`.** Script on an admin tab cannot fetch from localhost, a storefront host or any other host. A file reaches an admin page only through a file input, or is built in the page (for example from an existing asset fetched from `cdn.pixfizz.com`, patched and hash-checked before posting). *Verified by query, 2026-10-13 and 2026-10-14.* **The same policy blocks `eval`:** `script-src 'strict-dynamic' 'nonce-...'`, so code passed as a string cannot be evaluated. Inject it as a `<script>` element that carries the page's own nonce (`document.querySelector('script[nonce]').nonce`); `'strict-dynamic'` then lets it run. *Verified by query, 2026-10-19.*

**Snippet content.** `GET /site/<site>/admin/snippets/<id>/edit`, take the form whose `_method` is `patch`, resubmit every field plus `snippet[content]` (the editor field is not in the static HTML). Name, description and `allow_override` stay as they were. Use this when the § 13f API write does not take. (Corrected 2026-10-17: the earlier text said to take the form that holds `snippet[name]`.) **On a child override the patch form has no `snippet[name]` field**, only `snippet[content]` once the page script has run, so pick the form by `_method=patch` and `snippet[content]`, never by `snippet[name]`. The snippet list exposes ids as `<a id="snippet_<name>" href=".../snippets/<id>/edit">`. *Verified by use, 2026-10-13.*
- **On a snippet edit page the first form carrying `_method` is the DELETE form** (`button_to`). Only ever submit the form whose `_method` is `patch`. *Verified by query, 2026-10-06 (an override was deleted this way).*
- **An Ace-editor value read from an edit page carries `<\/script>`.** The page embeds the value as a JS string and only turns `<\/script>` back into `</script>` at runtime. Parsing the string yourself leaves the backslash, and writing it back stores a literal `<\/script>`, so an embedded script (for example a tool mount's JSON block) never closes. Replace `<\/script>` with `</script>` before posting. Applies to snippets and to a template option's `custom_script`. *Verified by query, 2026-10-06.*
- **A byte-exact edit of a large snippet without retyping it.** The edit page carries the stored content as a JS string, `const content = "...".replace(/<\\\/script>/g, '<\/script>')`. `JSON.parse` the literal, undo the `<\/script>` escape, apply each find/replace only if its find string occurs exactly once, then post `snippet[content]` back (patch form) or `setValue()` it into the Ace editor (`document.querySelector('.ace_editor').env.editor`) and click **Save & Continue**. Read the snippet back and compare. Worked on snippets up to about 30 KB. *Verified by query, 2026-11-03 and 2026-11-05.* Read the snippet fresh immediately before the edit: another editor can change it in between. After an Ace save, a read-back through a separate snippet read came back with LF line ends, while the § 13f API read returns CRLF: fold CRLF to LF on every compare. *Verified by read-back, 2026-11-05.*
- Create in one request: the admin snippet create form (`POST /site/<site>/admin/snippets`) accepts `snippet[content]` together with name, description and allow_override. Allow Override is on by default. *Verified by query, 2026-10-03.*
- **Saving content: two observations disagree.** The form route above saved on 2026-10-05. On 2026-10-08, on a child site, a fetch of the edit page plus `FormData` did not save content, because the hidden `snippet[content]` field only exists after the page script runs. What worked there: load the edit page in a same-origin iframe, wait for `.ace_editor`, call `editor.setValue(text, 1)`, click **Save & Continue**, then reload in a second iframe and compare `getValue()` (verified on 8 snippets). After a programmatic `setValue` the first Save click sometimes does not persist; click again. Whichever route, read the snippet back before reporting it saved. Custom Type instances save the same way (iframe, set the editors and inputs, click Save); **Add Instance** creates an empty record at once.
- **Override a parent snippet on a child from the admin form:** `POST /site/<child>/admin/snippets` with `authenticity_token`, `save_and_continue=true`, `parent_id=<parent snippet id>` and `commit=Create` (the parent ids are the options of `#parent_id` on `/admin/snippets/new`). It redirects to the new override's edit page. **Whether the override starts as a copy of the parent or empty was seen both ways** (a copy on 2026-10-06 and again on 2026-10-26, empty on three overrides on 2026-10-08): always write the full body and read it back. The API route is in § 13f.
  - **2026-10-31 observation (a Shopify + Pixfizz child):** Override Snippet in admin (`snippets/new`, `parent_id` select) created an **empty** override, filled at once with `POST /v1/admin/cms_snippets/<id>.json?sitename=<child>`, `_method=put` and `snippet[content]` (§ 13f). *Verified by query, 2026-10-31.*
  - **2026-11-06 observation (3 overrides on a new child):** the create POST made the override with the **parent's** content, and a `snippet[content]` sent in that same POST was ignored. The content was then saved with a second request: the edit page's form whose `_method` is `patch` (never the DELETE form), `new FormData(form)` plus `snippet[content]` and `snippet[description]`, posted with the headers `HX-Request: true` and `HX-Boosted: true`, and read back from the edit page's `const content = "..."` script. Whether the two headers are required was not tested. *Verified by query, 2026-11-06.* So: create first, then write the content and description, then read back.

**Replace an asset in place.** See § 13c *Assets*.

**Template options.**
- Create: `GET /site/<site>/admin/templates/<template_id>/options/new`, then POST `template_option[name]`, `template_option[code]`, `template_option[value_type]`, `template_option[published]`, `template_option[required]`. The response redirects to `/options/<id>/edit`, which carries the new id.
- Custom fields are a second form on that edit page, `template_option_type[custom][...]`. `custom_script` is a snippet-type field and is not in the static HTML: append `template_option_type[custom][custom_script]` yourself. It saves and renders. The full-field rule and the DELETE-form trap on this page are in `18_ADMIN_NAVIGATION.md` § Bulk Update Tools.

**Variant types and values.** POST `/site/<site>/admin/products/<id>/variant_types` with `variant_type[name|code|value_type|required|published]`, then POST `/site/<site>/admin/variant_types/<id>/variant_values` with `variant_value[value|code|price|default|published]`. Each response redirects to the variant type's edit page, which carries its id.
- **`variant_value[image]` takes an asset name** (the same create form). *Verified by use and read-back, 2026-10-31.*
- **Reorder variant values:** POST the table's `data-rearrange-url` (`/site/<site>/admin/variant_values/<variant type id>/reorder`) with a URL-encoded body of `variant_values[]=<id>` repeated in the target order (the parameter name is the table id). `ids[]`, or a `FormData` body, answers 500. *Verified by use and read-back, 2026-10-31.*
- **A layout substitution on a variant value.** The New Element Substitution picker does not list "layout", but `GET /site/<site>/admin/element_substitutions/new?owner_id=<value id>&owner_type=VariantValue&substitution_type=layout` returns the form (`element_name`, and `content` = the layout name); POST it to `/site/<site>/admin/element_substitutions`. Other substitution types use the same route (`18_ADMIN_NAVIGATION.md` § Bulk Update Tools; template option values: `22_OPTION_VARIANT_RENDERING.md`). *Verified by use and read-back, 2026-10-31.*

Template option create, short form: POST `/site/<site>/admin/templates/<template>/options` with `template_option[name|code|value_type]` also worked; `custom_script` read back from the edit page's `const content = "..."` comes back with CRLF line ends. *Verified by use and read-back, 2026-10-31.*

**Extra Fees.** Create with `POST /site/<site>/admin/extra_fees` carrying `authenticity_token`, `extra_fee[name]`, `extra_fee[code]`, `extra_fee[taxable]` (hidden `0` plus checkbox `1`) and `extra_fee[formula]` (the Ace field is not in the static HTML). Read it back from the edit page's embedded `const content = "..."` (unescape `<\/script>` as for snippets). *Verified by query, 2026-10-10.*

**Collections.** The collection show page, `/site/<site>/admin/theme_categories/<id>`, is also its edit page. `/theme_categories/<id>/edit` returns 500.
- **Collection image.** The image is the collection's `asset_name`, and it accepts any asset name on the site, WebP included. Take the form that contains `theme_category[asset_name]` (fields `_method=patch`, `authenticity_token`, `display_name`, `name`, `asset_name`, `description`), resubmit every field unchanged except `asset_name`, and POST to the form action. Afterwards `GET /v1/theme_categories.json?sitename=<site>` shows `image` as the asset's CDN URL. Template-level (Shopper 24): `collection/shop-all` shows this image, or `.shop-all-placeholder` when there is none.
- **Collection custom fields.** The second form on the same page. Fields seen missing from the static HTML: `banner_html`, `collection_footer`, `collection_filters`. A `PATCH /site/<site>/admin/theme_categories/<id>` with `_method=patch`, `authenticity_token` and only the fields to change was **partial**: other fields, snippet-type included, were kept (verified by query, 2026-10-03). A 2026-10-05 run still advises resending `collection_filters` on every save in case it is lost; unconfirmed which holds, so resend it. **2026-11-03: a second confirmation of the merge.** A save of the collection custom field form that did not send `collection_filters` left its stored value in place (a test value survived), across four client sites. *Verified by query, 2026-11-03.* Posting only the fields you change is now supported by two runs; resending `collection_filters` remains the cautious choice when the form is replayed whole.
- **Collection preview images are not a custom field.** They are the `linked_assets/replace` form on the collection page (`owner_type=ThemeCategory`, `mapped_preview=false`, `asset_ids[]` by asset id), and they feed the gallery block that design products in that collection show (Shopper 24 `product/gallery/standard`). *Verified by query, 2026-11-03.*
- **Collection custom fields with Ace editors.** To write them through the page's PATCH form, resubmit every field plus each snippet-type field (for example `collection_filters`) read from the page's `const content = "..."` script; those fields are not in the fetched form. *Verified by query, 2026-10-26.*
- **Hide a sub-collection.** The same `PATCH /site/<site>/admin/theme_categories/<id>` with `_method=patch`, `authenticity_token` and `theme_category[custom][unpublished]=1` saved as a partial update (read back), and the collection no longer showed on `/site/shop` or on its parent collection page. The `PATCH` is platform-level; `unpublished` is a Shopper template custom field, so the hiding is template-level. *Verified by query on the headline demo site, 2026-10-21.* `unpublished` hides the tile only: the collection page still opens by direct link. To close the page itself, unpublish every design in the collection (the `product_theme` published flag); `/site/shop/<path>` then shows "Coming soon". *Verified by query, 2026-11-03.*
- **Product order.** The Design Products table is bound to `POST /site/<site>/admin/theme_categories/<id>/order`. The body is `product_themes[]`, repeated once per row with the row's `tr` `data-id`, in the full target order. Every Move to Top or Move to Bottom click sends the whole list, so one call re-sorts the whole collection:
  ```js
  $j.ajax({url, type:'POST', data:{product_themes: ids}})
  ```
  Run it from the collection page, with `url` set to the `/order` path and `ids` the full ordered list. Verified on a 55-row collection, by a fresh fetch of the admin page and on the logged-out storefront. **Send every row id.** Not tested: what a partial list does to the rows left out. The Move to Top and Move to Bottom items are `li` elements already in each row's DOM, so a JS `click()` on one fires the request without opening the popover.
- Each move saves immediately. Rows added to a collection append in call order (seen with `add_themes.json` on 2026-10-04 and with the publish wizard's POST on 2026-10-12), so add a size range in size order: on `pdp_layout` pages the collection order drives the size tile order (`50_SHOPPER_TEMPLATE_REFERENCE.md`). *Verified by query, 2026-10-04 and 2026-10-12.*
- **Remove a product from a collection.** The per-row `remove_theme` form (DELETE, `product_theme=<id>`). Reversible: add the product again.
- **Add designs to a collection: use the Publish Products wizard route.** (Corrected 2026-10-17: this section previously documented `POST /admin/theme_categories/add_themes.json` with `product_id`, `category_ids[]`, `theme_ids[]`, and noted it could answer `{"error":"Not Found"}` and still have added the designs, as seen on 2026-10-05.) From 2026-10-12 to 2026-10-14, on several sites and on both the `/admin` and `/site/<site>/admin` paths, `add_themes.json` answered `{"error":"Not Found"}` and **added nothing** (row count unchanged on re-read). The wizard is what works:
  - By hand: the collection page's **Add Design Product** link (`/site/<site>/admin/manage_products/print_products?theme_category_ids[]=<collection id>`), search the template, Select Designs, Product Attributes (pick the existing product, not "new"), Choose Collections (tick only the target), then "Publish N designs to 1 collection".
  - By URL: the last step opens directly at `/site/<site>/admin/manage_products/theme_categories?print_theme_ids[]=<design>&product_id=<product>&template_id=<template>&theme_category_ids[]=<collection>` (repeat `print_theme_ids[]` per design, or pass a position range as `print_themes=<range>`, for example `9-24`, which is what the Select Designs step writes into its hidden field). It also pairs designs with the right product attribute. One click on the publish button, or a POST of its form to `/site/<site>/admin/manage_products/publish` with `authenticity_token`, `print_themes`, `product_id`, `template_id`, `parent_id=''`, `theme_category_name=''` and `theme_category_ids[]`, lands the rows published.
  - **The publish POST needs `print_themes` positions.** (Corrected 2026-10-22: this step previously said `print_theme_ids[]` or `print_themes`.) A `POST .../manage_products/publish` with `print_theme_ids[]` now answers 200, redirects to `manage_products/print_themes?...` (the "choose at least one design" step) and publishes nothing. Send `print_themes`, with the `authenticity_token` read from `manage_products/theme_categories?print_themes=<positions>&product_id=..&template_id=..&theme_category_ids[]=..`. *Verified by query, 2026-10-20 (68 designs).* `print_themes` encodes **0-based** design positions, as a range (`0-84` for 85 designs, `0` for a template with one design), a dot list (`0.1.2`) or both (`15-29.31`); the positions are those of the `print_theme_position` checkboxes on `manage_products/print_themes?template_id=<template>`, which also lists the designs with their names. *Verified by query, 2026-10-19 to 2026-10-21.* `GET manage_products/product_attributes?continue=1` with an existing `product_id` redirects to the `theme_categories` step.
  - **Create a collection inside the wizard.** The Collections index shows no create button (only Import), but a collection can be created two ways: the form at `/site/<site>/admin/theme_categories/new` (`POST /site/<site>/admin/theme_categories` with its token, *verified by query, 2026-10-19*: `18_ADMIN_NAVIGATION.md` § Products (Collections)), or the wizard's last step, which has a **Create and Tick** button: a submit with `formaction` `/site/<site>/admin/manage_products/create_theme_category`, sending the whole publish form plus `parent_id` (a collection id, or the "Top level" option) and `theme_category_name`. A POST of that form from page script created the collection, slugged the name and prefixed it with the parent path (`<parent path>/<slug>`), and returned the same step with the new collection ticked; posting that form to `manage_products/publish` then published the designs. On a consumer site the same run with `product_id=new` also created the product attribute on the provider's template. *Verified by query on experience.pixfizz.com, 2026-10-21.*
  - On a consumer of a template provider, `manage_products/product_attributes?continue=1&template_id=<provider template>&print_theme_ids[]=<design>&product[name]=..&product[code]=..&product[price]=..&product[category]=..&product[fulfillment]=` returns the publish form with `product_id=new`, so the publish also creates the lab's product attribute; the redirect to `manage_products/published` carries the new product id.
  - **Only a template provider's templates were offered this way.** On a new Shopper child, the same `product_attributes?continue=1&template_id=..&print_themes=0&product[name]=..&product[price]=..` GET worked for a template from the Pixfizz templates site (a template provider in the same super account), but a template owned by another ordinary site in the same super account answered "Please choose a template" (the redirect described in `18_ADMIN_NAVIGATION.md`). Those were copied by template export and import instead. *Verified by query, 2026-11-06.* This sits uneasily with the Super Admin help text that sites in the same super account already share templates (`18_ADMIN_NAVIGATION.md`): pending confirmation with the core developer whether same-account sharing covers publishing, or only templates on a site flagged as provider.
  - **Add an existing design product to one more collection.** `GET /site/<site>/admin/manage_products/theme_categories?print_themes=0&product_id=<product>&template_id=<template>`, take the form that posts to `manage_products/publish`, keep its already-ticked `theme_category_ids[]`, add the new collection id and POST. It is add-only: existing collections stay, and no duplicate product is created. *Verified by query on two client sites, 2026-10-26 and 2026-10-27.* The admin screen route is in `18_ADMIN_NAVIGATION.md`.
  - Static products still use `add_products` (`18_ADMIN_NAVIGATION.md` § Import and publish routes). Always re-read the collection rows after any publish.
  - A template import file can be generated as JSON (`---` then `JSON.stringify(...)`) with no YAML library; route and success and failure signals: `18_ADMIN_NAVIGATION.md` § Import and publish routes. *Verified by query, 2026-11-04.*
  *Verified by query and on the storefront, 2026-10-12 to 2026-10-14.*

**Design display name.** POST `/site/<site>/admin/print_theme/theme/<design_id>` with `theme[name]`. The design code is untouched. The form carries `print_product_id`, `name`, `code`, `description` and `editor_configuration_id`: resubmit it whole. *Verified by query, 2026-10-03.*

**Design custom fields.** `PATCH /site/<site>/admin/print_theme/update/<design id>` is partial: only the fields sent change. Use it instead of the `/v1` design PUT, which saves nothing (§ 13c). *Verified by query, 2026-10-03.*

**Site settings.** Settings → General is `/site/<site>/admin/site/index` (form `update_site_settings`). Its `website[title]` is what the parent-rendered legal pages (Terms, Shipping) and the services heading print as `{{ website.title }}`, so a blank or old company name there comes from this setting, not from the `website/brand` snippet. Platform-level setting, read by Shopper 24 pages. *Verified by query and by reading the Shopper 24 parent backup, 2026-11-02 and 2026-11-03.*

**Shipping services.** `/site/<site>/admin/shipping_services/<id>`, field `shipping_service[formula]`. A formula of `0` makes that service free at checkout. *Verified by query, 2026-11-03.* Launch check: `80_ONBOARDING.md` § Post-Import Checks a Tar Cannot Cover.

**Assets have no show page.** `/site/<site>/admin/assets/<id>` is 404; use `/assets/<id>/edit` (replace in place, § 13c). *Verified by query, 2026-11-03.* (One run concluded that an image can only be changed by uploading under a new name and repointing the reference; replace in place through the edit form was verified again on 2026-11-06, so that conclusion is not adopted.)

**A slash in a new collection name is flattened.** Create and Tick (`manage_products/create_theme_category`) with `theme_category_name=greeting-cards/photo` slugifies the path to `greeting-cards-photo` (the display name keeps the slash). For a nested path, POST the collection form on `/site/<site>/admin/theme_categories/<id>` with `theme_category[name]` set to the slash path and `theme_category[display_name]`, then read it back from the same page, or pick the parent in Create and Tick's `parent_id` (above). *Verified by query, 2026-11-05.*

**Fonts.** Copy a font between sites from an admin tab: the source font page `/site/<source>/admin/print_fonts/<id>` links the `.ttf` on `admin.pixfizz.com/fz/...` (same origin, so the page script can fetch it); POST it to `/site/<target>/admin/print_fonts` with `print_font[name]` and `font_file`. On a Shopper child, add fonts to the products parent (§ 12). *Verified by query, 2026-11-06.*

## 13i. Snippets, Pages Entries and Resources From an Admin Session (2026-11)

Platform-level. *Verified by query, 2026-11-10 unless marked.*
- **Update one large snippet in place:** `POST /v1/admin/cms_snippets/<id>.json?sitename=<site>` with `_method=put` and `snippet[content]` (200, read back equal). Back up the old content first. `POST /v1/admin/cms_snippets.json` with an existing name answers 422; it does not overwrite.
- **Child override of a parent snippet by form:** `GET /site/<site>/admin/snippets/new`, post its form to `/site/<site>/admin/snippets` with `parent_id` = the parent snippet id and `save_and_continue=true`; the answer is the new override's edit page.
- **Pages entries:** create with `POST /v1/admin/custom_types/<ct>/custom_type_instances.json?sitename=<site>` and `custom_type_instance[custom][...]` (200, returns the id; `'false'` strings land as false); update with `_method=put` on `.../custom_type_instances/<id>.json`.
- **Placeholder image of a design page:** upload once with `POST /upload/resource?tag=<resources tag>&sitename=<site>` (fields `code`, `data`); the returned id works as `src="db:<id>"` on any design of that site. On a provider design this changes every consuming site.
- **Asset URLs are signed per host.** The asset `url` the admin API returns (custom domain) carries a host-signed key; the same key on cdn.pixfizz.com answers "invalid security key". Download assets from the URL exactly as returned, or read the CDN URL from the rendered page; never rewrite the host.

## 14. Retrieval Pointer

| Topic | File |
|---|---|
| Shopify JS API (`Pixfizz.Shopify.*`) | `60_SHOPIFY_INTEGRATION.md` |
| Fulfillment job ticket schema | `31_FULFILLMENT_ENGINE.md` |
| Pixfizz Liquid objects (user, order, etc.) | `50_LIQUID_REFERENCE.md` |
| Shopper template cart/checkout | `20_SHOPPER_CART_RULES.md`, `21_SHOPPER_CHECKOUT_POLICY.md` |
| Template responsibility boundaries | `13_TEMPLATE_BOUNDARIES.md` |
| Server-side GA4 `purchase` from the order webhook | `85_GA4_SERVER_SIDE_PURCHASE.md` |
| Order lifecycle and confirmed status | `32_ORDER_LIFECYCLE.md` |
| Price variable formulas and save rules | `30_PRICING_ENGINE.md` |
| Snippet overrides on Shopper child sites | `50_SHOPPER_TEMPLATE_REFERENCE.md` |
| Admin paths, Bulk Update Tools, per-template edit routes | `18_ADMIN_NAVIGATION.md` |

---

## Changelog
- 2026-03-30: Initial version. Compiled from Pixfizz Notion wiki: Pixfizz API Documentation, Create Order API Endpoint, Callbacks from Fulfillment Partners, Creating a Project, Dynamic Design Previews, Custom eCommerce CMS Integration Notes.
- 2026-06-15: Documented login-capable vs external user creation (external_id / external_source param makes a user external; omit for login-capable accounts; merge to repair). Documented preview resolution cap (width 1200, lower quality) and the production-quality /v1/pages/<id>.jpg?fulfillment=true endpoint (superadmin-only). Source: slack-kb-sync (Matjaz, #development).
- 2026-07-01: Added § 8a — Individual Orders (list/read/create/update via `/v1/orders` and `/v1/admin/orders`), including the PUT `order[status]=S` pattern to mark an order Shipped and the custom-field-only update path for regular users. Source: claude-chat.
- 2026-07-04: Documented that `/copy` creates an unsaved project (set `book[saved]=1` on the new ID), and the cross-origin `PUT` CORS-preflight trap (use `POST` + `_method=put`). Source: claude-chat.
- 2026-07-20: Documented the `fulfillment` param on the theme/project preview endpoint (placeholder-vs-resolution trade-off, not officially documented); added §13a no-fixed-outbound-IP networking note; added §13b reprint order-ID uniqueness (append a letter suffix). Source: support-ticket, fulfillment-integration call, #development.
- 2026-07-25: Added § 13c Admin Content API — custom type list/read/instance-create, asset upload and list, and the custom-field update endpoints for products, designs (`theme`), and collections (`theme_category`). Marked not publicly announced; documented the genuine `/admin` vs `/v1/admin` prefix inconsistency. Source: internal notes (Matjaz).
- 2026-08-21: Added full Promocodes / Gift Vouchers API section (§13a) including create, read, update, delete endpoints and gift voucher (reuse_credit) pattern. Source: api-docs + slack-message.
- 2026-09-09: Added §13d Order Webhook — customers register it themselves in Pixfizz admin (the same mechanism OrderHub uses), and the payload carries `orderlines[].product_id` (numeric internal id) and not `product_code`, so any consumer keying items by product code cannot join (verified by query). Added §13e What Is Not Possible Today — Price Variables are not reachable via the API (confirmed by the core developer 2026-09-07) and there is no template import endpoint, so bulk-generated template tars are imported one at a time through admin, blocked on large-file handling and progress tracking. Both recorded as current limitations, not roadmap. Added retrieval pointer rows for `85_GA4_SERVER_SIDE_PURCHASE.md` and `32_ORDER_LIFECYCLE.md`. Source: slack-message, fireflies-call.
- 2026-09-16: Added § 13f Experimental Admin API (price variables, CMS pages, snippets, layouts: shared list/read/create/update/delete pattern, 100 per page, parameter lists, override and rename consequences). Staging only, not on production (verified by test 2026-09-16). Superseded the § 13e 'Price Variables are not reachable via the API' entry. Added the `/admin` → `/v1/admin` retirement notice to § 13c with the confirmed replacement table; collections update has no confirmed replacement yet. Source: notion-page (Dashboard), fireflies-call.
- 2026-09-19: Replaced the § 4 Data retention table. The previous table (unsaved 1 year, saved 2 years, ordered indefinitely) was wrong on the point that matters: ordered projects lose their images 6 months after the order for cut prints and 3 years for every other type. Added the full deletion policy (carts, galleries, images, PDFs, uploaded files, users, crawls), the 4-year inactive-user rule that deletes all galleries and saved projects including guests, and the inactive-site rule. Source: notion-page (Pixfizz Wiki, Deletion Policies).
- 2026-09-24: Page size varies by endpoint (products: 20); never hardcode it. Session cookie now `__Host-` prefixed. The `/admin/...` retirement is live (cross-host redirect drops Authorization; use `/v1/admin/...` with manual redirects); `/v1/admin` is not a public integration surface. Price Variables read/update confirmed on production. `:5748` writes to the production database. Added § 13g Admin Products API. Source: claude-chat, slack-message, fireflies-call.
- 2026-09-29: Added API keys (pxk_ key as Basic-auth username, per user per site, shown once, one key per integration). HTTP Basic now points to API keys first; email and password marked legacy. Added the admin UI host vs API host table (admin moved to admin.pixfizz.com; API stays on the site host; admin host /v1 is 404). Removed the stale 'retirement not yet live' line in § 13c. Moved the § 13c custom type endpoints to /v1/admin with what is verified. Moved the § 13c asset endpoints to /v1/admin; list verified. Flagged the collections custom-field PUT as retired with no confirmed /v1 replacement. § 13f: cms_snippets and cms_pages confirmed on production with their read shapes; cms_layouts and writes unchecked. Preview cap is on the longest side; page render includes bleed. Source: claude-chat, notion-page.
- 2026-10-06: § 6 image upload moved to `POST /upload/image` (data or url+name, optional gallery_id); old gallery images POST deprecated; gallery size guidance; admin-key gallery create for a user. § 7 GET `_uid` lookup by external ID. § 13c custom type instances full CRUD on /v1 (real PUT, merge update, delete 200 empty body); asset multipart upload verified; asset replace in place via admin form; multitext custom fields need array form; collections still have no /v1 write. § 13f snippet writes verified on production (CRLF, child-only snippet creation, JSON `cms_snippet` body saves nothing). § 13g code, name and custom fields proven writable. New § 13h Admin Form Writes (snippet, template options, variant types and values, collection image, custom fields, product order, remove product, add_themes false error, design name). Also from other groups' spill: § 3 users/me returns id null for a visitor; § 4 read a project's pages; § 6 single image read is 403 without a session, customer upload by script via `/upload/image` verified as guest; § 9 png extension, transparency and page mask via SVG, reading a design and a font; § 13c duplicate asset name returns 200 with an error body, design previews (linked assets), product image and description, design description. From group D2 spill: public `/v1/products.json` pages at 20 (`per_page` works), public catalog reads, the add_themes.json route. From late spills (groups B2, C, E): § 4 project update limits and the project gallery; § 6 copy an image between galleries, delete a gallery; § 13c asset PUT replace and DELETE on /v1 (corrected), upload response shape, admin-host upload, custom type admin-form routes, design custom field PUT saves nothing (corrected); § 13f child snippet overrides created and deleted by API, asset deletion verified; § 13g no storefront URL in the admin product list, collections read fields; § 13h snippet DELETE-form trap, Ace `<\/script>` unescape, snippet create with content, collection custom fields PATCH partial, design form and design custom fields route. Source: claude-chat, vault-doc.
- 2026-10-06 (later): § 6 `/upload/image`: `name` is optional and defaults to the last URL segment. Source: gmail (core developer, 2025).
- 2026-10-09: § 2 an admin.pixfizz.com login is not a storefront admin session (`user.is_admin` pages need a storefront sign-in). § 4 create with `book[template_options]`; `book[pages]` on create is honored and not clamped to the template minimum; guest `pages.json` read; new Add to the cart from a script (the `editor=add-to-cart` button value, posting `/cart/add_print_product`, one line per product). § 9 project preview: `page=` not `template_name`, unsaved `book[template_options]`, `px-project-preview` attributes. § 13 SiteFlow status callbacks (`siteflow_update.json`, lowercase statuses, no cross-account callbacks today). § 13g corrected: experimental `/v1/admin` variant writes opened 2026-10-06 (value price verified); quantity limits writable, `unit_intervals` not in the API. § 13h snippet content save routes disagree, iframe and Ace route; child override from the admin form. Source: claude-chat, notion-page, slack-message.
- 2026-10-17: § 1 admin-host assets list pages at 50, users at 20. § 3 admin users list: fixed 20 per page, addresses read, telephone-only PUT and its loud failures. § 4 calendar create (`start_month` zero-padded, `book[calendars][]`); projects cannot be deleted (not verified by query); `pages.json` paging. § 6 `_mine.json` redirect gives the user id. § 9 unpublished option keeps published values, template options only on the theme read, definition readable client-side. § 11 user calendar created by `POST /v1/calendars.json` (both observations kept, role question open). § 13c custom field definitions by admin form; instance JSON body must be wrapped; ~60 KB custom field size limit (422); admin-host duplicate asset 422; `?sitename=` ignored on a storefront origin; `/v1/admin/assets.json` 413 on large files (exact limit not verified); `/fz/` URLs host-signed and long-lived; signed-link fallback when the cdn fetch fails; an API key cannot add products to a collection. § 13d file upload options are `db:<id>` outside Liquid, generated files only on later events, `custom.order_source`, local `created_at`, no tip/staff/store fields. § 13g CSRF-free admin-host product PUT, `""`/`null` fulfillment read-back, design product create, collections read not paged with `sub_categories`, static products and themes per collection, lookup by full subcollection name, signed-out `price_forecast`, variant value rename. § 13h Product Attribute Ace field, admin CSP, snippet patch form (corrected: match `_method=patch`, not `snippet[name]`), Extra Fee create, `add_themes.json` corrected to the Publish Products wizard route (by hand, by URL, consumer-site product creation). Source: claude-chat.
- 2026-10-22: § 2 an admin page cannot call the site host (admin CSP); use `?sitename=` on the admin host or move the tab to the site domain. § 4 user books list fields with an admin key; no admin list of projects across users; book detail fields (`images[]`, `source_book_id`, `page_count`, `dpi`); new subsection: a saved project and its ordered project are two books (`ordered: false` does not mean not bought, link by `source_book_id`, webhook fallback); create takes `variants[<code>]` for unpublished variant types and substitutes text template options into the page XML; a preview request can bump `updated_at`. § 6 `/v1/images` needs an admin login on a site domain; new subsection: file uploads are content addressed. § 9 CORRECTED `width` (no width gave 150 x 150, not 100; `size`/`max_size` ignored); new `hide_placeholders` row (hides placeholder text too); preview is the preview set, empty placeholder color, theme preview not a usable tile, image upload option check by byte compare; new `layouts.json`; `preview.png` RGBA; print render `/v1/pages/<id>.jpg?fulfillment=true` scaled by width, omits placeholders, 500 on a stale mask id; theme JSON `images[]` and `target_element_names`. § 12 `/v1/fonts.json` shape. § 13c `.glb` refused with a 200 error body, `.json` and `.js` served with CORS from the CDN, C2PA chunk appended to WebP, admin-host image `/fz/` links load without login. § 13g product attribute create in one call, `product[units]` ignored on create; public `/v1/products.json` exposes custom fields and formula source (security consequence); links to another design product need the design id; `price_forecast` prices only what is sent. § 13h admin CSP blocks eval (inject with the page nonce); CORRECTED publish POST needs `print_themes` positions (0-based ranges and dot lists), not `print_theme_ids[]`; create a collection with Create and Tick in the wizard; hide a sub-collection with `custom[unpublished]`. § 7 merging users: `imsure=true` skips the warning, merge direction and the 403. § 8a new Draft orderlines (`draft`, `orderline.is_draft`, admin "(Draft)"). § 13d webhooks retry and deactivate after 20 consecutive failures, email, manual deactivation (commit titles only). § 13c Admin form route: select the form by a field and check `_method` before `new FormData(form)`. § 13h collection creation reconciled: `/theme_categories/new` form and Create and Tick both work. Source: claude-chat, slack-message.
- 2026-10-29: § 3 bulk user create with `POST /v1/users.json` (no CSV import, external_id trap, about 4% dropped in parallel bursts, `user.created` webhooks). § 4 a cart line from template options only (`/v1/books.json` plus `/cart/add_print_product`), File Upload option ids, upload elements on Pages pages, orderline delete and `/cart/clear` by script. § 6 `Filedata` as a second working field name on `/upload/image`, PDFs rasterized at 300 dpi, new `POST /upload/file`. § 9 font palettes, element geometry in the theme JSON, a space blanks a preview text. § 13c admin upload ceiling of 50 MB, storefront-issued `/fz/` URLs load cross-site (cross-site asset library), `asset_url` hashed per file with open CORS, CSRF header observation. § 13g `fulfillment` write, consumer products on a template provider. § 13h one more override seen as a copy; add an existing design product to another collection by replaying the publish form; collection Ace fields from `const content`. § 13c resize an asset in place (old URL keeps answering). Source: claude-chat.
- 2026-11-09: § 2 CORRECTED API keys (super user keys in the super admin reach every site the super user can access; site-admin keys reach one site); service users; new Pixfizz MCP connector paragraph. § 4 change a cart line's quantity by script (`orderline_update` with `id:`/`class:`), add-to-cart at a quantity. § 6 gallery custom fields writable; a photo subset as its own gallery; copy by URL keeps PNG alpha and works from another site's CDN asset; `/upload/file` copy by URL; a third `/v1/images` observation (staff account, pending). § 9 CORRECTED placeholder grey is per site (210,208,208 is the default only); `preview.webp` with `product_id` and `height` usable on non-photo designs; new Render speed (WebP fastest, private cache, no duplicate in-flight renders). § 11 what a user calendar date holds (64-char `date_type`, year kept, PUT and DELETE via `_method`) and what prints. § 12 child `/v1/fonts.json` lists the products parent's fonts. § 13c `.html` assets refused silently, prototype page as a `.js` asset in an iframe; admin 50 MB is a browser check (150 MB with `large_format`), raise agreed but not live; custom field definitions copied per object by `export_all?type=`, a new child has 0 definitions; custom types export by `custom_type_ids[]`. § 13d `order.fulfilled` event; super account and global webhooks with the wrapped payload. § 13g `price_forecast` with `variants[...]` and `variants_applied`. § 13h child override observations (empty on 2026-10-31, parent copy with content ignored on 2026-11-06, HX headers), template provider vs same-account publish (pending), byte-exact snippet edit from `const content`, variant value image, reorder and layout substitution routes, short template option create, collection merge confirmed again, collection preview images form, `unpublished` hides the tile only, site settings `website[title]`, shipping formula 0 is free, assets have no show page, slash in Create and Tick is flattened, fonts copy. Round 2: § 4 calendar start and holidays fixed at creation; `/cart/add_print_product` fetch needs `redirect: 'manual'`; minimal `/v1/books.json` create. § 5 production file from the admin project page. § 6 guest's own upload answers 500 on `/v1/images`. § 11 admin calendar forms (create, `create_date`, copy between sites). § 13g product custom values store without a definition. § 13h template import file as JSON (pointer). Source: claude-chat, notion-page, slack-message.
- 2026-11-13: Pixfizz MCP connector: set up from the Claude desktop app on Windows, ChatGPT not completed. New § 13i: snippet update in place by /v1 PUT (create with an existing name is 422), child override by form, Pages entry create and update, design placeholder image via /upload/resource, asset URLs are signed per host. Source: claude-chat, fireflies-call.
