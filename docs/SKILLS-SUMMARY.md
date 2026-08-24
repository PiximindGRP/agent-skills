# Skills summary

Catalog of every skill under `skills/`. One concern per skill. Inspect the current repository first and follow its existing folders. Do not introduce a second architecture.

Compose with security skills whenever the change touches secrets, PII, tokens, HTML, or TLS. Visual identity (`frontend-design-pixi`) owns palette and look; atomic skills own structure and naming.

| Domain | Flutter | NestJS | Next.js | React SPA / AdminJS |
|--------|---------|--------|---------|---------------------|
| Architecture | `piximind-architecture-flutter` | TypeORM: `piximind-architecture-nestjs` · Prisma: `piximind-architecture-prisma` | `piximind-architecture-nextjs` | `piximind-architecture-react` |
| TDD | `piximind-tdd-flutter` | `piximind-tdd-nestjs` | `piximind-tdd-nextjs` | `piximind-tdd-react` |
| Offline | `piximind-offline-flutter` | — (client concern) | `piximind-offline-nextjs` | only if a PWA already exists |
| Atomic | `piximind-atomic-flutter` | — | `piximind-atomic-web` | `piximind-atomic-web` |
| SEO | `piximind-seo-flutter` | — | `piximind-seo-web` | usually `noindex` |
| Security | `piximind-security-flutter` | `piximind-security-js-ts` | `piximind-security-js-ts` | `piximind-security-js-ts` |
| Visual identity | `frontend-design-pixi` | — | `frontend-design-pixi` | `frontend-design-pixi` |

---

## Cross-cutting rules

- **Inspect first.** If the repo already has a layout, follow it. Do not paste Feature-Sliced Design, hexagonal ports, or a second design system. Do not dump folder trees into chat — skills point at `references/` for trees.
- **One skill, one concern.** Architecture ≠ tests ≠ SEO ≠ palette.
- **Seams, not internals.** Test and depend on public contracts (UseCase, repository interface, HTTP envelope, organism props / `*Params` / `IOrganism*`).
- **Vertical TDD.** One failing behavior → minimal code → next behavior. No horizontal “all tests then all code”.
- **No secrets in fixtures, Floor, Cache Storage, logs, or `NEXT_PUBLIC_`.**
- **No `--force` / `--legacy-peer-deps`** unless the user ordered it.
- **Strict typing.** TypeScript: never `any` / `as any` (named interface or `unknown` + narrowing). Flutter widgets: never `dynamic` or ad-hoc `Map` props — typed named constructors or `*Params`.
- **Sonar-style (compact, not a manual):** bugs and vulnerabilities first; then smells (cognitive complexity, duplicated blocks, unused locals, empty `catch`); coverage at seams only.
- **Design source.** If Figma MCP is connected (or a `figma.com` URL is given), pull tokens/components from it. Else read `design.md` / `DESIGN.md` / existing token files before inventing a palette (`frontend-design-pixi`).

---

## Architecture

### `piximind-architecture-flutter`

**When:** Adding or reviewing Flutter features, layers, DI, routing, or BLoCs.

**Tree:** `lib/features/{feature}/{domain,data,presentation}`. Shared UI and infra in `lib/core`. DI = `get_it` (`injector.dart`). Routing = `go_router`. State = `flutter_bloc` / `hydrated_bloc` in `lib/core/blocs/`.

**Most important rules**
- Domain is Equatable entities + abstract `Repository` + `UseCase.call`. No Flutter, Floor, `http`, `get_it`, or widgets in `domain/`.
- Data = datasources + `*_repository_impl.dart`. HTTP through `ApiRepository` / `IApiRepository`, not raw `http` in pages.
- Presentation composes `template_` / `organism_` widgets. UI dispatches BLoC **events**; never instantiate BLoCs in widgets.
- Widget props are typed (`*Params` under `models/params/` when that folder exists, else named constructors). No `dynamic` / `Map` props.
- Do not add Riverpod, AutoRoute, or repo-root `lib/{data,domain,presentation}`.
- Do not migrate leaky older features inside an unrelated PR.

**Seams:** `UseCase.call`, domain repository contract, datasource public methods, BLoC event → state, organism/template public params.

---

### `piximind-architecture-nestjs`

**When:** Adding or reviewing NestJS modules, controllers, services, repositories, or DTOs.

**Tree:** `src/components/{domain}/` = module + controller + service + repository. Entities in `src/entities/`, DTOs in `src/dto/{domain}/`, contracts in `src/interfaces/{domain}/`. Legacy src-root modules: follow that tree; do not migrate in a feature PR.

