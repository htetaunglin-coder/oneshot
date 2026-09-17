# S5 — effect-after-guard

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui, TanStack Query). The account feature has an overview component that shows a sign-in prompt when there is no session and otherwise loads the account and renders tabs. The file has shipped this way for months; the session never changes while the page is mounted, so nobody has seen it misbehave.

`src/features/account/components/account-overview.tsx`

```tsx
"use client";

import { useState } from "react";
import { useQuery } from "@tanstack/react-query";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { useSession } from "@/features/auth/hooks/use-session";
import { fetchAccount } from "@/features/account/lib/api";
import { AccountSkeleton } from "@/features/account/components/account-skeleton";
import { ProfileTab } from "@/features/account/components/profile-tab";
import { SecurityTab } from "@/features/account/components/security-tab";
import { SignIn } from "@/features/auth/components/sign-in";

type AccountTab = "profile" | "security";

export function AccountOverview() {
  const session = useSession();

  if (!session) return <SignIn returnTo="/account" />;

  const { data: account, isLoading } = useQuery({
    queryKey: ["account", session.user.id],
    queryFn: () => fetchAccount(session.user.id),
  });
  const [tab, setTab] = useState<AccountTab>("profile");

  if (isLoading || !account) return <AccountSkeleton />;

  return (
    <Tabs value={tab} onValueChange={(value) => setTab(value as AccountTab)}>
      <TabsList>
        <TabsTrigger value="profile">Profile</TabsTrigger>
        <TabsTrigger value="security">Security</TabsTrigger>
      </TabsList>
      <TabsContent value="profile">
        <ProfileTab account={account} />
      </TabsContent>
      <TabsContent value="security">
        <SecurityTab account={account} />
      </TabsContent>
    </Tabs>
  );
}
```

`useSession()` returns `Session | null`. `src/lib/analytics.ts` exports `track(event: string, props?: Record<string, string>)`.

## Change request

"Analytics wants an `account_viewed` event with `{ userId }` once when a signed-in user lands on this component. Signed-out visitors must not send it, and it must not fire again when the user switches tabs. Use `track` from `@/lib/analytics`."

Make the change and explain in one or two sentences how you shaped the conditional rendering.
