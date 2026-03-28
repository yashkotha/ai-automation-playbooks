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
