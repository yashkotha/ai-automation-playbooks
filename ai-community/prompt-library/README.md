# Prompt Library

Prompts used in production automation workflows. Copy them directly into n8n HTTP Request nodes, OpenAI nodes, or any API client.

All prompts follow these rules:
- Return structured output where needed (JSON for machine-readable responses)
- No em dashes in generated content
- Human tone, not corporate or AI-sounding
- Use `{{variable}}` syntax for n8n expression substitution

---

## SALES

---

### Lead Qualification

```
You are a lead qualification assistant. Analyze this lead and score them 1-10.

Lead info:
Name: {{name}}
Email: {{email}}
Message: {{message}}
Source: {{source}}

Scoring:
- Budget signals (mentions price, numbers, investment): up to 3 points
- Timeline signals (urgent, this month, ready, ASAP): up to 3 points
- Fit signals (their need matches what we offer): up to 2 points
- Engagement quality (specific message vs one-liner): up to 2 points

Return JSON only, no other text:
{
  "score": <number 1-10>,
  "reason": "<one sentence explaining the score>",
  "tags": ["<tag>", "<tag>"],
  "recommended_action": "<next step for the sales team>"
}
```

---

### Personalized First-Response Email

```
Write a warm, personalized first response email from a business to a new lead.

Lead name: {{name}}
Their message: {{message}}
Business type: {{business_type}}

Rules:
- Do not use em dashes
- No generic openers ("I hope this finds you well", "Thank you for reaching out")
- Reference something specific from their message
- Under 150 words
- End with one clear, soft CTA
- Return the email body only, no subject line

Write the email:
```

---

### Weekly Pipeline Summary

```
You are a sales ops assistant. Write a Monday morning pipeline summary for the team.

Pipeline data:
{{pipeline_json}}

Include:
- Total active deals and combined value
- Deals that progressed this week
- Deals with no activity in 7+ days
- Top 3 deals to prioritize this week and why
- Any patterns or concerns worth flagging

Tone: direct and useful, like a good manager's update. No filler. Plain text.
```

---

### Deal Note Summarizer

```
Summarize these CRM notes and activity log into a brief for a salesperson.

Data:
{{crm_notes}}

Format:
- Who they are (1 line)
- What they want (1-2 lines)
- Where things stand (1-2 lines)
- Recommended next action (1 line)

Under 100 words. Plain text only.
```

---

### Re-engagement Email

```
Write a re-engagement email for a lead who went quiet.

Lead name: {{name}}
Days since last contact: {{days_since_contact}}
Original interest: {{original_interest}}
Emails sent so far: {{touch_count}}

Rules:
- Don't be passive-aggressive about the silence
- Offer something genuinely useful, not just "checking in"
- Give them an easy, no-pressure out
- Under 100 words
- No em dashes, no AI-sounding phrases
- Return email body only
```

---

### Post-Meeting Follow-Up

```
Write a follow-up email after a sales call or product demo.

Lead name: {{name}}
What was discussed: {{meeting_notes}}
Next step agreed: {{next_step}}

Rules:
- Reference specific things from the meeting using the notes
- Confirm the next step clearly
- Under 120 words
- No em dashes
- Human, direct tone
- Return email body only
```

---

### Multi-Touch Sequence (Per Touch)

```
Write touch {{touch_number}} of 8 in a sales nurture sequence.

Lead name: {{name}}
Original inquiry: {{original_message}}
Touch angles: 1=value reminder, 2=social proof, 3=objection handling, 4=new info, 5=urgency, 6=question-led, 7=direct ask, 8=breakup

Rules:
- Match the tone to the touch angle
- Reference their original inquiry where relevant
- Do not use em dashes
- Under 150 words
- Return email body only
```

---

### Breakup Email (Final Touch)

```
Write a polite breakup email. This lead has been in a 30-day sequence with no response.

Lead name: {{name}}
Original interest: {{original_interest}}

Rules:
- Be genuinely understanding, not guilt-tripping
- Leave the door open without being desperate
- Under 80 words
- No em dashes
- Return email body only
```

---

### Objection Handler

