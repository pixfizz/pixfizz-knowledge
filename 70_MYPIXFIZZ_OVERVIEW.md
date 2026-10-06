# 70 — MyPixfizz Overview

**Authority Scope:** System identity, architecture, portals, tech stack, and integrations for my.pixfizz.com.

_Last updated: 2026-10-06_

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
- **Auth:** Supabase Auth. The sign-in screen offers Password, Email code and Magic link (Corrected 2026-10-06).
- **Key external integrations:** Fireflies, QuickBooks, GA4, Pixfizz Order Webhook, Pixfizz admin API (per brand), Klaviyo (read only), Slack (urgent support alerts), Calendly

---

## Portal Structure

### Admin Portal
Main routes: `/admin/support` (Support Overview, the staff landing page), `/admin/support-inbox`, `/executive`, `/assistant`, `/organizations`, `/contacts`, `/brands`, `/performance`, `/admin/onboarding`, `/projects`, `/call-log`, `/pipeline`, `/lead-inbox`, `/targets`, `/product/ideas`, `/product/roadmap`, `/product/intelligence`, `/admin/marketing`, `/admin/ga4-events`, `/admin/events`, `/announcements`, `/admin/videos`, `/admin/catalog-manager`, `/admin/pod-catalog`, `/admin/tools`, `/admin/reviews`, `/billing`, `/invoices`, `/suppliers`, `/private/costs`, `/users`, `/admin/portal-access`, `/admin/settings/support-contacts`, `/infrastructure`. On a phone, staff use the mobile support app at `/m/support`. (Corrected 2026-10-06: the old list named `/dashboard`, `/leads`, `/tasks`, `/onboarding`, `/ideas`, `/roadmap` and `/product-intelligence` as admin routes. Full map in `71_MYPIXFIZZ_FEATURES_ROUTES.md`.)

Full access for Pixfizz staff. No org-scoping — sees all data.

### Customer Portal
Routes sit at the root, not under `/portal` (Corrected 2026-10-06): `/dashboard`, `/marketing`, `/events`, `/support`, `/call-log-portal`, `/whats-new`, `/videos`, `/roadmap`, `/brand`, `/assets`, `/tools` and the tool pages under `/tools/*`. Before kickoff a customer lands on `/welcome`.

Scoped strictly to the user's own organization. Customers cannot see other orgs' data. RLS enforced at DB level.

### Admin Portal Preview
Route: `/admin/portal-preview/:orgId`

Lets a Pixfizz admin impersonate a customer's portal view — see exactly what that customer sees. Used for support and onboarding. Every customer route is mirrored under `/admin/portal-preview/:orgId/...`.

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

A brand is one **storefront**; the app uses both words for the same record. Each has a storefront type (Shopper, Shopify + Pixfizz, Custom setup or Custom CMS) that decides which brand page tabs it gets; once set, only staff can change it. Brands are archived, never hard-deleted, and a duplicate is merged into the brand kept through `/brands/merge`. *Verified by reading source (Lovable, commit d6290ee), 2026-10-06.*

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
| **Google Reviews** | Customer-facing connector on the brand page's Connections tab. Uses the brand's Pixfizz admin API credential (`brand_api_credentials`). Reviews sync from Google into a review inbox and publish to the storefront as instances of a Shopper custom type, which the storefront widgets read. See `71_MYPIXFIZZ_FEATURES_ROUTES.md` § Brand Connections and Catalog. |
| **Pixfizz admin API** | Per-brand credential (§ Credential Storage). Used by Catalog Manager, Shopper Configure, Shopify Style, Blog, Merge users, Static Product Uploader, Store health check, Promo codes, Customer export and Shopper Upgrades. Every write is read back before it counts as done. |
| **Klaviyo** | Per-brand private key for Marketing, kept in the vault (`brand_klaviyo_credentials`), set and cleared only through a backend function, never sent to the browser. Read only in the current build (scheduled emails, flows, lists). *Verified by reading source, 2026-10-06.* |
| **Slack** | Urgent (blocking) support cases post an alert that mentions staff with a Slack user id (`71_MYPIXFIZZ_FEATURES_ROUTES.md` § Support). |
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
**API keys (2026-09-28).** The brand credential is either a Pixfizz admin API key (`pxk_...`, sent
as the Basic-auth username with an empty password) or a legacy `username:password`, one per brand.
Every Pixfizz caller builds its Authorization header through one shared helper
(`_shared/pixfizz-auth.ts`), and only the first 12 characters of a key are ever stored outside
the vault or displayed. The card is **Pixfizz API access** on the brand's Connections tab, with
**Replace key**, **Test connection** and **Clear**; a brand still on a password shows an amber
"Switch to an API key" pill. New connections use a key created on a dedicated API user in that
site's admin, one key per integration (`61_PIXFIZZ_API.md` § 2). The Admin URL field converts a
pasted `admin.pixfizz.com/site/<slug>/admin` link to the API base `https://<slug>.pixfizz.com` on
Save. *Verified by reading source, and an end-to-end key test by query, 2026-09-28.*

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

