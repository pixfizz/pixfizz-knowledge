# 72 — MyPixfizz Data Model

**Authority Scope:** Supabase database schema for my.pixfizz.com.

_Last updated: 2026-03-26_

---

## Enums

| Enum | Values |
|---|---|
| `app_role` | `admin`, `customer` |
| `invoice_line_category` | `Subscription`, `TransactionFees`, `ProServices`, `Discount`, `Other` |

---

## Views

| View | Purpose |
|---|---|
| `organizations_safe` | Restricted view of `organizations` — non-sensitive columns only (id, name, status, type, country, timezone, onboarding_status, customer_since, created_at, updated_at). Used for customer-facing queries. |

---

## Core Entity Map

Everything in MyPixfizz ultimately connects to `organizations`. Key relationships:

- `organizations` → `brands` (one-to-many)
- `organizations` → `contacts` (one-to-many)
- `organizations` → `projects` (one-to-many)
- `organizations` → `opportunities` (one-to-many)
- `organizations` → `tasks` (many — via organization_id)
- `organizations` → `support_cases` (one-to-many)
- `organizations` → `financial_invoices` (one-to-many)
- `organizations` → `onboarding_projects` (one-to-many)
- `organization_members` — maps auth.users to organizations
- `user_roles` — maps auth.users to role (admin / customer)

---

## Table Reference

### announcements
Stores system-wide or org-specific announcements shown in the customer portal.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| body | text | |
| category | text | default value |
| priority | text | default value |
| system_scope | text | |
| is_published | boolean, nullable | |
| published_at | timestamptz, nullable | |
| requires_acknowledgement | boolean | |
| image_url | text, nullable | |
| video_url | text, nullable | |
| link_url | text, nullable | |
| link_label | text, nullable | |
| organization_id | uuid, nullable | FK → organizations (null = system-wide) |
| created_by | uuid, nullable | |
| created_at | timestamptz, nullable | |

---

### billing_invoices
Monthly billing invoices per brand, linked to billing runs.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| billing_run_id | uuid | FK → billing_runs |
| brand_id | uuid | FK → brands |
| organization_id | uuid | FK → organizations |
| currency | text | |
| gross_revenue | numeric | |
| gross_revenue_currency | text, nullable | |
| transaction_fees | numeric | |
| subscription_fees | numeric | |
| sms_fees | numeric | |
| perfectly_clear_fees | numeric | |
| promo_credits | numeric | |
| loyalty_discount_pct | numeric | |
| loyalty_discount_amount | numeric | |
| net_before_vat | numeric | |
| vat_rate | numeric | |
| vat_amount | numeric | |
| total_due | numeric | |
| total_gbp | numeric | Normalized to GBP for reporting |
| csv_filename | text, nullable | |
| csv_row_count | integer | |
| qb_invoice_id | text, nullable | QuickBooks invoice ID |
| qb_status | text | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### billing_line_items
Individual line items within a billing invoice.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| invoice_id | uuid | FK → billing_invoices |
| category | text | |
| row_count | integer | |
| subtotal | numeric | |
| created_at | timestamptz | |

---

### billing_payment_lines
Links payments to specific QuickBooks invoices with amounts applied.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| payment_id | uuid | FK → billing_payments |
| invoice_qb_id | text | |
| amount_applied | numeric | |
| created_at | timestamptz | |

---

### billing_payments
Payments received, synced from QuickBooks.

| Column | Type | Notes |
|---|---|---|
| id | text, PK | QuickBooks payment ID |
| customer_qbo_id | text | |
| customer_display_name | text, nullable | |
| total_amount | numeric | |
| currency | text | |
| txn_date | text, nullable | |
| payment_method | text, nullable | |
| deposit_to_account | text, nullable | |
| organization_id | uuid, nullable | FK → organizations |
| realm_id | text, nullable | |
| raw_json | jsonb, nullable | |
| last_synced_at | timestamptz | |
| created_at | timestamptz | |

---

### billing_run_fx_rates
FX rates used for a specific billing run.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| billing_run_id | uuid | FK → billing_runs |
| source_currency | text | |
| target_currency | text | |
| rate | numeric | |
| created_at | timestamptz | |

---

