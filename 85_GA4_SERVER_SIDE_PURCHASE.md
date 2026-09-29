# 85 — GA4 Server-Side Purchase

**Authority Scope:** The server-side GA4 `purchase` pipeline — what the storefront must supply, how the order webhook decides whether to send, what the skip reasons mean, what is admin-visible, and the known defects. Storefront-side GA4 tagging is in `50_SHOPPER_TEMPLATE_REFERENCE.md`; the platform side of the order webhook is in `61_PIXFIZZ_API.md`.

_Last updated: 2026-09-09_

---

## 1. What It Is

**Verified by reading source.** On brands wired to myPixfizz, the `purchase` event is sent to
Google Analytics 4 **server-side**, using the GA4 **Measurement Protocol**, from the myPixfizz
side rather than from the shopper's browser. It has been **live since March 2026**.

It is driven by the Pixfizz **order webhook**: the platform posts the order, myPixfizz decides
whether this order should produce a `purchase`, and if so builds the GA4 payload and sends it.

Two things follow immediately and both catch people out:

- **It is invisible from the storefront.** Nothing in the Shopper template mentions it, so a
  session that reasons about `purchase` from the storefront alone will conclude it does not
  exist. See §8.
- **It is per brand, not per platform.** A brand that is not wired sends nothing server-side. A
  brand that is wired sends server-side *in addition to* whatever the storefront still sends.
  See §7.

---

## 2. The Shopper Side — What the Storefront Must Supply

**Verified by reading source.** The storefront's entire contribution is a set of **order-level
custom fields**, all of type text:

| Field | Carries |
|---|---|
| `ga_client_id` | The GA4 client id read from the browser's `_ga` cookie |
| `ga_session_id` | The GA4 session id |
| `utm_source` | Campaign source at the time of the order |
| `utm_medium` | Campaign medium |
| `utm_campaign` | Campaign name |
| `utm_term` | Campaign term |
| `utm_content` | Campaign content |
| `order_source` | Where the order originated |

**Nothing works without them.** They are captured by the storefront and travel with the order into
the webhook payload. If they are absent the pipeline still sends revenue, but it sends it
unattributed — see the fabricated-client-id defect in §6.

These fields are set up on the storefront once, per site. A site with a broken analytics
bootstrap — no GA4 initialization, therefore no `_ga` cookie — has nothing to write into
`ga_client_id`, and no amount of server-side work can recover it.

**Where the capture happens (Shopper 24 parent).** `checkout/utm-cart-code`, included on `pages/cart`, writes `ga_client_id` and `ga_session_id` as cart custom fields, which promote to order custom fields at checkout. The UTM landing cookie `_pf_utm` is set by `integrations/google/utm-capture`. *Verified by reading source (shopper24 backup), 2026-09-28.* In the webhook payload the values sit at `raw_payload.custom.ga_client_id` and `raw_payload.custom.ga_session_id`; the client id is the `_ga` cookie value with the `GA1.1.` prefix stripped. *Verified by query of the event outbox, 2026-09-14.*

---

## 3. How the Webhook Function Decides

**Verified by reading source.** In order:

### Brand resolution

The incoming order is matched to a brand by, in order: **subdomain → brand id → website code**.
The first that resolves wins. A brand that cannot be resolved cannot be sent for.

**In production the only key present is the website code.** The real Pixfizz order webhook payload is flat (no `order` wrapper) and carries no subdomain and no brand id, only a top-level `website` holding the website code. A brand row without the correct `website_code` therefore never resolves: the webhook answers 404 and nothing reaches the outbox, not even a skipped row. *Verified by query across every `sent` event in a 30-day window, 2026-09-14.*

**Duplicate brand rows are dangerous, not untidy.** The customer-side connectors flow can create a second brand row for a storefront that already has one. Orders then resolve to whichever row holds the website code, which need not be the row carrying the GA4 configuration, and the admin Brand dialog and the customer brand screen can end up editing different rows (symptom: the two forms show different values). A live case was found and fixed on 2026-09-27/28 (verified by query).

- Before wiring GA4 for any brand, query the brands for that organization and check there is exactly one row per storefront.
- **Never hard-delete a brand row.** Merge or archive it. A delete cascades: the brand's saved API credential and its `labworks_integrations` rows are deleted (`ON DELETE CASCADE`), while its brand profile survives unlinked (`brand_id` set null). *Verified by query of the table constraints, 2026-09-28.* Hard delete was removed from the admin in the brand clean-up of 2026-09-28.

### Signature handling — observe, then enforce

The webhook signature is **always verified and always logged**, with the outcome recorded as one
of **`valid` / `invalid` / `missing_headers` / `no_secret`**. The request is **rejected only when
that brand has enforcement switched on**.

