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
- 2026-07-28: Added Collection Filter Drilldown blank-PDP entry — stale or invalid dependent filter values break the drilldown; fix is a three-tier selection cascade in `product/product-details-filter` and `product/details-filter-dual-mode`. Source: claude-chat.
