# Marketing Track

This track replaces Klaviyo (email sequences), Intercom (nurture and onboarding messaging), a dedicated lead scoring tool like 6sense or Clearbit, and a sales engagement platform like Outreach or Salesloft. The output is a self-hosted marketing automation stack that runs on n8n, Resend, OpenAI, and HubSpot Free. A single inbound form submission triggers AI qualification, lead scoring, deal creation, and the start of a personalized 28-day nurture sequence -- all within 30 seconds of submission, all without manual intervention, and all for under $30/month in infrastructure costs.

---

## Cost Comparison

| Capability | SaaS Tool | SaaS Cost | Self-Hosted | Self-Hosted Cost |
|---|---|---|---|---|
| Workflow automation | Zapier (Professional) | $299/mo | n8n (self-hosted) | $0 |
| Email delivery | Mailchimp (Standard) or ActiveCampaign | $100-300/mo | Resend | $0 up to 3k/mo, $20/mo after |
| AI lead scoring | 6sense or Clearbit | $1,000-2,500/mo | OpenAI API + n8n | $5-15/mo |
| Sales sequences | Outreach or Salesloft | $100-150/seat/mo | n8n + Resend | Included above |
| Multi-touch cadences | HubSpot Sales Hub Pro | $500/seat/mo | n8n workflows | Included above |
| CRM | Salesforce or HubSpot Pro | $75-150/seat/mo | HubSpot Free | $0 |
| **Total** | | **$800-2,000+/mo** | | **Under $30/mo** |

The primary costs on the self-hosted stack are OpenAI API usage (scales with volume but stays low for most early-stage teams) and a VPS for n8n (a $6/month Hetzner or DigitalOcean instance handles hundreds of workflows per day). Resend's free tier covers 3,000 emails per month, which is enough for most teams under 500 active leads.

---

## What's in Each Folder

### Email Sequences (`./email-sequences/`)

Complete multi-touch email sequence templates with AI personalization, plus full deliverability infrastructure documentation.

**10 sequence types:**

1. **Inbound lead nurture (8-touch, 28 days)** -- For contacts who submitted a form expressing interest. Day 1 (first response within 30 minutes), Day 3, Day 5, Day 7, Day 10, Day 14, Day 21, Day 28. Each touch has a different angle: value demonstration, social proof, objection handling, urgency, final breakup.

2. **Cold outbound** -- 5-touch sequence over 14 days targeting contacts added manually or from an enrichment source. Heavier personalization requirements; AI pulls from LinkedIn data or company news if available.

3. **Re-engagement** -- For contacts who went silent after initial interest. 4 touches over 21 days with increasing urgency. Last touch is a explicit breakup email that often generates replies.

4. **Post-purchase onboarding** -- 6-touch sequence triggered by Closed Won. Day 1 (welcome + login), Day 3 (first feature), Day 7 (success check-in), Day 14 (advanced features), Day 30 (review request), Day 60 (expansion conversation).

5. **Trial-to-paid** -- 5-touch sequence triggered by trial start. Timed to key trial milestones (Day 3: have they activated the core feature?), not just calendar days.

6. **Event/webinar follow-up** -- 3-touch post-event sequence: same-day thank-you + recording link, Day 3 related resource, Day 7 conversation starter. Varies by whether the contact attended live or registered but did not show.

7. **Referral** -- 2-touch sequence sent to warm leads referred by existing customers. Uses the referrer's name and context in the opening.

8. **Review/testimonial request** -- 2-touch sequence triggered 30 days after Closed Won or after a positive NPS response. Asks for G2, Trustpilot, or Google review with direct link.

9. **Proposal follow-up** -- 4-touch sequence triggered when a deal moves to "Proposal Sent." Day 2, Day 5, Day 10, Day 15. Stops automatically when the deal stage changes (reply detected or manual update).

10. **Closed-lost revival** -- 2-touch sequence triggered 90 days after Closed Lost with `close_reason` of "timing." Simple, low-pressure check-in referencing the original conversation.

**Per-sequence documentation includes:**
- Exact day-by-day send schedule with time-of-day recommendations
- AI personalization approach for each touch (what context the prompt pulls in, what it is instructed to vary)
- Subject line strategy: question vs. statement vs. re-use of previous subject line for later touches
- Primary CTA per touch: book a call, reply to this email, view a resource, confirm interest, or no CTA (value-only touch)

**Deliverability section:**
- SPF, DKIM, and DMARC DNS record configurations for both Resend and Postmark
- 8-week IP warmup schedule: Week 1 (50 emails/day), Week 2 (100/day), Week 3 (200/day), scaling to full volume by Week 8
- Reputation monitoring: how to check Google Postmaster Tools, Microsoft SNDS, and Resend's bounce dashboard
- Inbox placement testing using tools like GlockApps or Mail Tester before launching a new sequence
- Bounce management: hard bounces set `bounced = true` in HubSpot and immediately halt the sequence; soft bounces retry once after 24 hours

