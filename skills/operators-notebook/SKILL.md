---
name: operators-notebook
description: Operator's Notebook design register — a dense BI/admin dashboard visual language built on OKLCH ink-blue tokens, hairline dividers, Bricolage Grotesque display, and 60/30/10 accent discipline. Use when designing or migrating admin dashboards, internal tools, BI consoles, or operator-facing surfaces that need editorial clarity over SaaS chrome. Triggers on tasks involving shadcn/ui dashboards, Next.js admin apps, Tailwind v4 token migration, or rewriting "default shadcn" SaaS-flavored UIs into a disciplined editorial register.
---

# Operator's Notebook

A complete design register for dense, glance-and-decide internal dashboards. Cool off-white canvas biased toward ink-blue, hairlines instead of card shadow, editorial display type, semantic-token-only color discipline. The visual opposite of "default shadcn SaaS."

## When to Apply

Reach for this register when building:

- Admin dashboards (ops, finance, content moderation)
- Internal BI consoles reading from PostHog / Metabase / warehouse
- CRUD-heavy operator tools (player lists, store management, audit queues)
- Multi-tenant back-office surfaces
- Any Next.js + Tailwind v4 + shadcn/ui app where the current look reads as "generic SaaS" and you want editorial discipline
- Migrations from default shadcn (12px rounded cards, blue-500 primary, Geist Sans) to a single coherent voice

Avoid for:

- Consumer marketing sites (the register is intentionally chrome-poor)
- Mobile-first apps where dense hairline strips collapse poorly
- Brand-led product surfaces that need a distinctive accent system (this register reserves accent for "active" state only)

## What You Get

1. **A complete OKLCH token contract** (h~250, `--radius: 0.25rem`, no dark-mode drift)
2. **A three-font stack** (Bricolage Grotesque display + body font + Sometype Mono numerics)
3. **shadcn/ui primitive variants** tuned to the register (no shadow, no pillowy radius, transparent outline, ring-1 focus)
4. **Shared primitives** (`PageHeader`, `StatCard`, `SectionLabel`, `EmptyState`)
5. **Layout chrome** (60-wide hairline sidebar with ink-dot active, solid header, CSS-only mobile slide-in)
6. **Chart typography contract** (recharts constants — flat bars, dashed grid, hue-locked series)
7. **A migration playbook** for taking a default-shadcn admin to this register without touching data flow

## Quick Reference

### Token contract (the only colors you ever use)

```css
:root {
  --background:             oklch(98% 0.003 250);
  --foreground:             oklch(22% 0.020 250);
  --card:                   oklch(99% 0.002 250);
  --primary:                oklch(35% 0.040 250);
  --accent:                 oklch(92% 0.025 250);
  --muted:                  oklch(96% 0.003 250);
  --muted-foreground:       oklch(52% 0.010 250);
  --destructive:            oklch(52% 0.140 28);
  --border:                 oklch(88% 0.004 250);
  --radius:                 0.25rem;
}
```

Full contract (chart tokens, popover, ring, semantic foregrounds) in [`references/recipes/tokens.md`](references/recipes/tokens.md).

### Semantic mapping (the only `text-*`/`bg-*` color utilities allowed in app code)

| Role                         | Token                                       |
| ---------------------------- | ------------------------------------------- |
| Canvas                       | `bg-background`                             |
| Card / popover surface       | `bg-card` (visually equal to background)    |
| Primary ink                  | `text-foreground`                           |
| Faded ink (labels, captions) | `text-muted-foreground`                     |
| Active / confirmed / positive | `text-primary`, `bg-primary/5`, `border-primary/40` |
| Negative / destructive       | `text-destructive`, `bg-destructive/5`     |
| Hairline                     | `border-border`                             |
| Subtle fill                  | `bg-muted`, `bg-accent/55`                  |

