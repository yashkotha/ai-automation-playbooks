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

## Figma MCP Workflow

The Figma MCP server gives Claude Code direct access to your Figma files. This eliminates the copy-paste cycle entirely when it's wired up correctly.

### Getting Design Context

Use the `get_design_context` tool to pull structured node data from Figma:

```
Use the Figma MCP to read the component at [Figma URL or node ID].
Give me the layout structure, spacing values, color values, and typography.
Then implement it as a React component using Tailwind.
```

The MCP returns: node hierarchy, auto-layout settings, padding/gap values, fill colors (as hex or variable references), typography (family, size, weight, line-height), border radius, shadow values, and opacity.

### Reading Variables

Figma variables (the newer system, not styles) map directly to design tokens. When a file uses variables, the MCP exposes them through `get_variable_defs`:

```
Read the variable definitions from this Figma file.
Map them to a Tailwind theme extension and a CSS custom properties file.
```

The output will include variable collections (typically: Primitives, Semantic Tokens, Component Tokens) and their values across modes (Light, Dark).

### Implementing Components from MCP Data

Workflow for a single component:

1. Point Claude at the component node: `get_design_context` with the component's Figma URL
2. Ask Claude to identify which shadcn/ui primitives it can compose from before building from scratch
3. Ask for the component + Storybook story in one pass
4. Review in browser against a Figma screenshot (use `get_screenshot` from the MCP)

```
Here is the Figma design context for the PricingCard component: [MCP output]

1. Identify which shadcn/ui primitives (Card, Badge, Button) map to parts of this design
2. Build the component composing from those primitives
3. Add the variants: featured (highlighted border), standard, disabled
4. Output the component and a Storybook story
Tech stack: Next.js App Router, Tailwind, shadcn/ui
```

### Using get_screenshot for Visual Diffing

After building a component, use the MCP screenshot tool to get the Figma render, then compare:

```
Get a screenshot of [node URL] from Figma.
Here is my current implementation: [paste code or share browser screenshot].
What are the visual gaps? List specific CSS fixes.
```

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

## Component Library Approach

### shadcn/ui

shadcn/ui is the right default for most Next.js projects. It gives you Radix UI primitives with Tailwind styling that you own — no external dependency at runtime.

When building from Figma, tell Claude:
```
Use shadcn/ui components as the base where possible. Extend their variants
using the `cva` pattern already established in the project.
Do not create wrapper components that add no logic — just extend the existing ones.
```

Install components you'll need up front:
```bash
npx shadcn@latest add button card dialog select tabs badge avatar
```

Component-to-shadcn mapping cheat sheet:
- Figma Button variants → shadcn Button with `variant` prop
- Figma Dialog/Modal → shadcn Dialog + DialogContent
- Figma Dropdown → shadcn DropdownMenu or Select
- Figma Tooltip → shadcn Tooltip
- Figma Toast/Snackbar → shadcn Sonner (or useToast)
- Figma Table → shadcn Table
- Figma Tabs → shadcn Tabs

### Radix UI Primitives (headless)

When you need full visual control that departs significantly from shadcn defaults, use Radix primitives directly:

```bash
npm install @radix-ui/react-dialog @radix-ui/react-popover @radix-ui/react-tooltip
```

```typescript
// Full control over styling while keeping accessibility behavior
import * as Dialog from '@radix-ui/react-dialog'

export function CustomModal({ children, trigger }: Props) {
  return (
    <Dialog.Root>
      <Dialog.Trigger asChild>{trigger}</Dialog.Trigger>
      <Dialog.Portal>
        <Dialog.Overlay className="fixed inset-0 bg-black/50 backdrop-blur-sm" />
        <Dialog.Content className="fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 ...">
          {children}
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  )
}
```

### Headless UI (Tailwind Labs)

Better choice when you're already on Tailwind and want simpler API than Radix:

```bash
npm install @headlessui/react
```

