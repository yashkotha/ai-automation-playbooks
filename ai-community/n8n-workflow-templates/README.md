# n8n Workflow Templates

Workflow architectures tested in production across real estate, SaaS, and service businesses. Described as patterns rather than raw JSON exports so you can implement them in your own n8n instance without credential conflicts.

---

## Template 1: AI Lead Qualifier

**Use case:** Score and tag new leads automatically as they come in from any source (form, ad, CRM import, webhook). Eliminate manual triage and route hot leads to sales instantly.

**Trigger:** Webhook (always-on). Leads arrive in real time from Typeform, Webflow, Meta Lead Ads, etc.

**Node sequence:**
1. Webhook trigger - receives lead data from form or CRM
2. Set node - normalize field names (map incoming fields to a consistent internal schema: `name`, `email`, `message`, `source`, `created_at`)
3. HTTP Request -> OpenAI - send qualification prompt with lead data
4. Code node - parse JSON response, extract `score`, `reason`, `tags`
5. IF node - if score >= 7, route to high-priority path; else route to standard nurture
6. CRM node (HubSpot/Pipedrive/etc.) - update contact with score, tags, and qualification reason
7. Slack / Email - notify sales team for hot leads only (score >= 7)
8. (Optional) Trigger Template 2 (First Response Email) by writing to a queue or setting a flag

**Key configuration notes:**
- Use `gpt-4o-mini` for speed and cost efficiency on high-volume lead flows
- Prompt the AI to return structured JSON only (no preamble, no explanation)
- Add error handling: if the AI call fails (timeout, rate limit), set score = 0 and add tag `needs_manual_review`
- Set a maximum token limit of 200 on the AI response to prevent runaway costs
- Use a Set node before the AI call to truncate `message` to 500 characters if long-form submissions come in

**Common gotchas:**
- Meta Lead Ads sends a webhook verification challenge on first setup. Handle the GET request separately with a Respond to Webhook node that echoes back the `hub.challenge` parameter.
- Some CRMs deduplicate by email on create. Run a "search contact first" step before create to avoid duplicate records.
- If your form allows blank messages, add an IF node to skip the AI call and default to score = 5.

**Webhook vs poll:** Use webhook here. Lead qualification is time-sensitive. A 10-minute polling delay on a hot inbound lead loses deals.

---

## Template 2: AI First-Response Email (Poll-Based)

**Use case:** Send a personalized AI-written email within 10 minutes of any new form submission. Works for any business where speed-to-lead matters.

**Trigger:** Schedule (every 10 minutes). More reliable than webhook for email sends.

**Node sequence:**
1. Schedule trigger (every 10 min)
2. CRM search - contacts where `first_response_sent` is not set (or is false)
3. IF node - exit if no results (prevents empty loop execution)
4. Loop Over Items
5. HTTP Request -> OpenAI - generate personalized email using lead's name and original message
6. HTTP Request -> Resend/SendGrid/Postmark - send email
7. CRM update - set `first_response_sent = true` + timestamp
8. CRM note - log the email body and send timestamp for reference

**Why poll-based over webhook:**
Webhooks can fire before the CRM contact is fully created. Polling every 10 minutes is more reliable, easier to debug, and handles edge cases (e.g., a lead created manually by a rep) automatically. A 10-minute response time is still very fast relative to human response rates.

**Key configuration notes:**
- Set the flag AFTER successful send, never before. If the send fails, you want the workflow to retry on the next run.
- Log every send as a CRM note with timestamp so reps can see what the lead received
- Add a `max_retries` guard: add a `response_attempts` counter to the contact and skip if it is >= 3
- Filter out contacts with `email_invalid = true` or `unsubscribed = true` in the CRM search step
- Batch the CRM search to return a maximum of 10-20 contacts per run to avoid long execution times

**Common gotchas:**
- If your email provider deduplicates by message-id, make sure you are not sending the same email twice. The `first_response_sent` flag prevents this, but add logging to verify.
- OpenAI rate limits: if you are processing 50+ leads per run, add a Wait node (1-2 seconds) inside the loop to avoid 429 errors.

**Webhook vs poll:** Poll intentionally. Webhook is faster but less reliable for this use case.

---

## Template 3: Multi-Touch Nurture Sequence (30 days)

**Use case:** Automated 8-email sequence over 30 days, each email AI-personalized to the contact based on their original message and current touch.

**Trigger:** Schedule (e.g., Tue + Fri, 9:30 AM local time).

