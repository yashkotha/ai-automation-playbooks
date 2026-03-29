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
----------|--------------------|
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

---

## Firmographic Scoring

The AI qualification score is based on what the lead said. Firmographic scoring is based on who they are — their company size, industry, job title. This is orthogonal data and should be treated as a separate dimension.

### Scoring matrix

Apply firmographic points on top of the AI behavioral score. Store in a separate property: `firmographic_score`.

**Company size:**
| Employees | Points |
|-----------|--------|
| 1-10 | +0 |
| 11-50 | +1 |
| 51-200 | +2 |
| 201-500 | +3 |
| 500+ | +2 (large companies often have longer sales cycles) |

**Industry vertical** (customize for your ICP):
| Industry | Points |
|----------|--------|
| Your top ICP industries | +3 |
| Adjacent industries | +1 |
| Poor fit industries | -2 |

**Job title / seniority:**
| Title level | Points |
|-------------|--------|
| C-suite (CEO, CFO, COO) | +3 |
| VP / Director | +2 |
| Manager | +1 |
| Individual contributor | 0 |
| Student / intern | -1 |

**Annual revenue:**
| Revenue band | Points |
|--------------|--------|
| $1M-$10M | +1 |
| $10M-$50M | +2 |
| $50M+ | +3 |
| Unknown | 0 |

### Where firmographic data comes from

1. **The form** — ask for job title and company name. Company size and revenue can be looked up.
2. **HubSpot enrichment** — HubSpot automatically enriches company records with size and industry from its database when you have a domain.
3. **Clearbit / Apollo / Clay** — richer enrichment if HubSpot's built-in isn't sufficient. Send a webhook to an enrichment API on contact creation and write back to HubSpot.
4. **LinkedIn** — if you're running LinkedIn Lead Gen ads, LinkedIn passes job title and company through the form.

### n8n implementation

In your lead scoring workflow, after getting the AI score, add a Code node:

```javascript
const contact = $input.first().json;

let firmographicScore = 0;

// Company size scoring
const employees = parseInt(contact.numberofemployees || '0');
if (employees >= 201 && employees <= 500) firmographicScore += 3;
else if (employees >= 51 && employees <= 200) firmographicScore += 2;
else if (employees >= 11 && employees <= 50) firmographicScore += 1;

// Job title scoring
const title = (contact.jobtitle || '').toLowerCase();
if (title.includes('ceo') || title.includes('coo') || title.includes('cfo') || title.includes('founder')) {
  firmographicScore += 3;
} else if (title.includes('vp') || title.includes('vice president') || title.includes('director')) {
  firmographicScore += 2;
} else if (title.includes('manager') || title.includes('head of')) {
  firmographicScore += 1;
}

// Industry scoring (customize)
const industry = (contact.industry || '').toLowerCase();
const topIndustries = ['saas', 'software', 'technology', 'fintech'];
const poorFitIndustries = ['student', 'education', 'non-profit'];
if (topIndustries.some(i => industry.includes(i))) firmographicScore += 3;
else if (poorFitIndustries.some(i => industry.includes(i))) firmographicScore -= 2;

return [{ json: { ...contact, firmographic_score: firmographicScore } }];
```

Then update the HubSpot contact with `firmographic_score`.

### Combined score view

Create a HubSpot saved view: `ai_lead_score >= 7 AND firmographic_score >= 4`. These are leads who both expressed intent and match your ideal customer profile. This is your highest-priority segment.

---

## Intent Scoring from Page Visits

Page visits are the strongest behavioral signal you have before a conversation. Track them via HubSpot's tracking code.

### Signal hierarchy

| Page visited | Intent score added |
|-------------|-------------------|
| Pricing page | +3 |
| Demo / Book a call page | +3 |
| Case studies | +2 |
| Product features pages | +2 |
| Blog post | +1 |
| Homepage / About | +0 |
| Careers page | -1 (probably not a buyer) |

### HubSpot tracking setup

1. Install the HubSpot tracking script on your website (Settings -> Tracking & Analytics -> Tracking Code)
2. HubSpot automatically logs page views for known contacts (those who submitted a form)
3. In workflows: trigger on contact property `hs_analytics_last_url` contains "pricing" → add intent score

### n8n page visit scoring workflow

Trigger this via a HubSpot webhook when `hs_analytics_last_url` changes:

```javascript
const url = $input.first().json.propertyValue || '';
const currentScore = parseInt($input.first().json.contact?.ai_lead_score || '0');

let intentBoost = 0;
if (url.includes('/pricing')) intentBoost = 3;
else if (url.includes('/demo') || url.includes('/book')) intentBoost = 3;
else if (url.includes('/case-study') || url.includes('/customers')) intentBoost = 2;
else if (url.includes('/features') || url.includes('/product')) intentBoost = 2;
else if (url.includes('/blog')) intentBoost = 1;
else if (url.includes('/careers')) intentBoost = -1;

const newScore = Math.min(10, Math.max(1, currentScore + intentBoost));

return [{ json: { contactId: $input.first().json.objectId, newScore } }];
```