Headless UI works well for: Combobox, Listbox, Menu, Transition, Disclosure. Less comprehensive than Radix but the Transition component is excellent for animations.

---

## Accessibility Implementation from Design

Figma designs rarely include accessibility annotations. You have to add them during implementation.

### What to add that's not in the design

**Interactive elements:**
```typescript
// Every icon button needs an accessible name
<button aria-label="Close dialog" onClick={onClose}>
  <XIcon className="h-4 w-4" />
</button>

// Every image needs alt text (or aria-hidden if decorative)
<Image src={avatar} alt={`${user.name}'s avatar`} />
<Image src={decoration} alt="" aria-hidden="true" />
```

**Form fields:**
```typescript
// Labels must be associated — don't use placeholder as the only label
<div>
  <Label htmlFor="email">Email address</Label>
  <Input
    id="email"
    type="email"
    aria-describedby="email-hint email-error"
    aria-invalid={!!errors.email}
  />
  <p id="email-hint" className="text-sm text-muted-foreground">
    We'll never share your email.
  </p>
  {errors.email && (
    <p id="email-error" role="alert" className="text-sm text-destructive">
      {errors.email.message}
    </p>
  )}
</div>
```

**Dynamic content:**
```typescript
// Announce loading states to screen readers
<div aria-live="polite" aria-busy={isLoading}>
  {isLoading ? <Spinner /> : <DataTable data={data} />}
</div>
```

**Focus management for modals:**
```typescript
// Radix Dialog handles this automatically
// For custom implementations, use focus-trap-react
import FocusTrap from 'focus-trap-react'

<FocusTrap active={isOpen}>
  <div role="dialog" aria-modal="true" aria-labelledby="dialog-title">
    ...
  </div>
</FocusTrap>
```

### Color contrast

Figma's contrast checker plugin (or the built-in accessibility panel) will flag issues. In code, the standard is:
- Normal text: 4.5:1 minimum (WCAG AA)
- Large text (18px+ or 14px+ bold): 3:1 minimum
- UI components and icons: 3:1 minimum

Add this to your prompt when implementing:
```
Ensure all text/background combinations meet WCAG AA contrast.
Flag any Figma colors that don't meet the threshold and suggest alternatives.
```

---

## Dark Mode Implementation

### Approach: CSS variables + Tailwind

The most maintainable approach maps Figma semantic color variables to CSS custom properties, then references those in Tailwind.

**Step 1 — CSS variables in globals.css:**
```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --card: 0 0% 100%;
  --card-foreground: 222.2 84% 4.9%;
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
  --muted: 210 40% 96.1%;
  --muted-foreground: 215.4 16.3% 46.9%;
  --border: 214.3 31.8% 91.4%;
  --radius: 0.5rem;
}

.dark {
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;
  --card: 222.2 84% 4.9%;
  --card-foreground: 210 40% 98%;
  --primary: 210 40% 98%;
  --primary-foreground: 222.2 47.4% 11.2%;
  --muted: 217.2 32.6% 17.5%;
  --muted-foreground: 215 20.2% 65.1%;
  --border: 217.2 32.6% 17.5%;
}
```

**Step 2 — Tailwind config:**
```typescript
// tailwind.config.ts
export default {
  darkMode: ['class'],
  theme: {
    extend: {
      colors: {
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        border: 'hsl(var(--border))',
      },
    },
  },
}
```

**Step 3 — Dark mode toggle:**
```typescript
// Next.js App Router: use next-themes
// app/layout.tsx
import { ThemeProvider } from 'next-themes'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
          {children}
        </ThemeProvider>
      </body>
    </html>
  )
}
```

**Using semantic colors in components:**
```typescript
// Always use semantic tokens, never raw colors
// Good:
<div className="bg-background text-foreground border border-border">
// Bad:
<div className="bg-white text-gray-900 border border-gray-200">
```

### Handling images in dark mode

```typescript
// Use CSS filter for icons that need to invert
<img
  src="/logo.svg"
  alt="Logo"
  className="dark:invert"
