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

## Cold Outbound Sequence

Cold outbound is fundamentally different from inbound nurture. The recipient has no context for who you are, has never expressed interest, and your tolerance for error is lower — one off-tone email and they unsubscribe or mark as spam permanently.

**Core differences from inbound:**
- Subject lines must earn attention, not assume it
- Email 1 is about relevance, not value — prove you know who they are
- Sequences are shorter (5-7 touches max)
- Personalization must be specific and researched, not just name + company

**Sequence structure:**

| Touch | Day | Goal | Length |
|-------|-----|------|--------|
| 1 | 0 | Relevant hook, one specific observation about them | 3-4 sentences |
| 2 | 3 | Value proof without pitching | 4-5 sentences |
| 3 | 7 | Case study or outcome for their type of company | 5-6 sentences |
| 4 | 12 | Handle the most common objection for their segment | 4-5 sentences |
| 5 | 18 | Soft ask — offer something, not a meeting | 3-4 sentences |
| 6 | 25 | Direct ask | 2-3 sentences |
| 7 | 32 | Final value + breakup | 2-3 sentences |

**AI prompt for cold outbound email 1:**
```
Write a cold outreach email.

Recipient: {{name}} at {{company}}
Their role: {{title}}
What we know about them: {{research_context}}
What we do: {{your_offer}}
Our ICP: {{ideal_customer_profile}}

Rules:
- Open with a specific, researched observation about their company or role (not a compliment)
- Do not pitch in email 1 — establish relevance only
- No subject line with their first name as the opener
- 3-4 sentences maximum
- Do not use phrases like "I hope this finds you well" or "I wanted to reach out"
- End with one low-commitment question, not "do you have 15 minutes?"
- Return: subject line + email body only
```

**Personalization sources to feed the AI:**
- Recent LinkedIn posts or articles they've published
- Company news (funding round, new product launch, hiring surge)
- Job description keywords (tells you their current priorities)
- Mutual connections or shared communities
- Recent company blog posts or press releases

---

## Re-Engagement Campaign (Dead Contacts)

For contacts who entered a sequence, didn't reply, and have gone dark for 60+ days. Not the same as the original sequence — different tone, different angle.

**Who qualifies:**
- Completed the original nurture sequence with no reply
- No email activity (open or click) in 60+ days
- Not unsubscribed, not bounced, not closed won/lost

**Sequence structure (3-touch, not 8):**

| Touch | Timing | Angle |
|-------|--------|-------|
| 1 | Day 0 | Acknowledge the silence, no pressure |
| 2 | Day 7 | Share something genuinely new (product update, case study, stat) |
| 3 | Day 14 | One final soft close, permission to remove them |

**Sample touch 1:**
```
Subject: Still on your radar?

Hi {{first_name}},

We exchanged a few emails back in {{month}} and I never heard back — no worries at all.

Things change and timing matters. If {{original_problem}} is still something you're thinking about, I'd love to reconnect.

If not, just reply "not now" and I'll stop reaching out.

[Your name]
```

**Sample touch 3 (permission email):**
```
Subject: Last one from me

{{first_name}},

This is the last email I'll send. I don't want to be in your inbox if it's not useful.

If you're ever in the market for [what you do], we're here.

[link to a useful resource]

[Your name]
```

The permission email consistently outperforms aggressive closes on dead contacts. Giving people an easy exit generates more replies than one more push.

---

## Post-Purchase Onboarding Sequence

Triggered the moment a purchase is confirmed or a contract is signed. Goal: get them to first value as fast as possible. Churn happens when people don't reach the "aha moment" in the first 30 days.

**Trigger:** Payment confirmed / deal stage moved to "Closed Won"

| Touch | Timing | Goal |
|-------|--------|------|
| 1 | Immediate | Welcome + single most important first step |
| 2 | Day 1 | Quick win — simplest thing they can do in 5 minutes |
| 3 | Day 3 | Next milestone + proof others have hit it |
| 4 | Day 7 | Check-in: have they done step 1? Offer help if not |
| 5 | Day 14 | Share an advanced use case they might not know about |
| 6 | Day 30 | Ask for feedback. Are they getting value? |

