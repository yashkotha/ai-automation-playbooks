# HubSpot CRM Setup from Scratch

A practical guide for getting HubSpot production-ready, not just configured.

---

## Step 1: Design Your Pipelines

Create pipelines that match your actual sales process, not HubSpot's defaults.

**For a service or real estate business:**
1. New Lead
2. Contacted
3. Meeting Scheduled
4. Proposal / Quote Sent
5. Negotiation
6. Closed Won
7. Closed Lost

**For high-volume B2C:**
1. New Lead
2. Qualified
3. Active
4. Closed Won
5. Closed Lost

Don't add stages for things that don't require a distinct action. More stages means more cognitive load and more deals stuck in limbo.

---

## Step 2: Add Custom Properties

HubSpot's default properties cover most basics. Add custom ones for:
- AI-generated scores and tags
- Email automation tracking flags
- Engagement data (opens, clicks)

See [Custom Properties](../custom-properties/) for the full schema to copy.

---

## Step 3: Connect Your Email

Settings -> Integrations -> Email Integrations

Connect Google Workspace or Outlook 365. Once connected, all replies from contacts are automatically logged on the contact timeline.

For Outlook 365, SMTP AUTH must be enabled at the tenant level before the integration works. See [n8n Integrations](../n8n-integrations/README.md) for the setup steps.

---

## Step 4: Keep Data Clean from Day One

These habits prevent hours of cleanup later:
- Set required fields on all forms (minimum: name + email)
- Never import without mapping every column
- Use a naming convention for test records (prefix with "TEST" or "DELETE")
- Set Lead Status as a required field on import

---

## Step 5: Build Your Core Views

Create these saved views immediately:

| View | Filter |
|------|--------|
| Hot Leads | `ai_lead_score >= 8` |
| Needs First Response | Contacts: `first_response_sent` not set |
| No Activity 7 Days | Deals: `last_activity_date` older than 7 days |
| Active Deals by Stage | Open deals, grouped by stage |

These replace a daily manual review of the full contact list.

---

## Step 6: Add a Product Catalog

Settings -> Sales -> Products

Add your products or service tiers with prices. Attach them to deals to get accurate pipeline value. This makes the AI pipeline summary prompt much more useful because it can report on real revenue numbers.

---

## Step 7: Connect Instagram Lead Ads (if applicable)

Settings -> Integrations -> Ad Networks -> Facebook

This auto-imports new Instagram and Facebook Lead Ad submissions as contacts. Pair with the [Instagram Lead Import workflow](../../ai-community/n8n-workflow-templates/README.md) to trigger the AI qualifier immediately on import.
