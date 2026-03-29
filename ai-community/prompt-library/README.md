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

---

## n8n WORKFLOW ENGINEERING

Prompts for building, documenting, and debugging n8n workflows via AI.

---

### Workflow Description Generator

```
Write a clear, professional description for an n8n workflow.

Workflow name: {{workflow_name}}
Trigger type: {{trigger}} (e.g., Webhook, Schedule, HubSpot contact created)
What it does step by step: {{workflow_steps}}
Tools/integrations used: {{integrations}}
Business outcome it achieves: {{outcome}}

Return JSON only:
{
  "short_description": "<one sentence: what triggers it and what it produces>",
  "full_description": "<3-5 sentences explaining the full workflow from trigger to output>",
  "use_cases": ["<when would you use this>", "<when would you use this>"],
  "required_credentials": ["<service name>", "<service name>"],
  "estimated_run_time": "<fast (under 5s) | medium (5-30s) | slow (30s+)>",
  "error_points": ["<where this workflow is most likely to fail>"]
}
```

---

### n8n Code Node: Data Transform

```
Write a JavaScript function for an n8n Code node.

Input data structure: {{input_data}}
Transformation needed: {{transformation_description}}
Output data structure expected by the next node: {{output_structure}}

Rules:
- Use the n8n Code node format: items comes in as the `items` array
- Access input fields via `items[0].json.fieldName`
- Return array format: `return [{ json: { ... } }]` or multiple items
- Handle null/undefined values defensively
- Add a comment explaining each major step
- Do not use external libraries (only built-in Node.js methods)

Return the JavaScript code only, no explanation:
```

---

### n8n Error Handler Message

```
Write an error notification message for an n8n error handler workflow.

Workflow name: {{workflow_name}}
Error message: {{error_message}}
Node that failed: {{failed_node}}
Input data that caused the failure: {{input_data}}
Timestamp: {{timestamp}}
Environment: {{environment}} (production / staging)

Rules:
- Format as a Slack message block (plain text, not rich formatting)
- Include: what failed, when, what the input was, and what someone should check first
- Under 200 characters for the main alert line
- Include a separate "Debug info" section with full error details
- No em dashes

Return JSON only:
{
  "alert_line": "<short Slack alert message>",
  "debug_block": "<multiline debug information formatted for Slack code block>",
  "suggested_fix": "<one sentence on what to investigate first>"
}
```

---

### Workflow Audit Checklist Generator

```
Generate a pre-deployment audit checklist for an n8n workflow.

Workflow name: {{workflow_name}}
Trigger type: {{trigger}}
Integrations used: {{integrations}}
Data it handles: {{data_description}} (e.g., customer PII, financial data, public data)
Is this workflow idempotent: {{idempotent}} (yes/no/unknown)

Return JSON only:
{
  "checklist": [
    {
      "category": "<Security | Error Handling | Performance | Data Integrity | Testing>",
      "item": "<specific thing to check>",
      "why_it_matters": "<one sentence>",
      "status": "pending"
    }
  ],
  "critical_items": ["<items that must pass before deploying>"],
  "notes": "<any workflow-specific risks to flag>"
}
```

---

### Webhook Payload Parser Prompt

```
Parse an incoming webhook payload and extract the fields needed for downstream processing.

Payload source: {{source}} (e.g., Stripe, GitHub, Typeform, HubSpot)
Raw payload: {{raw_payload}}
Fields I need: {{required_fields}}

Return JSON only — extract only the fields listed in "Fields I need". Flatten nested objects into top-level keys. If a field is missing from the payload, set it to null.

Apply these transforms:
- Dates: convert to ISO 8601 format
- Amounts: ensure they are numbers, not strings
- Boolean strings ("true"/"false"): convert to actual booleans

Return:
{
  "extracted": {
    "<field_name>": <value>
  },
  "missing_fields": ["<field that was null or not found>"],
  "parse_warnings": ["<any type mismatches or unexpected formats>"]
}
```

---

### Cron Schedule Plain-English Explainer

```
Explain this cron schedule in plain English and list the next 5 run times.

Cron expression: {{cron_expression}}
Timezone: {{timezone}}
Current time: {{current_time}}

Return JSON only:
{
  "plain_english": "<what this schedule means in plain language>",
  "frequency": "<e.g., every weekday at 9am>",
  "next_5_runs": ["<ISO timestamp>", "<ISO timestamp>", "<ISO timestamp>", "<ISO timestamp>", "<ISO timestamp>"],
  "potential_issues": "<any concerns about this schedule, e.g., runs during likely peak hours, or too frequently>"
}
```

---

### Workflow to API Documentation

```
Generate API-style documentation for an n8n webhook workflow.

Workflow name: {{workflow_name}}
Webhook URL: {{webhook_url}}
HTTP method: {{method}}
Expected request headers: {{headers}}
Expected request body schema: {{request_body}}
What the workflow does with the request: {{processing_steps}}
Response body the workflow sends back: {{response_body}}
Authentication required: {{auth_type}}

Output as plain text documentation in the format of a REST API reference page:
- Endpoint
- Method
- Description
- Authentication
- Request headers table
- Request body schema with field descriptions and types
- Example request (curl)
- Response schema
- Example response
- Error codes
```

---

### Loop Performance Estimator

```
Estimate performance and cost for an n8n loop-based workflow.

Loop node input: {{number_of_items}} items
Operations per item: {{operations}} (e.g., "1 HTTP request to OpenAI, 1 HubSpot PATCH")
Average operation latency: {{avg_latency_ms}} ms per operation
API rate limits: {{rate_limits}} (e.g., "OpenAI: 60 req/min, HubSpot: 100 req/10s")
AI tokens per item (if applicable): {{tokens_per_item}}
AI model: {{model}} (e.g., gpt-4o, claude-3-5-haiku)
Token price per 1k: {{token_price}}

Return JSON only:
{
  "estimated_total_time_seconds": <number>,
  "estimated_total_time_human": "<e.g., 4 minutes 30 seconds>",
  "bottleneck": "<which API will rate-limit first>",
  "recommended_batch_delay_ms": <number>,
  "estimated_ai_cost_usd": <number>,
  "estimated_api_calls": <number>,
  "optimization_suggestions": ["<suggestion>", "<suggestion>"]
}
```

