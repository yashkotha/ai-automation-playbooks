# HubSpot Track

This track covers everything you need to run HubSpot as a production CRM with AI automation layered on top, without paying for Sales Hub Pro or Operations Hub. It is organized into three focused areas: setting up the CRM itself (pipelines, permissions, dashboards, integrations), defining the custom property schema that AI and n8n workflows depend on, and building the n8n automation layer that connects HubSpot to the outside world. The target audience is founders, solo operators, and small sales teams who want enterprise-grade pipeline management without the $500/month seat fees. All core patterns run on HubSpot's free tier.

---

## Why HubSpot + n8n

### What HubSpot's free tier actually gives you

HubSpot's free CRM is genuinely useful out of the box. You get unlimited contacts, a deal pipeline, email tracking, a meeting scheduler, and basic task management at no cost. The limitation is not the data model -- it is the automation. Free and Starter tier workflows can trigger on contact creation and a handful of property changes, but they cannot branch on AI-generated scores, cannot call external APIs, cannot run multi-step logic, and cannot handle retry logic or error states. Anything involving cross-object actions (updating a company when a deal changes, creating a task when a call is logged) requires Operations Hub Professional, which starts at $720/month.

### What n8n unlocks

n8n connects to HubSpot's REST API directly, which means every capability in the API is available -- not just the subset HubSpot exposes in its workflow builder. You can respond to HubSpot webhooks (contact created, deal stage changed, form submitted), run that data through an AI model to generate a lead score, write the result back to a custom property, create a follow-up task, and send a Slack notification to the assigned rep, all in one workflow that costs nothing to run beyond your n8n server. The API supports batch operations (update 100 contacts in a single call), complex search queries with date range filters and compound conditions, and full association management between contacts, deals, companies, and activities. None of that requires a paid HubSpot tier.

### Cost comparison

| Capability | HubSpot native | With n8n |
|---|---|---|
| Custom workflow automation | Operations Hub Pro: $720/mo | n8n self-hosted: $0 |
| AI-powered lead scoring | Not available at any tier | OpenAI API: ~$0.01-0.03/lead |
| Cross-object automation | Operations Hub Pro | Included in n8n |
| Custom reporting beyond 25 dashboards | Reporting Add-on: $200/mo | n8n + any BI tool |
| Sales sequences | Sales Hub Pro: $500/seat/mo | n8n + Resend: ~$20/mo |
| Duplicate detection and data hygiene | Operations Hub Pro | n8n workflow: $0 |

A two-person sales team using HubSpot Pro with Sales Hub and Operations Hub runs roughly $2,400/month. The same capabilities with HubSpot free + n8n self-hosted runs under $30/month for the server.

---

## What's in Each Folder

### CRM Setup (`./crm-setup/`)

Configuration guides and templates for standing up a production HubSpot instance from scratch.

**Pipeline configurations** -- four complete pipeline setups with every stage defined, including stage names, probability values, required fields per stage, and automation rules attached to each transition:

- **SaaS trial-to-paid**: Trial Started, Onboarding Scheduled, Feature Activated, Expansion Opportunity, Proposal Sent, Closed Won, Closed Lost
- **Agency/services**: Inquiry Received, Discovery Call Booked, Discovery Complete, Proposal Sent, Contract Out, Closed Won, Closed Lost
- **E-commerce**: Cart Abandoned, Checkout Started, Payment Failed, Recovered, Fulfilled
- **Real estate**: Inquiry, Showing Scheduled, Offer Submitted, Under Contract, Closed

**Team permissions matrix** -- exact HubSpot permission sets for four roles: Admin (full access including settings and integrations), Sales Rep (own records only, no bulk editing, no exports), Marketing (contact and company read/write, no deals), Read-Only (view everything, edit nothing). Covers object-level, property-level, and tool-level permissions.

**Deal automation rules** -- n8n workflows triggered by deal stage changes: auto-assign to rep based on deal source, send stage change notifications to Slack with deal value and next steps, auto-create tasks at each stage with due dates, escalate deals stuck in a stage for more than N days.