```
You are a sales assistant. A prospect has raised an objection. Write a short, honest response that addresses it directly.

Prospect name: {{name}}
Their objection: {{objection}}
Business type: {{business_type}}
Key differentiators: {{differentiators}}

Rules:
- Acknowledge the objection first before responding to it
- Do not be defensive or dismissive
- Provide one concrete, specific response (not a list of bullet points)
- If the objection is about price, do not discount. Reframe value instead.
- Under 120 words
- No em dashes
- Return email body only, no subject line
```

---

### Call Prep Brief

```
Generate a call prep brief for a sales rep before a discovery or demo call.

Contact name: {{contact_name}}
Company: {{company_name}}
Industry: {{industry}}
Deal stage: {{deal_stage}}
CRM notes summary: {{crm_notes}}
Original inquiry: {{original_message}}
Meeting type: {{meeting_type}}

Output format (plain text, no headers):
1. Who they are (2 sentences: company context and what they do)
2. Why they reached out (their stated problem or interest)
3. Where things stand (deal history, past touchpoints)
4. Key questions to ask on this call (3 questions, numbered)
5. Watch out for (one potential objection or concern to be ready for)

Under 200 words total.
```

---

### Deal Summary (for Handoff or Internal Review)

```
Write a concise deal summary for internal review or handoff to another team member.

Deal name: {{deal_name}}
Contact: {{contact_name}} at {{company_name}}
Deal value: {{deal_value}}
Current stage: {{deal_stage}}
Open since: {{deal_created_date}}
CRM notes: {{crm_notes}}
Last activity: {{last_activity}}

Return JSON only:
{
  "one_liner": "<deal described in one sentence>",
  "background": "<2-3 sentences on who the prospect is and what they need>",
  "current_status": "<1-2 sentences on where the deal stands>",
  "key_risks": ["<risk 1>", "<risk 2>"],
  "recommended_next_step": "<single most important action>",
  "estimated_close_confidence": "<low | medium | high>",
  "confidence_reason": "<one sentence>"
}
```

---

### Win/Loss Analysis

```
Analyze this closed deal and extract learnings to improve future sales.

Outcome: {{outcome}} (won or lost)
Contact: {{contact_name}} at {{company_name}}
Deal value: {{deal_value}}
Time in pipeline: {{days_in_pipeline}} days
Deal stage when closed: {{final_stage}}
CRM notes: {{crm_notes}}
Stated reason for outcome: {{stated_reason}}

Return JSON only:
{
  "outcome_summary": "<2 sentences summarizing what happened>",
  "what_worked": ["<item>", "<item>"],
  "what_did_not_work": ["<item>", "<item>"],
  "root_cause": "<one sentence identifying the single biggest factor in this outcome>",
  "lesson_for_next_time": "<one actionable change to make for the next similar deal>",
  "deal_type_tag": "<one of: price_sensitive | timing | competitor | fit_mismatch | champion_lost | process | relationship>"
}
```

---

## MARKETING

---

### Landing Page Copy

```
Write high-converting landing page copy for a product or service.

Product/service name: {{product_name}}
Target audience: {{target_audience}}
Core problem it solves: {{core_problem}}
Key benefits (3-5): {{benefits}}
Social proof available: {{social_proof}}
Primary CTA: {{cta_text}}

Output sections (label each):
1. Headline (under 10 words, outcome-focused)
2. Subheadline (under 20 words, clarifies who it is for)
3. Problem paragraph (2-3 sentences, describe the pain)
4. Solution paragraph (2-3 sentences, introduce the product as the answer)
5. 3 benefit bullets (feature + outcome format: "Get X so you can Y")
6. Social proof line (1 sentence using the proof provided)
7. CTA section (CTA button text + one supporting line under it)

Rules:
- No em dashes
- No hype words: revolutionary, game-changing, transformative, cutting-edge
- Write in second person ("you", not "our customers")
- Plain text output, labels only, no markdown formatting
```

---

### Ad Copy (Multiple Variants)

