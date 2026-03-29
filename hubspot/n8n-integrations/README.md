# n8n + HubSpot Integration

How to connect n8n to HubSpot and the most commonly needed API patterns.

---

## Authentication

Use a Private App token. It's simpler than OAuth and doesn't expire.

HubSpot: Settings -> Integrations -> Private Apps -> Create app

Scopes needed for most automation workflows:
- `crm.objects.contacts.read`
- `crm.objects.contacts.write`
- `crm.objects.deals.read`
- `crm.objects.deals.write`
- `crm.objects.companies.read`
- `crm.objects.companies.write`
- `crm.objects.notes.write`
- `crm.objects.tasks.write`
- `crm.objects.meetings.write`
- `timeline` (for logging notes and activities)
- `crm.schemas.deals.read` (for reading pipeline stage IDs)

In n8n, add as a Header Auth credential:
- Name: `Authorization`
- Value: `Bearer pat-xx-xxxxxxxxx`

---

## Core API Patterns

### Search contacts with a filter

```
POST https://api.hubapi.com/crm/v3/objects/contacts/search
```

```json
{
  "filterGroups": [{
    "filters": [{
      "propertyName": "ai_first_response_sent",
      "operator": "NOT_HAS_PROPERTY"
    }]
  }],
  "properties": ["email", "firstname", "lastname", "hs_lead_status"],
  "limit": 100
}
```

### Update a contact property

```
PATCH https://api.hubapi.com/crm/v3/objects/contacts/{{contactId}}
```

```json
{
  "properties": {
    "ai_lead_score": "8",
    "ai_first_response_sent": "true"
  }
}
```

### Log a note on the contact timeline

```
POST https://api.hubapi.com/crm/v3/objects/notes
```

```json
{
  "properties": {
    "hs_note_body": "AI first response sent at {{timestamp}}",
    "hs_timestamp": "{{timestamp_ms}}"
  },
  "associations": [{
    "to": {"id": "{{contactId}}"},
    "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 202}]
  }]
}
```

---

## Deal CRUD Operations

### Create a deal

```
POST https://api.hubapi.com/crm/v3/objects/deals
```

```json
{
  "properties": {
    "dealname": "{{contact.firstname}} {{contact.lastname}} - Inquiry",
    "dealstage": "{{stageId}}",
    "pipeline": "{{pipelineId}}",
    "amount": "5000",
    "closedate": "2026-06-30T00:00:00Z",
    "hubspot_owner_id": "{{ownerId}}"
  },
  "associations": [
    {
      "to": {"id": "{{contactId}}"},
      "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 3}]
    }
  ]
}
```

`dealstage` must be the internal stage ID string, not the label. Fetch your pipeline stages first (see below).

### Read a deal

```
GET https://api.hubapi.com/crm/v3/objects/deals/{{dealId}}?properties=dealname,dealstage,amount,closedate,hubspot_owner_id,deal_priority
```

Add as many `properties=` params as needed, comma-separated or with multiple `&properties=` params.

### Update a deal

```
PATCH https://api.hubapi.com/crm/v3/objects/deals/{{dealId}}
```

```json
{
  "properties": {
    "dealstage": "{{newStageId}}",
    "last_stage_change_date": "2026-03-29",
    "close_reason": "Won: Product Fit"
  }
}
```

### Delete a deal

```
DELETE https://api.hubapi.com/crm/v3/objects/deals/{{dealId}}
```

This moves the deal to the "Recycling Bin" — it's not permanently deleted immediately. Restore from Settings -> Recycling Bin within 90 days.

### Get all pipeline stage IDs

You need stage IDs to set `dealstage` correctly. Fetch them once and store them.

```
GET https://api.hubapi.com/crm/v3/pipelines/deals
```

Response structure:
```json
{
  "results": [
    {
      "id": "default",
      "label": "Sales Pipeline",
      "stages": [
        {"id": "appointmentscheduled", "label": "New Lead"},
        {"id": "qualifiedtobuy", "label": "Contacted"},
        {"id": "closedwon", "label": "Closed Won"}
      ]
    }
  ]
}
```

In n8n, run this once in a Code node and save the stage IDs as variables or in a lookup table.

---

## Company API Patterns

### Create a company

```
POST https://api.hubapi.com/crm/v3/objects/companies
```

```json
{
  "properties": {
    "name": "Acme Corp",
    "domain": "acme.com",
    "industry": "Technology",
    "numberofemployees": "250",
    "annualrevenue": "5000000",
    "city": "Austin",
    "state": "TX",
    "country": "United States",
    "company_size_band": "201-500"
  }
}
```

