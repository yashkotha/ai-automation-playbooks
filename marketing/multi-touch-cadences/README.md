# Multi-Touch Cadences

A 30-day, 8-touch nurture sequence. Each touch is AI-written with a different angle.

## Architecture

- Runs on a cron schedule (twice a week works well: Tue + Fri)
- Tracks each touch with boolean flags in the CRM
- Calculates which touch to send based on `contact_created_date` vs today
- Stops automatically on unsubscribe, reply, or deal closed

## Touch Map

| Touch | Day | Goal | CTA Type |
|-------|-----|------|----------|
| 1 | 3 | Remind them of the value | Soft (learn more) |
| 2 | 7 | Social proof | Soft (see results) |
| 3 | 10 | Handle the top objection | Soft (reassure) |
| 4 | 14 | Share something new or useful | Medium (read this) |
| 5 | 17 | Real urgency if applicable | Medium (act now) |
| 6 | 21 | Ask a question, flip the dynamic | Soft (reply) |
| 7 | 24 | Direct ask | Hard (book a call) |
| 8 | 28 | Breakup email | Soft (door stays open) |

## CRM Flags Required

Create these boolean properties in your CRM before building the workflow:

```
touch_day3_sent
touch_day7_sent
touch_day10_sent
touch_day14_sent
touch_day17_sent
touch_day21_sent
touch_day24_sent
touch_day28_sent
```

The workflow checks which flags are not set, calculates whether the contact is old enough for the next touch, and sends accordingly.

## Workflow Logic (n8n)

```
Schedule trigger (Tue + Fri, 9:30 AM)
  -> Search CRM: contacts created 1-35 days ago
  -> Loop over each contact
     -> Calculate days since created
     -> Determine which touch is next
     -> IF: flag not set AND contact is old enough for this touch
        -> Generate email with AI (pass touch number + lead info)
        -> Send via Resend
        -> Update CRM: set flag = true
        -> Log note on contact timeline
     -> ELSE: skip
```

## AI Prompt for the Sequence

Pass the touch number so the AI picks the right angle:

```
Write touch {{touch_number}} of 8 in a sales nurture sequence.

Lead name: {{name}}
Original inquiry: {{original_message}}
Touch angles: 1=value, 2=social proof, 3=objection handling, 4=new info, 5=urgency, 6=question, 7=direct ask, 8=breakup

Rules:
- Match the tone to the touch angle
- Reference their original inquiry where relevant
- Do not use em dashes
- Under 150 words
- Return email body only
```

## Stopping the Sequence

Add these checks before each send:
- `unsubscribed = true` -> skip and mark sequence complete
- Deal stage = Closed Won or Closed Lost -> stop
- `email_bounced = true` -> stop
- Contact replied to a previous email -> flag for manual follow-up, stop automation

## Results to Expect

With a well-written sequence and a qualified lead list:
- Open rates: 35-55% (well above industry average because the emails are personalized)
- Reply rates: 5-15% depending on industry and offer
- Unsubscribe rates: under 1% if the content is genuinely useful

The quality of the original lead source matters more than the sequence itself. A sequence sent to unqualified leads won't perform well regardless of how good the emails are.

---

## B2B vs B2C Cadence Differences

The same sequence architecture runs both, but the content, timing, and stopping conditions are different.

### B2B

- **Decision cycle:** Days to months. Longer sequences are appropriate.
- **Multiple stakeholders:** The person who replied may not be the decision maker. Touch 4-5 should offer assets useful for internal selling (one-pager, ROI calculator).
- **Right time to send:** Tuesday–Thursday, 8-10am or 1-3pm in the recipient's time zone. Avoid Monday mornings and Friday afternoons.
- **Subject lines:** Professional, specific to their role or industry. No emoji.
- **Tone:** Consultative, not urgent. The typical B2B sale requires trust before commitment.
- **Stopping condition:** Any reply (positive or negative) should stop automation. B2B replies, even objections, are worth a human response.
- **CTA ladder:** Soft (learn more) → Medium (case study) → Hard (schedule a call). Don't ask for a call before you've provided value.

### B2C

- **Decision cycle:** Hours to days. Shorter sequences with faster follow-up.
- **One decision maker:** Write for one person. No need to arm them for an internal sales process.
- **Right time to send:** Evening and weekend sends often outperform business hours. Test your own audience.
- **Subject lines:** Emotional, benefit-driven, can use emoji if it fits the brand.
- **Tone:** Conversational, direct, can create urgency more aggressively.
- **Stopping condition:** Purchase (connect your e-commerce platform to n8n). A purchase moves them out of the nurture sequence and into an onboarding sequence.
- **CTA ladder:** Soft (explore) → Direct (buy now). The conversion event is a transaction, not a conversation.