---

## HUBSPOT CRM OPERATIONS

Prompts for contact management, deal ops, reporting, and workflow logic in HubSpot.

---

### Contact Enrichment Summary

```
Enrich and summarize a HubSpot contact record for a sales rep.

Contact data:
Name: {{full_name}}
Email: {{email}}
Company: {{company}}
Job title: {{jobtitle}}
Source: {{hs_analytics_source}}
Original message: {{message}}
Page views: {{hs_analytics_num_page_views}}
Email opens: {{email_open_count}}
Lead score: {{ai_lead_score}}
Last activity: {{notes_last_updated}}

Write a 3-4 sentence contact brief a rep can read in 20 seconds before calling. Include:
- Who they are
- Why they reached out
- Their engagement level based on the data
- One specific recommended opener for the first call

Plain text, no headers, no bullet points, no em dashes.
```

---

### Deal Stage Move Justification

```
A deal just moved stages in HubSpot. Write an internal note explaining why.

Deal name: {{deal_name}}
Previous stage: {{previous_stage}}
New stage: {{new_stage}}
What triggered the move: {{trigger_event}}
CRM notes context: {{recent_notes}}
Deal owner: {{owner_name}}

Rules:
- Write as a CRM activity note in third person
- Explain what specifically happened to justify this stage progression
- Flag any risks or next steps
- Under 100 words
- No em dashes

Write the CRM note:
```

---

### HubSpot Workflow Logic Builder

```
Design the logic for a HubSpot contact-based workflow.

Goal: {{workflow_goal}}
Trigger: {{trigger_event}} (e.g., contact submits form, property changes, deal created)
Filters/enrollment criteria: {{filters}}
Available contact properties: {{available_properties}}
Available integrations: {{integrations}} (e.g., Slack, n8n webhook, email)

Return JSON only:
{
  "workflow_name": "<descriptive workflow name>",
  "trigger": {
    "type": "<form submission | property change | deal stage change | etc>",
    "details": "<specific trigger configuration>"
  },
  "enrollment_filters": [
    {"property": "<property>", "operator": "<equals | contains | is known | etc>", "value": "<value>"}
  ],
  "actions": [
    {
      "step": <number>,
      "action_type": "<Set property | Send email | Create task | Webhook | Delay>",
      "configuration": "<what to set/send/do>",
      "reason": "<why this step is needed>"
    }
  ],
  "exit_criteria": "<when should a contact leave this workflow>",
  "re_enrollment": "<can contacts re-enroll? Under what conditions?>"
}
```

---

### HubSpot Report Interpretation

```
Interpret a HubSpot report and write a plain-language summary for a non-technical team.

Report name: {{report_name}}
Report type: {{report_type}} (e.g., deal pipeline, email performance, contact activity)
Date range: {{date_range}}
Raw report data: {{report_data}}
Team receiving this: {{audience}} (e.g., sales team, exec team, marketing)

Rules:
- Lead with the single most important takeaway
- Compare to {{comparison_period}} if data is available
- Flag anything that requires action
- Under 200 words
- No jargon (explain any CRM-specific terms)
- No em dashes
- Plain text, no headers

Write the summary:
```

---

### Sequence Performance Review

```
Analyze the performance of a HubSpot or n8n email sequence.

Sequence name: {{sequence_name}}
Contacts enrolled: {{enrolled_count}}
Touch-by-touch data: {{touch_data}}
(Format: [{ touch: 1, sent: 100, opened: 45, clicked: 12, replied: 3, unsubscribed: 1 }, ...])
Goal of sequence: {{sequence_goal}}
Industry average open rate: {{industry_avg_open}}%

Return JSON only:
{
  "overall_health": "<strong | average | underperforming>",
  "best_performing_touch": <touch number>,
  "worst_performing_touch": <touch number>,
  "drop_off_point": "<where most contacts disengage>",
  "open_rate_vs_benchmark": "<above | below | at> benchmark by <X>%",
  "insights": ["<insight>", "<insight>", "<insight>"],
  "recommended_changes": [
    {
      "touch": <number>,
      "change": "<what to change>",
      "expected_impact": "<why this change should improve performance>"
    }
  ]
}
```

---

### CRM Data Hygiene Audit

```
Audit a batch of CRM contact records for data quality issues.

Records: {{contact_records_json}}
Required fields for your business: {{required_fields}}
Optional but important fields: {{optional_fields}}

For each record, flag:
- Missing required fields
- Likely duplicate emails (same domain, similar names)
- Invalid email formats
- Clearly fake entries (e.g., "test@test.com", "aaa@bbb.com")
- Job titles that look like spam (random strings, test data)

Return JSON only:
{
  "total_records": <number>,
  "clean_records": <number>,
  "records_with_issues": [
    {
      "contact_id": "<id>",
      "email": "<email>",
      "issues": ["<issue>"],
      "severity": "<critical | warning | minor>",
      "recommended_action": "<delete | merge | enrich | flag for review>"
    }
  ],
  "summary": {
    "missing_required_fields_count": <number>,
    "likely_duplicates_count": <number>,
    "invalid_emails_count": <number>,
    "fake_entries_count": <number>
  }
}
```

---

### Deal Forecast Commentary

