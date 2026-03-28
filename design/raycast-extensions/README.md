# Raycast Extensions

Building Raycast extensions that connect to your automation stack. Raycast sits at the intersection of design, development, and daily operations. Extensions that bridge your CRM, workflows, and AI tools into a single keyboard shortcut save more time than almost anything else.

---

## Extensions Worth Building

### HubSpot Quick Actions
Search contacts, view open deals, log a quick note, and change lead status without opening a browser. Saves 2-3 context switches per hour for anyone who lives in their CRM.

### n8n Workflow Trigger
Trigger specific n8n workflows by name, view recent execution status, and see failures. Useful for workflows you run on demand rather than on a schedule.

### AI Prompt Launcher
Quick access to your saved prompt library. Select a prompt, fill in variables through a Raycast form, copy the result to clipboard. Pairs directly with the [Prompt Library](../../ai-community/prompt-library/) in this repo.

### Lead Dashboard
View your hot leads (score >= 8) from HubSpot without opening a browser. One command to see who needs attention today, with a direct link to each contact.

---

## How to Build One

### 1. Scaffold

```bash
npx create-raycast-extension@latest
```

Choose TypeScript. Pick List for data browsing, Detail for single-item views, or Form for input-heavy commands.

### 2. Project Structure

```
my-extension/
├── src/
│   ├── index.tsx      - main command entry point
│   ├── api.ts         - all external API calls
│   └── types.ts       - TypeScript interfaces
├── package.json
└── README.md
```

### 3. Basic List Command

```typescript
import { List, ActionPanel, Action } from "@raycast/api"
import { useEffect, useState } from "react"
import { fetchHotLeads } from "./api"
import type { Lead } from "./types"

export default function Command() {
  const [leads, setLeads] = useState<Lead[]>([])
  const [isLoading, setIsLoading] = useState(true)

  useEffect(() => {
    fetchHotLeads().then((data) => {
      setLeads(data)
      setIsLoading(false)
    })
  }, [])

  return (
    <List isLoading={isLoading} searchBarPlaceholder="Search hot leads...">
      {leads.map((lead) => (
        <List.Item
          key={lead.id}
          title={lead.name}
          subtitle={lead.email}
          accessories={[{ text: `Score: ${lead.score}` }]}
          actions={
            <ActionPanel>
              <Action.OpenInBrowser
                title="Open in HubSpot"
                url={`https://app.hubspot.com/contacts/${lead.id}`}
              />
            </ActionPanel>
          }
        />
      ))}
    </List>
  )
}
```

### 4. Calling External APIs

```typescript
// api.ts
import { getPreferenceValues } from "@raycast/api"

interface Preferences {
  hubspotToken: string
}

export async function fetchHotLeads() {
  const { hubspotToken } = getPreferenceValues<Preferences>()

  const res = await fetch("https://api.hubapi.com/crm/v3/objects/contacts/search", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${hubspotToken}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      filterGroups: [{
        filters: [{
          propertyName: "ai_lead_score",
          operator: "GTE",
          value: "8",
        }],
      }],
      properties: ["firstname", "lastname", "email", "ai_lead_score"],
      limit: 20,
    }),
  })

  const data = await res.json()
  return data.results.map((c: any) => ({
    id: c.id,
    name: `${c.properties.firstname} ${c.properties.lastname}`,
    email: c.properties.email,
    score: parseInt(c.properties.ai_lead_score),
  }))
}
```

### 5. Store API Tokens Securely

In `package.json`:

```json
"preferences": [
  {
    "name": "hubspotToken",
    "title": "HubSpot Private App Token",
    "type": "password",
    "required": true
  }
]
```

Raycast stores these in the system keychain. Never hardcode tokens in source files.

---

## Performance Tips

**Cache API responses.** Use `useCachedState` from `@raycast/utils` for data that doesn't change every second. HubSpot has rate limits and Raycast users expect speed.

**Always show loading states.** Users expect instant feedback. Anything slower than 200ms needs a spinner.

**Add keyboard shortcuts.** Power users navigate entirely by keyboard. Every primary action should have a shortcut.

**Handle empty states explicitly.** Show a useful message when there are no results, not a blank screen.

---

## Triggering n8n Workflows from Raycast

```typescript
export async function triggerWorkflow(webhookUrl: string, payload: object) {
  const res = await fetch(webhookUrl, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(payload),
  })

  if (!res.ok) {
    throw new Error(`Workflow trigger failed: ${res.status}`)
  }

  return res.json()
}
```

In n8n, use a Webhook trigger node set to "POST" and copy the webhook URL into your Raycast extension preferences.

---

## Publishing to the Raycast Store

1. Fork [raycast/extensions](https://github.com/raycast/extensions) on GitHub
2. Add your extension in a new folder under `extensions/`
3. Follow their README requirements (screenshots, description, categories)
4. Open a PR

For private use only: skip publishing, run `npm run dev` in Raycast to load it locally.
