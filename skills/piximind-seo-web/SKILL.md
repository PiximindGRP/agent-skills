---
name: piximind-seo-web
description: Enforces Next.js SEO audit-loop — per-route generateMetadata, sitemap/robots, next-intl hreflang, RSC-indexable content, next/image alt, noindex on private routes. Use when adding App Router pages, metadata, locales, or crawling/indexability work in Next.js.
paths:
  - "**/*.{ts,tsx}"
  - "**/src/app/**"
---

# Next.js SEO

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

Public marketing pages ≠ authenticated admin. Auth-gated apps `noindex` private routes.

## Anti-patterns
- **Do NOT** ship the same title and description on every locale layout.
- **Do NOT** make indexable content depend only on a client fetch. Put crawlable content in RSC.
- **Do NOT** `index` authenticated admin routes.
- **Do NOT** move secrets into `NEXT_PUBLIC_` for Open Graph images or analytics (`piximind-security-js-ts`).
- **Do NOT** skip `alt` on content images or use raw `<img>` when `next/image` is the repo pattern.
- **Do NOT** treat AdminJS / internal SPAs as public sites (usually `noindex`; see React architecture when present).
- **Do NOT** use `any` in `generateMetadata`, sitemap entries, or JSON-LD builders. Type the metadata return.
- **Do NOT** pull extra client JS for the H1 / primary copy. Indexable content stays in RSC.

## Approved patterns
- **Do** set per-route `generateMetadata`: title template, description, canonical, `metadataBase`.
- **Do** ship `sitemap` + `robots` that allow only public routes.
- **Do** wire locale + hreflang via next-intl (`[locale]`). Unique title/description **per locale**.
- **Do** keep indexable copy in Server Components. `"use client"` stays on leaf widgets (`piximind-atomic-web`).
- **Do** use `next/image` + meaningful `alt`. Optional JSON-LD on public pages only.
- **Do** watch Core Web Vitals (LCP/CLS) when changing hero images or fonts — do not block rendering on client-only data for the H1.

## Seams (test / depend here)
- `generateMetadata` return shape (title, description, alternates/canonical, robots).
- `sitemap.ts` / `robots.ts` public URL lists.
- next-intl messages used in metadata (not only in client trees).
- JSON-LD component props on public pages.
- Assert these in `piximind-tdd-nextjs`; do not snapshot the whole HTML document as the only SEO test.

## Workflow
1. Classify the route: public marketing vs auth-gated app.
2. Implement metadata + indexability, then run the audit checklist.
3. If HTML, tokens, or PII could leak into analytics/metadata, also follow `piximind-security-js-ts`.

## Checklist
### Metadata
- [ ] Unique title + description per route **and** locale (title template, not a clone).
- [ ] Canonical + `metadataBase` set.
- [ ] OG/Twitter tags do not contain secrets or PII.

### Indexability
- [ ] Public routes reachable in `sitemap`; `robots` does not block them.
- [ ] Private/auth routes `noindex` (and not in sitemap).
- [ ] Primary content is in RSC HTML, not only after client fetch.

### i18n
- [ ] `[locale]` + hreflang/alternates for each public locale.
- [ ] No identical default title/description copied across locales.

### Media & structured data
- [ ] `next/image` + alt on content images.
- [ ] JSON-LD only on public pages that need it, and it matches visible content.

### Loop
- [ ] Self-check against this list after generating the page (crawl, metadata, indexability, i18n, structured data).
- [ ] No `any` in metadata/JSON-LD. No secrets in OG tags or logs.
- [ ] Sonar-style: unused metadata vars, empty `catch` around sitemap fetch, duplicated locale clones.

## Out of scope
- Flutter ASO (`piximind-seo-flutter`).
- Nest JSON APIs (not indexable pages).
- Inventing a second metadata library when `generateMetadata` already exists.

## Token efficiency
- Inspect existing `generateMetadata` / `sitemap.ts`; do not paste the App Router tree into chat.
- One concern: SEO audit loop. Layout → architecture skill. Secrets → `piximind-security-js-ts`.
