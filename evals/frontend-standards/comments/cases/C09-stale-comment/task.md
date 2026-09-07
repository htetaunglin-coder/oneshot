# C09 — stale-comment

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). Checkout charges the customer through a server action that wraps the payment provider's SDK.

`src/features/checkout/actions/charge-order.ts`

```ts
"use server";

import { paymentClient, PaymentError } from "@/lib/payments";
import { sleep } from "@/lib/sleep";

type ChargeInput = {
  orderId: string;
  amountCents: number;
  paymentMethodId: string;
};

type ChargeResult =
  | { ok: true; chargeId: string }
  | { ok: false; error: string };

const MAX_ATTEMPTS = 3;
const BASE_DELAY_MS = 500;

// Retries three times because the payment API rate-limits bursts.
export async function chargeOrder(input: ChargeInput): Promise<ChargeResult> {
  for (let attempt = 1; attempt <= MAX_ATTEMPTS; attempt++) {
    try {
      const charge = await paymentClient.charges.create({
        amount: input.amountCents,
        currency: "usd",
        payment_method: input.paymentMethodId,
        idempotency_key: input.orderId,
      });
      return { ok: true, chargeId: charge.id };
    } catch (error) {
      const retryable =
        error instanceof PaymentError && error.status === 429 && attempt < MAX_ATTEMPTS;
      if (!retryable) {
        return {
          ok: false,
          error: error instanceof Error ? error.message : "Charge failed",
        };
      }
      await sleep(BASE_DELAY_MS * attempt);
    }
  }

  return { ok: false, error: "Charge failed" };
}
```

## Change request

On-call report: during the Friday promo, charges that got a 429 kept failing after all three attempts because the retries from concurrent checkouts landed at the same instant. Raise the attempt count to five and add a random jitter of 0–250 ms to every delay so concurrent retries spread out.

Make the change and explain in one or two sentences how you decided what to comment.
