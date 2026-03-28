# Prompt Library

Prompts used in production automation workflows. Copy them directly into n8n HTTP Request nodes, OpenAI nodes, or any API client.

All prompts follow these rules:
- Return structured output where needed (JSON for machine-readable responses)
- No em dashes in generated content
- Human tone, not corporate or AI-sounding

---

## Lead Qualification

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
  "reason": "<one sentence>",
  "tags": ["<tag>", "<tag>"]
}
```

---

## Personalized First-Response Email

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

## Weekly Pipeline Summary

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

## Deal Note Summarizer

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

## Re-engagement Email

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

## Post-Meeting Follow-Up

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

## Multi-Touch Sequence (Per Touch)

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

## Breakup Email (Final Touch)

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
