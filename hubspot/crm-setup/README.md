# HubSpot CRM Setup from Scratch

A practical guide for getting HubSpot production-ready, not just configured.

---

## Step 1: Design Your Pipelines

Create pipelines that match your actual sales process, not HubSpot's defaults.

**For a service or real estate business:**
1. New Lead
2. Contacted
3. Meeting Scheduled
4. Proposal / Quote Sent
5. Negotiation
6. Closed Won
7. Closed Lost

**For high-volume B2C:**
1. New Lead
2. Qualified
3. Active
4. Closed Won
5. Closed Lost

Don't add stages for things that don't require a distinct action. More stages means more cognitive load and more deals stuck in limbo.

---

## Pipeline Types by Business Model

Different business models need fundamentally different pipeline structures. Here are production-tested templates.

### SaaS Pipeline

Free trials and product-led growth have a different motion than high-touch sales. Use this if your product has a trial or freemium tier.

| Stage | Probability | Purpose |
|-------|-------------|---------|
| Trial Started | 5% | User signed up for free trial |
| Trial Active | 15% | Logged in at least twice |
| Trial Engaged | 30% | Used a core feature |
| Demo Requested | 50% | Requested to speak with sales |
| Proposal Sent | 65% | Pricing proposal delivered |
| Contract Out | 80% | Contract sent, awaiting signature |
| Closed Won | 100% | Paid customer |
| Closed Lost | 0% | Churned or disqualified |

**Custom properties to add for SaaS:**
- `trial_start_date` — date the trial began
- `trial_end_date` — when it expires
- `product_tier_interest` — Starter / Pro / Enterprise (dropdown)
- `mrr_potential` — estimated monthly recurring revenue (number)
- `expansion_opportunity` — checkbox for upsell candidates

**Key SaaS pipeline automations:**
- When trial expires and deal is still in "Trial Active" → move to "Closed Lost", set reason "Trial Expired"
- When demo is booked → assign to sales rep, create follow-up task for 1 hour before meeting
- When deal reaches "Contract Out" → create reminder task for 3 days, escalate to manager at 7 days

---

### Agency / Consulting Pipeline

Project-based work where scope definition is its own phase.

| Stage | Probability | Purpose |
|-------|-------------|---------|
| Inquiry | 10% | Initial contact made |
| Discovery Call | 25% | Introductory call scheduled or completed |
| Needs Assessment | 40% | Questionnaire sent, requirements gathering |
| Scoping | 55% | Writing the proposal/SOW |
| Proposal Sent | 65% | Awaiting client review |
| Negotiation | 75% | Pricing or scope discussion |
| Contract Signed | 90% | SOW signed, deposit pending |
| Closed Won | 100% | Deposit received, project active |
| Closed Lost | 0% | Did not proceed |
| On Hold | 0% | Paused by client, revisit date set |

**Custom properties for agency:**
- `project_type` — Web / Brand / SEO / Social / Full-Service (multi-select)
- `estimated_project_value` — number
- `project_start_date` — date
- `referral_source_name` — text (who referred them)
- `client_tier` — A / B / C based on LTV potential

---

### E-commerce Pipeline (Wholesale / B2B)

For businesses selling physical products in bulk to retailers or distributors.

| Stage | Probability | Purpose |
|-------|-------------|---------|
| New Account Inquiry | 5% | Retail buyer reached out |
| Qualification | 20% | Verified they meet minimum order criteria |
| Sample Request | 35% | Physical samples sent |
| Line Review | 50% | Buyer is evaluating for their assortment |
| PO Negotiation | 70% | Terms, pricing, MOQ being discussed |
| PO Received | 85% | Purchase order in hand |
| Fulfilled | 100% | Order shipped |
| Closed Lost | 0% | Did not order |

**Custom properties for e-commerce:**
- `account_type` — Retail / Distributor / Marketplace / Direct (dropdown)
- `minimum_order_met` — checkbox
- `territory` — region this account covers
- `annual_order_volume_target` — number

---

### Real Estate Pipeline

For residential or commercial real estate where each transaction is a deal.

