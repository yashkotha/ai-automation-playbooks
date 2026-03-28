# Marketing Automation Playbooks

AI-first marketing automation for small teams. These patterns replace tools that charge $300-800/month with a self-hosted stack that runs for under $30/month.

## Tracks

- [Email Sequences](./email-sequences/) - AI-personalized campaigns that don't read like AI
- [Lead Scoring](./lead-scoring/) - Score leads automatically based on signals and behavior
- [Multi-Touch Cadences](./multi-touch-cadences/) - 30-day nurture sequences that convert

## Core Stack

| Tool | Purpose | Cost |
|------|---------|------|
| n8n (self-hosted) | Workflow automation | Free |
| OpenAI API | Email personalization | ~$0.01-0.02 per email |
| Resend | Email delivery | Free up to 3k/month |
| HubSpot (free) | CRM and contact tracking | Free |

## What Makes This Different

Most marketing automation is template-based. Mail merge with {{first_name}}. The person on the receiving end can tell.

The patterns here use AI to write each email individually, pulling from the lead's original message, their behavior, and their context. It reads differently because it is different.

The first response email goes out within 10 minutes of a form submission. The follow-up sequence runs automatically for 30 days. All of it can be run by one person without a marketing team.