```
Write a revenue forecast commentary for a sales pipeline.

Team: {{team_name}}
Period: {{forecast_period}}
Quota: {{quota}}
Pipeline data:
- Committed deals (>80% close probability): {{committed_value}}
- Best case deals (50-79%): {{best_case_value}}
- Pipeline total: {{pipeline_total}}
- Closed-won this period: {{closed_won}}
Top risks: {{risks}}

Rules:
- Write as a 3-paragraph sales leadership commentary
- Para 1: current position vs quota
- Para 2: key deals that will make or break the number
- Para 3: risks and what needs to happen
- Honest, direct tone — like something a VP would send to the CEO
- No em dashes, no corporate euphemisms

Write the commentary:
```

---

## PINECONE + VECTOR SEARCH

Prompts for query rewriting, result synthesis, and semantic search workflows.

---

### Semantic Search Query Rewriter

```
Rewrite a user's raw search query to improve semantic search results in a vector database.

Original query: {{raw_query}}
Index content type: {{index_description}} (e.g., "product descriptions", "support articles", "sales call transcripts")
User context: {{user_context}} (e.g., "sales rep looking for competitors mentioned in calls")

Generate 3 alternative query formulations that would surface different but relevant results:
- One more specific version
- One broader/more conceptual version
- One that uses different vocabulary to describe the same thing

Return JSON only:
{
  "original_query": "{{raw_query}}",
  "rewritten_queries": [
    {"variant": "specific", "query": "<rewritten>", "rationale": "<why this variant>"},
    {"variant": "broad", "query": "<rewritten>", "rationale": "<why this variant>"},
    {"variant": "alternative_vocabulary", "query": "<rewritten>", "rationale": "<why this variant>"}
  ],
  "recommended_query": "<which of the 4 to use and why>"
}
```

---

### Vector Search Result Synthesizer

```
Synthesize multiple vector search results into a single coherent answer.

User's original question: {{user_question}}
Search results (ranked by similarity): {{search_results}}
(Format: [{ id: "...", score: 0.92, text: "..." }, ...])
Minimum similarity threshold: {{min_score}}

Rules:
- Only use information from results with score >= {{min_score}}
- If no results meet the threshold, say "I don't have enough information to answer this confidently"
- Cite which results you're drawing from (by id or position)
- Synthesize, don't just concatenate — find the common thread
- Under 200 words
- No em dashes

Answer:
```

---

### Pinecone Namespace Strategy Planner

```
Design a Pinecone namespace strategy for a multi-tenant or multi-content-type application.

Application type: {{app_type}} (e.g., multi-tenant SaaS, knowledge base, product catalog)
Number of tenants / content types: {{scale}}
Data types to index: {{data_types}}
Typical query patterns: {{query_patterns}}
Expected data volume per namespace: {{data_volume}}

Return JSON only:
{
  "recommended_strategy": "<one namespace per tenant | content-type namespaces | flat with metadata filters | hybrid>",
  "namespace_schema": [
    {
      "namespace_name": "<naming convention>",
      "what_it_contains": "<description>",
      "example": "<example namespace name>"
    }
  ],
  "metadata_fields_to_index": [
    {"field": "<name>", "type": "<string | number | boolean>", "purpose": "<why filter on this>"}
  ],
  "query_pattern_examples": [
    {"use_case": "<when>", "namespace": "<which namespace>", "filter": "<metadata filter if any>"}
  ],
  "tradeoffs": "<brief note on what this strategy optimizes for and what it sacrifices>"
}
```

---

### Embedding Pre-processing Prompt

```
Prepare this raw text for embedding in a vector database.

Raw text: {{raw_text}}
Content type: {{content_type}} (e.g., support article, product description, email thread, meeting transcript)
Max chunk size: {{max_tokens}} tokens
Overlap between chunks: {{overlap_tokens}} tokens

Process:
1. Clean the text (remove HTML tags, excessive whitespace, email headers if present)
2. Split into semantically coherent chunks — do not split mid-sentence or mid-paragraph
3. For each chunk, prepend a context prefix to improve retrieval

Return JSON only:
{
  "chunks": [
    {
      "chunk_id": "<sequential id>",
      "text": "<chunk text with context prefix>",
      "estimated_tokens": <number>,
      "metadata": {
        "source_type": "<content_type>",
        "chunk_position": "<early | middle | late>",
        "key_topics": ["<topic>", "<topic>"]
      }
    }
  ],
  "total_chunks": <number>,
  "cleaning_notes": "<anything unusual that was removed or modified>"
}
```

---

### Semantic Similarity Lead Matcher

```
Use semantic similarity scores to match a new lead to the most similar past converted lead.

New lead profile:
Company: {{company}}
Industry: {{industry}}
Company size: {{size}}
Their message/inquiry: {{inquiry}}
Lead score: {{score}}

Top 5 similar past leads from vector search:
{{similar_leads_json}}
(Format: [{ id, similarity_score, company, industry, outcome, deal_value, time_to_close_days }])

Return JSON only:
{
  "best_match": {
    "lead_id": "<id>",
    "similarity_score": <number>,
    "why_similar": "<2-3 sentences explaining the match>"
  },
  "predicted_outcome": "<likely to convert | uncertain | unlikely to convert>",
  "predicted_deal_value_range": "<e.g., $5,000-$10,000>",
  "predicted_time_to_close_days": <number>,
  "recommended_approach": "<what worked with similar leads — what should the rep do differently or the same>",
  "confidence": "<high | medium | low>",
  "confidence_reason": "<one sentence>"
}
```

---

### RAG Answer Quality Evaluator

