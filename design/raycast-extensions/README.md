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

### Daily Standup Generator
Pull your GitHub commits, Jira tickets, and calendar events from the past 24 hours and generate a standup summary. One keystroke, paste into Slack.

### GitHub PR Queue
See all open PRs waiting for your review across all repos. Filter by age, repo, or author. Open in browser, approve, or request changes directly from Raycast.

### Bookmark Launcher with AI Search
Semantic search over your browser bookmarks. Instead of remembering exact titles, type a description and the AI finds the right URL. Uses a local embeddings index updated on save.

### Meeting Prep Brief
Before a calendar event, pull the attendee's LinkedIn summary, last email thread, last CRM note, and any open deal. Surfaces as a Detail view you open 5 minutes before the call.

### Invoice Generator
Fill out a Raycast form (client name, line items, rate, hours), generate a PDF invoice via an n8n workflow, and get a Dropbox link back — all without opening a browser.

### Snippet Expander with Variables
A smarter text snippet tool. Store templates like "cold email opener for {{industry}} companies", fill in variables through a Raycast Form, and copy the result. No app switching.

### DNS Lookup
Quick DNS record lookup for any domain. A/AAAA, MX, TXT (SPF/DKIM/DMARC), NS — displayed as a clean Detail view. Saves opening a web-based DNS tool for quick checks.

### Resend Email Preview
See the last 20 emails sent through your Resend account, view open/click data, and resend to a test address. For monitoring transactional email health.

### Time Zone World Clock
Show current time in your configured list of cities. As a MenuBar command so it's always visible. Click a city to copy its time to clipboard (for scheduling across time zones).

### Airtable Record Creator
Submit records to a specific Airtable base/table through a Raycast Form. Useful for quick logging: bug reports, content ideas, expense entries, anything you capture repeatedly.

### Screenshot to Figma
Take a screenshot region, upload it to a Figma file as a new frame. One keyboard shortcut to capture → annotate in Figma. Saves the import workflow.

### Vercel Deployment Status
See the latest deployment status for all your Vercel projects. One command to check if a deploy is building, succeeded, or failed. Open logs directly in browser.

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

## Form Command Example

Use `Form` when you need structured input before performing an action — creating a record, triggering a workflow with parameters, generating a document.

```typescript
import { Form, ActionPanel, Action, showToast, Toast } from "@raycast/api"
import { useState } from "react"
import { createAirtableRecord } from "./api"

interface Values {
  title: string
  description: string
  priority: string
  dueDate: Date
}

export default function CreateTaskCommand() {
  const [isLoading, setIsLoading] = useState(false)

  async function handleSubmit(values: Values) {
    setIsLoading(true)
    try {
      await createAirtableRecord({
        Title: values.title,
        Description: values.description,
        Priority: values.priority,
        "Due Date": values.dueDate.toISOString().split("T")[0],
      })
      await showToast({
        style: Toast.Style.Success,
        title: "Task created",
        message: values.title,
      })
    } catch (error) {
      await showToast({
        style: Toast.Style.Failure,
        title: "Failed to create task",
        message: String(error),
      })
    } finally {
      setIsLoading(false)
    }
  }

  return (
    <Form
      isLoading={isLoading}
      actions={
        <ActionPanel>
          <Action.SubmitForm title="Create Task" onSubmit={handleSubmit} />
        </ActionPanel>
      }
    >
      <Form.TextField
        id="title"
        title="Title"
        placeholder="What needs to be done?"
        autoFocus
      />
      <Form.TextArea
        id="description"
        title="Description"
        placeholder="Optional details..."
        enableMarkdown
      />
      <Form.Dropdown id="priority" title="Priority" defaultValue="medium">
        <Form.Dropdown.Item value="low" title="Low" />
        <Form.Dropdown.Item value="medium" title="Medium" />
        <Form.Dropdown.Item value="high" title="High" />
      </Form.Dropdown>
      <Form.DatePicker id="dueDate" title="Due Date" type={Form.DatePicker.Type.Date} />
    </Form>
  )
}
```

---

## Detail Command Example

Use `Detail` for rich single-item views — contact profiles, deal summaries, meeting briefs, document previews. Supports Markdown rendering.

```typescript
import { Detail, ActionPanel, Action } from "@raycast/api"
import { useEffect, useState } from "react"
import { fetchContactDetail } from "./api"

export default function ContactDetailCommand({ contactId }: { contactId: string }) {
  const [contact, setContact] = useState<ContactDetail | null>(null)
  const [isLoading, setIsLoading] = useState(true)

  useEffect(() => {
    fetchContactDetail(contactId).then((data) => {
      setContact(data)
      setIsLoading(false)
    })
  }, [contactId])

  const markdown = contact
    ? `