/>

// Or provide separate images per theme
<Image
  src={resolvedTheme === 'dark' ? '/logo-dark.svg' : '/logo-light.svg'}
  alt="Logo"
  width={120}
  height={40}
/>
```

---

## Responsive Implementation Patterns

### Figma → CSS breakpoint mapping

Figma frames are typically designed at fixed widths. Map them to Tailwind breakpoints:

| Figma Frame | Tailwind Breakpoint |
|-------------|---------------------|
| Mobile (375-390px) | default (no prefix) |
| Tablet (768px) | `md:` |
| Desktop (1280px) | `xl:` |
| Wide (1440px+) | `2xl:` |

### Complex layout: dashboard with sidebar

```typescript
// Responsive dashboard shell
export function DashboardLayout({ children }: { children: React.ReactNode }) {
  const [sidebarOpen, setSidebarOpen] = useState(false)

  return (
    <div className="min-h-screen bg-background">
      {/* Mobile overlay */}
      {sidebarOpen && (
        <div
          className="fixed inset-0 z-20 bg-black/50 lg:hidden"
          onClick={() => setSidebarOpen(false)}
        />
      )}

      {/* Sidebar */}
      <aside
        className={cn(
          'fixed left-0 top-0 z-30 h-full w-64 transform bg-card transition-transform duration-200 ease-in-out',
          'lg:translate-x-0 lg:static lg:z-auto',
          sidebarOpen ? 'translate-x-0' : '-translate-x-full'
        )}
      >
        <Sidebar />
      </aside>

      {/* Main content */}
      <div className="lg:pl-64">
        <header className="sticky top-0 z-10 border-b bg-background/95 backdrop-blur">
          <button className="lg:hidden" onClick={() => setSidebarOpen(true)}>
            <MenuIcon className="h-5 w-5" />
          </button>
        </header>
        <main className="p-4 md:p-6 lg:p-8">{children}</main>
      </div>
    </div>
  )
}
```

### Complex layout: masonry/Pinterest grid

```typescript
// CSS columns approach (simpler, less JS)
export function MasonryGrid({ items }: { items: Item[] }) {
  return (
    <div className="columns-1 gap-4 sm:columns-2 lg:columns-3 xl:columns-4">
      {items.map((item) => (
        <div key={item.id} className="mb-4 break-inside-avoid">
          <ItemCard item={item} />
        </div>
      ))}
    </div>
  )
}
```

### Complex layout: sticky headers with multiple scroll contexts

```typescript
// When a design has nested sticky elements
<div className="flex h-screen overflow-hidden">
  {/* Left panel: scrolls independently */}
  <div className="w-80 flex-shrink-0 overflow-y-auto border-r">
    <div className="sticky top-0 bg-background p-4 border-b">
      <h2>Panel Header</h2>
    </div>
    <div className="p-4">
      {leftContent}
    </div>
  </div>

  {/* Right panel: its own scroll context */}
  <div className="flex-1 overflow-y-auto">
    <div className="sticky top-0 bg-background/95 backdrop-blur z-10 p-4 border-b">
      <h1>Main Header</h1>
    </div>
    <div className="p-6">{mainContent}</div>
  </div>
</div>
```

---

## Animation Implementation from Design Specs

Figma supports prototype animations (Smart Animate, Dissolve, Move In/Out). Translate these to CSS or Framer Motion.

### Mapping Figma animations to code

| Figma Prototype | CSS / Framer Motion equivalent |
|----------------|-------------------------------|
| Dissolve | `opacity` transition |
| Smart Animate (position) | `transform: translate` or layout animation |
| Smart Animate (size) | `transform: scale` or explicit `width`/`height` |
| Move In from right | `translateX(100%) → translateX(0)` |
| Slide Up | `translateY(20px) → translateY(0)` |
| Spring easing | `type: "spring"` in Framer Motion |

### CSS transitions (simple interactions)

```typescript
// Hover state from Figma — card lifts on hover
<div className="transition-all duration-200 ease-out hover:-translate-y-1 hover:shadow-lg">
  <Card />