```
Evaluate whether a RAG (retrieval-augmented generation) response is grounded in the provided context.

User question: {{question}}
Retrieved context used: {{context}}
Generated answer: {{answer}}

Evaluate on:
1. Groundedness: Is every claim in the answer supported by the context?
2. Completeness: Does the answer address the full question given what's available?
3. Hallucination: Does the answer make any claims NOT present in the context?
4. Relevance: Is the context actually relevant to the question?

Return JSON only:
{
  "groundedness_score": <1-5>,
  "completeness_score": <1-5>,
  "hallucination_detected": <true | false>,
  "hallucinated_claims": ["<claim not in context>"],
  "relevance_score": <1-5>,
  "overall_quality": "<pass | needs_review | fail>",
  "improvement_suggestion": "<one specific thing to fix if quality is below pass>"
}
```

---

## GROQ / LLM INFERENCE

Prompts optimized for speed-first inference with Groq's API and fast models.

---

### System Prompt Generator for Groq Workflows

```
Write a system prompt for a Groq-powered AI agent used in an n8n automation workflow.

Agent role: {{agent_role}} (e.g., lead qualifier, email writer, data extractor)
Input it will receive: {{input_description}}
Output it must produce: {{output_description}}
Output format: {{output_format}} (JSON / plain text / markdown)
Constraints: {{constraints}} (e.g., under 200 words, no em dashes, return only the JSON)
Edge cases to handle: {{edge_cases}}

Rules for the system prompt you write:
- Be explicit about output format requirements — models follow explicit instructions
- Put output format instructions at the END of the system prompt (models attend more to recent tokens)
- Add a brief "If you are unsure..." instruction for edge cases
- Keep the system prompt under 400 tokens for Groq speed optimization
- No vague instructions — every sentence should constrain behavior, not just describe it

Write the system prompt:
```

---

### Groq Model Selector

```
Recommend the best Groq model for a given automation use case.

Task description: {{task_description}}
Input length: {{avg_input_tokens}} tokens typical
Output length: {{avg_output_tokens}} tokens typical
Latency requirement: {{latency}} (real-time <1s | fast <3s | batch acceptable)
Volume: {{requests_per_day}} requests/day
Budget sensitivity: {{budget}} (cost-critical | balanced | quality-first)
Quality threshold: {{quality}} (high-stakes decision | medium | informational / low-stakes)

Groq models available (as of 2025):
- llama-3.3-70b-versatile: Best general quality, 70B, ~200 tok/s
- llama-3.1-8b-instant: Fastest, lowest cost, 8B, ~800 tok/s
- mixtral-8x7b-32768: Long context (32k), mixture of experts
- gemma2-9b-it: Fast, Google's model, good instruction following
- llama-3.2-11b-vision-preview: Multimodal, for image inputs

Return JSON only:
{
  "recommended_model": "<model id>",
  "reasoning": "<2-3 sentences on why this model fits the requirements>",
  "alternative_model": "<second best option>",
  "alternative_reasoning": "<when to use the alternative instead>",
  "estimated_cost_per_1k_requests_usd": <number>,
  "expected_latency_ms": <number>,
  "caveats": "<anything to watch out for with this choice>"
}
```

---

### Structured Extraction Prompt (Groq-Optimized)

```
Extract structured data from unstructured text. Optimized for fast Groq inference.

Text to process: {{raw_text}}
Extract these fields: {{fields_to_extract}}

CRITICAL INSTRUCTIONS:
- Return valid JSON only. No preamble, no explanation, no markdown code fences.
- If a field cannot be found, use null — never guess or infer.
- For arrays, return [] if empty, not null.
- For dates, use ISO 8601 format (YYYY-MM-DD).
- For numbers, return as numeric type, not string.

Schema:
{{output_schema}}

Return the JSON now:
```

---

### LLM Response Validator

```
Validate whether an LLM's JSON response is well-formed and matches the expected schema.

Expected schema: {{expected_schema}}
LLM raw response: {{llm_response}}

Check for:
1. Valid JSON (parseable without errors)
2. All required fields present
3. Correct data types for each field
4. No extra fields that could break downstream parsing
5. String fields within expected length bounds (if specified)
6. Enum fields only contain allowed values (if specified)

Return JSON only:
{
  "is_valid": <true | false>,
  "parse_error": "<JSON parse error message if invalid JSON, else null>",
  "missing_fields": ["<field>"],
  "type_mismatches": [{"field": "<field>", "expected": "<type>", "got": "<type>"}],
  "extra_fields": ["<unexpected field>"],
  "enum_violations": [{"field": "<field>", "value": "<invalid value>", "allowed": ["<allowed>"]}],
  "corrected_response": "<attempt to fix the response if fixable, else null>"
}
```

---

### Prompt Version Comparison

```
Compare two versions of a prompt and recommend which to use in production.

Task the prompt performs: {{task_description}}
Prompt A: {{prompt_a}}
Prompt B: {{prompt_b}}
Test input: {{test_input}}

Evaluate each prompt on:
- Clarity of instructions (will the model understand what to do?)
- Output format specificity (are format requirements explicit enough?)
- Edge case handling (what happens with unusual inputs?)
- Token efficiency (is it concise without sacrificing clarity?)
- Potential failure modes (what could go wrong?)

Return JSON only:
{
  "prompt_a_scores": {
    "clarity": <1-5>,
    "format_specificity": <1-5>,
    "edge_case_handling": <1-5>,
    "token_efficiency": <1-5>,
    "failure_risk": <1-5>
  },
  "prompt_b_scores": {
    "clarity": <1-5>,
    "format_specificity": <1-5>,
    "edge_case_handling": <1-5>,
    "token_efficiency": <1-5>,
    "failure_risk": <1-5>
  },
  "recommended_prompt": "<A | B>",
  "reasoning": "<2-3 sentences>",
  "suggested_hybrid": "<if neither is ideal, what to take from each>"
}
```

---

### Batch Classification Prompt

