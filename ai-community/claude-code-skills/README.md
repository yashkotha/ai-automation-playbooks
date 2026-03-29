# Claude Code Skills

Drop-in skill templates for Claude Code. These are markdown files that go in your `.claude/skills/` directory and give Claude specialized workflows for specific tasks.

## What Are Skills?

Skills are structured markdown files that Claude Code reads to guide its approach. When you invoke a skill, Claude follows its instructions rather than its default behavior. Good skills encode your specific standards, project context, and preferred workflows.

## How to Install

1. Find your Claude config directory: `~/.claude/` (global) or `.claude/` in your project root
2. Create a `skills/` folder inside it
3. Drop any `.md` skill file from this folder into it
4. Invoke in Claude Code with `/<skill-name>`

---

## Available Skills

### crm-workflow-builder
Guides Claude through designing and implementing n8n workflows that integrate with a CRM. Covers trigger selection, data mapping, error handling, and test strategy.

**Best for:** Building automation workflows that read from or write to a CRM.

### email-automation-planner
Helps design full email sequences: writing AI prompts per touch, planning cadence, setting up CRM tracking flags, and choosing the right send architecture.

**Best for:** Planning a new email automation before building it in n8n.

### vps-setup-guide
Step-by-step checklist for spinning up a VPS with Docker, n8n, Caddy, and custom domains.

**Best for:** Setting up a new self-hosted automation environment from scratch.

### api-integration-architect
Given an API you want to connect, designs the n8n workflow architecture, authentication method, error handling approach, and test plan.

**Best for:** Connecting an unfamiliar API without wasting time on trial and error.

### raycast-extension-builder
Guides Claude through building a Raycast extension: scaffolding, command design, API integration, preference handling, and store submission.

**Best for:** Building Raycast extensions that connect to external services or automate local tasks.

### framer-designer
Designing, editing, and managing Framer sites via MCP. Covers CMS content, code components, color and text styles, and page structure.

**Best for:** Making design or content changes to a Framer site without opening the Framer UI.

### hubspot-automator
Building and debugging HubSpot + n8n workflows. Covers contact/deal management, custom properties, pipeline automation, and common HubSpot API gotchas.

**Best for:** Any workflow that reads from or writes to HubSpot CRM.

### email-deliverability-auditor
Reviewing and fixing email deliverability issues. Covers DNS records (SPF, DKIM, DMARC), blacklist checks, content scoring, and sending infrastructure recommendations.

**Best for:** Investigating why emails are landing in spam or getting low open rates.

### vps-debugger
Diagnosing and fixing self-hosted service failures. Covers Docker container debugging, Caddy/HTTPS issues, disk and memory pressure, and service recovery.

**Best for:** When something on your VPS breaks and you need a systematic fix process.

### lead-sequence-builder
Designing complete lead capture → qualify → nurture systems. Covers form integration, lead scoring, CRM tagging, email sequence logic, and handoff to sales.

**Best for:** Building a full lead lifecycle system from scratch.

### design-system-builder
Creating consistent design systems in Figma or Framer. Covers token naming, component hierarchy, color and type scale, and documentation.

**Best for:** Starting or cleaning up a design system across a project.

### content-repurposer
Turning one piece of long-form content into multiple formats. Covers extraction prompts, format-specific rewrite rules, platform tone guidelines, and scheduling.

**Best for:** Maximizing reach from a single blog post, podcast, or video script.

---

## Templates

---

## Template: crm-workflow-builder.md

Save this as `.claude/skills/crm-workflow-builder.md`:

```markdown
---
name: crm-workflow-builder
description: Design and build n8n workflows that integrate with CRM systems
---

## Process

1. Clarify the trigger (webhook, schedule, manual, or CRM event)
2. Map the full data flow: source -> transform -> destination
3. List required credentials and API scopes
4. Design error handling for every external call
5. Plan how to test with a single record before enabling for all

## Rules

- Set CRM flags AFTER successful operations, never before
- Use poll-based triggers when webhook timing is uncertain
- Log every significant action as a CRM note for auditability
- Never hardcode credentials, always use the n8n credential store
- Add a timeout on every HTTP Request node
- Always handle the empty-results case in search nodes

## Testing Protocol

1. Test with a single contact or deal, not the full database
2. Verify each node's output before wiring the next
3. Confirm CRM writes happened correctly in the CRM UI
4. Run the workflow twice to confirm idempotency (no duplicates)
```

