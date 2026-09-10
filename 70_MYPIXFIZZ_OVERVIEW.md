# 70 — MyPixfizz Overview

**Authority Scope:** System identity, architecture, portals, tech stack, and integrations for my.pixfizz.com.

_Last updated: 2026-09-09_

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
| **GA4** | Server-side `purchase` via the GA4 Measurement Protocol, live since March 2026. Per-brand measurement id, API secret, enabled flag, debug flag and billing currency. The Pixfizz order webhook feeds an event outbox, which sends to the Measurement Protocol with per-order idempotency and attempt logging. Depends on the Shopper order custom fields `ga_client_id` and `ga_session_id` being present on the storefront. See `85_GA4_SERVER_SIDE_PURCHASE.md`. |
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

## Credential Storage — One Store for Pixfizz Admin Access

Design rule established 2026-08-25.

**`brand_api_credentials` is the single store for Pixfizz admin access.** A new
connector asks the customer for a login **only** when
`brand_api_credentials.credential_vault_id` is null, and when it does ask it writes
at brand level via `customer_set_brand_api_credentials`.

Per-connector credentials are an override, never the default path. Anything that
prompts for a Pixfizz login it could have inherited is a bug in the connector.

### What is held, and where

*Verified by reading source, 2026-08-25.* The credential's **secret lives in a vault**, not in a
readable column, and is **write-only from the client** — it is set and rotated from the UI and
never returned to it. The base URL is hardened to a Pixfizz host, so a connector cannot be
pointed at an arbitrary destination.

The customer-facing controls on the brand's Connectors section are set, masked, rotate, test and
clear. The same card appears in the admin brand dialog.

---

## Recurring Defect Patterns

Four failure shapes that have each cost more than one debugging session in this codebase. They
are worth checking for by name whenever a customer-facing feature "works for us and not for
them". *Each verified by reading source, and each was a live defect.*

- **An RLS SELECT policy that re-reads its own table.** An insert then cannot see its own returned
  row, so any insert-and-return path fails for the customer while succeeding for staff. This kept
  a customer wizard broken for over a month.
- **An edge function gated on an admin-role check.** Every customer action against it returns a
  flat 401. The feature works perfectly in every internal test, because every internal tester
  holds the role.
- **A 200 carrying an `ok: false` body.** The transport succeeded, the operation did not, and the
  caller reports success. Check the body, not the status code — and on the writing side, do not
  ship an error path that returns 200 without the caller being written to read the flag.
- **A "test connection" probe that hits a different route family than the feature.** It returns
  Verified while the feature is broken, which is worse than having no test at all. A connectivity
  test must exercise the same route family the feature uses.

---

## RLS and Aggregates in Triggers — the Rule

*Verified by reading source and by test, 2026-09-09. This is the highest-value finding of the
week and it generalizes well past the case that produced it.*

**A PL/pgSQL trigger function without `SECURITY DEFINER` runs as the calling user, so every query
inside it — including a `SELECT MAX()` over its own table — is filtered by that user's RLS
policies.**

It follows that **any "next number" derived from an aggregate inside an invoker-rights trigger is
computed from the subset of rows the caller can see**, not from the table. It is correct only for
users whose SELECT policy is unscoped. That is exactly why this class of bug is **invisible to
staff and reproduces only for customers**: the staff policy is unscoped, so the aggregate is
right every time an internal tester runs it.

The presenting symptom was a **unique-constraint violation on submit** — a customer creating a
record from a portal form, where the scoped aggregate produced a number another organization
already held.

**The fix is a sequence, not merely adding `SECURITY DEFINER`.** A sequence is also immune to
deletes, and it drops a full-table scan and an advisory lock from every insert. Adding
`SECURITY DEFINER` alone fixes the visibility problem and leaves the rest.

**Where else to look.** Audit any trigger or RPC that derives a value from an aggregate over an
RLS-protected table and feeds it into something with a uniqueness or ordering guarantee. The tell
is an aggregate selected into a variable inside a function that is not `SECURITY DEFINER`.

