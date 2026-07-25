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

## Site Assets — No Parent→Child Inheritance

Site assets (files uploaded under **Main Admin > Website > Assets**: JS, CSS, images, fonts) behave the same way as custom fields. An asset uploaded to the parent template site is **not served to child sites**. Each site holds its own asset library.

This matters most for JavaScript delivered as a site asset. A build uploaded to the parent and referenced from an inherited snippet will resolve to nothing on the child, and the failure is silent — the snippet renders, the script simply never loads.

Two rules follow:

- Upload the asset to **every site that needs it**, not just the parent.
- Assets are aggressively browser-cached. When a JS asset is re-uploaded, expose a version marker on the script's public namespace (e.g. `MyTool.version`) so the deployed build can be confirmed from the browser console in one step, rather than guessing whether a change failed or is simply cached.

Note the asymmetry with snippets: snippets **are** inherited parent→child (and child sites can only override existing parent snippets, never create new ones). Custom fields and assets are not inherited. Do not generalise from one to the other.

## Changelog
- 2026-03-13: Added Shopify deployment path as a distinct boundary layer.
- 2026-07-25: Added Site Assets — No Parent→Child Inheritance, including the silent-failure mode for JS assets and the version-marker practice for confirming a deployed build past browser cache. Source: claude-chat.
