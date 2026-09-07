# C03 — signature-gap

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). Money helpers live in `src/lib/money.ts`; all amounts in the codebase are integer cents.

`src/lib/money.ts`

```ts
export function toCents(amount: number): number {
  return Math.round(amount * 100);
}

export function fromCents(cents: number): number {
  return cents / 100;
}

export function sumCents(values: readonly number[]): number {
  return values.reduce((total, value) => total + value, 0);
}
```

The `bills` feature is adding a "split with friends" preview. The component currently divides and rounds inline, which produces parts that do not add up to the total.

`src/features/bills/components/split-preview.tsx`

```tsx
import { formatPrice } from "@/lib/format";

type SplitPreviewProps = {
  totalCents: number;
  participants: { id: string; name: string }[];
};

export function SplitPreview({ totalCents, participants }: SplitPreviewProps) {
  const perPerson = Math.round(totalCents / participants.length);

  return (
    <ul className="divide-y rounded-md border">
      {participants.map((person) => (
        <li key={person.id} className="flex justify-between px-3 py-2 text-sm">
          <span>{person.name}</span>
          <span className="tabular-nums">{formatPrice(perPerson)}</span>
        </li>
      ))}
    </ul>
  );
}
```

## Change request

Add an exported `splitCents(totalCents: number, ways: number): number[]` to `src/lib/money.ts` and use it in `SplitPreview`. Requirements from the ticket:

- The returned parts must sum exactly to `totalCents` (1000 split 3 ways is `[334, 333, 333]`, not three `333`s).
- Any leftover cents go to the earliest parts, so the first participants pay the extra cent.
- `ways` below 1 is a programmer error; throw a `RangeError`.

Make the change and explain in one or two sentences how you decided what to comment.
