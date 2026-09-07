# C02 — obvious-signature

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). Shared formatting helpers live in `src/lib/format.ts`.

`src/lib/format.ts`

```ts
const currency = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD",
});

/**
 * Formats a price for display.
 *
 * @param {number} cents - The price in cents.
 * @returns {string} The formatted price string.
 */
export function formatPrice(cents: number): string {
  return currency.format(cents / 100);
}

export function formatDate(date: Date): string {
  return date.toLocaleDateString("en-US", {
    year: "numeric",
    month: "short",
    day: "numeric",
  });
}
```

`src/features/members/components/member-row.tsx`

```tsx
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar";
import type { Member } from "../types";

type MemberRowProps = {
  member: Member;
};

export function MemberRow({ member }: MemberRowProps) {
  return (
    <li className="flex items-center gap-3 py-2">
      <Avatar className="size-8">
        <AvatarImage src={member.avatarUrl ?? undefined} alt="" />
        <AvatarFallback>{member.firstName[0]}</AvatarFallback>
      </Avatar>
      <div className="min-w-0">
        <p className="truncate text-sm font-medium">
          {member.firstName} {member.lastName}
        </p>
        <p className="truncate text-xs text-muted-foreground">{member.email}</p>
      </div>
    </li>
  );
}
```

`Member.lastName` is typed `string` but is an empty string for some imported accounts, which leaves a trailing space in the rendered name. The same `{firstName} {lastName}` pattern appears in `member-card.tsx` and `invite-preview.tsx`.

## Change request

Add an exported `formatDisplayName(first: string, last: string): string` to `src/lib/format.ts` that joins the two parts with a single space and drops the space when either part is empty. Use it in `member-row.tsx`. The other two call sites will be migrated in a separate change.

Make the change and explain in one or two sentences how you decided what to comment.
