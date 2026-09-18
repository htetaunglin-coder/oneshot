# P3 — derived-value-synced-by-effect

## Scenario

A Next.js App Router project (React 19, TypeScript, Tailwind v4, shadcn/ui), organised as `src/features/<x>/`, `src/components/ui`, `src/lib`. The cart is client-only and persisted with Zustand. `formatMoney(cents: number): string` in `src/lib/format.ts` renders `1299` as `$12.99`.

`src/features/cart/types.ts`

```ts
export type CartItem = {
  id: string;
  productId: string;
  name: string;
  unitCents: number;
  quantity: number;
  imageUrl: string;
};

export type Promo = {
  code: string;
  percentOff: number;
};
```

`src/features/cart/store.ts`

```ts
import { create } from "zustand";
import { persist } from "zustand/middleware";
import type { CartItem } from "@/features/cart/types";

type CartState = {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  updateQuantity: (id: string, quantity: number) => void;
  removeItem: (id: string) => void;
};

export const useCartStore = create<CartState>()(
  persist(
    (set) => ({
      items: [],
      addItem: (item) => set((s) => ({ items: [...s.items, item] })),
      updateQuantity: (id, quantity) =>
        set((s) => ({ items: s.items.map((i) => (i.id === id ? { ...i, quantity } : i)) })),
      removeItem: (id) => set((s) => ({ items: s.items.filter((i) => i.id !== id) })),
    }),
    { name: "cart" },
  ),
);
```

`src/features/cart/components/cart-summary.tsx`

```tsx
"use client";

import { useEffect, useState } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from "@/components/ui/card";
import { Separator } from "@/components/ui/separator";
import { CartLine } from "@/features/cart/components/cart-line";
import { useCartStore } from "@/features/cart/store";
import { formatMoney } from "@/lib/format";

const SHIPPING_CENTS = 599;

export function CartSummary() {
  const items = useCartStore((s) => s.items);
  const updateQuantity = useCartStore((s) => s.updateQuantity);
  const removeItem = useCartStore((s) => s.removeItem);
  const [subtotal, setSubtotal] = useState(0);
  const [total, setTotal] = useState(0);

  useEffect(() => {
    const next = items.reduce((sum, item) => sum + item.unitCents * item.quantity, 0);
    setSubtotal(next);
    setTotal(next + SHIPPING_CENTS);
  }, [items]);

  return (
    <Card>
      <CardHeader>
        <CardTitle>Your cart</CardTitle>
      </CardHeader>
      <CardContent className="flex flex-col gap-3">
        {items.map((item) => (
          <CartLine
            key={item.id}
            item={item}
            onQuantityChange={(quantity) => updateQuantity(item.id, quantity)}
            onRemove={() => removeItem(item.id)}
          />
        ))}
        <Separator />
        <dl className="grid grid-cols-[1fr_auto] gap-y-1 text-sm">
          <dt className="text-muted-foreground">Subtotal</dt>
          <dd>{formatMoney(subtotal)}</dd>
          <dt className="text-muted-foreground">Shipping</dt>
          <dd>{formatMoney(SHIPPING_CENTS)}</dd>
          <dt className="font-medium">Total</dt>
          <dd className="font-medium">{formatMoney(total)}</dd>
        </dl>
      </CardContent>
      <CardFooter>
        <Button className="w-full" disabled={items.length === 0}>
          Checkout
        </Button>
      </CardFooter>
    </Card>
  );
}
```

`src/features/cart/components/promo-code-input.tsx` already exists. `validatePromo(code: string): Promise<Promo>` in `src/features/cart/api.ts` resolves with the promo or rejects when the code is unknown or expired.

```tsx
"use client";

import { useState } from "react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { validatePromo } from "@/features/cart/api";
import type { Promo } from "@/features/cart/types";

type PromoCodeInputProps = {
  onApply: (promo: Promo) => void;
};

export function PromoCodeInput({ onApply }: PromoCodeInputProps) {
  const [code, setCode] = useState("");
  const [error, setError] = useState<string | null>(null);
  const [pending, setPending] = useState(false);

  async function handleApply() {
    setPending(true);
    setError(null);
    try {
      const promo = await validatePromo(code.trim());
      onApply(promo);
      setCode("");
    } catch {
      setError("That code is not valid.");
    } finally {
      setPending(false);
    }
  }

  return (
    <div className="flex flex-col gap-1">
      <div className="flex gap-2">
        <Input
          value={code}
          onChange={(e) => setCode(e.target.value)}
          placeholder="Promo code"
          aria-label="Promo code"
        />
        <Button variant="outline" onClick={handleApply} disabled={pending || code.trim() === ""}>
          Apply
        </Button>
      </div>
      {error && <p className="text-destructive text-xs">{error}</p>}
    </div>
  );
}
```

## Change request

"Promo codes on the cart. Mount `PromoCodeInput` above the totals in `CartSummary`. When a code is applied, add a line between Subtotal and Shipping labelled `Discount (CODE)` that shows the amount as a negative, and the Total must come down by the same amount. Discount is `percentOff` of the subtotal, rounded to the nearest cent. One code at a time; applying another replaces it. Shipping is never discounted."

Make the change and explain in one or two sentences how you decided where the value lives.
