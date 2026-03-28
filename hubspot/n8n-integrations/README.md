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
- `timeline` (for logging notes and activities)

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

## Common Gotchas

**Everything is a string.** HubSpot returns all values as strings, even numbers and booleans. Always parse before comparing: `parseInt(score) >= 7`, not `score >= 7`.

**NOT_HAS_PROPERTY vs EQ false are different.** Contacts created before you added a checkbox property won't match `EQ false`. Use `NOT_HAS_PROPERTY` for new boolean flags.

**Rate limits.** Free tier: 100 requests per 10 seconds. Add a Wait node between batches when processing large contact lists.

**Always add hs_timestamp when creating notes.** Without it, notes sort incorrectly on the contact timeline.

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
