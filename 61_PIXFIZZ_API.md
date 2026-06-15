# 61 — Pixfizz API Reference

**Authority Scope:** Pixfizz REST API (v1), JS API, user handoff, project/fulfillment endpoints, dynamic previews, and custom eCommerce integration.

_Last updated: 2026-03-30. Compiled from Pixfizz Notion wiki (API section)._

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
