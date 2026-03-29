# AI Automation Playbooks
<img width="1200" height="675" alt="LinkedIn – Open Source Contributions" src="https://github.com/user-attachments/assets/1fc8881a-867b-4eb7-abca-19b0be20d61f" />

Production-tested AI automation workflows, infrastructure guides, and prompt libraries. Built from real systems running real workloads, not from tutorials or toy projects. If you want to understand how AI automation actually works at the seams, you're in the right place.

---

## Who This Is For

**AI builders and indie hackers**
You're running n8n on a VPS or thinking about it, you've used OpenAI's API, and you're tired of Medium posts that stop before the hard parts. This repo gives you working Docker Compose setups, 15 production workflow templates, 90+ battle-tested prompts, and a set of Claude Code skill templates you can drop in today. The self-hosted stack guide covers everything from zero to a hardened VPS with Caddy, fail2ban, and automated backups.

**Marketers running AI-assisted outreach**
You need to move faster than your current tool stack allows. The marketing track covers lead scoring with n8n and Groq, AI-written email sequences that don't sound like AI wrote them, and multi-touch cadences that adjust based on contact behavior. Everything uses HubSpot as the source of truth and Resend for delivery.

**HubSpot users and RevOps teams**
The HubSpot track is for people who want their CRM to actually do work rather than just store data. It covers a complete custom properties schema for a B2B SaaS use case, n8n workflows that sync, score, and notify without HubSpot's native automation fees, and a CRM setup guide that explains why certain decisions were made, not just what they are.

**Designers and developers building with AI tools**
The design and dev track covers Framer with AI-generated content and CMS integration, Raycast extensions for personal automation, and design-to-code workflows that produce usable output rather than vibe-coded noise. If you're building on top of Framer or want to speed up repetitive design tasks, start here.

---

## What's Inside

| Track | Folder | Content |
|-------|--------|---------|
| **AI Community** | [`ai-community/`](./ai-community/) | Self-hosted stack (n8n + Postgres + Redis + Qdrant + Ollama + MinIO + Portainer + Uptime Kuma), Caddy reverse proxy, security hardening, automated backups, VPS cost tiers, 15 n8n workflow templates, 90+ prompts across 15 categories, 12 Claude Code skill templates |
| **Marketing** | [`marketing/`](./marketing/) | AI email sequences built for HubSpot + Resend, lead scoring workflows using Groq for real-time inference, multi-touch cadences with behavioral branching, sequence copy templates, deliverability setup guide |
| **HubSpot** | [`hubspot/`](./hubspot/) | Full CRM setup walkthrough for a B2B SaaS use case, custom properties schema with field types and rationale, n8n-to-HubSpot integration patterns for contacts/deals/activities, pipeline architecture decisions |
| **Design + Dev** | [`design/`](./design/) | Framer AI workflows for landing pages and CMS, design-to-code patterns with Claude Code, Raycast extension templates for local automation, component generation prompts |

### AI Community -- detailed contents

**Self-Hosted Stack** covers the full Docker Compose setup to run n8n alongside Postgres, Redis, Qdrant (vector DB), Ollama (local LLM inference), MinIO (object storage), Portainer (container management), and Uptime Kuma (monitoring) on a single VPS. Includes Caddy as a reverse proxy with automatic HTTPS, ufw and fail2ban hardening, non-root container configuration, secrets management via environment files, an automated daily backup script to MinIO with cron, a safe n8n upgrade procedure that avoids data loss, a four-tier VPS cost breakdown from $6/month to $40/month, and a troubleshooting section for the ten most common issues.

**n8n Workflow Templates** includes 15 production workflow JSON files: lead qualification, first-response email, HubSpot contact sync, Slack deal notifications, AI lead scoring, blog content pipeline, meeting scheduling with calendar check, CRM activity logging, proposal generation from deal data, abandoned cart recovery, voicemail transcription to CRM note, new contact onboarding sequence, Airtable-to-HubSpot sync, content repurposing from long-form to short-form, and weekly pipeline summary digest. Each template comes with a comment block explaining its trigger logic, error handling approach, and what to configure before running. The guide also covers five universal error handling patterns, the webhook vs. poll vs. schedule decision framework, and loop optimization for high-volume workflows.

