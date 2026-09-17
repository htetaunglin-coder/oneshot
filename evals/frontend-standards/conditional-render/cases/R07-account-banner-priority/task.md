# R07 — account-banner-priority

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`.

`src/features/account/lib/types.ts`

```ts
export type Account = {
  id: string;
  name: string;
  isTrial: boolean;
  trialDaysLeft: number;
  isPastDue: boolean;
  hasUnverifiedEmail: boolean;
  isSuspended: boolean; // added by the backend team last sprint, not yet used in the UI
};
```

`src/features/account/components/account-header.tsx`

```tsx
import Link from "next/link";
import { Alert, AlertDescription, AlertTitle } from "@/components/ui/alert";
import type { Account } from "../lib/types";

type AccountHeaderProps = {
  account: Account;
};

export function AccountHeader({ account }: AccountHeaderProps) {
  const { isTrial, trialDaysLeft, isPastDue, hasUnverifiedEmail } = account;

  return (
    <div className="flex flex-col gap-4">
      {isPastDue ? (
        <Alert variant="destructive">
          <AlertTitle>Payment failed</AlertTitle>
          <AlertDescription>
            Update your card to keep your workspace active.{" "}
            <Link href="/billing" className="underline">
              Go to billing
            </Link>
          </AlertDescription>
        </Alert>
      ) : hasUnverifiedEmail ? (
        <Alert>
          <AlertTitle>Verify your email</AlertTitle>
          <AlertDescription>Check your inbox for the verification link.</AlertDescription>
        </Alert>
      ) : isTrial ? (
        <Alert>
          <AlertTitle>Trial</AlertTitle>
          <AlertDescription>
            {trialDaysLeft} days left.{" "}
            <Link href="/billing" className="underline">
              Choose a plan
            </Link>
          </AlertDescription>
        </Alert>
      ) : null}
      <h1 className="text-2xl font-semibold">{account.name}</h1>
    </div>
  );
}
```

## Change request

Suspended accounts must see a banner. When `isSuspended` is true, show a `destructive` alert titled "Workspace suspended" with the description "Contact support to restore access." and a link to `/support` labelled "Contact support". Only one banner shows at a time, and the order of precedence is now: suspended, then past due, then unverified email, then trial. Everything else in the header stays as it is.

Make the change and explain in one or two sentences how you shaped the conditional rendering.
