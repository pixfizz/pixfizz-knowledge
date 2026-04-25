# 12 — Objects & Ownership

**Authority Scope:** Object relationships and ownership rules only.

_Last updated: 2026-02-26_

---

# 03 — Objects & Ownership

All objects attach to the current user identity (anonymous/guest/registered) and follow identity conversion rules.

## Cart & orderlines
- `cart.orderlines` are the atomic commerce units.
- Orderlines may reference product, chosen variants/options, and (sometimes) a project.

## Projects
- Design flows create/attach projects.
- Cart edit routing (Shopper) can send user to project-edit page or editor based on `orderline.product.custom.btn_add_to_cart`.

## Photo prints
- Special UI builds selections; cart receives orderlines by size group.
- Quantity derived via `cut_print_quantity`.