# ${contact.name}

**Company:** ${contact.company}
**Title:** ${contact.title}
**Email:** ${contact.email}

## Last Activity
${contact.lastNote}

## Open Deals
${contact.deals.map((d) => `- **${d.name}** — ${d.stage} — $${d.amount}`).join("\n")}
    `.trim()
    : "Loading..."

  return (
    <Detail
      isLoading={isLoading}
      markdown={markdown}
      metadata={
        contact ? (
          <Detail.Metadata>
            <Detail.Metadata.Label title="Lead Score" text={String(contact.score)} />
            <Detail.Metadata.Label title="Lifecycle Stage" text={contact.stage} />
            <Detail.Metadata.Separator />
            <Detail.Metadata.Link
              title="HubSpot Profile"
              target={`https://app.hubspot.com/contacts/${contactId}`}
              text="Open"
            />
          </Detail.Metadata>
        ) : undefined
      }
      actions={
        <ActionPanel>
          <Action.OpenInBrowser
            title="Open in HubSpot"
            url={`https://app.hubspot.com/contacts/${contactId}`}
          />
          <Action.CopyToClipboard
            title="Copy Email"
            content={contact?.email ?? ""}
            shortcut={{ modifiers: ["cmd"], key: "e" }}
          />
        </ActionPanel>
      }
    />
  )
}
```

---

## MenuBar Command Example

MenuBar commands run in the background and show in the macOS menu bar. Ideal for things you want always visible: clock, deploy status, unread count, active timer.

```typescript
import { MenuBarExtra, open } from "@raycast/api"
import { useCachedPromise } from "@raycast/utils"
import { fetchDeployStatus } from "./api"

export default function DeployStatusMenuBar() {
  const { data: deploys, isLoading } = useCachedPromise(fetchDeployStatus, [], {
    keepPreviousData: true,
  })

  const latestDeploy = deploys?.[0]
  const statusEmoji =
    latestDeploy?.state === "READY" ? "✓" :
    latestDeploy?.state === "ERROR" ? "✗" :
    latestDeploy?.state === "BUILDING" ? "⟳" : "?"

  return (
    <MenuBarExtra
      icon={{ source: "deploy-icon.png" }}
      title={isLoading ? "..." : `${statusEmoji} ${latestDeploy?.name ?? "No deploys"}`}
      isLoading={isLoading}
    >
      {deploys?.map((deploy) => (
        <MenuBarExtra.Item
          key={deploy.id}
          title={deploy.name}
          subtitle={deploy.state}
          onAction={() => open(deploy.url)}
        />
      ))}
      <MenuBarExtra.Separator />
      <MenuBarExtra.Item
        title="Open Vercel Dashboard"
        onAction={() => open("https://vercel.com/dashboard")}
      />
    </MenuBarExtra>
  )
}
```

---

## useCachedPromise and useFetch Hooks

These hooks from `@raycast/utils` are the recommended way to fetch data. They handle caching, loading states, and revalidation automatically.

### useCachedPromise

Best for: custom async functions (API calls with auth, complex transformations).

```typescript
import { useCachedPromise } from "@raycast/utils"
import { fetchHotLeads } from "./api"

export default function Command() {
  const { data: leads, isLoading, revalidate } = useCachedPromise(
    fetchHotLeads,
    [], // arguments to pass to fetchHotLeads
    {
      keepPreviousData: true, // show stale data while refreshing
      initialData: [],
    }
  )

  return (
    <List
      isLoading={isLoading}
      actions={
        <ActionPanel>
          <Action title="Refresh" onAction={revalidate} shortcut={{ modifiers: ["cmd"], key: "r" }} />
        </ActionPanel>
      }
    >
      {leads.map((lead) => (
        <List.Item key={lead.id} title={lead.name} />
      ))}
    </List>
  )
}
```

### useFetch

Best for: simple REST endpoints where you just need the JSON response.

```typescript
import { useFetch } from "@raycast/utils"
import { getPreferenceValues } from "@raycast/api"

interface Preferences { apiKey: string }

