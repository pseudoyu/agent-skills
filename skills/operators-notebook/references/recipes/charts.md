# Recipe: Chart Typography (`lib/chart-typography.ts`)

Recharts constants for the register's flat, editorial chart voice. Import these everywhere instead of inlining chart styling per file — that's how drift creeps in.

## `lib/chart-typography.ts`

```ts
/**
 * Chart typography + style constants for the Operator's Notebook register.
 * Import everywhere instead of inlining recharts styling.
 *
 * Rules encoded here:
 * - Dashed grid (2 4) on stroke-border — editorial, not solid
 * - Small ticks (11px) for axis labels; medium (12px) for prominent axes
 * - Solid card-bg tooltips with hairline border — no blur, no shadow
 * - Hue-locked chart series — never reach for saturated qualitative palettes
 */

export const CHART_GRID_DASH = "2 4";

export const CHART_TICK_SM = {
  fontSize: 11,
  fill: "var(--muted-foreground)",
} as const;

export const CHART_TICK_MD = {
  fontSize: 12,
  fill: "var(--muted-foreground)",
} as const;

export const CHART_TOOLTIP_CONTENT_STYLE = {
  background: "var(--card)",
  border: "1px solid var(--border)",
  borderRadius: 4,
  fontSize: 12,
  color: "var(--foreground)",
  padding: "8px 12px",
  // No box-shadow — BAN 5
} as const;

export const CHART_TOOLTIP_LABEL_STYLE = {
  fontSize: 11,
  fontWeight: 500,
  color: "var(--muted-foreground)",
  marginBottom: 4,
  textTransform: "uppercase" as const,
  letterSpacing: "0.08em",
} as const;

export const CHART_SERIES_COLORS = [
  "var(--chart-1)",
  "var(--chart-2)",
  "var(--chart-3)",
  "var(--chart-4)",
  "var(--chart-5)",
] as const;

export const CHART_PRIMARY_LINE = {
  stroke: "var(--primary)",
  strokeWidth: 1.5,
  dot: false,
  activeDot: { r: 3, fill: "var(--primary)" },
} as const;

export const CHART_PRIMARY_AREA = {
  stroke: "var(--primary)",
  strokeWidth: 1.5,
  fill: "var(--primary)",
  fillOpacity: 0.12,
} as const;

export const CHART_PRIMARY_BAR = {
  fill: "var(--primary)",
  // Reason: flat-top bars. Rounded bar caps (radius={[4, 4, 0, 0]}) read as SaaS — banned here.
  radius: [0, 0, 0, 0] as [number, number, number, number],
} as const;
```

## Example — line chart

```tsx
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from "recharts";
import {
  CHART_GRID_DASH,
  CHART_TICK_SM,
  CHART_TOOLTIP_CONTENT_STYLE,
  CHART_TOOLTIP_LABEL_STYLE,
  CHART_PRIMARY_LINE,
} from "@/lib/chart-typography";

export function ActivityChart({ data }: { data: Array<{ date: string; value: number }> }) {
  return (
    <ResponsiveContainer width="100%" height={240}>
      <LineChart data={data} margin={{ top: 8, right: 8, bottom: 8, left: 8 }}>
        <CartesianGrid strokeDasharray={CHART_GRID_DASH} stroke="var(--border)" vertical={false} />
        <XAxis
          dataKey="date"
          tick={CHART_TICK_SM}
          tickLine={false}
          axisLine={{ stroke: "var(--border)" }}
        />
        <YAxis tick={CHART_TICK_SM} tickLine={false} axisLine={false} width={36} />
        <Tooltip
          contentStyle={CHART_TOOLTIP_CONTENT_STYLE}
          labelStyle={CHART_TOOLTIP_LABEL_STYLE}
          cursor={{ stroke: "var(--border)", strokeDasharray: CHART_GRID_DASH }}
        />
        <Line type="monotone" dataKey="value" {...CHART_PRIMARY_LINE} />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

## Example — bar chart

```tsx
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from "recharts";
import {
  CHART_GRID_DASH,
  CHART_TICK_SM,
  CHART_TOOLTIP_CONTENT_STYLE,
  CHART_TOOLTIP_LABEL_STYLE,
  CHART_PRIMARY_BAR,
} from "@/lib/chart-typography";