### billing_runs
Monthly billing run records with period and status.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| period_year | integer | |
| period_month | integer | |
| status | text | |
| fx_rate_usd_gbp | numeric | |
| fx_rate_eur_gbp | numeric | |
| notes | text, nullable | |
| finalized_at | timestamptz, nullable | |
| created_by | uuid | |
| created_at | timestamptz | |

---

### brands
Customer brands/storefronts, each linked to an organization.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| organization_id | uuid | FK → organizations |
| subdomain | text, nullable | Pixfizz subdomain |
| website_code | text, nullable | |
| logo_url | text, nullable | |
| brand_guide | jsonb, nullable | |
| storefront_type | text, nullable | |
| billing_currency | text, nullable | |
| ga4_enabled | boolean | |
| ga4_measurement_id | text, nullable | |
| ga4_api_secret | text, nullable | |
| ga4_send_debug | boolean | |
| webhook_secret | text, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### call_log
Fireflies-synced meeting transcripts and call records.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| fireflies_transcript_id | text, unique | |
| title | text, nullable | |
| summary | text, nullable | |
| overview | text, nullable | |
| shorthand_bullets | text, nullable | |
| detailed_notes | text, nullable | |
| action_items | jsonb, nullable | |
| attendees | jsonb, nullable | |
| keywords | text[], nullable | |
| meeting_date | timestamptz, nullable | |
| duration_minutes | integer, nullable | |
| transcript_url | text, nullable | |
| organization_id | uuid, nullable | FK → organizations |
| contact_id | uuid, nullable | FK → contacts |
| opportunity_id | uuid, nullable | FK → opportunities |
| project_id | uuid, nullable | FK → projects |
| match_status | text | Matching state to org/project |
| customer_title | text, nullable | |
| customer_notes | text, nullable | |
| synced_at | timestamptz | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### company_costs
Internal company operating costs and expenses.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| category | text | |
| vendor | text, nullable | |
| amount | numeric | |
| currency | text | |
| frequency | text | |
| start_date | date | |
| end_date | date, nullable | |
| billing_day | integer, nullable | |
| invoice_date | date, nullable | |
| invoice_ref | text, nullable | |
| is_paid | boolean | |
| notes | text, nullable | |
| receipt_url | text, nullable | |
| supplier_id | uuid, nullable | FK → suppliers |
| created_by | uuid | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### contacts
CRM contacts — people linked to organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| first_name | text | |
| last_name | text | |
| email | text | |
| phone | text, nullable | |
| job_title | text, nullable | |
| department | text, nullable | |
| organization_id | uuid | FK → organizations |
| is_primary | boolean | |
| status | text | |
| preferred_contact_method | text | |
| linkedin_url | text, nullable | |
| notes | text, nullable | |
| user_id | uuid, nullable | Implicit FK → auth.users (bidirectional sync) |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### daily_goals
Per-user daily planning goals with sub-goals.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| user_id | uuid | |
| goal_date | date | |
| goal_text | text, nullable | |
| sub_goals | jsonb | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### email_invoice_inbox
Inbound invoice emails parsed and linked to costs/suppliers.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| sender_email | text | |
| subject | text, nullable | |
| received_at | timestamptz | |
| status | text | pending/processing/completed/failed/duplicate/skipped |
| pdf_filename | text, nullable | |
| extracted_data | jsonb, nullable | |
| error_message | text, nullable | |
| idempotency_key | text, nullable | |
| linked_cost_id | uuid, nullable | FK → company_costs |
| linked_supplier_id | uuid, nullable | FK → suppliers |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### events
Public/internal events (webinars, workshops) shown to customers.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| description | text, nullable | |
| event_date | timestamptz | |
| event_type | text, nullable | |
| status | text | upcoming/live/completed |
| duration_minutes | integer, nullable | |
| zoom_link | text, nullable | |
| youtube_id | text, nullable | For past events |
| thumbnail_url | text, nullable | |
| ai_summary | text, nullable | |
| is_visible | boolean, nullable | |
| created_at | timestamptz, nullable | |
| updated_at | timestamptz, nullable | |

---

### feature_feedback
Customer feedback messages on specific roadmap features.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| feature_id | uuid | FK → features |
| user_id | uuid, nullable | |
| organization_id | uuid, nullable | |
| contact_email | text | |
| company_name | text, nullable | |
| subject | text | |
| body | text | |
| created_at | timestamptz | |

---

### feature_ideas
Join table linking features to ideas they originated from.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| feature_id | uuid | FK → features |
| idea_id | uuid | FK → ideas |
| created_at | timestamptz | |