**Touch 1 — Welcome email structure:**
1. Confirm the purchase (order/plan details)
2. State clearly: the one thing they should do first
3. Link directly to that one thing (not your dashboard homepage)
4. Support contact + response time

**Touch 4 — Check-in using behavioral data:**

```
If [first login happened]: "Great to see you in — here's what's next."
If [no login yet]:          "Haven't seen you set up yet — can I help?"
```

Wire this in n8n by checking the "first_login_date" CRM property. If it's empty after 7 days, the check-in email takes a different branch with an offer to jump on a call.

---

## Trial-to-Paid Conversion Sequence

For SaaS products with a free trial. The goal is to convert before the trial expires — or if they don't convert, understand why.

**Timing assumption:** 14-day trial. Adjust days if your trial is different.

| Touch | Day | Goal |
|-------|-----|------|
| 1 | Trial day 1 | Onboarding (same as post-purchase, but lighter) |
| 2 | Trial day 3 | Highlight the 1-2 features that drive conversion |
| 3 | Trial day 7 | Mid-trial check. Show usage stats if you have them. |
| 4 | Trial day 10 | Feature they haven't tried yet |
| 5 | Trial day 12 | Trial ending in 2 days. Clear upgrade CTA. |
| 6 | Trial day 14 | Trial ended. What did they think? |
| 7 | Day 17 (post-trial) | For non-converters: what would make it worth paying? |

**Touch 5 — Pre-expiry urgency (no fake scarcity):**
```
Subject: Your trial ends in 48 hours

{{first_name}},

Your free trial of [product] ends on [date].

You've [used X feature / created Y items / done Z]. [Personalize with actual usage data if available.]

To keep everything you've built: [upgrade link]

Questions about plans or pricing? Reply here.

[Your name]
```

**Touch 7 — The survey email (non-converters):**

This is gold. A short 1-question reply survey for people who didn't convert tells you more than any analytics:

```
Subject: Quick question

{{first_name}},

Your trial ended. You didn't upgrade — that's completely fine.

Could you tell me why? One honest sentence is enough.
Was it price, timing, missing features, or something else entirely?

Replying to this email goes directly to me.

[Founder's name]
```

---

## Event / Webinar Promotion Sequence

For promoting a live event: webinar, workshop, summit, product launch.

**Timing:** Reverse-engineer from event date.

| Touch | Timing | Goal |
|-------|--------|------|
| 1 | 3 weeks before | Announcement + early registration |
| 2 | 2 weeks before | What they'll learn / who else is attending |
| 3 | 1 week before | Speaker or session spotlight |
| 4 | 3 days before | Registration closing / seats limited |
| 5 | Day before | Logistics: link, time, what to expect |
| 6 | Day of, 1hr before | Live reminder + join link |
| 7 | Day after (non-attendees) | Recording available |
| 8 | Day after (attendees) | Follow-up + next step CTA |

**Segment non-attendees from attendees for post-event emails.** The recording email to someone who didn't register is different from the one to someone who registered but didn't show up.

---

## Referral Request Sequence

Triggered 30-60 days post-purchase, when the customer has had enough time to get value.

**Timing rule:** Only send after the customer has hit a meaningful milestone (first success, renewed, or reached a usage threshold). Don't ask for referrals from people who haven't won yet.

| Touch | Timing | Goal |
|-------|--------|------|
| 1 | Day 0 | Ask for a referral with specific, easy instructions |
| 2 | Day 7 (no action) | Reminder + make the ask easier (share a link, not a name) |
| 3 | Day 14 (no action) | Final ask |

**Touch 1 structure:**
```
Subject: Know anyone who'd benefit?

{{first_name}},

[One sentence personalizing to their result: "Now that you've done X with us..."]

If you know anyone who's dealing with [the problem you solve], I'd love an introduction.

It doesn't need to be formal — forwarding this email with a note works perfectly.

[Optional: describe any referral incentive here]

Thanks,
[Your name]
```

The easier you make the referral action, the higher the conversion. "Forward this email" converts better than "fill out a referral form."

---

## Review / Testimonial Request Sequence

Triggered after a successful outcome milestone — not at a fixed time interval.

**Trigger examples:**
- Customer renewed
- Customer reached 90 days with no support tickets
- Customer hit a result milestone (e.g., "you've processed 100 orders")
- NPS survey returned score 8+

