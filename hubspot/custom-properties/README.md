# Custom Properties Schema

Properties to add to HubSpot for AI automation and email tracking. Create in Settings -> Properties -> Contact properties.

---

## AI Scoring

| Property Name | Type | Purpose |
|--------------|------|---------|
| `ai_lead_score` | Number | AI-generated score 1-10 |
| `ai_lead_tags` | Multi-select | Tags from AI qualification (e.g., High Budget, Urgent) |
| `ai_qualification_reason` | Single-line text | One-sentence reason from AI |

---

## First Response Tracking

| Property Name | Type | Purpose |
|--------------|------|---------|
| `ai_first_response_sent` | Checkbox | Has the AI first email been sent? |
| `ai_first_response_sent_at` | Date and time | When it was sent |

---

## Follow-Up Sequence Flags

| Property Name | Type | Purpose |
|--------------|------|---------|
| `touch_day3_sent` | Checkbox | Touch 1 sent |
| `touch_day7_sent` | Checkbox | Touch 2 sent |
| `touch_day10_sent` | Checkbox | Touch 3 sent |
| `touch_day14_sent` | Checkbox | Touch 4 sent |
| `touch_day17_sent` | Checkbox | Touch 5 sent |
| `touch_day21_sent` | Checkbox | Touch 6 sent |
| `touch_day24_sent` | Checkbox | Touch 7 sent |
| `touch_day28_sent` | Checkbox | Touch 8 sent |

---

## Email Engagement Tracking

| Property Name | Type | Purpose |
|--------------|------|---------|
| `email_open_count` | Number | Total email opens |
| `email_click_count` | Number | Total link clicks |
| `email_last_opened_at` | Date and time | Last open timestamp |
| `email_last_clicked_at` | Date and time | Last click timestamp |
| `email_bounced` | Checkbox | Email bounced flag |
| `email_last_replied_at` | Date and time | Last reply timestamp |

---

## Deal-Level Custom Properties

Create these in Settings -> Properties -> Deal properties.

| Property Name | Type | Purpose |
|--------------|------|---------|
| `deal_source_campaign` | Single-line text | UTM campaign that generated the deal |
| `deal_source_medium` | Dropdown | organic / paid / referral / direct / email |
| `close_reason` | Dropdown | Won: Price / Won: Relationship / Won: Product Fit / Lost: Price / Lost: Competitor / Lost: Timing / Lost: Budget / Lost: No Decision |
| `last_stage_change_date` | Date | When the deal last moved stages (set via workflow) |
| `days_in_current_stage` | Number | Updated via n8n daily workflow |
| `proposal_sent_date` | Date | When the proposal was sent |
| `competitor_mentioned` | Multi-select | Which competitors came up in this deal |
| `product_tier` | Dropdown | Starter / Pro / Enterprise / Custom |
| `mrr_potential` | Number | Estimated monthly recurring revenue |
| `contract_length_months` | Number | Length of contract being proposed |
| `discount_applied` | Number | Discount percentage given |
| `deal_priority` | Dropdown | Low / Medium / High / Critical |

**Why track close reason:** If 60% of your losses are "Lost: Price", that's a positioning problem. If it's "Lost: Competitor", that's a differentiation problem. You can't see this without capturing it.

---

## Company-Level Custom Properties

Create these in Settings -> Properties -> Company properties.

| Property Name | Type | Purpose |
|--------------|------|---------|
| `company_size_band` | Dropdown | 1-10 / 11-50 / 51-200 / 201-500 / 500+ employees |
| `industry_vertical` | Dropdown | Your relevant industry options |
| `annual_revenue_band` | Dropdown | <$1M / $1M-$10M / $10M-$50M / $50M+ |
| `tech_stack` | Multi-select | Key technologies used (Shopify, Salesforce, WordPress…) |
| `account_tier` | Dropdown | Strategic / Growth / Standard / Dormant |
| `account_manager` | HubSpot user | Assigned account owner |
| `contract_renewal_date` | Date | When existing contract renews |
| `nps_score` | Number | Last NPS survey result |
| `total_revenue_to_date` | Number | Lifetime revenue from this company |
| `parent_company` | Company association | For subsidiaries — link to parent |

---

## Lifecycle Stage Automation Properties

These properties are set by workflows to drive lifecycle automation, not by humans manually.

Create as Contact properties:

