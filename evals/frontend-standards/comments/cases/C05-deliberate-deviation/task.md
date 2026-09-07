# C05 — deliberate-deviation

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). Feedback is submitted through a server action. ESLint runs with `@typescript-eslint/no-floating-promises` enabled and CI fails on lint errors.

`src/features/feedback/actions/submit-feedback.ts`

```ts
"use server";

import { redirect } from "next/navigation";
import { z } from "zod";
import { getCurrentUser } from "@/lib/auth";
import { db } from "@/lib/db";

const feedbackSchema = z.object({
  rating: z.coerce.number().int().min(1).max(5),
  message: z.string().trim().max(2000).optional(),
});

export async function submitFeedback(formData: FormData) {
  const user = await getCurrentUser();
  if (!user) redirect("/login");

  const parsed = feedbackSchema.safeParse({
    rating: formData.get("rating"),
    message: formData.get("message"),
  });
  if (!parsed.success) {
    return { error: "Rating must be between 1 and 5." };
  }

  await db.feedback.create({
    data: { userId: user.id, ...parsed.data },
  });

  redirect("/feedback/thanks");
}
```

`src/lib/analytics.ts`

```ts
export async function trackEvent(
  name: string,
  props: Record<string, string | number> = {},
): Promise<void> {
  const res = await fetch(process.env.ANALYTICS_URL!, {
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify({ name, props, ts: Date.now() }),
  });
  if (!res.ok) {
    throw new Error(`analytics: ${res.status}`);
  }
}
```

## Change request

Record a `feedback_submitted` event with the rating after the row is saved. The analytics endpoint has a p95 of about 800 ms and is occasionally down; the redirect must not wait for it and an analytics failure must not surface to the user or fail the action.

Make the change and explain in one or two sentences how you decided what to comment.
