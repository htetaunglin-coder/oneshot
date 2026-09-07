# C10 — business-rule

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The `orders` feature decides whether an order goes through manager approval.

`src/features/orders/lib/approval.ts`

```ts
import type { Order } from "../types";

const APPROVER_ROLES = new Set(["manager", "admin"]);

export function needsManagerApproval(order: Order): boolean {
  if (order.totalCents < 25_000) {
    return false;
  }
  if (APPROVER_ROLES.has(order.placedBy.role)) {
    return false;
  }
  return true;
}

export function canApprove(order: Order, approverRole: string): boolean {
  return needsManagerApproval(order) && APPROVER_ROLES.has(approverRole);
}
```

Ticket ORD-731, filed by the team lead:

> It should be noted that basically, when it comes to the approval workflow, a decision was made at some point by the Finance team (this is captured, as far as we are aware, in policy FIN-12, which was updated at some point last year I believe) to the effect that orders whose total comes in at less than $250 are, generally speaking, not required to be routed through the manager approval step, the reasoning being that the cost of a review would in those cases be considered to outweigh the risk that is involved. It has been requested that the code be made clearer in this regard for whoever happens to look at it next, given that a new hire asked about the number last week and it was found that nobody could recall where it had come from.

## Change request

Resolve ORD-731: make the origin of the `25_000` threshold clear at the point in the code where it is applied. No behavior change.

Make the change and explain in one or two sentences how you decided what to comment.
