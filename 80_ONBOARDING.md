# 80 — Onboarding & Email Notifications

**Authority Scope:** Onboarding process, phases, and email notification templates only.

_Last updated: 2026-04-23_

---

## Onboarding Overview

Setting up Pixfizz involves a structured onboarding process managed through myPixfizz. A typical onboarding project contains 25–40 tasks across 5–6 phases.

### Pre-Onboarding (Pixfizz internal)
Before the kickoff call, Pixfizz handles:
- Organization created in myPixfizz (contacts, billing, subscription)
- Pixfizz environment provisioned
- Asset storage and SFTP access configured (if needed)
- Base product templates prepared
- Production pipeline validated end-to-end

---

## Full Pixfizz — Onboarding Phases

### Phase 1: Store Foundations
Select theme, configure branding, upload logo, set up domain and SSL, build site navigation and homepage.

### Phase 2: Product Setup
Define product categories, create product and personalization templates, configure variants, set pricing rules.

### Phase 3: Checkout Configuration
Connect payment gateway, configure shipping rules, tax handling, order confirmation/notification emails.

### Phase 4: Production Setup
Verify artwork generation, test production file output, confirm OrderHub job creation, validate production downloads.

### Phase 5: Launch Readiness
Place internal test orders, validate full order lifecycle, confirm payment capture and production routing, soft launch.

---

## Shopify + Pixfizz — Onboarding Phases

### Phase 1: Shopify Store Access
Grant Pixfizz admin access, confirm plan and theme compatibility.

### Phase 2: Pixfizz App Installation
Install Pixfizz Shopify app, generate API credentials, connect systems, configure webhooks.

### Phase 3: Product Synchronization
Map Shopify products to Pixfizz templates, configure personalization, map variants, validate pricing.

### Phase 4: Checkout Flow Validation
Validate cart integration, confirm personalization data transfer, verify orders flow from Shopify to Pixfizz.

### Phase 5: Production Verification
Place test orders through Shopify, confirm artwork generation, OrderHub job creation, production routing.

### Phase 6: Launch
Validate full personalization and checkout UX, confirm production output, enable products for public purchase.

---

## What Customers Need for Onboarding

- Brand assets (logo, colors, fonts)
- Product catalog details (categories, variants, pricing)
- Sample product images for testing personalization
- Payment gateway credentials
- Shipping rate structure
- Shopify admin access (if Shopify + Pixfizz)

---

## Email Notification Templates

Pixfizz Core includes 14 email templates mapping to the order lifecycle. Configured in admin: **Settings > Email Notifications**.

### Order Lifecycle Emails
- Order Pending
- Order Draft
- Order Confirmed
- Order Downloaded
- Order Manufactured
- Order Shipped
- Order Fulfilled
- Order Error
- Order Canceled
- Orderline Fulfilled

### Other Emails
- Cart Abandoned
- User Signup
- Password Reset
- Sign-in Token Renewed

Each template can be enabled or disabled individually. Default email address and HTML formatting are configured in Email Notifications settings.

---

## Changelog
- 2026-03-30: Created from master platform documentation export.

---

## Content Completeness Before Launch

Before going live, verify that all products and pages have descriptions populated. Missing descriptions are a common missed step during setup.

Missing descriptions cause:
- **Sitemap quality degradation** — blank description entries reduce the usefulness of the XML sitemap submitted to search engines
- **Product feed issues** — Google Shopping and Meta catalogue feeds require descriptions; missing ones cause feed errors or rejected products
- **Google Search Console warnings** — structured data validation flags missing or empty description fields

Add a description check to the final pre-launch validation step for both Full Pixfizz and Shopify + Pixfizz setups.

## Changelog
- 2026-03-30: Created from master platform documentation export.
- 2026-04-23: Added content completeness (descriptions) pre-launch checklist item.