---

### feature_opportunities
Links features to sales opportunities blocked by them. Feeds sales_blocker_bonus scoring.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| feature_id | uuid | FK → features |
| opportunity_id | uuid | FK → opportunities |
| estimated_subscription_mrr | numeric | |
| created_at | timestamptz | |

---

### feature_organizations
Links features to requesting/interested organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| feature_id | uuid | FK → features |
| organization_id | uuid | FK → organizations |
| created_at | timestamptz | |

---

### feature_reactions
Customer sentiment reactions on roadmap features.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| feature_id | uuid | FK → features |
| user_id | uuid | |
| organization_id | uuid, nullable | |
| reaction | text | positive/neutral/negative — validated by trigger |
| created_at | timestamptz | |

---

### features
Product roadmap features with priority scoring.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| description | text, nullable | |
| status | text | |
| target_window | text, nullable | |
| effort_days | numeric | |
| complexity_score | integer | |
| strategic_fit_score | numeric | |
| competitive_pressure_score | numeric | |
| weighted_vote_score | integer | Aggregated from idea_votes |
| revenue_impact_score | numeric | |
| sales_blocker_bonus | integer | From feature_opportunities |
| effort_penalty | numeric | |
| priority_score | numeric | Composite scoring — see 71 for formula |
| manual_priority_override | numeric, nullable | |
| public_visible | boolean | Controls customer portal visibility |
| owner_user_id | uuid, nullable | |
| shipped_at | timestamptz, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### financial_invoice_line_items
QuickBooks-synced invoice line items with categorization.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| invoice_id | uuid | FK → financial_invoices |
| description | text, nullable | |
| qty | numeric | |
| unit_price | numeric | |
| line_total | numeric | |
| category | enum: invoice_line_category | Subscription/TransactionFees/ProServices/Discount/Other |
| qb_line_id | text, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### financial_invoice_sync_log
Audit log for QuickBooks invoice sync events.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| event_type | text | |
| status | text | |
| qb_invoice_id | text, nullable | |
| organization_id | uuid, nullable | |
| details | jsonb, nullable | |
| created_at | timestamptz | |

---

### financial_invoices
QuickBooks-synced invoices with payment status.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| qb_invoice_id | text | |
| qb_customer_id | text | |
| invoice_number | text, nullable | |
| customer_display_name | text, nullable | |
| organization_id | uuid, nullable | FK → organizations |
| status | text | Draft/Sent/Open/Overdue/Paid/Void/Deleted — validated by trigger |
| invoice_date | date, nullable | |
| due_date | date, nullable | |
| currency | text | |
| subtotal_amount | numeric | |
| tax_amount | numeric | |
| discount_amount | numeric | |
| total_amount | numeric | |
| balance_due | numeric | |
| email_status | text, nullable | |
| pdf_url | text, nullable | |
| realm_id | text, nullable | |
| raw_json | jsonb, nullable | |
| qb_last_updated_at | timestamptz, nullable | |
| synced_at | timestamptz | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### financial_settings
Key-value store for financial configuration (e.g. tax rates).

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| key | text, unique | |
| value | numeric | |
| description | text, nullable | |
| updated_at | timestamptz | |

---

### fulfillment_map
Maps product categories to fulfillment methods per organization.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| product_category | text | |
| method | text | |
| equipment | text, nullable | |
| created_at | timestamptz | |

---

### fx_rate_history
Historical foreign exchange rate snapshots.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| source_currency | text | |
| target_currency | text | |
| rate | numeric | |
| fetched_at | timestamptz | |

---

### ga4_event_attempts
Individual GA4 Measurement Protocol send attempts.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| outbox_id | uuid | FK → ga4_event_outbox |
| organization_id | uuid | FK → organizations |
| endpoint | text | |
| http_status | integer, nullable | |
| response_body | text, nullable | |
| error | text, nullable | |
| attempted_at | timestamptz | |

---