**Most important rules**
- Controller is HTTP-only: parse DTO, call service, return `{ statusCode, data }` or `{ statusCode, error }`.
- Business rules in the service (`I{Domain}Service`). Persistence only in the repository (`I{Domain}Repository`).
- Validate with class-validator. TypeORM via `PROVIDERS` + `DataSource.getRepository` — no Prisma/Mongoose/QueryBuilder in controllers.
- Entities extend `Based`; table names from `EDataTable`; add a migration when the schema changes.
- Never mix staff JWT (`JwtAuthGuard`) with guest JWT (`JwtGuestAuthGuard`). Do not disable guards on scoped mutations.
- Config via `ConfigService` / `IEnv`. No raw `process.env` or `synchronize: true`.
- Never `any`. No N+1 in services (load relations in the repository). Empty `catch` forbidden.

**Seams:** HTTP contract, `I{Domain}Service`, `I{Domain}Repository`, DTOs, `PROVIDERS.*`.

Inspect `src/` first. If the Nest app uses Prisma (`schema.prisma`, `PrismaService`) instead of TypeORM, follow `piximind-architecture-prisma`.

---

### `piximind-architecture-prisma`

**When:** Adding or reviewing NestJS Prisma modules, `schema.prisma`, repositories, services, or DTOs.

**Tree:** `src/components/{domain}/` = module + controller + service + repository. Global `PrismaModule` (`PrismaService extends PrismaClient`). DTOs in `src/dto/request/{domain}/`. Payload contracts in `src/interface/`. Relation graphs in `src/model` (`IDB{Name}`). Schema + migrations under `prisma/`.

**Most important rules**
- Controller is HTTP-only. New Prisma calls live in the repository. Business rules in the service.
- Type with `Prisma.{model}WhereInput` / `Include` / `CreateInput`, `IDB{Name}`, and `I*` payloads. Never `any`. Services are classes — not TypeORM-style `implements I{Domain}Service`.
- Extract include graphs; `select` for narrow reads; list + `count` share `where`; `skip`/`take` pagination. `$transaction` for multi-write. `@@index` on filtered FKs.
- No Prisma query logs, no `$queryRawUnsafe` / SQL concatenation. Env via `ConfigService` / `IEnv`. PII → `piximind-security-js-ts`.
- If the repo is TypeORM (`src/entities`, `DataSource`), use `piximind-architecture-nestjs` instead. Do not mix.

**Seams:** HTTP envelope `{ ...data, statusCode, message }`, service public methods, repository public methods, DTOs, `PrismaService` mock.

---

### `piximind-architecture-nextjs`

**When:** Adding or reviewing Next.js App Router pages, BFF route handlers, API modules, or folder layout.

**Tree:** `src/app/` (routes, layouts, metadata, Route Handlers) + `server-side` / `client-side` / `common-side`. If the repo uses `src/features/` instead, follow that tree; do not migrate it.

**Most important rules**
- Nest is called only from `server-side/api/{domain}` (`{Domain}Api` + mapper + `BACKEND_API_ROUTER`). Never from client atoms or pages.
- Browser calls `{Domain}ClientApi` + `FRONTEND_API_ROUTER` + `ApiSSRService({ type: 'SSR' })`.
- BFF handlers stay thin: `getAuthSession()` → `{Domain}Api` → JSON.
- Screens live in `client-side/pages`; `app/.../page.tsx` is a thin wrapper. `"use client"` only on DS/pages that need events or hooks.
- No FSD, tRPC, RTK Query, second BFF, or leftover `src/components/` for new UI.

**Seams:** `{Domain}Api`, pure mappers, Route Handler contract, `{Domain}ClientApi`, router key maps.

---

### `piximind-architecture-react`

**When:** React SPA, Vite, CSR, or AdminJS UI. **Not** Next.js App Router / RSC.

**Trees:** Vite = `Page` → `Template` → `DesignSystem` + `{Domain}API`. AdminJS = `src/admin/components/{pages,actions,properties}` registered only in `register-components.ts`.

**Most important rules**
- HTTP only in `{Domain}API` (Vite) or `ApiClient` (AdminJS). Atoms never fetch, never run Prisma, never gate roles.
- AdminJS resources reference components by **string name**. Do not dump `.tsx` into Nest `src/resources/`.
- Authz for AdminJS stays on the server (`isAccessible` / `isVisible`).
- One atomic naming system per repo. No FSD, no second HTTP client, no service worker by default.

**Seams:** `{Domain}API` / action handlers, Page → Template props, `ACCESSES` / `isAccessible`, organism public props.

---

## TDD

Shared rule: one behavior per red-green slice; name tests after observable outcomes; expected values are independent fixtures.

### `piximind-tdd-flutter`

**When:** Flutter tests, test-first features, `*_test.dart` / `bloc_test` review.