| Property Name | Type | Purpose |
|--------------|------|---------|
| `lifecycle_automation_status` | Dropdown | Active Sequence / Paused / Completed / Opted Out |
| `current_sequence_name` | Single-line text | Name of the active n8n/HubSpot sequence |
| `sequence_start_date` | Date | When current sequence began |
| `sequence_current_step` | Number | Which step they're on (1-8 for 8-touch) |
| `lifecycle_entered_lead_date` | Date | When they became a Lead |
| `lifecycle_entered_mql_date` | Date | When they became Marketing Qualified |
| `lifecycle_entered_sql_date` | Date | When they became Sales Qualified |
| `lifecycle_entered_customer_date` | Date | When they converted |
| `days_to_first_response` | Number | Set once — how long until first reply |
| `days_lead_to_customer` | Number | Set once — total sales cycle length |

These allow you to calculate average sales cycle length, sequence drop-off rates, and lifecycle velocity in reporting.

---

## Product / Service Interest Tracking

Use these to know what the lead is actually interested in before you talk to them.

Create as Contact properties:

| Property Name | Type | Purpose |
|--------------|------|---------|
| `service_interest` | Multi-select | Your service/product options |
| `product_pages_viewed` | Multi-select | Which product pages they visited (set via tracking workflow) |
| `pricing_page_viewed` | Checkbox | Did they hit the pricing page? |
| `pricing_page_view_date` | Date | When they last viewed pricing |
| `demo_requested` | Checkbox | Did they request a demo? |
| `demo_request_date` | Date | When they requested |
| `content_downloaded` | Multi-select | Lead magnets they downloaded |
| `webinar_attended` | Checkbox | Attended any webinar |
| `webinar_name` | Single-line text | Which webinar they attended |

The pricing page view is particularly high-signal. A lead who views pricing within 48 hours of submitting a form is significantly more likely to convert than one who doesn't. Set a workflow: `pricing_page_viewed` changes to true → add tag "High Purchase Intent" → notify sales.

---

## Source Attribution Properties

First-touch attribution is not enough. Multi-touch tells the real story.

Create as Contact properties:

| Property Name | Type | Purpose |
|--------------|------|---------|
| `first_touch_source` | Dropdown | organic / paid-search / paid-social / email / referral / direct / event |
| `first_touch_campaign` | Single-line text | UTM campaign of first touch |
| `first_touch_date` | Date | Date of first touch |
| `last_touch_source` | Dropdown | Same options as first touch |
| `last_touch_campaign` | Single-line text | UTM campaign of converting touch |
| `last_touch_date` | Date | Date of converting touch |
| `touch_count` | Number | Total marketing touches before conversion |
| `referral_partner` | Single-line text | If referred, who sent them |
| `referral_partner_type` | Dropdown | Client / Partner / Affiliate / Organic |

Set first-touch properties on contact creation via workflow (they should never be overwritten). Set last-touch on each form submission. This lets you report on both first-touch and last-touch attribution in the same dashboard.

---

## GDPR Compliance Properties

If you have any EU-based contacts, these are not optional.

Create as Contact properties:

| Property Name | Type | Purpose |
|--------------|------|---------|
| `gdpr_consent_given` | Checkbox | Did they provide explicit consent? |
| `gdpr_consent_date` | Date | When consent was given |
| `gdpr_consent_source` | Single-line text | Which form/page consent was collected on |
| `gdpr_consent_type` | Dropdown | Marketing / Processing / Both |
| `gdpr_deletion_requested` | Checkbox | Has the contact requested data deletion? |
| `gdpr_deletion_request_date` | Date | When they requested deletion |
| `gdpr_data_export_sent` | Checkbox | Have you fulfilled a data export request? |
| `gdpr_legal_basis` | Dropdown | Consent / Legitimate Interest / Contract / Legal Obligation |

**Workflow to build:** When `gdpr_deletion_requested` changes to true:
1. Opt out of all marketing emails
2. Remove from all sequences
3. Create task "GDPR: Delete data for {{contact.email}} — required within 30 days", high priority
4. Assign to your DPO or admin

**Important:** HubSpot has a native GDPR delete feature (Settings -> Privacy & Consent). The properties above are in addition to HubSpot's built-in consent tools, for your own tracking and audit trail.

---

## Re-engagement Tracking

Track the re-engagement lifecycle separately so you don't confuse re-engaged contacts with new leads.

Create as Contact properties:

| Property Name | Type | Purpose |
|--------------|------|---------|
| `re_engagement_status` | Dropdown | Not Started / In Re-Engagement / Re-Engaged / Unresponsive |
| `re_engagement_start_date` | Date | When the re-engagement sequence began |
| `re_engagement_trigger` | Dropdown | Manual / Score Decay / No Activity 90 Days / Price Change / Product Update |
| `re_engagement_opens` | Number | Opens during the re-engagement sequence |
| `last_meaningful_engagement` | Date | Last open, click, reply, or meeting (rolled up manually or via workflow) |
| `times_re_engaged` | Number | How many re-engagement cycles this contact has been through |
| `original_disqualify_reason` | Single-line text | Why they were originally marked lost/cold |

