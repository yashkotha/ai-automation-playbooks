# AI Community

For builders who want real infrastructure, real workflows, and real prompts. Not walkthroughs of n8n's drag-and-drop UI. Not "here's how to connect two nodes." The content here assumes you're trying to run something in production and want to understand the decisions behind the setup, not just copy the config.

---

## What's in Each Folder

### Self-Hosted Stack -- [`./self-hosted-stack/`](./self-hosted-stack/)

A complete Docker Compose setup for running a production-grade AI automation stack on a single VPS. All services are containerized, all communicate over an internal Docker network, and Caddy handles TLS termination so nothing listens on raw HTTP in production.

**Services included in the compose file:**
- **n8n** -- workflow automation engine with Postgres as the persistent backend (not SQLite, which breaks under concurrent load)
- **Postgres** -- n8n's database, also usable by other services
- **Redis** -- used for n8n's queue mode and as a general-purpose cache
- **Qdrant** -- self-hosted vector database for RAG workflows and semantic search
- **Ollama** -- local LLM inference for tasks where you don't want to send data to an external API
- **MinIO** -- S3-compatible object storage, used for n8n backups and binary file handling
- **Portainer** -- web UI for container management without needing SSH for routine tasks
- **Uptime Kuma** -- lightweight uptime monitoring with Slack/email alerting

**Networking and TLS:**
Caddy is configured as a reverse proxy for all HTTP-facing services. It handles automatic certificate issuance via Let's Encrypt and redirects HTTP to HTTPS. Each service gets its own subdomain. The Caddyfile template is included with comments explaining the reverse proxy and header configuration.

**Security hardening:**
- `ufw` configured to allow only ports 22, 80, and 443 by default
- `fail2ban` configured for SSH brute-force protection with n8n webhook log monitoring
- All containers run as non-root users
- No container mounts host directories with write access except the declared data volumes
- Secrets (API keys, database passwords) are passed via `.env` file, not hardcoded in the compose file
- `.env.example` template included; `.env` is gitignored

**Automated backups:**
A cron job runs nightly at 2am to dump Postgres, compress the n8n workflow JSON exports, and push both to MinIO under a dated prefix. Retention policy defaults to 30 days. The backup script, cron configuration, and a restore walkthrough are all included.

**Safe n8n upgrade procedure:**
The guide covers how to pull a new n8n image without losing workflow data, execution history, or credentials. It includes the correct sequence of `docker compose pull`, backup verification, and service restart, and what to check in the n8n logs after a version bump.

**VPS cost tiers:**

| Tier | Specs | Monthly cost | What fits |
|------|-------|-------------|-----------|
| Minimal | 1 vCPU / 2 GB RAM | ~$6 | n8n + Postgres only |
| Standard | 2 vCPU / 4 GB RAM | ~$12 | Full stack without Ollama |
| Comfortable | 4 vCPU / 8 GB RAM | ~$20 | Full stack including Ollama (small models) |
| Heavy | 8 vCPU / 16 GB RAM | ~$40 | Full stack with Ollama running 7B+ models |

Providers benchmarked: Hetzner (best price/performance in EU), Hostinger (competitive globally), DigitalOcean (higher cost, better support docs).

**Troubleshooting section covers:** n8n container failing to start due to Postgres connection timing, Caddy not issuing certificates when DNS hasn't propagated, Qdrant OOM on small instances, MinIO bucket permissions on first run, Redis connection refused when queue mode is enabled without correct env vars, and Ollama first-model-pull timeouts.

---

### n8n Workflow Templates -- [`./n8n-workflow-templates/`](./n8n-workflow-templates/)

15 production workflow JSON files, each importable directly into n8n. Each file includes a comment node at the top of the canvas explaining what the workflow does, what to configure before activating, and any known edge cases.

**The 15 templates:**

