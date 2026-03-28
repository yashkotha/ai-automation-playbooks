# AI Email Sequences

Email architecture for AI-personalized campaigns. Not mail merge. The AI reads each lead's original message and writes a unique email for them.

## Requirements

- n8n (self-hosted or cloud)
- OpenAI API key
- Resend account (or any transactional email provider)
- A CRM with API access (HubSpot, Pipedrive, Airtable, etc.)

---

## Email 1: First Response (within 10 min)

**Goal:** Acknowledge their inquiry, build trust fast, drive toward the next step.

**What to give the AI:**
- Lead name
- Their original message / form submission
- What your business does
- The CTA you want them to take

**Workflow:** See [n8n Template 2](../../ai-community/n8n-workflow-templates/README.md)

**Prompt:** See [Prompt Library - Personalized First-Response Email](../../ai-community/prompt-library/README.md)

---

## Emails 2-8: Multi-Touch Sequence

Each email has a different angle. Don't repeat yourself across touches.

| Touch | Day | Angle |
|-------|-----|-------|
| 1 | 3 | Value reminder |
| 2 | 7 | Social proof |
| 3 | 10 | Answer a common objection |
| 4 | 14 | New information or update |
| 5 | 17 | Real urgency, not fake scarcity |
| 6 | 21 | Question-led, flip the dynamic |
| 7 | 24 | Direct ask |
| 8 | 28 | Breakup email |

See [Multi-Touch Cadences](../multi-touch-cadences/) for the full implementation.

---

## Deliverability Checklist

- Send from a custom domain. Not Gmail or Outlook for bulk sends.
- Warm up new domains before sending volume. Start with 10/day, ramp over 2-4 weeks.
- Keep HTML simple. Fancy templates get flagged more than plain text.
- Text-to-HTML ratio: aim for at least 60% text.
- Always include a plain text version alongside HTML.
- Set up SPF, DKIM, and DMARC on your sending domain before sending anything.
- Include an unsubscribe link. Resend handles this automatically.

---

## Resend Setup

1. Add your sending domain in the Resend dashboard
2. Add the DNS records Resend provides (SPF, DKIM)
3. Create an API key
4. In n8n: HTTP Request node, POST to `https://api.resend.com/emails`

```json
{
  "from": "Your Name <you@yourdomain.com>",
  "to": ["{{email}}"],
  "subject": "{{subject}}",
  "html": "{{email_body_html}}",
  "text": "{{email_body_text}}",
  "reply_to": "you@yourdomain.com"
}
```

---

## Tracking Opens and Clicks

Use the [Email Tracking Webhook template](../../ai-community/n8n-workflow-templates/README.md) to write open and click events back to your CRM as they happen. This gives you a live behavioral layer on top of AI lead scoring.
