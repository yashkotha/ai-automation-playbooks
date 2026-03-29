# Design + Dev Track

This track is for designers and developers who use AI to compress the distance between an idea and working software. It covers three high-leverage workflows: building and managing Framer sites with Claude, converting Figma designs into production-ready component code, and writing Raycast extensions that connect your daily tools without opening a browser. Each section is self-contained but they connect: a Framer form submits to an n8n webhook, a Raycast command queries the HubSpot API you built in the automation track, and the component patterns from design-to-code ship into the same Next.js app your n8n workflows power.

---

## What's Inside

| Folder | Focus | Start Here If... |
|---|---|---|
| [`./framer-ai/`](#framer--ai) | Framer site building with Claude + MCP | You build marketing sites or landing pages in Framer |
| [`./design-to-code/`](#design-to-code) | Figma to production component code | You work from Figma files and hand off to or write React code |
| [`./raycast-extensions/`](#raycast-extensions) | Custom Raycast commands for your tools | You want keyboard-first access to HubSpot, n8n, Notion, GitHub |

---

## Framer + AI

**Path:** `./framer-ai/`

Everything you need to use Claude Code as your primary Framer collaborator. Covers setup through publishing.

### Setup and MCP Configuration
- Installing Framer MCP in Claude Code via `claude mcp add`
- Connecting to a live Framer project by sharing the project URL
- Environment setup: Node version requirements, local dev server, Claude Code project config
- Verifying the connection with `getProjectXml` and reading the current project tree

### Design with AI
- Generating full page layouts from a text prompt using `updateXmlForNode`
- Building color systems: generating a 10-stop palette, semantic token naming, dark/light variants
- Typography scales: fluid type with `clamp()`, modular scale ratios, pairing fonts via Claude prompts
- Component variants: asking Claude to generate all interactive states (default, hover, focus, disabled, loading)
- Responsive breakpoints: mobile-first layouts, container width tokens, stacking rules for small screens

### Component Library
Full component set with code examples for each:

| Component | Variants Covered |
|---|---|
| Button | Primary, secondary, ghost, destructive, icon-only, loading state, disabled |
| Card | Default, elevated, interactive (hover lift), media card, stat card |
| Navigation | Desktop (horizontal nav with dropdowns), mobile (hamburger + slide-out drawer) |
| Hero | Centered text, split layout, background image, video background |
| Feature Grid | 2-column, 3-column, icon + text, alternating image/text |
| Testimonial | Single quote, carousel, grid of cards, avatar + rating |
| Pricing | 2-tier, 3-tier with highlighted plan, toggle monthly/annual |
| Footer | Minimal, multi-column with links, with newsletter form |

### Animations
- Scroll-triggered reveals with `withScrollReveal`: fade-up, fade-in, slide-from-left
- Parallax effects with `withParallax`: background layer, foreground layer, depth ratios
- Sticky header: scroll position detection, background blur on scroll, shrink on scroll
- Staggered entrance animations: list items, grid cards, nav links cascading in
- Page transitions with Framer Motion: `AnimatePresence`, `motion.div` with `initial`/`animate`/`exit`
- `useReducedMotion` hook integration for accessibility compliance

### CMS
- Setting up collections with `createCMSCollection`: field types, slugs, required fields
- Dynamic pages: binding collection data to page templates, URL structure
- Filtering: tag-based filters, category dropdowns, client-side vs. server-side
- Search: full-text search over CMS fields, debounced input
- Pagination: page size, prev/next navigation, total count display
- Blog index: filterable tag list, featured post slot, date sorting, read time calculation

### Framer MCP Tools Reference

| Tool | What It Does |
|---|---|
| `getProjectXml` | Read the full project node tree as XML |
| `getNodeXml` | Read a specific node by ID |
| `updateXmlForNode` | Replace a node's XML with new content |
| `createPage` | Add a new page to the project |
| `createCodeFile` | Create a new code override or component file |
| `updateCodeFile` | Edit an existing code file |
| `upsertCMSItem` | Create or update a CMS record |
| `getCMSCollections` | List all CMS collections and their schemas |
| `manageColorStyle` | Create or update a color style |
| `manageTextStyle` | Create or update a text style |

### Code Overrides
- `withN8nForm`: connects any Framer form to an n8n webhook URL; handles loading state, success message, error fallback
- `CookieBanner`: GDPR-compliant cookie consent; stores preference in localStorage; hides on accept
- `GitHubStatsCard`: fetches stars, forks, last commit from GitHub REST API and renders as a live stat block

### Publishing and SEO
- Meta tags: title, description, canonical URL via Framer site settings
- Open Graph: `og:title`, `og:description`, `og:image` with proper image dimensions
- Sitemap: auto-generated by Framer, verifying it includes CMS dynamic pages
- `robots.txt`: default Framer config, adding `Disallow` rules for staging subdomains
- Custom domain: DNS configuration, SSL provisioning wait times, www redirect
- Framer Analytics: enabling built-in analytics, reading page view data

### Plugin Ecosystem
Plugins that integrate well with AI-assisted workflows:

| Plugin | Use Case |
|---|---|
| Fonts | Browse and apply Google Fonts or custom fonts |
| Icons | Import Lucide, Phosphor, or custom SVG icon sets |
| Unsplash | Pull placeholder images directly into frames |
| Spline | Embed 3D scenes as background elements |
| Lottie | Add Lottie animations as code overrides |

### AI Prompt Library for Framer
15+ ready-to-use prompts organized by task:

- **Layout generation:** "Generate a SaaS landing page with hero, 3-feature grid, pricing section, and footer. Use a dark background with a blue accent color."
- **Component creation:** "Create a Card component with an image slot, title, body text, and a CTA button. Include hover state with subtle lift and shadow."
- **Color system:** "Generate a 10-stop color scale for a brand primary color of #4F46E5. Include semantic tokens for background, surface, border, text, and interactive states."
- **Typography:** "Create a fluid type scale using clamp() for a viewport range of 375px to 1440px. Include h1-h4, body large, body, body small, label, and caption."
- **Animation:** "Add a scroll-triggered fade-up animation to each card in the feature grid with a 100ms stagger between each item."
- **CMS setup:** "Create a Blog CMS collection with fields: title (text), slug (text), publishedAt (date), coverImage (image), excerpt (text), body (rich text), tags (multi-select)."
- **Responsive layout:** "Convert this desktop hero section to stack vertically on mobile. Text should be centered on mobile and left-aligned on desktop."
- **Dark mode:** "Add a dark mode toggle that stores the preference in localStorage and applies a CSS class to the root element."
- **Form integration:** "Add a contact form with name, email, and message fields. On submit, POST to this n8n webhook URL and show a success message."
- **Navigation:** "Create a responsive navigation with a logo on the left, 5 nav links in the center, and a CTA button on the right. On mobile, collapse to a hamburger menu."
- Plus 5 more prompts covering: testimonial carousels, pricing toggles, sticky announcement bars, cookie banners, and page transition effects

### Common Patterns
- Dark/light mode toggle: CSS custom properties approach, system preference detection with `prefers-color-scheme`, manual override with localStorage
- Mobile navigation: focus trap on open, scroll lock on body, `Escape` key to close, ARIA attributes
- Dynamic content from external APIs: `useEffect` fetch with loading skeleton, error state, SWR-style caching

---

## Design to Code

**Path:** `./design-to-code/`

A workflow for converting Figma designs into accessible, production-quality React components. Covers everything from extracting design tokens to visual regression testing.

### Figma MCP Workflow
- Using `get_design_context` to extract component specs, spacing, colors, and typography from a Figma file
- Reading component properties: auto-layout direction, padding, gap, fill, stroke, corner radius
- Generating React component code from a Figma frame: asking Claude to interpret layout constraints
- Handling Figma-specific patterns that don't map directly to CSS (auto-layout wrap, absolute positioning within frames)

### Component Libraries Deep-Dives
- **shadcn/ui**: installing with `npx shadcn@latest init`, adding components with `npx shadcn@latest add`, customizing via `components.json`, extending with `className` and `cva`
- **Radix UI primitives**: `@radix-ui/react-dialog`, `@radix-ui/react-dropdown-menu`, `@radix-ui/react-select` with custom styling and compound component patterns
- **Headless UI**: `Combobox`, `Listbox`, `Disclosure` components with Tailwind CSS styling

### Design Token Pipeline
Full pipeline from Figma to code:

```
Figma Variables
    -> Export as JSON via Figma Tokens plugin
    -> CSS custom properties (--color-primary-500: #4F46E5)
    -> Tailwind config (colors.primary[500]: 'var(--color-primary-500)')
    -> Component tokens (Button: bg-primary-500 hover:bg-primary-600)
```

- Figma Variables: color, typography, spacing, radius, shadow variable collections
- CSS custom properties: semantic naming, light/dark theme overrides on `:root` and `[data-theme="dark"]`
- Tailwind config: extending the default theme, `tailwind-merge` for conditional classes
- Component-level tokens: using `cva` from `class-variance-authority` to encode variants

### Accessibility Patterns
- ARIA roles: when to use `role="button"` vs. native `<button>`, landmark roles (`main`, `nav`, `aside`)
- Keyboard navigation: tab order, focus indicators, arrow key navigation in menus and comboboxes
- Focus management: trapping focus in modals with `@radix-ui/react-focus-trap`, restoring focus on close
- Screen reader testing: VoiceOver on macOS, NVDA on Windows, common failure patterns to check
- Color contrast: WCAG AA minimums (4.5:1 text, 3:1 UI), checking with the `axe-core` API

### Dark Mode Implementation
- CSS variables approach: all color values as custom properties, swapped via `[data-theme="dark"]` selector
- System preference detection: `window.matchMedia('(prefers-color-scheme: dark)')` with event listener
- Manual override: storing user preference in `localStorage`, applying on page load before first paint (no flash)
- Next.js integration: `next-themes` package, `ThemeProvider` wrapper, `useTheme` hook

### Complex Responsive Layouts
- Grid systems: CSS Grid with `auto-fill` and `minmax()`, named grid lines for complex layouts
- Container queries: `@container` with `cq-*` properties for component-level breakpoints
- Fluid typography: `clamp(1rem, 2.5vw, 1.5rem)` for size, line-height, and letter-spacing
- Intrinsic layouts: `fit-content`, `min-content`, `max-content` for natural sizing

### Framer Motion Integration
- `motion.div` with `initial`, `animate`, `exit` for component-level animations
- `AnimatePresence` for exit animations on unmount
- `useReducedMotion()` hook: wrapping all animation configs to respect `prefers-reduced-motion`
- Layout animations: `layout` prop for smooth reflow when list items are added or removed
- Shared layout transitions: `layoutId` for animating elements across page transitions

### Next.js App Router Patterns
- Server Components: fetching data directly in `async` components, no `useEffect`
- `loading.tsx`: skeleton UI that renders immediately while async components resolve
- `error.tsx`: error boundary with retry button, scoped to a route segment
- `Suspense` boundaries: wrapping individual async components for granular loading states
- Route handlers in `app/api/`: type-safe request/response with Zod validation

### Visual Testing
- Storybook: `@storybook/nextjs` setup, writing stories for each component variant and state
- Chromatic: connecting to GitHub CI, snapshot testing, visual diff review workflow
- Percy: alternative to Chromatic, DOM snapshots vs. screenshot comparison tradeoffs

### Figma-to-Code Gaps Reference
Common places where AI-generated code diverges from the Figma design:

| Figma Behavior | What AI Often Generates | Correct Fix |
|---|---|---|
| Auto-layout with "hug contents" | Fixed height | `height: fit-content` or no height |
| Absolute position inside frame | `position: fixed` | `position: absolute` inside `position: relative` parent |
| Figma shadow (multiple layers) | Single `box-shadow` | Multiple comma-separated `box-shadow` values |
| Stroke inside | `border` (outside by default) | `box-shadow: inset 0 0 0 1px` |
| Overflow hidden on frame | No overflow rule | `overflow: hidden` on the container |
| Fill with opacity | `rgba()` on background | `background-color` with opacity, not on text color |

### Complete Workflow
End-to-end path from Figma file to production:

1. Extract design tokens from Figma Variables with Figma MCP or Figma Tokens plugin
2. Configure Tailwind + CSS custom properties from the token JSON
3. Generate component scaffold with Claude using Figma MCP context
4. Fix gaps using the reference table above
5. Write Storybook stories for all variants
6. Connect Chromatic to CI for visual regression baseline
7. Ship to production via Next.js App Router

---

## Raycast Extensions

**Path:** `./raycast-extensions/`

Build keyboard-first tools that connect your stack without opening a browser. Covers all four Raycast command types, API integration patterns, and publishing.

### Extension Ideas with Full Specs
15+ fully specced extensions ready to build:

| Extension | Command Type | APIs Used |
|---|---|---|
| HubSpot Contact Lookup | List | HubSpot Contacts API v3 |
| n8n Workflow Runner | List + Form | n8n REST API |
| Pinecone Semantic Search | List | Pinecone Query API |
| Lead Qualifier | Form + Detail | HubSpot + OpenAI API |
| Email Template Picker | List + Detail | HubSpot Transactional Email API |
| AI Text Tools | Form | Anthropic Messages API |
| GitHub PR Reviewer | List + Detail | GitHub REST API v3 |
| Notion Page Creator | Form | Notion API v1 |
| Figma Component Search | List | Figma REST API |
| Daily Digest Reader | MenuBar + Detail | Multiple RSS/API sources |
| Color Token Converter | Form | Local computation |
| Deployment Status Checker | MenuBar | Vercel API |
| HubSpot Deal Pipeline | List | HubSpot Deals API v3 |
| n8n Error Monitor | MenuBar | n8n Executions API |
| Pinecone Namespace Browser | List | Pinecone Index Stats API |

Each spec includes: command type, required API keys, data schema, UI layout, and error handling notes.

### All 4 Command Types with Code Examples

**List** -- search and browse records:
```tsx
export default function Command() {
  const { isLoading, data } = useFetch<Contact[]>(
    "https://api.hubapi.com/crm/v3/objects/contacts?limit=50",
    { headers: { Authorization: `Bearer ${prefs.hubspotToken}` } }
  );
  return (
    <List isLoading={isLoading} searchBarPlaceholder="Search contacts...">
      {data?.map((c) => (
        <List.Item key={c.id} title={c.properties.firstname} subtitle={c.properties.email} />
      ))}
    </List>
  );
}
```

**Detail** -- view a single record with rich markdown:
```tsx
<Detail markdown={`# ${contact.name}\n\n**Email:** ${contact.email}`} />
```

**Form** -- collect input and trigger an action:
```tsx
<Form actions={<ActionPanel><Action.SubmitForm onSubmit={handleSubmit} /></ActionPanel>}>
  <Form.TextField id="query" title="Search Query" />
</Form>
```

**MenuBar** -- persistent status in the menu bar:
```tsx
export default function Command() {
  const { data } = useCachedPromise(fetchDeploymentStatus);
  return <MenuBarExtra icon={data?.ok ? Icon.Checkmark : Icon.XMarkCircle} title={data?.status} />;
}
```

### API Hooks

| Hook | When to Use |
|---|---|
| `useCachedPromise` | Async functions (HubSpot API, custom fetchers) with automatic caching |
| `useFetch` | Simple GET requests with built-in loading/error state |
| `useCachedState` | Local state that persists between command opens |

- `useCachedPromise` with `keepPreviousData: true` for seamless search-as-you-type
- Cache invalidation: calling `revalidate()` after a mutation
- `execute` option on `useCachedPromise` for conditional fetching

### Toast Lifecycle
```tsx
const toast = await showToast({ style: Toast.Style.Animated, title: "Running workflow..." });
try {
  await triggerWorkflow(id);
  toast.style = Toast.Style.Success;
  toast.title = "Workflow triggered";
} catch (e) {
  toast.style = Toast.Style.Failure;
  toast.title = "Failed";
  toast.message = e.message;
}
```

### Navigation Patterns
- Push/pop: `<Action.Push name="View Details" target={<DetailView item={item} />} />`
- Detail panels: `<List.Item detail={<List.Item.Detail markdown={...} />} />`
- Drill-down: List of deals -> push to contacts associated with deal -> push to contact detail

### AI Integration
- OpenAI API: `openai.chat.completions.create` with streaming response rendered into a Detail view
- Anthropic Messages API: `client.messages.create` with `claude-sonnet-4-6`, streaming to `markdown` prop
- Keeping API keys in Raycast Preferences (never hardcoded): `getPreferenceValues<Preferences>()`
- Streaming text into a Detail view as it arrives: updating state on each `stream.text_delta` event

### File System, Clipboard, Shell

```tsx
// Read a local file
import { readFile } from "fs/promises";
const content = await readFile("/path/to/file", "utf8");

// Write to clipboard
await Clipboard.copy(result);

// Run a shell command
import { execAsync } from "@raycast/api";
const { stdout } = await execAsync("git log --oneline -10");
```

### Deep Linking
- Open a URL in default browser: `open("https://app.hubspot.com/contacts/...")`
- Open in specific app: `open("figma://file/abc123")`
- Raycast deep links to other Raycast commands: `raycast://extensions/author/extension/command`

### Preferences System
```tsx
interface Preferences {
  hubspotToken: string;
  n8nBaseUrl: string;
  n8nApiKey: string;
}
const prefs = getPreferenceValues<Preferences>();
```
- Defined in `package.json` under `preferences`
- Types: `password` for secrets (masked in UI), `textfield`, `checkbox`, `dropdown`
- Per-command preferences vs. extension-level preferences

### Publishing to Raycast Store Checklist
- [ ] `package.json` has `name`, `title`, `description`, `icon`, `author`, `license`
- [ ] All preferences have `title`, `description`, `required` set correctly
- [ ] README includes screenshot GIFs of each command
- [ ] All API keys use `type: "password"` in preferences schema
- [ ] Error states handled with `showToast(Toast.Style.Failure, ...)`
- [ ] Loading states use `isLoading` prop or `Toast.Style.Animated`
- [ ] Fork `raycast/extensions` repo and open PR from your fork

---

## Who Each Track Is For

**Framer + AI** is for marketers, indie hackers, and frontend developers who build and maintain sites in Framer. You ship landing pages, product sites, and blogs. You want Claude to handle the repetitive layout and component work so you can focus on copy and conversion.

**Design to Code** is for product designers who write code, or frontend engineers who work from Figma files. You receive a Figma spec and want a repeatable, systematic process for getting to production-quality, accessible React components without rewriting from scratch every time.

**Raycast Extensions** is for developers who live in the keyboard and are tired of context-switching to the browser to look up a HubSpot contact, trigger an n8n workflow, or check a deployment. If you have written TypeScript and worked with REST APIs before, you can ship a working extension in an afternoon.

---

## Common Use Cases

1. **HubSpot contact lookup from Raycast**: Search contacts by name from the menu bar, see AI lead score, last activity date, lifecycle stage, and open deal value without opening a browser tab.

2. **Framer blog with filtered CMS**: Build a blog index page in Framer with a CMS collection, filterable by tag using client-side filtering, with dynamic pages generated per post and Open Graph images auto-populated from the cover image field.

3. **Figma design to shadcn component**: Use Figma MCP to read a Card component spec, pipe it to Claude with a shadcn/ui context prompt, and get a production-ready `Card.tsx` with all variants, TypeScript props, and a Storybook story.

4. **n8n workflow runner from Raycast**: List all active n8n workflows in a Raycast List command, filter by name, and trigger any workflow with a Form command that accepts custom input fields mapped to the webhook body.

5. **Contact form to CRM via n8n**: A Framer site contact form using the `withN8nForm` code override POSTs to an n8n webhook that creates a HubSpot contact, sends a confirmation email via SMTP, and adds the contact to a HubSpot sequence.

6. **Design token pipeline from Figma to Tailwind**: Export Figma Variables as JSON, run them through a Claude-assisted transform script to generate CSS custom properties and a Tailwind config extension, then regenerate the config whenever the Figma file updates.

7. **Deployment status in the menu bar**: A MenuBar Raycast extension that polls the Vercel API every 60 seconds using `useCachedPromise`, shows a green or red icon based on the latest production deployment status, and opens the deployment URL on click.

8. **Semantic search over your Notion notes**: A Raycast List command that embeds the search query using the OpenAI Embeddings API and queries a Pinecone index populated from your Notion pages via an n8n sync workflow, showing results ranked by cosine similarity.

---

## Prerequisites

### Before Starting Framer + AI
- Framer account with an existing project (free plan works for development)
- Claude Code installed and running locally
- Framer MCP added to your Claude Code config: `claude mcp add framer`
- Basic familiarity with Framer's canvas and code override system
- Node.js 18+ for local development

### Before Starting Design to Code
- Figma account with at least one design file
- Figma MCP or Figma Tokens plugin installed
- Next.js 14+ project with App Router (the patterns here target App Router, not Pages Router)
- Tailwind CSS configured in the project
- Familiarity with TypeScript and React component patterns

### Before Starting Raycast Extensions
- Raycast installed (macOS only)
- Node.js 18+ and npm
- Raycast developer mode enabled: `Raycast Settings > Advanced > Developer Mode`
- A bootstrapped extension: `npx create-raycast-extension@latest`
- API keys for any external services you are connecting (HubSpot, n8n, Pinecone, etc.)

---

## Connecting Design to Automation

The design track is the front-end of the automation system.

**Framer forms to n8n**: The `withN8nForm` code override in `./framer-ai/` POSTs form data to any n8n webhook URL. Combine this with the n8n playbooks to route submissions to HubSpot, send transactional email, trigger Slack notifications, or start an AI enrichment workflow.

**Raycast extensions to HubSpot and n8n**: The extension specs in `./raycast-extensions/` use the same HubSpot API v3 and n8n REST API that appear throughout the automation track. The HubSpot Contact Lookup extension is a direct keyboard interface to the same CRM data your n8n workflows write to.

**Design-to-code accelerates shipping**: Faster component generation means faster iteration on the interfaces your automation workflows power. A lead qualification form, a client portal, or a reporting dashboard built from design tokens ships faster when the Figma-to-code gap is systematic rather than ad hoc.

**Shared token system**: The design token pipeline in `./design-to-code/` produces CSS custom properties and Tailwind config values that apply to any app in your stack, whether it is a Next.js app deployed on Vercel, a Framer site, or a Storybook component library.

**Framer to production code**: The `exportReactComponents` Framer MCP tool lets you export a Framer component as React code, which you can bring into a Next.js app and refine using the patterns in `./design-to-code/`. This bridges the gap between Framer prototyping and production codebases.
