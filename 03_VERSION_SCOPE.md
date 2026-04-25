# 03 — Version Scope Convention

**Authority Scope:** Defines how Shopper version references work across all knowledge files, skills, and memory. This file does not introduce platform or template rules — it governs documentation clarity only.

_Last updated: 2026-04-07_

---

## The two Shopper versions

| | Shopper v1 | Shopper v2 |
|---|---|---|
| **Bootstrap** | 4.6 | 5 |
| **Status** | Production (current live sites) | In development (new template) |
| **Parent template** | shopper24.pixfizz.com | New v2 parent (BS5-based) |
| **Admin** | /site/setup/* — 150+ flat toggles (never completed) | 4-page admin: Brand & Theme, Storefront, SEO, Integrations |
| **Theming** | custom.css + checklist keys | brand.md config files |
| **New sites** | Existing sites remain on v1 | New sites will launch on v2 |

---

## Default version rule

**Unlabelled "Shopper" = Shopper v1.**

Any reference to "Shopper", "Shopper template", or "Shopper site" without an explicit version qualifier describes **Shopper v1** (Bootstrap 4.6, current production template).

This applies across all knowledge files, skill files, and memory files.

---

## How to label version-specific content

### In knowledge files (10–90 series)

- **v1-only content:** Can remain unlabelled (default), but adding `(v1)` is preferred for clarity when v2 content exists in the same file.
- **v2-only content:** MUST be labelled. Use one of:
  - Inline: `Shopper v2`, `(v2 only)`, `(BS5)`
  - Section header: `### Checkout flow (v2)`
  - Authority scope line: `**Authority Scope:** Shopper v2 only.`
- **Content that applies to both:** Label as `(v1 + v2)` or `(all versions)`.
- **Content that differs by version:** Use a comparison table or separate sections with clear headers.

### In skill files

- Skills that apply to v1 only: state `Shopper v1` or just `Shopper` (default).
- Skills that apply to v2 only: MUST state `Shopper v2` in the description and body.
- Skills that apply to both: state `Shopper (v1 + v2)` in the description.

### In memory files

- Memory entries about v2 work already use `v2` in their filenames (e.g., `project_shopper_v2_bs5.md`). Continue this convention.
- Generic Shopper memories without a version qualifier refer to v1.

---

## Key differences to watch for

These are the areas where v1 and v2 diverge enough that version confusion causes real bugs or wrong advice:

1. **CSS framework** — v1 uses Bootstrap 4.6 classes; v2 uses Bootstrap 5. Class names, grid system, utility classes, and JS plugins differ.
2. **Admin configuration** — v1 uses checklist keys and /site/setup/* toggles; v2 uses brand.md files and a 4-page admin.
3. **Theming** — v1 uses custom.css snippets and style tokens; v2 uses brand.md-driven theming.
4. **Navigation** — v1 nav styles (style1–style4, megamenu) may differ structurally in v2.
5. **Custom fields** — `51_CUSTOM_FIELDS_REFERENCE.md` is v2-derived. Field names may differ from v1.
6. **Snippet namespaces** — v2 may use different snippet names or structures.
7. **JS patterns** — v2 drops jQuery dependency patterns from BS4.6.

---

## Current audit status (2026-04-07)

### Knowledge files (10–90 series)

| File | Current state | Action needed |
|---|---|---|
| 51_CUSTOM_FIELDS_REFERENCE.md | Explicitly v2 | None |
| 50_SHOPPER_TEMPLATE_REFERENCE.md | Unlabelled (v1) | Add authority scope note: v1 |
| 20_SHOPPER_CART_RULES.md | Unlabelled (v1) | Add authority scope note: v1 |
| 21_SHOPPER_CHECKOUT_POLICY.md | Unlabelled (v1) | Add authority scope note: v1 |
| 22_OPTION_VARIANT_RENDERING.md | Unlabelled (v1) | Add authority scope note: v1 |
| 40_PLAYBOOK_UPDATED.md | Unlabelled (v1) | Add authority scope note: v1 |
| 90_FAQ.md | Unlabelled (mixed?) | Review: some answers may apply to both |
| 00_PROJECT_DIRECTIVE.md | Unlabelled (v1) | Add version convention reference |
| 02_RETRIEVAL_MAP.md | Unlabelled | Add version-routing guidance |
| 10_CORE_MENTAL_MODEL.md | Unlabelled (v1) | Minor — platform-level, version-neutral concepts |
| 13_TEMPLATE_BOUNDARIES.md | Unlabelled (v1) | Minor — responsibility matrix applies to both |
| 14_GLOSSARY.md | Unlabelled | Add v1/v2 definitions |
| 15_PLATFORM_ARCHITECTURE.md | Unlabelled | Minor — platform-level |
| 18_ADMIN_NAVIGATION.md | Unlabelled (v1) | Add note: v1 admin layout |
| 61_PIXFIZZ_API.md | Unlabelled | Minor — API layer, version-neutral |
| 12_OBJECTS_AND_OWNERSHIP.md | Unlabelled | Minor — platform-level |
| 31_FULFILLMENT_ENGINE.md | Unlabelled | Minor — platform-level |
| 19_XML_TEMPLATE_REFERENCE.md | Explicitly platform-level | None |
| 23_XML_CALENDAR_REFERENCE.md | Explicitly platform-level | None |

### Skill files

| File | Current state | Action needed |
|---|---|---|
| pixfizz-liquid-patterns/SKILL.md | Unlabelled | Add version scope note (v1 default, flag v2 differences) |
| pixfizz-shopper-ux/SKILL.md | Unlabelled | Add version scope note |
| pixfizz-onboarding-playbook/SKILL.md | Unlabelled | Add version scope note (onboarding may differ for v2) |

### Memory files

| File | Current state | Action needed |
|---|---|---|
| project_shopper_v2_*.md (5 files) | Explicitly v2 | None |
| project_skip_cart_redirect.md | Unlabelled | Minor — feature exists in v1, likely also v2 |
| reference_pixfizz_knowledge_files.md | Mixed | Already updated with v2 custom fields ref |

---

## Retrieval guidance

When answering a Shopper question:

1. **Determine the version context first.** If the user or site is on v2, prioritise v2-labelled files. If unclear, ask.
2. **Check this file (03_VERSION_SCOPE.md)** for which files are v1 vs v2.
3. **Do not mix v1 and v2 guidance** in a single answer without clearly labelling which is which.
4. **When in doubt, state the version** of the advice you are giving.

---

## Changelog

- 2026-04-07: Created. Initial audit of all knowledge files, skills, and memory.