**Prompt Library** contains 90+ prompts organized across 15 categories: Sales, Marketing, Customer Success, Operations, Product, Content, n8n Workflow Engineering, HubSpot CRM Operations, Pinecone and Vector Search, Groq/LLM Inference, Framer and Design, Claude/AI Agent Ops, Automation and Data Processing, Raycast Extension Dev, and Customer Intelligence. Every prompt uses `{{variable}}` syntax for n8n compatibility, produces structured JSON output where needed, and avoids AI-sounding phrasing. Highlights include: ICP builder, lead qualifier, first-response email writer, churn risk analyzer, pipeline summarizer, RAG evaluator, prompt debugger, deal summary generator, and persona creator.

**Claude Code Skills** provides 12 custom SKILL.md templates ready to install into Claude Code: `hubspot-automator`, `framer-designer`, `email-deliverability-auditor`, `vps-debugger`, `lead-sequence-builder`, `design-system-builder`, `content-repurposer`, `n8n-workflow-builder`, `crm-data-cleaner`, `raycast-extension-dev`, `vector-search-architect`, and `onboarding-sequence-writer`. Each skill includes a full SKILL.md with frontmatter, a defined process, rules, output format, and a testing protocol. Installation instructions are included.

---

## The Stack

| Layer | Tool | Tier / Cost | Notes |
|-------|------|-------------|-------|
| Automation | n8n (self-hosted) | Free | Runs on your VPS via Docker |
| Compute | VPS (Hetzner / Hostinger / DigitalOcean) | $6-40/month | 2 vCPU / 4 GB RAM handles most workloads |
| AI inference (cloud) | OpenAI GPT-4o / GPT-4o-mini | Pay per token (~$5-20/month) | GPT-4o-mini for high-volume tasks, GPT-4o for quality-sensitive ones |
| AI inference (fast + cheap) | Groq API | Free tier / ~$2-8/month | Llama 3 and Mixtral at 300+ tokens/second, best for real-time scoring |
| AI agent work | Claude API (claude-sonnet-4-x) | Pay per token (~$5-15/month) | Claude Code for local dev; API for agentic workflows |
| Vector database | Pinecone | Free tier (1 index, 100k vectors) | Used for semantic search, RAG, lead matching |
| Vector DB (self-hosted) | Qdrant | Free (Docker) | Runs on VPS alongside n8n |
| Email delivery | Resend | Free (3k/month) / $20/month (50k) | Developer-friendly, strong deliverability |
| CRM | HubSpot | Free tier | Contacts, deals, companies, pipelines |
| Site builder | Framer | $5-15/month per site | Used in design track |
| Local automation | Raycast | Free / $8/month Pro | Extension dev covered in design track |
| AI coding | Claude Code | $20/month (Pro) or API billing | Used throughout; skills folder extends it |
| Secret storage | .env files + Docker secrets | Free | No paid vault needed at this scale |
| Monitoring | Uptime Kuma | Free (Docker) | Runs on VPS |
| Object storage | MinIO | Free (Docker) | Used for n8n backups on VPS |

Total for a working production setup: roughly $25-45/month depending on AI usage.

---

## What You Can Build

These are complete systems, each using multiple components from this repo:

1. **Lead qualification pipeline.** A HubSpot form submission triggers an n8n webhook, runs the contact through a Groq-powered scoring prompt, writes the score and reasoning back to custom HubSpot properties, and sends a Slack notification to the rep with the top three reasons to follow up. Total elapsed time from form fill to Slack ping: under 10 seconds.

2. **AI-assisted email sequence.** n8n watches a HubSpot list enrollment, pulls contact context via the HubSpot API, passes it through a GPT-4o-mini prompt with a personalization template, and queues the email via Resend with a configurable delay between steps. The sequence pauses automatically if the contact replies or books a meeting.

3. **Self-hosted RAG assistant.** Qdrant stores embedded versions of your documentation, past proposals, or product knowledge base. An n8n workflow accepts a natural language query, embeds it via OpenAI, runs a nearest-neighbor search in Qdrant, and returns the top three results as context to a Claude prompt that writes a final answer. No external retrieval SaaS needed.

4. **CRM activity logger for sales calls.** A voicemail or call recording hits an n8n webhook, gets transcribed via Whisper, summarized via a Claude prompt into next steps and key topics, and logged as a HubSpot note on the correct contact with the deal associated. The rep gets a Slack message with the summary.

5. **Weekly pipeline digest.** A scheduled n8n workflow runs every Monday at 8am, pulls all open deals from HubSpot, sends them through a summarization prompt that highlights stalled deals and deals advancing, and emails the output to the sales lead as a formatted digest via Resend.