Then PATCH the contact with `ai_lead_score: newScore.toString()`.

### Pricing page trigger

Pricing page visits deserve special handling. Build a dedicated workflow:

Trigger: `pricing_page_viewed` changes to true
Actions:
1. Set `pricing_page_view_date` = today
2. If `ai_lead_score` >= 5 → add tag "High Purchase Intent", notify deal owner via Slack
3. If contact has no associated open deal → create deal automatically
4. Enroll in a shortened 3-touch sequence instead of the full 8-touch sequence

---

## Scoring Decay Over Time

A score of 8 from 60 days ago is not the same as a score of 8 from yesterday. Without decay, your "Hot Leads" view fills with contacts who were once interested but have gone cold.

### Decay model

Run a daily n8n workflow at 6am:

```javascript
const contacts = $input.all();

return contacts.map(item => {
  const contact = item.json;
  const currentScore = parseFloat(contact.ai_lead_score || '0');
  const lastActivity = contact.email_last_opened_at || contact.ai_first_response_sent_at;

  if (!lastActivity || currentScore <= 1) {
    return { json: { ...contact, skip: true } };
  }

  const daysSinceActivity = (Date.now() - new Date(lastActivity).getTime()) / (1000 * 60 * 60 * 24);

  let decayedScore = currentScore;

  // Decay schedule:
  // 0-14 days: no decay
  // 15-30 days: -0.5 per week
  // 31-60 days: -1 per week
  // 60+ days: -1.5 per week

  if (daysSinceActivity > 14 && daysSinceActivity <= 30) {
    const weeks = (daysSinceActivity - 14) / 7;
    decayedScore = currentScore - (weeks * 0.5);
  } else if (daysSinceActivity > 30 && daysSinceActivity <= 60) {
    const weeksBase = (30 - 14) / 7;
    const weeksExtra = (daysSinceActivity - 30) / 7;
    decayedScore = currentScore - (weeksBase * 0.5) - (weeksExtra * 1.0);
  } else if (daysSinceActivity > 60) {
    const weeksBase = (30 - 14) / 7;
    const weeksMiddle = (60 - 30) / 7;
    const weeksExtra = (daysSinceActivity - 60) / 7;
    decayedScore = currentScore - (weeksBase * 0.5) - (weeksMiddle * 1.0) - (weeksExtra * 1.5);
  }

  decayedScore = Math.max(1, Math.round(decayedScore * 10) / 10);

  const changed = decayedScore !== currentScore;

  return {
    json: {
      contactId: contact.id || contact.hs_object_id,
      ai_lead_score: decayedScore.toString(),
      decay_applied: changed,
      previousScore: currentScore
    }
  };
}).filter(item => !item.json.skip && item.json.decay_applied);
```

Filter to only contacts where decay actually changed the score, then batch-update them.

### Decay reset

When a lead re-engages (opens an email, clicks a link, visits a page), reset the decay clock:

Workflow trigger: `email_last_opened_at` changes
Action: If `ai_lead_score` < previous score before decay started AND decay had been applied → boost score by +1 (capped at original score) and set `re_engagement_status` = "Re-Engaged"

---

## Manual Score Override Workflow

Sometimes an AI score is wrong and a rep knows it. Build a clean override path so reps can correct scores without breaking the automation.

### The problem with direct edits

If reps just edit `ai_lead_score` directly, you can't distinguish a decayed score from an intentional override, and your automation may overwrite their change.

### Override properties to create

| Property | Type | Purpose |
|----------|------|---------|
| `score_override` | Number | Rep's manual score (blank = no override) |
| `score_override_reason` | Single-line text | Why they changed it |
| `score_override_by` | Single-line text | HubSpot user who set the override |
| `score_override_date` | Date | When override was set |
| `score_use_override` | Checkbox | Whether to use override instead of AI score |

### Effective score logic

Add a Code node at the start of any workflow that reads scores:

```javascript
const contact = $input.first().json;

const useOverride = contact.score_use_override === 'true';
const overrideScore = parseFloat(contact.score_override || '0');
const aiScore = parseFloat(contact.ai_lead_score || '0');

const effectiveScore = useOverride && overrideScore > 0 ? overrideScore : aiScore;

return [{ json: { ...contact, effectiveScore } }];
```

All downstream logic uses `effectiveScore`, not `ai_lead_score` directly.

