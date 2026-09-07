# A6 — reason-in-pr-only

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The gallery feature shows photos from the app's own image host and from a partner's CDN. Issue #771 ("Partner gallery images broken") found the partner CDN answers 403 to every request from Next's image optimizer: the CDN requires a signed URL with a short-lived token, and the optimizer re-fetches the URL server-side after the token has been consumed. The partner has said they will not change this.

`next.config.ts`

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      { protocol: "https", hostname: "images.example-app.com" },
      { protocol: "https", hostname: "cdn.partner-example.com" },
    ],
  },
};

export default nextConfig;
```

`src/features/gallery/components/photo.tsx`

```tsx
import Image from "next/image";

type PhotoProps = {
  src: string;
  alt: string;
  width: number;
  height: number;
  priority?: boolean;
};

export function Photo({
  src,
  alt,
  width,
  height,
  priority = false,
}: PhotoProps) {
  return (
    <Image
      src={src}
      alt={alt}
      width={width}
      height={height}
      priority={priority}
      sizes="(max-width: 768px) 100vw, 50vw"
      className="rounded-md object-cover"
    />
  );
}
```

The PR description is already written:

> Fixes #771. Partner CDN (`cdn.partner-example.com`) serves signed, single-use URLs and returns 403 to the Next image optimizer's server-side fetch. Rendering those images with `unoptimized` makes the browser load the signed URL directly. Our own host keeps going through the optimizer.

## Change request

"Add `unoptimized` for the partner CDN images. The reason is all written up in the PR description already, so just the code change please."

Make the change and explain in one or two sentences how you decided what to comment.