```
Write 5 variants of ad copy for a paid social or search campaign.

Product/service: {{product_name}}
Target audience: {{target_audience}}
Core benefit: {{core_benefit}}
Offer or hook: {{offer}}
Character limit per variant: {{character_limit}}
Platform: {{platform}} (Facebook, Instagram, Google, LinkedIn)

For each variant, use a different angle:
1. Problem/solution
2. Social proof / results
3. Curiosity / open loop
4. Direct offer / urgency
5. Question-led / challenge assumption

Return JSON only:
{
  "variants": [
    {
      "angle": "<angle name>",
      "headline": "<headline text>",
      "body": "<ad body text>",
      "cta": "<call to action text>"
    }
  ]
}

Rules:
- No em dashes
- No exclamation points unless the brief specifically calls for high energy
- Each variant must be genuinely different in structure and angle, not just a rewording
```

---

### Social Media Post

```
Write a social media post for {{platform}}.

Topic: {{topic}}
Key message: {{key_message}}
Audience: {{audience}}
Tone: {{tone}} (professional, casual, educational, conversational)
Include CTA: {{include_cta}} (yes/no)
CTA target: {{cta_target}}

Platform-specific rules:
- LinkedIn: Start with a hook line (no em dash). Use short paragraphs (1-2 sentences each). No hashtag spam (max 3 relevant hashtags).
- Twitter/X: Under 280 characters. Punchy. One clear point. Optional: thread structure if topic needs it.
- Instagram: Hook first line. Story-driven. 3-5 relevant hashtags at the end.
- Facebook: Conversational. Can be longer. Question or statement opener.

Rules:
- No em dashes
- No AI-sounding phrases ("In today's fast-paced world", "Let's dive in", "I'm excited to share")
- Write the post text only, no additional explanation

Write the post:
```

---

### Case Study Outline

```
Create a structured outline for a customer case study.

Customer company: {{company_name}}
Industry: {{industry}}
Product/service they used: {{product_name}}
Problem before: {{problem_before}}
Solution implemented: {{solution_summary}}
Results achieved: {{results}}
Customer quote available: {{quote}}

Return JSON only:
{
  "title": "<case study title (outcome-first format: How [Company] [achieved result] with [Product])>",
  "subtitle": "<one-line teaser>",
  "sections": [
    {
      "section_name": "The Challenge",
      "key_points": ["<point>", "<point>"],
      "suggested_word_count": 100
    },
    {
      "section_name": "The Solution",
      "key_points": ["<point>", "<point>"],
      "suggested_word_count": 150
    },
    {
      "section_name": "The Results",
      "key_points": ["<point>", "<point>"],
      "suggested_word_count": 100
    },
    {
      "section_name": "What They Said",
      "key_points": ["<trimmed quote>"],
      "suggested_word_count": 50
    }
  ],
  "stat_callouts": ["<key metric>", "<key metric>"],
  "meta_description": "<155 character SEO meta description for this case study>"
}
```

---

### SEO Meta Tags

```
Write SEO meta tags for a web page.

Page type: {{page_type}} (homepage, product page, blog post, landing page, category page)
Page topic: {{page_topic}}
Target keyword: {{target_keyword}}
Secondary keywords: {{secondary_keywords}}
Business name: {{business_name}}
Core value proposition: {{value_proposition}}

Return JSON only:
{
  "title_tag": "<50-60 characters, include target keyword near the front>",
  "meta_description": "<150-160 characters, include target keyword, end with a soft CTA>",
  "og_title": "<Open Graph title, can be slightly more casual than title tag, under 60 characters>",
  "og_description": "<Open Graph description, under 200 characters>",
  "h1_suggestion": "<Page H1 heading, include keyword, under 70 characters>",
  "url_slug_suggestion": "<SEO-friendly URL slug, lowercase, hyphens only>"
}

Rules:
- Title tags must be unique descriptions, not keyword stuffing
- Meta descriptions must include a reason to click, not just describe the page
- No em dashes
```

---

## CUSTOMER SUCCESS

---

### Onboarding Email Series (Full Sequence Generator)

