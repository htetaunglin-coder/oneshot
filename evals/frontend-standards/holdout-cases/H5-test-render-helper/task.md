# H5 — A test wrapper that a second feature's test needs

## Scenario

The cart drawer test defines its own render wrapper so the component under test gets a QueryClient and the theme provider. The helper is local to the test file.

```tsx
// src/features/cart/components/cart-drawer.test.tsx
import { render, screen, type RenderOptions } from "@testing-library/react";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ThemeProvider } from "next-themes";
import { describe, it, expect, vi } from "vitest";
import type { ReactElement, ReactNode } from "react";
import { CartDrawer } from "./cart-drawer";

function makeQueryClient() {
  return new QueryClient({
    defaultOptions: { queries: { retry: false, gcTime: 0 } },
  });
}

function Providers({ children }: { children: ReactNode }) {
  return (
    <QueryClientProvider client={makeQueryClient()}>
      <ThemeProvider attribute="class" defaultTheme="light" enableSystem={false}>
        {children}
      </ThemeProvider>
    </QueryClientProvider>
  );
}

function renderWithProviders(ui: ReactElement, options?: Omit<RenderOptions, "wrapper">) {
  return render(ui, { wrapper: Providers, ...options });
}

vi.mock("@/features/cart/api/get-cart", () => ({
  getCart: vi.fn().mockResolvedValue({ items: [{ sku: "A1", name: "Mug", qty: 2, unitPrice: 1200 }] }),
}));

describe("CartDrawer", () => {
  it("lists cart items", async () => {
    renderWithProviders(<CartDrawer open onOpenChange={() => {}} />);
    expect(await screen.findByText("Mug")).toBeInTheDocument();
  });

  it("shows the subtotal", async () => {
    renderWithProviders(<CartDrawer open onOpenChange={() => {}} />);
    expect(await screen.findByText("$24.00")).toBeInTheDocument();
  });
});
```

Project test setup: `vitest.config.ts` at the repo root, `environment: "jsdom"`, `setupFiles: ["./vitest.setup.ts"]`, and the `@/` alias mapped to `src/`. There is no test-utilities folder yet.

## Change request

`src/features/orders/components/order-list.test.tsx` is being written now and needs the same wrapper (QueryClient with retries off, theme provider). Decide where the new/changed code lives and explain in one or two sentences.
