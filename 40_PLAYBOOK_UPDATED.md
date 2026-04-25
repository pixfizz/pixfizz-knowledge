# 40 --- Playbook

**Authority Scope:** Operational reasoning and troubleshooting guidance.

*Last updated: 2026-04-23*

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

The Pixfizz editor runs inside an iframe. Styles delivered via
`style/custom.css` (the storefront CSS page) do not reach inside the
iframe.

Symptoms:
- CSS rules targeting `.px-header`, `.px-cart-button`, or other
  `px-*` classes have no effect
- JS-driven label/content changes inside the editor cannot be
  overridden with CSS

If a client reports a styling issue inside the editor, CSS is not the
solution. Raise with Pixfizz platform team.

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

## Changelog
- 2026-03-21: Initial content from platform documentation export.
- 2026-04-23: Added CSS snippet logs diagnostic note, password reset Liquid deprecation pattern, fulfillment template DPI failure, URL reserved parameter 404 gotcha, Stripe pending-without-payment issue, FTP original files intermittent failure.
