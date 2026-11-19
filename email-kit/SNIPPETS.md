# Email kit: snippet list

Every snippet in the kit, with the Description it has on the Shopper parent. The file for `email-kit/<folder>/<name>` is `snippets/<folder>/<name>.liquid` in this folder.

## templates

| Snippet | What it does |
|---|---|
| `email-kit/templates/cart-abandoned` | Cart reminder email, sends 1 to 3, optional discount code and help box. Admin Body passes cart, user and sequence_number. |
| `email-kit/templates/order-canceled` | Order Canceled email. Admin Body passes order and user. |
| `email-kit/templates/order-confirmed` | Order Confirmed email. Admin Body passes order and user. |
| `email-kit/templates/order-downloaded` | Order Downloaded (in production) email. Admin Body passes order and user. |
| `email-kit/templates/order-draft` | Order Draft email with a 3-month personalize link. Admin Body passes order and user. |
| `email-kit/templates/order-error` | Order Error team email with an admin link. Admin Body passes order and user. |
| `email-kit/templates/order-fulfilled` | Order Fulfilled team email with an admin link. Admin Body passes order and user. |
| `email-kit/templates/order-manufactured` | Order Manufactured (finished) email. Admin Body passes order and user. |
| `email-kit/templates/order-pending` | Order Pending email. Admin Body passes order and user. |
| `email-kit/templates/order-shipped` | Order Shipped email, with tracking for shipped orders and pickup details for store pickup. Admin Body passes order and user. |
| `email-kit/templates/orderline-fulfilled` | Orderline Fulfilled team email for one line. Admin Body passes orderline, order and user. |
| `email-kit/templates/password-reset` | Password reset email. Admin Body passes reset_url and user. |
| `email-kit/templates/signin-token-renewed` | New sign-in link email. Admin Body passes signin_token and user. |
| `email-kit/templates/user-signup` | Welcome email for a new account. Admin Body passes user. |

## subjects

| Snippet | What it does |
|---|---|
| `email-kit/subjects/cart-abandoned` | Subject of the Cart Abandoned email, sends 1 to 3. Text from email-kit/cart/subject-1, -2, -3. |
| `email-kit/subjects/order-canceled` | Subject of the Order Canceled email. |
| `email-kit/subjects/order-confirmed` | Subject of the Order Confirmed email. |
| `email-kit/subjects/order-downloaded` | Subject of the Order Downloaded email. |
| `email-kit/subjects/order-draft` | Subject of the Order Draft email. |
| `email-kit/subjects/order-error` | Subject of the Order Error team email. |
| `email-kit/subjects/order-fulfilled` | Subject of the Order Fulfilled team email. |
| `email-kit/subjects/order-manufactured` | Subject of the Order Manufactured email. |
| `email-kit/subjects/order-pending` | Subject of the Order Pending email. |
| `email-kit/subjects/order-shipped` | Subject of the Order Shipped email, with a store pickup version. |
| `email-kit/subjects/orderline-fulfilled` | Subject of the Orderline Fulfilled team email. |
| `email-kit/subjects/password-reset` | Subject of the Password Reset email. |
| `email-kit/subjects/signin-token-renewed` | Subject of the Sign-in Token Renewed email. |
| `email-kit/subjects/user-signup` | Subject of the User Signup email. |

## messages

