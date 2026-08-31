---
name: piximind-architecture-nextjs
description: "Enforces Next.js App Router layers (server-side / client-side / common-side) and named route/response interfaces. Use when editing src/app or BFF API modules — not Vite or AdminJS."
paths:
  - "**/src/app/**"
  - "**/src/server-side/**"
  - "**/src/client-side/**"
  - "**/src/common-side/**"
  - "**/src/i18n/**"
---

# Next.js architecture

Inspect `src/` first. Follow its existing folders. Do not introduce a second architecture (no Feature-Sliced `entities/widgets/features`, no hexagonal ports, no second BFF).

Canonical new work: `server-side` / `client-side` / `common-side`. If the repo uses `src/features/` or `src/common/` + `src/router/` instead, follow **that** tree inside a feature PR — do not migrate it.

Tree: [references/tree.md](references/tree.md). Route/response/JWT contracts: [references/interfaces.md](references/interfaces.md).

## Anti-patterns
- **Do NOT** fetch Nest (or `BACKEND_API_ROUTER`) from client atoms, molecules, or pages. Nest lives in `server-side/api` (or the repo’s existing API layer).
- **Do NOT** put mapping in atoms or Route Handlers. Mappers belong in `server-side/api/{domain}/*.mapper.ts`.
- **Do NOT** add `"use client"` on `app/` layouts or RSC `page.tsx` wrappers that only compose a client page.
- **Do NOT** introduce FSD, tRPC, RTK Query, or a second HTTP stack. Reuse `ApiSSRService` + `{Domain}Api` / `{Domain}ClientApi`.
- **Do NOT** add a parallel `middleware.ts` when `src/proxy.ts` exists, or a second env module beside `ConfigService`.
- **Do NOT** put new screens in leftover `src/components/` when `client-side/ds` exists. If the repo already uses `components/design-system` + `components/pages`, follow **that** (`piximind-atomic-web`).
- **Do NOT** leak tokens into `NEXT_PUBLIC_`, the service worker, or types imported by client DS (`piximind-security-js-ts`).
- **Do NOT** use `any`, `as any`, `Record<string, any>`, implicit any, or `Promise<any>`. Envelope `data: any` is forbidden — use `ApiResponse<T>`.
- **Do NOT** inline API path strings. Duplicate path literals are a Sonar smell — keys live on the existing router const.
- **Do NOT** declare fetch/Nest response shapes only inside `route.ts` or a page. Named `interface` in the api-interface file.
- **Do NOT** call `jwtDecode` untyped or cast JWT payloads `as any`.
- **Do NOT** inline DS props as `type Props = {` in the `.tsx`.
- **Do NOT** empty-`catch` fetch/mapper errors. No unused locals. Do not fetch on the client when RSC/`{Domain}Api` prefetch already exists.

## Approved patterns
- **Do** keep `src/app/` = routes, layouts, `generateMetadata`, Route Handlers. Locale under `[locale]`; private vs public via `(private)` / `(public)`.
- **Do** put Nest calls in `src/server-side/api/{domain}/`: `{domain}.api.ts` singleton (`{Domain}Api`) + `{domain}.api.interface.ts` + mapper when DTO ≠ UI. URLs only via `BACKEND_API_ROUTER`.
- **Do** keep BFF handlers thin: `getAuthSession()`, 401 without token, `{Domain}Api.getInstance()`, typed `NextResponse.json`. Annotate Nest result: `const apiRes: I{Domain}ListResponse`. Explicit `Promise<NextResponse>` on exported handlers.
- **Do** put browser calls in `src/client-side/api/{domain}/` as `{Domain}ClientApi` with `ApiSSRService({ type: 'SSR' })` and `FRONTEND_API_ROUTER`. One class per domain. Methods return `Promise<I{Domain}…Response>`.
- **Do** put screens in `client-side/pages` (or `components/pages` if that is the tree); `app/.../page.tsx` imports them. RSC may prefetch via `{Domain}Api`. `"use client"` on DS/pages that need events or hooks — never on a layout that can stay RSC.
- **Do** share HTTP/config/enums in `src/common-side/` (or `src/common/` if that is the tree). i18n in `src/i18n/` + `public/locales`. Auth: next-auth; Bearer to Nest only on the server.
- **Do** keep TypeScript strict (`strict: true`). Explicit return types on every exported function. DS props: `piximind-atomic-web` + [references/interfaces.md](references/interfaces.md).
- **Do** keep cognitive complexity down: fetch in the API class, map in the mapper, HTTP in the Route Handler. No unused locals (`@typescript-eslint/no-unused-vars`).

