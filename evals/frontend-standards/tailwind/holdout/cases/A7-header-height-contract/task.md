# Task

## Codebase

React + TypeScript, Next.js App Router, Tailwind v4, shadcn/ui. Feature code lives in `src/features/<name>/components`, layout in `src/components/layout`, shared primitives in `src/components/ui`, `cn` is exported from `src/lib/utils`, theme lives in `src/app/globals.css`.

### `src/components/layout/app-shell.tsx`

```tsx
import { Header } from "./header";

type AppShellProps = {
  children: React.ReactNode;
};

export function AppShell({ children }: AppShellProps) {
  return (
    <div className="min-h-dvh bg-background">
      <Header />
      <main className="pt-14">{children}</main>
    </div>
  );
}
```

### `src/components/layout/header.tsx`

```tsx
import Link from "next/link";
import { UserMenu } from "@/features/auth/components/user-menu";

export function Header() {
  return (
    <header className="fixed inset-x-0 top-0 z-40 flex h-14 items-center border-b bg-background/95 px-4 backdrop-blur">
      <Link href="/" className="font-semibold">
        Acme
      </Link>
      <div className="ml-auto">
        <UserMenu />
      </div>
    </header>
  );
}
```

### `src/features/docs/components/toc-sidebar.tsx`

```tsx
import type { Heading } from "../types";

export function TocSidebar({ headings }: { headings: Heading[] }) {
  return (
    <nav aria-label="On this page" className="sticky top-14 hidden max-h-[calc(100dvh-3.5rem)] overflow-y-auto xl:block">
      <ul className="space-y-1 text-sm text-muted-foreground">
        {headings.map((h) => (
          <li key={h.id}>
            <a href={`#${h.id}`} className="hover:text-foreground">
              {h.text}
            </a>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

### `src/features/chat/components/scroll-button.tsx`

```tsx
import { ArrowDown } from "lucide-react";
import { Button } from "@/components/ui/button";

export function ScrollButton({ onClick }: { onClick: () => void }) {
  return (
    <Button
      size="icon"
      variant="secondary"
      onClick={onClick}
      aria-label="Scroll to latest"
      className="fixed right-4 top-[calc(3.5rem+1rem)] z-30 rounded-full shadow"
    >
      <ArrowDown className="size-4" />
    </Button>
  );
}
```

## Request

The header must grow to `h-16` on `md:` screens and up (stay `h-14` below that). Everything that sits under or beside the header (the `main` padding, the docs table of contents, the chat scroll button) must follow the header's height at every breakpoint so nothing overlaps or drifts.

Make the change and explain in one or two sentences how you structured the classes.