HubSpot will auto-enrich company records if you provide a domain. It may fill in logo, description, and industry automatically.

### Search companies by domain

```
POST https://api.hubapi.com/crm/v3/objects/companies/search
```

```json
{
  "filterGroups": [{
    "filters": [{
      "propertyName": "domain",
      "operator": "EQ",
      "value": "acme.com"
    }]
  }],
  "properties": ["name", "domain", "numberofemployees", "hubspot_owner_id"],
  "limit": 1
}
```

Use this before creating a company to avoid duplicates. If the search returns a result, update the existing company. If it returns nothing, create new.

### Update a company

```
PATCH https://api.hubapi.com/crm/v3/objects/companies/{{companyId}}
```

```json
{
  "properties": {
    "account_tier": "Strategic",
    "nps_score": "8",
    "contract_renewal_date": "2027-01-01"
  }
}
```

---

## Association API (Linking Contacts, Deals, Companies)

Associations are the relationships between objects. Always create them after the objects exist.

### Associate a contact to a company

```
PUT https://api.hubapi.com/crm/v3/objects/contacts/{{contactId}}/associations/companies/{{companyId}}/contact_to_company
```

No request body needed. This is an upsert — calling it multiple times is safe.

### Associate a deal to a contact

```
PUT https://api.hubapi.com/crm/v3/objects/deals/{{dealId}}/associations/contacts/{{contactId}}/deal_to_contact
```

### Associate a deal to a company

```
PUT https://api.hubapi.com/crm/v3/objects/deals/{{dealId}}/associations/companies/{{companyId}}/deal_to_company
```

### Get all associations for a contact (find their deals and companies)

```
GET https://api.hubapi.com/crm/v3/objects/contacts/{{contactId}}/associations/deals
```

```
GET https://api.hubapi.com/crm/v3/objects/contacts/{{contactId}}/associations/companies
```

Response includes an array of associated object IDs. You'll need to then fetch each deal/company individually if you need their property values.

### Batch create associations

For bulk workflows, use v4:

```
POST https://api.hubapi.com/crm/v4/associations/contacts/deals/batch/create
```

```json
{
  "inputs": [
    {
      "from": {"id": "{{contactId1}}"},
      "to": {"id": "{{dealId1}}"},
      "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 4}]
    },
    {
      "from": {"id": "{{contactId2}}"},
      "to": {"id": "{{dealId2}}"},
      "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 4}]
    }
  ]
}
```

Association type IDs:
- Contact → Deal: `4` | Deal → Contact: `3`
- Contact → Company: `1` | Company → Contact: `2`
- Deal → Company: `5` | Company → Deal: `6`
- Note → Contact: `202` | Note → Deal: `214`
- Task → Contact: `204` | Task → Deal: `216`

---

## Creating Tasks via API

Tasks are the to-do items that appear in the Sales -> Tasks queue and on contact/deal timelines.

```
POST https://api.hubapi.com/crm/v3/objects/tasks
```

```json
{
  "properties": {
    "hs_task_subject": "Follow up with {{contact.firstname}} re: proposal",
    "hs_task_body": "They viewed the pricing page twice. Mention the Q2 discount.",
    "hs_timestamp": "{{dueDateMs}}",
    "hs_task_priority": "HIGH",
    "hs_task_type": "CALL",
    "hubspot_owner_id": "{{ownerId}}"
  },
  "associations": [
    {
      "to": {"id": "{{contactId}}"},
      "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 204}]
    },
    {
      "to": {"id": "{{dealId}}"},
      "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 216}]
    }
  ]
}
```

Field notes:
- `hs_timestamp`: due date in milliseconds (Unix timestamp * 1000). In n8n Code node: `new Date('2026-04-01').getTime()`
- `hs_task_type`: `CALL`, `EMAIL`, or `TODO`
- `hs_task_priority`: `LOW`, `MEDIUM`, or `HIGH`
- `hubspot_owner_id`: the HubSpot user ID of the person this task is assigned to

In n8n, calculate due date: `Date.now() + (3 * 24 * 60 * 60 * 1000)` for "3 days from now".

---

## Creating Meeting and Call Activities

Meetings and calls are logged as engagement objects on the timeline.

### Log a completed call

```
POST https://api.hubapi.com/crm/v3/objects/calls
```

