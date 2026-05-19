# Recipe: Chrome — Sidebar, Header, Mobile Nav, Layout Shell

The chrome carries the register's voice on every route. Get this right and feature pages can be sloppy and still read coherent; get this wrong and even disciplined feature pages will feel off.

All chrome is **CSS-transition only**. No framer-motion in sidebar, header, or mobile-nav. (BAN 10.)

## `components/sidebar.tsx`

```tsx
"use client";

import Link from "next/link";
import { usePathname } from "next/navigation";
import {
  LayoutDashboard,
  Users,
  Store,
  Gamepad2,
  CalendarDays,
  Wallet,
  Shield,
  History,
  Settings,
  // ...your icons
} from "lucide-react";
import { cn } from "@/lib/utils";

interface NavItem {
  href: string;
  label: string;
  icon: React.ComponentType<{ className?: string }>;
  /** Optional role gate */
  roles?: string[];
}

const NAV_ITEMS: NavItem[] = [
  { href: "/", label: "总览", icon: LayoutDashboard },
  { href: "/players", label: "玩家", icon: Users },
  { href: "/stores", label: "门店", icon: Store },
  { href: "/games", label: "赛事", icon: Gamepad2 },
  { href: "/appointments", label: "预约", icon: CalendarDays },
  { href: "/finance", label: "财务", icon: Wallet },
  { href: "/auth", label: "审核", icon: Shield, roles: ["admin", "operator"] },
  { href: "/auth/audit", label: "审计", icon: History, roles: ["admin"] },
  { href: "/settings", label: "设置", icon: Settings, roles: ["admin"] },
];

export function Sidebar({ userRole }: { userRole?: string }) {
  const pathname = usePathname();
  const visibleItems = NAV_ITEMS.filter((item) => !item.roles || (userRole && item.roles.includes(userRole)));

  return (
    <aside className="fixed inset-y-0 hidden w-60 flex-col border-r border-border bg-background md:flex">
      {/* Brand block */}
      <div className="flex items-center gap-2 border-b border-border px-4 py-4">
        <LayoutDashboard className="h-5 w-5 text-primary" />
        <span className="font-display text-base font-semibold tracking-tight text-foreground">
          游酒YoJoe宇宙
        </span>
      </div>

      {/* Nav rows */}
      <nav className="flex-1 space-y-1 px-2 py-4">
        {visibleItems.map((item) => {
          // Exact match for "/", prefix match for everything else
          const isActive =
            item.href === "/"
              ? pathname === "/"
              : pathname === item.href || pathname.startsWith(item.href + "/");

          return (
            <Link
              key={item.href}
              href={item.href}
              className={cn(
                "flex items-center gap-2 rounded-sm px-3 py-2 text-sm transition-colors hover:bg-accent/30",
                isActive ? "bg-accent/55 text-foreground" : "text-muted-foreground",
              )}
            >
              {/* Reason: ink-dot active marker (BAN 1 forbids border-l accent stripe) */}
              <span
                className={cn(
                  "h-1.5 w-1.5 shrink-0 rounded-full transition-colors",
                  isActive ? "bg-primary" : "bg-transparent",
                )}
              />
              <item.icon className="h-4 w-4 shrink-0" />
              <span className="truncate">{item.label}</span>
            </Link>
          );
        })}
      </nav>
    </aside>
  );
}
```

Key decisions:

- 60-wide rail (`w-60` = 240px). Wide enough for two-character + label, narrow enough to give pages real estate.
- Single `border-r border-border` hairline on the right edge — that's the rail's only edge.
- Active state: `bg-accent/55` row tint + ink-dot. No side stripe.
- No `framer-motion`, no `layoutId="activeNav"`, no `whileHover={{ x: 4 }}`. Plain CSS transitions.
- Role filter happens client-side here; do the auth check server-side too.

## `components/header.tsx`

```tsx
"use client";

import { useAuth } from "@/components/auth-provider";
import { Button } from "@/components/ui/button";
import { Menu } from "lucide-react";

interface HeaderProps {
  onMobileNavOpen?: () => void;
}

export function Header({ onMobileNavOpen }: HeaderProps) {
  const { user, logout } = useAuth();

  return (
    <header className="sticky top-0 z-30 flex h-14 items-center justify-between border-b border-border bg-background px-4 md:px-6">
      <div className="flex items-center gap-3">
        {onMobileNavOpen && (
          <Button
            variant="ghost"
            size="icon-sm"
            onClick={onMobileNavOpen}
            className="md:hidden"
            aria-label="打开导航"
          >
            <Menu className="h-4 w-4" />
          </Button>
        )}
      </div>
      <div className="flex items-center gap-3">
        {user && (
          <span className="font-mono text-xs tabular-nums text-muted-foreground">
            {user.displayName}
          </span>
        )}
        <Button variant="ghost" size="sm" onClick={logout}>
          退出
        </Button>
      </div>
    </header>
  );
}
```

Key decisions:

- Solid `bg-background`, no `backdrop-blur`, no `bg-card/50`. (BAN 9.)
- `border-b border-border` hairline carries the seam with the page body.
- User chip uses `font-mono tabular-nums` — fits the operator-ledger voice.
- Logout is `variant="ghost" size="sm"` — quiet, not a primary CTA.
- Optional mobile-nav toggle slot for the responsive shell.

## `components/mobile-nav.tsx`

CSS slide-in panel, no framer-motion. Visible only on `< md` viewports.