**Task system configuration** -- HubSpot task types, queue setup, default due date logic per deal type, and task templates for each pipeline stage.

**Email templates** -- ready-to-use templates for: first contact (inbound form response), three-day follow-up, proposal sent confirmation, closed won (handoff to onboarding), closed lost (breakup with door open). Each template has subject line, body, and AI personalization notes.

**Reporting dashboards** -- five dashboard configurations with the specific reports inside each:

1. Pipeline Velocity: average days per stage, deals by stage, stage-to-stage conversion rates
2. Deal Source Attribution: closed revenue by source (organic, paid, referral, outbound), source-to-close rate by channel
3. Email Performance: open rate, click rate, reply rate, unsubscribe rate per sequence and per rep
4. Team Activity: calls logged, emails sent, meetings booked, tasks completed per rep per week
5. Forecast: weighted pipeline by close date, commit vs best case vs pipeline, monthly/quarterly view

**Integrations** -- step-by-step setup for Slack (deal notifications, assignment alerts, stage change pings) and Google Calendar (two-way sync with HubSpot meetings, automatic contact creation from calendar invites).

**Data hygiene automation** -- n8n workflows for: duplicate detection using email + company domain matching with a merge-or-flag decision tree, stale deal alerts (deals with no activity in 14 days), missing field flagging (contacts without company, phone, or lead source), and a weekly data quality report delivered to Slack.

**Migration guides** -- field mapping and import procedures for migrating from Salesforce, Pipedrive, and spreadsheets into HubSpot, including how to preserve activity history and association data.

---

### Custom Properties (`./custom-properties/`)

The full property schema used across all n8n automation workflows. Every property is documented with its internal name, type, field type, and which workflow reads or writes it.

**80+ custom properties across 8 categories:**

**AI Scoring**
- `ai_lead_score` (number) -- 1-10 score assigned by the AI qualification workflow
- `ai_lead_tags` (multi-checkbox) -- tags like "high_intent", "budget_confirmed", "decision_maker"
- `ai_qualification_reason` (multi-line text) -- the AI's plain-English explanation for the score

**First Response Tracking**
- `ai_first_response_sent` (checkbox) -- true once the first automated response has been sent
- `ai_first_response_sent_at` (datetime) -- timestamp of that first response

**Follow-Up Sequence Flags** (8 touches across 28 days)
- `touch_day3_sent`, `touch_day5_sent`, `touch_day7_sent`, `touch_day10_sent`, `touch_day14_sent`, `touch_day18_sent`, `touch_day21_sent`, `touch_day28_sent` -- each is a checkbox, set to true once that email has been sent; used to prevent re-sends if a workflow runs more than once

**Email Engagement**
- `open_count` (number) -- total email opens across all sequences
- `click_count` (number) -- total link clicks
- `last_opened_at` (datetime) -- timestamp of most recent open
- `last_clicked_at` (datetime) -- timestamp of most recent click
- `bounced` (checkbox) -- true if any email to this contact has bounced
- `last_replied_at` (datetime) -- timestamp of most recent inbound reply

**Deal-Level Properties**
- `deal_source_campaign` (single-line text) -- UTM campaign value from the originating form submission
- `close_reason` (dropdown) -- why the deal closed: price, competitor, timing, budget, no_need, ghosted
- `days_in_current_stage` (number) -- calculated by n8n each night; used for stale deal alerts
- `mrr_potential` (number) -- estimated monthly recurring revenue, set by rep or by AI
- `product_tier` (dropdown) -- starter, growth, pro, enterprise
- `competitor_mentioned` (single-line text) -- name of competitor mentioned in the original form submission or call notes

**Company-Level Properties**
- `company_size_band` (dropdown) -- 1-10, 11-50, 51-200, 201-500, 501-1000, 1000+
- `industry_vertical` (dropdown) -- 20 values mapped to ICP tiers
- `account_tier` (dropdown) -- tier_1 (ICP fit), tier_2 (marginal), tier_3 (outside ICP)
- `nps_score` (number) -- most recent NPS response
- `total_revenue_to_date` (number) -- cumulative closed revenue across all deals for this company