### Override protection in decay workflow

In the decay workflow, skip contacts where `score_use_override` = true. The decay clock only runs on AI-scored leads.

### Rep interface

In HubSpot, create a "Sales Rep Score Override" card on the contact sidebar:
- Show: `ai_lead_score` (read-only label), `score_override`, `score_override_reason`, `score_use_override` toggle
- Reps can flip the toggle and enter their reasoning without touching any other fields

---

## Integration with Pinecone for Semantic Similarity Scoring

Standard keyword-based AI scoring can miss nuanced intent. A lead who writes "we're drowning in manual workflows" scores differently than "interested in automation" even though both indicate high fit. Pinecone semantic search adds this layer.

### The approach

1. Embed known "ideal customer" descriptions using an embedding model
2. When a new lead comes in, embed their inquiry text
3. Search Pinecone for semantic similarity to your best historical leads
4. Use the similarity score as an additional scoring signal

### Setup

**Step 1: Create a Pinecone index**

Use the Pinecone console or API to create an index:
- Dimension: 1536 (for OpenAI `text-embedding-3-small`)
- Metric: cosine

**Step 2: Populate with your best leads**

From your CRM, export closed-won contacts and their initial inquiry messages. Embed each message and upsert to Pinecone:

```javascript
// n8n Code node — prepare records for Pinecone upsert
const contacts = $input.all();

return contacts.map(item => {
  const c = item.json;
  return {
    json: {
      id: `contact-${c.id}`,
      text: c.initial_inquiry_message,
      outcome: c.hs_lead_status,
      score: c.ai_lead_score,
      industry: c.industry,
      closed_won: c.lifecyclestage === 'customer'
    }
  };
});
```

Then call OpenAI Embeddings API and upsert to Pinecone with `{ id, values: embedding, metadata: { outcome, closed_won, industry } }`.

**Step 3: Query on new lead arrival**

In your lead qualification workflow, after getting the AI score, add a Pinecone similarity search:

1. Embed the new lead's inquiry text (OpenAI HTTP Request)
2. Query Pinecone with the embedding:

```json
{
  "vector": "{{embedding}}",
  "topK": 5,
  "includeMetadata": true,
  "filter": {
    "closed_won": {"$eq": true}
  }
}
```

3. Calculate semantic similarity boost in a Code node:

```javascript
const matches = $input.first().json.matches || [];
const currentScore = parseFloat($('Get AI Score').first().json.ai_lead_score);

// Average similarity of top 3 matches
const topMatches = matches.slice(0, 3);
const avgSimilarity = topMatches.reduce((sum, m) => sum + m.score, 0) / topMatches.length;

// Boost: similarity > 0.85 = +2, > 0.75 = +1, < 0.5 = -1
let boost = 0;
if (avgSimilarity > 0.85) boost = 2;
else if (avgSimilarity > 0.75) boost = 1;
else if (avgSimilarity < 0.50) boost = -1;

const finalScore = Math.min(10, Math.max(1, Math.round(currentScore + boost)));

return [{
  json: {
    finalScore,
    semanticSimilarity: avgSimilarity,
    semanticBoost: boost,
    similarLeads: topMatches.map(m => m.metadata)
  }
}];
```

4. Store `semantic_similarity_score` as a custom property on the contact for reference.

### Keeping the index fresh

Monthly maintenance workflow:
1. Fetch all newly closed-won contacts from the last 30 days
2. Embed their inquiry messages
3. Upsert to Pinecone
4. This ensures the index stays representative of your actual customers, not just your early customers

---

## A/B Testing Your Scoring Model

Your initial scoring model is a hypothesis. A/B test it before trusting it.

### What to test

Not the score itself, but the thresholds and weights. Examples:
- Test A: Hot threshold at score >= 7
- Test B: Hot threshold at score >= 8
- Measure: conversion rate to customer, average close time

### Implementation

Add a property `scoring_model_version` to contacts: `v1` or `v2`. Assign alternately on contact creation.

```javascript
// Assign to A or B based on contact ID parity
const contactId = parseInt($input.first().json.contactId);
const version = contactId % 2 === 0 ? 'v1' : 'v2';
return [{ json: { scoringVersion: version } }];
```

Apply different thresholds based on `scoring_model_version`:
- `v1`: Hot = >= 7, Warm = 4-6
- `v2`: Hot = >= 8, Warm = 5-7

After 90 days (or 200 leads, whichever comes first), compare:
- Conversion rate: v1 hot leads vs v2 hot leads
- Close rate: % of hot leads that became customers
- Wasted effort: how many v1 "hot" leads were actually cold (poor use of sales time)

### Statistical validity

