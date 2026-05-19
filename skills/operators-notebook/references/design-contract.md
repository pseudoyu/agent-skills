# Operator's Notebook — Design Contract

The full register specification. Read this before designing or migrating; the recipes assume you understand the why.

## Aesthetic Direction

**Operator's Notebook.** A dense BI/admin console read in short, focused bursts. Cool off-white biased toward ink-blue (never warm cream). Foreground is near-black ink-blue. Hairline dividers carry hierarchy instead of card chrome and shadow. All neutrals sit on the `h~250` hue family so the palette reads as one family. Accent is reserved for a single active/confirmed state per screen — 60/30/10 weight discipline.

The visual opposite of:
- "Default shadcn SaaS" (blue-500 primary, 12px rounded cards, soft shadows on white)
- Notion clones (overly airy whitespace, decorative emoji)
- Stripe-flavored finance UI (gradient stat cards, animated number tickers)
- Linear-flavored issue trackers (high-contrast purple accent, glassy popovers)
- Mixpanel/Amplitude analytics tools (saturated multi-color charts, badge-heavy chrome)

It shares discipline with:
- The Economist's data graphics (flat, hue-locked, dense)
- Financial Times' app numerics (Sometype/tabular-figures everywhere)
- Bloomberg Terminal density (without the dark canvas)
- An accountant's ledger book (hairlines, mini-cap labels, mono numerics)

## Why a Dedicated Register

Default shadcn/ui ships a register that fits consumer SaaS dashboards. For an internal operator tool — read 8 hours a day by the same five people — that voice creates friction:

- Soft shadows on cards add visual weight without information value
- Pillowy 12px radii make corners read as decorative rather than precise
- Blue-500 primary fights with status colors (success green, warning amber)
- Geist Sans body + Geist Mono numeric is correct but generic; nothing about the type carries the operator's voice
- Stagger animations on cards are charming once, then they cost every reload

The Operator's Notebook fixes all of these by collapsing chrome (hairlines + whitespace), tightening type (Bricolage + Sometype carry the voice; body is just legible), and reserving accent for one role per screen (60/30/10).

## Token Contract

OKLCH on the `h~250` hue family. Light mode is canonical. **Dark mode is not part of this register** — if a project needs dark mode, that's a separate token contract and should be tested per screen before shipping.

```css
:root {
  --background:             oklch(98% 0.003 250);  /* cool off-white, never warm */
  --foreground:             oklch(22% 0.020 250);  /* near-black ink-blue */
  --card:                   oklch(99% 0.002 250);  /* visually equal to background */
  --card-foreground:        oklch(22% 0.020 250);
  --popover:                oklch(99% 0.002 250);
  --popover-foreground:     oklch(22% 0.020 250);
  --primary:                oklch(35% 0.040 250);  /* deep ink-blue, the only accent */
  --primary-foreground:     oklch(98% 0.003 250);
  --secondary:              oklch(94% 0.004 250);
  --secondary-foreground:   oklch(28% 0.020 250);
  --muted:                  oklch(96% 0.003 250);
  --muted-foreground:       oklch(52% 0.010 250);
  --accent:                 oklch(92% 0.025 250);  /* hover/active row tint */
  --accent-foreground:      oklch(28% 0.030 250);
  --destructive:            oklch(52% 0.140 28);   /* the only off-family hue */
  --destructive-foreground: oklch(98% 0.003 250);
  --border:                 oklch(88% 0.004 250);  /* hairlines */
  --input:                  oklch(88% 0.004 250);
  --ring:                   oklch(35% 0.040 250);
  --radius:                 0.25rem;               /* never pillowy */

  /* Chart series — hue-locked, intentionally desaturated.
   * Cycle 1→5 for multi-series; never reach for a saturated qualitative palette. */
  --chart-1: oklch(42% 0.080 250);
  --chart-2: oklch(55% 0.060 220);
  --chart-3: oklch(62% 0.050 280);
  --chart-4: oklch(48% 0.070 200);
  --chart-5: oklch(58% 0.055 260);
}
```

Do not nudge these per-screen. Per-screen tweaks compound into drift.

## Typography

Three-font stack. Each font has exactly one job; never mix roles.

### Body (legible, language-aware)

The body font must support the user-facing language. Recommended:

- **`Noto Sans SC`** for Simplified Chinese UI
- **`Noto Sans JP`** for Japanese UI
- **`Inter`** or **`Noto Sans`** for Latin-first UI
- Weights: 400 / 500 / 600 / 700

The body font does not carry voice. It's the legibility floor.

