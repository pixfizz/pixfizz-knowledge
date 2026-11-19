# Shopper email kit

The notification emails for a Pixfizz Shopper site: order confirmation, shipping, store pickup, cart reminders, account emails and team alerts. 14 emails, one shared design, your logo and brand color, copy you can change without touching code.

This folder holds the complete current kit so you can read every template, set it up on your own site, and change the wording or design yourself.

- **Kit version:** 1.1.x (cart reminders 1 to 3, optional discount code, help box). Copied from the Shopper parent on 2026-11-19.
- **Level:** template-level (Shopper). The snippets live on the Shopper parent, so every Shopper site can call them. Your site's 14 email templates are per site and are not inherited (`13_TEMPLATE_BOUNDARIES.md`).
- **Behavior and rules:** `32_ORDER_LIFECYCLE.md` § How notification email templates behave and § The Shopper email kit.

| File | What it is |
|---|---|
| `README.md` | This guide: setup per site and how to customize |
| `SNIPPETS.md` | Every kit snippet with what it does |
| `snippets/` | The Liquid of every snippet. `email-kit/parts/line` is `snippets/parts/line.liquid` |

## How the kit works

Each email template in your admin holds one line for the Subject and one line for the Body. Those lines call kit snippets on the Shopper parent:

- `email-kit/templates/<email>` builds the email from parts (heading, message, button, order lines, totals, address).
- `email-kit/layout` and `email-kit/head` are the shell every email shares: inbox preview line, logo, optional banner, store details, footer, mobile and dark mode styles.
- `email-kit/messages/*`, `email-kit/subjects/*` and `email-kit/cart/*` hold the words. Messages are plain text with tokens `{first_name}`, `{order_code}`, `{store_name}` and, on cart reminders, `{discount}`.
- `email-kit/settings/*` are switches and values (brand color, logo width, banner, cart discount).
- `email-kit/store/*` read your store details from the `website/contact/*` snippets, the same ones your storefront footer uses.

Because your templates only call the snippets, an improvement to the kit on the Shopper parent reaches your emails automatically, unless you have overridden that snippet on your site.

## Set up the emails on your site

### 1. Sender settings

In admin, Settings > Email Notifications:

- **Default Email (From):** an address on your own domain, for example `orders@yourdomain.com`. Not a Gmail or other free address, and not the placeholder `info@yoursite.com`. Your domain needs sending authentication or mail lands in spam: ask Pixfizz support (my.pixfizz.com) to set it up, then add the DNS records they send you.
- **Orders Email:** your order inbox. Team emails go here.
- **HTML Format:** on.

### 2. Store details and logo

- Fill the snippets `website/contact/title`, `address`, `city`, `state`, `zip-code`, `support-telephone`, `support-email-address` and `support-hours`. Emails show your name, address, phone, email and hours from these. They also change your storefront footer.
- Upload a transparent PNG logo, about 960 px wide, as the asset `email-logo.png`. Without it the kit uses `logo.png`, else your store name as text. Do not use an SVG logo: Gmail and Outlook do not show SVG.

### 3. The 14 templates

For each email, open it in Settings > Email Notifications and paste the Subject and Body below. **An email is on when its Body is not empty. To keep an email off, leave its Body empty.** "Default" is what Pixfizz recommends.

| Email | Default | To | Subject | Body |
|---|---|---|---|---|
| Order Pending | On | leave empty | `{% snippet 'email-kit/subjects/order-pending', order: order %}` | `{% snippet 'email-kit/templates/order-pending', order: order, user: user %}` |
| Order Draft | On | leave empty | `{% snippet 'email-kit/subjects/order-draft', order: order %}` | `{% snippet 'email-kit/templates/order-draft', order: order, user: user %}` |
| Order Confirmed | On | leave empty | `{% snippet 'email-kit/subjects/order-confirmed', order: order %}` | `{% snippet 'email-kit/templates/order-confirmed', order: order, user: user %}` |
| Order Downloaded | Off | leave empty | `{% snippet 'email-kit/subjects/order-downloaded', order: order %}` | `{% snippet 'email-kit/templates/order-downloaded', order: order, user: user %}` |
| Order Manufactured | Off | leave empty | `{% snippet 'email-kit/subjects/order-manufactured', order: order %}` | `{% snippet 'email-kit/templates/order-manufactured', order: order, user: user %}` |
| Order Shipped | On | leave empty | `{% snippet 'email-kit/subjects/order-shipped', order: order %}` | `{% snippet 'email-kit/templates/order-shipped', order: order, user: user %}` |
| Order Fulfilled (team) | Off | your Orders Email | `{% snippet 'email-kit/subjects/order-fulfilled', order: order %}` | `{% snippet 'email-kit/templates/order-fulfilled', order: order, user: user %}` |
| Order Error (team) | On | your Orders Email | `{% snippet 'email-kit/subjects/order-error', order: order %}` | `{% snippet 'email-kit/templates/order-error', order: order, user: user %}` |
| Order Canceled | On | leave empty | `{% snippet 'email-kit/subjects/order-canceled', order: order %}` | `{% snippet 'email-kit/templates/order-canceled', order: order, user: user %}` |
| Orderline Fulfilled (team) | Off | your Orders Email | `{% snippet 'email-kit/subjects/orderline-fulfilled', order: order %}` | `{% snippet 'email-kit/templates/orderline-fulfilled', orderline: orderline, order: order, user: user %}` |
| Cart Abandoned | On | leave empty | `{% snippet 'email-kit/subjects/cart-abandoned', sequence_number: sequence_number %}` | `{% snippet 'email-kit/templates/cart-abandoned', cart: cart, user: user, sequence_number: sequence_number %}` |
| User Signup | On | leave empty | `{% snippet 'email-kit/subjects/user-signup' %}` | `{% snippet 'email-kit/templates/user-signup', user: user %}` |
| Password Reset | On | leave empty | `{% snippet 'email-kit/subjects/password-reset' %}` | `{% snippet 'email-kit/templates/password-reset', reset_url: reset_url, user: user %}` |
| Sign-in Token Renewed | On | leave empty | `{% snippet 'email-kit/subjects/signin-token-renewed' %}` | `{% snippet 'email-kit/templates/signin-token-renewed', signin_token: signin_token, user: user %}` |

