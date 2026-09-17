# M01 — retry-delay-cap

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui); features live under `src/features/<name>/components|hooks|lib`, shared code under `src/lib`.

`src/lib/sleep.ts`

```ts
export function sleep(ms: number): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, ms));
}
```

`src/lib/fetch-with-retry.ts`

```ts
import { sleep } from "@/lib/sleep";

type RetryOptions = {
  signal?: AbortSignal;
  onRetry?: (attempt: number, delayMs: number) => void;
};

export async function fetchWithRetry(
  input: RequestInfo | URL,
  init: RequestInit = {},
  options: RetryOptions = {},
): Promise<Response> {
  let attempts = 0;

  while (true) {
    attempts += 1;
    const response = await fetch(input, { ...init, signal: options.signal });

    if (response.status !== 429) {
      return response;
    }

    if (attempts >= 5) {
      throw new Error(`Rate limited after ${attempts} attempts`);
    }

    const retryAfterSeconds = Number(response.headers.get("Retry-After"));
    const delayMs =
      retryAfterSeconds > 0 ? retryAfterSeconds * 1000 : 250 * attempts;

    options.onRetry?.(attempts, delayMs);
    await sleep(delayMs);
  }
}
```

`src/features/search/lib/search-client.ts`

```ts
import { fetchWithRetry } from "@/lib/fetch-with-retry";
import type { SearchResult } from "@/features/search/lib/types";

export async function searchProducts(
  query: string,
  signal?: AbortSignal,
): Promise<SearchResult[]> {
  const url = `/api/search?q=${encodeURIComponent(query)}`;
  const response = await fetchWithRetry(url, {}, { signal });

  if (!response.ok) {
    throw new Error(`Search failed with status ${response.status}`);
  }

  return (await response.json()) as SearchResult[];
}
```

## Change request

A staging API answered `Retry-After: 120` and the search box sat frozen for two minutes. Cap the wait between attempts at three seconds, no matter what the header asks for; a header value below the cap is still honoured. Report the capped value through `onRetry`.

Make the change and explain in one or two sentences how you decided which values get a name.