export default function StatusPage() {
  const { apiKey } = getPreferenceValues<Preferences>()

  const { data, isLoading } = useFetch(
    "https://api.statuspage.io/v1/pages/YOUR_PAGE_ID/incidents.json",
    {
      headers: { Authorization: `OAuth ${apiKey}` },
      mapResult(result: Incident[]) {
        return {
          data: result.filter((i) => i.status !== "resolved"),
        }
      },
      keepPreviousData: true,
    }
  )

  return (
    <List isLoading={isLoading}>
      {data?.map((incident) => (
        <List.Item key={incident.id} title={incident.name} subtitle={incident.status} />
      ))}
    </List>
  )
}
```

---

## Toast Notifications

Toasts are the primary feedback mechanism for async actions. Always show toasts for operations that take > 200ms or can fail.

```typescript
import { showToast, Toast } from "@raycast/api"

// Animated loading toast → update to success or failure
async function performAction() {
  const toast = await showToast({
    style: Toast.Style.Animated,
    title: "Sending email...",
  })

  try {
    await sendEmail(payload)
    toast.style = Toast.Style.Success
    toast.title = "Email sent"
    toast.message = "Check your sent folder"
  } catch (error) {
    toast.style = Toast.Style.Failure
    toast.title = "Failed to send"
    toast.message = String(error)
  }
}
```

---

## confirmAlert Dialogs

Use `confirmAlert` before destructive or irreversible actions.

```typescript
import { confirmAlert, Alert, showToast, Toast } from "@raycast/api"

async function deleteContact(contact: Contact) {
  const confirmed = await confirmAlert({
    title: "Delete Contact",
    message: `Are you sure you want to delete ${contact.name}? This cannot be undone.`,
    primaryAction: {
      title: "Delete",
      style: Alert.ActionStyle.Destructive,
    },
  })

  if (confirmed) {
    await deleteFromHubSpot(contact.id)
    await showToast({ style: Toast.Style.Success, title: "Contact deleted" })
  }
}
```

---

## Navigation Between Views

Push new views onto the navigation stack with `useNavigation` or `<Action.Push>`.

```typescript
import { List, ActionPanel, Action, Detail, useNavigation } from "@raycast/api"

// Option 1: Action.Push (declarative)
<Action.Push
  title="View Details"
  target={<ContactDetail contactId={contact.id} />}
  shortcut={{ modifiers: ["cmd"], key: "return" }}
/>

// Option 2: useNavigation (imperative, for complex flows)
function LeadList() {
  const { push } = useNavigation()

  return (
    <List>
      <List.Item
        title="John Doe"
        actions={
          <ActionPanel>
            <Action
              title="View Profile"
              onAction={() => push(<MeetingBrief contactId="123" />)}
            />
          </ActionPanel>
        }
      />
    </List>
  )
}
```

---

## AI Integration in Extensions

### Using OpenAI

```typescript
import { getPreferenceValues, showToast, Toast } from "@raycast/api"

interface Preferences {
  openaiKey: string
}

export async function generateWithAI(prompt: string): Promise<string> {
  const { openaiKey } = getPreferenceValues<Preferences>()

  const res = await fetch("https://api.openai.com/v1/chat/completions", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${openaiKey}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      model: "gpt-4o",
      messages: [{ role: "user", content: prompt }],
      max_tokens: 500,
    }),
  })

  const data = await res.json()
  return data.choices[0].message.content
}
```

### Using Claude (Anthropic)

```typescript
export async function generateWithClaude(prompt: string): Promise<string> {
  const { anthropicKey } = getPreferenceValues<{ anthropicKey: string }>()

  const res = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: {
      "x-api-key": anthropicKey,
      "anthropic-version": "2023-06-01",
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      model: "claude-opus-4-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: prompt }],
    }),
  })

  const data = await res.json()
  return data.content[0].text
}
```

### Pattern: AI + Form

```typescript
// User fills form → submit → AI generates → copy result
async function handleSubmit(values: { context: string; tone: string }) {
  const toast = await showToast({ style: Toast.Style.Animated, title: "Generating..." })

  const result = await generateWithClaude(
    `Write a cold email for this context: ${values.context}. Tone: ${values.tone}`
  )

  await Clipboard.copy(result)
  toast.style = Toast.Style.Success
  toast.title = "Copied to clipboard"
}
```

---

## Local File System Access

```typescript
import { environment } from "@raycast/api"
import { readFileSync, writeFileSync, existsSync } from "fs"
import { join } from "path"

// Raycast provides a persistent support directory per extension
const cacheFile = join(environment.supportPath, "cache.json")

