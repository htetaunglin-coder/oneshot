# M02 — price-thousands-separator

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`, shared code under `src/lib`. Tests run with Vitest.

`src/lib/format.ts`

```ts
export function formatPrice(cents: number): string {
  const sign = cents < 0 ? "-" : "";
  const dollars = Math.abs(cents) / 100;
  return `${sign}$${dollars.toFixed(2)}`;
}

export function formatPercent(ratio: number): string {
  return `${(ratio * 100).toFixed(1)}%`;
}

export function formatSavings(originalCents: number, saleCents: number): string {
  const savedCents = originalCents - saleCents;
  const ratio = originalCents === 0 ? 0 : savedCents / originalCents;
  return `Save ${formatPrice(savedCents)} (${formatPercent(ratio)})`;
}
```

`src/lib/format.test.ts`

```ts
import { describe, expect, it } from "vitest";
import { formatPercent, formatPrice, formatSavings } from "@/lib/format";

describe("formatPrice", () => {
  it("renders cents as dollars with two decimals", () => {
    expect(formatPrice(1999)).toBe("$19.99");
  });

  it("keeps the sign in front of the currency symbol", () => {
    expect(formatPrice(-250)).toBe("-$2.50");
  });
});

describe("formatPercent", () => {
  it("renders a ratio as a percentage with one decimal", () => {
    expect(formatPercent(0.125)).toBe("12.5%");
  });
});

describe("formatSavings", () => {
  it("shows the amount and the percentage saved", () => {
    expect(formatSavings(10000, 7500)).toBe("Save $25.00 (25.0%)");
  });
});
```

## Change request

Large amounts are hard to read: `formatPrice(123456789)` currently gives `$1234567.89`. Add thousands separators (English grouping) so it gives `$1,234,567.89`, keep the two decimals and the sign placement, and add a test for it. `formatPercent` and `formatSavings` keep their current output.

Make the change and explain in one or two sentences how you decided which values get a name.
