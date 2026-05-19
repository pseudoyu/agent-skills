# Recipe: Big-Bang Migration Playbook

Migrating an existing default-shadcn admin app to this register. Order matters — token drift propagates fast, so do the foundation in one pass, then walk the routes.

This assumes:
- Next.js 13+ App Router
- Tailwind v4 (or v3 with `theme.extend.colors` adapted)
- shadcn/ui in `components/ui/`
- A real working app with data flow you want to preserve

The migration is **visual layer only**. No oRPC calls, React Query hooks, form schemas, mutation callbacks, permission checks, or business logic should move. If a step asks you to touch data flow, that's not this migration.

## Pre-flight

Take inventory:

```bash
# Routes
ls app/         # or pages/

# Feature components
ls components/

# UI primitives
ls components/ui/

# Existing color literals — this is your "before" baseline
grep -rE "(bg|text)-(emerald|green|amber|sky|purple|red|blue|gray|slate|zinc|neutral|yellow)-[0-9]" components/ app/ | wc -l

# Existing shadow / pillowy radius
grep -rE "shadow-(sm|md|lg|xl|2xl)|rounded-(xl|2xl|3xl)" components/ app/ | wc -l

# framer-motion chrome consumers
grep -rl "from ['\"]framer-motion['\"]" components/sidebar* components/header* components/page-* | wc -l
```

These numbers should drop to zero across the migration.

Confirm with the user:
1. **Body font** — what language is the UI? Pick `Noto_Sans_SC` / `Noto_Sans_JP` / `Inter`.
2. **Drop dark mode?** — recommended yes for this register. If they need dark mode, plan a separate dark-token contract; don't just add a `.dark` block.
3. **Migration strategy** — big-bang (recommended for ≤20 routes) or staged (one route at a time)?
4. **Verification** — Playwright route-by-route screenshot + console.error check?

## Phase 1 — Foundation (tokens + fonts + layout)

These edit the foundation. Do them in one commit; don't ship without all three.

1. **Rewrite `app/globals.css`** with the OKLCH token block from [`tokens.md`](tokens.md). Delete:
   - Existing `:root` color block
   - Entire `.dark` block
   - `@custom-variant dark (&:where(.dark, .dark *))`
   - `@import "tw-animate-css"`
   - Sidebar-specific tokens (`--sidebar-*`)
   - Any `text-shadow:` declarations
   - Geist `font-family` declarations

2. **Create `lib/fonts.ts`** with three `next/font/google` exports per [`fonts.md`](fonts.md).

3. **Edit `app/layout.tsx`**:
   - Drop `GeistSans` / `GeistMono` imports
   - Wire `fontBody.variable`, `fontDisplay.variable`, `fontMono.variable` on `<html>`
   - Drop `next-themes` `ThemeProvider`
   - Set `<html lang>` correctly
   - Keep all data providers (QueryProvider, AuthProvider, etc.)

4. **Update `package.json`**:
   ```bash
   pnpm remove geist next-themes tw-animate-css
   ```
   Keep `framer-motion` for now if you have non-chrome consumers (modals, multi-step forms).

5. **Verify**:
   ```bash
   pnpm typecheck
   pnpm build
   ```
   The build should compile but the app will look broken (default shadcn primitives still expect the old token names). That's expected — proceed to Phase 2.

## Phase 2 — shadcn/ui primitive transplant

Replace each primitive in `components/ui/` with the Notebook-tuned variant from [`primitives.md`](primitives.md).

Priority order (do these first — they cover ~80% of surface):
1. `button.tsx`
2. `card.tsx`
3. `input.tsx`
4. `dialog.tsx`
5. `badge.tsx`

Then the rest:
6. `label.tsx`, `textarea.tsx`, `form.tsx`
7. `tabs.tsx`, `select.tsx`, `dropdown-menu.tsx`, `tooltip.tsx`
8. `switch.tsx`, `checkbox.tsx`
9. `scroll-area.tsx`, `separator.tsx`, `skeleton.tsx`, `avatar.tsx`
10. `table.tsx`
11. `toast.tsx`, `toaster.tsx`

Bulk scrub for the rest:

```bash
perl -i -pe '
  s|shadow-(sm|md|lg|xl|2xl)\b||g;
  s|rounded-(xl|2xl|3xl)\b|rounded-md|g;
  s|ring-2 ring-ring|ring-1 ring-ring|g;
  s|bg-white\b|bg-card|g;
  s|backdrop-blur-[a-z]+\b||g;
' components/ui/*.tsx
```