| Snippet | What it does |
|---|---|
| `email-kit/messages/cart-abandoned` | Main message of the first cart reminder. Plain text, tokens {first_name} {store_name}. |
| `email-kit/messages/cart-abandoned-2` | Main message of the second cart reminder. Plain text, tokens {first_name} {store_name}. |
| `email-kit/messages/cart-abandoned-3` | Main message of the third cart reminder. Plain text, tokens {first_name} {store_name} {discount}. |
| `email-kit/messages/order-canceled` | Main message of the Order Canceled email. Plain text, tokens {first_name} {order_code} {store_name}. |
| `email-kit/messages/order-confirmed` | Main message of the Order Confirmed email. Plain text, tokens {first_name} {order_code} {store_name}. |
| `email-kit/messages/order-downloaded` | Main message of the Order Downloaded email. Plain text, tokens {first_name} {order_code} {store_name}. |
| `email-kit/messages/order-draft` | Main message of the Order Draft email. Plain text, tokens {first_name} {order_code} {store_name}. |
| `email-kit/messages/order-error` | Main message of the Order Error team email. Plain text, tokens {first_name} {order_code} {store_name}. |
| `email-kit/messages/order-fulfilled` | Main message of the Order Fulfilled team email. Plain text, tokens {first_name} {order_code} {store_name}. |
| `email-kit/messages/order-manufactured` | Main message of the Order Manufactured email. Plain text, tokens {first_name} {order_code} {store_name}. |
| `email-kit/messages/order-pending` | Main message of the Order Pending email. Plain text, tokens {first_name} {order_code} {store_name}. |
| `email-kit/messages/order-shipped` | Main message of the Order Shipped email for shipped orders. Plain text, tokens {first_name} {order_code} {store_name}. |
| `email-kit/messages/order-shipped-pickup` | Main message of the Order Shipped email for store pickup orders. Plain text, tokens {first_name} {order_code} {store_name}. |
| `email-kit/messages/orderline-fulfilled` | Main message of the Orderline Fulfilled team email. Plain text, tokens {first_name} {order_code} {store_name}. |
| `email-kit/messages/password-reset` | Main message of the Password Reset email. Plain text, tokens {first_name} {store_name}. |
| `email-kit/messages/signin-token-renewed` | Main message of the Sign-in Token Renewed email. Plain text, tokens {first_name} {store_name}. |
| `email-kit/messages/user-signup` | Main message of the User Signup email. Plain text, tokens {first_name} {store_name}. |

## cart

| Snippet | What it does |
|---|---|
| `email-kit/cart/heading-1` | Heading of the first cart reminder. Plain text. |
| `email-kit/cart/heading-2` | Heading of the second cart reminder. Plain text, token {discount} when this send carries the code. |
| `email-kit/cart/heading-3` | Heading of the third cart reminder. Plain text, token {discount} when this send carries the code. |
| `email-kit/cart/help-link-label` | Label of the site link button in the cart help box, for example Guidelines Library. Blank (default) hides the button. |
| `email-kit/cart/help-link-url` | Address of the site link button in the cart help box, a path like /site/faq or a full URL. Blank (default) hides the button. |
| `email-kit/cart/help-title` | Title of the help box on the second cart reminder. Plain text. |
| `email-kit/cart/preheader-1` | Inbox preview line of the first cart reminder. Plain text. |
| `email-kit/cart/preheader-2` | Inbox preview line of the second cart reminder. Plain text, token {discount} when this send carries the code. |
| `email-kit/cart/preheader-3` | Inbox preview line of the third cart reminder. Plain text, token {discount} when this send carries the code. |
| `email-kit/cart/subject-1` | Subject of the first cart reminder. Plain text. |
| `email-kit/cart/subject-2` | Subject of the second cart reminder. Plain text, token {discount} when this send carries the code. |
| `email-kit/cart/subject-3` | Subject of the third cart reminder. Plain text, token {discount} when this send carries the code. |

## settings

