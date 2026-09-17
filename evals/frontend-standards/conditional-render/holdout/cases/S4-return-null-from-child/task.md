# S4 — return-null-from-child

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The dashboard page shows a promo banner above the main grid for users on an eligible plan. The banner component takes the user and decides whether to show itself.

`src/features/dashboard/components/dashboard-header.tsx`

```tsx
import { PromoBanner } from "@/features/dashboard/components/promo-banner";
import { WorkspaceSwitcher } from "@/features/workspaces/components/workspace-switcher";
import type { DashboardUser } from "@/features/dashboard/lib/types";

type DashboardHeaderProps = {
  user: DashboardUser;
};

export function DashboardHeader({ user }: DashboardHeaderProps) {
  return (
    <header className="flex flex-col gap-4">
      <div className="flex items-center justify-between">
        <h1 className="text-xl font-semibold">Welcome back, {user.firstName}</h1>
        <WorkspaceSwitcher current={user.workspaceId} />
      </div>
      <PromoBanner user={user} />
    </header>
  );
}
```

`src/features/dashboard/components/promo-banner.tsx`

```tsx
"use client";

import { Button } from "@/components/ui/button";
import { dismissPromo } from "@/features/dashboard/lib/actions";
import type { DashboardUser } from "@/features/dashboard/lib/types";

type PromoBannerProps = {
  user: DashboardUser;
};

export function PromoBanner({ user }: PromoBannerProps) {
  if (!user.eligibleForPromo) return null;

  return (
    <div className="bg-primary/10 flex items-center justify-between rounded-md px-4 py-3">
      <p className="text-sm">
        Upgrade before {user.promoEndsAt} and get two months free.
      </p>
      <div className="flex gap-2">
        <Button size="sm" asChild>
          <a href="/settings/billing">Upgrade</a>
        </Button>
        <Button size="sm" variant="ghost" onClick={() => dismissPromo(user.id)}>
          Dismiss
        </Button>
      </div>
    </div>
  );
}
```

`DashboardUser` in `src/features/dashboard/lib/types.ts`:

```ts
export type DashboardUser = {
  id: string;
  firstName: string;
  workspaceId: string;
  eligibleForPromo: boolean;
  promoEndsAt: string;
  dismissedPromo: boolean;
};
```

`dismissPromo` is a server action that sets `dismissedPromo` to true and revalidates the dashboard.

## Change request

"The Dismiss button writes `dismissedPromo`, but the banner still comes back on the next render. It must stay hidden once `user.dismissedPromo` is true, on top of the eligibility check that is already there."

Make the change and explain in one or two sentences how you shaped the conditional rendering.