**Architecture:**
- Runs on cron twice a week
- Tracks each touch with boolean flags on the CRM contact: `touch_day3_sent`, `touch_day7_sent`, etc.
- AI writes each email based on touch number, lead info, and original inquiry
- Sequence stops automatically when deal stage changes to Closed Won, Closed Lost, or lead unsubscribes

**Node sequence:**
1. Schedule trigger (Tue/Fri, 9:30 AM)
2. CRM search - active leads in nurture sequence where `sequence_complete = false` and `unsubscribed = false`
3. IF node - exit if no results
4. Loop Over Items
5. Code node - calculate which touch to send based on `contact_created_date` vs today
6. IF node - skip if this touch has already been sent (check flag)
7. HTTP Request -> OpenAI - generate email for this specific touch number and lead
8. HTTP Request -> email provider - send email
9. CRM update - set the touch flag for this touch number
10. CRM note - log touch number, subject, and timestamp
11. IF node - if touch 8 sent, set `sequence_complete = true` on contact

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

**Key configuration notes:**
- Calculate which touch to send based on `contact_created_date` vs today
- Never send if `unsubscribed = true`, `deal_stage = closed_won`, or `deal_stage = closed_lost`
- Use different CTAs per touch, not the same link every time
- Pass the full touch context to the AI (touch number, angle, original message) so it can match tone

**Common gotchas:**
- If your workflow runs twice on the same day (e.g., a manual re-run after a fix), the flag check in step 6 prevents double sends.
- Day calculation: use UTC timestamps consistently. Mixing UTC and local time causes sends to happen a day early or late.
- Some CRMs limit how many boolean properties you can have. Alternative: store the last touch number as a single integer field and compare to the expected touch for today.

**Webhook vs poll:** Poll. Nurture sequences are batch operations, not real-time triggers.

---

## Template 4: Email Open/Click Tracking

**Use case:** Track email engagement and write it back to your CRM in real time. Use engagement data to prioritize follow-up.

**Trigger:** Webhook (always-on). Email providers push events to your endpoint instantly.

**Node sequence:**
1. Webhook (always-on) - receives events from Resend, SendGrid, or Postmark
2. Code node - verify webhook signature (Svix / HMAC validation using the signing secret)
3. Switch node - route by event type: `opened`, `clicked`, `bounced`, `complained`, `unsubscribed`
4. CRM update - update tracking properties on the contact (open count, click count, last event timestamp)
5. CRM note - log event with full detail (link clicked, timestamp, IP if available)
6. (On bounce/complaint) - set `email_invalid = true` or `unsubscribed = true` and suppress future sends

**Key configuration notes:**
- Always verify webhook signatures before processing. Reject unverified events with a 401 response.
- Build idempotency: email providers sometimes send duplicate events. Use the event ID as a deduplication key and check against a lookup table (Airtable, Redis, or a Google Sheet works fine at low volume).
- Track at minimum: `open_count`, `click_count`, `last_opened_at`, `last_clicked_at`, `bounced`, `complained`
- Soft bounces (mailbox full) are different from hard bounces (address invalid). Track them separately.

**Common gotchas:**
- Apple Mail Privacy Protection pre-fetches pixels, causing false open events. Filter out known Apple MPP user agents if open rates seem inflated.
- SendGrid sends events as arrays, not individual objects. Loop over the array in your Code node before routing.

**Webhook vs poll:** Webhook only. Email events are time-sensitive and high-volume. Polling would miss events or arrive too late to be actionable.

---

## Template 5: Stale Deal Alert

**Use case:** Alert the sales team when deals have had no activity in 14+ days. Prevent deals from dying silently in the pipeline.

**Trigger:** Schedule (daily, 8 AM).

**Node sequence:**
1. Schedule trigger (daily, 8 AM)
2. CRM search - open deals where `last_activity_date` is older than 14 days (adjust threshold per your sales cycle)
3. IF node - exit if no results
4. Loop Over Items
5. HTTP Request -> OpenAI (optional) - generate a brief AI note about why this deal might be stalling based on its stage and history
6. Slack message to deal owner - include deal name, company, stage, value, days since last activity, and AI note
7. Email to deal owner (optional backup if they are not active on Slack)
8. CRM note - log that the stale alert was sent, with timestamp