```
Classify a batch of items into predefined categories. Return results as a JSON array.

Items to classify: {{items_array}}
Categories: {{categories}}
Classification criteria: {{criteria}}

For each item, assign:
- The single best-fit category
- A confidence score (0.0 to 1.0)
- A one-word reason

Return JSON only — no preamble, no markdown:
{
  "classifications": [
    {
      "id": "<item id or index>",
      "item": "<original item text>",
      "category": "<category>",
      "confidence": <0.0-1.0>,
      "reason": "<one word>"
    }
  ],
  "low_confidence_items": ["<ids where confidence < 0.6>"]
}
```

---

## FRAMER + DESIGN

Prompts for Framer components, design systems, animations, and CMS content.

---

### Framer Component Spec Generator

```
Generate a technical specification for a Framer code component.

Component name: {{component_name}}
What it does: {{component_description}}
Variants needed: {{variants}} (e.g., size: sm/md/lg, state: default/hover/active/disabled)
Props it should accept: {{props}}
Design system tokens to use: {{design_tokens}} (colors, spacing, typography)
Animation behavior: {{animation}} (e.g., fade in on mount, hover scale, click ripple)
Responsive behavior: {{responsive}}

Return JSON only:
{
  "component_name": "<PascalCase name>",
  "file_name": "<kebab-case>.tsx",
  "props_interface": [
    {
      "prop_name": "<name>",
      "type": "<TypeScript type>",
      "required": <true | false>,
      "default_value": "<default or null>",
      "description": "<what this prop controls>"
    }
  ],
  "property_controls": [
    {
      "prop": "<prop_name>",
      "control_type": "<Boolean | String | Number | Enum | Color | File>",
      "title": "<Framer property panel label>",
      "options": ["<enum option>"]
    }
  ],
  "animation_spec": "<Framer Motion implementation notes>",
  "accessibility_requirements": ["<ARIA roles>", "<keyboard support>"],
  "usage_example": "<JSX usage snippet>"
}
```

---

### Framer CMS Collection Schema Designer

```
Design a Framer CMS collection schema for a content type.

Content type: {{content_type}} (e.g., blog posts, team members, case studies, products)
Website purpose: {{website_purpose}}
Fields you know you need: {{known_fields}}
How this content will be displayed: {{display_contexts}} (e.g., grid cards on /blog, full post at /blog/[slug], featured on homepage)

Return JSON only:
{
  "collection_name": "<collection name>",
  "slug_field": "<which field to use as URL slug>",
  "fields": [
    {
      "name": "<field name>",
      "type": "<Text | Number | Boolean | Date | Image | Link | Color | File | Reference>",
      "required": <true | false>,
      "purpose": "<what this field is for>",
      "display_context": ["<where this field is shown>"]
    }
  ],
  "recommended_filters": ["<fields worth filtering/sorting by in Framer CMS>"],
  "seo_fields": ["<which fields map to page title, description, og:image>"],
  "notes": "<anything to be aware of about Framer CMS limitations for this use case>"
}
```

---

### Design System Token Audit

```
Audit a set of design tokens and identify inconsistencies, redundancies, and gaps.

Token set: {{tokens_json}}
(Format: { colors: {...}, spacing: {...}, typography: {...}, shadows: {...}, radii: {...} })
Design framework: {{framework}} (Tailwind / CSS variables / Framer tokens / custom)
Brand guidelines: {{brand_guidelines}}

Return JSON only:
{
  "issues": [
    {
      "category": "<Colors | Spacing | Typography | Shadows | Radii>",
      "issue_type": "<redundancy | inconsistency | gap | naming>",
      "description": "<specific issue>",
      "affected_tokens": ["<token name>"],
      "suggested_fix": "<what to change>"
    }
  ],
  "redundant_tokens": ["<tokens that could be removed>"],
  "missing_tokens": ["<tokens that should exist but don't>"],
  "naming_issues": ["<tokens with unclear or inconsistent names>"],
  "overall_health": "<well-structured | needs cleanup | significant issues>",
  "priority_fixes": ["<3 most important changes to make first>"]
}
```

---

### Animation Concept to Framer Motion Code

```
Convert an animation concept description into Framer Motion implementation notes.

Animation concept: {{animation_description}} (e.g., "cards slide up and fade in staggered when the section scrolls into view")
Target element: {{element}} (e.g., grid of 6 cards, hero heading, navigation menu)
Trigger: {{trigger}} (scroll into view / hover / click / page load / state change)
Duration preference: {{duration}} (snappy <0.3s / standard 0.3-0.6s / dramatic >0.6s)
Easing preference: {{easing}} (ease-out / spring / ease-in-out / bounce)
Must respect prefers-reduced-motion: true

Return JSON only:
{
  "framer_motion_approach": "<whileInView | animate | variants | AnimatePresence | layout>",
  "parent_config": "<motion props for the parent element if needed>",
  "child_config": "<motion props for child elements>",
  "variants_object": "<variants object as code string if using variants pattern>",
  "stagger_config": "<staggerChildren and delayChildren values if applicable>",
  "reduced_motion_fallback": "<what to do when prefers-reduced-motion is true>",
  "implementation_notes": "<gotchas or tips specific to this animation>",
  "example_jsx": "<minimal JSX showing how to apply this>"
}
```

---

### Framer Page SEO Content Generator

```
Generate SEO content for a Framer-powered web page.

Page type: {{page_type}} (homepage / service page / blog post / case study / landing page)
Business: {{business_name}}
Core offering: {{offering}}
Target keyword: {{primary_keyword}}
Supporting keywords: {{secondary_keywords}}
Audience: {{target_audience}}
Page URL: {{page_url}}

Return JSON only:
{
  "page_title": "<60 characters max, keyword near front>",
  "meta_description": "<155-160 characters, compelling, includes keyword>",
  "h1": "<page heading, keyword-optimized>",
  "h2_suggestions": ["<section heading>", "<section heading>", "<section heading>"],
  "og_title": "<open graph title for social sharing>",
  "og_description": "<open graph description, under 200 characters>",
  "twitter_card_type": "summary_large_image",
  "schema_type": "<WebPage | Service | BlogPosting | FAQPage | LocalBusiness>",
  "canonical_url": "{{page_url}}",
  "internal_link_opportunities": ["<page on site worth linking from or to>"],
  "keyword_density_note": "<target usage count for primary keyword in body copy>"
}
```