---

## Template: raycast-extension-builder.md

Save this as `.claude/skills/raycast-extension-builder.md`:

```markdown
---
name: raycast-extension-builder
description: Build Raycast extensions that connect to external APIs
---

## Process

1. Define the command type: List, Detail, or Form
2. Identify the external API and required endpoints
3. Plan the preference fields (API tokens, URLs, config)
4. Design the data flow: fetch -> transform -> display
5. Add error handling and loading states
6. Prepare store submission if publishing publicly

## Rules

- Store all API tokens as password preferences, never in source code
- Always show isLoading state while data is fetching
- Cache API responses with useCachedState to avoid rate limits
- Handle empty states explicitly (no results, error, no permissions)
- Add keyboard shortcuts to all primary actions
- Keep commands fast: under 2 seconds to first render

## Structure

src/
  index.tsx       - main command
  api.ts          - all external API calls
  types.ts        - TypeScript interfaces
  utils.ts        - helpers and formatters
```

---

## Template: framer-designer.md

Save this as `.claude/skills/framer-designer.md`:

```markdown
---
name: framer-designer
description: Design and edit Framer sites using MCP tools
---

## Process

1. Read the current project structure with getProjectXml to understand the page layout
2. Identify the specific node or component to edit using getSelectedNodesXml or getNodeXml
3. Make targeted changes via updateXmlForNode or updateCodeFile
4. Verify changes with getProjectWebsiteUrl or zoomIntoView
5. For CMS content, use getCMSCollections before writing, then upsertCMSItem

## Rules

- Always read before writing — call getNodeXml or getProjectXml first to understand the current state
- Never create duplicate color or text styles — call manageColorStyle/manageTextStyle to check existing styles first
- For code components, always readCodeFile before updating it
- Prefer editing existing nodes over creating new ones unless explicitly adding new content
- Keep CMS field names consistent with the existing collection schema — check getCMSItems before upserting
- For font changes, use searchFonts to confirm the font name exists in Framer before applying
- Batch related changes into a single session — read all needed nodes, then write all changes

## Key Checks Before Finishing

- [ ] Preview the live site URL from getProjectWebsiteUrl to confirm changes appear correctly
- [ ] Confirm CMS items show the correct field values with getCMSItems
- [ ] Verify no duplicate styles were created by listing manageColorStyle / manageTextStyle
- [ ] For code component changes, confirm the file compiled without errors by checking logs

## Common Workflows

**Update hero text on homepage:**
1. getProjectXml to find the page structure
2. getNodeXml on the hero section node ID
3. updateXmlForNode with new text content

**Add a new blog post to CMS:**
1. getCMSCollections to get the blog collection ID
2. getCMSItems to understand the field schema
3. upsertCMSItem with all required fields

**Change brand colors:**
1. manageColorStyle (list) to see existing color tokens
2. manageColorStyle (update) with new hex values — this propagates everywhere
```

---

## Template: hubspot-automator.md

Save this as `.claude/skills/hubspot-automator.md`:

```markdown
---
name: hubspot-automator
description: Build and debug HubSpot + n8n workflows for CRM automation
---

## Process

1. Clarify the HubSpot object type involved (Contact, Deal, Company, Ticket, or custom)
2. Identify the trigger: HubSpot workflow -> webhook to n8n, or n8n polling HubSpot API
3. Map which properties need to be read and written
4. Design the n8n workflow nodes in order
5. Handle HubSpot API rate limits and pagination

## HubSpot API Notes

- Rate limit: 100 requests/10 seconds (Basic), 150 for paid tiers
- Contact search: use POST /crm/v3/objects/contacts/search, not GET with filters
- Always use internal property names (e.g., `hs_lead_status`), not display labels
- Associations (Contact <-> Deal) require a separate associations API call
- Custom properties must be created in HubSpot before writing to them
- Batch upsert endpoint handles up to 100 records per call — use for bulk operations

## Rules

- Never delete HubSpot records from an automation — archive instead
- Always check if a contact exists before creating (search first, create if not found)
- Store HubSpot Private App token in n8n credential store, never in workflow expressions
- Set `lifecyclestage` only when moving forward — never downgrade lifecycle stage automatically
- Log the HubSpot record ID and timestamp in every significant workflow for auditability
- Use HubSpot workflow enrollment triggers (not n8n cron) for real-time CRM events when possible

## Common n8n Node Patterns

**Find or create contact:**
1. HTTP Request: POST /crm/v3/objects/contacts/search with email filter
2. IF node: check if results.length > 0
3. Branch A (found): return existing contact ID
4. Branch B (not found): HTTP Request: POST /crm/v3/objects/contacts with properties

**Update deal stage:**
1. HTTP Request: PATCH /crm/v3/objects/deals/{dealId}
2. Body: `{ "properties": { "dealstage": "STAGE_ID" } }`
3. Add error handling for 404 (deal not found) and 409 (conflict)

**Attach note to contact:**
1. HTTP Request: POST /crm/v3/objects/notes
2. Body: `{ "properties": { "hs_note_body": "...", "hs_timestamp": "..." } }`
3. HTTP Request: POST /crm/v4/associations/note/contact/batch/create to link it

## Testing Protocol

1. Test with a single real (non-production) contact
2. Verify property values in HubSpot CRM UI after each write
3. Check n8n execution log for actual API response codes
4. Run twice to confirm the workflow handles the "already exists" case
5. Confirm associations appear correctly on the HubSpot record timeline
```