## Seams (test / depend here)
- `{Domain}Api` public methods (Nest + `accessToken`) and their `I{Domain}*Response` interfaces.
- Pure mappers (`*.mapper.ts`).
- Route Handler HTTP contract (`/api/{resource}`) — typed body in, typed JSON out.
- `{Domain}ClientApi` public methods (this app’s `/api`).
- `BACKEND_API_ROUTER` / `FRONTEND_API_ROUTER` keys (or the repo’s `*_END_POINT`).
- Organism public props (`IOrganism*`); `generateMetadata` return shape (`piximind-tdd-nextjs`, `piximind-seo-web`).
- `IDecodedToken` / session interfaces — not raw JWT.

## Workflow
1. Locate `src/app`, `src/server-side`, `src/client-side`, `src/common-side`. If those folders are missing, match the repo you are in — do not paste this tree.
2. Follow an existing Nest-backed domain (API + mapper + BFF + `*.api.interface.ts`) and an existing `{Domain}ClientApi`. Match the domain you edit if it already diverges.
3. Run the checklist.
4. If auth, secrets, PII, or HTML are involved, also follow `piximind-security-js-ts`. Tests: `piximind-tdd-nextjs`. UI naming: `piximind-atomic-web`. Offline/PWA: `piximind-offline-nextjs`. Metadata: `piximind-seo-web`.

## Checklist
- [ ] Inspected `src/` and copied an existing domain, not a textbook App Router essay.
- [ ] Nest only from `server-side/api` + router const; BFF route uses `getAuthSession`; handler stays thin.
- [ ] Client uses `{Domain}ClientApi` + `FRONTEND_API_ROUTER`; atoms import no `server-side`.
- [ ] Mapper (if any) is in the server-side API module, not in an atom.
- [ ] New page is `client-side/pages` + thin RSC `app/[locale]/.../page.tsx`; `"use client"` only where needed (no extra client bundle).
- [ ] Route paths are router-const keys; Nest/BFF responses are named interfaces; exported functions have return types; no `any`.
- [ ] DS props are `IAtom*` / `IMolecule*` / `IOrganism*` in the repo’s interfaces folder (`piximind-atomic-web`).
- [ ] JWT decoded as `IDecodedToken`; no secrets on client-imported types (`piximind-security-js-ts`).
- [ ] Locale strings in `public/locales`; no second i18n library; no FSD / second API layer / new env module.
- [ ] Sonar-style: unused locals, empty `catch`, duplicated mapper/handler blocks — extract at `{Domain}Api` / mapper seams (`piximind-tdd-nextjs`).

## Out of scope
- Vite SPA / AdminJS (`piximind-architecture-react` when present).
- Rewriting a `src/features/` App Router tree to `server-side`/`client-side`, or filling empty `SSRApi` / `NEXT_END_POINT` shells.
- TDD, SEO audit loops, atomic prefixes, PWA cache lists (sibling skills).
- Palette (`piximind-frontend-design`).

## Token efficiency
- Inspect `src/`; do not paste the App Router tree into chat — use [references/tree.md](references/tree.md).
- One concern: Next layers. Tests → `piximind-tdd-nextjs`. Secrets → `piximind-security-js-ts`.