| Stage | Probability | Purpose |
|-------|-------------|---------|
| New Lead | 5% | Inquiry from any source |
| Contacted | 15% | Successfully spoke with the lead |
| Consultation Scheduled | 25% | Meeting booked |
| Active Buyer / Seller | 40% | Working relationship established |
| Offer Submitted | 60% | Offer in on a property |
| Under Contract | 75% | Offer accepted, in due diligence |
| Clear to Close | 90% | Conditions met, closing scheduled |
| Closed Won | 100% | Transaction complete |
| Closed Lost | 0% | Lost to another agent or withdrawn |

**Custom properties for real estate:**
- `buyer_seller` — Buyer / Seller / Both (dropdown)
- `price_range_min` / `price_range_max` — numbers
- `pre_approval_status` — Not Started / In Progress / Approved (dropdown)
- `target_neighborhoods` — multi-select
- `transaction_side` — Buy-Side / List-Side / Dual Agency

---

## Step 2: Add Custom Properties

HubSpot's default properties cover most basics. Add custom ones for:
- AI-generated scores and tags
- Email automation tracking flags
- Engagement data (opens, clicks)

See [Custom Properties](../custom-properties/) for the full schema to copy.

---

## Step 3: Connect Your Email

Settings -> Integrations -> Email Integrations

Connect Google Workspace or Outlook 365. Once connected, all replies from contacts are automatically logged on the contact timeline.

For Outlook 365, SMTP AUTH must be enabled at the tenant level before the integration works. See [n8n Integrations](../n8n-integrations/README.md) for the setup steps.

---

## Step 4: Keep Data Clean from Day One

These habits prevent hours of cleanup later:
- Set required fields on all forms (minimum: name + email)
- Never import without mapping every column
- Use a naming convention for test records (prefix with "TEST" or "DELETE")
- Set Lead Status as a required field on import

---

## Step 5: Build Your Core Views

Create these saved views immediately:

| View | Filter |
|------|--------|
| Hot Leads | `ai_lead_score >= 8` |
| Needs First Response | Contacts: `first_response_sent` not set |
| No Activity 7 Days | Deals: `last_activity_date` older than 7 days |
| Active Deals by Stage | Open deals, grouped by stage |

These replace a daily manual review of the full contact list.

---

## Step 6: Add a Product Catalog

Settings -> Sales -> Products

Add your products or service tiers with prices. Attach them to deals to get accurate pipeline value. This makes the AI pipeline summary prompt much more useful because it can report on real revenue numbers.

---

## Step 7: Connect Instagram Lead Ads (if applicable)

Settings -> Integrations -> Ad Networks -> Facebook

This auto-imports new Instagram and Facebook Lead Ad submissions as contacts. Pair with the [Instagram Lead Import workflow](../../ai-community/n8n-workflow-templates/README.md) to trigger the AI qualifier immediately on import.

---

## Team Permissions Setup

Use HubSpot's user roles and teams to control what each person can see and do. Misconfigured permissions lead to reps overwriting each other's data or seeing deals they shouldn't.

### Role Structure (recommended minimum)

| Role | Permissions | Who |
|------|-------------|-----|
| Admin | Full access, can edit properties, pipelines, automations | Owner / ops person only |
| Sales Rep | Can create/edit/delete their own records. View-only on others | Each rep |
| Sales Manager | Can view/edit all records in their team. Cannot change pipeline config | Team leads |
| Marketing | Can view contacts and deals, edit marketing properties, cannot delete | Marketing team |
| Read Only | View everything, edit nothing | Executives, external contractors |

### How to configure

1. Settings -> Users & Teams -> Create teams by department
2. Settings -> Users & Teams -> Permissions -> Create custom permission sets
3. Assign each user to a team and a permission set
4. For deal visibility: Settings -> Properties -> Deal Owner field — ensure it's always set on import/creation

**Critical:** Enable "Record-level permissions" on deals (Sales Hub Professional+). This lets you restrict reps to only see deals they own. Without it, reps can see the entire pipeline.

### Team-based views

Create separate saved views per team so each team's default view shows only their records:
- Filter: `Deal Owner is any of [rep1, rep2, rep3]`
- Pin this as the default view for each sales team

---

## Deal Automation Rules