---

## Template: email-deliverability-auditor.md

Save this as `.claude/skills/email-deliverability-auditor.md`:

```markdown
---
name: email-deliverability-auditor
description: Review and fix email deliverability issues for any sending domain
---

## Process

1. Gather the sending domain and current ESP (email service provider)
2. Check all DNS authentication records (SPF, DKIM, DMARC)
3. Check blacklist status
4. Review sending infrastructure (dedicated IP vs shared, IP age, volume ramp)
5. Audit email content for spam triggers
6. Review engagement metrics (open rate, click rate, bounce rate, unsubscribes)
7. Produce a prioritized fix list

## DNS Records to Check

**SPF (Sender Policy Framework)**
- Look up: `dig TXT yourdomain.com | grep spf`
- Should include your ESP's include tag (e.g., `include:sendgrid.net`)
- Must NOT have more than 10 DNS lookups total — exceeding this causes SPF permerror
- Only one SPF record allowed per domain — merge if multiple exist
- End with `~all` (softfail) minimum; `+all` is a red flag

**DKIM (DomainKeys Identified Mail)**
- Look up: `dig TXT selector._domainkey.yourdomain.com`
- Key length should be 2048-bit minimum (1024-bit is deprecated)
- Verify selector matches what the ESP sends (check raw email headers)
- If using multiple ESPs, each needs its own selector

**DMARC (Domain-based Message Authentication)**
- Look up: `dig TXT _dmarc.yourdomain.com`
- Start with `p=none` to monitor without blocking
- Move to `p=quarantine` then `p=reject` after 30+ days of clean reports
- Add `rua=mailto:dmarc@yourdomain.com` to receive aggregate reports
- `pct=100` applies policy to 100% of mail

**MX records**
- Verify MX records point to the correct mail server
- Check that the mail server PTR (reverse DNS) matches the sending hostname

## Blacklist Checks

Check these manually or via MXToolbox:
- Spamhaus ZEN (covers SBL, XBL, PBL)
- Barracuda BRBL
- SpamCop
- Microsoft JMRP / SNDS (for Outlook/Hotmail deliverability)

If listed: identify the cause first (spam complaint spike, compromised account, bulk unsolicited mail), fix it, then submit delisting request.

## Content Audit Rules

- Avoid all-caps subject lines and excessive punctuation (!!!, ???)
- Images-to-text ratio should be roughly 60% text, 40% images minimum
- Every email must have a working unsubscribe link
- Unsubscribe must process within 10 business days (CAN-SPAM) or immediately (Gmail/Yahoo 2024 requirement)
- Avoid spam trigger words: "free money", "guaranteed", "no credit check", "act now"
- Plain text version must exist alongside HTML version
- From name must match the brand — avoid generic names like "noreply" or "info"

## Engagement Metric Thresholds

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|---------|
| Open rate | >25% | 15–25% | <15% |
| Click rate | >2% | 0.5–2% | <0.5% |
| Hard bounce | <0.5% | 0.5–1% | >1% |
| Spam complaint | <0.08% | 0.08–0.1% | >0.1% |
| Unsubscribe | <0.2% | 0.2–0.5% | >0.5% |

Google and Yahoo now enforce complaint rate thresholds for bulk senders. Above 0.1% complaint rate triggers filtering; above 0.3% triggers blocking.

## Microsoft 365 SMTP AUTH Note

If sending through Microsoft 365 (smtp.office365.com port 587):
- SMTP AUTH must be enabled at the tenant level first (Azure AD > Security > Auth policies)
- Then enabled per-mailbox (Exchange Admin Center > Mailboxes > [mailbox] > Mail flow settings > SMTP AUTH)
- Use app passwords if MFA is enabled on the account
- Do not use basic auth with a user's main password — it will break when Microsoft enforces MFA

## Prioritized Fix Order

1. Fix SPF failures — blocks email entirely
2. Fix DKIM — required for DMARC alignment
3. Implement DMARC at p=none — start collecting data
4. Remove hard bounces from list immediately
5. Suppress high-complaint segments
6. Fix unsubscribe flow if broken
7. Tighten DMARC to quarantine/reject after 30 days clean
8. Move to dedicated sending IP if volume > 50k/mo
```