```json
{
  "properties": {
    "hs_call_title": "Discovery call",
    "hs_call_direction": "OUTBOUND",
    "hs_call_disposition": "CONNECTED",
    "hs_call_duration": "900000",
    "hs_call_body": "Discussed pricing and timeline. They need to check with their CFO. Following up Friday.",
    "hs_call_status": "COMPLETED",
    "hs_timestamp": "{{timestampMs}}",
    "hubspot_owner_id": "{{ownerId}}"
  },
  "associations": [
    {
      "to": {"id": "{{contactId}}"},
      "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 194}]
    },
    {
      "to": {"id": "{{dealId}}"},
      "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 206}]
    }
  ]
}
```

`hs_call_duration` is in milliseconds. 15 minutes = 900000.

`hs_call_disposition` options: `CONNECTED`, `LEFT_LIVE_MESSAGE`, `LEFT_VOICEMAIL`, `NO_ANSWER`, `BUSY`, `WRONG_NUMBER`

### Log a meeting

```
POST https://api.hubapi.com/crm/v3/objects/meetings
```

```json
{
  "properties": {
    "hs_meeting_title": "Product demo - Acme Corp",
    "hs_meeting_body": "Showed core features. They had questions about integrations.",
    "hs_meeting_start_time": "{{startTimeMs}}",
    "hs_meeting_end_time": "{{endTimeMs}}",
    "hs_meeting_outcome": "COMPLETED",
    "hs_timestamp": "{{startTimeMs}}",
    "hubspot_owner_id": "{{ownerId}}"
  },
  "associations": [
    {
      "to": {"id": "{{contactId}}"},
      "types": [{"associationCategory": "HUBSPOT_DEFINED", "associationTypeId": 200}]
    }
  ]
}
```

`hs_meeting_outcome` options: `SCHEDULED`, `COMPLETED`, `NO_SHOW`, `CANCELLED`

---

## Searching Deals with Complex Filters

The search endpoint supports multiple filter groups (OR logic between groups) and multiple filters within a group (AND logic within a group).

### Find all open deals assigned to a rep with no activity in 7 days

```
POST https://api.hubapi.com/crm/v3/objects/deals/search
```

```json
{
  "filterGroups": [
    {
      "filters": [
        {
          "propertyName": "dealstage",
          "operator": "NOT_IN",
          "values": ["closedwon", "closedlost"]
        },
        {
          "propertyName": "hubspot_owner_id",
          "operator": "EQ",
          "value": "{{ownerId}}"
        },
        {
          "propertyName": "hs_last_activity_date",
          "operator": "LT",
          "value": "{{sevenDaysAgoMs}}"
        }
      ]
    }
  ],
  "properties": ["dealname", "dealstage", "amount", "hubspot_owner_id", "hs_last_activity_date"],
  "sorts": [{"propertyName": "hs_last_activity_date", "direction": "ASCENDING"}],
  "limit": 100
}
```

### Find deals closing this month above a value threshold

```json
{
  "filterGroups": [
    {
      "filters": [
        {
          "propertyName": "closedate",
          "operator": "BETWEEN",
          "value": "{{firstDayOfMonthMs}}",
          "highValue": "{{lastDayOfMonthMs}}"
        },
        {
          "propertyName": "amount",
          "operator": "GTE",
          "value": "5000"
        },
        {
          "propertyName": "dealstage",
          "operator": "NOT_IN",
          "values": ["closedwon", "closedlost"]
        }
      ]
    }
  ],
  "properties": ["dealname", "amount", "closedate", "dealstage"],
  "limit": 100
}
```

In n8n Code node, calculate date ranges:
```javascript
const now = new Date();
const firstDay = new Date(now.getFullYear(), now.getMonth(), 1).getTime();
const lastDay = new Date(now.getFullYear(), now.getMonth() + 1, 0, 23, 59, 59).getTime();
return [{ json: { firstDayOfMonthMs: firstDay.toString(), lastDayOfMonthMs: lastDay.toString() } }];
```

Filter operators reference:
- `EQ` — equals
- `NEQ` — not equals
- `GT` / `GTE` / `LT` / `LTE` — greater/less than
- `BETWEEN` — requires both `value` (low) and `highValue` (high)
- `IN` / `NOT_IN` — match against array in `values`
- `HAS_PROPERTY` — property exists and is not empty
- `NOT_HAS_PROPERTY` — property doesn't exist or is empty
- `CONTAINS_TOKEN` — for multi-select, checks if value is in the set

---

## Bulk Operations with Pagination Handling

The search endpoint returns a max of 100 records at a time. Use `after` cursors to paginate through large result sets.

### Pattern in n8n (using a Loop node)

Set up your n8n workflow:

1. **HTTP Request node** — runs the search with `limit: 100`
2. **Code node** — extracts the records and checks if `paging.next.after` exists
3. **IF node** — if `after` cursor exists, loop back to step 1 with the cursor; otherwise proceed
4. **Aggregate node** — collects all pages of results

