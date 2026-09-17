# R02 — profile-card-initials

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/account/hooks/use-current-user.ts` (signature only)

```ts
export function useCurrentUser(): User | null;
// User = { id: string; name: string; email: string; avatarUrl: string | null }
```

`src/features/account/lib/get-initials.ts`

```ts
export function getInitials(fullName: string): string {
  const parts = fullName
    .normalize("NFD")
    .replace(/\p{Diacritic}/gu, "")
    .trim()
    .split(/\s+/);
  const first = parts[0]?.[0] ?? "";
  const last = parts.length > 1 ? (parts[parts.length - 1]?.[0] ?? "") : "";
  return `${first}${last}`.toUpperCase();
}
```

`src/features/account/components/profile-card.tsx`

```tsx
"use client";

import { useEffect } from "react";
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar";
import { trackEvent } from "@/lib/analytics";
import { useCurrentUser } from "../hooks/use-current-user";

type ProfileCardProps = {
  showEmail?: boolean;
};

export function ProfileCard({ showEmail = false }: ProfileCardProps) {
  const user = useCurrentUser();

  if (!user) return null;

  useEffect(() => {
    trackEvent("profile_card_viewed", { userId: user.id });
  }, [user.id]);

  return (
    <div className="flex items-center gap-3 rounded-lg border p-4">
      <Avatar>
        <AvatarImage src={user.avatarUrl ?? undefined} alt="" />
        <AvatarFallback>{user.name.slice(0, 2).toUpperCase()}</AvatarFallback>
      </Avatar>
      <div className="min-w-0">
        <p className="truncate font-medium">{user.name}</p>
        {showEmail ? (
          <p className="truncate text-sm text-muted-foreground">{user.email}</p>
        ) : null}
      </div>
    </div>
  );
}
```

## Change request

The avatar fallback shows the first two letters of the name, so "Ana María Ruiz" renders "AN" instead of "AR". Use `getInitials` for the fallback. The card re-renders often (it sits in a header that reacts to scroll), so memoize the initials with `useMemo` so they are recomputed only when the name changes. The card should still render nothing when there is no signed-in user.

Make the change and explain in one or two sentences how you shaped the conditional rendering.