This is deliberate: it lets a brand's signature behavior be observed against real traffic before
enforcement is turned on, so enforcement is a one-field flip with evidence behind it rather than a
change that might silently stop a live feed. The corollary is that **enforcement being available
is not the same as enforcement being on** — check the brand, do not assume.

### The send rule

`purchase` is sent only when the order carries a **`confirmed_at`**. An order that arrives already
confirmed — the pay-in-store and counter cases — sends on creation. Everything else waits for the
status change that sets `confirmed_at`.

### Idempotency

Two layers, because one is not enough:

1. A unique **idempotency key** of the shape `ga4:<event>:<brand_id>:<order_id>`.
2. A **pre-check on the per-order sent timestamp** before anything is built or sent.

An order that has already been sent for is skipped at the pre-check and never reaches the key.

### Currency

**Currency comes from the brand's billing currency only. It is never taken from the order.**

If the brand has no billing currency set, the send is **skipped deliberately** rather than
defaulted. That is the correct behavior: a skipped purchase surfaces as a visible gap, whereas a
purchase sent in the wrong currency silently corrupts every revenue figure downstream and is not
retractable.

### Retry

A failed send is **retried on a short delay** rather than abandoned.

---

## 4. Skip Reasons, and What Each One Means

**Verified by reading source.** These appear per event in the admin GA4 Event Log.

| Skip reason | What it means | What to do |
|---|---|---|
| `already_sent` | This order has already produced a `purchase`. Idempotency working as designed. | Nothing. Expected volume, not a fault. |
| `order_created_not_confirmed` | The order arrived unconfirmed, so `purchase` correctly waits. | Nothing, unless a brand shows **only** this and never sends — then its orders are not reaching a confirmed state through the webhook at all. |
| `no_confirmed_at` | The event should have carried a confirmation timestamp and did not. | Check the order's status path on the platform side. |
| `missing_billing_currency` | The brand has no billing currency set. Deliberate skip, see §3. | Set the brand's billing currency. Sends resume immediately; the skipped orders are not replayed. |
| unknown event | The webhook delivered an event type the function does not handle. | Usually benign. Worth reading once, in case an event was renamed upstream. |

A status of **`disabled`** is different from a skip: it means **the brand's GA4 configuration is
missing or switched off**. Nothing will send for that brand until it is configured and enabled.

---

## 5. What Is Visible in Admin

**Verified by reading source.** The **GA4 Event Log** in the myPixfizz admin is the debugging
surface. Per event it shows:

- the **raw payload** received from the order webhook
- the **GA4 payload** that was built from it
- the **client id** used
- the **attempt history** — every send attempt and its outcome

That is enough to answer "did this order send, and if not why" without a database query, and it is
the first place to look on any report of missing revenue in GA4.

### Where the webhook secret is set

The **Webhook Secret** exists only in the admin Brand dialog: myPixfizz admin → Brands → open the brand → **Integrations** tab → **Pixfizz Webhook** section. The **Test Webhook** button appears there only once a secret is set. The customer-side "Google Analytics 4" card carries only Enabled, Measurement ID, API secret and Debug mode, so a customer setting up GA4 on their own cannot set the secret (product gap). Signature enforcement was off on every brand checked, so a missing secret logs `no_secret` and does not block sends today; set it anyway. *Verified by reading source (`BrandDialog.tsx`) and by screenshot, 2026-09-27.*

### What Test Webhook proves, and what it does not

- The test payload carries a brand id and subdomain and **no `website`**, so it resolves by brand id. It does **not** prove website-code resolution, which is what real orders use (§ 3).
- It carries no `confirmed_at`, so it is **always skipped with `no_confirmed_at`**. That skip is the expected result, not a fault.
- It sends no signature headers, so it always logs `missing_headers`. Harmless while enforcement is off; it would be rejected once enforcement is on.
- **The real proof is the first live order on the brand that reaches Confirmed** and shows `sent` in the Event Log.

*Verified by query of the event outbox and the signature log, 2026-09-28.* Product gap: Test Webhook should send a confirmed order, the `website` code and signed headers, so it exercises the real path.

---

## 6. Known Defects, as at 2026-09-09

All three are **verified by query**. None is fixed. Read this section before trusting an item-level
or attribution figure from a wired brand.

### `item_id` is the numeric internal product id

Server-side, `item_id` is the **numeric internal product id**. Every client-side event uses the
**product code**.

Consequences, both of them real reporting damage:

- **Item-level funnels never join.** `view_item` → `add_to_cart` → `purchase` cannot be followed
  for any product, because the purchase step identifies the item differently from every step
  before it.
- **Item reports carry two rows per product** — one keyed by code, one a bare integer.

**Blocked, and not by a one-line change: the webhook payload does not carry a product code at
all.** The platform side has to add it before the server side can use it. Until then any change is
a guess. See `61_PIXFIZZ_API.md` §13d.