**Lifecycle Automation**
- `sequence_current_step` (number) -- which touch number the contact is currently on (1-8); used to resume sequences after a reply
- `days_to_first_response` (number) -- calculated: time between contact creation and first response sent
- `days_lead_to_customer` (number) -- calculated: time between MQL and Closed Won

**GDPR Compliance**
- `gdpr_consent_given` (checkbox) -- true if the contact has affirmatively opted in
- `gdpr_consent_date` (datetime) -- when consent was captured
- `gdpr_deletion_requested` (checkbox) -- triggers a data deletion workflow in n8n when set to true

**Full API examples included:**
- Batch update 100 contacts in a single API call using the `/crm/v3/objects/contacts/batch/update` endpoint
- Batch read with property filtering using `/crm/v3/objects/contacts/batch/read`
- Association v4 API with explicit type IDs for all six association types
- Creating a deal with embedded contact and company associations in a single `POST /crm/v3/objects/deals` call

---

### n8n Integrations (`./n8n-integrations/`)

A complete reference for building n8n workflows against HubSpot's API, plus ready-to-use workflow templates.

**API reference** for all seven core objects:
- Contacts: create, read, update, delete, batch operations, search
- Deals: create with associations, stage updates, pipeline management
- Companies: create, associate with contacts, batch update
- Associations: v4 API, all type IDs, bidirectional association management
- Tasks: create with due dates and assigned owners, mark complete
- Calls: log with outcome and duration, associate with contact and deal
- Meetings: log scheduled meetings, link to calendar events
- Notes: create with HTML body, associate across multiple objects

**All 6 association type IDs** for the v4 Association API:
- Contact-to-Deal: `contact_to_deal` (type ID 4)
- Deal-to-Contact: `deal_to_contact` (type ID 3)
- Contact-to-Company: `contact_to_company` (type ID 1)
- Company-to-Contact: `company_to_contact` (type ID 2)
- Deal-to-Company: `deal_to_company` (type ID 5)
- Company-to-Deal: `company_to_deal` (type ID 6)

**Complex search queries** -- example filter groups for:
- Contacts created in the last 7 days with `ai_lead_score` greater than 7
- Deals in a specific pipeline with no activity in the last 14 days using `BETWEEN` on `notes_last_updated`
- Contacts where `touch_day3_sent` is false and `ai_first_response_sent` is true (ready for next touch)
- Companies with `account_tier` equal to tier_1 and `industry_vertical` containing specific values using `EQ`, `NEQ`, `CONTAINS`, `NOT_HAS_PROPERTY` operators

**Bulk pagination** -- how to iterate through large contact lists using the `after` cursor in search responses, with a loop node pattern in n8n that handles datasets of any size without hitting memory limits.

**Webhook setup** -- configuring HubSpot's webhook subscriptions for contact creation, deal stage changes, and form submissions; verifying the `X-HubSpot-Signature` header in n8n using the app secret; handling duplicate webhook deliveries with idempotency checks.

**Error handling** -- per status code:
- `400 Bad Request`: log property name and value, send to a dead-letter queue for manual review
- `401 Unauthorized`: alert on token expiry, trigger token rotation flow
- `403 Forbidden`: log the specific permission missing, escalate to admin
- `404 Not Found`: handle gracefully (contact may have been deleted), mark workflow as skipped
- `429 Rate Limited`: exponential backoff starting at 1 second, doubling up to 32 seconds, then alerting if still failing
- `500 Server Error`: retry three times with linear backoff, then escalate to Slack

**Creating deals with associations** -- a single API call pattern that creates a deal, links it to an existing contact, and links it to an existing company without requiring three separate requests.

---

## Common Workflows You Can Build

1. **AI lead qualification on form submit** -- When a HubSpot form is submitted, a webhook fires to n8n within seconds. n8n sends the form data to OpenAI with a scoring prompt, writes `ai_lead_score`, `ai_lead_tags`, and `ai_qualification_reason` back to the contact, and sends a Slack message to the assigned rep if the score is 8 or higher. Total latency from form submit to scored contact: under 30 seconds.