```javascript
// Code node: extract results and next cursor
const response = $input.first().json;
const contacts = response.results || [];
const nextCursor = response.paging?.next?.after || null;

return [{
  json: {
    contacts: contacts,
    nextCursor: nextCursor,
    hasMore: nextCursor !== null
  }
}];
```

In the HTTP Request node for subsequent pages, add `"after": "{{nextCursor}}"` to the request body.

```json
{
  "filterGroups": [...],
  "properties": ["email", "ai_lead_score"],
  "limit": 100,
  "after": "{{nextCursor}}"
}
```

### Batch update with pagination

When you need to update all contacts from a search:
1. Page through all results, collecting contact IDs
2. Split into chunks of 100
3. Call `POST /crm/v3/objects/contacts/batch/update` with each chunk
4. Add a Wait node (0.5-1 second) between batch calls

---

## Webhook Setup (HubSpot → n8n)

Use webhooks to have HubSpot push events to n8n in real time instead of polling.

### Setting up in HubSpot

1. HubSpot: Settings -> Integrations -> Private Apps -> your app -> Webhooks tab
2. Click "Create subscription"
3. Object type: Contact (or Deal, Company)
4. Event: `contact.propertyChange` or `contact.creation`
5. Property to watch: e.g., `ai_lead_score` or `hs_lead_status`
6. Target URL: your n8n webhook URL

### Setting up in n8n

1. Add a "Webhook" trigger node
2. Set Method to POST
3. Copy the webhook URL and paste into HubSpot
4. The webhook will receive a payload like:

```json
[
  {
    "eventId": 1,
    "subscriptionId": 12345,
    "portalId": 67890,
    "occurredAt": 1711670400000,
    "subscriptionType": "contact.propertyChange",
    "attemptNumber": 0,
    "objectId": 101,
    "changeSource": "API",
    "propertyName": "ai_lead_score",
    "propertyValue": "9"
  }
]
```

Note: HubSpot sends an array even for a single event. In n8n, use a "Split Out" node on the array before processing individual events.

### Webhook event types

| Event | Trigger |
|-------|---------|
| `contact.creation` | New contact created |
| `contact.deletion` | Contact deleted |
| `contact.propertyChange` | Specific property changes |
| `deal.creation` | New deal created |
| `deal.deletion` | Deal deleted |
| `deal.propertyChange` | Deal property changes (use for stage changes) |
| `company.creation` | New company created |
| `company.propertyChange` | Company property changes |

To watch deal stage changes: subscribe to `deal.propertyChange` with property `dealstage`.

### Webhook security

HubSpot signs webhook payloads. Verify the signature in n8n to reject forged requests:

1. HubSpot sends `X-HubSpot-Signature` header
2. The signature is HMAC-SHA256 of `clientSecret + requestBody`
3. In n8n Code node:

```javascript
const crypto = require('crypto');
const clientSecret = 'your-private-app-client-secret';
const body = JSON.stringify($input.first().json);
const expected = crypto.createHmac('sha256', clientSecret).update(body).digest('hex');
const received = $input.first().headers['x-hubspot-signature'];

if (expected !== received) {
  throw new Error('Webhook signature verification failed');
}

return $input.all();
```

---

## Error Handling Patterns for HubSpot API

HubSpot returns structured errors. Handle these specifically.

### Common error codes

| Status | Meaning | Fix |
|--------|---------|-----|
| 400 | Bad request — invalid property value or missing required field | Check property name and value format |
| 401 | Invalid token | Regenerate private app token |
| 403 | Insufficient scopes | Add required scope to private app |
| 404 | Object not found | Contact/deal ID doesn't exist |
| 409 | Conflict — duplicate detected | Contact with this email already exists; search first |
| 429 | Rate limit exceeded | Back off and retry |

### Error response structure

```json
{
  "status": "error",
  "message": "Property \"nonexistent_property\" does not exist",
  "error": "PROPERTY_DOESNT_EXIST",
  "category": "VALIDATION_ERROR"
}
```

### n8n error handling setup

In every HTTP Request node calling HubSpot:
1. Open the node settings
2. Set "On Error" to "Continue (using error output)"
3. Connect the error output to an IF node
4. IF `statusCode` = 429 → route to wait-and-retry branch
5. IF `statusCode` = 404 → route to "skip this record" branch
6. IF `statusCode` = 409 → route to "search for existing and update" branch
7. Otherwise → log the error and alert

### 409 Conflict handling (duplicate contact)

When you try to create a contact and get a 409, extract the existing contact's ID from the error and update it instead:

