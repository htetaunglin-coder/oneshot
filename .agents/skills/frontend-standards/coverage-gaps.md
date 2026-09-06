# Coverage gaps

Places the rules are silent or answered by analogy. Source: eval agents' "unclear or missing" reports. An entry leaves when a rule covers it or a decision says it stays out.

## colocation

Open
- App-layer client glue (a `"use client"` file that reads one feature's hook and feeds another feature's props): lives in the route segment beside `page.tsx`. Not yet in the rule. H2 run 2.
- `import type` across features: default is no; the consumer declares the prop shape it needs. Not yet in the rule. H2 run 2.
- Under-limit file: "earns the extraction" reads as may, not must. Held until it fails a case.
- Boundaries enforcement on Biome/Ultracite repos: no `eslint-plugin-boundaries` equivalent, only `noRestrictedImports`. Tooling note, not rule text.

Out of scope, decided
- Hook shape when the caller supplies the varying part (argument vs options vs configurator). H1.
- Extracted-file naming; where "the size is reported". Formatting.
- `src/lib` sub-folder naming. Repo convention.

Closed 2026-09-07 by the Layers section and scope edits
- Cross-feature imports, domain state consumed by several features, route handler as a second consumer, config/constants scope, test utilities scope, `"use client"` leaf in a server component.