---

## CLAUDE / AI AGENT OPERATIONS

Prompts for Claude Code skills, agent orchestration, code review, and AI ops.

---

### Claude Code Skill Writer

```
Write a Claude Code skill file (SKILL.md) for a reusable technique or workflow.

Skill name: {{skill_name}} (kebab-case)
What the skill teaches: {{skill_description}}
When it applies: {{when_to_use}}
The core technique or process: {{core_technique}}
Common mistakes to prevent: {{common_mistakes}}
Code example to include (if applicable): {{code_example}}

Follow this exact format:

---
name: {{skill_name}}
description: Use when {{when_to_use_short}}
---

# [Skill Name]

## Overview
[Core principle in 1-2 sentences]

## When to Use
[Bullet list of triggering scenarios]
[When NOT to use]

## Core Pattern
[The technique with code example if applicable]

## Common Mistakes
[Table of mistake + fix]

Write the full SKILL.md content:
```

---

### Agent Task Decomposer

```
Break down a complex task into parallel and sequential subtasks for a multi-agent workflow.

Task: {{task_description}}
Available agent types: {{agent_types}} (e.g., researcher, coder, writer, reviewer)
Constraints: {{constraints}} (e.g., must finish within 3 minutes, cannot access external APIs)

Return JSON only:
{
  "task_summary": "<one sentence description of the full task>",
  "phases": [
    {
      "phase": <number>,
      "can_run_parallel": <true | false>,
      "subtasks": [
        {
          "task_id": "<id>",
          "agent_type": "<which agent type>",
          "instruction": "<specific instruction for this agent>",
          "depends_on": ["<task_id of prerequisite, if any>"],
          "expected_output": "<what this agent should return>",
          "estimated_time_seconds": <number>
        }
      ]
    }
  ],
  "final_synthesis_needed": <true | false>,
  "synthesis_instruction": "<how to combine outputs from all agents, if needed>"
}
```

---

### Code Review Request Formatter

```
Format a code review request for the Claude Code code-reviewer agent.

What was built: {{feature_description}}
Files changed: {{files_changed}}
PR or diff: {{diff_or_code}}
Specific concerns: {{concerns}}
Context on the codebase: {{codebase_context}}
Review focus areas: {{focus_areas}} (e.g., security, performance, maintainability)

Write a structured code review request that instructs the reviewer to:
1. Check for bugs and logic errors in the diff
2. Flag security vulnerabilities (injection, auth, data exposure)
3. Review for adherence to: {{coding_standards}}
4. Check test coverage for the changed code
5. Note any performance concerns
6. Rate overall code quality (1-5) with one sentence reason

Format as a prompt for the reviewer agent, not as a question to the user.
```

---

### AI Output Quality Rubric Generator

```
Create a quality evaluation rubric for AI-generated content in a specific workflow.

Workflow name: {{workflow_name}}
AI task: {{ai_task}} (e.g., write first-response emails, qualify leads, extract data)
What good output looks like: {{good_output_criteria}}
What bad output looks like: {{bad_output_criteria}}
How it's used downstream: {{downstream_use}}

Return JSON only:
{
  "rubric_name": "<descriptive name>",
  "dimensions": [
    {
      "dimension": "<what is being evaluated>",
      "weight_percent": <number — all weights should sum to 100>,
      "score_5_criteria": "<what a perfect score looks like>",
      "score_3_criteria": "<what an acceptable score looks like>",
      "score_1_criteria": "<what a failing score looks like>"
    }
  ],
  "auto_fail_conditions": ["<condition that automatically fails the output regardless of score>"],
  "evaluation_prompt": "<a prompt you can use to have Claude score outputs against this rubric>"
}
```

---

### Prompt Debugging Assistant

```
Debug why an AI prompt is producing bad outputs.

Prompt: {{prompt}}
Model used: {{model}}
Input that caused the bad output: {{bad_input}}
Bad output received: {{bad_output}}
Expected output: {{expected_output}}

Diagnose the root cause. Check for:
- Ambiguous instructions that could be interpreted multiple ways
- Missing constraints or output format specification
- Instructions placed in ineffective positions (critical rules buried mid-prompt)
- Conflicting instructions that cancel each other out
- Instructions that are too vague to act on
- Missing examples when the format is non-standard

Return JSON only:
{
  "root_cause": "<primary reason for the bad output>",
  "secondary_causes": ["<additional contributing factors>"],
  "specific_problem_lines": ["<exact lines from the prompt causing issues>"],
  "fixes": [
    {
      "problem": "<what's wrong>",
      "fix": "<what to change and how>",
      "priority": "<critical | important | nice to have>"
    }
  ],
  "revised_prompt_snippet": "<show the revised version of the problematic section only>"
}
```

---

## AUTOMATION + DATA PROCESSING

Prompts for data transformation, pipeline validation, API integration, and reporting.

---

### API Response Normalizer

```
Normalize API responses from different sources into a consistent internal schema.

Source API: {{api_name}}
Raw response: {{raw_response}}
Target schema: {{target_schema}}

Rules:
- Map source fields to target schema fields
- Handle null/missing values by setting them to the default in the target schema
- Convert data types as needed (date strings to ISO 8601, amounts to numbers)
- Flatten nested objects where the target schema is flat
- If a required target field cannot be derived from the source, flag it

Return JSON only:
{
  "normalized": {{target_schema_with_values}},
  "missing_required_fields": ["<field>"],
  "transformation_log": [
    {"source_field": "<name>", "target_field": "<name>", "transformation": "<what was done>"}
  ],
  "warnings": ["<any data quality concerns>"]
}
```

