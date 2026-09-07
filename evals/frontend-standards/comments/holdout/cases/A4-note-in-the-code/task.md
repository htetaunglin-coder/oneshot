# A4 — note-in-the-code

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). Last week a customer was charged twice (incident INC-2041, tracked as issue #689). The client retried `placeOrder` after a timeout; the retry created a second `order` row with a new id, so the payment provider saw a new idempotency key and charged again.

`src/features/checkout/actions/place-order.ts`

```ts
"use server";

import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";
import { auth } from "@/lib/auth";
import { db } from "@/lib/db";
import { payments } from "@/lib/payments";

type PlaceOrderInput = {
  cartId: string;
};

export async function placeOrder({ cartId }: PlaceOrderInput) {
  const session = await auth();
  if (!session) {
    throw new Error("Unauthorized");
  }

  const cart = await db.cart.findUnique({
    where: { id: cartId, userId: session.user.id },
    include: { items: true },
  });
  if (!cart || cart.items.length === 0) {
    return { error: "Your cart is empty." } as const;
  }

  const totalCents = cart.items.reduce(
    (sum, item) => sum + item.unitPriceCents * item.quantity,
    0,
  );

  const order = await db.order.create({
    data: {
      userId: session.user.id,
      cartId: cart.id,
      status: "pending",
      totalCents,
    },
  });

  const charge = await payments.charge({
    customerId: session.user.paymentCustomerId,
    amountCents: totalCents,
    idempotencyKey: order.id,
  });

  await db.order.update({
    where: { id: order.id },
    data: { status: "paid", chargeId: charge.id },
  });
  await db.cart.delete({ where: { id: cart.id } });

  revalidatePath("/orders");
  redirect(`/orders/${order.id}`);
}
```

`payments.charge` forwards `idempotencyKey` to the provider, which returns the original charge for a repeated key within 24 hours. The cart is deleted after a successful charge, so a cart id is never reused for a second purchase.

## Change request

"Switch the idempotency key from `orderId` to `cartId + userId` (there was a double-charge when retries created a new orderId). Please leave a note in the code saying what you changed and when, so reviewers and future readers see it."

Make the change and explain in one or two sentences how you decided what to comment.
