# Universal BANs

These are absolute. They are not stylistic preferences; they are the boundary that separates "Operator's Notebook" from "default shadcn SaaS." If a PR adds one of these, the PR is wrong — fix the PR, don't relax the BAN.

Each BAN comes with:
- **Why** it's banned
- **What to do instead**
- **Grep command** to enforce it in CI / pre-commit

---

## BAN 1 — No `border-left` / `border-right` ≥ 2px as accent stripe

**Why**: Side-stripes are the laziest "this is the active state" affordance in SaaS dashboards. They consume horizontal space, fight with the hairline grid, and signal "callout component" rather than "selected row." This register marks active state with an ink-dot + row tint instead.

**Instead**:
- Active nav row: `bg-accent/55` + `<span className="h-1.5 w-1.5 rounded-full bg-primary" />`
- Callout / quote block: leading icon + `bg-muted/40` body, no stripe
- Dialogue block: brackets 「」 or `“ ”` carry the role

**Grep**:
```bash
grep -rE "border-l-[2-9]\b|border-r-[2-9]\b" components/ app/
# Zero hits expected. `border-l` (1px) is fine; `border-l-2` and above are banned.
```

---

## BAN 2 — No gradient text

**Why**: `background-clip: text` + any gradient is the marketing-page anti-pattern. It fails accessibility contrast checks, breaks high-contrast modes, and pulls visual weight without information value. Text fills are solid color, always.

**Instead**:
- Brand titles: solid `text-foreground` or `text-primary`
- Emphasis: weight (`font-semibold`), tracking, or hairline underline — not gradient

**Grep**:
```bash
grep -rE "bg-clip-text|text-transparent" components/ app/
# Zero hits. (`text-transparent` only legitimate use is on tooltip arrows, which should not need it.)
```

---

## BAN 3 — No bounce / elastic easing

**Why**: Bounce and elastic curves (`cubic-bezier(0.68, -0.55, 0.27, 1.55)` and friends) are charming once. By the tenth interaction in a workday they're noise. Operator tools should feel responsive, not playful.

**Instead**:
- `transition-all ease-out duration-150` for hover / focus
- `ease-out` curves only — `ease-out-quart`, `ease-out-quint`, `ease-out-expo` if you want named curves
- One authored anticipation curve per app, max — and not on chrome (e.g., a celebratory toast animation can have a single +1.05 overshoot; nothing else)

**Grep**:
```bash
grep -rE "cubic-bezier\([0-9.]+,\s*-" components/ app/ styles/
# Negative second value indicates overshoot easing.
```

---

## BAN 4 — No `animate-{width,height,top,left,padding,margin}`

**Why**: Layout-prop animations trigger reflow on every frame. They jank, they fight scroll restoration, and they break `prefers-reduced-motion` even when the property name doesn't include "motion." Transform + opacity stay on the compositor.

**Instead**:
- Slide-in panel: `transform: translateX(-100%)` → `translateX(0)` with `transition-transform`
- Reveal: `opacity: 0` → `opacity: 1` with `transition-opacity`
- Resize: don't — design a layout that doesn't need to animate to a new size

**Grep**:
```bash
grep -rE "transition-(width|height|all)|animate-\[(width|height|top|left|padding|margin)" components/ app/
```

`transition-all` is a soft ban — it catches layout props by accident. Prefer named `transition-{colors,transform,opacity}`.

---

## BAN 5 — No `bg-card` shadow chrome

**Why**: This register's `--card` token is visually equal to `--background`. The whole point is that hierarchy comes from hairlines + whitespace + type, not surface elevation. If you reach for `shadow-sm` / `shadow-md` to make a card stand out, you've broken the Notebook voice.

**Instead**:
- Use `border border-border` to mark a surface
- Use `border-y border-border` for a strip (no left/right border, no shadow)
- Use a hairline divider (`<Separator />` or `border-b border-border pb-2`) for sectioning within a card

**Grep**:
```bash
grep -rE "shadow-(sm|md|lg|xl|2xl)\b" components/ app/
# Zero in app code. UI primitives that ship shadcn defaults are an allowed exception
# until you transplant the Notebook variants — at which point they should also be zero.
```

---

## BAN 6 — No `bg-white` / `bg-black`

**Why**: Raw `#fff` and `#000` break the cool off-white canvas and ink-blue foreground tokens. They read as un-themed, and they will not respond to any future token tweak.

**Instead**:
- Surface: `bg-card` or `bg-background`
- Switch thumb / form control surface: `bg-card` (the OKLCH near-white reads as "white" but stays on-family)
- Modal scrim / overlay: `bg-foreground/40` (warm sumi-feel, sits naturally on the canvas)
- Letterbox / video bars: `bg-foreground` or `bg-[oklch(14%_0.02_250)]`

**Grep**:
```bash
grep -rE "bg-white\b|bg-black\b" components/ app/
```

The only legitimate exception is `bg-white/0` or `bg-black/0` (transparent shorthands). Even then, prefer the named `bg-transparent`.

---

## BAN 7 — No raw Tailwind color literals

**Why**: `bg-emerald-100`, `text-red-500`, `bg-blue-50` and friends couple your component to a specific hue rather than to a *role*. They break dark mode (if you ever add it), they fight the OKLCH token family, and they read as un-considered.

**Instead**:

| Old literal                        | Notebook replacement                                          |
| ---------------------------------- | ------------------------------------------------------------- |
| `text-green-600` / `text-emerald-*`  | `text-primary` (positive / active)                           |
| `text-red-500` / `text-red-600`    | `text-destructive` (negative)                                 |
| `text-amber-600` / `text-yellow-*` | `text-foreground` (neutral warning) or `text-destructive`     |
| `bg-emerald-100 text-emerald-700`  | `border border-primary/40 bg-primary/5 text-primary`         |
| `bg-blue-100 text-blue-700`        | `border border-border bg-card text-foreground`               |
| `bg-amber-100 text-amber-700`      | `border border-border bg-muted text-foreground`              |
| `bg-purple-100 text-purple-700`    | `border border-border bg-muted text-foreground`              |
| `bg-gray-100 text-gray-600`        | `border border-border bg-muted text-muted-foreground`        |
| `bg-emerald-500` (dot indicator)   | `bg-primary`                                                  |
| `bg-red-500` (dot indicator)       | `bg-destructive`                                              |
| `bg-blue-500` (dot indicator)      | `bg-foreground` (neutral status dot)                          |

**Grep**:
```bash
grep -rE "(bg|text)-(emerald|green|amber|sky|purple|red|blue|gray|slate|zinc|neutral|yellow|orange|pink|rose|cyan|teal|indigo|violet|fuchsia|lime)-[0-9]+" components/ app/
# Zero hits in app code (after the migration). UI primitives that ship shadcn defaults
# are an allowed exception until they get the Notebook transplant.
```

**Migration tip**: a perl one-liner can replace most of these in bulk:

```bash
perl -i -pe '
  s|bg-emerald-100 text-emerald-700|border border-primary/40 bg-primary/5 text-primary|g;
  s|bg-amber-100 text-amber-700|border border-border bg-muted text-foreground|g;
  s|bg-blue-100 text-blue-700|border border-border bg-card text-foreground|g;
  s|bg-gray-100 text-gray-600|border border-border bg-muted text-muted-foreground|g;
  s|text-green-600|text-primary|g;
  s|text-red-(500|600)|text-destructive|g;
  s|text-emerald-(600|700)|text-primary|g;
  s|text-amber-600|text-foreground|g;
  s|bg-emerald-500\b|bg-primary|g;
  s|bg-red-500\b|bg-destructive|g;
  s|bg-blue-500\b|bg-foreground|g;
' $(grep -rlE "(bg|text)-(emerald|green|amber|red|blue)-[0-9]" components/ app/)
```

Run, grep again, fix any contextual mistakes by hand.

---

## BAN 8 — No icons in the metric strip

**Why**: A 5-cell metric strip is scanning surface. Icons next to numbers compete for attention with the numbers, slow the scan, and rarely add information (the label already tells you what the metric is).

**Instead**:
- Cells are label + figure + optional hint + optional delta. That's it.
- Icons belong on action buttons, nav rails, and inline content — never on a stat cell.
- If you genuinely need iconography to distinguish similar-looking metrics, you have too many cells in the strip. Split or reorder.

**Visual check**: take a screenshot of your dashboard and squint. If you can read the numbers, the strip is right. If iconography draws your eye first, remove the icons.

---

## BAN 9 — No glassmorphism (`backdrop-blur` on chrome)

**Why**: `backdrop-blur` reads as macOS-flavored chrome — Stripe Sigma, Linear, Apple's marketing site. It's gorgeous in product photography and miserable to read against a dense dashboard background. It also tanks scroll performance on lower-end hardware.

**Instead**:
- Sticky header: solid `bg-background border-b border-border`
- Dialog overlay: solid `bg-foreground/40` (warm sumi scrim)
- Popover / dropdown: solid `bg-popover border border-border`
- Tooltip: solid `bg-card border border-border`

**Grep**:
```bash
grep -rE "backdrop-blur" components/ app/
# Zero hits.
```

---

## BAN 10 — No layout-prop animation in chrome

**Why**: framer-motion is excellent for genuine orchestration (multi-step forms, modal entrances, drag-and-drop sequences). It's overkill for "fade the sidebar in" or "stagger the nav rows on mount." Chrome motion that re-runs on every navigation is a tax on every page load.

**Instead**:
- Sidebar / header / mobile-nav: CSS `transition-{colors,transform,opacity}` only, no framer-motion
- Page transitions: don't. Let Next.js render the page; the user wants the content, not the choreography
- Nav-active indicator: pure CSS pseudo-element underline or border, not `layoutId="activeNav"`
- Card hover lifts: `hover:bg-accent/30` or `hover:border-primary/40`, not `whileHover={{ y: -2 }}`

framer-motion stays as a dependency where it genuinely earns its keep — modal slide-in with stagger, dragging a draft into a queue, an animated step indicator on a multi-page form. Audit your imports periodically; if a component imports `motion` but only uses it for chrome, swap to CSS.

**Grep**:
```bash
grep -rlE "from ['\"](framer-motion|motion/react)['\"]" components/ app/ \
  | xargs grep -l "motion\.\(div\|header\|aside\|nav\|button\)\b" \
  | xargs grep -l "whileHover\|whileTap\|layoutId\|AnimatePresence"
# Any file that hits all three patterns is a chrome consumer of framer-motion — review.
```

---

## Enforcement workflow

A reliable enforcement loop:

1. **Pre-commit hook** runs the grep suite above; refuses commits with violations
2. **CI step** repeats the grep suite + a Playwright visual regression sweep
3. **Periodic audit** (monthly): a refactor agent runs the grep suite + reads any matching files, files PRs to fix drift
4. **PR review checklist** includes the 10 BANs as a one-line yes/no

The greps are fast — they should fit in the same `lint` step as ESLint without noticeable overhead. Build them into `pnpm lint:design` and call it from `pnpm lint`.