### ga4_event_outbox
Outbox queue for GA4 events pending delivery via Measurement Protocol.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| brand_id | uuid | FK → brands |
| organization_id | uuid | FK → organizations |
| source_system | text | |
| source_event | text | |
| source_event_id | text | |
| idempotency_key | text | |
| client_id_used | text | |
| ga4_payload | jsonb | |
| raw_payload | jsonb | |
| status | text | pending/sent/failed/disabled/skipped — validated by trigger |
| skip_reason | text, nullable | |
| attempt_count | integer | |
| last_attempt_at | timestamptz, nullable | |
| next_attempt_at | timestamptz | |
| last_http_status | integer, nullable | |
| last_error | text, nullable | |
| ga4_purchase_sent_at | timestamptz, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### idea_comments
User comments on ideas.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| idea_id | uuid | FK → ideas |
| user_id | uuid | |
| content | text | |
| is_deleted | boolean | |
| created_at | timestamptz | |

---

### idea_votes
Weighted votes on ideas, capped per organization.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| idea_id | uuid | FK → ideas |
| user_id | uuid | |
| organization_id | uuid | |
| vote_weight | integer | Auto-calculated by trigger based on org revenue tier |
| created_at | timestamptz | |

---

### ideas
Product ideas from customers, meetings, or internal submissions.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| description | text, nullable | |
| status | text | |
| source | text | |
| organization_id | uuid, nullable | FK → organizations |
| contact_id | uuid, nullable | FK → contacts |
| created_by_user_id | uuid | |
| reviewed_by_user_id | uuid, nullable | Fireflies-created ideas require human review before publishing |
| review_notes | text, nullable | |
| transcript_reference | text, nullable | |
| weighted_vote_score | integer | |
| raw_vote_count | integer | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### infrastructure_checks
Health check results for monitored infrastructure services.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| service_id | uuid | FK → infrastructure_services |
| status | text | |
| status_code | integer, nullable | |
| response_time_ms | integer, nullable | |
| error_message | text, nullable | |
| metadata | jsonb, nullable | |
| checked_at | timestamptz | |

---

### infrastructure_services
Definitions of monitored infrastructure services.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| description | text, nullable | |
| category | text | |
| check_type | text | |
| check_url | text, nullable | |
| expected_status | integer, nullable | |
| check_config | jsonb, nullable | |
| display_order | integer | |
| public_visible | boolean | Controls customer portal visibility |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### lead_inbox
Inbound sales leads from forms, Calendly, and manual entry.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| sender_email | text | |
| sender_name | text, nullable | |
| company_name | text, nullable | |
| phone | text, nullable | |
| message | text, nullable | |
| source | text | |
| status | text | |
| subject | text, nullable | |
| body_preview | text, nullable | |
| website_url | text, nullable | |
| assigned_to | uuid, nullable | |
| qualified_at | timestamptz, nullable | |
| converted_at | timestamptz, nullable | |
| converted_org_id | uuid, nullable | FK → organizations |
| converted_opp_id | uuid, nullable | FK → opportunities |
| deal_id | uuid, nullable | FK → opportunities |
| ai_analysis | jsonb, nullable | |
| ai_analysis_generated_at | timestamptz, nullable | |
| created_at | timestamptz | |

---

### marketing_channels
Marketing channel integrations (GA4, LinkedIn, YouTube).

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| display_name | text | |
| channel_type | text | ga4/linkedin/youtube — validated by trigger |
| sync_status | text | active/paused/error |
| external_account_id | text, nullable | |
| last_synced_at | timestamptz, nullable | |
| last_sync_error | text, nullable | |
| created_at | timestamptz | |

---

### marketing_metric_snapshots
Daily metric snapshots for marketing channels.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| channel_id | uuid | FK → marketing_channels |
| metric_date | date | |
| metric_key | text | |
| metric_value_numeric | numeric, nullable | |
| metric_value_text | text, nullable | |
| raw_payload | jsonb, nullable | |
| created_at | timestamptz | |

---

### notifications
In-app notifications delivered to users.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| user_id | uuid | |
| type | text | |
| title | text | |
| body | text, nullable | |
| read | boolean | |
| project_id | uuid, nullable | FK → projects |
| metadata | jsonb, nullable | |
| created_at | timestamptz | |

---

### onboarding_phases
Phases within an onboarding project.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| onboarding_project_id | uuid | FK → onboarding_projects |
| template_phase_id | uuid, nullable | FK → onboarding_template_phases |
| name | text | |
| order_index | integer | |
| status | text | pending/in_progress/completed/skipped — validated by trigger |
| completed_at | timestamptz, nullable | |
| created_at | timestamptz | |

---