Set these up in Settings -> Workflows. They run automatically and eliminate manual status updates.

### Auto-create a deal from a new contact

Trigger: Contact is created AND `lead_source` is "Website Form"
Action: Create Deal in "New Lead" stage, set Deal Name to "{{contact.firstname}} {{contact.lastname}} - Inquiry", associate deal to contact

### Move deal when meeting is booked

Trigger: Meeting is associated with a deal AND deal stage is "Contacted"
Action: Move deal to "Meeting Scheduled" stage, set `meeting_date` property

### Flag deals with no activity

Trigger: Scheduled workflow runs daily at 8am
Filter: Open deals where `hs_last_activity_date` is more than 7 days ago
Action: Add internal note "No activity in 7+ days", notify deal owner via email

### Auto-close stale deals

Trigger: Scheduled, weekly
Filter: Open deals where `createdate` is more than 90 days ago AND stage is "New Lead" or "Contacted"
Action: Move to "Closed Lost", set close reason "Gone Cold - Auto Closed", notify owner

### Pipeline stage change timestamp

Trigger: Deal property `dealstage` is changed
Action: Set custom property `last_stage_change_date` to today
(This lets you track velocity — how long deals spend in each stage.)

---

## Task and Reminder Setup

Tasks in HubSpot keep follow-ups from falling through the cracks. Build these standard tasks via workflow automations.

### Automatic task templates

**On new hot lead:**
- Trigger: Contact `ai_lead_score` changes to >= 8
- Action: Create task "Call {{contact.firstname}} — HOT LEAD", due in 30 minutes, assign to contact owner, priority High

**On proposal sent:**
- Trigger: Deal stage changes to "Proposal Sent"
- Action: Create task "Follow up on proposal — {{deal.dealname}}", due in 3 days, assign to deal owner

**On no reply after follow-up 2:**
- Trigger: `touch_day7_sent` = true AND `email_last_replied_at` not set AND date is 7+ days after `touch_day7_sent`
- Action: Create task "Manual check-in needed — {{contact.firstname}}", assign to contact owner

### Manual task queue

In HubSpot, go to Sales -> Tasks. Create a queue called "Daily Call List". When reps start their day, they work through this queue rather than managing their own list. Tasks created by the above automations feed into this queue automatically when you assign them to the "Daily Call List" queue in workflow actions.

---

## Email Templates in HubSpot

Templates live in Sales -> Templates (or Conversations -> Templates for shared ones). These are for manual sends, not sequences.

### Templates to create immediately

**Template 1: Initial follow-up (manual override)**
```
Subject: Re: Your inquiry about [SERVICE]

Hi {{contact.firstname}},

Thanks for reaching out. I've reviewed your message and wanted to follow up personally.

[3-4 lines customized to their specific situation]

Would it make sense to jump on a quick 15-minute call this week?

[CALENDLY LINK]
```

**Template 2: Proposal follow-up**
```
Subject: Following up — {{deal.dealname}} proposal

Hi {{contact.firstname}},

I wanted to check in on the proposal I sent over. Happy to answer any questions or walk through it together on a quick call.

Let me know if you'd like to talk through anything.
```

**Template 3: Breakup email (last touch)**
```
Subject: Closing the loop

Hi {{contact.firstname}},

I haven't heard back from you so I'll assume the timing isn't right.

I'll leave the door open — feel free to reach out whenever it makes sense for you.
```

**Template 4: Re-engagement (for cold leads 90+ days)**
```
Subject: Still relevant for you?

Hi {{contact.firstname}},

We spoke/connected a while back. Things may have changed since then.

If [PROBLEM YOU SOLVE] is still something you're thinking about, I'd be happy to pick up where we left off.
```

How to create: Sales -> Templates -> New Template. Use `{{contact.property_name}}` tokens for personalization. These tokens auto-populate when you use the template from a contact or deal record.

---

## Reporting Dashboards to Build

Go to Reports -> Dashboards -> Create Dashboard.

### Dashboard 1: Daily Sales Overview

| Report | Type | What it shows |
|--------|------|---------------|
| Deals by stage | Funnel chart | Where deals are right now |
| Deals closing this month | Deal list | Revenue at risk |
| New leads today | Contact list | Today's inbound volume |
| Tasks due today | Activity list | What reps need to do |
| Hot leads (score >= 8) | Contact list | Prioritized follow-up queue |