**Key configuration notes:**
- Route the alert to the deal owner, not a generic channel. Use the CRM's owner email to look up their Slack user ID.
- Customize the stale threshold by deal stage. A deal in "Proposal Sent" going quiet for 7 days is more alarming than one in "Initial Contact" at 14 days.
- Do not alert on deals where the next scheduled activity is in the future (rep is already following up).

**Common gotchas:**
- CRM `last_activity_date` fields may not update when you log a CRM note via API. Test this. Some CRMs only update the field on manual activity, not API activity.
- Avoid alerting on weekends. Wrap the entire workflow in an IF node that checks `$today.weekday()` and exits on Saturday/Sunday.

**Webhook vs poll:** Poll (scheduled daily). This is a batch reporting job.

---

## Template 6: Weekly AI Pipeline Summary

**Use case:** Monday morning AI-written pipeline summary delivered to the whole team. Replaces the "can everyone update their deals" Slack message.

**Trigger:** Schedule (Monday, 8 AM).

**Node sequence:**
1. Schedule trigger (Monday, 8 AM)
2. CRM search - all open deals with full properties (stage, value, owner, last activity date, next step, close date)
3. Code node - format deal data into a clean JSON summary for the AI
4. HTTP Request -> OpenAI - write the pipeline summary
5. Send to Slack (channel post) or email to leadership/team
6. (Optional) CRM note on each stale deal flagged by the AI

**Prompt:** See [Prompt Library](../prompt-library/README.md)

**Key configuration notes:**
- Pull deals from the last 90 days only to avoid including ancient closed deals
- Include deal owner names in the summary so the AI can call out individual deal owners
- Use `gpt-4o` (not mini) here. Pipeline summaries are read by leadership; quality matters more than cost.

**Common gotchas:**
- CRM APIs often paginate. Make sure your CRM search node fetches all pages, not just the first 50 results.
- Some CRM fields return null for deals where the rep hasn't filled in next steps. The AI should be instructed to flag these explicitly.

**Webhook vs poll:** Poll (scheduled). This is a reporting job triggered on a fixed schedule.

---

## Template 7: Instagram / Form Lead Import

**Use case:** Auto-import leads from Instagram Lead Ads or any web form into your CRM. Ensure zero leads slip through by eliminating manual data entry.

**Trigger:** Webhook (for real-time imports from Meta, Typeform, Webflow). Alternatively, Schedule poll for platforms without webhook support.

**Node sequence:**
1. Webhook trigger - receives lead data from Meta, Typeform, Webflow, etc.
2. Set node - map incoming fields to CRM field names (`first_name`, `last_name`, `email`, `phone`, `source`, `form_name`)
3. CRM search - check if contact already exists by email (deduplication)
4. IF node - if exists, update; if not, create
5. CRM create / update contact
6. CRM note - log source, form name, and raw submission data
7. Trigger Lead Qualifier workflow (Template 1) by calling its webhook URL

**For Meta Lead Ads:**
- Set up the webhook in Meta Business Suite under Lead Ads forms
- Add your n8n webhook URL in the form settings
- Handle the Meta webhook verification challenge on first setup: n8n's Respond to Webhook node should return `hub.challenge` when `hub.mode = subscribe`
- Meta sends lead data in a nested format. Use a Code node to extract `field_data` array and map to flat key-value pairs.

**Common gotchas:**
- Typeform sends field answers as an array with `field.ref` as the key. Write a Code node to map these to named fields rather than relying on array position, which changes if you reorder form questions.
- Webflow form submissions do not include field labels by default. Map by field ID, not label.

**Webhook vs poll:** Webhook for Meta, Typeform, Webflow. Use poll (every 15 min) for Google Forms or platforms that lack webhook support (via Zapier Tables, Airtable, or a Google Sheet intermediary).

---

## Template 8: AI Meeting Scheduler / Calendar Booking Confirmation

**Use case:** When a prospect books a meeting via Calendly, Cal.com, or any booking tool, automatically send a confirmation email, create a CRM deal or update the contact, brief the sales rep in Slack, and create a meeting prep note.

**Trigger:** Webhook (booking confirmation event from Calendly or Cal.com).