**Add missing primitives**: if the admin needs `table`, `tabs`, `select`, `dropdown-menu`, `tooltip`, `switch`, `checkbox`, `scroll-area`, `separator`, `skeleton`, `avatar` but they don't exist in `components/ui/`, install them via `npx shadcn-ui@latest add` and then run the perl scrub above.

Verify:
```bash
pnpm typecheck
pnpm build
```

The app should now compile and look "register-shaped" but feature pages still have color literals and old chrome.

## Phase 3 — Shared Notebook primitives

Create `components/shared/`:
- `page-header.tsx` — from [`shared-primitives.md`](shared-primitives.md)
- `section-label.tsx`
- `stat-card.tsx`
- `empty-state.tsx`

Create `lib/chart-typography.ts` — from [`charts.md`](charts.md).

These are additions, not replacements. They sit alongside existing components until feature pages migrate.

## Phase 4 — Chrome (sidebar, header, mobile-nav, layout)

Rewrite chrome per [`chrome.md`](chrome.md):

1. **`components/sidebar.tsx`** — rewrite following the recipe. Preserve all nav items, role gates, `usePathname` exact-vs-prefix matching, brand block. Strip every framer-motion import, `staggerContainer`, `whileHover`, `layoutId="activeNav"`.

2. **`components/header.tsx`** — rewrite. Solid `bg-background`, mono displayName chip, ghost logout, optional mobile-nav toggle slot. Strip every `<motion.header>` / `<AnimatePresence>`.

3. **`components/mobile-nav.tsx`** — new file. CSS slide-in panel.

4. **`app/(dashboard)/layout.tsx`** — switch to `md:pl-60` shell pattern with sidebar + mobile-nav + header + main.

5. **Delete**:
   - `components/page-transition.tsx`
   - `components/motion/` (if it only re-exports framer-motion variants for chrome — keep if it has modal orchestration variants you still use)
   - `app/globals.css` keyframes that were used only for chrome stagger animations

Verify:
```bash
pnpm typecheck
pnpm build
pnpm dev   # walk through the dashboard, check chrome looks right
```

## Phase 5 — Auth pages

Rewrite `app/login/page.tsx` and `components/SmsLoginCard.tsx` (or your equivalent) per [`login.md`](login.md).

**Critical**: keep every oRPC call, validation schema, mutation callback, error handler, toast call. Only swap the visual layer — Inputs, Buttons, Labels, layout structure.

## Phase 6 — Page-by-page chrome swap

For each route, the surgical swap pattern:

| Old                                             | New                                                                |
| ----------------------------------------------- | ------------------------------------------------------------------ |
| `<Card className="border bg-card shadow-sm">`   | `<Card>` (now Notebook-styled by default)                          |
| `<h1 className="text-3xl font-semibold">`       | `<PageHeader title={...} />`                                       |
| Stat grid: `<div className="grid grid-cols-3 gap-6">` with cards | Hairline strip per [`shared-primitives.md`](shared-primitives.md) |
| Section header: `<h2 className="text-lg font-semibold mb-4">` | `<SectionLabel>` mini-cap                                          |
| Empty state: random centered text + icon       | `<EmptyState>` primitive                                           |
| Tables: shadcn defaults                         | Notebook `<Table>` (head = mini-cap)                              |
| Tabs: shadcn defaults                           | Notebook `<Tabs>` (underline active state)                         |
| Modals: shadcn defaults                         | Notebook `<Dialog>` (no backdrop-blur)                             |
| Numeric values: `<span className="font-bold">`  | `<span className="font-mono tabular-nums">`                        |
| Dates: ad-hoc `toLocaleString()`                | Shared `formatDate` / `formatDateTime` utility                     |
| `bg-emerald-100 text-emerald-700` badge         | `border-primary/40 bg-primary/5 text-primary`                      |
| `text-green-600` positive delta                 | `text-primary`                                                     |
| `text-red-500` negative delta                   | `text-destructive`                                                 |
| `bg-blue-50` info strip                         | `border-border bg-muted text-foreground`                           |

**Do not touch**:
- `useQuery` calls or any React Query hook
- oRPC client invocations
- Form submit handlers, mutation callbacks
- Permission checks, role gates
- Computed display data (selectors, memoized derivations)
- Modal open/close state machines

If you find yourself changing data flow to "fit the new design," stop. The migration is visual layer only.

Walk every route. After each:
```bash
# Visual check
pnpm dev   # then visit the route

# Compile check
pnpm typecheck

# Lint check
pnpm lint
```

## Phase 7 — Cleanup + verification