### The testing rule this establishes

**A staff account cannot test anything gated by RLS.** Admin policies pass every scoped policy,
and every scoped query run as an admin silently returns the full table — no error, no warning,
just a different answer. **Any check of a customer-facing write path has to run as a real
customer account, or it proves nothing.** This is the second time an admin-only test has hidden a
customer-only failure.

---

## Webhook Endpoint Design Rules

*Verified by reading source and by live test, 2026-09-09.*

**Status codes.** A bad or missing key returns **401**. Everything else returns **200 on
purpose**, including errors. The reason is deliberate: a sender that sees a 5xx retries, so a
parse bug on one message turns into a retry loop against every message. Return 200, record the
failure on your own side, and surface it in an operator view rather than to the sender.

**A secret in the query string makes the URL a credential.** Where a provider cannot send an
Authorization header, the shared secret rides in the query string, and then **the endpoint URL
*is* a bearer credential**. It must not be pasted into tickets, chat or documentation. Rotating
it means changing the secret and the provider's entry **together, in that order** — secret first,
then the provider, or every message in between is rejected.

---

## Copying Mail to an Ingestion Endpoint — Use a Transport Rule

*Verified by reading source, 2026-09-09.*

To copy inbound mail into an ingestion endpoint, use a **mail transport rule**, not mailbox
forwarding.

- The tenant's **outbound spam policy governs automatic forwarding** to external domains. An
  ingestion subdomain is not an accepted domain, so mailbox forwarding to it **can be silently
  dropped** — no bounce, no error, just nothing arriving.
- **Transport rules are not subject to that control.**
- A **Bcc leaves normal delivery intact**, so the original mailbox keeps receiving everything and
  the change is additive rather than a cutover.

**Rollback is disabling one rule.** Design the launch so that is true.

---

## Support Intake

**Support intake moved off the previous helpdesk on 2026-09-09.** *Verified live.* Inbound
support email now routes into the myPixfizz support system, where it lands as a case.

The old helpdesk **stays accessible for history and open tickets but receives nothing new**.

**Documentation impact:** this invalidates any knowledge base entry or help article that tells
customers to email the old helpdesk address. Anything giving customers a support address needs
re-checking against the current route.

---

## Changelog
- 2026-03-26: Initial version. Compiled from Lovable project summary.
- 2026-08-29: Added Credential Storage — `brand_api_credentials` is the single store for Pixfizz admin access; a connector prompts for a login only when `credential_vault_id` is null and writes at brand level via `customer_set_brand_api_credentials`, with per-connector credentials as an override rather than the default. Source: claude-chat.
- 2026-09-09: REPLACED the one-line GA4 row in the Integrations table with the full description of the server-side `purchase` pipeline (Measurement Protocol, live since March 2026; per-brand measurement id, API secret, enabled and debug flags, billing currency; order webhook into an event outbox with per-order idempotency and attempt logging; dependent on the Shopper order custom fields `ga_client_id` and `ga_session_id`), pointing at `85_GA4_SERVER_SIDE_PURCHASE.md`. Extended Credential Storage with what is held and where — secret in a vault, write-only from the client, base URL hardened to a Pixfizz host. Added Recurring Defect Patterns (self-referential RLS SELECT policy; edge function gated on an admin-role check; a 200 carrying `ok: false`; a test-connection probe on a different route family). Added the RLS-and-aggregates rule — a PL/pgSQL trigger without `SECURITY DEFINER` runs as the caller, so an aggregate inside it is RLS-filtered, which is why the bug is invisible to staff and reproduces only for customers; a sequence is the right fix; and the testing rule that a staff account cannot test anything gated by RLS. Added webhook endpoint design rules (401 for a bad key, 200 for everything else on purpose, and a query-string secret making the URL a credential that rotates with the provider entry, in that order). Added the mail transport rule versus mailbox forwarding for copying mail to an ingestion endpoint. Added Support Intake — intake moved off the previous helpdesk on 2026-09-09, which invalidates any article giving customers the old address. Source: claude-chat, fireflies-call.
