# Recipe: shadcn/ui Primitive Variants

The shadcn/ui primitives ship with sensible defaults that read as "default SaaS." This register tunes each one to drop chrome (shadows, pillowy radius), tighten focus, and use only semantic tokens.

Transplant these in `components/ui/`. Start with `button`, `card`, `input`, `dialog`, `badge` — they cover ~80% of surface.

## `button.tsx`

```tsx
import * as React from "react";
import { Slot } from "@radix-ui/react-slot";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const buttonVariants = cva(
  // Reason: no shadow, tight radius, ring-1 focus, transitions on colors/transform only
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        // Reason: outline is transparent — uses canvas, hairline border, accent hover
        outline: "border border-border bg-transparent text-foreground hover:bg-accent/40",
        // Reason: ghost stays flat, accent on hover (no underline, no shadow)
        ghost: "text-foreground hover:bg-accent/40",
        destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        link: "text-foreground underline-offset-4 hover:underline",
      },
      size: {
        default: "h-9 px-4 py-2",
        sm: "h-8 px-3 text-xs",
        lg: "h-10 px-6",
        icon: "h-9 w-9",
        "icon-sm": "h-8 w-8",
        "icon-lg": "h-10 w-10",
      },
    },
    defaultVariants: { variant: "default", size: "default" },
  },
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button";
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  },
);
Button.displayName = "Button";

export { Button, buttonVariants };
```

What changed vs default shadcn:

- No `shadow-sm` on any variant
- `outline` is transparent (default shadcn has `bg-background`); the hairline + canvas reads more editorial
- `ghost` uses `hover:bg-accent/40` (40% wash), not `hover:bg-accent`
- `ring-1 ring-ring` focus, not `ring-2 ring-ring`
- Added `icon-sm` (32px) and `icon-lg` (40px) sizes for layout flexibility

## `card.tsx`

```tsx
import * as React from "react";
import { cn } from "@/lib/utils";

const Card = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
  ({ className, ...props }, ref) => (
    <div
      ref={ref}
      // Reason: hairline border + tight radius + no shadow. The card surface is
      // visually equal to background; hierarchy comes from border, not elevation.
      className={cn("rounded-md border border-border bg-card text-card-foreground", className)}
      {...props}
    />
  ),
);
Card.displayName = "Card";

const CardHeader = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
  ({ className, ...props }, ref) => (
    <div ref={ref} className={cn("flex flex-col space-y-1.5 p-6", className)} {...props} />
  ),
);
CardHeader.displayName = "CardHeader";

const CardTitle = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
  ({ className, ...props }, ref) => (
    <div
      ref={ref}
      // Reason: card titles use Bricolage display face
      className={cn("font-display text-base font-semibold tracking-tight", className)}
      {...props}
    />
  ),
);
CardTitle.displayName = "CardTitle";

const CardDescription = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
  ({ className, ...props }, ref) => (
    <div ref={ref} className={cn("text-sm text-muted-foreground", className)} {...props} />
  ),
);
CardDescription.displayName = "CardDescription";

const CardContent = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
  ({ className, ...props }, ref) => (
    <div ref={ref} className={cn("p-6 pt-0", className)} {...props} />
  ),
);
CardContent.displayName = "CardContent";

const CardFooter = React.forwardRef<HTMLDivElement, React.HTMLAttributes<HTMLDivElement>>(
  ({ className, ...props }, ref) => (
    <div ref={ref} className={cn("flex items-center p-6 pt-0", className)} {...props} />
  ),
);
CardFooter.displayName = "CardFooter";

export { Card, CardHeader, CardFooter, CardTitle, CardDescription, CardContent };
```

What changed:

- `rounded-md` (not `rounded-xl`)
- `border border-border` (single 1px hairline, no shadow)
- `CardTitle` uses `font-display`

## `input.tsx`

```tsx
import * as React from "react";
import { cn } from "@/lib/utils";

const Input = React.forwardRef<HTMLInputElement, React.InputHTMLAttributes<HTMLInputElement>>(
  ({ className, type, ...props }, ref) => {
    return (
      <input
        type={type}
        // Reason: h-10 (slightly taller than default 9 for comfortable text input),
        // bg-card (not background — gives a subtle field affordance),
        // ring-1 focus (no glow)
        className={cn(
          "flex h-10 w-full rounded-md border border-input bg-card px-3 py-2 text-sm text-foreground transition-colors file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring disabled:cursor-not-allowed disabled:opacity-50",
          className,
        )}
        ref={ref}
        {...props}
      />
    );
  },
);
Input.displayName = "Input";

export { Input };
```

## `dialog.tsx` — overlay change

The key change in `Dialog` is the overlay: solid `bg-foreground/40` warm sumi scrim, no `backdrop-blur`:

```tsx
const DialogOverlay = React.forwardRef<
  React.ElementRef<typeof DialogPrimitive.Overlay>,
  React.ComponentPropsWithoutRef<typeof DialogPrimitive.Overlay>
>(({ className, ...props }, ref) => (
  <DialogPrimitive.Overlay
    ref={ref}
    // Reason: solid scrim, no glass — BAN 9
    className={cn(
      "fixed inset-0 z-50 bg-foreground/40 data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0",
      className,
    )}
    {...props}
  />
));
DialogOverlay.displayName = DialogPrimitive.Overlay.displayName;

const DialogContent = React.forwardRef<
  React.ElementRef<typeof DialogPrimitive.Content>,
  React.ComponentPropsWithoutRef<typeof DialogPrimitive.Content>
>(({ className, children, ...props }, ref) => (
  <DialogPortal>
    <DialogOverlay />
    <DialogPrimitive.Content
      ref={ref}
      className={cn(
        // Reason: hairline border, tight radius, no shadow — the scrim does
        // the lifting, not surface elevation
        "fixed left-[50%] top-[50%] z-50 grid w-full max-w-lg translate-x-[-50%] translate-y-[-50%] gap-4 border border-border bg-popover p-6 duration-200 sm:rounded-md data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[state=closed]:slide-out-to-left-1/2 data-[state=closed]:slide-out-to-top-[48%] data-[state=open]:slide-in-from-left-1/2 data-[state=open]:slide-in-from-top-[48%]",
        className,
      )}
      {...props}
    >
      {children}
      <DialogPrimitive.Close className="absolute right-4 top-4 rounded-sm opacity-70 ring-offset-background transition-opacity hover:opacity-100 focus:outline-none focus:ring-1 focus:ring-ring disabled:pointer-events-none">
        <X className="h-4 w-4" />
        <span className="sr-only">Close</span>
      </DialogPrimitive.Close>
    </DialogPrimitive.Content>
  </DialogPortal>
));
DialogContent.displayName = DialogPrimitive.Content.displayName;
```

What changed vs default shadcn:

- Overlay: `bg-foreground/40` (not `bg-black/80`), no `backdrop-blur`
- Content: `border border-border` (not `border bg-background shadow-lg`)
- `ring-1` close button focus, not `ring-2`

## `badge.tsx`

```tsx
import * as React from "react";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const badgeVariants = cva(
  "inline-flex items-center rounded-sm border px-2 py-0.5 text-[11px] font-medium uppercase tracking-[0.08em] transition-colors focus:outline-none focus:ring-1 focus:ring-ring",
  {
    variants: {
      variant: {
        default: "border-primary/40 bg-primary/5 text-primary",
        secondary: "border-border bg-muted text-foreground",
        destructive: "border-destructive/40 bg-destructive/5 text-destructive",
        outline: "border-border bg-transparent text-foreground",
        // Reason: count chips on active buttons — paper-ink stamp, hue-neutral
        // so it never collides with any active tint
        count: "border-transparent bg-foreground text-background font-mono tabular-nums uppercase tracking-normal",
      },
    },
    defaultVariants: { variant: "default" },
  },
);

export interface BadgeProps
  extends React.HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof badgeVariants> {}

function Badge({ className, variant, ...props }: BadgeProps) {
  return <div className={cn(badgeVariants({ variant }), className)} {...props} />;
}

export { Badge, badgeVariants };
```

What changed:

- All variants use semantic tokens only (no `bg-blue-100`)
- Adds `count` variant for paper-ink count chips (`bg-foreground text-background font-mono`)
- `text-[11px] uppercase tracking-[0.08em]` matches the mini-cap label vocabulary

## Other primitives

For `tabs`, `select`, `dropdown-menu`, `tooltip`, `switch`, `checkbox`, `scroll-area`, `separator`, `skeleton`, `avatar`, `table`, `label`, `textarea`, `form`, `toast`:

- Strip every `shadow-sm` / `shadow-md` / `shadow-lg`
- Replace any `rounded-xl` / `rounded-2xl` / `rounded-3xl` with `rounded-md`
- Replace any raw color literal (`bg-blue-50`, `text-emerald-600`, etc.) with semantic tokens per the [BAN 7 mapping table](../bans.md#ban-7--no-raw-tailwind-color-literals)
- Set `ring-1` (not `ring-2`) on focus states
- For `tabs`, active state uses underline (`data-[state=active]:after:bg-foreground`), not pill background
- For `table`, head row uses mini-cap (`text-[11px] uppercase tracking-[0.1em] text-muted-foreground font-display`)
- For `switch`, on-state thumb uses `bg-primary`, off-state uses `bg-muted-foreground/40`

The reliable approach is to start with shadcn's stock components, then run a `perl` scrub:

```bash
perl -i -pe '
  s|shadow-(sm|md|lg|xl|2xl)\b||g;
  s|rounded-(xl|2xl|3xl)\b|rounded-md|g;
  s|ring-2 ring-ring|ring-1 ring-ring|g;
  s|bg-white\b|bg-card|g;
  s|backdrop-blur-[a-z]+\b||g;
' components/ui/*.tsx
```

Then read each file by hand, fix anything contextual, and run `pnpm typecheck` to catch broken class names.
