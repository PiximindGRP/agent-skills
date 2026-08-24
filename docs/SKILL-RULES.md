# Piximind Agent Skills — Team Rules (Wave 1)

These rules tell developers **which skills to write**, **what each skill must encode**, and **what Tech Lead will reject at review**. Do not invent a generic internet architecture. Encode **the architecture we already use**.

Security skills already shipped. Do not rewrite them. New skills must **compose** with them.

| Status | Skill | Path |
|--------|--------|------|
| Shipped | JS/TS security guard | `skills/piximind-security-js-ts/` |
| Shipped | Flutter security guard | `skills/piximind-security-flutter/` |
| Shipped | Frontend visual identity | `skills/frontend-design-pixi/` |
| To write | Architecture, TDD, offline-first, atomic design, SEO | this document |

---

## 1. Authoring rules (non-negotiable)

Copy ideas from [skills.sh](https://skills.sh), not their folder trees. Public catalogs (TDD, architecture, SEO audit, Flutter/Nest/Next packs) are **inspiration**. Our code is the **source of truth**.

### Shape

1. One concern per skill. Split if the file mixes architecture + tests + SEO.
2. `SKILL.md` ≤ 200 lines (target 80–120). Details go in `references/*.md`, one level deep.
3. Frontmatter `name` is `piximind-{domain}-{tech}` (kebab-case). `description` states **what** and **when**, third person.
4. Open with: **inspect the current repo first**. If the project already has a layout, follow it. Do not propose Feature-Sliced Design, hexagonal ports we do not use, or a second design system.
5. Encode **anti-patterns** (never do) and **approved patterns** (always do), then a **checklist** the agent can tick.
6. Name **seams** (public boundaries). Agents test and depend on seams, not private helpers.
7. Use our vocabulary: UseCase, Repository, DataSource, BLoC, Atom/Molecule/Organism/Template, Module/Controller/Service/Repository, `server-side` / `client-side` / `common-side`.
8. Cross-link security skills when the domain touches secrets, PII, tokens, or HTML.
9. No secrets, no live credentials, no `--force` / `--legacy-peer-deps` unless the user ordered it.
10. Do not publish until Tech Lead review (section 3).

### Folder

```text
skills/piximind-{domain}-{tech}/
├── SKILL.md
└── references/          # optional
    ├── seams.md
    └── examples.md
```

Use [SKILL-TEMPLATE.md](SKILL-TEMPLATE.md).

---

## 2. Ideas taken from skills.sh (adapt, do not fork)

| Source | Idea we keep |
|--------|----------------|
| `tdd` (mattpocock) | Vertical slices (one failing test → minimal code → repeat). Tests at **seams**. Behavior names, not implementation. No horizontal “write all tests then all code”. |
| `improve-codebase-architecture` | Deepen **our** modules. Do not redesign from a textbook. Survey existing folders before suggesting structure. |
| Agent-skills-standard catalogs | Organize by **technology then domain**. P0 = architecture + security. Keep skills short; progressive disclosure. |
| SEO cluster (`seo-audit`, `programmatic-seo`, `ai-seo`, GEO) | SEO is a **checklist + audit loop**, not a paragraph of meta-tag advice. Cover metadata, indexability, i18n, structured data, and (for Flutter) ASO + deep links. |
| Audits / Packs | Skills are reviewed before install. Later we ship a Piximind **pack**, not 20 unrelated skills. |
| `frontend-design` | Visual identity is already `frontend-design-pixi`. Atomic design skills own **structure and naming**, not palette. |

---

## 3. Review gate (Tech Lead, before deploy)

A skill is rejected if any box fails.

- [ ] Encodes **our** tree (Flutter feature-first, Nest `components/{domain}`, Next `server-side`/`client-side`, React DS or AdminJS split) — not a generic Clean Architecture blog.
- [ ] `description` has triggers (framework + domain words).
- [ ] Anti-patterns + approved patterns + checklist.
- [ ] Seams listed for that technology.
- [ ] Security skill linked where data/auth/HTML is involved.
- [ ] No generic FSD / Clean Architecture layer dump that contradicts the trees in this document (Flutter feature-first, Nest `components/{domain}`, Next `server-side`/`client-side`, React DS or AdminJS).
- [ ] Examples use our names (`atom_cta`, `OrganismTable`, `{Domain}Api`, `IApiRepository`).
- [ ] `SKILL.md` under 200 lines; extras in `references/`.
- [ ] Does not duplicate a shipped security skill.

---

## 4. Required skills — by technology and domain

Write **these** skills. Do not add extra domains in Wave 1.

### 4.1 Flutter

Inspect the existing Flutter app. Follow its folders.

| Domain | Skill to prepare | Essentials | Requirements |
|--------|------------------|------------|--------------|
| **Clean architecture** | `piximind-architecture-flutter` | Feature-first: `lib/features/{feature}/{domain,data,presentation}`. Domain = entities + repository **interfaces** + use cases. Data = local/remote datasources + `*_repository_impl.dart` + models. Presentation = pages/widgets/BLoC only. Shared UI and infra live in `lib/core`. DI = `get_it` (`injector.dart`). Routing = `go_router`. State = `flutter_bloc` / `hydrated_bloc`. | **Must follow this tree.** Domain must not import Flutter, Floor, `http`, or widgets. New features copy a complete `features/{feature}/` slice (`domain` + `data` + `presentation`). Do not introduce Riverpod, AutoRoute, or layer-based `lib/{data,domain,presentation}` at repo root. |
| **TDD** | `piximind-tdd-flutter` | Red → green vertical slices. Seams: UseCase.call, Repository contract, BLoC event → state. Widget tests only at organism/template **public API**. | If the repo has no `*_test.dart` yet, the skill must say where files go (`test/features/{feature}/...`) and forbid testing private widgets. Mock datasources, not UseCases, when testing presentation. Independent expected values (no tautological tests). |
| **Offline-first** | `piximind-offline-flutter` | `NetworkBloc` + `connectivity_plus`. HTTP cache via Floor (`IApiRepository`, `supportOffline`). Local + remote datasources in the repository. `HydratedBloc` for session-ish state. `cached_network_image`. Connectivity banner on `template_scaffold`. | Reads must degrade to Floor cache when offline. Mutations: define queue vs fail-fast per feature; do not invent a second DB. No tokens in Floor — `flutter_secure_storage` (security skill). Never disable TLS to “make offline work”. |
| **Atomic design** | `piximind-atomic-flutter` | `lib/core/common/presentation/{atoms,molecules,organisms,templates,skeletons,sheets,tokens}`. Prefixes: `atom_`, `molecule_`, `organism_`, `template_`. | Atoms have no feature imports. Pages in `features/*/presentation` compose templates/organisms. No one-off `Container` soup in pages. Tokens (colors, typography) stay in `tokens/`. |
| **SEO / ASO** | `piximind-seo-flutter` | Store listing (ASO), unique titles/descriptions per flavor, deep links via `go_router` (App Links / Universal Links), indexable public routes only. Flutter **web** SEO only if the project actually ships web. | Do not treat a native app as a website. No PII in Analytics/Sentry extras. Do not mix store metadata across flavors. Link security skill for deeplink tokens. |
| **Security** | shipped | Secure storage, no `print` of secrets, pub.dev hygiene, TLS. | Extend only if a new threat appears. |

### 4.2 NestJS

Inspect `src/` first. Canonical new work: `src/components/{domain}/`. If the repo uses legacy modules at `src/` root, follow that tree — do not migrate it inside a feature PR. TypeORM vs Prisma is a fork (`piximind-architecture-nestjs` vs `piximind-architecture-prisma`).

| Domain | Skill to prepare | Essentials | Requirements |
|--------|------------------|------------|--------------|
| **Clean architecture** | `piximind-architecture-nestjs` | Inspect `src/` first. Canonical: `src/components/{domain}/` = module + controller + service + repository; `src/entities`, `src/dto/{domain}`, `src/interfaces/{domain}`, `src/common` (guards, DB, strategies). Tree: `skills/piximind-architecture-nestjs/references/tree.md`. | Controller is HTTP only. Business rules in service. Persistence in repository. DTOs validate input (`class-validator`). Do not put Prisma/Mongoose calls in controllers. Do not introduce a `domain/` folder unless the repo already has one. Config via existing `ConfigService` / env interfaces — never hardcoded secrets (JS/TS security skill). |
| **TDD** | `piximind-tdd-nestjs` | Jest. Colocate `*.spec.ts` next to the unit under test (service/controller). E2E stays in `test/`. Seams: service public methods; HTTP contract for e2e. | Mock repositories/providers, not the class under test. Cover auth guards on protected routes. No real Keycloak/DB in unit tests (`src/test/*-mock.ts` pattern). Vertical slices: one behavior per spec case. |
| **Offline-first** | *not in Wave 1* | APIs stay online. Offline is a **client** concern. | Nest skills may mention idempotent writes and cache headers if a client already depends on them. Do not invent server-side “offline mode”. |
| **Atomic design** | n/a | Backend has no UI atoms. | Reject any Nest “atomic” skill. |
| **SEO** | n/a | JSON APIs are not indexable pages. | If a Nest app serves public HTML, that is an exception — call it out in review. Otherwise omit. |
| **Security** | shipped (`piximind-security-js-ts`) | Env discovery, validation, zero-trust payloads. | New Nest architecture/TDD skills must defer secrets and validation to this skill. |

### 4.3 Next.js (App Router + TS)

Inspect the existing App Router app (`src/app`, `server-side` / `client-side` / `common-side`). If the repo uses `src/features/` instead, follow that tree — do not migrate it.

| Domain | Skill to prepare | Essentials | Requirements |
|--------|------------------|------------|--------------|
| **Clean architecture** | `piximind-architecture-nextjs` | `src/app/` = routes, layouts, `generateMetadata`. `src/server-side/api/{domain}/` = `*.api.ts` + `*.api.interface.ts` + `*.mapper.ts`. `src/server-side/router` = backend URL map. `src/client-side/` = DS + pages. `src/common-side/` = shared API types/services. `src/i18n/` = next-intl. | Do not fetch Nest directly from client atoms. Mappers belong in server-side API modules. `"use client"` only at DS/pages that need it. Do not introduce FSD (`entities/widgets/features`) or a second API layer. |
| **TDD** | `piximind-tdd-nextjs` | Seams: mappers (pure), API modules (mocked fetch), organisms (RTL, public props), `generateMetadata` (return shape). | No snapshot-only UI tests. Do not assert internal `useState`. Prefer Vitest/Jest already in the repo. Vertical slices. |
| **Offline-first / PWA** | `piximind-offline-nextjs` | Existing `ServiceWorkerRegister`, `InstallPrompt`, `manifest`. Cache **static shell + agreed GET reads**. | Never cache tokens or PII in SW. Respect `NEXT_PUBLIC_` vs server secrets (security skill). Offline UX must show a degraded state, not a blank page. Do not add a random Workbox stack if the project already has a SW. |
| **Atomic design** | `piximind-atomic-web` (shared with React) | `src/client-side/ds/{atoms,molecules,organisms}` + colocated `*.interface.ts`. Prefix `Atom*`, `Molecule*`, `Organism*`. | Atoms wrap native HTML only (see `AtomDiv`). Organisms may use feature types; atoms must not import `server-side`. Pages live under `client-side/pages` or `app/`, composing organisms. |
| **SEO** | `piximind-seo-web` | Per-route `generateMetadata` (title template, description, canonical, `metadataBase`). `sitemap` / `robots`. Locale + hreflang via next-intl (`[locale]`). Indexable content in RSC, not only after client fetch. Images: `next/image` + alt. Optional JSON-LD on public pages. Core Web Vitals. | Auth-gated apps: `noindex` private routes. Do not ship identical title/description on every locale layout. Public marketing pages ≠ back-office. Audit checklist like `seo-audit` on skills.sh: crawl, metadata, indexability, i18n, structured data. |
| **Security** | shipped | Env prefixes, no secret leakage to client, Zod/class-validator. | SEO/PWA skills must not move secrets into `NEXT_PUBLIC_`. |

### 4.4 React.js (TS) — admin / AdminJS / non-Next SPA

Inspect first: AdminJS (`src/admin/components`) vs Vite SPA (`Page` / `DesignSystem`) vs a DS used outside App Router.

| Domain | Skill to prepare | Essentials | Requirements |
|--------|------------------|------------|--------------|
| **Clean architecture** | `piximind-architecture-react` | Inspect first. AdminJS: `pages/` / `actions/` / `properties/` then domain; register in `register-components.ts` only. SPA: reuse `ds/{atoms,molecules,organisms}` or the repo’s `components/base` + `components/application`. | Do not dump React into Nest `resources/`. Do not invent FSD. Business rules stay out of presentational atoms. |
| **TDD** | `piximind-tdd-react` | RTL at component public props. Colocate `*.spec.ts` / `*.test.tsx` as the repo already does (AdminJS actions next to the action). | Mock ESM-hostile libs the way the repo already mocks `adminjs`. Test handlers and visibility, not CSS. |
| **Offline-first** | only if the SPA is a PWA | Same rules as Next PWA. | Most AdminJS apps are online-only — say so in the skill. Do not add SW by default. |
| **Atomic design** | `piximind-atomic-web` | Same skill as Next. If the repo uses `components/base` instead of `ds/`, map: base ≈ atoms/molecules, application ≈ organisms. | One naming system **per repo**. Do not mix `AtomButton` and `components/base/buttons` in a new screen. |
| **SEO** | usually n/a | Authenticated admin UIs are `noindex`. | If a React app is a public site, reuse `piximind-seo-web` patterns (Helmet/metadata). |
| **Security** | shipped | XSS (`dangerouslySetInnerHTML`), authz in AdminJS actions. | Atomic/architecture skills must flag raw HTML and defer to the JS/TS security skill. |

---

## 5. Wave 1 delivery list (prepare these PRs)

| # | Skill folder | Tech | Domain | Priority |
|---|--------------|------|--------|----------|
| 1 | `piximind-architecture-flutter` | Flutter | Clean architecture | P0 |
| 2 | `piximind-architecture-nestjs` | NestJS | Clean architecture | P0 |
| 3 | `piximind-architecture-nextjs` | Next.js | Clean architecture | P0 |
| 4 | `piximind-architecture-react` | React | Clean architecture | P0 |
| 5 | `piximind-tdd-flutter` | Flutter | TDD | P0 |
| 6 | `piximind-tdd-nestjs` | NestJS | TDD | P0 |
| 7 | `piximind-tdd-nextjs` | Next.js | TDD | P0 |
| 8 | `piximind-tdd-react` | React | TDD | P0 |
| 9 | `piximind-offline-flutter` | Flutter | Offline-first | P0 |
| 10 | `piximind-offline-nextjs` | Next.js | Offline / PWA | P1 |
| 11 | `piximind-atomic-flutter` | Flutter | Atomic design | P0 |
| 12 | `piximind-atomic-web` | Next.js + React | Atomic design | P0 |
| 13 | `piximind-seo-web` | Next.js | SEO | P0 |
| 14 | `piximind-seo-flutter` | Flutter | SEO / ASO | P1 |

Security + visual design: already deployed. Do not include them in Wave 1 PRs except to **link** them.

---

## 6. How to submit for review

1. Branch: `skill/{skill-name}` from `main` in `agent-skills`.
2. One skill per PR.
3. PR description: which **tree** you encoded, which **seams**, which **anti-patterns**. No project or customer names.
4. Tech Lead reviews against section 3. Merge = deploy to the team Cursor setup.