### Dashboard 2: Pipeline Health

| Report | Type | What it shows |
|--------|------|---------------|
| Deal velocity by stage | Bar chart | Avg days in each stage |
| Win/loss ratio | Pie chart | % won vs lost |
| Deals by close date | Timeline | Pipeline spread |
| Average deal value by rep | Bar chart | Rep performance |
| Deals with no activity 7d | List | At-risk deals |

### Dashboard 3: Lead Source Analysis

| Report | Type | What it shows |
|--------|------|---------------|
| Contacts by original source | Pie chart | Where leads come from |
| Lead-to-deal conversion by source | Funnel | Which sources convert best |
| Revenue by source | Bar chart | Which sources generate most revenue |
| Avg lead score by source | Bar chart | Which sources send quality leads |

### Dashboard 4: Email Engagement

| Report | Type | What it shows |
|--------|------|---------------|
| Email open rate over time | Line chart | Trend in engagement |
| Click-through rate by template | Bar chart | Which templates work |
| Avg opens per contact | Number | Engagement depth |
| Contacts never opened | Count | At risk of disengagement |

To build custom reports that use your custom properties (like `ai_lead_score`), use Reports -> Custom Report Builder. Choose "Contacts" as the data source, add your properties as columns.

---

## Integrations Beyond Email

### Slack Integration

Install: App Marketplace -> HubSpot for Slack (official)

Once installed, you can:
- Get deal notifications in a Slack channel when a deal changes stage
- Get notified when a hot lead comes in
- Use `/hubspot` slash command to look up contact info from Slack

Recommended notification setup:
1. Create a `#crm-hot-leads` channel
2. Set workflow: when `ai_lead_score` >= 8 → send Slack notification to `#crm-hot-leads` with contact name, score, source, email
3. Create `#deals-won` channel with workflow for Closed Won deal notifications

### Calendar Integration (Google / Outlook)

Settings -> Integrations -> Calendar

Once connected:
- Meetings booked via HubSpot meeting links appear on your calendar
- Calendar events can be logged as activities on contact timelines
- HubSpot will suggest "this meeting was with X contact" based on attendee email matching

Set up a HubSpot Meeting Link (Sales -> Meeting Links):
- Connects to your calendar
- Prospects pick a time from your real availability
- Meeting auto-creates on the contact record when booked
- Use in email templates as your CTA link

### Zapier / Make (for connecting tools HubSpot doesn't natively support)

For tools without a native HubSpot integration, build via Zapier or Make:
- Typeform → HubSpot contact creation
- Calendly → Deal stage update on meeting booked
- Stripe → Deal update when payment received
- Shopify → Contact creation on first purchase

For n8n-based integrations, see [n8n Integrations](../n8n-integrations/README.md).

---

## Data Hygiene Automation

Bad data compounds. A contact with a wrong email address will skew your open rate metrics, waste your follow-up sequences, and pollute your AI scoring. Build these automations on day one.

### Bounce handling

Trigger: Contact property `hs_email_hard_bounce_reason` is known (has a value)
Action:
1. Set `email_bounced` = true
2. Set `hs_email_optout` = true (prevents accidental sends)
3. Set contact `hs_lead_status` = "Unqualified"
4. Add internal note "Email hard-bounced — do not email"
5. Create task "Verify alternate email for {{contact.firstname}}", assign to contact owner

### Unsubscribe cleanup

Trigger: Contact `hs_email_optout` changes to true
Action:
1. Move all associated open deals to "Closed Lost" with reason "Unsubscribed"
2. Remove from all active sequences
3. Add internal note with timestamp

### Missing required field alert

Trigger: Contact is created AND `phone` is not known AND `email` is not known
Action: Create task "Missing contact info — add phone or verify email for {{contact.firstname}}", high priority, assign to contact owner

### Stale contact archiving

Trigger: Scheduled, monthly
Filter: Contacts where `hs_last_activity_date` > 365 days ago AND `ai_lead_score` < 4 AND `email_open_count` < 1
Action: Set `hs_lead_status` = "Unqualified", add tag "Archived - No Engagement"