**Node sequence:**
1. Webhook trigger - receives booking created event from Calendly or Cal.com
2. Set node - extract: invitee name, invitee email, meeting start time, meeting type, answers to intake questions, booking URL
3. HTTP Request -> OpenAI - generate personalized confirmation email referencing the meeting type and any intake questions the prospect answered
4. Email node (Resend/SendGrid) - send AI-written confirmation to the prospect
5. CRM search - find or create contact by email
6. CRM update - log meeting as an activity, set `meeting_booked = true`, update lifecycle stage if appropriate
7. HTTP Request -> OpenAI - generate meeting prep brief for the sales rep (based on CRM data + intake answers)
8. Slack message to sales rep - meeting details, prospect background from CRM, prep brief, direct link to CRM record
9. Google Calendar / Outlook event update (optional) - add prep notes to the calendar event description
10. (Optional) Schedule a reminder workflow: 1 hour before meeting, send rep a final reminder with the brief

**Key configuration notes:**
- Calendly webhook events include custom intake question answers. Pull these into the confirmation email and prep brief.
- If the contact already exists in your CRM (returning prospect), pull their history for context in the prep brief.
- Use the meeting type (e.g., "Discovery Call" vs "Demo") to adjust the tone and content of the confirmation email.
- For Cal.com: use the `BOOKING_CREATED` event type. Cal.com also supports `BOOKING_RESCHEDULED` and `BOOKING_CANCELLED` — handle all three.

**Common gotchas:**
- Calendly sends a test event when you first configure the webhook. Add a guard that checks for `event = invitee.created` before processing.
- Time zones: Calendly returns times in UTC. Convert to the prospect's local time zone (included in the payload as `invitee_timezone`) before displaying in the confirmation email.
- If you cancel/reschedule handling, make sure to update the CRM and re-send to the rep. Add a `booking_status` field on the CRM contact.

**Webhook vs poll:** Webhook only. Meeting confirmations are time-sensitive. A prospect expects an instant confirmation.

---

## Template 9: Airtable / HubSpot Two-Way Sync

**Use case:** Keep Airtable (used by operations, marketing, or CS teams) in sync with HubSpot (used by sales). Changes in either system flow to the other within minutes. Eliminates "which system is right?" confusion.

**Trigger:** Two separate workflows, each polling one system on a schedule (every 5-10 minutes).

**Workflow A: HubSpot -> Airtable**

