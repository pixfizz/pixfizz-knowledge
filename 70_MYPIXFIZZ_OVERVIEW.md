# 70 — MyPixfizz Overview

**Authority Scope:** System identity, architecture, portals, tech stack, and integrations for my.pixfizz.com.

_Last updated: 2026-03-26_

---

## What MyPixfizz Is

MyPixfizz (`my.pixfizz.com`) is the internal ERP/CRM and customer self-service portal for Pixfizz. It is a separate product from the Pixfizz personalization platform — it does not handle photo product creation, ordering, or fulfillment. Its role is operational management.

Two audiences:
- **Pixfizz staff** — sales, ops, leadership. Full operational control: pipeline, CRM, tasks, finance, support, onboarding, roadmap.
- **Pixfizz customers** — photo labs and print businesses that subscribe to Pixfizz. Self-service access to their account, projects, support, and roadmap.

---

## Tech Stack

- **Frontend:** React + Vite + Tailwind CSS
- **Backend:** Lovable Cloud (Supabase under the hood — Postgres + Edge Functions + RLS)
- **Design style:** Dark-mode-first, Apple-inspired minimal SaaS UX
- **Auth:** OTP and password-based sign-in via Supabase Auth
- **Key external integrations:** Fireflies, QuickBooks, GA4, Pixfizz Order Webhook, Calendly

---

## Portal Structure

### Admin Portal
Routes: `/dashboard`, `/pipeline`, `/leads`, `/organizations`, `/contacts`, `/brands`, `/projects`, `/tasks`, `/onboarding`, `/invoices`, `/costs`, `/suppliers`, `/ideas`, `/roadmap`, `/product-intelligence`, `/call-log`, `/executive`, `/infrastructure`, `/admin/support`, `/admin/events`, `/admin/portal-preview/:orgId`

Full access for Pixfizz staff. No org-scoping — sees all data.

### Customer Portal
Routes: `/portal/*`

Scoped strictly to the user's own organization. Customers cannot see other orgs' data. RLS enforced at DB level.

### Admin Portal Preview
Route: `/admin/portal-preview/:orgId`

Lets a Pixfizz admin impersonate a customer's portal view — see exactly what that customer sees. Used for support and onboarding.

---

## Role & Access Model

- Roles stored in `user_roles` table
- Two primary roles: `admin` (Pixfizz staff), `customer` (subscriber org users)
- Protected routes with role checking at the router level
- Organization-scoped data isolation via RLS policies
- All customer portal queries are filtered to the user's `org_id`

---

## Key Concepts

### Organization
The master record for a Pixfizz customer (a photo lab or print business). Has a lifecycle status: Lead → Onboarding → Active → On Hold. All customer data (brands, projects, contacts, tasks, invoices, support cases) belongs to an organization.

### Brand
A brand within an organization. An org can have multiple brands. Each brand has its own logo, brand guide assets, GA4 config, and billing currency. Revenue data rolls up to brand level.

### Project
An internal Pixfizz project record for work being done for an org — e.g. a site build, integration, or feature rollout. Has notes, meetings, links, Loom videos, and a client-visibility toggle per item.

### Contact
A person at a customer organization. Bidirectional sync with platform `user` records. Contacts can be converted to users and vice versa.

### Task
The core execution unit. Tasks can belong to an org, brand, project, or be internal. Assignable to admin users, customer users, or contacts. Supports 27 standardized categories, recurring patterns, and a daily focus/priority system.

---

## Integrations Summary

| Integration | Purpose |
|---|---|
| **Fireflies** | Sync meeting transcripts, AI summaries, action item extraction, auto-match to org/project. Webhook ingestion. AI idea candidate creation (with mandatory human review before publishing). |
| **QuickBooks** | Invoice and payment sync. Token refresh. Webhooks for payment events. |
| **GA4** | Per-brand measurement IDs. Event outbox with retry logic. UTM attribution. Pixfizz order webhook feeds GA4 Measurement Protocol. |
| **Pixfizz Order Webhook** | Receives orders from the Pixfizz platform, processes for GA4 and reporting. |
| **Calendly** | Webhook ingestion of meeting events into the call log / CRM. |
| **Email (Edge Functions)** | Transactional emails: forgot password, onboarding invites, support agent notifications, idea promotion alerts. |

---

## Infrastructure Notes

- Edge Functions handle: SLA checking, GA4 event dispatch, QuickBooks token refresh, Fireflies sync, email sending, webhook processing.
- Infrastructure health check dashboard at `/infrastructure` — automated service monitoring with status strip visible in both portals.
- GA4 Event Log available for debugging the analytics pipeline.

---

## Relationship to Pixfizz Platform

MyPixfizz is **operationally connected** to the Pixfizz platform but is a separate codebase:
- Pixfizz orders flow into MyPixfizz via webhook for GA4 tracking and revenue reporting.
- Customer organizations in MyPixfizz correspond to Pixfizz subscriber accounts.
- MyPixfizz does not control the Pixfizz personalization engine, storefront, or fulfillment — those are handled by the Pixfizz CMS platform (see files 10–60).

---

## Changelog
- 2026-03-26: Initial version. Compiled from Lovable project summary.