**Most important rules**
- Place tests under `test/features/{feature}/...` (or the repo’s existing tree).
- Seams: `UseCase.call`, repository **interface**, BLoC event → state (`bloc_test`).
- Domain tests are pure Dart — no Flutter / Floor / `http`.
- Presentation tests mock the repository contract (or datasources), not the UseCase.
- Widget-test organism / template **public API** only. No private widgets, no tautological tests.

---

### `piximind-tdd-nestjs`

**When:** NestJS features, service/controller specs, e2e review.

**Most important rules**
- Colocate `*.spec.ts` next to the unit. E2E stays in `test/`.
- Mock repositories, `PROVIDERS.*`, and outbound services. No real DB, Keycloak, Redis, or third-party APIs in unit tests.
- Assert guards / 401 / 403 on protected routes. Cover staff vs guest separately.
- E2E asserts the HTTP envelope, not TypeORM internals. No live JWTs in fixtures.

---

### `piximind-tdd-nextjs`

**When:** Next.js App Router features and tests (Vitest/Jest already in the repo).

**Most important rules**
- Seams: pure mappers, server-side API modules (mocked fetch), RTL at organism public props, `generateMetadata` return shape.
- Do not assert `useState` or private hooks. No snapshot-only UI as sole coverage.
- Client tests never call Nest. Do not add a second test runner or MSW unless the repo already uses it.

---

### `piximind-tdd-react`

**When:** React SPAs, AdminJS actions/pages, design-system components outside App Router.

**Most important rules**
- RTL at public props, handlers, and visibility — not CSS or theme tokens.
- Colocate `*.spec.ts` / `*.test.tsx` as the repo already does. Mock ESM-hostile libs (e.g. `adminjs`) the same way.
- Do not dump React tests into Nest `resources/`. Do not unit-test `register-components.ts` unless registration is the bug.

---

## Offline

### `piximind-offline-flutter`

**When:** Flutter network, cache, Floor datasources, or connectivity UX.

**Most important rules**
- Reuse `NetworkBloc`, Floor, `IApiRepository.supportOffline`, `template_scaffold`. No second DB (Hive/Isar).
- Reads degrade to Floor when offline. Mutations: copy the feature’s queue vs fail-fast — do not invent a global outbox.
- Tokens stay in `flutter_secure_storage`, never Floor. Never disable TLS.
- Empty cache + offline = explicit degraded UI, plus connectivity banner on the template.

---

### `piximind-offline-nextjs`

**When:** Service workers, PWA install UX, or offline caching in a Next.js app that **already** has a SW.

**Most important rules**
- Reuse `ServiceWorkerRegister`, `InstallPrompt`, and the existing manifest. No second Workbox stack.
- Cache **static shell + agreed GET reads** only. Never cache tokens, cookies, PII, or mutations.
- Auth-gated HTML must not leak into shared caches. Offline UX is a degraded state, not a blank page.
- Do not add a SW to online-only AdminJS by default.

---

## Atomic design

### `piximind-atomic-flutter`

**When:** Flutter widgets, pages, skeletons, sheets, tokens — not palette.

**Most important rules**
- Shared UI: `lib/core/common/presentation/{atoms,molecules,organisms,templates,skeletons,sheets,tokens}`.
- Prefixes: `atom_`, `molecule_`, `organism_`, `template_`. Do not mix `AtomButton` style in Flutter.
- Atoms wrap primitives only and import no `lib/features/...`. Pages compose templates/organisms — no layout soup.
- Colors and typography come from `tokens/` (after `design.md` / Figma if present). No second design-system folder.
- Props are DS-typed: named + `required` where needed. Organisms/templates use `{Widget}Params` in `models/params/` (or colocated `*_params.dart`) when that tree exists; else typed constructors. Never `dynamic` / ad-hoc `Map` for widget props.

---

### `piximind-atomic-web`

**When:** UI in Next.js App Router or React SPAs (including AdminJS) — not palette.

**Most important rules**
- One naming system **per repo**: `Atom*` / `Molecule*` / `Organism*` under `client-side/ds`, **or** `components/base` ≈ atoms and `components/application` ≈ organisms.
- Atoms wrap native HTML only. No `server-side`, Nest clients, fetch, role checks, or DTO mapping.
- Organisms may take feature types. Pages compose organisms. AdminJS pieces register only in `register-components.ts`.
- Raw HTML → security skill. Palette → `frontend-design-pixi`.

---

## SEO

### `piximind-seo-web`

**When:** Next.js App Router pages, metadata, locales, crawlability.