---

### Data Pipeline Health Check

```
Analyze a data pipeline run log and identify failures, slowdowns, or anomalies.

Pipeline name: {{pipeline_name}}
Run log: {{run_log}}
(Format: array of { step, status, duration_ms, records_in, records_out, error_message })
Expected records: {{expected_records}}
SLA in seconds: {{sla_seconds}}

Return JSON only:
{
  "overall_status": "<success | partial_failure | failure | degraded>",
  "total_duration_ms": <number>,
  "sla_met": <true | false>,
  "failed_steps": [
    {"step": "<name>", "error": "<error message>", "records_lost": <number>}
  ],
  "slow_steps": [
    {"step": "<name>", "duration_ms": <number>, "expected_ms": <number>}
  ],
  "data_loss": {
    "records_expected": <number>,
    "records_processed": <number>,
    "loss_percent": <number>
  },
  "root_cause": "<most likely cause of any issues>",
  "recommended_action": "<what to do next>"
}
```

---

### Webhook Signature Verifier Logic

```
Write the logic to verify a webhook signature for a given platform.

Platform: {{platform}} (Stripe / GitHub / HubSpot / Shopify / Typeform / Twilio)
Secret: {{webhook_secret}} (use placeholder "WEBHOOK_SECRET" in output)
Raw request body: description only — do not include actual sensitive data
Signature header: {{signature_header}}

Write the n8n Code node JavaScript to:
1. Get the raw body and signature from the request
2. Compute the expected HMAC signature using the secret
3. Compare using a timing-safe comparison
4. Return { verified: true } or throw an error

Include comments explaining each step. Use Node.js built-in `crypto` only (no external libraries).

Return the JavaScript code only:
```

---

### Daily Digest Compiler

```
Compile a daily digest from multiple data sources into a single briefing.

Digest name: {{digest_name}}
Recipient: {{recipient}}
Date: {{date}}
Data sections:
- New leads today: {{new_leads_data}}
- Deals updated: {{deals_updated_data}}
- Emails sent/opened: {{email_stats}}
- Errors or workflow failures: {{error_log}}
- Tasks due today: {{tasks_due}}

Rules:
- Lead with the most important signal (a hot lead, a big deal movement, or a critical error)
- Each section should be scannable in 10 seconds
- Flag anything requiring immediate action in bold (use *asterisks* for bold in plain text)
- Under 300 words total
- No em dashes
- Write as if a smart assistant is giving a morning briefing to a busy person

Write the digest:
```

---

### Multi-Source Data Merge Prompt

```
Merge data from multiple sources about the same entity into a single enriched record.

Entity type: {{entity_type}} (contact / company / deal)
Entity identifier: {{entity_id}}

Source 1 ({{source_1_name}}): {{source_1_data}}
Source 2 ({{source_2_name}}): {{source_2_data}}
Source 3 ({{source_3_name}}): {{source_3_data}}

Merge rules:
- For conflicting values, use the source with the most recent `updated_at` timestamp
- If no timestamp available, prefer sources in this order: {{source_priority_order}}
- Combine array fields (tags, interests) by taking the union
- Flag any significant conflicts for human review

Return JSON only:
{
  "merged_record": {
    <merged fields>
  },
  "field_sources": {
    "<field_name>": "<which source this value came from>"
  },
  "conflicts": [
    {
      "field": "<field>",
      "values": {"<source>": "<value>"},
      "resolution": "<which value was used and why>",
      "flag_for_review": <true | false>
    }
  ]
}
```

---

## RAYCAST EXTENSION DEVELOPMENT

Prompts for building, documenting, and testing Raycast extensions.

---

### Raycast Command Spec Generator

```
Generate a technical specification for a Raycast extension command.

Extension name: {{extension_name}}
Command name: {{command_name}}
What it does: {{command_description}}
Command type: {{command_type}} (List / Detail / Form / MenuBar)
Input required: {{input}} (e.g., search query, form fields, no input)
API or service it connects to: {{api}}
Preferences needed: {{preferences}} (e.g., API key, base URL, default workspace)

Return JSON only:
{
  "command_name": "<camelCase>",
  "title": "<Human-readable title shown in Raycast>",
  "description": "<One sentence shown in Raycast search>",
  "mode": "<view | no-view | menu-bar>",
  "preferences": [
    {
      "name": "<camelCase>",
      "type": "<textfield | password | checkbox | dropdown | appPicker | file | directory>",
      "title": "<Label in preferences UI>",
      "description": "<Help text>",
      "required": <true | false>,
      "default": "<default value or null>"
    }
  ],
  "key_components": ["<Raycast API components to use>"],
  "state_management": "<useState + useCachedState | useCachedPromise | useFetch | custom>",
  "error_handling_approach": "<how to handle API errors>",
  "keyboard_shortcuts": [
    {"key": "<key combo>", "action": "<what it does>"}
  ]
}
```

---

### Raycast Action Panel Designer

```
Design the action panel for a Raycast List or Detail view.

Command name: {{command_name}}
Selected item type: {{item_type}} (e.g., HubSpot contact, GitHub PR, n8n workflow)
Primary action: {{primary_action}} (this is the Enter key action)
Secondary actions needed: {{secondary_actions}}
Destructive actions: {{destructive_actions}}

Return JSON only:
{
  "actions": [
    {
      "title": "<action label>",
      "shortcut": "<modifier+key or null>",
      "icon": "<Raycast Icon name>",
      "is_primary": <true | false>,
      "is_destructive": <true | false>,
      "action_type": "<open_url | copy_to_clipboard | push_view | run_api | open_app | show_toast>",
      "implementation_note": "<what the action should do>"
    }
  ],
  "sections": [
    {
      "section_title": "<section label or null for no section>",
      "actions": ["<action title>"]
    }
  ]
}
```