### A fabricated client id where the storefront captured no `_ga` cookie

When `ga_client_id` is absent, the function **invents a client id** and **omits the session id**
entirely. Revenue is still counted. Attribution is not: those purchases land in GA4 as
**Direct / (not set)**.

The two symptoms are one symptom — an event with no session id is an event with a fabricated
client id, and vice versa.

**The root cause is a broken storefront analytics bootstrap, and nothing server-side can fix it.**
If GA4 never initializes on the storefront there is no `_ga` cookie to read, so `ga_client_id` is
empty by construction. Fix the storefront's analytics bootstrap and the attribution returns.
Treat a brand with a high share of fabricated client ids as a **storefront** ticket, not a
myPixfizz one.

### No event timestamp is sent

The payload carries **no event timestamp**, so **GA4 stamps every purchase at the moment it is
received**.

That matters because measured **order-to-send delays run well outside GA4's 24-hour
session-attribution window**, and GA4's general backdating is capped at **72 hours**. So:

- events sent within the window are fine;
- events sent late lose session attribution purely on timing, even when the client id and session
  id are perfectly formed;
- events beyond the backdating cap cannot be rescued by adding a timestamp at all — those need the
  send to happen earlier, which is a product decision about what "purchase" means and not a bug.

---

## 7. The Double-Count Warning

**Verified live.** On a brand wired to myPixfizz, `purchase` arrives at GA4 from the server.
**If `purchase` is also left in the storefront container's trigger regex, revenue is
double-counted.**

Before trusting any revenue figure from a wired brand, check the container. This is the single
most common reason a wired brand's GA4 revenue does not reconcile with the platform's own numbers,
and it inflates rather than deflates, so it does not look like a fault.

**Before enabling a brand, compare Measurement IDs.** Read the GA4 Measurement ID the live storefront actually loads (usually through its GTM container) and compare it with the Measurement ID on the myPixfizz brand. A mismatch sends server-side revenue to a different property from the browser events. One wired brand was found in this state. *Verified live in the browser, 2026-09-28.*

The storefront-side tagging standard, and the "a container with no GA4 event tags shows traffic
and no revenue" trap that is its mirror image, are in `81_SEO_AND_GEO_REFERENCE.md` Part G and
`50_SHOPPER_TEMPLATE_REFERENCE.md`.

---

## 8. Read the System That Would Own It

A research rule, earned here and worth applying generally.

**A question framed as Shopper GA4 has its answer in the myPixfizz files.** A session working on
storefront GA4 searched the Shopper documentation, found no mention of a server-side `purchase`,
and proposed building what had already been live for six months. The search returned relevant
hits; none of them mentioned the pipeline, and that absence read as evidence of absence rather
than as the wrong search.

**Before proposing any build, search the knowledge base for the *system that would own the
capability*, not only the system the question arrived about.** Ownership follows the data, not the
user interface the question came through: revenue-bearing events are owned by whatever holds the
order after checkout, which is not the storefront.

---

## 9. Retrieval Pointer

| Topic | File |
|---|---|
| Storefront GA4 tagging, the container standard, checklist keys | `50_SHOPPER_TEMPLATE_REFERENCE.md` |
| The wider myPixfizz system, integrations, RLS and webhook design rules | `70_MYPIXFIZZ_OVERVIEW.md` |
| Order webhook registration and payload shape | `61_PIXFIZZ_API.md` |
| Order statuses and what "confirmed" means | `32_ORDER_LIFECYCLE.md` |
| SEO and analytics setup traps | `81_SEO_AND_GEO_REFERENCE.md` |

---

## Changelog
- 2026-09-09: Created. Documents the server-side GA4 `purchase` pipeline, live since March 2026 and previously covered by one line in `70_MYPIXFIZZ_OVERVIEW.md`: the Shopper order custom fields the storefront must supply; brand resolution order; observe-then-enforce signature handling; the `confirmed_at` send rule; the two idempotency layers; billing-currency-only currency with a deliberate skip when it is missing; short-delay retry; the skip reasons and the `disabled` status; the admin GA4 Event Log; the three known defects (numeric `item_id` blocked on the webhook payload carrying no product code, a fabricated client id where the storefront captured no `_ga` cookie, and no event timestamp against GA4's 24-hour attribution window and 72-hour backdating cap); the double-count warning for wired brands; and the "read the system that would own it" research rule. Source: claude-chat.
- 2026-09-29: §2 named the capture snippets and where the values land in the payload. §3 production resolves by website code only; duplicate brand rows and never hard-delete a brand. §5 where the webhook secret is set, and what Test Webhook does and does not prove. §7 compare the storefront's loaded Measurement ID with the brand's before enabling. Source: claude-chat.
