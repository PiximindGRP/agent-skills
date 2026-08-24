---
name: piximind-tdd-nextjs
description: Enforces vertical-slice TDD for Next.js App Router — pure mappers, mocked server-side API modules, RTL at organism public props, typed fixtures with no any. Use when adding Next.js features, tests, or reviewing App Router frontends (Vitest/Jest already in the repo).
paths:
  - "**/*.{ts,tsx}"
  - "**/*.{spec,test}.{ts,tsx}"
---

# Next.js TDD

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

Reference tree: `server-side` / `client-side` / `common-side`. Prefer the test runner already in the repo (Vitest or Jest). Do not add a second runner.

Contracts under test are the named interfaces from `piximind-architecture-nextjs` (`I{Domain}*Response`, `IOrganism*`).

## Anti-patterns
- **Do NOT** write all tests then all code. One failing behavior → minimal code → repeat.
- **Do NOT** ship snapshot-only UI tests as the sole coverage.
- **Do NOT** assert internal `useState` or private hooks. Test public props, rendered roles, and callbacks.
- **Do NOT** fetch Nest from client atoms inside a test (or in production code). Server-side API modules own fetch.
- **Do NOT** introduce Feature-Sliced `entities/widgets/features` or a second API layer to “make testing easier”.
- **Do NOT** put secrets in `NEXT_PUBLIC_` fixtures (`piximind-security-js-ts`).
- **Do NOT** use `any`, `as any`, or untyped JWT payloads in fixtures. Do not silence the compiler to make a test compile.

## Approved patterns
- **Do** test mappers as pure functions (`*.mapper.ts` in `src/server-side/api/{domain}/`). Input/output typed as the domain response / UI interfaces.
- **Do** test API modules with mocked `fetch` (or the repo’s HTTP helper), asserting router-const URL + mapped `I{Domain}*Response`.
- **Do** widget-test organisms with React Testing Library at public props (`IOrganismTable`, not inner cells).
- **Do** assert `generateMetadata` return shape (title, description, alternates/canonical) per route.
- **Do** name tests after behavior (`maps DTO to table row`, `metadata uses locale title`).
- **Do** colocate `*.spec.ts` / `*.test.tsx` the way the repo already does. Exported test helpers have explicit return types.

## Seams (test / depend here)
- Mappers (pure).
- Server-side API modules (`*.api.ts` + `*.api.interface.ts`) with mocked fetch.
- Organism public props (`IOrganism*`, RTL).
- `generateMetadata` return value.
- Do not treat `src/server-side/router` URL strings as something to re-implement in the client test — assert the existing key.

## Workflow
1. Locate existing specs and the App Router tree (`src/app`, `src/server-side`, `src/client-side`).
2. Start at the smallest seam that encodes the change (usually a mapper).
3. Add the failing test, then the minimum implementation. Repeat.
4. Run the repo’s test script. Do not add MSW unless the repo already uses it.
5. If auth, secrets, PII, or HTML are involved, also follow `piximind-security-js-ts`. SEO assertions: `piximind-seo-web`.

## Checklist
- [ ] Runner matches the repo (Vitest/Jest); no second harness.
- [ ] Mappers / API modules / organisms / metadata covered at seams — not internals.
- [ ] Fixtures use `I{Domain}*Response` / `IOrganism*`; no `any`, no live tokens or PII.
- [ ] No snapshot-only UI coverage. Client tests do not call Nest directly.
- [ ] Test command is green for the new files.
- [ ] Sonar-style: no unused fixtures, empty `catch` in tests, or duplicated mapper assertions. Coverage at seams only.

## Out of scope
- Adding a PWA/service-worker test stack (see `piximind-offline-nextjs`).
- Testing CSS or pixel layout.
- AdminJS / non-Next SPA (use `piximind-tdd-react`).

## Token efficiency
- Inspect existing specs; do not paste the App Router tree into chat.
- One concern: tests. Layout → architecture skill. SEO assertions → `piximind-seo-web`.