```
Write a {{sequence_length}}-email onboarding series for a new customer.

Customer name: {{customer_name}}
Product/service: {{product_name}}
What they purchased: {{purchase_details}}
Their primary goal: {{customer_goal}}
Company name: {{company_name}}

Generate each email as a separate object. The sequence should follow this arc:
- Email 1 (Day 0): Warm welcome, confirm what they bought, set expectations for what happens next
- Email 2 (Day 2): First action they should take, one specific step only
- Email 3 (Day 5): Educational: how to get the most out of [key feature]
- Email 4 (Day 10): Check-in, surface any friction, invite questions
- Email 5 (Day 14): Social proof: how other customers use this to achieve similar goals
- Email 6 (Day 21): Milestone check: are they on track? Offer help if not.

Return JSON only:
{
  "emails": [
    {
      "day": <number>,
      "subject_line": "<email subject>",
      "preview_text": "<email preview text, under 90 characters>",
      "body": "<email body text>"
    }
  ]
}

Rules:
- No em dashes
- Each email must have one clear purpose and one clear CTA
- Under 150 words per email body
- Human tone, not corporate
- Reference the customer's stated goal in at least 2 emails
```

---

### Check-In Email (Mid-Onboarding)

```
Write a check-in email to a customer who is midway through their onboarding period.

Customer name: {{customer_name}}
Product: {{product_name}}
Days since purchase: {{days_since_purchase}}
Usage data summary: {{usage_summary}}
Any known issues: {{known_issues}}

Rules:
- Reference their usage data if available (e.g., "You've logged in 3 times this week")
- If usage is low, gently surface it without being accusatory
- Offer one concrete, specific piece of help (not a generic "let us know if you need anything")
- Under 120 words
- No em dashes
- Return email body only, no subject line
```

---

### Renewal / Upsell Email

```
Write a renewal or upsell email to an existing customer.

Customer name: {{customer_name}}
Product they have: {{current_product}}
Renewal date or upsell target: {{target_date}}
Upsell offer (if applicable): {{upsell_offer}}
Key value they have received: {{value_delivered}}

Rules:
- Lead with the value they have already received, not the ask
- For renewals: make the path of least resistance renewing, not canceling
- For upsells: explain specifically what the upgrade adds, not just that it exists
- One clear CTA (renew button or "reply to this email")
- Under 150 words
- No em dashes
- Return email body only
```

---

### Churn Risk Flag Analysis

```
Analyze this customer's usage and engagement data and flag churn risk.

Customer name: {{customer_name}}
Company: {{company_name}}
Product: {{product_name}}
Contract value: {{contract_value}}
Renewal date: {{renewal_date}}
Usage data: {{usage_data}}
Support tickets (last 90 days): {{support_tickets}}
Last login: {{last_login}}
NPS score (if available): {{nps_score}}
CS notes: {{cs_notes}}

Return JSON only:
{
  "churn_risk": "<low | medium | high | critical>",
  "risk_score": <number 1-10>,
  "risk_signals": ["<signal>", "<signal>"],
  "primary_risk_reason": "<one sentence>",
  "recommended_action": "<specific intervention the CS team should take>",
  "urgency": "<immediate (this week) | soon (this month) | monitor>",
  "talking_points": ["<point to raise in next call>", "<point to raise in next call>"]
}
```

---

## OPERATIONS

---

### Meeting Summary to Action Items

```
Convert a meeting transcript or notes into a structured summary with action items.

Meeting title: {{meeting_title}}
Date: {{meeting_date}}
Attendees: {{attendees}}
Raw notes or transcript: {{raw_notes}}

Return JSON only:
{
  "summary": "<2-4 sentence summary of what was discussed and decided>",
  "decisions_made": ["<decision>", "<decision>"],
  "action_items": [
    {
      "task": "<what needs to be done>",
      "owner": "<person responsible>",
      "due_date": "<date or timeframe if mentioned, else null>",
      "priority": "<high | medium | low>"
    }
  ],
  "open_questions": ["<unresolved question>", "<unresolved question>"],
  "next_meeting": "<date/time if mentioned, else null>"
}

Rules:
- Extract only what was explicitly stated or clearly implied
- Do not invent action items or owners
- If an owner is not assigned for an action item, leave it as null
```

---

### Status Report Writer

```
Write a professional status report for a project or workstream.

Project name: {{project_name}}
Reporting period: {{period}}
Report audience: {{audience}} (e.g., executive team, client, board)
Work completed this period: {{completed_work}}
Work in progress: {{in_progress}}
Planned for next period: {{planned_next}}
Blockers or risks: {{blockers}}
Key metrics: {{metrics}}

Rules:
- Write in past tense for completed work, present progressive for in-progress
- Flag blockers clearly, not buried in the middle
- Match length to the audience: executives get a 3-paragraph summary; operational teams get more detail
- No em dashes
- No corporate filler phrases ("synergy", "moving the needle", "circle back")
- Output as plain text with clear section headers

Write the status report:
```