### Grep guards

```bash
# Color literals — should be zero in app code
grep -rE "(bg|text)-(emerald|green|amber|sky|purple|red|blue|gray|slate|zinc|neutral|yellow|orange|pink|rose|cyan|teal|indigo|violet|fuchsia|lime)-[0-9]+" components/ app/

# BAN 1 — side-stripe accents
grep -rE "border-l-[2-9]\b|border-r-[2-9]\b" components/ app/

# BAN 2 — gradient text
grep -rE "bg-clip-text" components/ app/

# BAN 5 — shadow chrome
grep -rE "shadow-(sm|md|lg|xl|2xl)\b" components/ app/ | grep -v "components/ui/"

# BAN 6 — bg-white / bg-black
grep -rE "bg-(white|black)\b" components/ app/

# BAN 9 — glassmorphism
grep -rE "backdrop-blur" components/ app/

# Pillowy radius
grep -rE "rounded-(xl|2xl|3xl)\b" components/ app/ | grep -v "components/ui/"

# Old display type
grep -rE "text-3xl|text-4xl" components/ app/

# Old motion patterns
grep -rE "motion\.\w+|whileHover|whileTap|layoutId|AnimatePresence" components/ app/ | grep -v "modal\|form"
```

All of these should return zero hits (or only hits in `components/ui/` if you haven't finished transplanting all primitives — those are tracked in Phase 2).

### Build + typecheck

```bash
pnpm typecheck && pnpm lint && pnpm build
```

All green before commit.

### Playwright sweep

For each route, automate:

```ts
test("operators-notebook chrome — /finance", async ({ page }) => {
  await page.goto("/finance");
  await page.waitForLoadState("networkidle");

  // Visual baseline
  await expect(page).toHaveScreenshot("finance.png");

  // No React warnings
  const messages = await page.evaluate(() => (window as any).__consoleMessages || []);
  expect(messages.filter((m: any) => m.type === "error")).toHaveLength(0);

  // Notebook chrome assertions
  await expect(page.locator("aside")).toHaveCSS("border-right-width", "1px");
  await expect(page.locator("header h1")).toHaveCSS("font-family", /Bricolage/);
  await expect(page.locator(".animate-pulse")).toHaveCount(0); // No legacy skeleton chrome
});
```

Repeat for every route. Any visual regression that's behavioral (data missing, button doesn't work) gets fixed in place; any that's purely visual either gets fixed or accepted into the new baseline.

### Reduced-motion check

```ts
test("operators-notebook respects prefers-reduced-motion", async ({ page }) => {
  await page.emulateMedia({ reducedMotion: "reduce" });
  await page.goto("/");
  // No visible animation jitter on first paint
  await expect(page).toHaveScreenshot("home-reduced-motion.png");
});
```

### Single commit

```bash
git add -A
git commit -m "feat(admin): operator's notebook chrome migration

Migrates the admin app from default shadcn SaaS to the Operator's Notebook
register: OKLCH ink-blue tokens, Bricolage Grotesque display, hairline
dividers in place of card shadow, ink-dot active nav.

All oRPC calls, React Query hooks, form schemas, and permission checks
unchanged. Verification via Playwright across all routes."
```

If the migration ends up touching > 50 files, that's normal for a register-level change — it's the entire visual layer.

## Common gotchas

- **Tailwind v4 + OKLCH** picks up the tokens via `@theme inline`. If you're on v3, you'll need to put the colors in `tailwind.config.ts` and convert OKLCH to HSL or use `hsl(var(--token))` everywhere.
- **`next-themes` removal** can break any component that still imports `useTheme()`. Grep for it before removing the package: `grep -r "useTheme\|next-themes"`.
- **Geist removal** can break any component that imports `GeistSans` directly. Grep: `grep -r "from ['\"]geist"`.
- **shadcn primitive transplant** may break component imports that destructure specific variants — re-check after each primitive swap.
- **Date formatters** that used to render with default `toLocaleString` may now look stylistically wrong (no mono / not tabular). Audit and switch to a shared formatter that returns `font-mono tabular-nums`-friendly strings.
- **OTP / verification code inputs** often have custom slot components from libraries (`input-otp`). The recipe in [`login.md`](login.md) uses a single `<Input>` with `font-mono tracking-[0.25em]` — simpler and matches the register. If you keep an OTP library, scrub its shipped styling against the BANs.
- **Charts** mid-migration can look mismatched (some pages have flat editorial, others still have rounded SaaS bars). Walk every chart in the same pass as Phase 3 — don't leave any with the old style.
