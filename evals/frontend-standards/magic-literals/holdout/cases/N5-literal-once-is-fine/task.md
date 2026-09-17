# N5 — literal-once-is-fine

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The auth feature is adding a "Resend code" button to the email verification screen. The button is disabled until a countdown reaches zero, then it becomes clickable and restarts the countdown when pressed.

The feature's other hook, for reference on how hooks in this folder are written:

`src/features/auth/hooks/use-session-timeout.ts`

```ts
"use client";

import { useEffect } from "react";
import { useRouter } from "next/navigation";

const IDLE_LIMIT_MS = 15 * 60 * 1000;

export function useSessionTimeout() {
  const router = useRouter();

  useEffect(() => {
    let timer = setTimeout(() => router.push("/login?reason=idle"), IDLE_LIMIT_MS);

    function reset() {
      clearTimeout(timer);
      timer = setTimeout(() => router.push("/login?reason=idle"), IDLE_LIMIT_MS);
    }

    window.addEventListener("pointerdown", reset);
    window.addEventListener("keydown", reset);

    return () => {
      clearTimeout(timer);
      window.removeEventListener("pointerdown", reset);
      window.removeEventListener("keydown", reset);
    };
  }, [router]);
}
```

The verification screen will call the new hook like this:

`src/features/auth/components/resend-code-button.tsx` (already written)

```tsx
"use client";

import { Button } from "@/components/ui/button";
import { useCountdown } from "@/features/auth/hooks/use-countdown";

type ResendCodeButtonProps = {
  onResend: () => Promise<void>;
};

export function ResendCodeButton({ onResend }: ResendCodeButtonProps) {
  const { remaining, restart } = useCountdown(45);

  async function handleClick() {
    await onResend();
    restart();
  }

  return (
    <Button variant="ghost" size="sm" disabled={remaining > 0} onClick={handleClick}>
      {remaining > 0 ? `Resend code in ${remaining}s` : "Resend code"}
    </Button>
  );
}
```

## Change request

"Write `useCountdown(seconds)` in `src/features/auth/hooks/use-countdown.ts`. It returns `{ remaining, restart }`. `remaining` starts at `seconds`, goes down by one each second, never goes below zero, and stops ticking once it reaches zero. `restart()` sets it back to `seconds` and starts ticking again. Clean up the timer on unmount."

Make the change and explain in one or two sentences how you decided which values get a name.
