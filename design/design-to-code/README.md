# Design to Code

Turning Figma designs into production-ready components using Claude Code and the Figma MCP server.

---

## Setup for Best Results

**In Figma:**
- Name all layers semantically (Button/Primary, Card/Default, not "Frame 47")
- Use components and variants, not one-off frames
- Use Figma variables for colors, spacing, and typography
- Add dev notes for interactions and non-obvious states

**In Claude Code:**
- Connect the Figma MCP server (Claude Code settings -> MCP)
- Work component by component, not full pages at once
- Specify your exact tech stack in every prompt

---

## Prompt Template

```
I have a Figma design for [component name].

Build this as a [React/Vue/etc.] component using [Tailwind/CSS modules/etc.].

Requirements:
- Match the design exactly
- Mobile-first and fully responsive
- Semantic HTML
- Hover and focus states included
- Props: [list what should be configurable]

Tech stack: [your full stack]

Design context: [paste from Figma MCP or describe the key visual details]
```

---

## Component-First Approach

Don't ask Claude to build an entire page. Build component by component:

1. Start with the simplest standalone pieces (buttons, inputs, badges, tags)
2. Move to composite components (cards, form groups, nav items, modals)
3. Build layout components last (page sections, grids, full page shells)

Each component is easier to review, test, and fix when it's isolated.

---

## Common Pitfalls

**AI over-engineers.** Ask for the simplest implementation that matches the design. If you don't specify, you'll get abstractions you don't need.

**Browser vs design gaps.** Always check spacing and typography in an actual browser. Figma spacing doesn't map 1:1 to CSS and line-height behaves differently.

**Responsive assumptions.** AI will guess at breakpoints. Specify them or review the responsive behavior carefully before shipping.

**Font loading.** Make sure the fonts from Figma are loaded in your project. AI won't add font imports unless you explicitly ask.

---

## Tech Stack Configs That Work Well

**Next.js + Tailwind + shadcn/ui**
Ask Claude to use existing shadcn components as the base and extend them. Avoids reinventing primitives and keeps your design system consistent.

**React + CSS Modules**
Better for design-heavy work where you need precise control. Ask Claude to use BEM-style class names for readability.

**Vue 3 + Tailwind**
Same pattern as React + Tailwind. Specify `<script setup>` and Composition API or Claude may default to Options API.