**Key B2C-specific touches:**
- Cart abandonment sequence (3 emails in 24 hours after abandonment)
- Browse abandonment (1 email when they view a product 3+ times)
- Price drop alert (triggered when a product they viewed drops in price)

---

## Short Cadences: 5-Day Fast Follow

For inbound leads that have high intent — they requested a demo, asked for pricing, or booked a call that didn't happen. Speed is more important than depth here.

**Who gets this:** Anyone who took a high-intent action in the last 5 days.

| Touch | Timing | Content |
|-------|--------|---------|
| 1 | Hour 0 | Immediate confirmation + single clear next step |
| 2 | Hour 24 | Brief value add, re-surface the CTA |
| 3 | Day 2 | One specific, relevant proof point |
| 4 | Day 3 | Objection handling: what usually stops people at this stage |
| 5 | Day 5 | Final ask. If no response, move to long cadence. |

**Tone shift:** These are short because high-intent leads shouldn't be waiting 28 days for a direct ask. They're ready now — meet them there.

**n8n trigger:** Webhook from your form/calendar tool fires the first email immediately. Subsequent emails are queued using a Wait node in n8n.

```
Webhook trigger (form submitted)
  -> Create contact in CRM
  -> Send touch 1 immediately
  -> Wait 24 hours
  -> Check: replied? purchased? → stop
  -> Send touch 2
  -> Wait 24 hours
  -> [continues for 5 days]
```

The Wait node in n8n handles the delay without a cron job. This is better for fast-follow because you don't have to wait for the next cron window.

---

## Long Cadences: 90-Day Enterprise

For high-ticket B2B deals where the sales cycle genuinely spans months. Trying to compress an enterprise sale into 28 days is counterproductive.

**Who gets this:**
- Deal size > $10,000 ACV
- First email indicated a multi-stakeholder decision process
- "Not now but maybe in Q2/Q3" reply

**Structure:**

| Phase | Days | Frequency | Approach |
|-------|------|-----------|---------|
| Active | 0-30 | Every 4-5 days | Standard 8-touch sequence |
| Nurture | 31-60 | Weekly | Educational content, industry news, no hard asks |
| Long-range | 61-90 | Bi-weekly | Check-in, new case study, seasonal/news hook |
| Re-entry | 91+ | Monthly | One email per month indefinitely until reply |

**Content for the nurture phase (days 31-60):**
- Industry benchmarks or reports you reference (not just "check out this link")
- A story from a similar client (anonymized or named with permission)
- A tool or resource they'd find useful — no strings
- One question that's relevant to what they told you in the original conversation

**Content for the long-range phase (days 61-90):**
- New feature or capability announcement
- Relevant news or event happening in their industry
- Direct check-in: "Is now a better time than [month they mentioned]?"

The long-range phase requires a human to review the AI-generated drafts before sending. At this volume and ticket size, a bad email is a meaningful risk.

---

## Seasonal and Event-Triggered Cadences

Some of your best sends have nothing to do with where a contact is in a nurture sequence. They're triggered by external events.

### Seasonal Cadences

**Q4 / End-of-year budget cadence:**
Trigger: October 1 every year, for all contacts tagged "budget pending" or in late-stage pipeline.

Many B2B buyers have use-it-or-lose-it budgets in Q4. A cadence specifically framing your product as a before-year-end purchase performs well between October and early December.

```
October 1:  "Q4 buying season — have you thought about locking in [result] before year end?"
October 15: Case study of a Q4 customer decision
November 1: Budget deadline framing
November 15: Final offer before most procurement freezes
```

**New Year fresh start:**
January cadence for contacts who went cold in Q4. "New year, new priorities" is a genuine re-engagement hook, not a fabricated urgency.

### Event-Triggered Cadences

**Trigger: Contact changes jobs (LinkedIn job change detected)**
When a past prospect moves to a new company, they often have fresh budget and fresh mandate. This is one of the highest-converting triggers in B2B.

```
Day 0: Congratulate on the new role (genuine, brief)
Day 7: Reintroduce what you do, relevant to their new company's profile
Day 14: Offer something specific to their new context
```

Detect job changes via LinkedIn Sales Navigator API, Clay, or by monitoring LinkedIn via a scraping tool connected to n8n.

**Trigger: Funding announcement**
A company that just raised a Series A or B is hiring, spending, and making new vendor decisions. Monitor Crunchbase or TechCrunch via RSS in n8n.

```
Day 0: Congratulate on the funding (brief, don't over-explain that you saw it)
Day 2: Connect their growth goals to what you offer
Day 7: Case study of another post-funding company you've worked with
```