These let you measure whether re-engagement is working. If contacts go through 3+ re-engagement cycles without converting, it's time to permanently archive them.

---

## How to Create

1. Settings -> Properties -> Contact properties (or Deal / Company as needed)
2. Click "Create property"
3. Set the field type and use the exact internal names from the tables above (snake_case)
4. Save

The internal name is what you reference in API calls and n8n. Use the names exactly as shown.

---

## API Usage

When updating properties via the HubSpot API:

```json
{
  "properties": {
    "ai_lead_score": "8",
    "ai_first_response_sent": "true",
    "email_open_count": "3"
  }
}
```

HubSpot returns all values as strings, even numbers and booleans. Parse them before comparing in n8n Code nodes.

---

## Bulk Property Updates via API

When you need to update a property across many contacts at once — for example, backfilling `first_touch_source` on an import — use the batch update endpoint instead of updating one by one.

### Batch update contacts

```
POST https://api.hubapi.com/crm/v3/objects/contacts/batch/update
```

```json
{
  "inputs": [
    {
      "id": "101",
      "properties": {
        "first_touch_source": "organic",
        "lifecycle_automation_status": "Active Sequence"
      }
    },
    {
      "id": "102",
      "properties": {
        "first_touch_source": "paid-social",
        "lifecycle_automation_status": "Completed"
      }
    }
  ]
}
```

The batch endpoint accepts up to 100 records per request. For larger sets, paginate in your n8n workflow with a loop — process 100 at a time with a short delay between batches to respect rate limits.

### Batch read contacts (get property values for many contacts at once)

```
POST https://api.hubapi.com/crm/v3/objects/contacts/batch/read
```

```json
{
  "properties": ["email", "ai_lead_score", "lifecycle_automation_status", "first_touch_source"],
  "inputs": [
    {"id": "101"},
    {"id": "102"},
    {"id": "103"}
  ]
}
```

---

## Association Creation via API

Associations link contacts to deals, deals to companies, and contacts to companies. Always create these after creating the objects themselves.

### Associate a contact to a deal

```
PUT https://api.hubapi.com/crm/v3/objects/contacts/{{contactId}}/associations/deals/{{dealId}}/contact_to_deal
```

This is a PUT with no request body. The association type `contact_to_deal` is a HubSpot-defined type.

### Associate a deal to a company

```
PUT https://api.hubapi.com/crm/v3/objects/deals/{{dealId}}/associations/companies/{{companyId}}/deal_to_company
```

### Batch create associations

For bulk operations, use the batch associations endpoint:

```
POST https://api.hubapi.com/crm/v4/associations/contacts/deals/batch/create
```

```json
{
  "inputs": [
    {
      "from": {"id": "{{contactId}}"},
      "to": {"id": "{{dealId}}"},
      "types": [
        {
          "associationCategory": "HUBSPOT_DEFINED",
          "associationTypeId": 4
        }
      ]
    }
  ]
}
```

Association type IDs (HUBSPOT_DEFINED):
- Contact → Deal: `4`
- Deal → Contact: `3`
- Contact → Company: `1`
- Company → Contact: `2`
- Deal → Company: `5`
- Company → Deal: `6`
- Note → Contact: `202`

---

## Deal Stage Updates via API

Moving a deal through stages programmatically requires knowing the internal stage ID.

### Get all pipeline stages and their IDs

```
GET https://api.hubapi.com/crm/v3/pipelines/deals/{{pipelineId}}
```

This returns each stage with its `id` field. Copy the stage IDs — you'll use them in deal updates.

Alternatively, fetch all pipelines:

```
GET https://api.hubapi.com/crm/v3/pipelines/deals
```

### Update a deal's stage

```
PATCH https://api.hubapi.com/crm/v3/objects/deals/{{dealId}}
```

```json
{
  "properties": {
    "dealstage": "{{stageId}}",
    "closedate": "2026-06-30T00:00:00Z",
    "deal_priority": "High"
  }
}
```

`closedate` must be an ISO 8601 timestamp in milliseconds or full datetime string.

### Create a deal with associations in one call (v3)

```
POST https://api.hubapi.com/crm/v3/objects/deals
```

```json
{
  "properties": {
    "dealname": "Acme Corp - Enterprise License",
    "dealstage": "{{stageId}}",
    "pipeline": "{{pipelineId}}",
    "amount": "12000",
    "closedate": "2026-06-30T00:00:00Z",
    "deal_source_campaign": "spring-2026-outbound"
  },
  "associations": [
    {
      "to": {"id": "{{contactId}}"},
      "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 3}]
    },
    {
      "to": {"id": "{{companyId}}"},
      "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 5}]
    }
  ]
}
```

This creates the deal and associates it to both the contact and company in a single API call.