1. **Lead Qualification** -- webhook trigger from a form submission, Groq call to score and categorize the lead, HubSpot contact create/update with score written to custom property
2. **First-Response Email** -- triggered on new HubSpot contact, GPT-4o-mini generates a personalized opener based on company name and source, sends via Resend within 90 seconds
3. **HubSpot Contact Sync** -- bidirectional sync between an Airtable base and HubSpot contacts, with field mapping config and conflict resolution logic
4. **Slack Deal Notifications** -- polls HubSpot deals on a 15-minute interval, posts to Slack when deal stage changes, includes deal value and next step in the message
5. **AI Lead Scoring** -- scheduled daily run across all new HubSpot contacts, Groq inference to score against ICP criteria, writes score + reasoning to contact notes
6. **Blog Content Pipeline** -- accepts a topic via webhook, Claude generates a structured outline, GPT-4o-mini expands each section, posts draft to Notion with metadata
7. **Meeting Schedule Confirmation** -- Calendly webhook triggers n8n, pulls contact from HubSpot, sends a custom confirmation email via Resend with relevant context
8. **CRM Activity Logger** -- n8n webhook accepts call transcript text, Claude summarizes it into next steps and topics, creates a HubSpot note with structured output
9. **Proposal Generator** -- triggered when HubSpot deal moves to Proposal stage, pulls deal and contact data, GPT-4o generates a structured proposal draft to a Notion page
10. **Abandoned Cart Recovery** -- webhook from e-commerce platform, waits 2 hours, checks if purchase was completed, sends recovery email via Resend if not
11. **Voicemail Transcription** -- audio file URL triggers n8n, Whisper API transcribes, Claude extracts key info, logs to HubSpot note and sends Slack summary to rep
12. **Onboarding Sequence** -- triggered on new HubSpot deal Closed Won, enrolls contact in a 5-step email sequence over 14 days via Resend with HubSpot activity logging
13. **Airtable Sync** -- scheduled sync from an Airtable base to HubSpot, handles new records and updates, skips records that haven't changed since last sync
14. **Content Repurposer** -- accepts a long-form URL or text, Claude splits and rewrites it into LinkedIn post, tweet thread, and email teaser, posts to Notion for review
15. **Pipeline Digest** -- runs every Monday at 8am, pulls all open HubSpot deals, Claude generates a prioritized summary highlighting stalled deals, emails to rep via Resend

**Architecture patterns covered in the guide:**

- **Webhook triggers** -- how to validate payloads, return fast 200s while processing asynchronously, and handle retries without duplicate records
- **Scheduled runs** -- how to prevent overlap when a workflow runs longer than its schedule interval
- **Error handling** -- five patterns: stop-and-alert, retry with backoff, continue-on-failure with logging, dead-letter queue via Postgres, and fallback branch execution
- **Idempotency** -- how to check whether a record already exists before writing to HubSpot or Notion, preventing duplicates on re-runs
- **Loop optimization** -- batching API calls instead of one-at-a-time, respecting rate limits without sleep nodes, and when to use n8n's Split in Batches vs. a manual loop

**Decision framework: webhook vs. poll vs. schedule**

The guide walks through when each trigger type is appropriate based on latency requirements, the reliability of the upstream system's webhooks, and whether the workflow can tolerate eventual consistency.

---

### Prompt Library -- [`./prompt-library/`](./prompt-library/)

90+ prompts used in production, organized so you can find what you need without scrolling through a flat file. Every prompt in this library follows four rules: structured JSON output where the response feeds another system, `{{variable}}` placeholder syntax compatible with n8n's expression engine, no em dashes, and writing that doesn't read like it was generated by a language model.

**Categories and what's in each:**

| Category | Prompt count | Highlights |
|----------|-------------|-----------|
| Sales | 8 | Lead qualifier, ICP matcher, deal summary, objection handler, follow-up writer |
| Marketing | 7 | Campaign brief generator, audience segmenter, A/B variant writer, ad copy generator |
| Customer Success | 6 | Churn risk analyzer, health score explainer, QBR agenda builder, escalation summary |
| Operations | 5 | SOP writer, process audit, meeting notes formatter, task extractor |
| Product | 5 | Feature request categorizer, user feedback summarizer, release note writer |
| Content | 8 | Blog outliner, section expander, headline variants, SEO meta writer |
| n8n Workflow Engineering | 9 | Workflow describer, error message explainer, node config generator, workflow debugger, expression builder |
| HubSpot CRM Operations | 7 | Property schema designer, list criteria builder, deal note summarizer, contact enricher |
| Pinecone + Vector Search | 5 | Query rewriter, chunk splitter, relevance evaluator, RAG answer generator |
| Groq/LLM Inference | 6 | Model selector, prompt compressor for fast inference, temperature guide, system prompt optimizer |
| Framer + Design | 5 | CMS field mapper, component copy writer, landing page section generator |
| Claude/AI Agent Ops | 7 | Agent instruction writer, tool use describer, multi-step task planner, output evaluator |
| Automation + Data Processing | 7 | JSON transformer, field mapper, deduplication logic writer, data validation prompt |
| Raycast Extension Dev | 4 | Extension scaffolder, command description writer, preference schema generator |
| Customer Intelligence | 6 | Persona builder, segment definer, customer story extractor, win/loss analyzer |

All prompts are stored as individual `.md` files with a YAML frontmatter block containing: category, model recommendation, temperature, output format, and use-case tags. This makes them searchable and usable programmatically.

---

### Claude Code Skills -- [`./claude-code-skills/`](./claude-code-skills/)

12 custom SKILL.md files that extend Claude Code for specific domains. Each skill gives Claude Code a defined persona, a set of tools it should use, a process to follow for common tasks, and rules that prevent it from doing things that cause problems in that domain.

**The 12 skills:**

1. **`hubspot-automator`** -- Creates and updates HubSpot objects, builds list criteria, designs property schemas, and writes n8n nodes for HubSpot API calls. Knows the HubSpot v3 API response structure and avoids common mistakes like missing required association objects.