Node sequence:
1. Schedule trigger (every 5 min)
2. HubSpot search - contacts modified since `last_sync_at` timestamp (store this in a reference record in Airtable or a global variable)
3. IF node - exit if no results
4. Loop Over Items
5. Airtable search - find matching record by HubSpot contact ID or email
6. IF node - if record exists, update; if not, create
7. Airtable update/create - write HubSpot fields to Airtable columns
8. Update `last_sync_at` timestamp (in Airtable config record or n8n's static data)

**Workflow B: Airtable -> HubSpot**

Node sequence:
1. Schedule trigger (every 5 min)
2. Airtable search - records where `last_modified` > `last_sync_at` AND `source != "hubspot"` (prevent sync loop)
3. IF node - exit if no results
4. Loop Over Items
5. HubSpot search - find contact by stored HubSpot ID or email
6. IF node - if contact exists, update; if not, create
7. HubSpot update/create - write Airtable fields to HubSpot properties
8. Airtable update - write back the HubSpot contact ID to the Airtable record (for future lookups)
9. Update `last_sync_at` timestamp

**Key configuration notes:**
- The most critical part of a two-way sync is preventing infinite loops. Use a `source` field (or `last_modified_by`) to mark which system last wrote the record. Each workflow should skip records it last wrote.
- Map fields explicitly. Do not try to auto-map by name. Airtable and HubSpot use completely different field naming conventions.
- Decide on a "source of truth" for conflicts (e.g., HubSpot always wins for deal stage; Airtable always wins for project status).
- Store the sync timestamp in Airtable so it persists across n8n restarts. Storing in n8n static data risks being lost on server restart.

**Common gotchas:**
- Airtable's `last_modified` field requires you to enable "Last modified time" as a field type on each base. It does not exist by default.
- HubSpot's `hs_lastmodifieddate` is in milliseconds (Unix timestamp). Airtable returns ISO 8601. Convert both to a consistent format in your Code node before comparing.
- Rate limits: Airtable allows 5 requests/second. HubSpot allows 100 requests/10 seconds. Add a Wait node if syncing large batches.
- Deleted records: HubSpot marks contacts as archived rather than deleting them. Handle archived contacts by checking `archived = true` in the response and archiving the Airtable record accordingly.

**Webhook vs poll:** Poll for both sides. Airtable webhooks require a paid plan and have reliability issues at the time of writing. HubSpot webhooks are available but add complexity for a two-way sync (you'd need to track which changes came from the sync vs. a human).

---

## Template 10: AI Content Repurposing (Blog Post -> LinkedIn + Twitter + Email Newsletter)

**Use case:** Turn one long-form blog post into four pieces of platform-native content automatically. Doubles or triples content output without additional writing time.

**Trigger:** Webhook (triggered when a new blog post is published in your CMS), or manual trigger with a URL/text input.

**Node sequence:**
1. Webhook trigger - receives new post event from WordPress, Ghost, Webflow, or Contentful (or Manual trigger with post URL)
2. HTTP Request - fetch the full post HTML/markdown from the CMS API (or extract text from the URL using an HTML parser)
3. Code node - strip HTML tags, clean up the text, extract title, meta description, key headings
4. HTTP Request -> OpenAI (call 1) - extract the 5 core insights from the post as JSON
5. HTTP Request -> OpenAI (call 2) - write LinkedIn article post (thought leadership angle, 150-250 words)
6. HTTP Request -> OpenAI (call 3) - write 5 Twitter/X posts as a thread (each under 280 characters)
7. HTTP Request -> OpenAI (call 4) - write email newsletter section (150-200 words, with link back to post)
8. (Optional) HTTP Request -> OpenAI (call 5) - write a short-form version for Instagram caption (under 150 words + hashtags)
9. Airtable / Notion create record - save all four variants with the original post title, URL, and date for the content calendar
10. Slack notification - send all four variants to the content/marketing Slack channel for review before publishing
11. (Optional) Post directly to LinkedIn via LinkedIn API if auto-publish is approved

**Key configuration notes:**
- Run each platform as a separate AI call, not one mega-prompt. Each platform has different tone, length, and structural requirements.
- Include the blog post URL in every variant so the AI can include it as a CTA where appropriate.
- LinkedIn performs best with a hook as the first line (before the "...see more" cutoff at ~150 characters). Instruct the AI explicitly.
- Twitter threads need each tweet to be self-contained and numbered (1/5, 2/5, etc.).
- Email newsletters need a clear subject line suggestion returned alongside the body.

**Common gotchas:**
- WordPress REST API requires authentication for private/draft posts. Use Application Passwords (wp-admin > Users > Application Passwords).
- Ghost webhooks fire for both published and draft saves. Add a guard: only process when `post.current.status = "published"`.
- OpenAI context limits: very long posts (5000+ words) may need to be chunked. Summarize to key points first (call 1), then generate variants from the summary, not the full text.
- Webflow CMS webhooks require a paid site plan.

**Webhook vs poll:** Webhook for CMS events. Poll (daily, check for new posts) as a fallback if your CMS does not support webhooks.

---

## Template 11: Slack -> CRM Deal Note Logger

**Use case:** When a sales rep mentions a deal name or uses a specific Slack command in a designated channel, automatically log that message as a note on the correct HubSpot deal. Keeps the CRM current without making reps manually copy-paste.

**Trigger:** Webhook (Slack Events API, listening for `message` events in specific channels).

**Node sequence:**
1. Webhook trigger - Slack Events API sends message events to your endpoint
2. Code node - Slack verification: respond to `url_verification` challenge on setup
3. Code node - extract message text, channel ID, user ID, timestamp
4. IF node - filter: only process messages from designated channels (e.g., `#deals`, `#sales`) or messages starting with `/crm` slash command
5. HTTP Request -> OpenAI - extract: deal name (or company name) mentioned in the message, note content to log, any action items mentioned
6. HTTP Request -> HubSpot search - find the deal by name or company name extracted by AI
7. IF node - if deal found (> 0 results), continue; else post a Slack reply: "Could not find a deal matching [name]. Use the full company name."
8. HTTP Request -> HubSpot - create note on the deal with: original message, logged by (Slack user name), timestamp, channel
9. Slack reply (ephemeral) - confirm to the rep: "Logged to [Deal Name] in HubSpot"

**Key configuration notes:**
- Set up the Slack app with `message.channels` and `message.groups` event subscriptions. Use a bot token with `chat:write` and `channels:history` scopes.
- Respond to Slack's HTTP request within 3 seconds or Slack will retry and you will get duplicate notes. Use n8n's "Respond to Webhook" node immediately, then continue processing asynchronously.
- The AI deal-name extraction should return JSON: `{"deal_name": "...", "note_content": "...", "action_items": [...]}`. Validate the JSON before proceeding.
- Filter out bot messages and your own app's messages using the `bot_id` field in the Slack payload.

**Common gotchas:**
- Slack retries failed webhook deliveries 3 times. If your n8n workflow takes > 3 seconds, Slack will retry, causing duplicate notes. Always respond immediately and process asynchronously.
- HubSpot deal search by name is fuzzy. If multiple deals match, return a Slack message listing all matches and ask the rep to confirm which one.
- Some reps will mention a company name that is spelled differently in HubSpot (e.g., "Acme Corp" vs "Acme Corporation"). Use the AI to suggest the closest match, not an exact string match.

**Webhook vs poll:** Webhook only. The entire point is real-time logging as conversations happen.

---

## Template 12: AI Proposal Generator

**Use case:** Generate a customized PDF proposal for a prospect by pulling deal data from HubSpot, running it through AI to write the proposal sections, assembling a PDF, and emailing it to the prospect and internal team.

**Trigger:** Manual trigger (rep clicks a button in HubSpot or a Slack command), or automatic trigger when deal reaches a specific stage (e.g., "Proposal Requested").

**Node sequence:**
1. Webhook trigger (from HubSpot workflow, Slack command, or manual trigger with deal ID)
2. HTTP Request -> HubSpot - fetch full deal data: deal name, value, stage, company, contact, associated notes, products
3. HTTP Request -> HubSpot - fetch associated company data: industry, size, website, description
4. HTTP Request -> OpenAI (call 1) - write executive summary section (2-3 paragraphs tailored to their industry and stated needs)
5. HTTP Request -> OpenAI (call 2) - write "Our Approach" section (methodology, timeline, deliverables)
6. HTTP Request -> OpenAI (call 3) - write "Investment" section (based on deal value, payment terms, what is included)
7. HTTP Request -> OpenAI (call 4) - write "Why Us" section (pull from a static context block about your company's differentiators)
8. Code node - assemble all sections into a structured JSON object
9. HTTP Request -> PDF generation API (e.g., Doppio, PDFShift, or a custom HTML template endpoint) - generate PDF from template + data
10. HTTP Request -> Resend/SendGrid - email PDF to prospect and BCC the deal owner
11. HTTP Request -> HubSpot - attach PDF URL to the deal, log an activity note, update deal stage to "Proposal Sent"
12. Slack notification to deal owner - "Proposal sent to [Prospect Name]. Deal link: [URL]"

**Key configuration notes:**
- Keep a static "company context" block (your pitch, differentiators, case studies) that gets injected into every AI call. Store this in n8n credentials or a Google Doc fetched at runtime.
- PDF generation: Doppio and PDFShift both accept HTML and return a PDF URL. Build your proposal template in HTML/CSS, then inject the AI-written content into placeholder `{{sections}}`.
- Include a version number and generation timestamp in the proposal footer.
- For multi-product deals, loop through the HubSpot line items and include them in the investment section.

**Common gotchas:**
- AI-generated proposals can sound generic if the deal notes are sparse. If `crm_notes` is empty, flag the proposal as needing review before sending.
- PDF generation APIs have a short timeout (usually 30-60 seconds). For long proposals, use async PDF generation and poll for completion.
- Some email clients block PDF attachments from automated sends. Host the PDF on S3 or Google Drive and send a download link instead of an attachment.

**Webhook vs poll:** Webhook (triggered on demand or by CRM stage change). Not a scheduled job.

---

## Template 13: New Customer Onboarding Sequence (Triggered on Closed Won)

**Use case:** The moment a deal is marked Closed Won in your CRM, automatically kick off the entire onboarding sequence: welcome email, internal team alert, project setup, and a structured multi-step email sequence to get the customer live.

**Trigger:** Webhook (HubSpot deal stage change webhook, or CRM workflow trigger).

**Node sequence:**
1. Webhook trigger - HubSpot sends event when deal stage changes to "Closed Won"
2. Set node - extract: contact name, contact email, company name, deal value, deal owner, product purchased
3. HTTP Request -> OpenAI - write personalized welcome email referencing what they purchased and why they chose you
4. Email node - send welcome email to customer
5. HTTP Request -> HubSpot - update contact lifecycle stage to "Customer", create onboarding task on the contact
6. Slack notification to team channel (#new-customers) - new customer announcement with deal details
7. Slack DM to account manager - their new account details, CRM link, suggested first 3 actions
8. HTTP Request -> project management tool (Asana, Notion, Linear, ClickUp) - create new client project from template
9. HTTP Request -> HubSpot - create a new "Onboarding" pipeline deal associated with the contact
10. Schedule: Day 3 onboarding email (educational, what to expect)
11. Schedule: Day 7 check-in email (any questions?)
12. Schedule: Day 14 milestone check email (are they getting value?)
13. Schedule: Day 30 success review invitation

**Scheduling the follow-up sequence:**
Use n8n's built-in ability to set a "wait" before the next node, or create separate scheduled workflows that poll for customers where `onboarding_day3_sent = false` and `customer_since_date` is 3 days ago (same polling pattern as Template 3).

**Key configuration notes:**
- The welcome email should be sent within 60 seconds of Closed Won, while the rep is still on the phone with the customer. Speed signals professionalism.
- Personalize the welcome email based on the product purchased. If you sell multiple products, use a Switch node to route to the correct email template for each.
- Create a Slack channel per customer for larger accounts (use Slack API `conversations.create`).

**Common gotchas:**
- HubSpot's deal stage change webhook fires for every stage change, not just to Closed Won. Add an IF node: `if new_stage = "closedwon"` before processing.
- If your deal is re-opened or stage changes back from Closed Won (it happens), add a guard to prevent duplicate onboarding sequences. Check `onboarding_started = true` on the contact before running.
- Project management tool templates: test that the "create from template" API call works and includes all subtasks. Some tools require an additional step to populate template fields.

**Webhook vs poll:** Webhook on deal stage change. Onboarding speed matters.

---

## Template 14: Inbound Call / Voicemail Transcription + CRM Logging

**Use case:** Every missed call or voicemail to your Twilio number is automatically transcribed using OpenAI Whisper, summarized by AI, and logged as a note on the matching CRM contact. Sales reps are notified instantly.

**Trigger:** Webhook (Twilio sends a webhook on call completion).

**Node sequence:**
1. Webhook trigger - Twilio sends `RecordingStatusCallback` or `StatusCallback` when a voicemail recording is ready
2. Set node - extract: caller phone number, call duration, recording URL, call SID, timestamp
3. HTTP Request - download the audio file from the Twilio recording URL (requires Twilio auth headers)
4. HTTP Request -> OpenAI Whisper API - transcribe the audio file (send as multipart/form-data)
5. HTTP Request -> OpenAI Chat - summarize the transcription: caller intent, urgency, any specific questions asked, suggested follow-up action
6. CRM search - find contact by phone number
7. IF node - if contact found, log note; if not found, create a new contact with the phone number and log note
8. CRM note - log: transcription, AI summary, caller phone, call timestamp, recording URL
9. Slack notification to sales channel - new voicemail alert with caller name (if found), summary, and direct CRM link
10. SMS reply via Twilio (optional) - send an automated acknowledgment SMS to the caller: "Got your voicemail, we'll call you back within [X hours]"

**Key configuration notes:**
- Twilio recordings are available at the URL within a few seconds of call completion. The `RecordingStatusCallback` fires when the recording is ready, making it the right trigger.
- OpenAI Whisper accepts MP3, MP4, WAV, and several other formats. Twilio recordings default to MP3. Pass the file as a Buffer in an HTTP Request node with `Content-Type: multipart/form-data`.
- Phone number matching: normalize all phone numbers to E.164 format (+1XXXXXXXXXX) before searching the CRM. Both Twilio and most CRMs can handle this format.
- For live call transcription (not just voicemail), use Twilio's Media Streams + a WebSocket connection. This is significantly more complex and requires a persistent WebSocket server.

**Common gotchas:**
- Twilio sends the `RecordingStatusCallback` before the recording is fully processed. Add a Wait node (3-5 seconds) before downloading the audio to avoid a 404.
- Large audio files (long voicemails, 2+ minutes) can exceed Whisper's payload limit (25MB). Twilio recordings are low bitrate, so this is rarely an issue for voicemails, but check for live calls.
- CRM phone number formats vary: some store as `+1XXXXXXXXXX`, others as `(XXX) XXX-XXXX`. Normalize before searching.
- GDPR / privacy: depending on jurisdiction, you may need to inform callers that voicemails are transcribed. Add a Twilio greeting: "Please leave a message. Voicemails may be transcribed for quality purposes."

**Webhook vs poll:** Webhook only. Twilio calls your endpoint in real time.

---

## Template 15: E-commerce Abandoned Cart Recovery (Shopify Webhook -> AI Email)

**Use case:** When a Shopify customer abandons their cart, send a personalized AI-written recovery email within 1 hour. If they do not recover after 3 touches, flag them for a discount offer.

**Trigger:** Webhook (Shopify `carts/update` or `checkouts/create` event, or use Shopify's native abandoned checkout webhook).

**Node sequence:**
1. Webhook trigger - Shopify sends `checkouts/update` event when a checkout is created or updated but not completed
2. Set node - extract: customer email, customer name, cart items (name, quantity, price), cart total, checkout URL, store name
3. Wait node - wait 1 hour (allow the customer time to complete the purchase naturally)
4. HTTP Request -> Shopify API - re-check: has the checkout been completed in the last hour? If yes, exit workflow.
5. IF node - if checkout still abandoned, continue; if recovered, exit
6. HTTP Request -> OpenAI - write personalized recovery email referencing specific products in their cart
7. Email node - send recovery email (Touch 1) with the checkout URL as CTA
8. Set flag `abandoned_cart_touch_1_sent = true` on the customer record (Shopify customer or your CRM)
9. Wait 24 hours
10. HTTP Request -> Shopify - re-check: still not purchased?
11. HTTP Request -> OpenAI - write Touch 2 email (different angle: "Your items are still available, but stock is limited")
12. Email node - send Touch 2
13. Wait 48 hours
14. HTTP Request -> Shopify - re-check: still not purchased?
15. HTTP Request -> OpenAI - write Touch 3 email with discount offer (generate a Shopify discount code via API)
16. HTTP Request -> Shopify - create a unique discount code for this customer
17. Email node - send Touch 3 with discount code

**Key configuration notes:**
- Shopify's native abandoned cart email and your custom workflow can conflict. Disable Shopify's built-in abandoned cart email if you are building your own.
- Use Shopify's `checkouts/create` webhook for the initial trigger, and then query the API to check for order completion. The API endpoint is `/admin/api/2024-01/orders.json?email=[email]&created_at_gte=[checkout_created_at]`.
- Personalize the email with the actual product names and images. Shopify's checkout payload includes line items with product titles and image URLs.
- Generate unique discount codes (not a shared promo code) so you can track attribution per abandoned cart.

**Common gotchas:**
- Shopify sends `checkouts/update` on every cart change (adding items, changing quantities). Debounce: only start the recovery sequence if the checkout has been idle for 1+ hour, not on every cart update.
- Guest checkouts (no account) may not have a customer ID. Use the email address as the primary identifier.
- If a customer has multiple abandoned checkouts, only the most recent one should be active. Cancel previous workflows for the same customer when a new checkout is created.
- Email frequency: if a customer abandoned a cart last week and abandoned again today, they may have already received 3 emails from the last sequence. Add a recency guard: check `last_abandoned_cart_email_sent_at` and skip if within 7 days.
- GDPR: make sure the customer has consented to marketing email (check Shopify's `email_marketing_consent` field on the customer record).

**Webhook vs poll:** Webhook for the initial cart event. The subsequent re-checks use the Shopify API (polling against the specific checkout) at fixed intervals within the same workflow execution.

---

## General Architecture Notes

### Webhook vs Poll Decision Framework

| Scenario | Use |
|----------|-----|
| Real-time user action (form submit, booking, payment) | Webhook |
| CRM syncs and batch operations | Poll (schedule) |
| Email event tracking (open, click, bounce) | Webhook |
| Reporting and summaries | Poll (schedule) |
| Pipeline alerts (stale deals, no activity) | Poll (schedule) |
| Onboarding triggers (deal closed) | Webhook |
| Nurture sequences | Poll (schedule) |

### Error Handling Patterns

Always implement these in every workflow:
1. **Try/Catch on AI calls** - AI APIs fail. Never let an OpenAI timeout kill an entire batch run.
2. **Dead letter queue** - Route failed items to a separate Airtable or Google Sheet for manual review.
3. **Idempotency flags** - Use CRM boolean fields or unique event IDs to prevent double-processing.
4. **Max retry guards** - Use a counter field to prevent infinite retry loops.
5. **Exit on empty** - Always add an IF node after any search/fetch step. Exit cleanly if no results.

### Cost Optimization

- Use `gpt-4o-mini` for high-volume, structured tasks (lead scoring, classification, short emails).
- Use `gpt-4o` for client-facing output (proposals, pipeline summaries, onboarding emails).
- Set `max_tokens` on every AI call to prevent runaway costs from unexpectedly long responses.
- Cache static context (your company description, product details) in n8n credentials, not in every prompt call.
