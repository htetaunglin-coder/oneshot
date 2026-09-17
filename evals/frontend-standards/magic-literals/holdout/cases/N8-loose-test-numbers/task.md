# N8 — loose-test-numbers

## Scenario

A Next.js App Router project (React, TypeScript, Vitest). The checkout feature prices shipping from weight and distance and applies Texas sales tax on top.

`src/features/checkout/lib/calculate-shipping.ts`

```ts
export const TX_SALES_TAX_RATE = 0.0825;

const BASE_FEE_CENTS = 5_00;
const PER_KG_CENTS = 1_50;
const LONG_DISTANCE_THRESHOLD_KM = 100;
const LONG_DISTANCE_SURCHARGE_CENTS = 8_00;

export type ShippingInput = { weightKg: number; distanceKm: number };
export type ShippingQuote = { subtotalCents: number; taxCents: number; totalCents: number };

export function calculateShipping({ weightKg, distanceKm }: ShippingInput): ShippingQuote {
  const surcharge = distanceKm > LONG_DISTANCE_THRESHOLD_KM ? LONG_DISTANCE_SURCHARGE_CENTS : 0;
  const subtotalCents = Math.round(BASE_FEE_CENTS + weightKg * PER_KG_CENTS + surcharge);
  const taxCents = Math.round(subtotalCents * TX_SALES_TAX_RATE);
  return { subtotalCents, taxCents, totalCents: subtotalCents + taxCents };
}
```

`src/features/checkout/lib/calculate-shipping.test.ts`

```ts
import { describe, expect, it } from "vitest";
import { calculateShipping } from "./calculate-shipping";

describe("calculateShipping", () => {
  it("charges the base fee plus weight for a short trip", () => {
    const quote = calculateShipping({ weightKg: 2.5, distanceKm: 40 });
    expect(quote.subtotalCents).toBe(875);
    expect(quote.taxCents).toBe(Math.round(875 * 0.0825));
  });

  it("adds the long-distance surcharge above 100 km", () => {
    const quote = calculateShipping({ weightKg: 2.5, distanceKm: 120 });
    expect(quote.subtotalCents).toBe(1675);
    expect(quote.taxCents).toBe(Math.round(1675 * 0.0825));
  });

  it("does not add the surcharge at exactly 100 km", () => {
    const quote = calculateShipping({ weightKg: 1, distanceKm: 100 });
    expect(quote.subtotalCents).toBe(650);
    expect(quote.taxCents).toBe(Math.round(650 * 0.0825));
  });

  it("returns subtotal plus tax as the total", () => {
    const quote = calculateShipping({ weightKg: 4, distanceKm: 10 });
    expect(quote.totalCents).toBe(quote.subtotalCents + quote.taxCents);
  });
});
```

## Change request

Review comment on the PR, on the test file: "There are a lot of loose numbers in these tests. Tidy them up before merge so the next person can tell what each one is."

Make the change and explain in one or two sentences how you decided which values get a name.