2. **Automatic deal creation from inbound leads** -- When a contact is scored 6+ and does not already have an open deal, n8n creates a deal in the appropriate pipeline, sets the deal source from the UTM data on the contact, associates the deal with the contact and their company, and assigns it to a rep based on round-robin or territory logic.

3. **Stage change notifications with context** -- When a deal moves to "Proposal Sent," n8n fetches the deal, the associated contact, and the associated company, formats a Slack message with deal value, contact name, company, days in pipeline, and the AI qualification reason, and posts it to the team's deal channel.

4. **Data hygiene nightly run** -- A scheduled n8n workflow runs every night at 2am, searches for contacts missing required fields, contacts with duplicate email addresses, and deals with no activity in 14 days. It updates `days_in_current_stage` on every open deal, creates tasks for reps who have stale deals, and posts a data quality summary to Slack each morning.

5. **GDPR deletion pipeline** -- When `gdpr_deletion_requested` is set to true on any contact, n8n triggers a workflow that deletes all notes, calls, and emails associated with the contact, removes the contact from all active sequences, and finally deletes the contact record from HubSpot. It logs the deletion event to an external audit table before the final delete.

6. **Competitor intelligence tracking** -- When a new contact is created and `competitor_mentioned` is not empty, n8n routes that lead to a specific rep who handles competitive deals, creates a task to prepare a competitive battlecard, and logs the competitor name to a running tally in a Google Sheet.

7. **Closed-lost revival trigger** -- When a deal is moved to Closed Lost and `close_reason` is set to "timing," n8n creates a task for 90 days in the future and adds the contact to a re-engagement sequence. If the reason is "price" or "competitor," it creates a task for 180 days instead.

8. **Meeting booked confirmation and prep** -- When a meeting is created in HubSpot (via Calendly webhook or directly), n8n fetches the contact's `ai_lead_score`, `ai_qualification_reason`, and recent activity, formats a briefing document, and sends it to the rep via Slack 30 minutes before the meeting.

---

## Prerequisites

**HubSpot tier required**: The free tier covers everything in this track. A few advanced reporting features (custom report builder, attribution reports) require Marketing Hub Starter ($20/month). HubSpot's Private App system is available on all tiers including free.

**Getting a Private App token**:
1. Go to Settings > Integrations > Private Apps
2. Create a new app with a descriptive name (e.g., "n8n Automation")
3. Under Scopes, enable: `crm.objects.contacts.read`, `crm.objects.contacts.write`, `crm.objects.deals.read`, `crm.objects.deals.write`, `crm.objects.companies.read`, `crm.objects.companies.write`, `crm.schemas.contacts.read`, `crm.schemas.contacts.write`, `timeline`, `sales-email-read`
4. Copy the token and store it in n8n as a credential of type "HTTP Header Auth" with the header name `Authorization` and value `Bearer YOUR_TOKEN`

**n8n**: Self-hosted on any server with 1GB RAM. The workflows in this track use HTTP Request nodes, not the native HubSpot node, so you can run them on n8n Community Edition without any paid features.

---

## Cost Reality

| Feature | Free | Paid (and what tier) |
|---|---|---|
| Contacts (unlimited) | Yes | -- |
| 1 pipeline, 1 deal board | Yes | -- |
| Email tracking | Yes | -- |
| Meeting scheduler | Yes | -- |
| Basic tasks | Yes | -- |
| Multiple pipelines | No | Sales Hub Starter: $20/mo |
| Sequences (native) | No | Sales Hub Pro: $500/seat/mo |
| Workflow automation (native) | 1 workflow on Starter | Operations Hub Pro: $720/mo |
| Custom report builder | No | Marketing Hub Starter: $20/mo |
| Calculated properties | No | Operations Hub Pro: $720/mo |
| Duplicate management (native) | No | Operations Hub Pro: $720/mo |

The strategy in this track: use HubSpot free for data storage, contact management, and activity logging. Use n8n for everything that would otherwise require a paid HubSpot tier. The only paid HubSpot feature worth considering at early stage is multiple pipelines (Sales Hub Starter at $20/month), if you have more than one product line or sales motion.