6. **Content repurposing engine.** A long-form blog post or transcript is sent to an n8n workflow that breaks it into sections, runs each through a prompt to generate a LinkedIn post, a tweet thread, and a short email teaser, then posts drafts to a Notion database for review. One source, four outputs.

7. **Churn risk monitor.** A scheduled n8n workflow pulls customer activity data from HubSpot, scores each account against a churn risk prompt, flags accounts above a threshold, creates a HubSpot task for the CSM, and sends a weekly churn risk summary to a Slack channel.

8. **Proposal generator.** When a HubSpot deal reaches the Proposal stage, n8n pulls the deal name, contact info, notes, and associated product line-items, passes them to a GPT-4o prompt with a proposal template, and writes the output to a Google Doc or Notion page linked back in the HubSpot deal record.

9. **Personal Raycast command center.** A set of Raycast extensions that connect to your n8n instance via webhook to trigger common workflows from the keyboard: log a call note to HubSpot, pull today's pipeline summary, draft a follow-up email, or check Uptime Kuma status, all without touching a browser.

10. **Framer site with AI-generated CMS content.** An n8n workflow accepts a topic or keyword, generates a structured blog post via Claude, formats it to match your Framer CMS schema, and publishes it via the Framer API. The site stays current without manual content work.

---

## Philosophy

The content here started from frustration with AI automation tutorials that stop right before the part where things break. They show you how to set up a webhook. They don't show you what happens when the downstream API rate-limits you at 2am, or how to make a workflow idempotent so re-runs don't create duplicate CRM records. Every pattern in this repo has encountered that problem and solved it.

The stack is deliberately cost-conscious. You don't need $500/month in SaaS subscriptions to build serious automation. A $10 VPS running Docker, a few API keys, and a HubSpot free account can handle the volume of a 10-person startup without hitting any limits that matter. When you do outgrow something, the architecture makes it obvious what to swap and what to keep.

The patterns are tool-agnostic by intention. n8n is used throughout because it's free to self-host, has excellent node coverage, and runs anywhere Docker runs. But the workflow logic, the prompt structure, the CRM data model, and the error handling patterns all transfer to Make, Zapier, or Pipedream without a rewrite. Learning the shape of the problem matters more than learning one tool's interface.

Self-hosting is a real choice here, not a hobby preference. When your automation touches customer data, outbound email, and CRM records, you want to know exactly where that data is and who can see it. Running n8n on your own VPS with Postgres means you own the execution logs, the workflow history, and the data in transit. The setup takes an afternoon. The ongoing maintenance is one `docker compose pull` per month.

---

## How to Navigate

**If you're starting from zero and want infrastructure first:**
Go to [`ai-community/self-hosted-stack/`](./ai-community/self-hosted-stack/). Get the VPS running, deploy the Docker Compose stack, and verify n8n is accessible. Then come back here.

**If you're already running n8n and want workflow patterns:**
Go to [`ai-community/n8n-workflow-templates/`](./ai-community/n8n-workflow-templates/). Import the templates relevant to your use case, read the comment blocks, and adapt.

**If HubSpot is your primary system and you want n8n connected to it:**
Start at [`hubspot/`](./hubspot/) for the data model, then [`hubspot/n8n-integrations/`](./hubspot/n8n-integrations/) for the workflow patterns.

**If you're running marketing sequences and want AI in the loop:**
Start at [`marketing/lead-scoring/`](./marketing/lead-scoring/) to understand how scoring works, then [`marketing/email-sequences/`](./marketing/email-sequences/) for the sequence architecture.

**If you use Claude Code daily and want to extend it:**
Go directly to [`ai-community/claude-code-skills/`](./ai-community/claude-code-skills/) and install the skills that match your work.

**If you're building on Framer or want Raycast automations:**
Start at [`design/`](./design/) and work through [`framer-ai/`](./design/framer-ai/) or [`raycast-extensions/`](./design/raycast-extensions/) depending on what you need.

---

## Contributing

If you've built something real using these patterns and it's materially different from what's already here, contributions are welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines. The bar is: has this run in production and solved an actual problem? If yes, it belongs here.

---

Built by [Yashwanth Kotha](https://github.com/yashkotha) | [@YashKotha on X](https://x.com/YashKotha) | [LinkedIn](https://www.linkedin.com/in/yashkotha)