export function RevenueBarChart({ data }: { data: Array<{ month: string; revenue: number }> }) {
  return (
    <ResponsiveContainer width="100%" height={240}>
      <BarChart data={data} margin={{ top: 8, right: 8, bottom: 8, left: 8 }}>
        <CartesianGrid strokeDasharray={CHART_GRID_DASH} stroke="var(--border)" vertical={false} />
        <XAxis dataKey="month" tick={CHART_TICK_SM} tickLine={false} axisLine={{ stroke: "var(--border)" }} />
        <YAxis tick={CHART_TICK_SM} tickLine={false} axisLine={false} width={48} />
        <Tooltip
          contentStyle={CHART_TOOLTIP_CONTENT_STYLE}
          labelStyle={CHART_TOOLTIP_LABEL_STYLE}
          cursor={{ fill: "var(--accent)", fillOpacity: 0.4 }}
        />
        <Bar dataKey="revenue" {...CHART_PRIMARY_BAR} />
      </BarChart>
    </ResponsiveContainer>
  );
}
```

## Example — multi-series

```tsx
import { LineChart, Line, /* ... */ } from "recharts";
import { CHART_SERIES_COLORS, CHART_GRID_DASH, CHART_TICK_SM } from "@/lib/chart-typography";

export function MultiSeriesChart({ data, series }: { data: any[]; series: string[] }) {
  return (
    <ResponsiveContainer width="100%" height={240}>
      <LineChart data={data}>
        <CartesianGrid strokeDasharray={CHART_GRID_DASH} stroke="var(--border)" vertical={false} />
        <XAxis dataKey="date" tick={CHART_TICK_SM} />
        <YAxis tick={CHART_TICK_SM} />
        {series.map((key, index) => (
          <Line
            key={key}
            type="monotone"
            dataKey={key}
            stroke={CHART_SERIES_COLORS[index % CHART_SERIES_COLORS.length]}
            strokeWidth={1.5}
            dot={false}
          />
        ))}
      </LineChart>
    </ResponsiveContainer>
  );
}
```

The five chart series tokens are hue-locked to the `h~250` family with intentional desaturation. They are not for "categorical variety" the way Mixpanel or Amplitude use rainbow palettes — they're for differentiating 2-5 related series on the same chart. If you need more than 5 series, the chart should probably be split or the categories grouped.

## Anti-patterns specific to charts

These compound the universal BANs:

- **No rounded bar caps.** `radius={[4, 4, 0, 0]}` is SaaS. Use flat-top: `radius={[0, 0, 0, 0]}` (or just omit, since 0 is the default).
- **No accent-tinted legend badges.** Legend uses text + token-tinted swatch only. `<span className="bg-primary/10 px-2 py-0.5">activeUsers</span>` next to a swatch reads as Mixpanel tooling — fights the editorial voice.
- **No glass tooltips.** The tooltip is `var(--card)` solid bg + `var(--border)` 1px border. `backdrop-blur` here is BAN 9.
- **No saturated qualitative palettes.** Don't reach for `tableau10` or `category10`. The five chart tokens are the qualitative palette — if you need more, you have a chart-design problem, not a token problem.
- **No animation on chart enter/exit by default.** Recharts ships with `animationDuration={1500}` — set `isAnimationActive={false}` on `Line`/`Bar`/`Area` for production dashboards. Charts should appear with the data, not slither in.

```tsx
<Bar dataKey="revenue" {...CHART_PRIMARY_BAR} isAnimationActive={false} />
```

The exception is one-shot dashboard mount — if every chart animates on initial load, the dashboard feels playful, which is fine for marketing but wrong for operators. If you want a subtle mount effect, do it once at the layout level (CSS `fade-in-0`), not per-chart.
