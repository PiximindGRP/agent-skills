---
name: piximind-offline-nextjs
description: Enforces Next.js PWA offline using the repo’s existing ServiceWorkerRegister, InstallPrompt, and manifest — cache static shell plus agreed GET reads only. Use when changing service workers, PWA install UX, or offline caching in a Next.js app that already has a SW.
paths:
  - "**/*.{ts,tsx,js}"
  - "**/public/**"
---

# Next.js offline / PWA

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

Reuse `ServiceWorkerRegister`, `InstallPrompt`, and the existing `manifest`. Do not add a random Workbox stack if a service worker already exists.

## Anti-patterns
- **Do NOT** cache tokens, session cookies, or PII in the service worker.
- **Do NOT** put server secrets in `NEXT_PUBLIC_` so the SW can “work offline” (`piximind-security-js-ts`).
- **Do NOT** cache authenticated mutations or POST/PUT/PATCH/DELETE by default.
- **Do NOT** leave offline users on a blank page. Show an explicit degraded state.
- **Do NOT** introduce a second SW library (Workbox, custom webpack plugin) when the project already registers a worker.
- **Do NOT** precache HTML for `noindex` / auth-gated app shells in a way that leaks private routes to shared caches.
- **Do NOT** type SW handlers, cache keys, or allowlists as `any`.
- **Do NOT** cache unbounded GET lists. Agreed reads only — extra client JS / SW work is a smell.

## Approved patterns
- **Do** cache the **static shell** plus **agreed GET reads** only (the list the app already treats as cacheable).
- **Do** keep SW registration in the existing `ServiceWorkerRegister` component (or equivalent).
- **Do** keep install UX in `InstallPrompt` / existing manifest icons and `display`.
- **Do** fail closed on API errors when the response is not in the agreed GET cache.
- **Do** scope caches so a logged-in user’s private data is never stored in a shared Cache Storage key.

## Seams (test / depend here)
- `ServiceWorkerRegister` (registration side effect).
- `InstallPrompt` public UI.
- Manifest `start_url` / `scope` / icons.
- Agreed GET cache allowlist (document it in the PR if you extend it).
- Offline fallback UI (degraded page, not a spinner that never ends).

## Workflow
1. Locate the existing SW, manifest, and register/install components.
2. Extend the GET allowlist only when product already treats that read as cacheable.
3. Run the checklist.
4. If secrets, tokens, or PII are involved, also follow `piximind-security-js-ts`. Architecture: `piximind-architecture-nextjs` when that skill is present.

## Checklist
- [ ] Existing SW reused; no second Workbox stack.
- [ ] Only static shell + agreed GET reads are cached.
- [ ] No tokens or PII in Cache Storage.
- [ ] `NEXT_PUBLIC_` still contains no server secrets.
- [ ] Offline UX is a degraded state, not a blank screen.
- [ ] Auth-gated routes are not precached as public HTML.
- [ ] No `any` in SW/register/allowlist code. No secrets in SW logs.
- [ ] Sonar-style: unused cache names, empty `catch` in fetch handlers, duplicated cache-key logic.

## Out of scope
- AdminJS / online-only admin UI (do not add a SW by default; see `piximind-architecture-react` when present).
- Inventing background sync / outbox for Nest mutations unless the repo already has it.
- Flutter offline (`piximind-offline-flutter`).

## Token efficiency
- Inspect the existing SW/manifest; do not paste the App Router tree into chat.
- One concern: PWA cache + offline UX. Secrets → `piximind-security-js-ts`. Routes → `piximind-architecture-nextjs`.