export function readCache<T>(): T | null {
  if (!existsSync(cacheFile)) return null
  try {
    return JSON.parse(readFileSync(cacheFile, "utf-8")) as T
  } catch {
    return null
  }
}

export function writeCache<T>(data: T): void {
  writeFileSync(cacheFile, JSON.stringify(data, null, 2), "utf-8")
}
```

`environment.supportPath` — persistent, survives restarts. Good for caches, user data.
`environment.assetsPath` — read-only assets bundled with the extension (icons, templates).

---

## Clipboard Management

```typescript
import { Clipboard, showHUD } from "@raycast/api"

// Copy text
await Clipboard.copy("text to copy")
await showHUD("Copied!")  // small overlay notification (no UI required)

// Copy rich content
await Clipboard.copy({ html: "<b>bold text</b>", text: "bold text" })

// Read from clipboard
const clipboardContent = await Clipboard.readText()

// Paste into the frontmost app
await Clipboard.paste("text to paste")
```

---

## Running Shell Commands

```typescript
import { exec } from "child_process"
import { promisify } from "util"
import { showToast, Toast } from "@raycast/api"

const execAsync = promisify(exec)

export async function runGitCommand(repoPath: string, command: string) {
  try {
    const { stdout, stderr } = await execAsync(`git ${command}`, { cwd: repoPath })
    return stdout.trim()
  } catch (error: any) {
    await showToast({
      style: Toast.Style.Failure,
      title: "Git command failed",
      message: error.message,
    })
    throw error
  }
}

// Example: get current branch name for the active project
const branch = await runGitCommand("/Users/you/projects/myapp", "rev-parse --abbrev-ref HEAD")
```

For longer-running shell commands, stream output into a Detail view with a live-updating markdown string.

---

## Deep Linking

Deep links open a specific command directly from a URL or keyboard shortcut outside Raycast.

```
raycast://extensions/your-username/your-extension/command-name
```

With arguments:
```
raycast://extensions/your-username/crm-tools/view-contact?arguments={"contactId":"12345"}
```

In the command, receive arguments:

```typescript
import { LaunchProps } from "@raycast/api"

interface Arguments {
  contactId: string
}

export default function Command({ arguments: args }: LaunchProps<{ arguments: Arguments }>) {
  // args.contactId is available immediately
  return <ContactDetail contactId={args.contactId} />
}
```

In `package.json`, declare the argument:
```json
"arguments": [
  {
    "name": "contactId",
    "placeholder": "Contact ID",
    "type": "text",
    "required": true
  }
]
```

---

## Working with Images and Icons

```typescript
import { List, Icon, Color, Image } from "@raycast/api"

// Built-in Raycast icons (SVG, theme-aware)
<List.Item icon={Icon.Person} title="John Doe" />

// Tinted icons for status indication
<List.Item
  icon={{ source: Icon.Circle, tintColor: Color.Green }}
  title="Active"
/>

// Remote image (URL)
<List.Item
  icon={{ source: "https://avatars.githubusercontent.com/u/123" }}
  title="GitHub Avatar"
/>

// Local asset from extension's assets/ folder
<List.Item
  icon={{ source: "hubspot-logo.png" }}
  title="HubSpot"
/>

// Circular avatar from URL
<List.Item
  icon={{
    source: contact.avatarUrl ?? Icon.Person,
    mask: Image.Mask.Circle,
  }}
  title={contact.name}
/>
```

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

## Performance Tips

**Cache API responses.** Use `useCachedPromise` or `useCachedState` from `@raycast/utils` for data that doesn't change every second. HubSpot has rate limits and Raycast users expect speed.

**Always show loading states.** Users expect instant feedback. Anything slower than 200ms needs a spinner.

**Add keyboard shortcuts.** Power users navigate entirely by keyboard. Every primary action should have a shortcut.

**Handle empty states explicitly.** Show a useful message when there are no results, not a blank screen.

**Debounce search.** If your list searches an API on every keystroke, add a debounce:

```typescript
const [query, setQuery] = useState("")
const debouncedQuery = useDebounce(query, 300)

const { data } = useCachedPromise(searchContacts, [debouncedQuery], {
  execute: debouncedQuery.length > 2,
})
```

---

## Publishing to the Raycast Store

1. Fork [raycast/extensions](https://github.com/raycast/extensions) on GitHub
2. Add your extension in a new folder under `extensions/`
3. Follow their README requirements (screenshots, description, categories)
4. Open a PR

For private use only: skip publishing, run `npm run dev` in Raycast to load it locally.