</div>

// Button press feedback
<button className="transition-transform duration-75 active:scale-95">
  Click me
</button>
```

### Framer Motion (complex animations)

```typescript
import { motion, AnimatePresence } from 'framer-motion'

// Page transition matching Figma's "Move In from Right"
const pageVariants = {
  initial: { opacity: 0, x: 20 },
  animate: { opacity: 1, x: 0 },
  exit: { opacity: 0, x: -20 },
}

export function PageWrapper({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      variants={pageVariants}
      initial="initial"
      animate="animate"
      exit="exit"
      transition={{ duration: 0.2, ease: 'easeOut' }}
    >
      {children}
    </motion.div>
  )
}

// List item stagger (common in Figma prototypes)
const containerVariants = {
  hidden: { opacity: 0 },
  show: {
    opacity: 1,
    transition: {
      staggerChildren: 0.05,
    },
  },
}

const itemVariants = {
  hidden: { opacity: 0, y: 10 },
  show: { opacity: 1, y: 0 },
}

export function AnimatedList({ items }: { items: Item[] }) {
  return (
    <motion.ul variants={containerVariants} initial="hidden" animate="show">
      {items.map((item) => (
        <motion.li key={item.id} variants={itemVariants}>
          <ListItem item={item} />
        </motion.li>
      ))}
    </motion.ul>
  )
}
```

### Respecting prefers-reduced-motion

Always wrap animations:

```typescript
// Hook
function usePrefersReducedMotion() {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(true)
  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)')
    setPrefersReducedMotion(mediaQuery.matches)
    const handleChange = () => setPrefersReducedMotion(mediaQuery.matches)
    mediaQuery.addEventListener('change', handleChange)
    return () => mediaQuery.removeEventListener('change', handleChange)
  }, [])
  return prefersReducedMotion
}

// Usage
const prefersReducedMotion = usePrefersReducedMotion()
const transition = prefersReducedMotion ? { duration: 0 } : { duration: 0.2, ease: 'easeOut' }
```

---

## Design Tokens: Figma Variables → CSS → Tailwind

This is the full pipeline for a properly set-up design system.

### Step 1: Structure tokens in Figma

Figma variables should have three tiers:
- **Primitives** (raw values): `blue-500 = #3B82F6`, `space-4 = 16`
- **Semantic tokens** (meaning): `color/primary = {blue-500}`, `spacing/section = {space-4}`
- **Component tokens** (scoped): `button/background = {color/primary}`

### Step 2: Export tokens

Use the Figma Tokens or Variables to JSON plugin, or read via the MCP `get_variable_defs` tool. The output is a JSON structure:

```json
{
  "color": {
    "primary": { "value": "#3B82F6", "type": "color" },
    "background": { "value": "#FFFFFF", "type": "color" },
    "background-dark": { "value": "#0F172A", "type": "color" }
  },
  "space": {
    "1": { "value": "4px", "type": "dimension" },
    "2": { "value": "8px", "type": "dimension" },
    "4": { "value": "16px", "type": "dimension" }
  },
  "typography": {
    "size-sm": { "value": "14px", "type": "dimension" },
    "size-base": { "value": "16px", "type": "dimension" },
    "size-lg": { "value": "18px", "type": "dimension" }
  }
}
```

### Step 3: Generate CSS custom properties

```css
/* tokens.css — generated from Figma, do not edit manually */
:root {
  /* Primitives */
  --color-blue-500: #3b82f6;
  --color-neutral-50: #f8fafc;

  /* Semantic */
  --color-primary: var(--color-blue-500);
  --color-background: var(--color-neutral-50);

  /* Spacing */
  --space-1: 4px;
  --space-2: 8px;
  --space-4: 16px;
  --space-8: 32px;

  /* Typography */
  --font-size-sm: 14px;
  --font-size-base: 16px;
  --font-size-lg: 18px;
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-bold: 700;
}

.dark {
  --color-background: #0f172a;
  --color-primary: var(--color-blue-400);
}
```