| Touch | Timing | Goal |
|-------|--------|------|
| 1 | Trigger day | Direct ask, multiple platform options |
| 2 | Day 5 (no review yet) | Reminder with simpler option (written quote vs public review) |
| 3 | Day 12 (no review yet) | Final ask with maximum simplicity |

**Touch 1 — Give them options by effort level:**
```
Subject: 2 minutes? Quick favor

{{first_name}},

[Personalized line about their specific result.]

If you've got 2 minutes, a review would mean a lot:
- Google: [link]
- G2: [link]
- LinkedIn recommendation: [link]

Or if you'd rather just reply with a sentence about your experience, I can take it from there.

Thanks,
[Your name]
```

---

## Deliverability

Getting into the inbox is a system problem, not just a content problem. You can write perfect emails and still land in spam if the technical foundation isn't set up.

### SPF / DKIM / DMARC Setup

All three must be configured before you send a single email from a new domain. In that order.

**SPF (Sender Policy Framework)**
Tells receiving mail servers which IP addresses are authorized to send email for your domain.

```dns
TXT @ "v=spf1 include:_spf.resend.com ~all"
```

If you send from multiple providers (Resend + Google Workspace), combine includes:
```dns
TXT @ "v=spf1 include:_spf.resend.com include:_spf.google.com ~all"
```

Avoid `+all` (permissive) and `~all` vs `-all` tradeoff: `~all` (softfail) is safer during setup; move to `-all` (hardfail) once you're confident all your sending sources are listed.

**DKIM (DomainKeys Identified Mail)**
Cryptographic signature that proves the email wasn't tampered with in transit.

Each sending provider generates a key pair. You add the public key as a DNS TXT record at the subdomain they specify:

```dns
TXT resend._domainkey.yourdomain.com "[public key from Resend]"
```

Resend, Postmark, and SendGrid all have dashboards that walk you through the specific record. The exact subdomain name varies by provider.

**DMARC (Domain-based Message Authentication)**
Tells receiving servers what to do when SPF or DKIM fails, and sends you reports.

Start in monitor mode (`p=none`). Move to quarantine after you've reviewed reports for 2 weeks. Move to reject after you're confident.

```dns
TXT _dmarc.yourdomain.com "v=DMARC1; p=none; rua=mailto:dmarc-reports@yourdomain.com; ruf=mailto:dmarc-failures@yourdomain.com; fo=1"
```