### onboarding_projects
Customer onboarding projects with status tracking.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| organization_id | uuid | FK → organizations |
| template_id | uuid, nullable | FK → onboarding_templates |
| status | text | active/paused/completed/cancelled — validated by trigger |
| started_at | date | |
| target_date | date, nullable | |
| completed_at | timestamptz, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### onboarding_task_files
File attachments on onboarding tasks.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| task_id | uuid | FK → onboarding_tasks |
| file_name | text | |
| file_url | text | |
| file_size | integer, nullable | |
| uploaded_by | uuid, nullable | |
| created_at | timestamptz | |

---

### onboarding_tasks
Individual tasks within onboarding phases.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| phase_id | uuid | FK → onboarding_phases |
| template_task_id | uuid, nullable | FK → onboarding_template_tasks |
| title | text | |
| description | text, nullable | |
| task_type | text | |
| assignee | text | |
| status | text | pending/in_progress/completed/skipped/blocked — validated by trigger |
| order_index | integer | |
| due_date | date, nullable | |
| visibility | text | |
| notes | text, nullable | |
| link_url | text, nullable | |
| link_instructions | text, nullable | |
| form_schema | jsonb, nullable | |
| form_response | jsonb, nullable | |
| approval_content | text, nullable | |
| approval_status | text, nullable | |
| completed_at | timestamptz, nullable | |
| completed_by | uuid, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### onboarding_template_phases
Reusable phase definitions for onboarding templates.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| template_id | uuid | FK → onboarding_templates |
| name | text | |
| description | text, nullable | |
| order_index | integer | |
| is_milestone | boolean | |
| created_at | timestamptz | |

---

### onboarding_template_tasks
Reusable task definitions within onboarding template phases.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| phase_id | uuid | FK → onboarding_template_phases |
| title | text | |
| description | text, nullable | |
| task_type | text | |
| default_assignee | text | |
| is_required | boolean | |
| offset_days | integer | |
| order_index | integer | |
| link_url | text, nullable | |
| link_instructions | text, nullable | |
| form_schema | jsonb, nullable | |
| approval_content | text, nullable | |
| created_at | timestamptz | |

---

### onboarding_templates
Named onboarding templates optionally tied to org types.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| description | text, nullable | |
| org_type | text, nullable | |
| is_active | boolean | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### opportunities
Sales pipeline deals linked to organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| primary_contact_id | uuid, nullable | FK → contacts |
| owner_user_id | uuid, nullable | |
| stage | text | Pipeline stage |
| estimated_subscription_mrr | numeric | |
| expected_monthly_transaction_revenue | numeric | |
| probability_pct | integer | |
| expected_close_date | date, nullable | |
| next_step | text, nullable | |
| lost_reason | text, nullable | |
| momentum_level | text | |
| risk_level | text | |
| stage_entered_at | timestamptz, nullable | Used for days-in-stage calculation |
| last_activity_at | timestamptz, nullable | Used for stale deal detection (14+ days) |
| closed_at | timestamptz, nullable | |
| ai_summary | jsonb, nullable | |
| ai_summary_generated_at | timestamptz, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### opportunity_activities
Activity log entries for sales opportunities.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| opportunity_id | uuid | FK → opportunities |
| organization_id | uuid, nullable | FK → organizations |
| contact_id | uuid, nullable | FK → contacts |
| type | text | |
| summary | text, nullable | |
| notes | text, nullable | |
| occurred_at | timestamptz | |
| external_ref | text, nullable | |
| fireflies_id | text, nullable | |
| is_pinned | boolean, nullable | |
| created_by | uuid | |
| created_at | timestamptz | |

---

### org_activity_events
Organization-level activity feed events. Auto-generated by DB triggers.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| event_type | text | |
| source_table | text, nullable | |
| source_id | uuid, nullable | |
| actor_id | uuid, nullable | |
| summary | text, nullable | |
| body | text, nullable | |
| visibility | text | |
| metadata | jsonb, nullable | |
| created_at | timestamptz | |

---

### org_kickoff_* tables (Kickoff Wizard)

Data collected in the Kickoff Wizard (`/kickoff/:orgId`). Most are one-to-one with `organizations` (PK = organization_id). Some are one-to-many.

