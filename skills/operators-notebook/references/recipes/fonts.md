# Recipe: Font Setup (`lib/fonts.ts` + `app/layout.tsx`)

Three-font stack via `next/font/google`. Each font has one job; never mix roles.

## `lib/fonts.ts`

```ts
import { Bricolage_Grotesque, Sometype_Mono, Noto_Sans_SC } from "next/font/google";

// Body — language-aware. Swap to Noto_Sans_JP for Japanese, Inter for Latin-first.
export const fontBody = Noto_Sans_SC({
  subsets: ["latin"],
  weight: ["400", "500", "600", "700"],
  variable: "--font-body",
  display: "swap",
});

// Display — Bricolage Grotesque (variable). Mini-cap labels, page headers,
// metric figures. Tight tracking at display sizes.
export const fontDisplay = Bricolage_Grotesque({
  subsets: ["latin"],
  weight: ["400", "500", "600", "700"],
  variable: "--font-display",
  display: "swap",
});

// Mono — Sometype Mono. IDs, timestamps, prices, OTP slots, pagination.
// Reserved for tabular contexts. Never running prose.
export const fontMono = Sometype_Mono({
  subsets: ["latin"],
  weight: ["400", "500", "600", "700"],
  variable: "--font-mono",
  display: "swap",
});
```

## Language variants

Pick the body font that matches your user-facing language. The display and mono stay the same.

### Simplified Chinese (zh-CN)

```ts
import { Noto_Sans_SC } from "next/font/google";

export const fontBody = Noto_Sans_SC({
  subsets: ["latin"],
  weight: ["400", "500", "600", "700"],
  variable: "--font-body",
  display: "swap",
});
```

Set `<html lang="zh-CN">` in `layout.tsx`.

### Japanese (ja)

```ts
import { Noto_Sans_JP } from "next/font/google";

export const fontBody = Noto_Sans_JP({
  subsets: ["latin"],
  weight: ["400", "500", "600", "700"],
  variable: "--font-body",
  display: "swap",
});
```

Set `<html lang="ja">`. Note: do not pair Bricolage Grotesque + Zen Old Mincho (mincho headlines clash with the editorial Latin display face). The mixed-script display story is "Bricolage everywhere, even for JP strings."

### Latin-first (en, fr, de, etc.)

```ts
import { Inter } from "next/font/google";

export const fontBody = Inter({
  subsets: ["latin"],
  weight: ["400", "500", "600", "700"],
  variable: "--font-body",
  display: "swap",
});
```

Set `<html lang="en">` (or appropriate locale).

## `app/layout.tsx`

```tsx
import type { Metadata } from "next";
import { fontBody, fontDisplay, fontMono } from "@/lib/fonts";
import "./globals.css";

// Provider imports — adjust to your stack.
import { QueryProvider } from "@/providers/query-provider";
import { AuthProvider } from "@/components/auth-provider";
import { Toaster } from "@/components/ui/toaster";

export const metadata: Metadata = {
  title: "<Your App>",
  description: "<Your description>",
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="zh-CN" className={`${fontBody.variable} ${fontDisplay.variable} ${fontMono.variable}`}>
      <body className="font-sans antialiased">
        <QueryProvider>
          <AuthProvider>
            {children}
            <Toaster />
          </AuthProvider>
        </QueryProvider>
      </body>
    </html>
  );
}
```

Critical bits:

- `<html lang>` matches the body font's script
- All three font `.variable` strings on the `<html>` className (not body)
- `body` className includes `font-sans` so the default body font kicks in
- **No `next-themes` ThemeProvider** — this register is light-only

## Usage in components

Tailwind v4 picks up the three font variables from `@theme inline` in `globals.css`:

```tsx
<h1 className="font-display text-2xl font-semibold tracking-tight">Title</h1>
<p className="font-sans text-sm text-muted-foreground">Body text.</p>
<span className="font-mono text-xs tabular-nums">YJ-1234567</span>
```

`font-sans` is the body font (alias from `@theme inline` mapping `--font-sans` → `--font-body`). Use `font-display` and `font-mono` for the other two roles.

## Removing Geist / next-themes

When migrating from default shadcn:

```bash
pnpm remove geist next-themes tw-animate-css
```

In `layout.tsx`, replace:

```tsx
// OLD
import { GeistSans } from "geist/font/sans";
import { GeistMono } from "geist/font/mono";
import { ThemeProvider } from "next-themes";

<html className={`${GeistSans.variable} ${GeistMono.variable}`}>
  <body>
    <ThemeProvider attribute="class" defaultTheme="system">
      {children}
    </ThemeProvider>
  </body>
</html>
```

With:

```tsx
// NEW
import { fontBody, fontDisplay, fontMono } from "@/lib/fonts";

<html lang="zh-CN" className={`${fontBody.variable} ${fontDisplay.variable} ${fontMono.variable}`}>
  <body className="font-sans antialiased">{children}</body>
</html>
```

`next-themes` is removed because the register has no dark mode. If you must support dark mode later, that's a separate token contract — do not just add a `.dark` block to the OKLCH values without testing every screen.
