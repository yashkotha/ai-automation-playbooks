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

## How to Create

1. Settings -> Properties -> Contact properties
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
