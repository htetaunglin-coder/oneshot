# T09 — button-look-on-link

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui).

`src/components/ui/button.tsx` (excerpt)

```tsx
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

export const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground shadow hover:bg-primary/90",
        outline: "border border-input bg-background shadow-sm hover:bg-accent hover:text-accent-foreground",
        ghost: "hover:bg-accent hover:text-accent-foreground",
      },
      size: {
        default: "h-9 px-4 py-2",
        sm: "h-8 rounded-md px-3 text-xs",
        lg: "h-10 rounded-md px-8",
      },
    },
    defaultVariants: { variant: "default", size: "default" },
  }
);

export type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> &
  VariantProps<typeof buttonVariants>;

export function Button({ className, variant, size, ...props }: ButtonProps) {
  return <button className={cn(buttonVariants({ variant, size }), className)} {...props} />;
}
```

`src/features/account/components/manage-plan-link.tsx`

```tsx
import Link from "next/link";

export function ManagePlanLink() {
  return (
    <Link
      href="/account/plan"
      className="inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring border border-input bg-background shadow-sm hover:bg-accent hover:text-accent-foreground h-9 px-4 py-2"
    >
      Manage plan
    </Link>
  );
}
```

Last sprint the button's focus ring was changed to `focus-visible:ring-2`, and the link was not updated, so the two now look slightly different on keyboard focus.

## Change request

Make `ManagePlanLink` look exactly like an outline button at the default size, and ensure it stays identical to the button whenever the button's styles change in future. It must stay a `<Link>` for prefetching.

Make the change and explain in one or two sentences how you structured the classes.