**Trigger: Competitor shuts down or raises prices**
If a competitor announces pricing changes or discontinuation, their customers are in the market right now.

---

## Reply Detection and Routing

The most important logic in any cadence is detecting a reply and routing correctly. Sending email 4 to someone who replied "let's talk" to email 2 is a credibility-destroying mistake.

### How Reply Detection Works

Every major transactional provider can send a webhook when someone replies to an email you sent.

**Resend:** Supports inbound email via the Resend Inbound feature. Point your `reply-to` domain's MX records at Resend, and they POST every inbound email to your webhook URL.

**Postmark:** Has a built-in inbound processing feature. Best in class for this. Set up an inbound stream, add the Postmark MX records, and every reply hits your webhook.

**Webhook payload (Postmark example):**
```json
{
  "FromFull": { "Email": "lead@company.com", "Name": "John Doe" },
  "Subject": "Re: Quick question",
  "TextBody": "Yes, I'd be interested in learning more...",
  "Headers": [
    { "Name": "In-Reply-To", "Value": "<message-id-of-your-email>" }
  ]
}
```

The `In-Reply-To` header links the reply to your original message. Store your sent message IDs in your CRM to match replies to the right contact.

### n8n Reply Routing Workflow

```
Webhook trigger (inbound reply received)
  -> Extract sender email from payload
  -> Search CRM: find contact by email
  -> IF contact found AND sequence is active:
     -> Set CRM property: sequence_stopped = true
     -> Set CRM property: reply_received = true
     -> Set CRM property: reply_text = [first 500 chars of reply]
     -> Set contact owner = assigned sales rep
     -> Create CRM task: "Reply received — follow up manually"
     -> Send Slack notification to sales rep:
        "Reply from {{name}} at {{company}}: {{reply_preview}}"
  -> ELSE: log and ignore
```

### Routing by Reply Sentiment

Extend the workflow with AI-powered sentiment classification:

```
AI classification prompt:
Classify this email reply as one of: POSITIVE, NEGATIVE, OBJECTION, UNSUBSCRIBE, OUT_OF_OFFICE, OTHER

Reply: {{reply_text}}

Return only the classification word.
```

