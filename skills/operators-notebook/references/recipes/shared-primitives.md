# Recipe: Shared Notebook Primitives

Four small components that encode the register's discipline. Build these once in `components/shared/`; reach for them on every page instead of re-inventing card titles and metric grids.

## `page-header.tsx`

```tsx
import { cn } from "@/lib/utils";

interface PageHeaderProps {
  /** Optional mini-cap eyebrow above the title (e.g., "Operator" / "管理后台") */
  eyebrow?: string;
  /** Main page title — rendered in Bricolage display face */
  title: string;
  /** Optional one-line description below the title */
  description?: string;
  /** Optional right-aligned actions slot (buttons, filters) */
  actions?: React.ReactNode;
  className?: string;
}

export function PageHeader({ eyebrow, title, description, actions, className }: PageHeaderProps) {
  return (
    <header className={cn("border-b border-border pb-4", className)}>
      <div className="flex items-start justify-between gap-4">
        <div className="min-w-0 space-y-1">
          {eyebrow && (
            <p className="font-display text-[11px] font-medium uppercase tracking-[0.1em] text-muted-foreground">
              {eyebrow}
            </p>
          )}
          <h1 className="font-display text-2xl font-semibold tracking-tight text-foreground">
            {title}
          </h1>
          {description && (
            <p className="text-sm text-muted-foreground">{description}</p>
          )}
        </div>
        {actions && <div className="flex shrink-0 items-center gap-2">{actions}</div>}
      </div>
    </header>
  );
}
```

Usage:

```tsx
<PageHeader
  eyebrow="财务管理"
  title="交易流水"
  description="按门店、商品、状态筛选近 90 天的财务记录。"
  actions={
    <>
      <DateRangePicker />
      <Button variant="outline" size="sm">导出</Button>
    </>
  }
/>
```

## `section-label.tsx`

```tsx
import { cn } from "@/lib/utils";

interface SectionLabelProps {
  children: React.ReactNode;
  /** Set to false to skip the bottom hairline (e.g., when nested in a card that already has structure) */
  divider?: boolean;
  className?: string;
}

export function SectionLabel({ children, divider = true, className }: SectionLabelProps) {
  return (
    <h2
      className={cn(
        "font-display text-[11px] font-medium uppercase tracking-[0.1em] text-muted-foreground",
        divider && "border-b border-border pb-2",
        className,
      )}
    >
      {children}
    </h2>
  );
}
```

Usage:

```tsx
<div className="space-y-3">
  <SectionLabel>预约日程</SectionLabel>
  <AppointmentCalendar />
</div>
```

## `stat-card.tsx`

The cell of the hairline metric strip. Never used standalone — always inside a grid wrapper.

```tsx
import { cn } from "@/lib/utils";
import { Loader2, TrendingDown, TrendingUp } from "lucide-react";

interface StatChange {
  /** Absolute difference vs previous period */
  absoluteDiff: number;
  /** Percent change vs previous period */
  percent: number;
  isIncrease: boolean;
}

interface StatCardProps {
  label: string;
  /** Numeric value; will render with tabular-nums. Pre-format strings for currency. */
  value: number | string | undefined;
  /** Optional one-line hint below the figure */
  hint?: string;
  /** Optional delta chip — shown when present */
  change?: StatChange | null;
  /** Set true for the single 60/30/10 accent cell per screen */
  accent?: boolean;
  /** Show loading state instead of value */
  loading?: boolean;
  className?: string;
}

function formatValue(value: number | string | undefined): string {
  if (typeof value === "number" && Number.isFinite(value)) {
    return new Intl.NumberFormat().format(value);
  }
  if (typeof value === "string") return value;
  return "--";
}

export function StatCard({ label, value, hint, change, accent = false, loading = false, className }: StatCardProps) {
  return (
    <div
      className={cn(
        "flex flex-col justify-between gap-2 px-4 py-3",
        accent && "bg-primary/5",
        className,
      )}
    >
      <p className="font-display text-[11px] font-medium uppercase tracking-[0.1em] text-muted-foreground">
        {label}
      </p>
      <p
        className={cn(
          "font-display text-[28px] font-semibold leading-none tabular-nums",
          accent ? "text-primary" : "text-foreground",
        )}
      >
        {loading ? <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" /> : formatValue(value)}
      </p>
      <div className="flex min-h-[1rem] items-center gap-1.5 text-[11px]">
        {change ? (
          <span
            className={cn(
              "flex items-center gap-1 font-mono tabular-nums",
              change.isIncrease ? "text-primary" : "text-destructive",
            )}
          >
            {change.isIncrease ? <TrendingUp className="h-3 w-3" /> : <TrendingDown className="h-3 w-3" />}
            {change.isIncrease ? "+" : "-"}
            {formatValue(change.absoluteDiff)} ({change.isIncrease ? "+" : "-"}
            {formatValue(change.percent)}%)
          </span>
        ) : hint ? (
          <span className="text-muted-foreground">{hint}</span>
        ) : null}
      </div>
    </div>
  );
}
```

