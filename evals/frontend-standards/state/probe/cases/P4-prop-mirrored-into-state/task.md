# P4 — prop-mirrored-into-state

## Scenario

A Next.js App Router project (React 19, TypeScript, Tailwind v4, shadcn/ui), organised as `src/features/<x>/`, `src/components/ui`, `src/lib`. Server data is fetched with TanStack Query (`QueryClientProvider` in the root layout). The account page shows a read-only profile card next to a settings form. `initials(name: string): string` in `src/lib/format.ts` turns `"Ada Lovelace"` into `"AL"`.

`src/features/profile/types.ts`

```ts
export type User = {
  id: string;
  name: string;
  email: string;
  title: string | null;
  avatarUrl: string | null;
  role: "owner" | "admin" | "member";
};
```

`src/features/profile/hooks/use-user.ts` (`fetchCurrentUser(): Promise<User>` and `updateProfile(input: { name: string; title: string }): Promise<User>` live in `src/features/profile/api.ts`)

```ts
import { useQuery } from "@tanstack/react-query";
import { fetchCurrentUser } from "@/features/profile/api";

export const userQueryKey = ["user", "me"] as const;

export function useUser() {
  return useQuery({
    queryKey: userQueryKey,
    queryFn: fetchCurrentUser,
  });
}
```

`src/features/profile/components/account-view.tsx` (rendered by the server page `src/app/(app)/account/page.tsx`)

```tsx
"use client";

import { Skeleton } from "@/components/ui/skeleton";
import { ProfileCard } from "@/features/profile/components/profile-card";
import { SettingsForm } from "@/features/profile/components/settings-form";
import { useUser } from "@/features/profile/hooks/use-user";

export function AccountView() {
  const { data: user, isPending, isError } = useUser();

  if (isPending) return <Skeleton className="h-96 w-full" />;
  if (isError) return <p className="text-destructive text-sm">Could not load your account.</p>;

  return (
    <div className="grid gap-6 lg:grid-cols-[320px_1fr]">
      <ProfileCard user={user} />
      <SettingsForm user={user} />
    </div>
  );
}
```

`src/features/profile/components/profile-card.tsx`

```tsx
"use client";

import { useState } from "react";
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar";
import { Badge } from "@/components/ui/badge";
import { Card, CardContent } from "@/components/ui/card";
import type { User } from "@/features/profile/types";
import { initials } from "@/lib/format";

type ProfileCardProps = {
  user: User;
};

export function ProfileCard({ user }: ProfileCardProps) {
  // TODO(PROF-212): inline rename from the card.
  const [name, setName] = useState(user.name);

  return (
    <Card>
      <CardContent className="flex flex-col items-center gap-3 pt-6 text-center">
        <Avatar className="size-20">
          <AvatarImage src={user.avatarUrl ?? undefined} alt="" />
          <AvatarFallback>{initials(name)}</AvatarFallback>
        </Avatar>
        <div className="flex flex-col gap-1">
          <p className="text-lg font-semibold">{name}</p>
          {user.title && <p className="text-muted-foreground text-sm">{user.title}</p>}
          <p className="text-muted-foreground text-sm">{user.email}</p>
        </div>
        <Badge variant="outline" className="capitalize">
          {user.role}
        </Badge>
      </CardContent>
    </Card>
  );
}
```

`src/features/profile/components/settings-form.tsx`

```tsx
"use client";

import { useMutation, useQueryClient } from "@tanstack/react-query";
import { toast } from "sonner";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { updateProfile } from "@/features/profile/api";
import { userQueryKey } from "@/features/profile/hooks/use-user";
import type { User } from "@/features/profile/types";

type SettingsFormProps = {
  user: User;
};

export function SettingsForm({ user }: SettingsFormProps) {
  const queryClient = useQueryClient();
  const mutation = useMutation({
    mutationFn: updateProfile,
    onSuccess: () => {
      toast.success("Profile updated");
      queryClient.invalidateQueries({ queryKey: userQueryKey });
    },
    onError: () => {
      toast.error("Could not save. Try again.");
    },
  });

  function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault();
    const form = new FormData(event.currentTarget);
    mutation.mutate({
      name: String(form.get("name") ?? ""),
      title: String(form.get("title") ?? ""),
    });
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle>Settings</CardTitle>
      </CardHeader>
      <CardContent>
        <form onSubmit={handleSubmit} className="flex flex-col gap-4">
          <div className="flex flex-col gap-2">
            <Label htmlFor="name">Display name</Label>
            <Input id="name" name="name" defaultValue={user.name} required />
          </div>
          <div className="flex flex-col gap-2">
            <Label htmlFor="title">Job title</Label>
            <Input id="title" name="title" defaultValue={user.title ?? ""} />
          </div>
          <Button type="submit" className="self-start" disabled={mutation.isPending}>
            Save
          </Button>
        </form>
      </CardContent>
    </Card>
  );
}
```

## Change request

"Bug: after renaming in Settings, the profile card still shows the old name until refresh. Steps: open `/account`, change Display name in the Settings card, press Save. The toast says "Profile updated" and the form keeps the new name, but the profile card on the left keeps the old name (and the old initials in the avatar) until a hard reload. Changing Job title shows up in the card straight away, it is only the name that sticks."

Make the change and explain in one or two sentences how you decided where the value lives.
