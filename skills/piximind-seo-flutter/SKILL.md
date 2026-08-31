---
name: piximind-seo-flutter
description: Enforces Flutter ASO and deep-link rules — unique store titles per flavor, go_router App Links / Universal Links, indexable public routes only. Use when editing store listings, flavors, deeplinks, Analytics/Sentry extras, or Flutter web SEO if the project actually ships web.
paths:
  - "**/fastlane/**"
  - "**/pubspec.yaml"
  - "**/*.arb"
  - "**/flavorizr.yaml"
  - "**/*deeplink*"
  - "**/ios/Runner/*.entitlements"
---

# Flutter SEO / ASO

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

Treat this as a **native app** unless the repo actually builds Flutter web.

## Anti-patterns
- **Do NOT** treat a native app as a website (no `robots.txt` / sitemap theater on mobile-only flavors).
- **Do NOT** mix store metadata across flavors. Each flavor has unique title and description.
- **Do NOT** put PII in Analytics, Firebase, or Sentry extras (`piximind-security-flutter`).
- **Do NOT** deep-link into authenticated screens with tokens in the URL query.
- **Do NOT** mark every `go_router` location as an Android App Link / iOS Universal Link. Public, indexable routes only.
- **Do NOT** add Flutter **web** meta-headers unless the project ships web.

## Approved patterns
- **Do** maintain unique store listing copy (ASO title, short/long description, keywords) **per flavor**.
- **Do** implement deep links with the repo’s `go_router` routes plus platform App Links / Universal Links files already in the project.
- **Do** keep public routes shareable; keep private routes behind auth without leaking tokens in the link.
- **Do** if (and only if) Flutter web is shipped: Semantics + document title/description on public web routes — still no PII.
- **Do** align flavor application IDs, store listings, and deep-link hostnames so they do not cross brands.

## Seams (test / depend here)
- Flavor-specific store metadata files / Fastlane / store config already in the repo.
- `go_router` route table (public vs auth).
- Android `assetlinks.json` / iOS Associated Domains (or the repo’s existing files).
- Analytics/Sentry attach helpers (must not accept PII maps).

## Workflow
1. Confirm flavors and whether web is actually built.
2. Change listing copy or link hosts only for the flavor you were asked to touch.
3. Run the checklist.
4. If deeplink tokens, PII, or logging are involved, also follow `piximind-security-flutter`. Routing structure stays with the Flutter architecture skill when present.

## Checklist
- [ ] Native vs web decided from the repo (no fake website SEO on mobile-only).
- [ ] Store title/description unique per flavor; do not reuse another flavor’s copy.
- [ ] Deep links use `go_router` + existing App/Universal Link config.
- [ ] Only public routes are indexable / advertised as links.
- [ ] No tokens in link query; no PII in Analytics/Sentry extras.
- [ ] Web meta/Semantics added only if Flutter web is shipped.

## Out of scope
- Next.js `generateMetadata` (`piximind-seo-web`).
- Visual identity (`piximind-frontend-design`) and atomic folder layout (`piximind-atomic-flutter`).
- Inventing a new flavor or store account.

## Token efficiency
- Inspect flavor/store files and `go_router`; do not paste the route table into chat.
- One concern: ASO + deeplinks. Tokens in URLs → `piximind-security-flutter`.