DMARC report processors: [dmarcian.com](https://dmarcian.com), [postmaster tools](https://postmaster.google.com) (Google), or Valimail Monitor (free tier).

**Verification:**
Use [mxtoolbox.com](https://mxtoolbox.com) or [mail-tester.com](https://www.mail-tester.com) to confirm all three records are valid before sending.

### Domain Warmup Strategy

A new sending domain has no reputation. ISPs don't trust it. If you send 1,000 emails on day one, most will land in spam.

**Warmup schedule:**

| Week | Emails/day | Notes |
|------|-----------|-------|
| 1 | 10-20 | Send only to known-good addresses (customers, team) |
| 2 | 30-50 | Expand to engaged subscribers |
| 3 | 75-100 | Monitor open rates carefully |
| 4 | 150-200 | Keep opens above 20% or slow down |
| 5-8 | Double each week | Continue until target volume |

**Rules during warmup:**
- Open rates below 15% = slow down immediately
- Spam complaint rate above 0.1% = stop and diagnose
- Use your most engaged contacts first (recent openers, customers)
- Don't use a warmup period to send promotional email — send newsletters or content, not hard sells

**Automated warmup services:** Warmy.io, Mailreach.io, Lemwarm (if you're using Lemlist). These auto-send between seed addresses to build reputation faster. Worth the cost if you're running high-volume outbound.

### Separate Domains for Transactional vs Marketing

Protect your transactional email reputation (password resets, receipts, alerts) from your marketing email reputation.

```
yourdomain.com        — website and main email
mail.yourdomain.com   — transactional (Resend/Postmark)
send.yourdomain.com   — marketing (SendGrid/Mailchimp)
```

If your marketing email gets spam complaints, it won't affect your transactional email delivery.

### Domain Reputation Monitoring

- **Google Postmaster Tools** — shows domain reputation (High/Medium/Low/Bad) and spam rate for Gmail. Free, essential. Connect your sending domain.
- **Microsoft SNDS** — same for Outlook/Hotmail. Less critical but worth checking monthly.
- **Validity (formerly Return Path)** — paid, monitors across all ISPs.
- **MXToolbox** — blacklist monitoring. Check your sending IP against 100+ blacklists.

Check Google Postmaster weekly when ramping volume. If reputation drops to "Low", pause and investigate immediately.

### Inbox Placement Testing

Before sending a campaign, test where it actually lands:

**GlockApps** — sends to seed addresses across Gmail, Outlook, Yahoo, and shows inbox vs spam vs promotional tab placement. Most comprehensive.

**Mail-tester.com** — free, quick. Sends to one seed address, scores your email on SPF/DKIM/DMARC, spam filter rules, content issues. Run every email through this.

**Litmus** — also tests visual rendering across 90+ email clients. More useful for HTML template QA than inbox placement.

**Content checklist before sending:**
- [ ] No URL shorteners (bit.ly, tinyurl — flagged by spam filters)
- [ ] No ALL CAPS in subject line
- [ ] Text-to-image ratio > 60% text
- [ ] Unsubscribe link present and functional
- [ ] Physical address in footer (CAN-SPAM requirement)
- [ ] "From" name matches sending domain
- [ ] No spam trigger words: "free money", "act now", "100% guaranteed", "no obligation"

---

## Resend vs Postmark vs SendGrid

All three are legitimate transactional email providers. The right choice depends on volume, use case, and how much you care about developer experience.

### Resend

Best for: developers building new projects. Clean API, React Email support, simple pricing.

**Strengths:**
- Developer-first: built around the `send()` function, not a GUI
- Native React Email integration — build templates in JSX
- Simple flat pricing (no tiers of features locked behind enterprise plans)
- Excellent API documentation
- Built-in domain verification flow

**Weaknesses:**
- Newer: smaller deliverability track record than Postmark
- Analytics are basic compared to SendGrid
- No built-in list management (you handle that in your CRM or database)

**Pricing:** Free up to 3,000 emails/month. $20/month for 50,000.

**n8n integration:** HTTP Request node, POST to `https://api.resend.com/emails` with `Authorization: Bearer [key]`.

### Postmark

Best for: transactional email with the highest deliverability bar. The go-to choice for password resets, receipts, alerts where delivery is non-negotiable.

**Strengths:**
- Industry-leading deliverability (separate pools for transactional vs bulk)
- Message streams (separate reputation for transactional and broadcasts)
- Detailed bounce, spam complaint, and open tracking
- Inbound email parsing (webhooks when someone replies or emails your domain)
- 45-day message log — you can see exactly what happened to any email

**Weaknesses:**
- More expensive than Resend at scale
- Template system is less modern than React Email
- Dashboard UI is dated but functional

**Pricing:** $15/month for 10,000 emails, scales linearly.

**Best for:** SaaS apps where a missed password reset email is a support ticket.

### SendGrid

Best for: high-volume marketing email, with a full suite of marketing tools you may or may not want.

**Strengths:**
- Battle-tested at massive scale (handles billions of emails)
- Full marketing platform: lists, segments, A/B tests, automation
- Detailed analytics and click tracking
- Dedicated IP options for high-volume senders
- SMTP relay works with almost anything

**Weaknesses:**
- Complex pricing (API + Marketing have separate costs)
- Feature bloat if you just want to send emails
- Support has gotten worse as the platform grew
- Owned by Twilio — some instability in product direction

**Pricing:** Free up to 100 emails/day. $19.95/month for 50,000.

**Best for:** Replacing a legacy marketing email tool (Mailchimp, Constant Contact) with something you control via API.

### Quick decision table

| Situation | Recommendation |
|-----------|---------------|
| New SaaS app, primarily transactional | Resend |
| High-stakes transactional (fintech, auth flows) | Postmark |
| Marketing email at 100k+ volume | SendGrid |
| Marketing email at < 50k volume | Resend |
| Need inbound email parsing | Postmark |
| Need a full marketing automation platform | SendGrid or a purpose-built tool (ActiveCampaign, Klaviyo) |

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