**Tool comparison:**

| | Resend | Postmark | SendGrid |
|---|---|---|---|
| Free tier | 3,000/mo | None | 100/day |
| Paid pricing | $20/mo for 50k | $1.25/1k | $19.95/mo for 40k |
| API quality | Excellent, modern SDK | Excellent | Good but dated |
| Analytics | Basic (open, click, bounce) | Detailed | Detailed |
| Best for | Developer-first startups | Transactional at scale | High volume |

Recommendation: Resend for under 50k emails/month. Postmark for high-volume transactional. Both integrate identically with n8n via HTTP Request nodes.

**Unsubscribe handling** -- one-click unsubscribe link in every email footer (required for CAN-SPAM and GDPR); n8n webhook endpoint that receives the unsubscribe event, sets `gdpr_consent_given = false` in HubSpot, and halts all active sequences for that contact immediately.

---

### Lead Scoring (`./lead-scoring/`)

A four-layer scoring architecture that produces a single composite score (0-100) for every contact, updated on each new signal.

**Composite score formula:**

```
composite_score = (ai_score * 0.4) + (firmographic_score * 0.3) + (behavioral_score * 0.3)
```

The composite score is written to a custom property and used to route leads: above 75 routes to immediate rep notification, 50-75 enters the standard nurture sequence, below 50 enters the low-intent drip.

**Layer 1: AI Qualification Score (weight: 40%)**

An OpenAI prompt evaluates the contact's form submission text, company name, job title, and any enriched data. It outputs a score from 1-10 plus a plain-English reason string. The prompt is templated and versioned in the repository; the score maps to the `ai_lead_score` property and the reason maps to `ai_qualification_reason`. A score of 8-10 triggers immediate rep notification via Slack.

**Layer 2: Firmographic Scoring Matrix (weight: 30%)**

Points assigned by company attributes:

| Attribute | Signal | Points |
|---|---|---|
| Industry | In ICP vertical | +20 |
| Industry | Adjacent vertical | +10 |
| Industry | Outside ICP | 0 |
| Company size | 11-200 employees | +20 |
| Company size | 201-500 | +15 |
| Company size | 1-10 or 500+ | +5 |
| Job title | C-suite or VP | +20 |
| Job title | Director or Manager | +15 |
| Job title | Individual contributor | +5 |
| Revenue band | $1M-$50M | +20 |
| Revenue band | $50M-$200M | +15 |
| Revenue band | Under $1M | +5 |

Firmographic data comes from the form submission fields or from enrichment via Clearbit, Apollo, or Hunter.io (all have free tiers for low volume).

**Layer 3: Behavioral/Intent Scoring (weight: 30%)**

Points assigned by tracked behaviors:

| Behavior | Points |
|---|---|
| Pricing page view | +15 |
| Demo request | +25 |
| Case study download | +10 |
| Documentation visit | +10 |
| Return visit within 48 hours | +15 |
| Email opened | +3 |
| Email link clicked | +8 |
| Reply to email | +20 |
| Meeting booked | +30 |

Behavioral events are tracked via HubSpot's tracking pixel (free) and written to the contact via n8n webhook on each event.

**Score decay** -- A nightly n8n workflow reduces behavioral scores by 10% for contacts with no activity in the last 7 days, and by 20% for contacts with no activity in 14+ days. This prevents old engagement from inflating scores on leads who have gone cold. Decay is applied to the behavioral component only; firmographic and AI scores are static until re-evaluated.

**Layer 4: Semantic Similarity via Pinecone (supplementary)**

New contacts are embedded and compared against a Pinecone index of historical closed-won contacts. A similarity score above 0.85 adds a +10 bonus to the composite score. This layer improves over time as more won deals are added to the index. The embedding uses the contact's form text, job title, and company description concatenated into a single string.

**Manual override workflow** -- Any rep can set `ai_lead_score` manually in HubSpot. When a manual override is detected (n8n webhook on property change, compared against last automated update timestamp), n8n logs the override to an audit table (contact ID, old score, new score, rep ID, timestamp) and freezes automated score updates for that contact for 30 days.

**A/B testing framework** -- Scoring weight configurations (the 0.4/0.3/0.3 split) are stored as n8n environment variables, not hardcoded. An A/B test assigns new contacts to variant A (current weights) or variant B (alternative weights) based on contact ID modulo 2. Outcomes (deal created, Closed Won) are logged per variant. A quarterly review compares conversion rates between variants and updates the weights accordingly.