This keeps your active contact list clean without deleting data (you may need it later).

---

## Duplicate Contact Merging Strategy

HubSpot creates duplicates when the same person submits forms multiple times, or when you import from multiple sources. Left unmanaged, a person gets the same email twice and your engagement tracking splits across two records.

### Finding duplicates

1. Go to Contacts -> Actions -> Manage Duplicates
2. HubSpot uses email address and name similarity to suggest pairs
3. Review each pair before merging — HubSpot is not always right

### Merge rules

When merging, HubSpot keeps one record as "primary" and pulls properties from both. Rules for which record to keep primary:
- Keep the record with more activity (notes, emails, calls)
- Keep the record with the lower HubSpot ID (older record), unless the newer one has more complete data
- The primary record's contact owner stays as owner

Properties that need manual review after merge:
- `ai_lead_score` — take the higher value
- `email_open_count` — HubSpot will sum these, verify the total is accurate
- `hs_lead_status` — take the more advanced status
- Sequence enrollment — check if the contact is enrolled in any active sequence and re-enroll if needed

### Preventing duplicates at the source

- On all forms: HubSpot deduplicates by email by default. Make email required on every form.
- On import: Before importing, deduplicate your CSV by email address using Excel or a tool like OpenRefine
- On API creation: Before creating a contact via API, always search first (`POST /crm/v3/objects/contacts/search` by email). Only create if not found.

### Automated dedup workflow (n8n)

Build a monthly n8n workflow that:
1. Exports all contacts from HubSpot
2. Groups by email address
3. Flags any email with more than one record
4. Creates a HubSpot task per duplicate set: "Merge duplicates: {{email}}"
5. Assigns to admin

This catches duplicates that HubSpot's built-in tool misses (different emails, same name/phone).

---

## Importing from Other CRMs

Moving from Salesforce, Pipedrive, Zoho, or a spreadsheet? The order of operations matters.

### Pre-import checklist

Before importing anything:
1. Export a full backup of your current CRM as CSV
2. Map every column in your export to a HubSpot property (or decide to skip it)
3. Identify which records are active vs archived — only import active records first
4. Remove duplicates from your export (deduplicate by email in Excel with `COUNTIF`)
5. Standardize phone number formats (+1 country code, no spaces)
6. Tag the import file with a source: add a column `import_source` = "Migrated from Salesforce"

### Import order

Import in this order to preserve associations:
1. Companies first (they have no dependencies)
2. Contacts second (associate to companies during import using company name or domain)
3. Deals third (associate to contacts using email)
4. Notes/activities last (if you need history — this is optional)

### HubSpot import settings

Contacts -> Import -> Start an import -> File from computer

Key settings:
- "Update existing" — yes, match on email address
- "Create associations" — yes, if your file includes company name/domain
- "Set as marketing contacts" — only if you plan to email them via HubSpot marketing tools

### Post-import cleanup

After import:
1. Check for contacts with no email address — set `hs_lead_status` = "Unqualified" for these
2. Check for deals with no associated contact — these are orphaned records, either fix associations or delete
3. Run the duplicate check (Contacts -> Manage Duplicates)
4. Set `ai_first_response_sent` = true on all imported contacts to prevent them getting your new-lead welcome email

### Migrating from specific CRMs

**From Pipedrive:**
- Pipedrive calls deals "Deals" and contacts "People" — maps cleanly
- Custom fields in Pipedrive become custom properties in HubSpot — create them before importing
- Pipeline stages won't match — map each Pipedrive stage to the closest HubSpot stage manually

**From Salesforce:**
- Export Objects: Lead, Contact, Account, Opportunity separately
- Merge Leads + Contacts into one file before importing as HubSpot contacts
- Salesforce "Opportunity" = HubSpot "Deal" — map Stage to dealstage
- Salesforce custom fields (prefixed with `__c`) → create equivalent HubSpot custom properties first

**From a spreadsheet:**
- Minimum required columns: Email, First Name, Last Name
- Recommended: Phone, Company, Lead Status, Source, Notes
- Add a column `import_date` = today's date so you can filter this cohort later
