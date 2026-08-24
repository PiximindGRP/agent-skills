<p align="center">
  <img src="docs/assets/piximind-logo.svg" alt="Piximind" width="280" />
</p>

# Piximind agent-skills

Internal Cursor skills for Flutter, NestJS (TypeORM or Prisma), Next.js, React, and TypeScript.

Skills auto-apply when the agent judges them relevant from `description`, and only when open files match `paths`. There is no `alwaysApply` / `always_true` field on `SKILL.md` ([Cursor Skills](https://cursor.com/docs/skills.md)). Always-on text lives in a **rule**: [`.cursor/rules/piximind-always.mdc`](.cursor/rules/piximind-always.mdc) (`alwaysApply: true`).

| Apply | Mechanism | Where |
|-------|-----------|--------|
| Every chat | `alwaysApply: true` | `.cursor/rules/*.mdc` |
| Matching files | `paths:` | `skills/*/SKILL.md` |
| Agent decides | `description` | `skills/*/SKILL.md` |
| Slash only | `disable-model-invocation: true` | not used here |

Authoring: [docs/SKILL-RULES.md](docs/SKILL-RULES.md) · [docs/SKILL-TEMPLATE.md](docs/SKILL-TEMPLATE.md) · full rules dump: [docs/SKILLS-SUMMARY.md](docs/SKILLS-SUMMARY.md).

One skill per PR on `skill/{skill-name}`. Do not merge until Tech Lead review.

---

## Catalog

| Domain | Flutter | NestJS | Next.js | React SPA / AdminJS |
|--------|---------|--------|---------|---------------------|
| Architecture | [flutter](skills/piximind-architecture-flutter/SKILL.md) | TypeORM [nestjs](skills/piximind-architecture-nestjs/SKILL.md) · Prisma [prisma](skills/piximind-architecture-prisma/SKILL.md) | [nextjs](skills/piximind-architecture-nextjs/SKILL.md) | [react](skills/piximind-architecture-react/SKILL.md) |
| TDD | [tdd-flutter](skills/piximind-tdd-flutter/SKILL.md) | [tdd-nestjs](skills/piximind-tdd-nestjs/SKILL.md) | [tdd-nextjs](skills/piximind-tdd-nextjs/SKILL.md) | [tdd-react](skills/piximind-tdd-react/SKILL.md) |
| Offline | [offline-flutter](skills/piximind-offline-flutter/SKILL.md) | — | [offline-nextjs](skills/piximind-offline-nextjs/SKILL.md) | only if a PWA already exists |
| Atomic | [atomic-flutter](skills/piximind-atomic-flutter/SKILL.md) | — | [atomic-web](skills/piximind-atomic-web/SKILL.md) | [atomic-web](skills/piximind-atomic-web/SKILL.md) |
| SEO | [seo-flutter](skills/piximind-seo-flutter/SKILL.md) | — | [seo-web](skills/piximind-seo-web/SKILL.md) | usually `noindex` |
| Security | [security-flutter](skills/piximind-security-flutter/SKILL.md) | [security-js-ts](skills/piximind-security-js-ts/SKILL.md) | same | same |
| Visual | [frontend-design-pixi](skills/frontend-design-pixi/SKILL.md) | — | same | same |

---

## Architecture

### `piximind-architecture-flutter`

**Apply:** `paths` `**/*.dart`, `**/*.yaml`, `**/*.arb`  
**When:** Flutter features, layers, DI, routing, BLoCs.

Feature-first `lib/features/{feature}/{domain,data,presentation}`. Domain = entities + repository contracts + `UseCase.call` (no Flutter / Floor / `http`). Data = datasources + `*_repository_impl.dart`. Presentation composes `template_` / `organism_`. DI `get_it`, routing `go_router`, state in `lib/core/blocs/`. Widget props typed (`*Params` or named constructors). No Riverpod / AutoRoute.

### `piximind-architecture-nestjs`

**Apply:** `paths` `**/*.ts`, `**/src/components/**`, `**/src/entities/**`, `**/src/dto/**`  
**When:** NestJS TypeORM modules, controllers, services, repositories, DTOs. Prisma apps use the Prisma skill instead.

`src/components/{domain}/` = module + HTTP-only controller + service + repository. Entities in `src/entities/` (`Based`, `EDataTable`). DTOs class-validator. Envelope `{ statusCode, data | error }`. No `any`, no N+1, no Prisma in this tree.

### `piximind-architecture-prisma`

**Apply:** `paths` `**/*.ts`, `**/*.prisma`, `**/prisma/**`, `**/src/components/**`  
**When:** NestJS + `schema.prisma` / `PrismaService`. TypeORM apps stay on the Nest skill.

Same `components/{domain}` layers; persistence is `PrismaService` in the repository. Types: `Prisma.*Input`, `IDB{Name}`, `I*` payloads. Extract includes, `select`, shared `where` for list+count, `$transaction`. No `$queryRawUnsafe`.

### `piximind-architecture-nextjs`

**Apply:** `paths` `**/*.{ts,tsx}`, `**/src/app/**`, `**/src/server-side/**`, `**/src/client-side/**`  
**When:** App Router pages, BFF handlers, API modules, TS contracts.

`server-side` / `client-side` / `common-side`. Nest only from `{Domain}Api`. Browser `{Domain}ClientApi`. Named `I{Domain}*Response` interfaces. DS props `IAtom*` / `IMolecule*` / `IOrganism*` in the repo’s interfaces folder. Never `any`. Thin RSC `page.tsx`.

### `piximind-architecture-react`

**Apply:** `paths` `**/*.{ts,tsx}`, `**/src/admin/**`, `**/src/Page/**`, `**/src/DesignSystem/**`  
**When:** Vite SPA / CSR / AdminJS. Not App Router.

Vite: Page → Template → DesignSystem + `{Domain}API`. AdminJS: `admin/components/{pages,actions,properties}` registered only in `register-components.ts`. Atoms never fetch. Typed route/response interfaces. No SW by default.

---

## TDD

Vertical slices only: one failing behavior → minimal code. Tests at **seams**, not private helpers.

### `piximind-tdd-flutter`

**Apply:** `paths` `**/*.dart`, `**/test/**`  
**When:** `*_test.dart`, test-first Flutter, `bloc_test`.

`test/features/{feature}/...`. Seams: `UseCase.call`, repository contract, BLoC event → state. Domain tests pure Dart. Widget-test organism/template public API / `*Params` only.

### `piximind-tdd-nestjs`

**Apply:** `paths` `**/*.spec.ts`, `**/test/**`, `**/*.ts`  
**When:** Nest specs and e2e.

Colocate `*.spec.ts`. Mock repositories / `PROVIDERS`. No real DB or Keycloak. Assert guards. Typed fixtures, no live JWTs.

### `piximind-tdd-nextjs`

**Apply:** `paths` `**/*.{ts,tsx}`, `**/*.{spec,test}.{ts,tsx}`  
**When:** App Router tests (Vitest/Jest already in the repo).

Seams: mappers, server API modules (mocked fetch), RTL organism props, `generateMetadata`. Client tests never call Nest. No `any` in fixtures.

### `piximind-tdd-react`

**Apply:** `paths` `**/*.{ts,tsx}`, `**/*.{spec,test}.{ts,tsx}`, `**/src/admin/**`  
**When:** SPA / AdminJS tests.

RTL at public props, handlers, visibility. Mock ESM-hostile libs the way the repo already mocks `adminjs`. No CSS-only assertions.

---

## Offline

### `piximind-offline-flutter`

**Apply:** `paths` `**/*.dart`  
**When:** Network, Floor cache, connectivity UX.

`NetworkBloc` + Floor + `IApiRepository.supportOffline`. No second DB. Tokens in `flutter_secure_storage`. Connectivity banner on `template_scaffold`.

### `piximind-offline-nextjs`

**Apply:** `paths` `**/*.{ts,tsx,js}`, `**/public/**`  
**When:** Existing PWA / service worker work.

Reuse `ServiceWorkerRegister` / `InstallPrompt` / manifest. Cache static shell + agreed GET only. Never cache tokens or mutations.

---

## Atomic design

### `piximind-atomic-flutter`

**Apply:** `paths` `**/*.dart`  
**When:** Widgets, pages, skeletons, sheets, tokens — not palette.

`lib/core/common/presentation/{atoms,molecules,organisms,templates,...}` with `atom_` prefixes. Atoms import no features. Props: `*Params` or typed constructors, never `dynamic`/`Map`.

### `piximind-atomic-web`

**Apply:** `paths` `**/*.{ts,tsx}`  
**When:** Next.js or React UI (including AdminJS) — not palette.

One naming system per repo (`Atom*` under `ds/` **or** `components/base` + `application`). Props as `IAtom*` / `IMolecule*` / `IOrganism*` in the interfaces folder. Atoms wrap HTML only.

---

## SEO

### `piximind-seo-web`

**Apply:** `paths` `**/*.{ts,tsx}`, `**/src/app/**`  
**When:** Metadata, locales, indexability on App Router.

Per-route `generateMetadata`, unique per locale. RSC for crawlable copy. Private routes `noindex`. `next/image` + alt.

### `piximind-seo-flutter`

**Apply:** `paths` `**/*.dart`, `**/*.yaml`, `**/fastlane/**`  
**When:** Store listings, flavors, deeplinks, Flutter web SEO if web is shipped.

Unique ASO copy per flavor. `go_router` + App/Universal Links. No tokens in query strings. Native-first — no fake `robots.txt` on mobile-only.

---

## Security

Do not rewrite these. New skills compose with them.

### `piximind-security-js-ts`

**Apply:** `paths` `**/*.{ts,tsx,js,jsx}`  
**When:** Env, deps, untrusted payloads, JWT typing, secrets in logs/tests.

Discover existing env pattern. No secrets in `NEXT_PUBLIC_`. No `--force` / `--legacy-peer-deps` unless ordered. Validate with Zod / class-validator. Never `any`.

### `piximind-security-flutter`

**Apply:** `paths` `**/*.dart`, `**/*.yaml`, `**/*.arb`  
**When:** Auth, storage, network, `/security-review`.

`flutter_secure_storage` for tokens/PII. No `print` of secrets. No TLS bypass. Stale pub.dev packages rejected.

---

## Visual identity

### `frontend-design-pixi`

**Apply:** `paths` `**/*.{tsx,jsx,css,scss,dart}`  
**When:** Palette, type, layout, copy tone. Figma URL or `design.md` / tokens.

Figma MCP first (`search_design_system`, variables, design context). Else `design.md`. Atomic skills own folders; this skill owns look.

---

## Cross-cutting (always-on rule)

See [`.cursor/rules/piximind-always.mdc`](.cursor/rules/piximind-always.mdc): inspect first, no `any`, typed Flutter props, no secrets, apply the matching skill, compose security.

Sonar-style (in skills, not a full manual): bugs and vulnerabilities first, then cognitive complexity, duplication, unused locals, empty `catch`. Coverage at seams.

Not a skill: `skills/piximind-security-flutter copy/` (accidental duplicate). Leave unused.