**Quarterly recalibration process:**
1. Export all contacts from the past quarter with their scores and outcomes
2. Calculate score-to-conversion rate by score band (0-25, 26-50, 51-75, 76-100)
3. Identify which scoring layer has the lowest predictive value
4. Adjust weights and ICP definitions accordingly
5. Re-score all active leads with the new weights
6. Update the Pinecone index with the quarter's new Closed Won contacts

---

### Multi-Touch Cadences (`./multi-touch-cadences/`)

The orchestration layer that manages which contact gets which email, when, and what happens based on their response.

**5 cadence designs:**

1. **Standard 8-touch 28-day** -- The default cadence for all inbound leads with `ai_lead_score` 5-7. Touches on Days 1, 3, 5, 7, 10, 14, 21, 28. Mixes value emails, case studies, and soft CTAs. Exits on reply, meeting booked, or Day 28 with no response.

2. **5-day fast-follow (high-intent)** -- For leads with `ai_lead_score` 8+ or `behavioral_score` including a demo request. Touches on Days 1, 2, 3, 5, 7. More direct CTAs. Rep is notified after every touch with no reply.

3. **90-day enterprise** -- For leads at companies with 500+ employees or deal value over $50k. Lower frequency (Days 1, 5, 10, 20, 35, 50, 70, 90). Longer emails with more context and social proof. Escalates to manager-to-manager outreach if no response by Day 50.

4. **Seasonal/event-triggered** -- Pauses all active cadences during major holidays (configurable list), resumes the morning after. Also triggered by external events: if a lead's company announces a funding round or a relevant industry event, n8n can inject a context-aware touch at that point in the sequence.

5. **Re-engagement with permanent disengagement logic** -- For contacts who completed the 28-day cadence with no response. 3 touches at 30-day intervals. If no response after all three re-engagement touches, `sequence_current_step` is set to 99 (permanently disengaged), the contact is excluded from all future automated sequences, and a task is created for the rep to review quarterly.

**B2B vs B2C differences** -- B2B cadences use longer gaps (3-7 days between touches), business-hours sending (Tuesday through Thursday, 8am-11am recipient time zone), and focus on ROI and business outcomes. B2C cadences use shorter gaps (1-3 days), include weekend sends, and focus on personal value and social proof. The templates include separate variants for each motion.

**Reply detection and sentiment routing** -- n8n polls for new email replies every 15 minutes (or receives webhook events from Resend). When a reply is detected:
- Positive sentiment (interested, wants to book, asking questions): halt sequence, notify rep immediately via Slack with full context, set `last_replied_at`
- Neutral sentiment (out of office, forwarded): pause sequence for 5 business days, then resume from current step
- Negative sentiment (not interested, unsubscribe request, wrong person): halt sequence immediately, update HubSpot accordingly, do not resume

Sentiment classification uses a simple OpenAI prompt. The prompt returns one of three values: `positive`, `neutral`, `negative`. No hallucination risk because the output is constrained.

**A/B subject line testing** -- Each touch in the standard cadence has two subject line variants stored in the template. New contacts are assigned to variant A or B at sequence enrollment (based on contact ID). n8n tracks open rate per variant per touch. After 100 sends per variant, the winner (higher open rate) becomes the default and the test closes. Results are logged to a Google Sheet.

**5 personalization depth levels:**

| Level | What's personalized | Use case |
|---|---|---|
| L1 | `{{first_name}}` only | High-volume cold outbound |
| L2 | First name + company name | Standard inbound nurture |
| L3 | First name, company, and form submission reference | High-quality inbound |
| L4 | L3 + AI-generated opening line based on company/role context | Scored 7+ leads |
| L5 | Full AI-written email using form text, AI reason, and behavioral signals | Scored 9-10 or enterprise leads |

L4 and L5 use an OpenAI call per email. At $0.01-0.02 per call, personalization at this depth costs roughly $10-20 per 1,000 emails sent.

**Re-entry trigger workflow** -- If a contact was Closed Lost and later returns to the website (tracked via HubSpot pixel), n8n detects the return visit, checks that `close_reason` is "timing" or "budget," and if more than 60 days have passed since the deal was lost, re-enrolls the contact in the closed-lost revival cadence. A task is also created for the original rep.

---

## End-to-End Workflow

This is the full journey from form submission to closed deal, showing how all three tracks connect.

**Step 1: Form submission (T+0)**
A visitor fills out the contact form on your website. HubSpot captures the form data and creates a contact. A HubSpot webhook fires to n8n.

**Step 2: AI qualification (T+5 seconds)**
n8n receives the webhook. It sends the form text, company name, and job title to OpenAI. The response sets `ai_lead_score`, `ai_lead_tags`, and `ai_qualification_reason` on the contact in HubSpot.