**No** `bg-emerald-*`, `bg-green-*`, `bg-amber-*`, `bg-blue-*`, `bg-purple-*`, `bg-sky-*`, `text-red-500` or any raw Tailwind color. Map every existing literal to a semantic token. The grep `grep -rE "(bg|text)-(emerald|green|amber|sky|purple|red|blue|gray|slate|zinc|neutral)-[0-9]"` should return zero hits in app code (UI primitives excepted only if they ship with the shadcn defaults you haven't yet touched).

### Type stack

| Role     | Font                               | CSS variable        | Notes                                                            |
| -------- | ---------------------------------- | ------------------- | ---------------------------------------------------------------- |
| Body     | Noto Sans SC / Noto Sans JP / Inter | `--font-body`       | Match the user-facing language. Weights 400/500/600/700.        |
| Display  | Bricolage Grotesque (variable)     | `--font-display`    | Section labels, headlines, metric figures. Tight tracking.       |
| Mono     | Sometype Mono                      | `--font-mono`       | IDs, timestamps, prices, tabular metrics. Never running prose.   |

Globally enable tabular figures: `body { font-feature-settings: "tnum" }`.

Recipes for `next/font/google` wiring in [`references/recipes/fonts.md`](references/recipes/fonts.md).

### Canonical patterns (memorize these five)

#### 1. Hairline metric strip (replaces the SaaS card grid)

```tsx
<div className="grid grid-cols-1 divide-x divide-y divide-border border border-border md:grid-cols-5 md:divide-y-0">
  {cells.map((c) => (
    <StatCard key={c.label} label={c.label} value={c.value} hint={c.hint} accent={c.accent} />
  ))}
</div>
```

- Each cell: label + Bricolage 28px figure + optional hint
- **No icons in the strip**
- **Exactly one cell per screen** carries `accent` — that's the 60/30/10 highlighter

#### 2. Mini-cap section label (replaces card titles)

```tsx
<h2 className="font-display text-[11px] font-medium uppercase tracking-[0.1em] text-muted-foreground border-b border-border pb-2">
  预约日程
</h2>
```

Same shape across dashboard, list views, and chart sections so the whole app reads as one ledger.

#### 3. Ink-dot active nav (no border-left stripe)

```tsx
<Link
  href={item.href}
  className={cn(
    "flex items-center gap-2 px-3 py-2 text-sm transition-colors hover:bg-accent/30",
    isActive ? "bg-accent/55 text-foreground" : "text-muted-foreground",
  )}
>
  {isActive ? <span className="h-1.5 w-1.5 rounded-full bg-primary" /> : <span className="h-1.5 w-1.5" />}
  <item.icon className="h-4 w-4" />
  <span>{item.label}</span>
</Link>
```

#### 4. Hairline card surface (replaces shadow + pillowy radius)

```tsx
<Card className="border border-border bg-card p-5">
  {/* No `shadow-sm`, no `rounded-2xl`, no backdrop-blur */}
</Card>
```

The `Card` primitive itself is tuned to ship `rounded-md border border-border bg-card` with no shadow — see [`references/recipes/primitives.md`](references/recipes/primitives.md).

#### 5. Charts (recharts, flat + editorial)

```ts
export const CHART_GRID_DASH = "2 4";
export const CHART_TICK_SM = { fontSize: 11, fill: "var(--muted-foreground)" };
export const CHART_TOOLTIP_CONTENT_STYLE = {
  background: "var(--card)",
  border: "1px solid var(--border)",
  borderRadius: 4,
  fontSize: 12,
  color: "var(--foreground)",
};
```

Full chart contract (axes, bar radius=0, area `fillOpacity: 0.12`, hue-locked series) in [`references/recipes/charts.md`](references/recipes/charts.md).

## Universal BANs (match-and-refuse)

These are absolute. If your PR adds one of these, the PR is wrong:

1. **No `border-left` / `border-right` ≥ 2px as accent stripe** — use a leading icon, tinted background, or the ink-dot pattern
2. **No gradient text** (`bg-clip-text` + any gradient) — text fills are solid color, always
3. **No bounce / elastic easing** — use `ease-out` curves only; one authored anticipation curve max per app
4. **No `animate-{width,height,top,left,padding,margin}`** — transform + opacity only
5. **No `bg-card` shadow chrome** — if you reach for `shadow-sm`/`shadow-md`, you've broken the register
6. **No `bg-white` / `bg-black`** — overlays use `bg-foreground/40`, surfaces use `bg-card`
7. **No raw Tailwind color literals** (`bg-emerald-100`, `text-red-500`, etc.) — map to semantic tokens
8. **No icons in the metric strip** — cells are label + figure only
9. **No glassmorphism** (`backdrop-blur` on chrome) — solid `bg-popover` / `bg-card`
10. **No layout-prop animation in chrome** — sidebar/header are CSS-transition only; framer-motion stays for genuine modal/form orchestration

Full taxonomy + the perl scrub commands in [`references/bans.md`](references/bans.md).

## Implementation Checklist (big-bang migration)

Order matters. Don't skip phases — token drift propagates fast.

- [ ] **Phase 1 — Foundation**: rewrite `globals.css` to the OKLCH contract, delete `.dark` block + `tw-animate-css` + `next-themes`, wire three fonts via `next/font/google`, set `lang` correctly
- [ ] **Phase 2 — shadcn/ui primitives**: transplant the Notebook-tuned variants of `button`, `card`, `input`, `dialog`, `badge`, `label`, `textarea`, `tabs`, `select`, `dropdown-menu`, `tooltip`, `switch`, `checkbox`, `scroll-area`, `separator`, `skeleton`, `avatar`, `table`
- [ ] **Phase 3 — Shared primitives**: create `components/shared/{page-header,section-label,stat-card,empty-state}.tsx` + `lib/chart-typography.ts`
- [ ] **Phase 4 — Chrome**: rewrite `sidebar`, `header`, `mobile-nav` (CSS slide-in), root layout (`md:pl-60` shell)
- [ ] **Phase 5 — Auth pages**: login → centered 360px column with mini-cap eyebrow + Bricolage title + hairline divider
- [ ] **Phase 6 — Page-level chrome swap**: visit every route, swap `<Card className="shadow-sm">` → `<Card>`, stat grids → hairline strips, section dividers → `<SectionLabel>`, color literals → semantic tokens. **Do not touch oRPC calls, React Query hooks, form schemas, mutation callbacks, or permission checks.**
- [ ] **Phase 7 — Verification**: `grep -rE "border-l-[2-9]|bg-clip-text|bg-(white|black)|shadow-sm|rounded-(xl|2xl|3xl)|text-[0-9]"` returns zero in app code. `pnpm typecheck && pnpm lint && pnpm build` all green. Playwright sweep every route, screenshot, assert no React warnings.

Full migration playbook in [`references/recipes/migration-checklist.md`](references/recipes/migration-checklist.md).

## File Map

| Path                                                    | What it gives you                                      |
| ------------------------------------------------------- | ------------------------------------------------------ |
| `references/design-contract.md`                         | Full register description (aesthetic, why, when, where) |
| `references/bans.md`                                    | All 10 absolute BANs + grep commands to enforce them   |
| `references/recipes/tokens.md`                          | Drop-in `globals.css` (full token block + keyframes)   |
| `references/recipes/fonts.md`                           | `lib/fonts.ts` + `app/layout.tsx` wiring               |
| `references/recipes/primitives.md`                      | shadcn/ui variant overrides (button/card/input/dialog) |
| `references/recipes/shared-primitives.md`               | `PageHeader`, `StatCard`, `SectionLabel`, `EmptyState` |
| `references/recipes/chrome.md`                          | Sidebar / header / mobile-nav / layout shell           |
| `references/recipes/login.md`                           | Centered 360px login chrome                            |
| `references/recipes/charts.md`                          | `chart-typography.ts` + recharts patterns              |
| `references/recipes/migration-checklist.md`             | Big-bang migration playbook                            |

## Key Principles

1. **Hairlines, not chrome.** A `border border-border` carries more hierarchy in this register than any shadow.
2. **One accent per screen.** Primary tint is the 10% of the 60/30/10 — it marks the single most-watched metric, the active nav row, the confirmed CTA. If two things on screen claim "primary," the screen is wrong.
3. **Semantic tokens, always.** Never reach for `bg-emerald-100`. If you can't name the role (`positive`, `destructive`, `muted`, `foreground`), the color doesn't belong.
4. **Density beats whitespace.** A 5-cell metric strip + chart + table on one viewport is the goal, not a 3-card grid with breathing room.
5. **Editorial type, not SaaS type.** Bricolage Grotesque on display moments + Sometype Mono on every number creates the voice. The body font carries language; the display and mono do the lifting.
6. **CSS for chrome motion, framer-motion for orchestration.** Sidebar/header/nav are pure transitions. Modals and multi-step forms can still use framer-motion where genuine orchestration helps.
7. **No dark mode unless explicitly required.** The register is light-mode canonical. Adding dark tokens without testing every screen creates drift.