| Snippet | What it does |
|---|---|
| `email-kit/settings/admin-site-key` | Site key used in the admin order link in team emails. Blank (default) takes it from a pixfizz.com host name. |
| `email-kit/settings/banner-alt` | Alt text for the email banner, shown when images are blocked. Blank by default. |
| `email-kit/settings/banner-link` | Web address the email banner links to. Blank (default) links to the store home page. |
| `email-kit/settings/banner-on` | Shows the email banner (asset email-banner.jpg or email-banner.png) on customer order and welcome emails. TRUE or FALSE (default). |
| `email-kit/settings/brand-color` | Email button and link color as a hex code, for example #1f6f5c. Blank (default) uses style/color-primary. |
| `email-kit/settings/button-text-color` | Email button text color as a hex code. Blank (default) picks dark or white text automatically from the brand color. |
| `email-kit/settings/cart-discount` | Discount code on a cart reminder, valid 24 hours. Blank (default) for no code, or 5, 10, 15 or 20 for that percentage off. The reminder is set in email-kit/settings/cart-discount-reminder. |
| `email-kit/settings/cart-discount-reminder` | Which cart reminder carries the discount code from email-kit/settings/cart-discount. 2 (default) or 3. |
| `email-kit/settings/cart-help` | Shows the help box with call, email and site link buttons on the second cart reminder. TRUE or FALSE (default). |
| `email-kit/settings/logo-width` | Email logo width in pixels, 60 to 300. Default 160. |

## parts

| Snippet | What it does |
|---|---|
| `email-kit/parts/address` | Shipping address and method, or pickup location and opening hours, in an email. |
| `email-kit/parts/admin-url` | Prints the admin link to an order for team emails. |
| `email-kit/parts/button` | Email call-to-action button in the brand color. Prints nothing when the link is blank. |
| `email-kit/parts/heading` | Email heading with optional order number and the internal badge for team emails. |
| `email-kit/parts/help` | Help box on the second cart reminder with call, email and one site link buttons. Prints nothing when there is nothing to link to. |
| `email-kit/parts/info-box` | Shaded box in an email for tracking numbers, discount codes and order facts. |
| `email-kit/parts/line` | One order or cart line in an email with preview, options, quantity and optional price. |
| `email-kit/parts/lines` | Table of order or cart lines in an email, with or without prices. |
| `email-kit/parts/link-fallback` | Prints a button link as plain text for mail apps that block buttons. |
| `email-kit/parts/message` | Editable main message of an email with the tokens {first_name}, {order_code}, {store_name} and {discount} filled in. |
| `email-kit/parts/note` | Short paragraph under the message in an email. Normal or small text. |
| `email-kit/parts/totals` | Order totals in an email. Subtotal, shipping, fees, tax, discount and total. |
| `email-kit/parts/tracking-url` | Prints the carrier tracking link for an order (UPS, USPS, FedEx, DHL or a full URL), or nothing. |

## brand

| Snippet | What it does |
|---|---|
| `email-kit/brand/banner` | Email banner row when email-kit/settings/banner-on is TRUE and a banner image exists. Otherwise prints nothing. |
| `email-kit/brand/button-text` | Resolved email button text color for the brand color passed in. Setting first, else automatic dark or white. |
| `email-kit/brand/color` | Resolved email brand color. Reads email-kit/settings/brand-color, then style/color-primary, else #1f2933. |
| `email-kit/brand/hex-value` | Helper that prints the number 0 to 15 for one hex digit. Used by email-kit/brand/button-text. |
| `email-kit/brand/logo` | Email logo linked to the store. Uses asset email-logo.png, then logo.png, else the store name as text. |

## store

| Snippet | What it does |
|---|---|
| `email-kit/store/address` | One-line store address for emails from website/contact/address, city, state and zip-code. |
| `email-kit/store/email` | Support email for emails from website/contact/support-email-address. Prints nothing for the shopper24 placeholder. |
| `email-kit/store/hours` | One-line store hours for emails from website/contact/support-hours. Prints nothing for the shopper24 placeholder. |
| `email-kit/store/name` | Store name for emails from website/contact/title. Falls back to the host name when blank or still My Brand. |
| `email-kit/store/phone` | Support phone for emails from website/contact/support-telephone. Prints nothing for the shopper24 placeholder. |

## (root)

| Snippet | What it does |
|---|---|
| `email-kit/head` | Email HTML head with mobile, dark mode and Outlook styles. Called by email-kit/layout. |
| `email-kit/layout` | Email shell for every kit email. Preheader, logo, banner, body, store details and footer. |

Total: 92 snippets.
