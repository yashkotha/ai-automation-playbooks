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
