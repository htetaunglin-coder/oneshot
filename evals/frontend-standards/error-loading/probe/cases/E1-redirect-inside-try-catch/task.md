# E1 — redirect-inside-try-catch

## Scenario

A Next.js App Router project (React 19, TypeScript, Tailwind v4, shadcn/ui), organised as `src/features/<x>/`, `src/components/ui`, `src/lib`, `src/app`. Server components fetch their own data; mutations are server actions in `src/features/<x>/actions.ts`, called from client forms through `useActionState`. `src/lib/db.ts` exports a Prisma client as `db`. `src/lib/auth.ts` exports `getSession(): Promise<Session | null>`, where `Session` has `userId` and `orgId`. Signed-in pages live under `src/app/(app)/`.

`src/features/invoices/actions.ts`

```ts
"use server";

import { revalidatePath } from "next/cache";
import { z } from "zod";
import { getSession } from "@/lib/auth";
import { db } from "@/lib/db";

const createInvoiceSchema = z.object({
  customerId: z.string().min(1, "Pick a customer"),
  amountCents: z.coerce.number().int().positive("Amount must be above zero"),
  dueDate: z.coerce.date(),
  memo: z.string().max(500, "Memo is too long").optional(),
});

type CreateInvoiceFields = z.infer<typeof createInvoiceSchema>;

export type CreateInvoiceState = {
  error?: string;
  fieldErrors?: Partial<Record<keyof CreateInvoiceFields, string>>;
  savedId?: string;
};

export async function createInvoice(
  _prev: CreateInvoiceState,
  formData: FormData,
): Promise<CreateInvoiceState> {
  const session = await getSession();
  if (!session) return { error: "Sign in to create invoices" };

  const parsed = createInvoiceSchema.safeParse({
    customerId: formData.get("customerId"),
    amountCents: formData.get("amountCents"),
    dueDate: formData.get("dueDate"),
    memo: formData.get("memo") || undefined,
  });

  if (!parsed.success) {
    const fieldErrors: CreateInvoiceState["fieldErrors"] = {};
    for (const issue of parsed.error.issues) {
      const field = issue.path[0] as keyof CreateInvoiceFields;
      fieldErrors[field] ??= issue.message;
    }
    return { fieldErrors };
  }

  try {
    const invoice = await db.invoice.create({
      data: {
        ...parsed.data,
        orgId: session.orgId,
        createdById: session.userId,
        status: "draft",
      },
      select: { id: true },
    });
    revalidatePath("/invoices");
    return { savedId: invoice.id };
  } catch (e) {
    console.error("createInvoice failed", e);
    return { error: "Could not save" };
  }
}
```

`src/features/invoices/components/new-invoice-form.tsx`

```tsx
"use client";

import { useActionState } from "react";
import Link from "next/link";
import { Alert, AlertDescription } from "@/components/ui/alert";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { CustomerSelect } from "@/features/customers/components/customer-select";
import { createInvoice, type CreateInvoiceState } from "@/features/invoices/actions";

const initialState: CreateInvoiceState = {};

export function NewInvoiceForm() {
  const [state, formAction, pending] = useActionState(createInvoice, initialState);

  return (
    <form action={formAction} className="flex max-w-lg flex-col gap-4">
      {state.error && (
        <Alert variant="destructive">
          <AlertDescription>{state.error}</AlertDescription>
        </Alert>
      )}
      {state.savedId && (
        <Alert>
          <AlertDescription>
            Saved as draft.{" "}
            <Link href={`/invoices/${state.savedId}`} className="underline">
              Open invoice
            </Link>
          </AlertDescription>
        </Alert>
      )}

      <div className="flex flex-col gap-1.5">
        <Label htmlFor="customerId">Customer</Label>
        <CustomerSelect id="customerId" name="customerId" />
        <FieldError message={state.fieldErrors?.customerId} />
      </div>

      <div className="flex flex-col gap-1.5">
        <Label htmlFor="amountCents">Amount (cents)</Label>
        <Input id="amountCents" name="amountCents" type="number" inputMode="numeric" />
        <FieldError message={state.fieldErrors?.amountCents} />
      </div>

      <div className="flex flex-col gap-1.5">
        <Label htmlFor="dueDate">Due date</Label>
        <Input id="dueDate" name="dueDate" type="date" />
        <FieldError message={state.fieldErrors?.dueDate} />
      </div>

      <div className="flex flex-col gap-1.5">
        <Label htmlFor="memo">Memo</Label>
        <Textarea id="memo" name="memo" rows={3} />
        <FieldError message={state.fieldErrors?.memo} />
      </div>

      <Button type="submit" disabled={pending}>
        {pending ? "Saving…" : "Save draft"}
      </Button>
    </form>
  );
}

function FieldError({ message }: { message?: string }) {
  if (!message) return null;
  return <p className="text-sm text-destructive">{message}</p>;
}
```

The form is rendered by `src/app/(app)/invoices/new/page.tsx`. `src/app/(app)/invoices/[id]/page.tsx` exists and shows one invoice by id; `src/app/(app)/invoices/page.tsx` lists them.

## Change request

"After 'Save draft' succeeds, take the user straight to the new invoice's page at `/invoices/<id>` instead of showing the 'Saved as draft' line with a link — people miss the link and save the same invoice twice. Validation errors and the 'Could not save' message must keep working exactly as they do now."

Make the change and explain in one or two sentences how you decided where the failure or waiting state is handled.
