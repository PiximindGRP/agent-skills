# Next.js App Router tree

App Router + TypeScript + next-intl + next-auth. New Nest-backed lists use `{Domain}Api` + mapper + BFF route. New browser calls use `{Domain}ClientApi` via `FRONTEND_API_ROUTER`.

## Stack
- Next.js App Router, React, TypeScript, `@/*` → `src/*`
- Auth: next-auth Credentials; session via `getAuthSession()` (`src/app/api/auth/auth.session.ts`)
- i18n: next-intl, `src/i18n/`, messages in `public/locales/{locale}.json`, routes under `src/app/[locale]/`
- HTTP: `ApiSSRService` (`common-side/api/api.ssr.service.ts`) — `type: 'BACKEND'` (Nest) or `'SSR'` (this app’s `/api`)
- Config: `ConfigService` singleton (`common-side/config/config.service.ts`). Do not add a second env module. Secrets vs `NEXT_PUBLIC_`: `piximind-security-js-ts`
- Edge auth + locale: `src/proxy.ts` (Next proxy; matcher skips `api|_next`). Do not add a parallel `middleware.ts`

## Source tree
```text
src/
  app/
    layout.tsx, manifest.ts
    api/{resource}/route.ts          BFF: getAuthSession → {Domain}Api → JSON
    api/auth/[...nextauth]/          next-auth
    [locale]/layout.tsx              generateMetadata + NextIntlClientProvider
    [locale]/(public)/login/
    [locale]/(private)/              pages + PrivateAppLayout (header/sidebar)
  server-side/
    api/{domain}/{domain}.api.ts     Nest client ({Domain}Api). ApiSSRService BACKEND
    api/{domain}/{domain}.api.interface.ts  I{Domain}*Response — mandatory
    api/{domain}/{domain}.mapper.ts  optional; use when Nest DTO ≠ UI
    router/backend.router.ts         BACKEND_API_ROUTER (Nest path map; no raw path strings)
  client-side/
    api/{domain}/*.api.ts            {Domain}ClientApi → this app /api
    router/fo.router.ts              FRONTEND_API_ROUTER
    ds/{atoms,molecules,organisms}   Atomic* — props are IAtom*/IMolecule*/IOrganism* in the repo’s interfaces folder (piximind-atomic-web)
    pages/                           feature screens; "use client" when needed
    modals/, templates/, pwa/, context/, customHook/
  common-side/
    api/api.ssr.service.ts
    config/config.service.ts
    interface/api/api.interface.ts   ApiResponse<T>, ICallApi — never data: any
    enum/, constants/, utils/, service/validation/
  i18n/                              routing.ts, request.ts, navigation.ts
  components/                        leftover UI (charts, date-picker, file-upload)
```

## Dependency rule

```text
app/[locale] page.tsx (RSC)
  → getAuthSession + {Domain}Api (+ mapper) + client-side/pages
app/api/*/route.ts
  → getAuthSession + {Domain}Api   (not DS, not Nest URLs inlined)
server-side/api
  → ApiSSRService(BACKEND) + BACKEND_API_ROUTER + *.api.interface
  → mapper may target organism props
client-side/pages + modals
  → ds + {Domain}ClientApi + types from *.api.interface (I{Domain}*Response)
  → not BACKEND_API_ROUTER, not ApiSSRService(BACKEND)
client-side/api
  → ApiSSRService(SSR) + FRONTEND_API_ROUTER
client-side/ds/atoms
  → native HTML + ds interfaces only (no server-side, no Nest)
common-side
  → fetch helper, config, shared enums/types — no React DS
```

Bearer to Nest only from server-side APIs and Route Handlers. Client never talks to Nest.

## Adding a domain (checklist order)

1. Nest path on `BACKEND_API_ROUTER`.
2. `src/server-side/api/{domain}/{domain}.api.ts` singleton `getInstance()`, `ApiSSRService({ type: 'BACKEND' })`, methods take `accessToken`, explicit `Promise<I{Domain}…Response>`.
3. `{domain}.api.interface.ts` for Nest DTOs / response shapes **before** the first fetch. No `any`. See [interfaces.md](interfaces.md).
4. `{domain}.mapper.ts` when Nest DTO ≠ UI. Skip if the page already consumes the DTO.
5. `src/app/api/{resource}/route.ts`: `getAuthSession`, 401 if no token, call `{Domain}Api`, annotate `const apiRes: I{Domain}ListResponse`, return `NextResponse.json`. Handler stays thin.
6. Path on `FRONTEND_API_ROUTER`.
7. `src/client-side/api/{domain}/{domain}.api.ts` as `{Domain}ClientApi` + `ApiSSRService({ type: 'SSR' })`. One class per domain. Return `Promise<I{Domain}…Response>`.
8. Screen under `client-side/pages/{Page}/` composing organisms. Thin `src/app/[locale]/(private|public)/.../page.tsx` that imports the page (RSC may prefetch via `{Domain}Api`).
9. Strings in `public/locales/{locale}.json`. Metadata: `piximind-seo-web`.

## Variants

| Pattern | Where | What to do |
|---------|--------|------------|
| Full Nest-backed slice | `{Domain}Api` + mapper + `/api/{resource}` + RSC prefetch | **Template for new Nest-backed lists.** |
| Client API via router | `{Domain}ClientApi` + `FRONTEND_API_ROUTER` | **Template for new `{Domain}ClientApi`.** |
| Duplicate client API | two fetch wrappers in one domain | Extend the domain you are in; do not add a third wrapper. Prefer `FRONTEND_API_ROUTER` + `ApiSSRService(SSR)` for new files. |
| No mapper | domains that pass DTO through | Keep mapping in `{Domain}Api` or pass DTO through; do not invent mappers for every field. |
| Leftover UI | `src/components/base` + `application` | Reuse only where existing screens already import it. New UI: `client-side/ds` (`piximind-atomic-web`). |
| Existing `components/design-system` + `common/interface/DS` | props in `common/interface/DS/{Atoms,Molecules,Organisms}`; paths in `src/router/` | **Follow that tree.** Do not migrate to `client-side/ds` inside a feature PR. |
| Empty unused shells | `server-side/api/ssr.api.ts`, unused `NEXT_END_POINT` stub | Do not fill or copy. If `*_END_POINT` is the **live** path map, extend it. |
| PWA | `client-side/pwa`, `app/manifest.ts` | `piximind-offline-nextjs`. |

## Other App Router layouts

If the repo uses `src/features/` (and possibly `src/app/api/v3/`) instead of `server-side`/`client-side`, follow **that** tree. Do not migrate it inside a feature PR, and do not bring `src/features/` into a `server-side`/`client-side` repo.

If the repo uses `src/common/` + `src/router/` + `src/components/design-system` + `src/common/interface/DS`, follow **that** tree. Paths stay on the existing `*_END_POINT` / FO router consts. Response interfaces stay under `common/interface/`. Do not introduce `server-side/` / `client-side/` beside it.
