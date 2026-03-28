# n8n Workflow Templates

Workflow architectures tested in production across real estate, SaaS, and service businesses. Described as patterns rather than raw JSON exports so you can implement them in your own n8n instance without credential conflicts.

---

## Template 1: AI Lead Qualifier

**Use case:** Score and tag new leads automatically as they come in.

**Node sequence:**
1. Webhook trigger - receives lead data from form or CRM
2. Set node - normalize field names
3. HTTP Request -> OpenAI - send qualification prompt
4. Code node - parse JSON response, extract score and tags
5. IF node - if score >= 7, route to high-priority path
6. CRM node - update contact with score and tags
7. Slack / Email - notify sales team for hot leads

**Key config:**
- Use `gpt-4o-mini` for speed and cost efficiency
- Prompt the AI to return structured JSON only
- Add error handling: if AI call fails, set score = 0 and flag for manual review

---

## Template 2: AI First-Response Email (Poll-Based)

**Use case:** Send a personalized AI-written email within 10 minutes of any new form submission.

**Node sequence:**
1. Schedule trigger (every 10 min)
2. CRM search - contacts where `first_response_sent` is not set
3. IF node - exit if no results
4. Loop Over Items
5. HTTP Request -> OpenAI - generate personalized email
6. HTTP Request -> Resend - send email
7. CRM update - set `first_response_sent = true` + timestamp

**Why poll-based over webhook:**
Webhooks can fire before the CRM contact is fully created. Polling every 10 minutes is more reliable, easier to debug, and handles edge cases automatically.

**Key config:**
- Set the flag AFTER successful send, never before
- Log every send as a CRM note with timestamp
- Add a `max_retries` guard so failed sends don't loop forever

---

## Template 3: Multi-Touch Nurture Sequence (30 days)

**Use case:** Automated 8-email sequence over 30 days, each email AI-personalized to the contact.

**Architecture:**
- Runs on cron (e.g., Tue + Fri, 9:30 AM)
- Tracks each touch with boolean flags: `touch_day3_sent`, `touch_day7_sent`, etc.
- AI writes each email based on touch number, lead info, and original inquiry
- Sequence stops automatically when deal stage changes or lead unsubscribes

**Touch schedule:**

| Touch | Day | Angle |
|-------|-----|-------|
| 1 | 3 | Value reminder |
| 2 | 7 | Social proof |
| 3 | 10 | Answer a common objection |
| 4 | 14 | New information |
| 5 | 17 | Urgency (real, not manufactured) |
| 6 | 21 | Different angle, question-led |
| 7 | 24 | Direct ask |
| 8 | 28 | Breakup email |

**Key config:**
- Calculate which touch to send based on `contact_created_date` vs today
- Never send if `unsubscribed = true` or deal is closed
- Use different CTAs per touch, not the same link every time

---

## Template 4: Email Open/Click Tracking

**Use case:** Track email engagement and write it back to your CRM in real time.

**Node sequence:**
1. Webhook (always-on) - receives events from Resend, SendGrid, or Postmark
2. Code node - verify webhook signature (Svix / HMAC)
3. Switch node - route by event type: opened, clicked, bounced, complained
4. CRM update - update tracking properties on the contact
5. CRM note - log event on contact timeline

**Key config:**
- Always verify webhook signatures before processing
- Build idempotency: don't double-count the same event
- Track at minimum: open count, click count, last opened at, last clicked at, bounced flag

---

## Template 5: Stale Deal Alert

**Use case:** Alert the sales team when deals haven't had any activity in 14+ days.

**Node sequence:**
1. Schedule trigger (daily, 8 AM)
2. CRM search - open deals where `last_activity_date` is older than 14 days
3. IF node - exit if no results
4. Loop Over Items
5. Slack / Email - alert deal owner: deal name, stage, days since last activity
6. CRM note - log that the alert was sent

---

## Template 6: Weekly AI Pipeline Summary

**Use case:** Monday morning AI-written pipeline summary delivered to the whole team.

**Node sequence:**
1. Schedule trigger (Monday, 8 AM)
2. CRM search - all open deals with properties
3. HTTP Request -> OpenAI - summarize pipeline, flag stale deals, highlight top 3 priorities
4. Send Email / Slack - deliver the summary

**Prompt:** See [Prompt Library](../prompt-library/README.md)

---

## Template 7: Instagram / Form Lead Import

**Use case:** Auto-import leads from Instagram Lead Ads or any web form into your CRM.

**Node sequence:**
1. Webhook trigger - receives lead data from Meta, Typeform, Webflow, etc.
2. Set node - map incoming fields to CRM field names
3. CRM create / update contact
4. Trigger Lead Qualifier workflow (Template 1)

**For Meta Lead Ads:**
- Set up the webhook in Meta Business Suite under Lead Ads forms
- Add your n8n webhook URL in the form settings
- Handle the Meta webhook verification challenge (one-time setup, n8n has a built-in way to do this)
