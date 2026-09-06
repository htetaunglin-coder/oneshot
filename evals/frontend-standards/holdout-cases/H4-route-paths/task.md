# H4 — Route paths used by a feature, the nav, and middleware

## Scenario

Billing pages are linked from three places that do not share code. The billing feature keeps its own path map. The main nav hard-codes a string. Middleware hard-codes both the matcher and the redirect.

```ts
// src/features/billing/lib/routes.ts
export const billingRoutes = {
  overview: "/billing",
  invoices: "/billing/invoices",
  invoice: (id: string) => `/billing/invoices/${id}`,
  paymentMethods: "/billing/payment-methods",
} as const;
```

```tsx
// src/features/billing/components/invoice-row.tsx (excerpt)
import { billingRoutes } from "@/features/billing/lib/routes";

<Link href={billingRoutes.invoice(invoice.id)} className="underline">
  {invoice.number}
</Link>
```

```tsx
// src/components/main-nav.tsx (excerpt)
const items = [
  { label: "Dashboard", href: "/dashboard" },
  { label: "Projects", href: "/projects" },
  { label: "Billing", href: "/billing" },
  { label: "Settings", href: "/settings" },
];
```

```ts
// src/middleware.ts
import { NextResponse, type NextRequest } from "next/server";
import { getSession } from "@/lib/auth/session";

export async function middleware(req: NextRequest) {
  const session = await getSession(req);
  if (!session) {
    const login = new URL("/login", req.url);
    login.searchParams.set("next", req.nextUrl.pathname);
    return NextResponse.redirect(login);
  }
  if (req.nextUrl.pathname.startsWith("/billing") && session.role !== "owner") {
    return NextResponse.redirect(new URL("/dashboard", req.url));
  }
  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*", "/projects/:path*", "/billing/:path*", "/settings/:path*"],
};
```

The route segments themselves live under `src/app/(app)/billing/...`.

## Change request

Billing moves under account: every `/billing/...` path becomes `/account/billing/...`. The team wants this to be the last time three files have to be edited by hand for a path change. Decide where the new/changed code lives and explain in one or two sentences.