| Table | Cardinality | Key columns / notes |
|---|---|---|
| org_kickoff_branding | one-to-one | primary_color, accent_color, font_preference, needs_font_help, logo/favicon URLs |
| org_kickoff_competitors | one-to-many | competitor_name, order_index |
| org_kickoff_featured_products | one-to-many | product_name, priority_order |
| org_kickoff_fulfillment | one-to-one | production_model, shipping_provider, pickup_in_store_enabled |
| org_kickoff_homepage | one-to-one | homepage_layout_style, pro_portal_required |
| org_kickoff_inspiration_links | one-to-many | url, order_index |
| org_kickoff_navigation_items | one-to-many | label, order_index |
| org_kickoff_product_categories | one-to-many | category_name, priority_order, produced_in_house |
| org_kickoff_production_software | one-to-many | software_name, order_index |
| org_kickoff_products | one-to-one | pricing_model, volume_pricing_enabled, tax_exempt_required, pro_discount_percent |
| org_kickoff_profiles | one-to-one | Existence marker only |
| org_kickoff_tax_payment | one-to-one | tax_region, payment_gateway, stripe_connected, stripe_webhook_ready |
| org_kickoff_vision | one-to-one | brand_positioning, primary_customer_type, launch_focus, target_launch_date |

---

### org_links
Bookmarked links for an organization with client visibility toggle.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| name | text | |
| url | text | |
| category | text | |
| client_visible | boolean | Controls customer portal visibility |
| created_by | uuid | |
| created_at | timestamptz | |

---

### org_meeting_action_items
Action items from organization meetings.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| meeting_id | uuid | FK → org_meetings |
| title | text | |
| owner_id | uuid, nullable | |
| due_date | date, nullable | |
| status | text | |
| linked_task_id | uuid, nullable | FK → tasks |
| created_at | timestamptz | |

---

### org_meeting_agenda_items
Agenda items for organization meetings.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| meeting_id | uuid | FK → org_meetings |
| topic | text | |
| details | text, nullable | |
| priority | text | |
| order_index | integer | |
| created_by | uuid, nullable | |
| created_at | timestamptz | |

---

### org_meeting_participants
Participants in organization meetings.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| meeting_id | uuid | FK → org_meetings |
| contact_id | uuid, nullable | FK → contacts |
| name | text, nullable | |
| email | text, nullable | |
| created_at | timestamptz | |

---

### org_meetings
Organization-level meetings with notes and customer visibility.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| title | text | |
| meeting_date | timestamptz, nullable | |
| status | text | |
| internal_summary | text, nullable | |
| decisions | text, nullable | |
| attachments | jsonb, nullable | |
| fireflies_url | text, nullable | |
| call_log_id | uuid, nullable | FK → call_log |
| visible_to_customer | boolean | |
| created_by | uuid, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### organization_members
Maps users to organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| user_id | uuid | FK → auth.users |
| organization_id | uuid | FK → organizations |
| role | text, nullable | |
| created_at | timestamptz | |

---

### organizations
Master company records — the core entity everything rolls up to.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| status | text | Lead/Onboarding/Active/On Hold |
| type | text | |
| country | text, nullable | |
| timezone | text, nullable | |
| onboarding_status | text | |
| pipeline_status | text | |
| integration_type | text | |
| customer_since | date, nullable | |
| subscription_rate | numeric | |
| billing_currency | text | |
| billing_account_id | text, nullable | |
| billing_email | text, nullable | |
| billing_link | text, nullable | |
| billing_notes | text, nullable | |
| billing_details | jsonb, nullable | |
| payment_terms_days | integer | |
| auto_charge_approved | boolean | |
| contract_json | jsonb | |
| loyalty_eligible | boolean | |
| is_ipi_member | boolean | |
| is_msp_subscriber | boolean | |
| quickbooks_customer_id | text, nullable | |
| quickbooks_customer_name | text, nullable | |
| expected_transaction_baseline | numeric, nullable | |
| has_physical_location | text, nullable | |
| location_count | text, nullable | |
| lead_source | text, nullable | |
| sales_owner_id | uuid, nullable | |
| sales_notes | text, nullable | |
| last_sales_activity_at | timestamptz, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### project_documents
Files uploaded to projects or organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| file_url | text | |
| project_id | uuid, nullable | FK → projects |
| organization_id | uuid, nullable | FK → organizations |
| uploaded_by | uuid | |
| client_visible | boolean | |
| created_at | timestamptz | |

---

### project_links
Bookmarked URLs attached to projects.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| project_id | uuid | FK → projects |
| name | text | |
| url | text | |
| description | text, nullable | |
| created_by | uuid | |
| created_at | timestamptz | |