Then branch in n8n:
- POSITIVE → notify sales rep, create task "hot reply — respond today"
- NEGATIVE → mark sequence complete, tag "not interested"
- OBJECTION → notify sales rep with objection text, suggest handling script
- UNSUBSCRIBE → unsubscribe in CRM, stop sequence
- OUT_OF_OFFICE → pause sequence 7 days, then resume (don't stop)
- OTHER → notify sales rep for human review

**Out-of-office handling** is commonly missed. An OOO auto-reply should not mark a sequence as "replied by human." Detect OOO patterns and resume the sequence when they're back.

---

## A/B Testing Subject Lines in n8n

Subject lines have more impact on open rate than any other variable. Test them systematically rather than going with your gut.

### Setup

In your n8n workflow, before sending each email, add a split node that routes each contact to variant A or variant B based on a random 50/50 split:

```
Generate email body (AI)
  -> Split: IF random() > 0.5 → Variant A ELSE → Variant B
  -> Variant A: assign subject_line_a + store "ab_variant = A" in CRM
  -> Variant B: assign subject_line_b + store "ab_variant = B" in CRM
  -> Send email
```

### Reading Results

After 72 hours (enough time for opens to stabilize):

```
n8n scheduled report:
  -> Query CRM: all contacts in this test batch
  -> Group by ab_variant
  -> Calculate open rate per variant: contacts_opened / contacts_sent
  -> Send Slack summary:
     "Subject line test results:
      A: '[subject A]' — X% open rate (N sends)
      B: '[subject B]' — Y% open rate (N sends)"
```

### What to Test

Only test one variable at a time. Priority order:

1. **Question vs statement:** "Are you still struggling with X?" vs "How we solved X for [similar company]"
2. **Length:** Short (< 5 words) vs conversational (8-12 words)
3. **Personalization depth:** First name only vs first name + company vs no personalization
4. **Curiosity gap:** "Something I noticed about [company]" vs direct "Reduce your [metric] by 30%"
5. **Lowercase vs title case:** `quick question` vs `Quick Question`

### Statistical Significance

Don't declare a winner with fewer than 200 sends per variant. Open rate differences of 1-2% on small samples are noise. At 200+, a 3% difference is meaningful.

---

## Personalization Depth Levels

More personalization takes more data and more AI tokens. Match the depth to the deal value and the volume.

### Level 1: Name Only

Appropriate for: high-volume low-ticket, B2C.

```
Hi {{first_name}},
```

Cost: negligible. Impact: marginal. Better than "Hi there" but not by much.

### Level 2: Company + Role Context

Appropriate for: mid-volume B2B.

```
Hi {{first_name}},

I help {{job_function}} teams at {{industry}} companies...
```

Requires: job title and company in CRM. Available from most lead capture forms and LinkedIn enrichment.

### Level 3: Original Message Reference

Appropriate for: inbound nurture sequences.

```
Hi {{first_name}},

When you reached out last week about {{original_inquiry_summary}}...
```

This is the highest-leverage personalization for inbound because it proves you actually read what they sent. Feed the original inquiry to the AI to write this line.

### Level 4: Research-Based Context

Appropriate for: high-ticket outbound (deal value > $5K), manually reviewed before sending.

```
Hi {{first_name}},

I saw that {{company}} recently {{recent_trigger}} — congrats on that.
It made me think about how we helped {{similar_company}} do X in a similar position.
```

Requires: enrichment data (Clay, Apollo, LinkedIn) pulled fresh before send. The AI generates the email; a human reviews it before it goes out.

### Level 5: Full Account Intelligence

Appropriate for: enterprise outbound, account-based marketing, < 50 targets.

Full context: company tech stack, hiring patterns, recent press, known pain points for their segment, names of decision makers, who you know there.

At this level, the AI is writing a first draft that your sales team edits. It's not fully automated.

### Enrichment Tools to Feed the AI

- **Apollo.io** — email, phone, company size, industry, tech stack
- **Clay** — the most powerful enrichment tool; pulls from 50+ sources, runs AI research waterfalls
- **Clearbit** (now Breeze by HubSpot) — real-time enrichment on form submit
- **Hunter.io** — email finder for outbound
- **PhantomBuster** — LinkedIn data extraction at scale

---

## Re-Entry Trigger: Contact Goes Cold After Sequence Ends

A contact who completed all 8 touches without replying is not permanently lost. 60-90 days later, their situation may have changed. A re-entry trigger puts them back into a condensed sequence automatically.

### Logic

```
Cron: runs monthly
  -> Query CRM: contacts where:
     - touch_day28_sent = true (completed sequence)
     - sequence_completed_date > 60 days ago
     - reply_received = false
     - unsubscribed = false
     - email_bounced = false
     - re_entry_sent = false (prevents re-entering more than once)
  -> For each qualifying contact:
     -> Set re_entry_started = true
     -> Enroll in re-entry sequence (3-touch)
```

### Re-Entry Sequence Structure

| Touch | Timing | Approach |
|-------|--------|---------|
| 1 | Day 0 | New angle or new proof point. Don't rehash the original sequence. |
| 2 | Day 7 | Something genuinely new: product update, case study, stat. |
| 3 | Day 14 | Permission email. Final message. Let them exit gracefully. |

### The Re-Entry Email

The tone is different from the original sequence. You're not pretending they haven't heard from you. Acknowledge the history, then give them a reason to re-engage.

```
Subject: Checking back in after a few months

Hi {{first_name}},

I reached out a few months ago about [topic]. I never heard back, which is completely fine.

A few things have changed since then: [one genuinely new thing — product update, new case study, pricing change, new offering].

If the timing is better now, I'd love to pick up the conversation.

[CTA]

[Name]
```

### Preventing Re-Entry Spam

Set `re_entry_sent = true` after the first re-entry sequence completes. Only re-enter a contact once. After two completed sequences with no reply, mark them as `permanently_disengaged` and remove from all automation. Continuing to email non-responders indefinitely hurts your sender reputation.

---

## Personalization at Scale: Prompt Architecture

When running a sequence at volume, structure your AI prompt so the personalization scales cleanly.

```
You are writing email {{touch_number}} of {{total_touches}} in a sales nurture sequence.

Contact:
- Name: {{first_name}}
- Company: {{company}}
- Role: {{job_title}}
- Industry: {{industry}}
- Original inquiry: {{original_message}}
- Days since inquiry: {{days_since_created}}

Sequence context:
- Touch angles: 1=value reminder, 2=social proof, 3=objection, 4=new info, 5=urgency, 6=question, 7=direct ask, 8=breakup
- Previous touches sent: {{sent_touches_list}}
- Sequence variant: {{ab_variant}} (A=standard, B=question-opener)

Constraints:
- Under 150 words
- No em dashes
- No "hope this finds you well" or "I wanted to reach out"
- Return subject line on line 1, blank line, then email body
- No sign-off or signature — system adds those
```

Keeping the prompt format consistent lets you update the sequence logic in one place. Change the "touch angles" line to change every email's angle without rebuilding the workflow.
