# N13 — processor-fee

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The checkout feature computes the card processing fee that is shown on the order summary and passed to the charge call, formats amounts for display, and retries a declined-but-retryable charge with a growing wait between attempts.

`src/features/checkout/lib/payment.ts`

```ts
import { chargeCard, type ChargeResult } from "@/lib/payments";

export function computeCardFeeCents(amountCents: number): number {
  return Math.round(amountCents * 0.029 + 30);
}

export function formatCents(cents: number): string {
  return new Intl.NumberFormat("en-US", { style: "currency", currency: "USD" }).format(cents / 100);
}

function sleep(ms: number) {
  return new Promise<void>((resolve) => setTimeout(resolve, ms));
}

export async function chargeWithRetry(
  token: string,
  amountCents: number,
  waitMs = 500,
): Promise<ChargeResult> {
  const first = await chargeCard(token, amountCents);
  if (first.ok || !first.retryable) return first;

  await sleep(waitMs);
  const second = await chargeCard(token, amountCents);
  if (second.ok || !second.retryable) return second;

  await sleep(waitMs * 2);
  return chargeCard(token, amountCents);
}
```

`src/features/checkout/components/order-summary.tsx` renders `formatCents(computeCardFeeCents(subtotalCents))` on a "Processing fee" row, and `src/features/checkout/lib/place-order.ts` adds the same function's result to the total before calling `chargeWithRetry`. Nothing else in the repo reads these numbers.

## Change request

"Our card processor changed the fixed part of the per-transaction fee from 30 cents to 35 cents, effective from the next deploy. The percentage part is unchanged. Update the fee calculation."

Make the change and explain in one or two sentences how you decided which values get a name.
