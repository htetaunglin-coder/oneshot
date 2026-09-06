# H2 — A feature context that the global header and another feature need

## Scenario

Cart state lives in a single file inside the cart feature. The provider is currently mounted by the cart page only, and the only consumers are cart components.

```tsx
// src/features/cart/components/cart-provider.tsx
"use client";

import { createContext, useContext, useReducer, type ReactNode } from "react";

export type CartItem = { sku: string; name: string; qty: number; unitPrice: number };

type CartState = { items: CartItem[] };

type CartAction =
  | { type: "add"; item: CartItem }
  | { type: "remove"; sku: string }
  | { type: "setQty"; sku: string; qty: number }
  | { type: "clear" };

function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case "add": {
      const existing = state.items.find((i) => i.sku === action.item.sku);
      if (!existing) return { items: [...state.items, action.item] };
      return {
        items: state.items.map((i) =>
          i.sku === action.item.sku ? { ...i, qty: i.qty + action.item.qty } : i,
        ),
      };
    }
    case "remove":
      return { items: state.items.filter((i) => i.sku !== action.sku) };
    case "setQty":
      return { items: state.items.map((i) => (i.sku === action.sku ? { ...i, qty: action.qty } : i)) };
    case "clear":
      return { items: [] };
  }
}

type CartContextValue = CartState & {
  add: (item: CartItem) => void;
  remove: (sku: string) => void;
  setQty: (sku: string, qty: number) => void;
  clear: () => void;
  count: number;
};

const CartContext = createContext<CartContextValue | null>(null);

export function CartProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });
  const value: CartContextValue = {
    ...state,
    add: (item) => dispatch({ type: "add", item }),
    remove: (sku) => dispatch({ type: "remove", sku }),
    setQty: (sku, qty) => dispatch({ type: "setQty", sku, qty }),
    clear: () => dispatch({ type: "clear" }),
    count: state.items.reduce((n, i) => n + i.qty, 0),
  };
  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}

export function useCart() {
  const ctx = useContext(CartContext);
  if (!ctx) throw new Error("useCart must be used inside CartProvider");
  return ctx;
}
```

```tsx
// src/app/(shop)/cart/page.tsx
import { CartProvider } from "@/features/cart/components/cart-provider";
import { CartView } from "@/features/cart/components/cart-view";

export default function CartPage() {
  return (
    <CartProvider>
      <CartView />
    </CartProvider>
  );
}
```

```tsx
// src/components/site-header.tsx (rendered by src/app/layout.tsx)
export function SiteHeader() {
  return (
    <header className="flex items-center justify-between px-6 py-3">
      <Logo />
      <nav className="flex gap-4">
        <Link href="/products">Products</Link>
        <Link href="/cart">Cart</Link>
      </nav>
    </header>
  );
}
```

## Change request

The site header must show a badge with the cart item count on every page, and `src/features/checkout/components/order-summary.tsx` must read the cart line items and call `clear()` after a successful order. Decide where the new/changed code lives (provider, hook, reducer, types, and where the provider is mounted) and explain in one or two sentences.