### More recurring defect patterns (2026-09-23)

- **An empty state that doubles as the error state.** A failed query and a genuinely empty list render the same UI, so a broken query looks like "no records yet" for days. Every list whose query can fail needs a distinct error branch.
- **An edge function fix is not live until it is published.** Functions ship on publish, not on commit. The stored shape of the data a function writes is evidence of which version ran.
  Unconfirmed (2026-10-06): two later builds report the opposite for this project, that edge functions, migrations and pg_cron jobs take effect on build and only the front end waits for Publish (§ Operational Rules for Changing myPixfizz). Until this is settled, check the deployed function's behavior or stored output before assuming either way.
- **An email that invites a reply, sent from an address that cannot receive one.** A `noreply@` sender with no `Reply-To` turns "reply to this email" into a silent bounce.

*Verified by reading source and by query, 2026-09-23.*

### Operational rules from two outages and a stuck home-screen app (2026-09-29)

- **Any every-minute pg_cron job needs a retention job for `cron.job_run_details` from day one.** Unbounded job history grew past 450 MB and starved the database twice (15 and 29 September): even `select 1` was cancelled. The fix is a nightly batched delete of history older than a few days, plus a vacuum. *Verified by query and logs, 2026-09-29.*
- **A prevention step written in an incident doc is not done until a query shows it exists.** The 15 September prune job was prescribed and never created, which is why the same outage happened again.
- **Never clear `ga4_event_outbox` to free space.** It is the order history behind Brand Performance and the brand comparisons; copy it elsewhere first.
- **Never remove a shipped service worker by deleting its file.** Replace it at the same path with a self-destroying worker (skipWaiting on install; on activate delete every cache, unregister, reload open windows; no fetch handler) and keep that file permanently. A deleted worker path falls through to the SPA fallback and the old worker stays installed, so a home-screen install keeps running old code against live data. The cleanup code in the new app never runs on a device still controlled by the old worker. *Verified by reading source (git history), 2026-09-29; the fix on iOS is not yet verified.*

### More recurring defect patterns (2026-10-06)

- **A customer page that resolves the organization from `organization_members` by user id, or links to a bare customer path, breaks portal preview.** In preview the signed-in user is staff, so the page loads the staff member's own organization (an order code then "matches none of your sites"), and a bare link such as `/support` drops staff out of the preview into their own portal, which looks like a logout. Every customer-facing page takes its organization from `useCustomerOrg()` and builds every internal link with `usePortalBasePath()`. *Verified by reading source and a fix tested in preview, 2026-09-30.*

---

## Operational Rules for Changing myPixfizz

Lovable Cloud behavior that decides how a change reaches the live site. *Verified by query and by reading source, 2026-09-29 to 2026-10-01, unless marked.*

