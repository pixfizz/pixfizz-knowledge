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