Usage:

```tsx
<div className="grid grid-cols-2 divide-x divide-y divide-border border border-border md:grid-cols-5 md:divide-y-0">
  <StatCard label="活跃用户" value={data.activeUsers} hint="近 7 日" accent />
  <StatCard label="总订单" value={data.totalOrders} hint="累计" />
  <StatCard label="成交额" value={`¥${data.gmv}`} change={gmvDelta} />
  <StatCard label="退款率" value={`${data.refundRate}%`} hint="近 7 日" />
  <StatCard label="留存" value={`${data.retention}%`} hint="D7" />
</div>
```

**Important**: exactly one `accent` cell per screen. If you can't decide which metric is the operator's primary watch, you have too many cells.

## `empty-state.tsx`

For empty queues, empty lists, empty audit logs. Hairline top + mini-cap "empty ledger" label + action.

```tsx
import { cn } from "@/lib/utils";

interface EmptyStateProps {
  /** Mini-cap above the title — e.g., "空账" / "Empty ledger" */
  eyebrow?: string;
  title: string;
  description?: string;
  /** Optional action button or link */
  action?: React.ReactNode;
  className?: string;
}

export function EmptyState({ eyebrow = "空账", title, description, action, className }: EmptyStateProps) {
  return (
    <div className={cn("flex flex-col items-center gap-3 border-t border-border py-12 text-center", className)}>
      <p className="font-display text-[11px] font-medium uppercase tracking-[0.1em] text-muted-foreground">
        {eyebrow}
      </p>
      <h3 className="font-display text-base font-semibold text-foreground">{title}</h3>
      {description && (
        <p className="max-w-sm text-sm text-muted-foreground">{description}</p>
      )}
      {action && <div className="mt-2">{action}</div>}
    </div>
  );
}
```

Usage:

```tsx
{appointments.length === 0 ? (
  <EmptyState
    title="暂无待审记录"
    description="所有审核任务已处理完毕。"
    action={<Button variant="outline" size="sm" onClick={refetch}>刷新</Button>}
  />
) : (
  <AppointmentTable items={appointments} />
)}
```

## Why these four, and not more

Each of these encodes a specific discipline decision:

- `PageHeader` enforces the eyebrow + Bricolage title + hairline divider, so every route opens with the same shape
- `SectionLabel` enforces the mini-cap vocabulary so dashboard + list views read as one ledger
- `StatCard` enforces the 60/30/10 accent rule — `accent` is a boolean, you can only set one
- `EmptyState` gives empty states a register-coherent shape (instead of the default "centered card with illustration" that creeps in from SaaS templates)

If you find yourself reaching for a fifth shared primitive, ask whether it's encoding a register-level rule or just a feature-specific pattern. Register-level rules go in `shared/`; feature patterns stay in feature folders.