---

## Template: vps-debugger.md

Save this as `.claude/skills/vps-debugger.md`:

```markdown
---
name: vps-debugger
description: Diagnose and fix self-hosted service failures on a VPS
---

## Process

1. Establish what is broken (specific service, or whole server?)
2. Check container/process status
3. Read recent logs for error messages
4. Check system resources (disk, RAM, CPU)
5. Identify root cause
6. Apply fix
7. Verify recovery
8. Add monitoring or alerting to prevent recurrence

## Diagnostic Commands

**Container health:**
```bash
docker compose ps
docker stats --no-stream
docker inspect <container_name> --format='ExitCode: {{.State.ExitCode}}, Restarts: {{.RestartCount}}'
```

**Recent logs:**
```bash
docker logs <container_name> --tail 100 --since 1h
journalctl -u caddy --since "1 hour ago"
```

**System resources:**
```bash
df -h                          # disk usage
free -h                        # memory
uptime                         # CPU load average
du -sh /var/lib/docker/*       # Docker storage breakdown
```

**Network and ports:**
```bash
ss -tlnp                       # open ports and listeners
ufw status                     # firewall rules
curl -sv https://yourdomain.com 2>&1 | head -30  # HTTPS connectivity test
```

## Common Failure Patterns and Fixes

**Service keeps restarting (restart loop)**
- Check exit code: non-zero means crash
- Read last 20 lines of logs before the crash
- Common causes: missing env variable, wrong database password, out of disk space

**502 Bad Gateway from Caddy**
- The upstream service is down or not listening
- Verify the service is running: `docker compose ps`
- Verify it binds to the correct port: `ss -tlnp | grep 5678`
- Check Caddy reverse_proxy target matches the actual port

**SSL certificate errors**
- Run: `sudo caddy validate --config /etc/caddy/Caddyfile`
- Check DNS: `dig A yourdomain.com` must return the VPS IP
- Check Let's Encrypt rate limits (5 cert failures per hour max)
- Temporary fix: add `tls internal` to Caddyfile to use self-signed cert while debugging

**Database connection refused**
- Check if Postgres container is healthy: `docker compose ps`
- Verify credentials in `.env` match what n8n expects
- Check Postgres logs: `docker logs n8n-stack-postgres-1 --tail 30`
- Common fix: `docker compose restart postgres` then `docker compose restart n8n`

**Out of disk space**
1. `df -h` — identify full partition
2. `docker system prune -af` — remove unused images, stopped containers
3. `journalctl --vacuum-time=7d` — trim system logs
4. `find /home/deploy/backups -mtime +14 -delete` — remove old backups
5. If still full: `du -sh /* 2>/dev/null | sort -rh | head -20` to find the culprit

**High memory / OOM kills**
- Check dmesg for OOM killer: `dmesg | grep -i "oom\|killed"`
- Identify memory hog: `docker stats --no-stream --format "table {{.Name}}\t{{.MemUsage}}"`
- Ollama models load entirely into RAM — if running local LLMs, ensure enough RAM or reduce model size
- Set memory limits in docker-compose.yml: `mem_limit: 2g`

## Rules

- Always read logs before making any changes
- Never restart everything at once — restart the failing service first, then its dependencies
- Always backup before changing database config or credentials
- Document what you changed and when in a `debug.log` file on the server
- After a fix, verify the service is truly healthy (not just started — check logs for errors continuing)
- If you can't identify the root cause in 30 minutes, snapshot the VPS before making major changes

## Post-Fix Checklist

- [ ] Service shows healthy in `docker compose ps`
- [ ] No new errors in logs since the restart
- [ ] Uptime Kuma shows green for the affected monitor
- [ ] End-to-end test (hit the actual URL, trigger a test workflow, send a test email)
- [ ] Root cause documented
- [ ] Monitoring updated to catch this type of failure earlier next time
```

---

## Template: lead-sequence-builder.md

Save this as `.claude/skills/lead-sequence-builder.md`:

```markdown
---
name: lead-sequence-builder
description: Design complete lead capture → qualify → nurture → handoff systems
---

## Process

1. Define the lead source (form, ad landing page, content download, demo request, referral)
2. Design the capture mechanism (form fields, data to collect, confirmation email)
3. Build the qualification logic (scoring, tagging, routing rules)
4. Design the nurture sequence (number of touches, cadence, content per touch)
5. Define the sales handoff trigger (score threshold, action taken, time elapsed)
6. Build the CRM pipeline stages that map to the lifecycle
7. Set up tracking and reporting

## Lead Scoring Model

Score each lead 0–100 based on:

**Demographic fit (0–40 points)**
- Company size matches ICP: +15
- Industry matches ICP: +15
- Role/title matches buyer persona: +10

**Behavioral signals (0–60 points)**
- Opened email: +2 per open (cap at 10)
- Clicked link in email: +5 per click
- Visited pricing page: +15
- Watched demo video >50%: +20
- Downloaded resource: +10
- Replied to email: +25
- Booked a meeting: immediate SQL escalation

Score thresholds:
- 0–29: Not ready (continue nurture)
- 30–59: Marketing Qualified Lead (MQL) — increase frequency
- 60–79: Sales Accepted Lead (SAL) — notify sales
- 80+: Sales Qualified Lead (SQL) — immediate follow-up required

## Email Sequence Structure

**Day 0 — Confirmation**
- Trigger: form submission
- Content: delivery of what they signed up for + one sentence about what to expect next
- No pitch

**Day 1 — Value**
- Content: best resource relevant to their signup context
- CTA: read/watch, no ask to buy

**Day 3 — Problem framing**
- Content: articulate the problem this lead likely has
- CTA: "Does this sound like you?" — reply link or short survey

**Day 7 — Social proof**
- Content: specific case study or result from a similar customer
- CTA: "See how [Company] did it" — link to full case study

**Day 14 — Soft offer**
- Content: introduce your solution naturally
- CTA: low-commitment offer (free audit, short call, demo request)

**Day 21 — Last touch before cool-down**
- Content: direct question — "Is this still a priority?"
- CTA: yes/no link (if yes, book a call; if no, tag as not-now and reduce frequency)

**Day 45+ — Re-engagement (monthly)**
- Content: new resource, product update, or relevant industry news
- Goal: stay top of mind without burning the list

## CRM Pipeline Stages

1. New Lead — just captured
2. Nurturing — in email sequence, not yet MQL
3. MQL — score threshold hit or key action taken
4. SAL — sales has accepted and is reviewing
5. SQL — sales has confirmed fit and is actively working
6. Opportunity — proposal sent
7. Closed Won / Closed Lost

## Rules

- Capture only fields you will actually use — every extra field drops conversion rate
- Always send the confirmation email within 60 seconds of form submission
- Tag the lead source on every record — never let source data go unmapped
- Score based on actions, not just profile — behavioral signals outweigh demographic fit
- Never add a lead to a sales sequence without qualifying them first
- Every email must have a plain-text version and a working unsubscribe
- Sales handoff emails must include: lead name, source, score, top actions taken, and recommended next step

## n8n Workflow Structure

1. Webhook node — receives form data
2. Set node — normalize and clean field values
3. HubSpot: Search Contact — check for existing record
4. IF node — new vs existing lead
5. HubSpot: Create or Update Contact — upsert with source tags
6. HubSpot: Create Deal — if demo request or high-intent form
7. Send confirmation email (Resend)
8. Enroll in sequence (set a scheduled workflow or use HubSpot sequences)
9. Notify Slack — if score immediately qualifies as MQL

## Testing Protocol

1. Submit a test form entry with your own email
2. Verify the contact appears in CRM within 30 seconds
3. Confirm all properties mapped correctly (source, score, tags)
4. Confirm confirmation email arrives within 60 seconds
5. Advance through sequence manually to verify each email sends
6. Test the score threshold: manually update a contact's score to the MQL threshold and confirm the sales notification fires
7. Test unsubscribe: click unsubscribe and verify the contact is suppressed from further sends
```

---

## Template: design-system-builder.md

Save this as `.claude/skills/design-system-builder.md`:

```markdown
---
name: design-system-builder
description: Create and document consistent design systems in Figma or Framer
---

## Process

1. Audit existing styles and components (do not start fresh if a system exists)
2. Define the token layer: color, spacing, typography, radius, shadow, motion
3. Build the component hierarchy from primitives up
4. Document usage rules and do/don't examples
5. Connect tokens to components so changes propagate automatically
6. Export or sync to the codebase

## Token Naming Convention

Use a three-tier naming system:

**Tier 1 — Primitive tokens (raw values, not used directly in UI)**
```
color.neutral.0     → #ffffff
color.neutral.900   → #0f0f0f
color.blue.500      → #2563eb
spacing.1           → 4px
spacing.2           → 8px
spacing.4           → 16px
font.size.sm        → 14px
font.size.base      → 16px
radius.sm           → 4px
radius.md           → 8px
```

**Tier 2 — Semantic tokens (purpose-named, reference primitives)**
```
color.background.default    → color.neutral.0
color.background.subtle     → color.neutral.50
color.text.primary          → color.neutral.900
color.text.secondary        → color.neutral.500
color.accent.primary        → color.blue.500
color.border.default        → color.neutral.200
spacing.component.padding   → spacing.4
```

**Tier 3 — Component tokens (optional, for very specific overrides)**
```
button.primary.bg           → color.accent.primary
button.primary.text         → color.neutral.0
card.padding                → spacing.component.padding
```

## Component Hierarchy

Build in this order — each level uses only the levels below it:

1. **Tokens** — all raw values and semantic aliases
2. **Primitives** — Box, Stack, Text, Icon (no logic, just layout/style)
3. **Base components** — Button, Input, Badge, Avatar (single responsibility)
4. **Compound components** — Card, Modal, Dropdown, Form (composed from base)
5. **Patterns** — Hero, PricingTable, ContactForm (page-level compositions)

## Rules

- Never use raw hex values in components — always reference a token
- Dark mode must be designed at the token layer, not by creating duplicate components
- Every interactive component needs all states: default, hover, focus, active, disabled, loading
- Spacing must come from the spacing scale — no arbitrary pixel values
- Typography must use the type scale — no arbitrary font sizes
- Components should accept a `size` prop (sm/md/lg) and a `variant` prop (primary/secondary/ghost) where applicable
- All components must pass WCAG AA contrast — check with the contrast plugin before finalizing

## Figma Workflow

1. Create a dedicated "Tokens" page with all color, spacing, and type styles
2. Use Figma Variables for primitive and semantic tokens
3. Build components in a dedicated "Components" page
4. Use the components in a "Patterns" page to validate compositions
5. Every component must have a usage notes section in Figma annotations
6. Publish the Figma file as a team library so all project files consume it

## Framer Workflow

1. Use manageColorStyle to create all color tokens in Framer
2. Use manageTextStyle to create typography scale entries
3. Build code components for complex interactive components (Button, Input)
4. Use Framer's component properties (variants, booleans, strings) to expose all states
5. Document token names so the dev team can map Framer → code tokens

## Key Checks Before Shipping

- [ ] All colors pass WCAG AA contrast in both light and dark mode
- [ ] All interactive states are designed (hover, focus, disabled, loading)
- [ ] Spacing uses only tokens — no raw pixel values in any component
- [ ] Typography uses only the type scale
- [ ] Naming is consistent (no "Button Copy 2" or "Old Card")
- [ ] Components are published to the team library (Figma) or exported as code (Framer)
- [ ] A one-page usage guide exists for the team
```

---

## Template: content-repurposer.md

Save this as `.claude/skills/content-repurposer.md`:

```markdown
---
name: content-repurposer
description: Turn one piece of long-form content into multiple platform-optimized formats
---

## Process

1. Read and extract the core insight, key points, and best quotes from the source content
2. Identify which formats to produce based on platform targets
3. Rewrite each format to platform norms — do not paste the same text everywhere
4. Adapt tone per platform (professional on LinkedIn, conversational on X, visual-first for Instagram)
5. Generate a publishing schedule
6. Produce assets list (images, captions, clips needed)

## Source Content Types

- Long blog post (1,500+ words)
- Podcast episode (transcript or summary)
- YouTube video (transcript)
- Webinar or talk recording (transcript)
- Case study or research report
- Long-form newsletter issue

## Output Formats

### LinkedIn post
- Length: 150–300 words
- Structure: hook line (no "I'm excited to share") → 3–5 short punchy paragraphs → CTA
- Use line breaks generously — walls of text get no engagement
- Include 3–5 relevant hashtags at the end, not inline
- Best performing formats: numbered lists, strong contrarian takes, behind-the-scenes stories

### X (Twitter) thread
- Thread opener: bold claim or counterintuitive insight (must stand alone as a retweet)
- 5–10 tweets per thread
- Each tweet: one idea, max 250 characters
- Last tweet: summary + CTA or link
- No "🧵 Thread:" — just start with the hook

### Instagram caption
- Lead with the most relatable line (emotion first)
- 100–200 words max
- Use line breaks every 2–3 sentences
- End with a question to drive comments
- 10–15 hashtags in a comment, not the caption
- Pair with: quote graphic, carousel, or short Reel

### Email newsletter section
- Subject line options: 3 variants (curiosity, direct benefit, question)
- Preview text: complements subject, adds intrigue
- Body: 200–400 words
- One clear CTA link
- Plain and scannable — short paragraphs, occasional bold for key points

### Short video script (60–90 seconds)
- Hook: first 3 seconds must earn the watch — ask a question or make a bold claim
- Problem: 10–15 seconds
- Solution / insight: 30–40 seconds
- Proof or example: 15–20 seconds
- CTA: 5–10 seconds
- Write in spoken English — contractions, short sentences, pauses marked

### Podcast episode notes / show notes
- Title: SEO-optimized, includes the main keyword
- Description: 100–150 words, covers what the listener will learn
- Timestamps: list 5–8 chapter markers
- Resources mentioned: bulleted list with links
- Transcript: summary or full depending on platform

## Tone by Platform

| Platform | Tone | Style |
|----------|------|-------|
| LinkedIn | Professional, authoritative | First-person insights, data-backed |
| X / Twitter | Direct, punchy, opinionated | Short sentences, strong takes |
| Instagram | Aspirational, relatable | Emotional resonance, visual language |
| Email | Personal, conversational | Like writing to one person |
| YouTube | Educational, confident | Structured, example-driven |
| TikTok | Casual, high energy | Trend-aware, hooks in 2 seconds |

## Rules

- Never copy-paste the source text directly into a new format — always rewrite for the medium
- The hook is the only part that truly matters — spend 30% of your writing time on it
- Each format must be able to stand alone — do not require the audience to have seen other formats
- Call to actions must be specific — not "check out my website" but "read the full guide at [link]"
- Strip jargon for public platforms (LinkedIn, Instagram, X) unless the audience is technical
- Repurposing is not summarizing — find angles, not excerpts

## Publishing Schedule Template

Produce all formats in one session, then schedule over 2 weeks:

| Day | Platform | Format |
|-----|----------|--------|
| 0 | Blog / Newsletter | Original long-form published |
| 1 | LinkedIn | Main post from key insight |
| 2 | X | Thread |
| 3 | Instagram | Quote graphic + caption |
| 5 | Email | Newsletter section to list |
| 7 | LinkedIn | Second angle from same piece |
| 10 | X | Single tweet highlight |
| 14 | YouTube / Podcast | Long-form video or audio version |

## Key Checks Before Publishing

- [ ] Each format reads naturally for its platform — no copy-paste artifacts
- [ ] Hooks are strong and platform-appropriate
- [ ] All links are correct and tracked (UTM parameters where applicable)
- [ ] CTAs are consistent across all formats and point to the same destination
- [ ] No platform-specific formatting bleeds into another (e.g., hashtags in email, emojis in LinkedIn body)
- [ ] Scheduled in a content tool (Buffer, Hypefury, etc.) or added to content calendar
```