### Display (Bricolage Grotesque, variable)

Used for:
- Mini-cap section labels (`text-[11px] font-medium uppercase tracking-[0.1em]`)
- Page headers (`text-2xl font-semibold tracking-tight`)
- Metric figures (`text-[28px] font-semibold leading-none`)
- Card titles where they exist

Tight tracking at display sizes. The geometric editorial face is what gives the register its voice — do not substitute.

### Mono (Sometype Mono)

Reserved for tabular contexts:
- IDs (`YJ-1234567`)
- Timestamps (`2026-05-19 14:32`)
- Prices and amounts (`¥12,345.67`)
- Pagination (`5 / 20`)
- OTP digit slots (`tracking-[0.25em]`)
- Live-update stamps in headers

**Never in running prose.** Sometype's character is warmer than JetBrains Mono; it sits with the editorial display face naturally.

### Tabular figures (global)

```css
body {
  font-feature-settings: "tnum";
}
```

So column-aligned numbers stay aligned regardless of digit width. Prose contexts can override per-element if needed (`font-feature-settings: "normal"` on a paragraph).

## Component Patterns

### 1. Hairline metric strip

Replaces the SaaS "card grid of stats." Five cells in a row, divided by hairlines, no per-cell border or shadow:

```tsx
<div className="grid grid-cols-2 divide-x divide-y divide-border border border-border md:grid-cols-5 md:divide-y-0">
  {cells.map((c) => (
    <StatCard key={c.label} label={c.label} value={c.value} hint={c.hint} accent={c.accent} />
  ))}
</div>
```

Each cell:
- Mini-cap label on top
- Bricolage 28px figure
- Optional 11px hint below
- Optional delta chip (mono, with `text-primary` for up, `text-destructive` for down)

**Exactly one cell** carries the `accent` flag — the operator's most-watched metric. Its value renders in `text-primary` and the cell gets a `bg-primary/5` wash. Every other cell uses foreground ink on plain background.

### 2. Mini-cap section label

```tsx
<h2 className="font-display text-[11px] font-medium uppercase tracking-[0.1em] text-muted-foreground border-b border-border pb-2">
  最近活动
</h2>
```

The same shape across dashboard, list views, and chart sections. Consistent eyebrow treatment is what makes the whole app read as one ledger rather than mix-and-match pages.

### 3. Page header

```tsx
<header className="border-b border-border pb-4">
  <p className="font-display text-[11px] font-medium uppercase tracking-[0.1em] text-muted-foreground">
    {eyebrow}
  </p>
  <h1 className="mt-2 font-display text-2xl font-semibold tracking-tight text-foreground">
    {title}
  </h1>
  {description && (
    <p className="mt-1 text-sm text-muted-foreground">{description}</p>
  )}
</header>
```

The mini-cap eyebrow is optional but recommended for consistency. The bottom hairline separates the header from the page body without needing margin alone to do the work.

### 4. Sidebar nav (60-wide, hairline right edge, ink-dot active)

```tsx
<aside className="fixed inset-y-0 hidden w-60 flex-col border-r border-border bg-background md:flex">
  <div className="border-b border-border px-4 py-4">
    <span className="font-display text-base font-semibold tracking-tight">
      {brand}
    </span>
  </div>
  <nav className="flex-1 space-y-1 px-2 py-4">
    {items.map((item) => {
      const isActive = pathname === item.href || pathname.startsWith(item.href + '/');
      return (
        <Link
          key={item.href}
          href={item.href}
          className={cn(
            "flex items-center gap-2 px-3 py-2 text-sm transition-colors hover:bg-accent/30",
            isActive ? "bg-accent/55 text-foreground" : "text-muted-foreground",
          )}
        >
          <span className={cn("h-1.5 w-1.5 rounded-full", isActive ? "bg-primary" : "bg-transparent")} />
          <item.icon className="h-4 w-4" />
          <span>{item.label}</span>
        </Link>
      );
    })}
  </nav>
</aside>
```

**No `border-l-2` accent stripe** on the active row. The ink-dot + row tint is the entire active vocabulary. (Active-state-by-side-stripe is BAN 1 in [`bans.md`](bans.md).)

### 5. Header (solid, no glass)

```tsx
<header className="sticky top-0 z-30 flex h-14 items-center justify-between border-b border-border bg-background px-4 md:px-6">
  <div className="flex items-center gap-3">
    {/* mobile-nav toggle slot */}
    <MobileNavToggle />
  </div>
  <div className="flex items-center gap-3">
    {user && (
      <span className="font-mono text-xs tabular-nums text-muted-foreground">
        {user.displayName}
      </span>
    )}
    <Button variant="ghost" size="sm" onClick={logout}>退出</Button>
  </div>
</header>
```