---

### project_looms
Loom video recordings attached to projects.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| project_id | uuid | FK → projects |
| title | text | |
| loom_url | text | |
| video_id | text | |
| visible_to_customer | boolean, nullable | |
| created_by | uuid | |
| created_at | timestamptz | |

---

### project_meetings
Fireflies meeting transcripts linked to projects.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| project_id | uuid | FK → projects |
| fireflies_transcript_id | text | |
| title | text, nullable | |
| summary | text, nullable | |
| meeting_date | timestamptz, nullable | |
| duration_minutes | integer, nullable | |
| transcript_url | text, nullable | |
| opportunity_id | uuid, nullable | FK → opportunities |
| visible_to_customer | boolean | |
| created_at | timestamptz | |

---

### project_notes
Free-text notes on projects or organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| project_id | uuid, nullable | FK → projects |
| organization_id | uuid, nullable | FK → organizations |
| author_id | uuid | |
| content | text | |
| created_at | timestamptz | |

---

### project_templates
Reusable project templates.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| description | text, nullable | |
| org_type | text, nullable | |
| created_at | timestamptz | |

---

### projects
Active internal projects linked to organizations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| description | text, nullable | |
| organization_id | uuid, nullable | FK → organizations |
| template_id | uuid, nullable | FK → project_templates |
| status | text | |
| visibility | text | |
| icon | text, nullable | |
| color | text, nullable | |
| started_at | date, nullable | |
| target_launch_date | date, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### qbo_tokens
QuickBooks OAuth token storage (single-row pattern).

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| refresh_token | text | |
| access_token | text, nullable | |
| access_token_expires_at | timestamptz, nullable | |
| updated_at | timestamptz | |

---

### service_cost_entries
Monthly revenue vs cost tracking per service type.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| service_type | text | |
| period_year | integer | |
| period_month | integer | |
| revenue_amount | numeric | |
| cost_amount | numeric | |
| currency | text | |
| notes | text, nullable | |
| created_by | uuid | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### suppliers
Vendor records linked to cost entries.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| contact_name | text, nullable | |
| contact_email | text, nullable | |
| website | text, nullable | |
| default_currency | text | |
| default_category | text | |
| notes | text, nullable | |
| quickbooks_vendor_id | text, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### support_case_satisfaction
CSAT ratings on resolved support cases.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| support_case_id | uuid | FK → support_cases (one-to-one) |
| user_id | uuid | |
| rating | integer | 1–5, validated by trigger |
| comment | text, nullable | |
| created_at | timestamptz, nullable | |

---

### support_cases
Customer support tickets and bug reports with SLA tracking.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| organization_id | uuid | FK → organizations |
| created_by_user_id | uuid | |
| type | text | bug/ticket — validated by trigger |
| status | text | new/awaiting_triage/awaiting_customer/in_progress/resolved/closed |
| severity | text, nullable | blocking/major/minor |
| category | text, nullable | |
| subject | text | |
| app_error_id | text, nullable | |
| domain | text, nullable | |
| app_context | jsonb, nullable | |
| priority | text, nullable | |
| assignee_contact_id | uuid, nullable | FK → support_contacts |
| case_number | integer | Auto-assigned by trigger |
| first_response_at | timestamptz, nullable | SLA tracking |
| resolved_at | timestamptz, nullable | SLA tracking |
| sla_first_response_hours | numeric, nullable | |
| sla_resolution_hours | numeric, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### support_contacts
Pixfizz staff displayed in the customer support hub.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| name | text | |
| email | text | |
| title | text, nullable | |
| contact_type | text | Pixfizz/Expert |
| calendly_url | text, nullable | |
| public_notes | text, nullable | |
| avatar_url | text, nullable | |
| is_active | boolean | |
| receive_ticket_notifications | boolean | |
| sort_order | integer | Drag-and-drop ordering |
| excluded_org_ids | text[] | Orgs that should not see this contact |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### support_message_attachments
File attachments on support messages.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| support_message_id | uuid | FK → support_messages |
| storage_path | text | |
| file_name | text | |
| mime_type | text | |
| size_bytes | integer | |
| created_at | timestamptz | |

---

### support_message_links
URL links attached to support messages.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| support_message_id | uuid | FK → support_messages |
| url | text | |
| label | text, nullable | |
| created_at | timestamptz | |