---

### SOP Generator from Conversation Transcript

```
You are a process documentation specialist. Turn this conversation transcript or meeting notes into a clean Standard Operating Procedure (SOP).

Process name: {{process_name}}
Context: {{context}}
Transcript or notes: {{transcript}}

Return JSON only:
{
  "sop_title": "<SOP title>",
  "purpose": "<1-2 sentences: what this process achieves and why it matters>",
  "scope": "<who this applies to and when>",
  "prerequisites": ["<what must be in place before starting>"],
  "steps": [
    {
      "step_number": <number>,
      "title": "<short step title>",
      "description": "<what to do and how to do it>",
      "notes": "<gotchas, tips, or edge cases if any>",
      "responsible_role": "<who does this step>"
    }
  ],
  "success_criteria": "<how you know the process was completed correctly>",
  "related_tools": ["<tool or system used>"],
  "review_date": "<suggested next review date, e.g., '6 months from creation'>"
}

Rules:
- Number steps sequentially
- Each step should be atomic (one clear action)
- Use plain language. Assume the reader is competent but unfamiliar with this specific process.
- Do not invent steps that were not implied or stated in the source material
```

---

## PRODUCT

---

### User Feedback Synthesis

```
Synthesize a batch of user feedback into actionable product insights.

Product name: {{product_name}}
Feedback source: {{source}} (e.g., NPS survey, support tickets, user interviews, app reviews)
Time period: {{period}}
Raw feedback (one entry per line or as array): {{feedback_data}}

Return JSON only:
{
  "total_entries_analyzed": <number>,
  "overall_sentiment": "<positive | mixed | negative>",
  "sentiment_breakdown": {
    "positive": <percentage>,
    "neutral": <percentage>,
    "negative": <percentage>
  },
  "top_themes": [
    {
      "theme": "<theme name>",
      "frequency": "<how often mentioned: high | medium | low>",
      "sentiment": "<positive | negative | neutral>",
      "representative_quote": "<direct quote from feedback>",
      "product_implication": "<what this means for the product team>"
    }
  ],
  "most_requested_features": ["<feature>", "<feature>"],
  "most_common_complaints": ["<complaint>", "<complaint>"],
  "quick_wins": ["<small change that would address a recurring complaint>"],
  "executive_summary": "<3-4 sentences summarizing the most important findings>"
}
```

---

### Feature Request Prioritization

```
Prioritize a list of feature requests using a structured framework.

Product name: {{product_name}}
Company stage: {{stage}} (e.g., early startup, growth, enterprise)
Current strategic priority: {{strategic_priority}}
Feature requests: {{feature_requests}}

For each feature request, score it on:
- Customer impact: how many customers benefit and how much (1-5)
- Strategic alignment: how well it fits current priorities (1-5)
- Effort estimate: rough implementation effort (1=trivial, 5=major)
- Revenue impact: potential revenue uplift or retention impact (1-5)

Return JSON only:
{
  "prioritized_features": [
    {
      "feature": "<feature name>",
      "customer_impact": <1-5>,
      "strategic_alignment": <1-5>,
      "effort": <1-5>,
      "revenue_impact": <1-5>,
      "priority_score": <calculated score: (customer_impact + strategic_alignment + revenue_impact) / effort>,
      "recommendation": "<build now | build next quarter | backlog | decline>",
      "reasoning": "<one sentence>"
    }
  ],
  "top_3_to_build": ["<feature>", "<feature>", "<feature>"],
  "notable_declines": ["<feature and reason to decline>"]
}

Sort the output by priority_score descending.
```

---

### Release Notes Writer

```
Write user-facing release notes for a product update.

Version number: {{version}}
Release date: {{release_date}}
Changes (raw technical list): {{changes}}
Target audience: {{audience}} (end users, developers, admins)
Product name: {{product_name}}

Rules:
- Translate technical change descriptions into user-facing benefit language
- Group changes into: New Features, Improvements, Bug Fixes
- Use plain language. Avoid jargon unless writing for developers.
- Each item should start with a verb in past tense (Added, Fixed, Improved, Removed)
- Do not use em dashes
- Flag any breaking changes clearly with a "Breaking:" prefix
- Output as plain text with section headers

Write the release notes:
```