Notes:

- **Orderline Fulfilled** is not on the Email Notifications list page. Open `/admin/site/email/email_template/orderline-fulfilled` on your admin.
- **Cart Abandoned** sends nothing until its Schedule has at least one row. Each row is one reminder, in hours after the cart was left (1 to 168). The kit handles up to 3 reminders. Pixfizz default: 1 and 48 hours.

### 4. Check before you go live

1. Open Preview on each email that is on, with a real order code from your site. Check the store name, logo, button color, order lines and totals, and that no "Liquid error" shows.
2. Send a test of Order Confirmed to yourself and check it reaches the inbox, not spam, in Gmail and Outlook.
3. If you use OrderHub notifications, make sure the customer does not get two emails for the same event.

Do not preview the cart reminder that carries a discount on a live site with a cart discount set: Preview creates a real promo code.

## Customize

### Settings (no code)

Create a child override of the setting snippet on your site and put the value in it. Leave it blank for the default.

| Setting | Values |
|---|---|
| `email-kit/settings/brand-color` | Hex code, for example `#1f6f5c`. Blank (default) uses `style/color-primary` |
| `email-kit/settings/button-text-color` | Hex code. Blank (default) picks dark or white automatically |
| `email-kit/settings/logo-width` | 60 to 300. Default 160 |
| `email-kit/settings/banner-on` | `TRUE` or `FALSE` (default). Shows asset `email-banner.jpg` or `email-banner.png` on customer order and welcome emails |
| `email-kit/settings/banner-link` | Web address for the banner. Blank (default) links to your home page |
| `email-kit/settings/banner-alt` | Banner alt text. Blank by default |
| `email-kit/settings/cart-discount` | Blank (default) for no code, or `5`, `10`, `15` or `20` percent off, valid 24 hours |
| `email-kit/settings/cart-discount-reminder` | Which cart reminder carries the code: `2` (default) or `3` |
| `email-kit/settings/cart-help` | `TRUE` or `FALSE` (default). Help box on the second cart reminder |
| `email-kit/settings/admin-site-key` | Site key for the admin order link in team emails. Blank (default) reads it from a pixfizz.com host name |

The cart help box shows Call and Email us buttons from your store details, plus one site link from `email-kit/cart/help-link-label` and `email-kit/cart/help-link-url` (a path like `/site/faq` or a full URL).

### Words (no code)

Override any of these on your site with your own plain text:

- `email-kit/messages/<email>`: the main message of each email. Tokens `{first_name}`, `{order_code}`, `{store_name}`; cart reminders also `{discount}`. A customer with no first name gets "Hi there".
- `email-kit/cart/subject-N`, `heading-N`, `preheader-N` (N = 1, 2 or 3): subject, heading and inbox preview line of each cart reminder.

### Design and layout (Liquid)

To change more than words, override the snippet on your site with your own version, starting from the file in `snippets/`. Keep the snippet name and the arguments it receives (listed in the comment at the top of each file). Rules for Liquid in emails:

- Use `currency`, not `money`, and absolute URLs for images and links (`50_LIQUID_REFERENCE.md`).
- Use PNG or JPG images. Inline SVG does not show in Gmail or Outlook.
- Keep inline styles: many mail apps ignore `<style>` blocks.
- An override stops your site getting kit improvements for that snippet. Override only what you change.

## Corrections

Something wrong or missing here: raise it in my.pixfizz.com, or email support@pixfizz.com.