2. **`framer-designer`** -- Writes Framer CMS schema, generates component copy, creates landing page section structures in Framer's XML format, and handles CMS item creation via the Framer API. Knows Framer's component model and what the MCP tools can and can't do.

3. **`email-deliverability-auditor`** -- Reviews email copy, HTML structure, and sending configuration for deliverability issues. Checks SPF/DKIM/DMARC setup, subject line spam triggers, image-to-text ratio, and link structure. Outputs a prioritized fix list.

4. **`vps-debugger`** -- Diagnoses Docker Compose issues, Caddy config errors, n8n startup failures, and networking problems on a Linux VPS. Uses the self-hosted stack structure from this repo as its mental model.

5. **`lead-sequence-builder`** -- Designs multi-step email sequences. Takes an ICP description and goal, outputs a step-by-step sequence with subject lines, email copy, delay logic, and the HubSpot list criteria to enroll contacts.

6. **`design-system-builder`** -- Creates Tailwind-based design token sets, component documentation, and color palette decisions. Outputs in formats usable in Framer, code, or a Notion doc.

7. **`content-repurposer`** -- Takes a long-form piece and outputs reformatted versions for LinkedIn, Twitter/X, email, and a short blog summary. Maintains the original voice and avoids AI-sounding phrasing.

8. **`n8n-workflow-builder`** -- Designs n8n workflow logic from a description. Outputs the node sequence, recommended node types, expression syntax for common transformations, and a list of credentials needed.

9. **`crm-data-cleaner`** -- Reviews HubSpot export data, identifies duplicates, missing fields, inconsistent formatting, and property misuse. Outputs a prioritized cleanup plan with specific HubSpot list criteria to find affected records.

10. **`raycast-extension-dev`** -- Scaffolds Raycast extensions, writes command logic, handles preference schemas, and produces README content formatted for the Raycast store. Knows the Raycast API and common patterns.

11. **`vector-search-architect`** -- Designs Pinecone or Qdrant index schemas, writes chunking strategies for different document types, and produces n8n nodes for embedding and retrieval. Includes namespace strategy for multi-tenant setups.

12. **`onboarding-sequence-writer`** -- Takes a product description and user type, outputs a complete onboarding email sequence with copy, timing, success criteria per step, and HubSpot enrollment logic.

**Each skill file includes:**
- YAML frontmatter: name, version, trigger phrases, model recommendation
- Process section: numbered steps Claude follows for the primary task type
- Rules section: what Claude must not do, stated as hard constraints
- Output format: exactly what the response structure should look like
- Testing protocol: 3-5 test inputs with expected output descriptions to verify the skill is working

**How to install a skill:**
Copy the SKILL.md file to your `~/.claude/skills/` directory (or the path configured in your Claude Code settings). Claude Code will pick it up on the next session. Trigger phrases listed in the frontmatter activate the skill automatically; you can also invoke it explicitly by name.

---

## Where to Start

**You're new to AI automation and don't have a running n8n instance yet.**
Start with [`self-hosted-stack/`](./self-hosted-stack/). Read the setup guide, provision a VPS, and run through the Docker Compose deployment. Get n8n accessible at a domain before doing anything else. Budget 2-3 hours for this step. Once n8n is running, import one of the simpler workflow templates (Lead Qualification or Slack Deal Notifications) to verify the setup works end to end.

**You have n8n running and want to build real workflows quickly.**
Skip the stack guide. Go to [`n8n-workflow-templates/`](./n8n-workflow-templates/) and import the template closest to your use case. Read the comment block inside the workflow, configure the credentials and field names for your setup, and test with a single record before activating. Then pull the relevant prompts from [`prompt-library/`](./prompt-library/) to customize the AI steps.

**You use Claude Code every day and want to extend what it can do.**
Go directly to [`claude-code-skills/`](./claude-code-skills/). Install the skills that match your current work domains. If you're working on HubSpot and n8n, install `hubspot-automator` and `n8n-workflow-builder` first. If you're working on content or email, install `content-repurposer` and `lead-sequence-builder`. You'll notice a difference in response quality within the first session.

---

## Skill Prerequisites

You'll get more out of this folder if you're already comfortable with:

- **Docker and Docker Compose** -- enough to read a `compose.yml` file, run `docker compose up -d`, and interpret `docker logs` output
- **Basic Linux command line** -- SSH, file permissions, editing files with `nano` or `vim`, understanding of cron syntax
- **n8n fundamentals** -- you should know what a node is, how credentials work, and how to test a workflow manually before reading the templates
- **HubSpot basics** -- contacts, deals, companies, and properties. The CRM setup guide in [`hubspot/`](./hubspot/) fills gaps if needed
- **API concepts** -- webhooks, REST, JSON payloads, and HTTP status codes. You don't need to write APIs, but you need to read them

You do not need to know Python, JavaScript, or any programming language to use the workflow templates or prompts. The Claude Code skills folder requires that you use Claude Code, but no coding experience is needed to install and use skills.
