# S2 — long-enterprise-block

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The settings feature shows a plan summary card. Enterprise workspaces get an extra section with contract details under the common rows. The section is plain markup fed by props: no state, no hooks, no handlers, nothing conditional inside it.

`src/features/settings/components/plan-summary.tsx`

```tsx
import { Badge } from "@/components/ui/badge";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Separator } from "@/components/ui/separator";
import { formatDate } from "@/lib/format";
import type { Workspace } from "@/features/settings/lib/types";

type PlanSummaryProps = {
  workspace: Workspace;
};

export function PlanSummary({ workspace }: PlanSummaryProps) {
  const { plan, seats, renewsAt, contract } = workspace;

  return (
    <Card>
      <CardHeader className="flex flex-row items-center justify-between">
        <CardTitle>Plan</CardTitle>
        <Badge variant="secondary" className="capitalize">
          {plan}
        </Badge>
      </CardHeader>
      <CardContent className="flex flex-col gap-4">
        <dl className="grid grid-cols-2 gap-y-2 text-sm">
          <dt className="text-muted-foreground">Seats</dt>
          <dd>{seats.used} of {seats.total}</dd>
          <dt className="text-muted-foreground">Renews</dt>
          <dd>{formatDate(renewsAt)}</dd>
        </dl>

        {plan === "enterprise" ? (
          <section className="flex flex-col gap-3">
            <Separator />
            <h3 className="text-sm font-medium">Contract</h3>
            <dl className="grid grid-cols-2 gap-y-2 text-sm">
              <dt className="text-muted-foreground">Contract ID</dt>
              <dd className="font-mono">{contract.id}</dd>
              <dt className="text-muted-foreground">Term</dt>
              <dd>
                {formatDate(contract.startsAt)} to {formatDate(contract.endsAt)}
              </dd>
              <dt className="text-muted-foreground">Billing cadence</dt>
              <dd className="capitalize">{contract.cadence}</dd>
              <dt className="text-muted-foreground">Payment terms</dt>
              <dd>Net {contract.paymentTermDays}</dd>
              <dt className="text-muted-foreground">Account manager</dt>
              <dd>
                <a href={`mailto:${contract.accountManager.email}`} className="underline">
                  {contract.accountManager.name}
                </a>
              </dd>
              <dt className="text-muted-foreground">Data region</dt>
              <dd>{contract.dataRegion}</dd>
              <dt className="text-muted-foreground">Audit log retention</dt>
              <dd>{contract.auditRetentionDays} days</dd>
              <dt className="text-muted-foreground">Custom domain</dt>
              <dd>{contract.customDomain ?? "Not configured"}</dd>
            </dl>
            <p className="text-muted-foreground text-xs">
              Contract changes are handled by your account manager.
            </p>
          </section>
        ) : null}
      </CardContent>
    </Card>
  );
}
```

`Workspace["contract"]` also carries `sso: { provider: string; enforced: boolean }` and `supportChannel: { label: string; href: string }`, both unused so far. A reviewer left one comment on the last PR that touched this file: "the enterprise part is getting long, hard to scan the card at a glance."

## Change request

"Add two rows to the end of the contract list: 'SSO' showing `contract.sso.provider` with '(enforced)' appended when `contract.sso.enforced` is true, and 'Support channel' as a link to `contract.supportChannel.href` with `contract.supportChannel.label` as the text. While you are in there, please address the reviewer's comment about readability in whatever way you think is right."

Make the change and explain in one or two sentences how you shaped the conditional rendering.