```javascript
// HubSpot 409 response includes the existing object's ID
const errorBody = $input.first().json;
const existingId = errorBody.message.match(/existing ID: (\d+)/)?.[1];

if (existingId) {
  return [{ json: { contactId: existingId, action: 'update' } }];
}
throw new Error('409 but could not extract existing ID: ' + JSON.stringify(errorBody));
```

---

## Rate Limit Handling with Exponential Backoff

HubSpot rate limits:
- Free/Starter: 100 requests per 10 seconds
- Professional/Enterprise: 150 requests per 10 seconds
- Burst allowed: up to 5x for short periods, then hard cutoff

A 429 response includes a `Retry-After` header with the number of seconds to wait.

### Exponential backoff pattern in n8n

Build a subworkflow for rate-limited requests:

1. **Execute Workflow trigger** — receives `{ url, method, body, attempt }`
2. **HTTP Request** — makes the call
3. **IF node** — checks if response status is 429
4. **Code node** (on 429 path) — calculates wait time:

```javascript
const attempt = $input.first().json.attempt || 0;
const retryAfter = parseInt($input.first().headers?.['retry-after'] || '10');
const backoffSeconds = Math.max(retryAfter, Math.pow(2, attempt));
const maxAttempts = 5;

if (attempt >= maxAttempts) {
  throw new Error(`Rate limit exceeded after ${maxAttempts} attempts`);
}

return [{
  json: {
    waitSeconds: backoffSeconds,
    nextAttempt: attempt + 1,
    url: $('Execute Workflow trigger').first().json.url,
    method: $('Execute Workflow trigger').first().json.method,
    body: $('Execute Workflow trigger').first().json.body
  }
}];
```

5. **Wait node** — set to `{{waitSeconds}}` seconds (use expression)
6. **Execute Workflow** — recursively calls itself with `nextAttempt` incremented

Backoff schedule: attempt 0 → wait 10s, attempt 1 → wait 20s, attempt 2 → wait 40s, attempt 3 → wait 80s, attempt 4 → wait 160s.

### Proactive rate limit avoidance

For batch jobs processing many contacts:
- Process 100 records at a time (the batch endpoint limit)
- Add a 1-second Wait node between each batch call
- At 1 batch/second = 60 batches/minute = 6,000 contact updates/minute — well within limits

For sequential contact processing (one HTTP call per contact):
- Add a Wait node of 0.1 seconds between calls
- This caps at 100 calls/10 seconds = exactly the rate limit
- Use 0.15 seconds for safety margin

---

## Common Gotchas

**Everything is a string.** HubSpot returns all values as strings, even numbers and booleans. Always parse before comparing: `parseInt(score) >= 7`, not `score >= 7`.

**NOT_HAS_PROPERTY vs EQ false are different.** Contacts created before you added a checkbox property won't match `EQ false`. Use `NOT_HAS_PROPERTY` for new boolean flags.

**Rate limits.** Free tier: 100 requests per 10 seconds. Add a Wait node between batches when processing large contact lists.

**Always add hs_timestamp when creating notes.** Without it, notes sort incorrectly on the contact timeline.

**Deal stage IDs are not the stage labels.** "Proposal Sent" might have an internal ID of `qualifiedtobuy`. Fetch your pipeline stages first and build a lookup map.

**Search returns a max of 100 records.** Always check `paging.next.after` and paginate for any workflow that might process more than 100 records.

**Association type IDs differ by direction.** Contact → Deal uses `4`, Deal → Contact uses `3`. Using the wrong one creates a broken association.

**Webhook arrays.** HubSpot always sends webhook payloads as an array. Use a Split Out node in n8n before processing.

---

## Microsoft 365 SMTP AUTH

If you're connecting Outlook 365 to send email via n8n or HubSpot, Microsoft 365 disables SMTP AUTH by default. It needs a two-step enable process.

**Step 1: Enable at tenant level**
1. Go to Exchange Admin Center: `admin.exchange.microsoft.com`
2. Settings -> Mail flow
3. Uncheck "Turn off SMTP AUTH protocol for your organization" and save

The checkbox says "Turn off" so it's checked by default (AUTH is off). Unchecking it turns AUTH on.

**Step 2: Enable per mailbox**
1. Microsoft 365 Admin Center -> Users -> Active users -> select the user
2. Mail tab -> Manage email apps
3. Check "Authenticated SMTP" and save

Wait 1-5 minutes for propagation.

**n8n SMTP credential settings:**
- Host: `smtp.office365.com`
- Port: `587`
- Secure: `false` (uses STARTTLS)
- User: the mailbox email address
- Password: account password