### Step 4: Extend Tailwind config

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss'

const config: Config = {
  darkMode: ['class'],
  theme: {
    extend: {
      colors: {
        primary: 'var(--color-primary)',
        background: 'var(--color-background)',
      },
      spacing: {
        // Map Figma spacing scale
        '1': 'var(--space-1)',
        '2': 'var(--space-2)',
        '4': 'var(--space-4)',
        '8': 'var(--space-8)',
      },
      fontSize: {
        sm: 'var(--font-size-sm)',
        base: 'var(--font-size-base)',
        lg: 'var(--font-size-lg)',
      },
    },
  },
}

export default config
```

### Step 5: Automate the sync

Set up a script that reads the Figma API and regenerates `tokens.css` and `tailwind.config.ts` on demand:

```bash
# package.json script
"tokens:sync": "node scripts/sync-figma-tokens.js"
```

---

## Next.js App Router Specific Patterns

### Server Components by default

```typescript
// Fetch data at the component level — no useEffect needed
// app/dashboard/page.tsx
async function DashboardPage() {
  const data = await fetchDashboardData() // runs on server
  return <DashboardClient initialData={data} />
}
```

### Client Components for interactivity

```typescript
'use client'
// Only add 'use client' when you need:
// - useState / useEffect
// - event listeners
// - browser APIs
// - Framer Motion animations
```

### Image optimization

Always use next/image. Map Figma image specs to the component:

```typescript
import Image from 'next/image'

// Figma: 320x200, object-fit: cover
<Image
  src={src}
  alt={alt}
  width={320}
  height={200}
  className="rounded-lg object-cover"
  priority={isAboveFold}
/>
```

### Font loading

Match Figma fonts to Next.js font system:

```typescript
// app/layout.tsx
import { Inter, Geist } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
})

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={inter.variable}>
      <body>{children}</body>
    </html>
  )
}
```

Then in Tailwind:
```typescript
fontFamily: {
  sans: ['var(--font-inter)', 'system-ui', 'sans-serif'],
}
```

### Parallel routes for complex layouts

When a Figma design has a modal that shows on top of a page (not replacing it), use parallel routes:

```
app/
  @modal/
    (.)photos/[id]/
      page.tsx  <- intercepted modal
  photos/
    [id]/
      page.tsx  <- full page (direct URL access)
  layout.tsx
```

---

## Testing Visual Output Against Designs

### Approach 1: Side-by-side screenshot comparison

Use the Figma MCP `get_screenshot` tool to pull a reference screenshot, then screenshot your running app:

```
Get a screenshot of the HeroSection component from Figma: [node URL]
Here is my implementation screenshot: [attach screenshot]
List every visual difference you see, grouped by: spacing, typography, color, layout.
```

### Approach 2: Chromatic / Storybook visual regression

```bash
npm install --save-dev @chromatic-com/storybook
```

Set up Storybook stories for every component. Chromatic captures screenshots on every PR and shows diffs. Connect Figma's design tokens to Storybook's theme controls so reviewers can toggle between Figma spec and implementation.

### Approach 3: Playwright visual snapshots

```typescript
// tests/visual/hero.spec.ts
import { test, expect } from '@playwright/test'