**Step 3: Firmographic and behavioral scoring (T+10 seconds)**
n8n calculates the firmographic score from the contact's industry, company size, and job title fields. Behavioral score starts at 0 (no behavior yet beyond the form submission itself). The composite score is calculated and written to HubSpot.

**Step 4: Routing decision (T+15 seconds)**
Based on composite score:
- Score 75+: Slack notification to the assigned rep with full context. Fast-follow cadence enrolled.
- Score 50-74: Standard 8-touch cadence enrolled. Rep notified same day.
- Score below 50: Low-intent drip enrolled. No rep notification unless score changes.

**Step 5: First response email (T+30 seconds)**
n8n sends the first touch via Resend. For L4/L5 leads, this is a fully AI-written email referencing the contact's specific form submission. For L1/L2 leads, it is a template with personalization tokens. `ai_first_response_sent` is set to true, `ai_first_response_sent_at` is recorded.

**Step 6: Deal creation (if score 6+)**
n8n creates a deal in HubSpot, associates it with the contact and their company, sets `deal_source_campaign` from the UTM data, and assigns it to a rep.

**Step 7: Sequence progression (Days 1-28)**
n8n checks each morning for contacts whose next touch is due (based on `sequence_current_step` and the last send timestamp). It sends the scheduled email, increments `sequence_current_step`, and sets the corresponding `touch_dayN_sent` flag.

**Step 8: Behavioral signals update the score**
As the contact opens emails, clicks links, or returns to the website, n8n receives those events and updates the behavioral score. If the composite score crosses 75 mid-sequence, the rep is notified and the sequence switches to fast-follow mode.

**Step 9: Reply triggers rep handoff**
When the contact replies, n8n classifies the sentiment, halts the sequence, and sends the rep a Slack message with the reply content, the contact's score, and the AI qualification reason. The rep takes it from there.

**Step 10: Post-close**
When the deal is marked Closed Won, n8n enrolls the contact in the onboarding sequence, updates `days_lead_to_customer`, adds the contact's embedding to the Pinecone index (improving future scoring), and triggers a review request email 30 days later.

---

## Performance Benchmarks

These are realistic targets based on the patterns in this track. Results vary by industry, list quality, and offer.

| Metric | Cold outbound | Inbound nurture | Re-engagement |
|---|---|---|---|
| Open rate (L1/L2 personalization) | 20-30% | 35-50% | 15-25% |
| Open rate (L4/L5 AI personalization) | 35-50% | 50-65% | 25-40% |
| Click rate | 2-5% | 5-12% | 2-6% |
| Reply rate | 1-3% | 3-8% | 2-5% |
| Meeting booked rate (of replies) | 20-35% | 35-55% | 15-30% |

The single biggest factor in open rate is subject line. The single biggest factor in reply rate is the opening line -- whether it references something specific to the recipient versus reads like a template. L4/L5 personalization measurably improves reply rates because the opening line is written by the AI using the contact's own form submission text, which almost always includes context that no template would catch.

Deliverability is the other major variable. A properly warmed sending domain with SPF, DKIM, and DMARC configured will land in the primary inbox. An improperly configured domain will land in spam regardless of content quality.

---

## What You Need to Get Started

**Required:**
- n8n instance (self-hosted on a VPS, or n8n Cloud)
- HubSpot account (free tier is sufficient)
- Resend account (free tier covers first 3,000 emails/month)
- OpenAI API key (GPT-4o-mini is sufficient for scoring and personalization; costs stay under $10/month for most early-stage volumes)
- A sending domain (separate from your main domain -- use something like `mail.yourdomain.com` or a dedicated sending domain)

**For lead scoring with semantic similarity:**
- Pinecone account (free tier includes one index, sufficient for most use cases)
- At least 20-30 historical Closed Won contacts to seed the index (the more the better)

**For deliverability:**
- DNS access to configure SPF, DKIM, and DMARC records on your sending domain
- 6-8 weeks for IP warmup before sending at full volume

**Recommended but optional:**
- An enrichment tool (Apollo, Hunter.io, or Clearbit) to fill in missing firmographic data on contacts that submit minimal form fields
- Google Sheets (used as a lightweight audit log for overrides, A/B test results, and recalibration data)
- A Slack workspace for rep notifications (the workflows default to Slack; substitute any messaging webhook if needed)

**HubSpot custom properties** -- Before the workflows will run correctly, you need to create the custom properties defined in the HubSpot track's `./custom-properties/` folder. The `n8n-integrations/` folder in the HubSpot track includes a script that creates all 80+ properties via the HubSpot API in a single run.
