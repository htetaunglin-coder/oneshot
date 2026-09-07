# C08 — section-dividers

## Scenario

A Next.js App Router project (React, TypeScript, Tailwind v4, shadcn/ui). The `onboarding` feature has a company form.

`src/features/onboarding/lib/countries.ts` exports `COUNTRIES: { code: string; name: string; eu: boolean }[]`.

`src/features/onboarding/components/company-form.tsx`

```tsx
"use client";

import { useState } from "react";
import { useRouter } from "next/navigation";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import { createCompany } from "../actions/create-company";
import { COUNTRIES } from "../lib/countries";

// ---- types ----

type FormErrors = Partial<
  Record<"name" | "country" | "website" | "vatNumber" | "form", string>
>;

export function CompanyForm() {
  const router = useRouter();

  // ---- state ----
  const [name, setName] = useState("");
  const [country, setCountry] = useState("");
  const [website, setWebsite] = useState("");
  const [vatNumber, setVatNumber] = useState("");
  const [errors, setErrors] = useState<FormErrors>({});
  const [submitting, setSubmitting] = useState(false);

  // ---- handlers ----
  async function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault();

    // Step 1: validate
    const nextErrors: FormErrors = {};
    if (name.trim().length < 2) {
      nextErrors.name = "Company name must be at least 2 characters.";
    }
    if (!country) {
      nextErrors.country = "Choose a country.";
    }
    if (website && !/^https?:\/\/.+\..+/.test(website)) {
      nextErrors.website = "Website must start with http:// or https://.";
    }
    if (Object.keys(nextErrors).length > 0) {
      setErrors(nextErrors);
      return;
    }
    setErrors({});

    // Step 2: build payload
    const payload = {
      name: name.trim(),
      country,
      website: website.trim() || null,
      vatNumber: vatNumber.trim() || null,
    };

    // Step 3: submit
    setSubmitting(true);
    const result = await createCompany(payload);
    setSubmitting(false);
    if (!result.ok) {
      setErrors({ form: result.error });
      return;
    }
    router.push(`/companies/${result.id}`);
  }

  // ---- render ----
  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div className="space-y-1">
        <Label htmlFor="name">Company name</Label>
        <Input id="name" value={name} onChange={(e) => setName(e.target.value)} />
        {errors.name && <p className="text-xs text-destructive">{errors.name}</p>}
      </div>

      <div className="space-y-1">
        <Label htmlFor="country">Country</Label>
        <Select value={country} onValueChange={setCountry}>
          <SelectTrigger id="country">
            <SelectValue placeholder="Select a country" />
          </SelectTrigger>
          <SelectContent>
            {COUNTRIES.map((c) => (
              <SelectItem key={c.code} value={c.code}>
                {c.name}
              </SelectItem>
            ))}
          </SelectContent>
        </Select>
        {errors.country && <p className="text-xs text-destructive">{errors.country}</p>}
      </div>

      <div className="space-y-1">
        <Label htmlFor="website">Website</Label>
        <Input
          id="website"
          value={website}
          onChange={(e) => setWebsite(e.target.value)}
          placeholder="https://"
        />
        {errors.website && <p className="text-xs text-destructive">{errors.website}</p>}
      </div>

      <div className="space-y-1">
        <Label htmlFor="vatNumber">VAT number</Label>
        <Input
          id="vatNumber"
          value={vatNumber}
          onChange={(e) => setVatNumber(e.target.value.toUpperCase())}
        />
        {errors.vatNumber && (
          <p className="text-xs text-destructive">{errors.vatNumber}</p>
        )}
      </div>

      {errors.form && <p className="text-sm text-destructive">{errors.form}</p>}

      <Button type="submit" disabled={submitting}>
        {submitting ? "Creating…" : "Create company"}
      </Button>
    </form>
  );
}
```

## Change request

When the selected country has `eu: true`, the VAT number is required and must match `/^[A-Z]{2}[0-9A-Z]{2,12}$/`. Show "VAT number is required for EU companies." when it is empty and "Enter a valid EU VAT number." when it does not match. Non-EU countries keep the field optional.

Make the change and explain in one or two sentences how you decided what to comment.
