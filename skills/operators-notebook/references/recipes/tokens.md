# Recipe: Token Contract (`app/globals.css`)

Drop-in `globals.css` for a Next.js + Tailwind v4 project. Replace your existing token block with this; do not merge them.

## Full file

```css
@import "tailwindcss";

@theme inline {
  --color-background:           var(--background);
  --color-foreground:           var(--foreground);
  --color-card:                 var(--card);
  --color-card-foreground:      var(--card-foreground);
  --color-popover:              var(--popover);
  --color-popover-foreground:   var(--popover-foreground);
  --color-primary:              var(--primary);
  --color-primary-foreground:   var(--primary-foreground);
  --color-secondary:            var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted:                var(--muted);
  --color-muted-foreground:     var(--muted-foreground);
  --color-accent:               var(--accent);
  --color-accent-foreground:    var(--accent-foreground);
  --color-destructive:          var(--destructive);
  --color-destructive-foreground: var(--destructive-foreground);
  --color-border:               var(--border);
  --color-input:                var(--input);
  --color-ring:                 var(--ring);
  --color-chart-1:              var(--chart-1);
  --color-chart-2:              var(--chart-2);
  --color-chart-3:              var(--chart-3);
  --color-chart-4:              var(--chart-4);
  --color-chart-5:              var(--chart-5);

  --radius-sm:                  calc(var(--radius) - 4px);
  --radius-md:                  calc(var(--radius) - 2px);
  --radius-lg:                  var(--radius);
  --radius-xl:                  calc(var(--radius) + 4px);

  --font-sans:                  var(--font-body);
  --font-display:               var(--font-display);
  --font-mono:                  var(--font-mono);
}

:root {
  /* Operator's Notebook — OKLCH h~250 family.
   * Light mode canonical. Do not add a `.dark` block to this register;
   * dark mode is a separate token contract that needs per-screen testing. */
  --background:             oklch(98% 0.003 250);
  --foreground:             oklch(22% 0.020 250);
  --card:                   oklch(99% 0.002 250);
  --card-foreground:        oklch(22% 0.020 250);
  --popover:                oklch(99% 0.002 250);
  --popover-foreground:     oklch(22% 0.020 250);
  --primary:                oklch(35% 0.040 250);
  --primary-foreground:     oklch(98% 0.003 250);
  --secondary:              oklch(94% 0.004 250);
  --secondary-foreground:   oklch(28% 0.020 250);
  --muted:                  oklch(96% 0.003 250);
  --muted-foreground:       oklch(52% 0.010 250);
  --accent:                 oklch(92% 0.025 250);
  --accent-foreground:      oklch(28% 0.030 250);
  --destructive:            oklch(52% 0.140 28);
  --destructive-foreground: oklch(98% 0.003 250);
  --border:                 oklch(88% 0.004 250);
  --input:                  oklch(88% 0.004 250);
  --ring:                   oklch(35% 0.040 250);
  --radius:                 0.25rem;

  /* Chart series — hue-locked, intentionally desaturated.
   * Cycle 1→5 for multi-series; never reach for a saturated qualitative palette. */
  --chart-1: oklch(42% 0.080 250);
  --chart-2: oklch(55% 0.060 220);
  --chart-3: oklch(62% 0.050 280);
  --chart-4: oklch(48% 0.070 200);
  --chart-5: oklch(58% 0.055 260);
}

@layer base {
  * {
    @apply border-border;
  }

  html {
    font-family: var(--font-body), system-ui, -apple-system, "Segoe UI", sans-serif;
  }

  body {
    @apply bg-background text-foreground;
    font-feature-settings: "tnum";
  }

  /* Tabular figures globally for column-aligned numerics.
   * Override per-element in prose contexts if needed. */
}

/* Notebook pulse — used on the single live-update indicator in the header.
 * Per motif scarcity: ONE pulse-dot per viewport, max. */
@keyframes notebook-pulse {
  0%, 100% { opacity: 0.45; transform: scale(1); }
  50%      { opacity: 1;    transform: scale(1.15); }
}

.notebook-pulse {
  animation: notebook-pulse 2.4s ease-in-out infinite;
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }

  .notebook-pulse {
    animation: none;
  }
}

/* Performance utility — large lists / virtualized tables.
 * Lets the browser skip rendering off-screen rows. */
.content-visibility-auto {
  content-visibility: auto;
  contain-intrinsic-size: 0 200px;
}
```

## What to delete from your old `globals.css`

When migrating from default shadcn:

- The entire `.dark` block (this register is light-only)
- `@custom-variant dark (&:where(.dark, .dark *))`
- The `tw-animate-css` import (`@import "tw-animate-css"`) — not needed
- Sidebar-specific tokens (`--sidebar-background`, `--sidebar-foreground`, etc.) — the register uses the same tokens as everything else
- Any `text-shadow:` declarations on body / nav
- Geist-specific `font-family` declarations (replaced by `--font-body`)

## What to delete from your old `package.json`

- `geist` (replaced by Bricolage Grotesque + body font + Sometype Mono via `next/font/google`)
- `tw-animate-css`
- `next-themes` (no dark mode)

Keep `framer-motion` for now if you still use it in genuine modal orchestration. Audit and remove if all consumers are chrome-only.

## Tailwind v4 vs v3 note

The `@theme inline` block above is Tailwind v4 syntax. For v3, you would put these in `tailwind.config.ts` under `theme.extend.colors` and use HSL or hex. The OKLCH values still work in v3 if you wrap them in `hsl()` — but v4 is recommended for any new project shipping this register.
