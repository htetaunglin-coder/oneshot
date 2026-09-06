# H3 — One validation rule, a client form, and a route handler

## Scenario

The signup form validates on the client with a Zod schema declared inline. The route handler that receives the POST validates again with its own inline schema. The two have already drifted: the form requires a password of 8 characters, the route requires 10.

```tsx
// src/features/signup/components/signup-form.tsx
"use client";

import { z } from "zod";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  displayName: z.string().min(2).max(40),
});

type FormValues = z.infer<typeof schema>;

export function SignupForm() {
  const form = useForm<FormValues>({ resolver: zodResolver(schema) });

  async function onSubmit(values: FormValues) {
    const res = await fetch("/api/signup", {
      method: "POST",
      headers: { "content-type": "application/json" },
      body: JSON.stringify(values),
    });
    if (!res.ok) form.setError("root", { message: "Signup failed" });
  }

  return (
    <form onSubmit={form.handleSubmit(onSubmit)} className="grid gap-3">
      <Input type="email" {...form.register("email")} />
      <Input type="password" {...form.register("password")} />
      <Input {...form.register("displayName")} />
      <Button type="submit">Create account</Button>
    </form>
  );
}
```

```ts
// src/app/api/signup/route.ts
import { NextResponse } from "next/server";
import { z } from "zod";
import { createUser } from "@/features/signup/api/create-user";

const bodySchema = z.object({
  email: z.string().email(),
  password: z.string().min(10),
  displayName: z.string().min(2).max(40),
});

export async function POST(req: Request) {
  const parsed = bodySchema.safeParse(await req.json());
  if (!parsed.success) {
    return NextResponse.json({ error: parsed.error.flatten() }, { status: 400 });
  }
  const user = await createUser(parsed.data);
  return NextResponse.json({ id: user.id }, { status: 201 });
}
```

## Change request

Make the form and the route handler validate with the same rules, settle the password minimum at 10, and add a rule that rejects emails whose domain is in a list of about forty disposable-mail domains (`mailinator.com`, `guerrillamail.com`, ...). The same error message must appear inline in the form and in the 400 response body. Decide where the new/changed code lives and explain in one or two sentences.