test('HeroSection matches design', async ({ page }) => {
  await page.goto('/storybook/iframe.html?id=sections-hero--default')
  await expect(page).toHaveScreenshot('hero-default.png', {
    maxDiffPixelRatio: 0.01, // 1% tolerance
  })
})
```

---

## Common Figma-to-Code Gaps

### Gap 1: Line height

Figma uses pixel line-heights. CSS and Tailwind use unitless multipliers by default.

```
Figma: font-size 16px, line-height 24px
CSS equivalent: font-size: 16px; line-height: 1.5
Tailwind: text-base leading-6
```

Tell Claude explicitly: "Convert all Figma pixel line-heights to unitless ratios."

### Gap 2: Letter spacing

Figma shows letter-spacing in percent or pixels. Tailwind uses em-based classes.

```
Figma: letter-spacing 5% at 14px = 0.7px
CSS: letter-spacing: 0.05em
Tailwind: tracking-wide (0.025em) — not exact, adjust in theme
```

### Gap 3: Box shadows

Figma shadows have X, Y, blur, spread, color, opacity. CSS shadows follow the same order but opacity is in the color:

```
Figma: X=0, Y=4, Blur=8, Spread=0, Color=#000 20%
CSS: box-shadow: 0 4px 8px 0 rgb(0 0 0 / 0.2)
```

### Gap 4: Figma "Auto Layout" → flexbox/grid

| Figma Auto Layout | CSS |
|-------------------|-----|
| Direction: Horizontal | `flex-direction: row` |
| Direction: Vertical | `flex-direction: column` |
| Spacing: Packed | `justify-content: flex-start` |
| Spacing: Space Between | `justify-content: space-between` |
| Align: Center | `align-items: center` |
| Gap: 8 | `gap: 8px` |
| Padding: 12 16 | `padding: 12px 16px` |
| Fill container | `flex: 1` |
| Hug contents | default (no explicit size) |
| Fixed width | `width: Xpx` |

### Gap 5: Figma gradients

```
Figma linear gradient at 135deg, #3B82F6 → #8B5CF6
CSS: background: linear-gradient(135deg, #3B82F6, #8B5CF6)
Tailwind: bg-gradient-to-br from-blue-500 to-violet-500
```

### Gap 6: Fonts not loading

AI won't add font imports. After building, check:
1. Is the font in `next/font/google` or imported in CSS?
2. Is the font family name in Tailwind config matching exactly?
3. Are the font weights you're using included in the subset?

### Gap 7: Figma prototypes vs CSS hover states

Figma hover states are modeled as separate component variants. In code, these become pseudo-classes:

```
Figma: Button/Default + Button/Hover variant
Code: <button className="bg-primary hover:bg-primary/90">
```

Ask Claude: "For every component variant that differs only on hover or focus, implement it as a CSS pseudo-class rather than a separate component or JS state."

### Gap 8: Clipping and overflow

Figma clips by default at frame boundaries. CSS doesn't. If elements are unexpectedly visible outside their container, add `overflow-hidden` to the parent.

---

## Tech Stack Configs That Work Well

**Next.js + Tailwind + shadcn/ui**
Ask Claude to use existing shadcn components as the base and extend them. Avoids reinventing primitives and keeps your design system consistent.

**React + CSS Modules**
Better for design-heavy work where you need precise control. Ask Claude to use BEM-style class names for readability.

**Vue 3 + Tailwind**
Same pattern as React + Tailwind. Specify `<script setup>` and Composition API or Claude may default to Options API.

---

## Common Pitfalls

**AI over-engineers.** Ask for the simplest implementation that matches the design. If you don't specify, you'll get abstractions you don't need.

**Browser vs design gaps.** Always check spacing and typography in an actual browser. Figma spacing doesn't map 1:1 to CSS and line-height behaves differently.

**Responsive assumptions.** AI will guess at breakpoints. Specify them or review the responsive behavior carefully before shipping.

**Font loading.** Make sure the fonts from Figma are loaded in your project. AI won't add font imports unless you explicitly ask.

**Skipping dark mode tokens early.** If you hardcode colors in the first sprint and add dark mode later, you'll refactor every component. Set up CSS variables from day one.

**Forgetting `use client` boundaries.** In Next.js App Router, any component that uses hooks, events, or Framer Motion needs `'use client'`. AI sometimes forgets this on animation wrappers.