- **The preview and my.pixfizz.com share one database.** Migrations apply to production the moment Lovable builds them. Front-end code reaches my.pixfizz.com only on Publish. On 29 Sep the database half of a stability build (indexes, grants, timeouts, realtime changes) was live hours before its front end was published.
- **Every migration must work with the front end that is currently published.** To tighten a constraint or rename stored values, first ship and publish code that writes the new values, then tighten the database in a later change. On 30 Sep a tightened `brands_storefront_type_check` made brand creation and every store type change fail on the live site with a raw check-constraint error while the preview worked, because the published code still sent the old labels.
- **Until a build is published, test user-facing changes in the Lovable preview,** not on my.pixfizz.com.
- **Publishing ships every unpublished build in the queue at once.** A small fix that lands on top of an unfinished batch cannot go live on its own; check what else is queued before promising when a fix will be live.
- **Database jobs and what they send are live on build.** pg_cron jobs, triggers and the support notification outbox ran on production before the matching front end was published (the awaiting-customer reminders first went out on 1 Oct, the morning they were built). Test such changes in a rolled-back transaction, or with throwaway rows deleted before the per-minute sender runs.
- **A fix made directly in the shared database needs a matching migration file,** or rebuilding from the repo brings the old version back. Example: the Klaviyo key check was relaxed directly in the database on 30 Sep while the repo migration still holds the old pattern. *Verified by reading source (commit d6290ee).*
- **Stability baseline (29 Sep).** `nightly_maintenance()` runs at 01:17 UTC: it keeps 7 days of `cron.job_run_details` and 60 days of `infrastructure_checks`, deletes in batches of 2,000, stops itself on a batch slower than 5 s, and writes each run to `maintenance_log`; a vacuum follows at 02:47 UTC (verified by reading source). `service_role` has a 30 s statement timeout, with longer limits set only inside the few long jobs; realtime is on `notifications` only; signed-out EXECUTE was revoked from the SECURITY DEFINER functions except `has_role`; and a test fails the build on any query refetch interval under 30 s (verified by query and from the build report, 29 Sep).
- **Default privileges now revoke EXECUTE from PUBLIC.** Any new function needs an explicit GRANT to the roles that call it, and an RPC that signed-out visitors call needs `GRANT EXECUTE ... TO anon`.
- **Anything new that polls, adds realtime, or adds a table that grows every day states its retention and interval** in the request. The pg_cron history rule above is the case that caused two outages.

---

## Organization Add-ons

Add-ons are recorded per organization in `organization_addons`. The accepted `addon_key` values are `orderhub`, `point_of_sale`, `pixfizz_conversations` and `s3_storage`; members of an organization can read their own add-ons. *Verified by reading source (migration of 2026-09-30).* Pixfizz Conversations is the website chat; how it works is in `45_ORDERHUB.md` § Website Chat (Twilio Conversations). Prices and packaging are not covered in this knowledge base.

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
- 2026-09-24: Added three recurring defect patterns: empty state as error state, unpublished edge functions, reply-inviting mail with no Reply-To. Source: claude-chat.
- 2026-09-29: Credential Storage: API keys supported (pxk_ or legacy user:pass, shared auth helper, 12-char hint only), UI labels, Admin URL normalization. Added Google Reviews to the Integrations Summary. Operational rules: pg_cron history retention, prevention steps verified by query, never clear ga4_event_outbox, service worker kill switch. Source: claude-chat.
- 2026-10-06: Corrected Portal Structure to the current routes (no `/portal` prefix; staff land on Support Overview; mobile support app) and the sign-in methods. Brand concept: storefront types, archive not delete, merge. Integrations: Pixfizz admin API, Klaviyo (read only, key in vault), Slack urgent alerts. Added a defect pattern (customer pages must use `useCustomerOrg()` and `usePortalBasePath()` or portal preview breaks), the Operational Rules for Changing myPixfizz section (shared database, migrations live on build, publish ships the whole queue, database jobs live on build, direct database fixes need a migration file, stability baseline, default privileges), and Organization Add-ons (keys only, cross-reference to 45). Flagged the edge-functions-ship-on-publish line as conflicting with later builds. Source: claude-chat.
