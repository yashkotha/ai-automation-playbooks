# Framer + AI

Using Claude Code and AI to build and iterate Framer sites faster.

## Setup

Framer sites are React under the hood. You can extend them with:
- Code overrides for custom interactions
- Custom code components in the Framer editor
- The Framer MCP server connected to Claude Code for natural language edits

---

## Workflow 1: Claude Code + Framer MCP

The Framer MCP server gives Claude Code direct access to your project. You can ask Claude to:
- Add and update pages
- Bulk update CMS content across multiple items
- Create CMS collections from structured data
- Write custom code components and overrides

**Setup:**
1. Open Claude Code settings -> MCP servers
2. Add the Framer MCP server
3. Open your Framer project in the browser
4. Claude can now interact with your project via natural language

**What works well:**
- Adding new sections to existing pages
- Updating copy across multiple pages at once
- Creating CMS collections from a JSON or CSV dataset
- Writing code overrides for custom animations or interactions

---

## Workflow 2: AI-Generated Copy

Prompt template for Framer page copy:

```
Write homepage copy for a [type of business].

Audience: [who they are]
Main value prop: [what you do better than others]
Tone: [direct / warm / professional]

Sections needed:
- Hero headline + subheadline (headline under 10 words, make a specific claim)
- 3 feature/benefit blocks (headline + 2 sentences each)
- Social proof section (structure only, I will fill in real numbers)
- CTA section

Rules:
- No em dashes
- No marketing cliches or jargon
- Write like a good copywriter, not a content team
- The headline should say something specific, not "We help businesses grow"
```

---

## Workflow 3: Code Overrides

Framer code overrides add behavior to any element without leaving Framer.

**Fade-in on scroll:**

```typescript
import { motion, useInView } from "framer-motion"
import { useRef } from "react"

export function withFadeIn(Component) {
  return function (props) {
    const ref = useRef(null)
    const isInView = useInView(ref, { once: true, margin: "-50px" })
    return (
      <motion.div
        ref={ref}
        initial={{ opacity: 0, y: 24 }}
        animate={isInView ? { opacity: 1, y: 0 } : {}}
        transition={{ duration: 0.5, ease: "easeOut" }}
      >
        <Component {...props} />
      </motion.div>
    )
  }
}
```

**Form POST to n8n webhook:**

```typescript
export function withWebhookForm(Component) {
  return function (props) {
    async function handleSubmit(e) {
      e.preventDefault()
      const data = Object.fromEntries(new FormData(e.target))
      await fetch("https://n8n.yourdomain.com/webhook/form", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data),
      })
    }
    return <Component {...props} onSubmit={handleSubmit} />
  }
}
```

---

## Common Patterns

**Data-driven sections:** Pull live data from an API into Framer via a code component. Works for testimonials from a CMS, live pricing, or real-time stats.

**Form handling without third-party tools:** Use a code override to POST directly to an n8n webhook. No Typeform, no Formspree, no dependencies.

**Conditional content:** Show different content based on URL params, time of day, or localStorage. All doable with code overrides, no custom code component needed.