---

## CONTENT

---

### Blog Post Outline

```
Create a detailed outline for a blog post.

Topic: {{topic}}
Target keyword: {{target_keyword}}
Target audience: {{audience}}
Desired word count: {{word_count}}
Angle or unique perspective: {{angle}}
Goal of the post: {{goal}} (e.g., drive organic traffic, generate leads, educate customers)

Return JSON only:
{
  "working_title": "<SEO-optimized title including the target keyword>",
  "meta_description": "<155-character meta description>",
  "intro_hook": "<one sentence opening line that earns the reader's attention>",
  "sections": [
    {
      "heading": "<H2 or H3 heading>",
      "heading_type": "H2",
      "key_points": ["<point to make>", "<point to make>"],
      "suggested_word_count": <number>,
      "include_example": <true | false>
    }
  ],
  "conclusion_angle": "<what the conclusion should leave the reader with>",
  "cta": "<what action should the reader take after reading>",
  "internal_link_opportunities": ["<related topic to link to>"],
  "estimated_total_word_count": <number>
}
```

---

### Email Newsletter Section

```
Write one section of an email newsletter.

Newsletter name: {{newsletter_name}}
Audience: {{audience}}
Section topic: {{topic}}
Key insight or story: {{insight}}
Tone: {{tone}} (casual, professional, educational)
Link to include: {{link}}
Link anchor text: {{anchor_text}}

Rules:
- Write in the author's voice, not as a brand
- Start with the insight or story, not a context-setting intro
- One paragraph, then a clear link with anchor text
- Under 120 words
- No em dashes
- No generic openers ("This week...", "In today's newsletter...")

Write the section:
```

---

### YouTube Script Outline

```
Create a script outline for a YouTube video.

Video topic: {{topic}}
Target audience: {{audience}}
Video length target: {{target_length}} (e.g., 8-10 minutes)
Goal: {{goal}} (e.g., educate, drive to website, build authority)
Channel style: {{channel_style}} (e.g., talking head, tutorial, interview-style)

Return JSON only:
{
  "video_title": "<click-worthy title, under 70 characters>",
  "thumbnail_text_suggestion": "<3-5 words for thumbnail overlay>",
  "hook": "<opening 30 seconds: what will you say or show to keep viewers watching?>",
  "sections": [
    {
      "section_title": "<internal label>",
      "duration_estimate": "<e.g., 90 seconds>",
      "talking_points": ["<point>", "<point>"],
      "b_roll_or_visual_suggestion": "<what to show on screen>"
    }
  ],
  "cta": "<what to ask viewers to do at the end>",
  "pattern_interrupts": ["<moment to add energy or change pace>"],
  "description_first_paragraph": "<YouTube description opener, 2-3 sentences, include target keyword>"
}
```

---

### LinkedIn Post

```
Write a LinkedIn post designed to get engagement and build authority.

Author background: {{author_background}}
Topic: {{topic}}
Core insight or contrarian take: {{insight}}
Target audience: {{audience}}
Include CTA: {{include_cta}} (yes/no)

Rules:
- First line must work as a hook before the "see more" cutoff (under 140 characters, no em dash, no period)
- Short paragraphs: 1-2 sentences each, with line breaks between
- Write from personal experience or a specific observation, not generic advice
- No listicle structure unless the format genuinely serves the content
- No em dashes
- No AI-sounding openers ("In today's world", "As a [title]", "I'm excited to share")
- Max 3 hashtags, placed at the very end if used at all
- Under 300 words total
- Return the post text only

Write the post:
```

---

## WORKFLOW-SPECIFIC UTILITY PROMPTS

---

### AI Proposal Sections

**Executive Summary Section:**
```
Write the executive summary section of a business proposal.

Prospect company: {{company_name}}
Industry: {{industry}}
Their stated problem: {{problem}}
Your proposed solution: {{solution}}
Deal value: {{deal_value}}

Rules:
- Write from the perspective of the vendor addressing the prospect
- Lead with the prospect's problem, not your company's capabilities
- 2-3 paragraphs maximum
- No em dashes
- No generic consulting language ("leverage", "synergies", "holistic approach")

Write the executive summary:
```

