# Lead Scoring

Automatically score incoming leads so you know where to focus attention.

## The Model

Four signals, weighted by reliability:

| Signal | Weight | What to detect |
|--------|--------|----------------|
| Budget signals | 30% | Mentions price, numbers, "investment", "budget" |
| Timeline signals | 30% | "soon", "this month", "ready", "urgent", "ASAP" |
| Fit signals | 20% | Their specific need matches what you offer |
| Engagement quality | 20% | Message length, specificity, questions asked |

## Score Tiers

| Score | Label | Action |
|-------|-------|--------|
| 8-10 | Hot | Notify sales immediately. Respond within 5 min. |
| 5-7 | Warm | Enter nurture sequence. Follow up within 24h. |
| 1-4 | Cold | Long-term nurture or disqualify. |

## Implementation

Use the [AI Lead Qualifier workflow template](../../ai-community/n8n-workflow-templates/README.md) with the [Lead Qualification prompt](../../ai-community/prompt-library/README.md).

Store the score in your CRM as a custom numeric property: `ai_lead_score`.

Create a saved view in your CRM: filter `ai_lead_score >= 8` and label it "Hot Leads". This is what your sales team checks every morning instead of the full contact list.

---

## Layer 2: Behavioral Scoring

AI scoring is a snapshot at the moment of inquiry. Layer behavioral data on top to track how interest evolves:

| Property | What it tells you |
|----------|--------------------|
| `email_open_count` | Are they reading your emails? |
| `email_click_count` | Are they clicking your CTAs? |
| `days_since_last_activity` | Are they going cold? |

A lead that scored 6 but has opened 5 emails and clicked 3 times is warmer than a lead that scored 8 and hasn't opened anything. Both signals matter.

---

## Disqualification

Not every lead deserves the full 30-day sequence. Disqualify if:
- Score is 1-2 AND no email engagement after touch 2
- Email bounced
- Replied to say they're not interested

Build these as IF node conditions in your n8n sequence workflow to stop the automation cleanly.
