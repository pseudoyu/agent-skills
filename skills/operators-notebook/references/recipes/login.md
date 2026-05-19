# Recipe: Login Chrome

Centered 360px column. Mini-cap eyebrow + Bricolage title + hairline divider + form. The auth form itself stays a feature component — only the chrome around it changes.

## `app/login/page.tsx`

```tsx
import { SmsLoginCard } from "@/components/SmsLoginCard";

export default function LoginPage() {
  return (
    <div className="flex min-h-screen items-center justify-center bg-background px-4">
      <div className="flex w-full max-w-[360px] flex-col gap-10">
        <header className="space-y-3">
          <p className="font-display text-[11px] font-medium uppercase tracking-[0.1em] text-muted-foreground">
            管理后台
          </p>
          <h1 className="font-display text-2xl font-semibold tracking-tight text-foreground">
            游酒YoJoe宇宙
          </h1>
          <p className="text-sm text-muted-foreground">
            请使用管理员手机号接收验证码登录。
          </p>
        </header>

        <div className="border-t border-border" />

        <SmsLoginCard />

        <p className="text-center font-mono text-[11px] tabular-nums text-muted-foreground">
          v1.0.0 · 2026.05
        </p>
      </div>
    </div>
  );
}
```

Key decisions:

- **No gradient background.** Solid `bg-background`. The login page is part of the same ledger as the dashboard.
- **No card with shadow.** The form sits directly on the canvas; the hairline divider (`border-t border-border`) separates eyebrow from form.
- **Mini-cap eyebrow + Bricolage title** matches the page-header vocabulary on every other route.
- **Mono version stamp at bottom** ties in the editorial-mono voice.
- **Form is a separate feature component** (`SmsLoginCard`) — data flow stays intact.

## `components/SmsLoginCard.tsx` (chrome only)

Show the chrome decisions; the data flow / oRPC call / validation stay as-is. Only swap the visual layer:

```tsx
"use client";

import { useState } from "react";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Label } from "@/components/ui/label";
import { Loader2 } from "lucide-react";
// ... your existing imports

export function SmsLoginCard() {
  // ... your existing state + mutations untouched
  const [phone, setPhone] = useState("");
  const [code, setCode] = useState("");
  const [step, setStep] = useState<"phone" | "code">("phone");
  // ...

  if (step === "phone") {
    return (
      <form className="space-y-6" onSubmit={handleSendCode}>
        <div className="space-y-2">
          <Label htmlFor="phone" className="font-display text-[11px] font-medium uppercase tracking-[0.1em] text-muted-foreground">
            手机号
          </Label>
          <Input
            id="phone"
            type="tel"
            inputMode="numeric"
            value={phone}
            onChange={(e) => setPhone(e.target.value)}
            placeholder="请输入 11 位手机号"
            className="font-mono tabular-nums"
            autoFocus
          />
        </div>
        {error && <p className="text-sm text-destructive">{error}</p>}
        <Button type="submit" className="w-full" disabled={isLoading}>
          {isLoading && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
          发送验证码
        </Button>
      </form>
    );
  }

  // OTP step — digit slots with mono + wide tracking
  return (
    <form className="space-y-6" onSubmit={handleVerify}>
      <div className="space-y-2">
        <Label htmlFor="code" className="font-display text-[11px] font-medium uppercase tracking-[0.1em] text-muted-foreground">
          验证码
        </Label>
        <Input
          id="code"
          type="text"
          inputMode="numeric"
          maxLength={6}
          value={code}
          onChange={(e) => setCode(e.target.value)}
          placeholder="6 位数字"
          // Reason: OTP digits use mono + wide tracking — paper-stamp typesetting
          className="text-center font-mono text-lg tabular-nums tracking-[0.25em]"
          autoFocus
        />
        <p className="font-mono text-[11px] tabular-nums text-muted-foreground">
          验证码已发送至 {phone}
        </p>
      </div>
      {error && <p className="text-sm text-destructive">{error}</p>}
      <Button type="submit" className="w-full" disabled={isLoading}>
        {isLoading && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
        登录
      </Button>
      <Button type="button" variant="ghost" size="sm" className="w-full" onClick={() => setStep("phone")}>
        重新输入手机号
      </Button>
    </form>
  );
}
```

Key chrome decisions:

- Labels use the mini-cap vocabulary (`font-display text-[11px] uppercase tracking-[0.1em]`)
- Phone input uses `font-mono tabular-nums` — same numeric voice as the dashboard
- OTP input uses `font-mono text-lg tabular-nums tracking-[0.25em]` — wide-tracked monospace reads as digit slots without needing a custom OTP component
- Submit button is full-width default `variant="default"` — the only primary CTA on the page
- Error text uses `text-destructive` — the only off-family color allowed in the register
- "Resend" / "back" actions are `variant="ghost" size="sm"` — quiet secondary actions

## What to delete from the old login

When migrating from default shadcn auth:

- Any gradient background (`bg-gradient-to-br from-blue-500 to-purple-600`)
- Any centered card with `shadow-2xl rounded-3xl`
- Any "Welcome back!" decorative subtitle that adds no information
- `backdrop-blur` on the form container
- Decorative illustrations / floating shapes

The login page reads as part of the dashboard, not a different product.