---

### support_messages
Messages within support case conversations.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| support_case_id | uuid | FK → support_cases |
| author_user_id | uuid | |
| visibility | text | external/internal — validated by trigger |
| body | text | |
| is_system | boolean | |
| created_at | timestamptz | |

---

### task_comments
Comments on tasks.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| task_id | uuid | FK → tasks |
| author_id | uuid | |
| content | text | |
| created_at | timestamptz | |

---

### tasks
Unified task system for all work items (sales, onboarding, internal, customer).

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| description | text, nullable | |
| status | text | |
| priority | text | |
| category | text | One of 27 standardized categories — synced between DB and UI |
| task_type | text | |
| visibility | text | |
| execution_state | text | |
| assignee_id | uuid, nullable | Admin user |
| assignee_contact_id | uuid, nullable | FK → contacts |
| organization_id | uuid, nullable | FK → organizations |
| project_id | uuid, nullable | FK → projects |
| brand_id | uuid, nullable | FK → brands |
| opportunity_id | uuid, nullable | FK → opportunities |
| source_meeting_id | uuid, nullable | FK → org_meetings |
| weekly_win_id | uuid, nullable | FK → weekly_wins |
| due_date | timestamptz, nullable | |
| completed_at | timestamptz, nullable | |
| order_index | integer | |
| manual_rank | integer | |
| today_flag | boolean | Daily focus flag |
| is_must_win_today | boolean | Must Win Today flag |
| must_win_date | date, nullable | |
| waiting_on | text, nullable | |
| reference_link | text, nullable | |
| recurrence_rule | jsonb, nullable | Recurring pattern config |
| delegation_context | jsonb, nullable | |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### template_tasks
Task definitions within project templates.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| template_id | uuid | FK → project_templates |
| title | text | |
| description | text, nullable | |
| default_category | text | |
| default_visibility | text | |
| default_waiting_on | text | |
| offset_days | integer | |
| order_index | integer | |
| reference_link | text, nullable | |
| created_at | timestamptz | |

---

### user_roles
Maps users to application roles.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| user_id | uuid | unique with role — FK → auth.users |
| role | enum: app_role | admin/customer |
| created_at | timestamptz | |

---

### weekly_wins
Weekly win/goal items per user.

| Column | Type | Notes |
|---|---|---|
| id | uuid, PK | |
| title | text | |
| description | text, nullable | |
| status | text | |
| week_start_date | date | |
| owner_user_id | uuid | |
| organization_id | uuid, nullable | FK → organizations |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

## Two Postgres / Supabase Rules That Cost a Month of Silent Breakage

Established 2026-08-25 while fixing a wizard that had been unusable since 22 July.

### 1. An RLS policy on table X must never call a helper that re-reads table X

`INSERT … RETURNING` cannot see its own row. A `STABLE SECURITY DEFINER` function
that re-reads the target table returns false, the `RETURNING` clause is refused,
and **Postgres reports it as a WITH CHECK violation**:

```
new row violates row-level security policy for table "review_domains"
```

That message points at the INSERT policy, which is innocent. The SELECT-side policy
is the one at fault.

- **Express the policy against the row's own columns.**
- **Isolate it** by running the insert with and without `RETURNING` inside a
  rolled-back transaction. If it succeeds without `RETURNING`, this is the bug.
- Policies on *other* tables that reference X are fine, because X's row already
  exists by then.

### 2. Any column PostgREST upserts on needs a **non-partial** unique index

Postgres will not accept a partial unique index as an `ON CONFLICT` arbiter unless
the statement repeats the predicate, and PostgREST's `{ onConflict: "brand_id" }`
emits a plain `ON CONFLICT (brand_id)`:

```
42P10: there is no unique or exclusion constraint matching the ON CONFLICT specification
```

## Changelog
- 2026-03-26: Full schema populated from Lovable export. 88 tables documented.
- 2026-08-29: Added two Postgres/Supabase rules — an RLS policy on a table must not call a helper that re-reads that same table, because `INSERT … RETURNING` cannot see its own row and Postgres misreports the failure as a WITH CHECK violation on the innocent INSERT policy (isolate by running the insert without `RETURNING` in a rolled-back transaction); and any column PostgREST upserts on needs a non-partial unique index, or `onConflict` fails with `42P10`. Source: claude-chat.
