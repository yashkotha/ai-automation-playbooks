# Framer + AI — Complete Playbook

Using Claude Code, Framer MCP, and AI to design, build, and ship Framer sites faster across every part of the workflow.

---

## Table of Contents

1. [Setup & Project Structure](#1-setup--project-structure)
2. [Design with AI](#2-design-with-ai)
3. [Components](#3-components)
4. [Animations & Interactions](#4-animations--interactions)
5. [CMS & Dynamic Content](#5-cms--dynamic-content)
6. [Templates](#6-templates)
7. [Plugins](#7-plugins)
8. [Publishing & SEO](#8-publishing--seo)
9. [AI Prompt Library](#9-ai-prompt-library)
10. [Common Patterns](#10-common-patterns)

---

## 1. Setup & Project Structure

### Connect Claude Code to Framer (MCP)

The Framer MCP server gives Claude Code live access to your project — pages, CMS, components, and code files — all through natural language.

**Install:**
1. Open Claude Code → Settings → MCP Servers
2. Add Framer MCP: follow the setup at [framer.com/developers/mcp](https://www.framer.com/developers/mcp)
3. Open your Framer project in the browser (must be open for MCP to connect)
4. Claude can now read and write your project directly

**What Claude can do through MCP:**
- Create, update, and delete pages
- Read and update CMS collections and items
- Write code components and code overrides
- Read the full project XML structure
- Manage color and text styles
- Zoom into any element

**Verify the connection:**
```
"What pages does my Framer project have?"
```
If Claude lists your pages correctly, the MCP is working.

---

### Project Structure

Framer doesn't enforce a file structure, but keeping it consistent matters when you're using AI to navigate your project.

**Recommended naming conventions:**
- Pages: `Home`, `About`, `Pricing`, `Blog`, `Blog Post` (for CMS templates)
- CMS collections: singular nouns — `Post`, `Case Study`, `Team Member`, `Product`
- Code files: kebab-case — `fade-in-override.tsx`, `testimonial-card.tsx`, `use-scroll-progress.ts`
- Color styles: `brand/primary`, `brand/secondary`, `neutral/100`–`neutral/900`, `status/success`, `status/error`
- Text styles: `heading/xl`, `heading/lg`, `body/md`, `body/sm`, `label/md`

**Tell Claude your conventions:**
```
"My Framer project uses these conventions: [paste your list].
Keep all code files in kebab-case, all CMS collections as singular nouns."
```

---

### Workspace Settings Worth Configuring

Before building, set these in Framer to avoid problems later:

- **Breakpoints:** Set mobile (390), tablet (768), and desktop (1280) breakpoints upfront
- **Canvas color:** Set to match your background — prevents white flash on dark sites
- **Fonts:** Add your fonts via Google Fonts or upload — do this before Claude writes any text styles
- **Custom domain:** Connect early, even if you're not ready to publish — DNS takes time

---

## 2. Design with AI

### Layout Generation

Ask Claude to generate entire page layouts using Framer's XML structure, or describe what you need and let it build it.

**Prompt: Generate a landing page layout**
```
Build a landing page layout in my Framer project for [product type].

Structure:
- Sticky nav with logo left, links center, CTA right
- Hero: full-width, left-aligned headline, subheadline, two CTAs, product screenshot right
- Social proof bar: 5 logos, muted background
- 3-column feature grid: icon, headline, 2-line description each
- Testimonial: single large quote, avatar, name, title
- Pricing: 3 tiers side by side, middle tier highlighted
- FAQ: accordion, 6 questions
- Footer: 4-column links + copyright

Design rules:
- Max content width: 1200px
- Horizontal padding: 24px mobile, 80px desktop
- Section vertical padding: 80px desktop, 48px mobile
- No unnecessary borders or shadows — use spacing to separate
```

---

### Typography System

Set up a type scale before writing any copy. Claude can apply it consistently across the whole project.

**Prompt: Create a type system**
```
Create a text style system in my Framer project using [font name].

Scale:
- Display: 64px / 1.1 line height / -0.02em letter spacing / Bold
- H1: 48px / 1.15 / -0.01em / Bold
- H2: 36px / 1.2 / -0.01em / Bold
- H3: 24px / 1.3 / 0 / Semibold
- Body LG: 18px / 1.6 / 0 / Regular
- Body MD: 16px / 1.6 / 0 / Regular
- Body SM: 14px / 1.5 / 0 / Regular
- Label: 12px / 1.4 / 0.04em / Medium / uppercase
- Caption: 12px / 1.4 / 0 / Regular

Apply these as Framer text styles named heading/display, heading/h1, etc.
```

---

### Color System

**Prompt: Build a color system**
```
Create a color style system in my Framer project.

Brand:
- Primary: #[hex] — use for CTAs, links, accents
- Primary Dark: #[hex] — hover states
- Secondary: #[hex]

Neutrals (for text and backgrounds):
- neutral/50: #fafafa
- neutral/100 through neutral/900 in increments
- neutral/950: #0a0a0a

Semantic:
- success: #22c55e
- warning: #f59e0b
- error: #ef4444

Apply as Framer color styles. Use the brand/primary color for all interactive elements.
```

---

### Spacing & Layout Tokens

Framer doesn't have native spacing tokens, but you can enforce them through prompts.

**Tell Claude your spacing scale:**
```
My spacing scale is 4px base (4, 8, 12, 16, 24, 32, 48, 64, 80, 96, 128).
When building any layout or component, only use values from this scale for
margins, padding, gaps, and border radii. Border radii: 4, 8, 12, 16, 24, full.
```

---

### Design Review with AI

**Prompt: Audit the design**
```
Review my Framer project's Home page.
Check for:
- Inconsistent font sizes or weights not in our type system
- Spacing values that don't follow the 4px grid
- Color values not in our color system
- Mobile breakpoints where text overflows or sections stack awkwardly
- Contrast issues (text on colored backgrounds)
List every issue with the element name and what to fix.
```

---

## 3. Components

### Custom Code Components

Code components are full React components that live in the Framer Assets panel. Use them for anything that needs logic, external data, or dynamic behavior.

**Prompt: Build a component**
```
Create a Framer code component called TestimonialCard.

Props (all configurable in Framer's property panel):
- quote: string (default: "Quote text here")
- author: string (default: "Name")
- title: string (default: "Title, Company")
- avatar: image (Framer image prop)
- rating: number 1–5 (show filled stars)
- accentColor: color (Framer color prop, default: #6366f1)

Design:
- White card, 24px padding, 12px border radius, subtle shadow
- Star rating row
- Quote in 16px italic
- Avatar 40px circle + name/title right of it
- Hover: lift 4px, shadow deepens, smooth 200ms transition

Export as a named export. Include proper TypeScript types. Add it to my Framer project.
```

---

### Component Variants

Framer's variant system lets you define interactive states (hover, pressed, etc.) visually. For code components, you handle this in React.

**Common variant patterns:**

```typescript
// Button with variants: default, hover, loading, disabled
import { addPropertyControls, ControlType } from "framer"
import { motion } from "framer-motion"
import { useState } from "react"

interface Props {
  label: string
  variant: "primary" | "secondary" | "ghost"
  size: "sm" | "md" | "lg"
  loading?: boolean
  disabled?: boolean
  onClick?: () => void
}

const sizes = {
  sm: { padding: "8px 16px", fontSize: 14 },
  md: { padding: "12px 24px", fontSize: 16 },
  lg: { padding: "16px 32px", fontSize: 18 },
}

export function Button({ label, variant, size, loading, disabled, onClick }: Props) {
  const s = sizes[size]
  const isPrimary = variant === "primary"

  return (
    <motion.button
      onClick={onClick}
      disabled={disabled || loading}
      whileHover={!disabled ? { scale: 1.02 } : {}}
      whileTap={!disabled ? { scale: 0.98 } : {}}
      style={{
        ...s,
        background: isPrimary ? "#6366f1" : "transparent",
        color: isPrimary ? "#fff" : "#6366f1",
        border: `2px solid ${isPrimary ? "transparent" : "#6366f1"}`,
        borderRadius: 8,
        cursor: disabled ? "not-allowed",
        opacity: disabled ? 0.5 : 1,
        fontWeight: 600,
      }}
    >
      {loading ? "Loading..." : label}
    </motion.button>
  )
}

addPropertyControls(Button, {
  label: { type: ControlType.String, defaultValue: "Click me" },
  variant: {
    type: ControlType.Enum,
    options: ["primary", "secondary", "ghost"],
    defaultValue: "primary",
  },
  size: {
    type: ControlType.Enum,
    options: ["sm", "md", "lg"],
    defaultValue: "md",
  },
  loading: { type: ControlType.Boolean, defaultValue: false },
  disabled: { type: ControlType.Boolean, defaultValue: false },
})
```

---

### Code Overrides

Overrides attach behavior to any Framer canvas element without making it a code component.

**Stagger children on load:**
```typescript
import { ComponentType, useEffect, useState } from "react"
import { motion } from "framer-motion"

export function withStaggerChildren(Component: ComponentType): ComponentType {
  return function (props: any) {
    return (
      <motion.div
        initial="hidden"
        animate="visible"
        variants={{
          hidden: {},
          visible: { transition: { staggerChildren: 0.08 } },
        }}
        style={{ display: "contents" }}
      >
        <Component {...props} />
      </motion.div>
    )
  }
}
```

**Parallax on scroll:**
```typescript
import { ComponentType } from "react"
import { motion, useScroll, useTransform } from "framer-motion"
import { useRef } from "react"

export function withParallax(Component: ComponentType): ComponentType {
  return function (props: any) {
    const ref = useRef(null)
    const { scrollYProgress } = useScroll({ target: ref, offset: ["start end", "end start"] })
    const y = useTransform(scrollYProgress, [0, 1], ["-15%", "15%"])

    return (
      <motion.div ref={ref} style={{ y, display: "contents" }}>
        <Component {...props} />
      </motion.div>
    )
  }
}
```

**Active nav link:**
```typescript
import { ComponentType } from "react"
import { useEffect, useState } from "react"

export function withActiveLink(Component: ComponentType): ComponentType {
  return function (props: any) {
    const [active, setActive] = useState(false)

    useEffect(() => {
      setActive(window.location.pathname === props.href)
    }, [props.href])

    return (
      <Component
        {...props}
        style={{
          ...props.style,
          fontWeight: active ? 700 : 400,
          color: active ? "#6366f1" : props.style?.color,
        }}
      />
    )
  }
}
```

---

### Building a Component Library

For larger sites, build reusable components once and reference them everywhere.

**Prompt: Audit and organize components**
```
Look at my Framer project. Identify:
1. Any design patterns repeated 3+ times that should be a reusable component
2. Any code components missing addPropertyControls (so they're not editable in the panel)
3. Components not following our naming convention

List each one with a recommendation.
```

---

## 4. Animations & Interactions

### Page Transitions

Add smooth transitions between pages using Framer's built-in transition system.

**Prompt:**
```
Add a fade + slight upward slide page transition to my Framer project.
Every page should fade in over 300ms with a 16px upward translate when navigating.
The exit transition should be faster: 200ms fade out.
Use Framer's page transition settings, not a code override.
```

---

### Scroll-triggered Animations

The most impactful animations for landing pages. All built with Framer Motion.

**Fade in + slide up (the standard):**
```typescript
import { motion, useInView } from "framer-motion"
import { useRef, ComponentType } from "react"

export function withScrollReveal(Component: ComponentType): ComponentType {
  return function (props: any) {
    const ref = useRef(null)
    const inView = useInView(ref, { once: true, margin: "-80px" })

    return (
      <motion.div
        ref={ref}
        initial={{ opacity: 0, y: 32 }}
        animate={inView ? { opacity: 1, y: 0 } : {}}
        transition={{ duration: 0.6, ease: [0.16, 1, 0.3, 1] }}
      >
        <Component {...props} />
      </motion.div>
    )
  }
}
```

**Counter animation (for stats sections):**
```typescript
import { ComponentType, useEffect, useState } from "react"
import { useInView } from "framer-motion"
import { useRef } from "react"

export function withCountUp(Component: ComponentType): ComponentType {
  return function (props: any) {
    const ref = useRef(null)
    const inView = useInView(ref, { once: true })
    const target = parseInt(props.children, 10) || 0
    const [count, setCount] = useState(0)

    useEffect(() => {
      if (!inView) return
      const duration = 1500
      const start = performance.now()

      const tick = (now: number) => {
        const progress = Math.min((now - start) / duration, 1)
        const ease = 1 - Math.pow(1 - progress, 3) // ease out cubic
        setCount(Math.floor(ease * target))
        if (progress < 1) requestAnimationFrame(tick)
      }

      requestAnimationFrame(tick)
    }, [inView, target])

    return <Component {...props} ref={ref}>{count}</Component>
  }
}
```

**Scroll progress bar (for articles/docs):**
```typescript
import { motion, useScroll, useSpring } from "framer-motion"

export function ScrollProgressBar() {
  const { scrollYProgress } = useScroll()
  const scaleX = useSpring(scrollYProgress, { stiffness: 100, damping: 30 })

  return (
    <motion.div
      style={{
        position: "fixed",
        top: 0,
        left: 0,
        right: 0,
        height: 3,
        background: "#6366f1",
        transformOrigin: "left",
        scaleX,
        zIndex: 9999,
      }}
    />
  )
}
```

---

### Hover Interactions

**Magnetic button:**
```typescript
import { ComponentType, useRef } from "react"
import { motion, useMotionValue, useTransform, useSpring } from "framer-motion"

export function withMagnetic(Component: ComponentType): ComponentType {
  return function (props: any) {
    const ref = useRef<HTMLDivElement>(null)
    const x = useMotionValue(0)
    const y = useMotionValue(0)
    const springX = useSpring(x, { stiffness: 200, damping: 20 })
    const springY = useSpring(y, { stiffness: 200, damping: 20 })

    const handleMouseMove = (e: React.MouseEvent) => {
      const rect = ref.current!.getBoundingClientRect()
      const cx = rect.left + rect.width / 2
      const cy = rect.top + rect.height / 2
      x.set((e.clientX - cx) * 0.3)
      y.set((e.clientY - cy) * 0.3)
    }

    return (
      <motion.div
        ref={ref}
        onMouseMove={handleMouseMove}
        onMouseLeave={() => { x.set(0); y.set(0) }}
        style={{ x: springX, y: springY, display: "contents" }}
      >
        <Component {...props} />
      </motion.div>
    )
  }
}
```

---

### Prompt Templates for Animations

**Request a specific animation:**
```
Add a scroll-triggered animation to the feature grid on my Home page.
Each card should:
- Start invisible (opacity 0) and 24px below final position
- Animate in when 80px into the viewport
- Stagger with 80ms delay between cards (left to right)
- Use an ease-out-quart curve (cubic-bezier 0.16, 1, 0.3, 1)
- Duration: 0.6s
- Only animate once (not on scroll back up)

Write this as a code override called withGridReveal.
```

---

## 5. CMS & Dynamic Content

### Setting Up a CMS Collection

**Prompt: Create a blog CMS collection**
```
Create a CMS collection in my Framer project called "Post" with these fields:

- title: text (required)
- slug: text (required, used in URL)
- excerpt: text (required, max 160 chars — used for SEO description)
- body: rich text (required)
- cover: image (required)
- author: text (required)
- author_avatar: image
- published_at: date
- category: text (options: Tutorial, Case Study, Update, Guide)
- featured: boolean (default false)
- reading_time: number (minutes)

After creating the collection, create a dynamic page template at /blog/[slug]
and a blog index page at /blog that lists all posts sorted by published_at descending.
```

---

### Bulk CMS Operations

Claude can create, update, or delete CMS items in bulk through the MCP — much faster than doing it manually.

**Prompt: Populate CMS from data**
```
I have this JSON data. Create CMS items in my "Team Member" collection for each entry.

[paste JSON]

Map the fields as:
- "name" → name
- "role" → title
- "bio" → bio
- "twitter" → social_url
```

**Prompt: Update all items**
```
In my "Product" CMS collection, find all items where the "status" field is "draft"
and change it to "active". List which items you updated.
```

---

### Dynamic Pages with Filters

**Prompt: Build a filterable blog index**
```
On my /blog page, add a category filter row above the post grid.

Behavior:
- Show buttons for "All" + each unique category value from the Post collection
- Default selection: "All" (shows all posts)
- Clicking a category filters the grid to only show matching posts
- Active category button gets a filled style, others are outlined
- Filter is client-side only (no page reload)
- On mobile, make the filter row horizontally scrollable

Write this as a code component called BlogFilter that accepts the posts array as a prop.
```

---

### Connecting External Data Sources

Pull live data from any API into a Framer code component:

```typescript
import { useEffect, useState } from "react"
import { addPropertyControls, ControlType } from "framer"

interface GitHubStats {
  stars: number
  forks: number
  watchers: number
}

export function GitHubStatsCard({ repo, accentColor }: { repo: string; accentColor: string }) {
  const [stats, setStats] = useState<GitHubStats | null>(null)
  const [error, setError] = useState(false)

  useEffect(() => {
    fetch(`https://api.github.com/repos/${repo}`)
      .then((r) => r.json())
      .then((d) => setStats({ stars: d.stargazers_count, forks: d.forks_count, watchers: d.subscribers_count }))
      .catch(() => setError(true))
  }, [repo])

  if (error) return <div>Failed to load stats</div>
  if (!stats) return <div>Loading...</div>

  return (
    <div style={{ display: "flex", gap: 24 }}>
      {[
        { label: "Stars", value: stats.stars },
        { label: "Forks", value: stats.forks },
        { label: "Watchers", value: stats.watchers },
      ].map(({ label, value }) => (
        <div key={label} style={{ textAlign: "center" }}>
          <div style={{ fontSize: 28, fontWeight: 700, color: accentColor }}>
            {value.toLocaleString()}
          </div>
          <div style={{ fontSize: 13, color: "#888" }}>{label}</div>
        </div>
      ))}
    </div>
  )
}

addPropertyControls(GitHubStatsCard, {
  repo: { type: ControlType.String, defaultValue: "owner/repo" },
  accentColor: { type: ControlType.Color, defaultValue: "#6366f1" },
})
```

---

### CMS + n8n Integration

Auto-populate your Framer CMS from external sources using n8n:

```
n8n workflow pattern:
1. Trigger: Schedule (every 24h) or webhook
2. Fetch data from source (Airtable, Notion, RSS, API)
3. Transform into Framer CMS field format
4. POST to Framer CMS API: https://api.framer.com/store/...
5. Send Slack notification with count of items synced
```

**Prompt to build the n8n workflow:**
```
Build an n8n workflow that:
1. Runs every day at 8am
2. Reads all rows from my Airtable base "[base name]", table "[table name]"
3. For each row, creates or updates a Framer CMS item in collection "Post"
   using the Framer API (I'll provide the API key and collection ID)
4. Matches on "slug" field to decide create vs update
5. Logs the result to a Google Sheet (columns: date, slug, action, status)
```

---

## 6. Templates

### Using Framer Templates

Templates are the fastest way to start. Once imported, Claude can fully customize them.

**Best template categories for AI customization:**
- **SaaS/startup landing pages** — clean structure, easy to repopulate
- **Portfolio templates** — good component variety to learn from
- **Blog/editorial** — CMS already set up
- **Product marketing** — showcase sections ready to adapt

**After importing a template:**
```
I just imported the "[template name]" Framer template.
Analyze the structure:
1. List all pages and their purpose
2. List all CMS collections and their fields
3. List any code components or overrides already in use
4. Identify what needs to be changed to fit [my product/brand]
```

---

### Customizing Templates with AI

**Full rebrand prompt:**
```
Rebrand this Framer template for [my company name], a [description].

Changes needed:
- Replace all colors with our color system: primary #[hex], neutral scale #[hex range]
- Replace all fonts with [font name] (already added to the project)
- Update the logo placeholder with the text "[Company]" in our brand font
- Replace all placeholder copy using this brief: [paste brief]
- Update the nav links to: [list pages]
- Remove the [section name] section — we don't need it
- Add a [new section] section after the features section

Keep all animations and layout structure intact.
```

---

### Building Template Systems

If you're building multiple sites (agencies, freelancers), create a master template:

**Prompt: Create a starter template**
```
Create a Framer project structure I can reuse as a starter template for client sites.

Include:
- Pages: Home, About, Services, Contact, Privacy, Terms
- Color styles: brand/primary, brand/secondary, neutral/50–950, semantic (success/warning/error)
- Text styles: full scale from display down to caption
- Components: Button (3 variants), Card, Badge, Avatar, Testimonial, FAQ Item
- CMS collection: Post (title, slug, excerpt, body, cover, date, category)
- A /blog index page and /blog/[slug] dynamic page

Use placeholder colors (#6366f1 primary) and Inter font — I'll swap them per client.
Keep everything well-named and organized.
```

---

## 7. Plugins

### Essential Framer Plugins

| Plugin | Use Case | Best For |
|--------|----------|----------|
| **Unsplash** | Free stock photos directly in canvas | Placeholder images |
| **Noun Project** | Icons | Icon-heavy UIs |
| **Lottie** | Animated illustrations | Hero sections, empty states |
| **Radix Icons** | Developer-style icon set | SaaS/product sites |
| **Content Buddy** | Bulk text replacement | Template customization |
| **Breakpoint Previewer** | Preview all breakpoints at once | Responsive QA |
| **Palette** | Generate color palettes from an image | Brand matching |
| **Stark** | Accessibility contrast checker | Accessibility audits |
| **Font Preview** | Preview fonts before committing | Typography decisions |
| **Hand Cursor** | Show hand cursor on hover | Interaction polish |

---

### Using Plugins with AI

Claude can't directly control Framer's plugin UI, but it can help you prepare data for plugins and post-process plugin output.

**Palette plugin + Claude:**
```
I used the Palette plugin and got these colors from my brand image:
#1a1a2e, #16213e, #0f3460, #e94560, #f5f5f5

Create a full color system from these:
- Identify primary, secondary, and accent roles
- Generate tinted/shaded versions for a full neutral scale
- Create semantic colors (success, warning, error) that harmonize
- Apply as Framer color styles
```

**Lottie + Claude:**
```
I want to add a Lottie animation to the hero section of my Home page.
Suggest 5 specific Lottie animation styles (with search terms to find them
on LottieFiles) that would work for a [product type] landing page.
For each, describe where on the canvas to place it and how to scale it.
```

---

### Building Custom Plugins

Framer plugins are React apps. Claude can build them.

**Prompt: Build a simple content audit plugin**
```
Build a Framer plugin that:
1. Scans all text elements on the current page
2. Flags any text that:
   - Uses a font size not in our scale (14, 16, 18, 24, 36, 48, 64)
   - Uses a color not in our defined palette (list of hex values)
   - Has a line height below 1.4
3. Shows results in a panel list with element name + issue
4. Clicking an item zooms Framer into that element

Use the Framer Plugin API. Package as a local plugin I can load in development.
```

---

## 8. Publishing & SEO

### Pre-publish Checklist Prompt

```
Before I publish my Framer site, check the project for:

SEO:
- Missing page titles (should be unique per page, under 60 chars)
- Missing meta descriptions (under 160 chars)
- Missing OG images (social share previews)
- Pages with no h1 tag
- Images missing alt text

Performance:
- Images that aren't using Framer's optimized image component
- Any external scripts loaded synchronously
- Fonts loading from too many weights

Accessibility:
- Interactive elements with no accessible label
- Color contrast issues (text on colored backgrounds)
- Missing focus states on buttons/links

List every issue with the page name and element.
```

---

### SEO Configuration

**Prompt: Set up SEO for all pages**
```
Set up SEO metadata for my Framer project.

For each page, write:
- Title tag (format: "[Page] — [Brand]", under 60 chars)
- Meta description (under 160 chars, includes primary keyword)
- OG title (can be more descriptive than title tag)
- OG description

Pages:
- Home: [brief about the site]
- About: [brief about the company]
- Pricing: [brief about pricing]
- Blog: [brief about content focus]

Apply these to the Framer SEO settings for each page.
```

---

### Custom Domain Setup

1. In Framer: **Site Settings → Custom Domain → Add Domain**
2. Copy the DNS records Framer provides
3. Add them in your DNS provider (Hostinger, Cloudflare, etc.)
4. DNS propagates in minutes (Cloudflare) to 24h (others)
5. Framer auto-provisions SSL once DNS resolves

**For Hostinger DNS with Claude:**
```
Add these DNS records to my domain [domain.com] on Hostinger:
[paste Framer DNS records]

Use the Hostinger MCP to add them. Confirm each record after adding.
```

---

### Staging vs Production

Framer doesn't have a native staging environment, but you can simulate one:

- **Free subdomain as staging:** Publish to `yoursite.framer.website` first
- **Branch projects:** Duplicate the project for testing, publish separately
- **Preview links:** Use Framer's shareable preview link for client review before going live

**Prompt: Set up a review workflow**
```
I want to set up a client review process before publishing changes to production.
My staging URL is [framer.website URL].
My production URL is [custom domain].

Write me a short checklist I can follow each time I make significant changes,
covering what to test in staging before pushing to production.
```

---

### Analytics Integration

**Add Plausible (privacy-first):**
```
Add Plausible Analytics to my Framer project with site ID "[yourdomain.com]".
Add it as a custom script in Site Settings → Custom Code → Head.
Also add event tracking for:
- CTA button clicks (data-analytics="cta-click")
- Form submissions (data-analytics="form-submit")
- Pricing page views
```

**Add PostHog:**
```
Add PostHog to my Framer site for session recording and heatmaps.
API key: [your key]
Add the snippet to the <head> via Framer's custom code settings.
Also set up a custom event for when users click the main CTA button.
```

---

## 9. AI Prompt Library

A quick-reference bank of prompts for common Framer tasks.

### Design
| Task | Prompt Starter |
|------|---------------|
| Generate a section | `"Add a [section type] section to my [page] page. [describe content and design requirements]"` |
| Audit spacing | `"Check every section on my Home page for spacing inconsistencies vs our 4px grid."` |
| Responsive fix | `"Fix the mobile layout of the [section] section — describe what's breaking and how to fix it."` |
| Dark mode toggle | `"Add a light/dark mode toggle to my site nav that persists in localStorage."` |

### Components
| Task | Prompt Starter |
|------|---------------|
| New component | `"Build a [component name] code component with these props: [list]. Design: [describe]."` |
| Add property controls | `"Add addPropertyControls to my [component name] component so it's editable in the Framer panel."` |
| Convert override to component | `"Convert my withFadeIn override into a full code component called FadeInWrapper."` |

### CMS
| Task | Prompt Starter |
|------|---------------|
| New collection | `"Create a CMS collection called [name] with these fields: [list with types]."` |
| Bulk add items | `"Add these [N] items to my [collection] CMS collection: [paste data]."` |
| Dynamic page | `"Create a dynamic page template for my [collection] collection at /[path]/[slug]."` |
| Filter UI | `"Add a [field] filter to my [page] page that filters the [collection] grid client-side."` |

### Animations
| Task | Prompt Starter |
|------|---------------|
| Scroll reveal | `"Add a scroll-triggered fade-up animation to [element/section] with [N]ms stagger."` |
| Hover effect | `"Add a [describe effect] hover animation to [element] using a code override."` |
| Page transition | `"Add a [describe] transition between pages in my Framer project."` |

### Publishing
| Task | Prompt Starter |
|------|---------------|
| SEO audit | `"Audit my Framer project for SEO issues on all pages."` |
| Write meta tags | `"Write title tags and meta descriptions for all pages. Context: [brief]."` |
| Add analytics | `"Add [Plausible/PostHog/GA4] to my Framer site. My [key/ID] is [value]."` |

---

## 10. Common Patterns

### Form → n8n → CRM

Route any Framer form to n8n, then into HubSpot/Airtable/Notion without third-party form tools.

```typescript
export function withN8nForm(Component: ComponentType): ComponentType {
  return function (props: any) {
    const [status, setStatus] = useState<"idle" | "loading" | "success" | "error">("idle")

    async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
      e.preventDefault()
      setStatus("loading")

      const data = Object.fromEntries(new FormData(e.currentTarget))

      try {
        await fetch("https://n8n.yourdomain.com/webhook/contact-form", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ ...data, source: window.location.href }),
        })
        setStatus("success")
      } catch {
        setStatus("error")
      }
    }

    return (
      <Component
        {...props}
        onSubmit={handleSubmit}
        data-status={status}
      />
    )
  }
}
```

---

### Conditional Content by URL Param

Show different headlines, CTAs, or sections based on UTM source or any query parameter:

```typescript
import { ComponentType, useEffect, useState } from "react"

export function withConditionalCopy(Component: ComponentType): ComponentType {
  return function (props: any) {
    const [variant, setVariant] = useState("default")

    useEffect(() => {
      const params = new URLSearchParams(window.location.search)
      const utm = params.get("utm_source") || "default"
      setVariant(utm)
    }, [])

    const copy = {
      default: props.defaultCopy || props.children,
      google: props.googleCopy || props.children,
      twitter: props.twitterCopy || props.children,
    }

    return <Component {...props}>{copy[variant] || copy.default}</Component>
  }
}
```

---

### Sticky Header with Scroll State

```typescript
import { ComponentType, useEffect, useState } from "react"
import { motion } from "framer-motion"

export function withStickyHeader(Component: ComponentType): ComponentType {
  return function (props: any) {
    const [scrolled, setScrolled] = useState(false)

    useEffect(() => {
      const handler = () => setScrolled(window.scrollY > 20)
      window.addEventListener("scroll", handler, { passive: true })
      return () => window.removeEventListener("scroll", handler)
    }, [])

    return (
      <motion.div
        animate={{
          backdropFilter: scrolled ? "blur(12px)" : "blur(0px)",
          background: scrolled ? "rgba(255,255,255,0.85)" : "rgba(255,255,255,0)",
          borderBottom: scrolled ? "1px solid rgba(0,0,0,0.08)" : "1px solid transparent",
        }}
        transition={{ duration: 0.2 }}
        style={{ position: "fixed", top: 0, left: 0, right: 0, zIndex: 100 }}
      >
        <Component {...props} />
      </motion.div>
    )
  }
}
```

---

### Cookie Consent Banner

```typescript
import { useState, useEffect } from "react"

export function CookieBanner({ onAccept, onDecline, accentColor = "#6366f1" }) {
  const [visible, setVisible] = useState(false)

  useEffect(() => {
    if (!localStorage.getItem("cookie-consent")) setVisible(true)
  }, [])

  const accept = () => {
    localStorage.setItem("cookie-consent", "accepted")
    setVisible(false)
    onAccept?.()
  }

  const decline = () => {
    localStorage.setItem("cookie-consent", "declined")
    setVisible(false)
    onDecline?.()
  }

  if (!visible) return null

  return (
    <div
      style={{
        position: "fixed", bottom: 24, left: 24, right: 24, maxWidth: 480,
        background: "#fff", borderRadius: 12, padding: "20px 24px",
        boxShadow: "0 8px 32px rgba(0,0,0,0.12)", zIndex: 9999,
        display: "flex", flexDirection: "column", gap: 16,
      }}
    >
      <p style={{ margin: 0, fontSize: 14, color: "#333", lineHeight: 1.5 }}>
        We use cookies to improve your experience. By continuing, you agree to our use of cookies.
      </p>
      <div style={{ display: "flex", gap: 8 }}>
        <button
          onClick={accept}
          style={{
            background: accentColor, color: "#fff", border: "none",
            borderRadius: 8, padding: "10px 20px", fontSize: 14,
            fontWeight: 600, cursor: "pointer",
          }}
        >
          Accept
        </button>
        <button
          onClick={decline}
          style={{
            background: "transparent", color: "#666",
            border: "1px solid #ddd", borderRadius: 8,
            padding: "10px 20px", fontSize: 14, cursor: "pointer",
          }}
        >
          Decline
        </button>
      </div>
    </div>
  )
}
```

---

### Prompt: Ask Claude to Do Anything

When in doubt, describe what you want in plain language. Claude understands Framer's structure, API, and component system well enough to handle most requests:

```
"My [page name] page needs a [describe what you want].
Here's how it should look: [describe].
Here's how it should behave: [describe].
Write the code and add it to the project."
```

The more specific you are about behavior, the better the output. Include edge cases, mobile behavior, and any existing style/component conventions.