You need enough leads per variant to draw conclusions. With a 5% conversion rate and you want to detect a 20% relative improvement:
- You need approximately 400 leads per variant
- At 100 leads/month, that's 4 months per test

Don't end tests early. Wait for statistical significance before declaring a winner.

### What to do with results

If v2 (higher threshold) has a significantly better conversion rate among "hot" leads but your total hot lead count dropped by 30%, calculate: did you close more revenue with fewer but better-qualified hot leads? Revenue is the metric, not conversion rate in isolation.

---

## Score Recalibration Process

Your scoring model degrades over time as your market, product, and ICP evolve. Run a recalibration quarterly.

### Recalibration checklist

**1. Pull a cohort report**

From HubSpot: export all contacts from the last 6 months with:
- `ai_lead_score` at time of qualification (use `ai_qualification_reason` as a proxy)
- Final lifecycle stage (customer / lost / still open)
- `close_reason`

**2. Calculate score accuracy**

Build a confusion matrix:
- True positives: scored >= 7, became customers
- False positives: scored >= 7, did NOT become customers
- True negatives: scored < 7, did not become customers (correct)
- False negatives: scored < 7, became customers (these are the costly misses)

If false positives > 30% of your hot leads: your threshold is too low. Raise it.
If false negatives > 15%: your AI prompt or keywords are missing real buying signals. Revise the prompt.

**3. Review the AI qualification prompt**

Check your AI qualifier prompt for:
- Are your budget signal keywords current? ("investment" vs "budget" vs "cost")
- Are there new objections or signals your prompt doesn't cover?
- Has your product changed in a way that changes what "fit" means?

**4. Recalibrate firmographic scores**

Look at your closed-won customers from the last 6 months:
- What company sizes closed the most?
- Which industries had the highest win rate?
- Which job titles were the actual decision makers?

Update your firmographic scoring matrix to reflect actual data, not assumptions.

**5. Update the Pinecone index**

After recalibration, re-embed your updated pool of closed-won leads and upsert to Pinecone. Old embeddings don't need to be deleted unless they represent a product or ICP you've moved away from.

**6. Version control your prompts**

Every time you update the AI qualifier prompt, increment the version in a `scoring_model_version` property and store the old prompt somewhere. This lets you compare before/after cohorts.

---

## Integrating Behavioral Email Signals with the Initial AI Score

The AI score is set at lead creation. Email behavior happens over the following weeks. These are different signals at different times — integrate them without overwriting the initial assessment.

### The composite score

Rather than updating `ai_lead_score` with every email open, maintain three separate properties:

| Property | What it captures |
|----------|-----------------|
| `ai_lead_score` | Initial AI qualification score — set once, decays over time |
| `behavioral_email_score` | Running score based on email engagement |
| `composite_score` | Calculated field combining both |

### Behavioral email scoring rules

Run this workflow on each email event (open, click, reply):

```javascript
const contact = $input.first().json;
const event = contact.emailEvent; // 'open', 'click', 'reply'

let current = parseFloat(contact.behavioral_email_score || '0');

const boosts = {
  open: 0.5,
  click: 1.5,
  reply: 3.0
};

const boost = boosts[event] || 0;

// Cap at 10, decay toward 0 is handled separately
const newScore = Math.min(10, current + boost);

return [{ json: { contactId: contact.id, behavioral_email_score: newScore.toString() } }];
```

### Composite score calculation

Run nightly:

```javascript
const contact = $input.first().json;

const aiScore = parseFloat(contact.ai_lead_score || '0');
const behavioralScore = parseFloat(contact.behavioral_email_score || '0');

// Weighted average: 60% AI (fit/intent at time of inquiry), 40% behavioral (engagement since)
const composite = Math.round((aiScore * 0.6 + behavioralScore * 0.4) * 10) / 10;

return [{ json: { contactId: contact.id, composite_score: composite.toString() } }];
```

### When to alert based on composite score

The composite score enables scenarios the AI score alone can't catch:

- **Cold lead goes warm:** AI score was 4, but opened 6 emails and clicked twice → composite now 6+ → trigger "Re-engagement detected" workflow, create task for sales rep
- **Hot lead goes cold:** AI score 8, but opened nothing in 14 days → composite drops to 5 → move from "Hot" view to "Warm" view
- **High behavioral, low AI:** Engaged with every email but inquiry was vague → flag for manual review — these are sleeper leads

### Integration with sequence enrollment

When enrolling contacts in sequences, use `composite_score` not `ai_lead_score`:
- composite >= 7 → high-touch sequence (shorter intervals, phone calls included)
- composite 4-6 → standard sequence
- composite < 4 → long-form content nurture (bi-weekly)

This means a lead who started cold but has been engaging with your emails gets upgraded to a higher-touch sequence automatically.
