# Task

## Codebase

React + TypeScript, Next.js App Router, Tailwind v4, shadcn/ui. Feature code lives in `src/features/<name>/components`, shared primitives in `src/components/ui` (`card.tsx`, `button.tsx`, `input.tsx`, etc. are installed), `cn` is exported from `src/lib/utils`, theme lives in `src/app/globals.css`.

The following five files each contain the same wrapper class string. Nothing else in the repo uses it.

### `src/features/billing/components/payment-method-form.tsx`

```tsx
"use client";

import { useActionState } from "react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { savePaymentMethod } from "../actions";

export function PaymentMethodForm() {
  const [state, action, pending] = useActionState(savePaymentMethod, null);

  return (
    <form
      action={action}
      className="rounded-lg border bg-card p-4 shadow-sm transition-shadow hover:shadow-md focus-within:ring-2 focus-within:ring-ring"
    >
      <div className="grid gap-3">
        <Label htmlFor="cardholder">Cardholder name</Label>
        <Input id="cardholder" name="cardholder" autoComplete="cc-name" required />
        <Label htmlFor="card">Card number</Label>
        <Input id="card" name="card" inputMode="numeric" autoComplete="cc-number" required />
      </div>
      {state?.error ? <p className="mt-2 text-sm text-destructive">{state.error}</p> : null}
      <Button type="submit" className="mt-4" disabled={pending}>
        Save card
      </Button>
    </form>
  );
}
```

### `src/features/billing/components/invoice-list.tsx`

```tsx
import Link from "next/link";
import type { Invoice } from "../types";

export function InvoiceList({ invoices }: { invoices: Invoice[] }) {
  return (
    <div className="rounded-lg border bg-card p-4 shadow-sm transition-shadow hover:shadow-md focus-within:ring-2 focus-within:ring-ring">
      <h2 className="text-sm font-medium text-muted-foreground">Invoices</h2>
      <ul className="mt-2 divide-y">
        {invoices.map((inv) => (
          <li key={inv.id} className="flex items-center justify-between py-2">
            <Link href={`/billing/invoices/${inv.id}`} className="text-sm underline-offset-4 hover:underline">
              {inv.number}
            </Link>
            <span className="tabular-nums">{inv.total}</span>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### `src/features/reports/components/revenue-chart.tsx`

```tsx
"use client";

import { Area, AreaChart, ResponsiveContainer, Tooltip, XAxis } from "recharts";
import type { RevenuePoint } from "../types";

export function RevenueChart({ data }: { data: RevenuePoint[] }) {
  return (
    <figure className="rounded-lg border bg-card p-4 shadow-sm transition-shadow hover:shadow-md focus-within:ring-2 focus-within:ring-ring">
      <figcaption className="mb-3 text-sm font-medium">Monthly revenue</figcaption>
      <ResponsiveContainer width="100%" height={220}>
        <AreaChart data={data}>
          <XAxis dataKey="month" tickLine={false} axisLine={false} fontSize={12} />
          <Tooltip cursor={false} />
          <Area type="monotone" dataKey="revenue" strokeWidth={2} fillOpacity={0.15} />
        </AreaChart>
      </ResponsiveContainer>
    </figure>
  );
}
```

### `src/features/reports/components/export-panel.tsx`

```tsx
"use client";

import { useTransition } from "react";
import { Button } from "@/components/ui/button";
import { exportReport } from "../actions";

export function ExportPanel({ reportId }: { reportId: string }) {
  const [pending, start] = useTransition();

  return (
    <aside className="rounded-lg border bg-card p-4 shadow-sm transition-shadow hover:shadow-md focus-within:ring-2 focus-within:ring-ring">
      <p className="text-sm text-muted-foreground">Download this report as a spreadsheet.</p>
      <div className="mt-3 flex gap-2">
        <Button size="sm" variant="outline" disabled={pending} onClick={() => start(() => exportReport(reportId, "csv"))}>
          CSV
        </Button>
        <Button size="sm" variant="outline" disabled={pending} onClick={() => start(() => exportReport(reportId, "xlsx"))}>
          XLSX
        </Button>
      </div>
    </aside>
  );
}
```

### `src/features/settings/components/notification-preferences.tsx`

```tsx
"use client";

import { Switch } from "@/components/ui/switch";
import { Label } from "@/components/ui/label";
import { usePreferences } from "../hooks/use-preferences";

export function NotificationPreferences() {
  const { prefs, update } = usePreferences();

  return (
    <section className="rounded-lg border bg-card p-4 shadow-sm transition-shadow hover:shadow-md focus-within:ring-2 focus-within:ring-ring">
      <h2 className="text-sm font-medium">Notifications</h2>
      <div className="mt-3 space-y-3">
        <div className="flex items-center justify-between">
          <Label htmlFor="email">Email digest</Label>
          <Switch id="email" checked={prefs.email} onCheckedChange={(v) => update({ email: v })} />
        </div>
        <div className="flex items-center justify-between">
          <Label htmlFor="push">Push alerts</Label>
          <Switch id="push" checked={prefs.push} onCheckedChange={(v) => update({ push: v })} />
        </div>
      </div>
    </section>
  );
}
```

## Change request

Design wants the panel borders softened in dark mode. Add `dark:border-border/60` to the wrapper in all five files above. This is the third time in a month someone has had to touch that exact string in every one of these files, so feel free to do whatever you think is right about that while you're in there.

Make the change and explain in one or two sentences how you structured the classes.