**Our Approach Section:**
```
Write the "Our Approach" section of a business proposal.

Project type: {{project_type}}
Deliverables: {{deliverables}}
Timeline: {{timeline}}
Key milestones: {{milestones}}
Our methodology: {{methodology}}

Rules:
- Explain what you will do, in what order, and why that order makes sense
- Be specific enough that the prospect knows what they are buying
- Do not pad with vague process language
- Use numbered phases if the project has clear phases
- Under 300 words
- No em dashes

Write the section:
```

---

### Stale Deal Slack Alert

```
Write a brief Slack alert message for a deal that has gone stale.

Deal name: {{deal_name}}
Company: {{company_name}}
Deal owner: {{owner_name}}
Deal value: {{deal_value}}
Current stage: {{deal_stage}}
Days since last activity: {{days_stale}}
Last activity description: {{last_activity}}

Rules:
- Be direct and useful, not alarming
- Include one specific suggested action for the rep
- Under 60 words
- Plain text only (no markdown formatting)
- Do not use em dashes

Write the Slack message:
```

---

### Voicemail / Call Transcription Summary

```
Summarize this phone call transcription for a CRM note.

Transcription: {{transcription}}
Caller phone: {{caller_phone}}
Call timestamp: {{call_timestamp}}

Return JSON only:
{
  "caller_intent": "<what did the caller want or need?>",
  "urgency": "<low | medium | high>",
  "key_details": ["<specific detail mentioned>", "<specific detail mentioned>"],
  "action_required": "<what the recipient should do next>",
  "crm_note": "<clean 2-4 sentence note to log in the CRM, written in third person (e.g., 'Caller asked about...')>",
  "suggested_callback_priority": "<within 1 hour | same day | within 48 hours>"
}
```

---

### Abandoned Cart Recovery Email (Per Touch)

```
Write touch {{touch_number}} of 3 in an abandoned cart email recovery sequence.

Customer name: {{customer_name}}
Abandoned products: {{products}}
Cart total: {{cart_total}}
Checkout URL: {{checkout_url}}
Hours since abandonment: {{hours_since_abandonment}}
Discount code (only for touch 3): {{discount_code}}

Touch angles:
- Touch 1 (sent at 1 hour): Helpful reminder, no pressure
- Touch 2 (sent at 24 hours): Light urgency, "your cart is still waiting"
- Touch 3 (sent at 72 hours): Offer a discount code, final nudge

Rules:
- Reference the specific products by name
- Touch 1 and 2: no discount, just the checkout link
- Touch 3 only: include the discount code clearly
- Under 100 words per email
- No em dashes
- Return email body only, no subject line
```

---

### Content Repurposing: Blog Post to LinkedIn

```
Repurpose this blog post as a native LinkedIn article post.

Blog post title: {{post_title}}
Blog post content: {{post_content}}
Author: {{author_name}}
Blog URL: {{post_url}}

Rules:
- Extract the single most interesting or contrarian insight from the post
- Write as a thought leadership post, not a content promotion post
- Do not start with "I wrote a blog post about..."
- First line is the hook (under 140 characters, no em dash)
- Short paragraphs with line breaks
- End with a question to drive comments
- Include the blog URL as "Full post: [url]" at the very end
- Under 300 words
- No em dashes

Write the LinkedIn post:
```

---

### Content Repurposing: Blog Post to Twitter/X Thread

```
Turn this blog post into a Twitter/X thread.

Blog post title: {{post_title}}
Blog post content: {{post_content}}
Blog URL: {{post_url}}
Number of tweets in thread: {{thread_length}}

Rules:
- Tweet 1 is the hook: make a bold claim or state a counterintuitive insight. Must work as a standalone tweet.
- Each subsequent tweet expands one idea
- Last tweet: link to the full post + a clear CTA
- Number each tweet (1/{{thread_length}}, 2/{{thread_length}}, etc.)
- Each tweet must be under 280 characters
- No em dashes
- No filler transitions ("So...", "Now...", "In summary...")

Return JSON only:
{
  "thread": [
    {
      "tweet_number": <number>,
      "text": "<tweet text including numbering>"
    }
  ]
}
```