---

### AI Command Prompt for Raycast

```
Write the AI prompt used inside a Raycast AI command.

Command purpose: {{command_purpose}} (e.g., summarize selected text, rewrite in a specific tone, extract action items)
Input source: {{input_source}} (selected text / clipboard / form input / active browser tab)
Output expected: {{output_type}} (replacement text / new text / list / JSON)
Tone/style constraints: {{constraints}}
Context Raycast provides: {{context_available}} (e.g., selected text is available as {selection})

Rules for the prompt you write:
- Use {selection} placeholder for Raycast-provided input
- Keep it under 200 tokens for fast response
- Output instructions must be specific (not "write a summary" but "summarize in 2-3 bullet points")
- No em dashes in generated content instruction
- Include a fallback instruction if input is empty or irrelevant

Write the Raycast AI command prompt:
```

---

### Extension README Writer

```
Write a README for a Raycast extension for the Raycast Store.

Extension name: {{extension_name}}
Author: {{author_name}}
What it does: {{description}}
Commands included: {{commands_list}}
Prerequisites: {{prerequisites}} (e.g., HubSpot account, API key from X)
Setup steps: {{setup_steps}}
Key features: {{features}}
Screenshots available: {{screenshots}} (yes/no)

Rules:
- Start with a one-sentence description of what the extension does
- Lead with the benefit, not the technical implementation
- Setup section should be numbered steps
- Under 400 words total
- No em dashes
- End with a "Feedback" line pointing to GitHub issues

Write the README:
```

---

## CUSTOMER INTELLIGENCE

Prompts for understanding customers from data signals, building personas, and segmentation.

---

### ICP Profile Builder

```
Build an Ideal Customer Profile (ICP) from a dataset of won deals.

Won deals data: {{won_deals_data}}
(Format: [{ company, industry, size, revenue_band, deal_value, time_to_close_days, product_tier, close_reason }])
Time period: {{period}}

Analyze patterns across won deals to define the ICP.

Return JSON only:
{
  "icp_summary": "<2-3 sentences describing who your ideal customer is>",
  "top_industries": [{"industry": "<name>", "win_rate_pct": <number>, "avg_deal_value": <number>}],
  "ideal_company_size": "<e.g., 51-200 employees>",
  "ideal_revenue_band": "<e.g., $1M-$10M>",
  "average_deal_value": <number>,
  "fastest_closing_segments": ["<segment description>"],
  "common_close_reasons": ["<reason>"],
  "red_flags_to_avoid": ["<profile that consistently loses>"],
  "scoring_weights": {
    "industry_match": "<high | medium | low>",
    "size_match": "<high | medium | low>",
    "revenue_match": "<high | medium | low>"
  },
  "one_line_icp": "<the ICP in one sentence for use in messaging>"
}
```

---

### Customer Persona Writer

```
Write a detailed buyer persona for a target customer segment.

Segment name: {{segment_name}}
Data available: {{segment_data}}
(Include: demographics, job titles, industries, common objections, buying triggers, preferred channels)
Product/service: {{product_name}}

Return JSON only:
{
  "persona_name": "<fictional first name for the persona>",
  "job_title": "<most common title>",
  "company_type": "<description>",
  "age_range": "<e.g., 35-50>",
  "goals": ["<professional goal>", "<professional goal>"],
  "frustrations": ["<pain point>", "<pain point>"],
  "buying_triggers": ["<what makes them start looking>"],
  "objections": ["<common objection and brief reframe>"],
  "preferred_channels": ["<LinkedIn | email | phone | referral>"],
  "decision_criteria": ["<what they evaluate when choosing a vendor>"],
  "deal_breakers": ["<what would make them not buy>"],
  "messaging_angle": "<the single most compelling message for this persona>",
  "avoid_saying": ["<phrases or approaches that turn this persona off>"]
}
```

---

### Segment Performance Comparison

```
Compare the performance of two customer segments to inform prioritization.

Segment A name: {{segment_a_name}}
Segment A data: {{segment_a_data}}
Segment B name: {{segment_b_name}}
Segment B data: {{segment_b_data}}

Metrics to compare (from data):
- Conversion rate (lead to customer)
- Average deal value
- Average sales cycle days
- LTV (if available)
- Churn rate (if available)
- NPS (if available)
- CAC (if available)

Return JSON only:
{
  "comparison_table": [
    {
      "metric": "<metric name>",
      "segment_a_value": "<value>",
      "segment_b_value": "<value>",
      "winner": "<A | B | tie>"
    }
  ],
  "overall_winner": "<A | B | too close to call>",
  "reasoning": "<2-3 sentences on which segment is more valuable to pursue and why>",
  "nuances": "<anything that complicates a simple winner/loser conclusion>",
  "recommended_resource_split": "<e.g., 70% effort to Segment A, 30% to Segment B>"
}
```

---

### Review / Testimonial Request Email

```
Write an email requesting a review or testimonial from a satisfied customer.

Customer name: {{customer_name}}
Product/service: {{product_name}}
How long they have been a customer: {{tenure}}
Specific win or success they achieved: {{customer_win}}
Where you want the review: {{review_platform}} (Google / G2 / Clutch / Trustpilot / LinkedIn)
Review URL: {{review_url}}

Rules:
- Reference their specific win (not generic praise)
- Make the ask feel natural, not transactional
- Give them an easy path: one link, one action
- Optional: offer a small thank-you without it feeling like a bribe
- Under 120 words
- No em dashes
- Return email body only
```

---