**Most important rules**
- Per-route `generateMetadata`: unique title + description **per locale**, canonical, `metadataBase`.
- Indexable copy lives in RSC HTML, not only after a client fetch. `next/image` + meaningful `alt`.
- `sitemap` / `robots` allow public routes only. Auth-gated and admin routes are `noindex`.
- next-intl `[locale]` + hreflang. Do not clone the same title/description across locales.
- Audit loop: crawl, metadata, indexability, i18n, structured data.

---

### `piximind-seo-flutter`

**When:** Store listings, flavors, deeplinks, Analytics/Sentry extras, or Flutter web SEO **if the project ships web**.

**Most important rules**
- Native app by default — no `robots.txt` theater on mobile-only flavors.
- Unique ASO title/description **per flavor**. Do not reuse another flavor’s copy.
- Deep links: `go_router` + existing App Links / Universal Links. Public routes only. No tokens in query strings.
- No PII in Analytics / Sentry extras. Web meta/Semantics only if Flutter web is actually built.

---

## Security (shipped — do not rewrite)

### `piximind-security-js-ts`

**When:** Next.js, React, or NestJS work involving env, deps, or untrusted data.

**Most important rules**
- Discover the existing env pattern (`ConfigService`, Zod/`env.mjs`, t3-env). Never assume raw `process.env`.
- `NEXT_PUBLIC_` never holds server secrets.
- Assess CVEs, maintenance, and bundle size before adding a package. No `--force` / `--legacy-peer-deps` unless ordered.
- Strip credentials from generated code, fixtures, and logs. Validate with Zod or class-validator.
- Never `any` / `as any`. Empty `catch` that hides auth failures is a vulnerability.

---

### `piximind-security-flutter`

**When:** Flutter/Dart auth, storage, network, or `/security-review`.

**Most important rules**
- Tokens and PII go in `flutter_secure_storage` (or the repo’s encrypted layer). Never `SharedPreferences` / Floor.
- No hardcoded secrets in the bundle. Inspect `envied` / existing codegen before inventing a loader.
- No `print` / `debugPrint` of HTTP bodies, JWTs, or user data. Do not disable TLS (`HttpOverrides.global`).
- Reject stale or vulnerable pub.dev packages. After fixes: `dart format` + `dart analyze` clean.
- No secrets in tests. Never `dynamic` for auth/storage APIs unless the repo documents an exception.

---

## Visual identity (shipped — do not rewrite)

### `frontend-design-pixi`

**When:** New UI or a visual reshape. Not atomic folder layout.

**Most important rules**
- Distinctive, brief-specific palette, type, and layout — avoid the usual AI-default looks.
- Ground choices in the subject. One justified aesthetic risk; spend boldness in one signature element.
- **Figma MCP** when connected (or a `figma.com` URL): search the design system, pull variables/node context, adapt to existing atoms. Else read `design.md` / `DESIGN.md` / token files first.
- Atomic skills own prefixes and folders; this skill owns look and copy tone.

---

## Strict typing / Sonar / design.md / MCP

Applies across skills. Do not duplicate a full Sonar or Figma manual in every `SKILL.md`.

| Topic | Rule |
|--------|------|
| TypeScript `any` | Forbidden (`any`, `as any`, `Record<string, any>`, `Promise<any>`). Named interface or `unknown` + narrowing. Untrusted JSON → Zod / class-validator DTO. |
| Flutter widget props | Typed named constructors. `*Params` in `models/params/` when that folder exists; else colocated `*_params.dart` or fields on the widget. Never `dynamic` / untyped `Map` unless the repo documents an exception. |
| Sonar-style | Bugs + vulnerabilities first. Smells: cognitive complexity, duplicated blocks, unused locals, empty `catch`. Coverage at seams (not private helpers). |
| Design source | Figma MCP (`search_design_system`, `get_variable_defs`, `get_design_context` after design-to-code guidance) **or** `design.md` / existing tokens. Do not invent a parallel palette. |
| Token efficiency | Inspect first; link `references/`; one concern per skill; do not paste folder trees into chat. |

Prisma persistence architecture is a **sibling** (`piximind-architecture-prisma`) — TypeORM Nest apps stay on `piximind-architecture-nestjs`. Do not mix Prisma and TypeORM trees in one feature.

---

## Authoring notes

Skills follow `docs/SKILL-TEMPLATE.md` and `docs/SKILL-RULES.md`. Frontmatter `name` is `piximind-{domain}-{tech}`. Each skill sets `paths` so Cursor only surfaces it on matching files ([skills docs](https://cursor.com/docs/skills.md)). `alwaysApply: true` is **rules-only** (`.cursor/rules/piximind-always.mdc`), not a SKILL.md field. `SKILL.md` stays short; trees live in `references/` one level deep. Catalog index: [README.md](../README.md).

Not a skill: `skills/piximind-security-flutter copy/` (accidental duplicate). Leave it unused.