Solid `bg-background`. **No `backdrop-blur`** (glassmorphism is BAN 9). The mono displayName chip and the ghost logout button keep the header weight-light.

### 6. Charts (recharts, flat + editorial)

Constants in `lib/chart-typography.ts`:

```ts
export const CHART_GRID_DASH = "2 4";
export const CHART_TICK_SM = { fontSize: 11, fill: "var(--muted-foreground)" };
export const CHART_TICK_MD = { fontSize: 12, fill: "var(--muted-foreground)" };
export const CHART_TOOLTIP_CONTENT_STYLE = {
  background: "var(--card)",
  border: "1px solid var(--border)",
  borderRadius: 4,
  fontSize: 12,
  color: "var(--foreground)",
};
```

Patterns:
- `CartesianGrid strokeDasharray="2 4"` on `stroke-border`
- Axes inherit `text-muted-foreground` ticks, no axis line color override
- `Area` / `Line` stroke: `var(--primary)` at `strokeWidth={1.5}`, `fillOpacity={0.12}`
- `Bar` fill: `var(--primary)`, **flat-top** `radius={[0, 0, 0, 0]}` (rounded bar caps read as SaaS, banned here)
- Tooltips: solid `var(--card)` bg + `var(--border)` 1px border + `var(--foreground)` text (no glass / blur)
- Multi-series: cycle `--chart-1..5`. Never reach for a saturated qualitative palette
- Legend: text + token-tinted swatch, no `bg-primary/10` chips next to swatches (that pattern reads as Mixpanel tooling)

## Active-State Vocabulary (one per surface type)

To prevent active-state drift across components, this register fixes one vocabulary per surface:

| Surface              | Active state                                                                 |
| -------------------- | ---------------------------------------------------------------------------- |
| Sidebar nav row      | `bg-accent/55 text-foreground` + ink-dot (`h-1.5 w-1.5 rounded-full bg-primary`) |
| Tab strip            | `data-[state=active]:after:bg-foreground` underline (no pill background)     |
| Toggle / Switch on   | `bg-primary` thumb track                                                     |
| Icon button on       | `bg-primary/10 text-primary`                                                 |
| Status badge active  | `border border-primary/40 bg-primary/5 text-primary`                         |
| Stat cell highlight  | `bg-primary/5` + value `text-primary`                                        |
| Count badge          | `bg-foreground text-background font-mono tabular-nums`                        |
| Form field focus     | `ring-1 ring-ring` (no glow, no shadow)                                      |

If your active state doesn't match one of these, you're inventing — re-use the closest match.

## 60/30/10 Accent Discipline

Per screen:
- **60%** is canvas + muted-foreground text (the bulk of the page is quiet)
- **30%** is foreground ink (the metrics, the table body, the form fields)
- **10%** is primary (the active nav row, the one accent stat cell, the confirmed CTA, the on-state of one toggle)

Count the primary-tinted surfaces on every screen you ship. If there are more than ~3, the accent is over-budget.

## Refresh / Caching Hints (optional, for polling dashboards)

If your operator dashboard polls upstream metrics every N seconds:

- **Frontend**: TanStack Query with `refetchInterval` and a `staleTime` slightly shorter than the interval (so manual refresh is responsive)
- **Backend**: wrap upstream calls in an in-memory LRU cache keyed by procedure + input hash, with TTL matched to the polling interval — absorbs the per-panel fan-out
- **Header timestamp**: render the *server-side* `lastUpdatedAt` in `font-mono tabular-nums`, not the client's `now()`. Operators see the real freshness floor, not the polling tick

## Process Standards

- Every design pass ends with `pnpm typecheck && pnpm lint && pnpm build` all green
- Grep guards in pre-commit / CI:
  - `grep -rE "border-l-[2-9]\b"` — zero hits
  - `grep -rE "bg-clip-text"` — zero hits
  - `grep -rE "bg-(white|black)\b"` — zero in app code
  - `grep -rE "(bg|text)-(emerald|green|amber|sky|purple|red|blue|gray|slate|zinc|neutral)-[0-9]"` — zero in app code (UI primitives are an allowed exception if they ship shadcn defaults)
- Visual verification via Playwright on every route after a register-level change
- Commit messages reference the register: `feat(admin): notebook chrome migration`, `polish(admin): swap remaining color literals to semantic tokens`
