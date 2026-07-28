# 13 — Template Responsibility Boundaries

**Authority Scope:** Defines CMS vs Shopper vs Custom vs Shopify responsibility.

_Last updated: 2026-03-13_

---

# 07 — Template Responsibility Boundaries

## Pixfizz CMS platform
- Objects, identity model, pricing engine, admin + custom fields.
- Applies to all deployment paths.

## Shopper template
- Cart UX and checkout policy engine (kiosk mode, shipping/pickup availability, film, digital-only logic).
- Only applies to sites using the Shopper template.

## Custom storefronts
- May reuse Pixfizz CMS but differ from Shopper behavior.

## Shopify deployment path
- Shopify owns the storefront, cart, checkout, and payment.
- Pixfizz owns personalization, project storage, and production orchestration.
- Cart and checkout rules from Shopper files (20/21) do not apply.
- Integration is handled via Shopify Liquid snippets, product metafields, and a JS API loaded from the Pixfizz subdomain.
- See **60_SHOPIFY_INTEGRATION.md** for all Shopify-specific implementation details.

## Where to implement changes

| Change type | Shopper sites | Shopify sites |
|---|---|---|
| Feature toggles / policy | Admin/Custom Admin checklist | Shopify product metafields |
| Cart/checkout rendering | Liquid snippets (Shopper) | Shopify theme snippets |
| Personalization launch | Pixfizz CMS product pages | Shopify product template + `pixfizz-launch-product-handler` |
| JS interaction | Interaction only — do not replace pricing/object truth | Pixfizz JS API (`Pixfizz.Shopify.*`) |
| Pricing | Pixfizz pricing formulas (Ruby) | Shopify product/variant prices + Pixfizz addon products |
| Order confirmation | Pixfizz checkout flow | Shopify payment webhook → Pixfizz |

## Custom Fields — No Parent→Child Inheritance

Custom fields are **site-specific**. A custom field created on the parent template site (e.g. `shopper24.pixfizz.com`) does **not** automatically appear on child sites.

Each site manages its own custom fields independently. If a new layout feature depends on a custom field (e.g. `product.custom.hide_gallery`), that field must be created manually on each child site that needs it.

This applies to all custom field object types: product, collection, design, user, order, and orderline.

RATIONALE: Confirmed during a client hide_gallery implementation — field created on parent was not present on child.
SOURCE: "Custom field to hide product gallery" chat, April 24

## Site Assets — Inherited Parent→Child

Site assets (files uploaded under **Main Admin > Website > Assets**: JS, CSS, images, fonts) **are** inherited parent→child, the same way snippets are. An asset uploaded to the parent template site resolves on every child site without being re-uploaded.

This corrects the entry previously held in this section, which stated the opposite. It was confirmed on a new child-site deployment of a custom design tool: the asset resolved correctly and the real fault was elsewhere.

**The failure mode is the include, not the asset.** A script asset is normally referenced from `integrations/custom-body-scripts`. That snippet is routinely overridden on child sites to carry their own analytics and third-party tags, so a `<script>` include added to the parent copy is invisible on any child that has an override. The symptom is identical to a missing asset — the page renders, the script never loads, no error.

Two rules follow:

- Where a tool depends on a script asset, have the tool's own product snippet load its dependencies rather than relying on a shared include snippet that child sites override. See **41_IMPLEMENTATION_PATTERNS_UPDATED.md § Custom Tool Dependency Loading**.
- Assets are aggressively browser-cached. When a JS asset is re-uploaded, expose a version marker on the script's public namespace (e.g. `MyTool.version`) so the deployed build can be confirmed from the browser console in one step, rather than guessing whether a change failed or is simply cached.

Note the asymmetry with custom fields: snippets and assets **are** inherited parent→child; custom fields are **not**. Do not generalise from one to the other.

## Parent-First Rule — What It Does Not Cover

Child sites can only override snippets that already exist on the parent; they cannot create net-new snippets. See **01_CODE_GOVERNANCE**.

That constraint applies to **snippets only**. It does not apply to:

- **Product template options and variants** — these are product data, created directly on whichever site owns the product. There is no parent equivalent to create first.
- **Custom fields** — created per site (see above).
- **Site assets** — uploaded on the parent and inherited.

Treating template options and variants as parent-first produces unnecessary edits on `shopper24.pixfizz.com`, which carry blast radius across every child site for no benefit.

## Changelog
- 2026-03-13: Added Shopify deployment path as a distinct boundary layer.
- 2026-07-25: Added Site Assets — No Parent→Child Inheritance, including the silent-failure mode for JS assets and the version-marker practice for confirming a deployed build past browser cache. Source: claude-chat.
- 2026-07-28: Corrected Site Assets section — assets ARE inherited parent to child; the silent failure previously attributed to asset inheritance is a child override of `integrations/custom-body-scripts`. Added Parent-First Rule scope note (snippets only, not template options or variants). Source: claude-chat.
