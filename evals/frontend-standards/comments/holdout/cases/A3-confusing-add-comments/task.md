# A3 — confusing-add-comments

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The pricing feature has one function that every quote goes through. Its author has left the company. The code is on the release branch and must keep identical behaviour.

`src/features/pricing/lib/quote.ts`

```ts
type Item = {
  p: number;
  n: number;
};

export type Quote = {
  items: Item[];
  n: number;
  s: string;
  w: number;
};

export type QuoteOptions = {
  c: boolean;
  p: boolean;
  r: boolean;
  d?: { k: "pct" | "flat"; v: number };
};

export function calc(q: Quote, o: QuoteOptions): number {
  let t = 0;
  for (const i of q.items) {
    t += i.p * i.n;
  }

  let r = t;
  if (o.c && !o.p && q.n > 1) {
    r = r * 0.9;
  }

  const d = o.d ? (o.d.k === "pct" ? r * (o.d.v / 100) : o.d.v) : 0;
  const s = r - d < 0 ? 0 : r - d;

  const f = q.s === "TX" ? s * 0.0825 : q.s === "OR" ? 0 : s * 0.06;

  const g = o.r ? 0 : q.w > 50 ? 25 : q.w > 0 ? 8 : 0;

  return Math.round((s + f + g) * 100) / 100;
}
```

What the team pieced together from old Slack threads and the tests:

- `Item.p` is the unit price in dollars, `Item.n` the quantity. `Quote.n` is the number of sites the customer is buying for, `Quote.s` the two-letter US state of the shipping address, `Quote.w` the shipment weight in kilograms.
- `QuoteOptions.c` means the customer is on a contract account, `p` means a promo code was already applied, `r` means in-store pickup, `d` is a coupon (percentage or flat dollars).
- Contract customers buying for more than one site get 10% off, unless a promo is already applied (contract discount and promo do not stack, sales rule from 2024).
- The company is registered in Texas, so Texas orders are taxed at the state rate of 8.25%. Oregon has no sales tax. Every other state gets a flat 6% estimate until the tax-service integration ships (issue #540).
- Shipping is $25 over 50 kg, $8 otherwise, free for pickup or for weightless (digital) quotes.
- Totals are rounded to cents at the end, not per line, because that is what the invoices already show.

## Change request

From a reviewer: "I can't follow `calc`. Please add comments so the next person can."

Make the change and explain in one or two sentences how you decided what to comment.