```tsx
"use client";

import { useEffect } from "react";
import Link from "next/link";
import { usePathname } from "next/navigation";
import { X } from "lucide-react";
import { cn } from "@/lib/utils";
import { Button } from "@/components/ui/button";

interface MobileNavProps {
  open: boolean;
  onClose: () => void;
  items: { href: string; label: string; icon: React.ComponentType<{ className?: string }> }[];
}

export function MobileNav({ open, onClose, items }: MobileNavProps) {
  const pathname = usePathname();

  // Close on route change
  useEffect(() => {
    onClose();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [pathname]);

  return (
    <>
      {/* Scrim — solid sumi, no blur */}
      <div
        className={cn(
          "fixed inset-0 z-40 bg-foreground/40 transition-opacity duration-200 md:hidden",
          open ? "opacity-100" : "pointer-events-none opacity-0",
        )}
        onClick={onClose}
        aria-hidden
      />

      {/* Slide-in panel */}
      <aside
        className={cn(
          "fixed inset-y-0 left-0 z-50 flex w-60 flex-col border-r border-border bg-background transition-transform duration-200 ease-out md:hidden",
          open ? "translate-x-0" : "-translate-x-full",
        )}
      >
        <div className="flex items-center justify-between border-b border-border px-4 py-4">
          <span className="font-display text-base font-semibold tracking-tight">导航</span>
          <Button variant="ghost" size="icon-sm" onClick={onClose} aria-label="关闭导航">
            <X className="h-4 w-4" />
          </Button>
        </div>
        <nav className="flex-1 space-y-1 px-2 py-4">
          {items.map((item) => {
            const isActive =
              item.href === "/"
                ? pathname === "/"
                : pathname === item.href || pathname.startsWith(item.href + "/");
            return (
              <Link
                key={item.href}
                href={item.href}
                className={cn(
                  "flex items-center gap-2 rounded-sm px-3 py-2 text-sm transition-colors hover:bg-accent/30",
                  isActive ? "bg-accent/55 text-foreground" : "text-muted-foreground",
                )}
              >
                <span
                  className={cn(
                    "h-1.5 w-1.5 shrink-0 rounded-full",
                    isActive ? "bg-primary" : "bg-transparent",
                  )}
                />
                <item.icon className="h-4 w-4 shrink-0" />
                <span>{item.label}</span>
              </Link>
            );
          })}
        </nav>
      </aside>
    </>
  );
}
```

Key decisions:

- `transition-transform` + `translate-x` — transform + opacity only (BAN 4 forbids layout-prop animation)
- Scrim is `bg-foreground/40`, no `backdrop-blur`
- Same nav vocabulary as desktop sidebar (`bg-accent/55` + ink-dot)
- Hidden on `md+` via `md:hidden`

## `app/(dashboard)/layout.tsx`

```tsx
"use client";

import { useState } from "react";
import { Sidebar } from "@/components/sidebar";
import { Header } from "@/components/header";
import { MobileNav } from "@/components/mobile-nav";
import { useAuth } from "@/components/auth-provider";

const NAV_ITEMS = [/* same as in sidebar */];

export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  const [mobileNavOpen, setMobileNavOpen] = useState(false);
  const { user } = useAuth();

  const visibleItems = NAV_ITEMS.filter(
    (item) => !item.roles || (user?.role && item.roles.includes(user.role)),
  );

  return (
    <div className="min-h-screen bg-background">
      <Sidebar userRole={user?.role} />
      <MobileNav open={mobileNavOpen} onClose={() => setMobileNavOpen(false)} items={visibleItems} />
      <div className="md:pl-60">
        <Header onMobileNavOpen={() => setMobileNavOpen(true)} />
        <main className="p-4 md:p-6 lg:p-8">{children}</main>
      </div>
    </div>
  );
}
```

Key decisions:

- `md:pl-60` shifts the page content to clear the fixed sidebar
- Header sits inside the content shift so it spans the full visual width on desktop
- `main` padding ramps up with viewport (`p-4 → md:p-6 → lg:p-8`)
- Mobile-nav state lives in the layout, not in the header or sidebar

## `components/page-layout.tsx` (optional convenience wrapper)

If you want every page to use `PageHeader` + a consistent max-width, wrap it:

```tsx
import { PageHeader } from "@/components/shared/page-header";
import { cn } from "@/lib/utils";

interface PageLayoutProps {
  title: string;
  eyebrow?: string;
  description?: string;
  actions?: React.ReactNode;
  maxWidth?: "default" | "narrow" | "wide" | "full";
  children: React.ReactNode;
}

const MAX_WIDTHS = {
  default: "max-w-7xl",
  narrow: "max-w-3xl",
  wide: "max-w-[90rem]",
  full: "max-w-none",
} as const;

export function PageLayout({ title, eyebrow, description, actions, maxWidth = "default", children }: PageLayoutProps) {
  return (
    <div className={cn("mx-auto w-full", MAX_WIDTHS[maxWidth])}>
      <PageHeader eyebrow={eyebrow} title={title} description={description} actions={actions} />
      <div className="mt-6 space-y-6">{children}</div>
    </div>
  );
}
```

Usage in a page:

```tsx
export default function FinancePage() {
  return (
    <PageLayout
      eyebrow="财务管理"
      title="交易流水"
      description="近 90 天的财务记录"
      actions={<DateRangePicker />}
    >
      <FinanceList />
    </PageLayout>
  );
}
```

## What to delete

When migrating from default shadcn admin:

- `components/page-transition.tsx` — CSS-only motion makes this obsolete
- `components/motion/` — if it only re-exports framer-motion variants for chrome
- Any `whileHover={{ x: 4 }}` / `whileTap={{ scale: 0.98 }}` on nav rows
- Any `layoutId="activeNavIndicator"` on the active nav marker
- Any `<motion.header>` / `<AnimatePresence>` wrapping the sticky header
- `staggerContainer` / `staggerItem` variants imported anywhere
- The `next-themes` `ThemeProvider` — register has no dark mode
