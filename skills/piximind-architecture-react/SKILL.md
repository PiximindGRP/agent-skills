---
name: piximind-architecture-react
description: Enforces React CSR layout — Vite Page→Template→DesignSystem plus {Domain}API with typed route/response interfaces, or AdminJS pages/actions/properties. Use when adding or reviewing React SPA, Vite, CSR, AdminJS UI, or TypeScript contracts. Not for Next.js App Router or RSC.
paths:
  - "**/src/admin/**"
  - "**/src/Page/**"
  - "**/src/DesignSystem/**"
  - "**/register-components.ts"
---

# React architecture (SPA / AdminJS)

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

If the repo is Next.js App Router (`src/app/`, `server-side/`, `client-side/`, `"use client"`, RSC), stop — use `piximind-architecture-nextjs`.

Vite SPA = `src/Page` + `DesignSystem` + `{Domain}API`. AdminJS = `src/admin/components`. Trees: [references/tree.md](references/tree.md).

## Anti-patterns
- **Do NOT** invent Feature-Sliced Design, hexagonal ports, UseCase folders, or `src/{data,domain,presentation}`.
- **Do NOT** dump `.tsx` into Nest `src/resources/`. Resources reference AdminJS components by **string name** only.
- **Do NOT** put Prisma, Nest services, or `isAccessible` logic in presentational atoms.
- **Do NOT** fetch Nest (or `{Domain}API`) from `DesignSystem/Atoms` / `ds/atoms`. Business rules stay in pages, AdminJS handlers, or `{Domain}API`.
- **Do NOT** mix naming systems on a new screen (`AtomButton` vs `components/base/buttons`). One system per repo (`piximind-atomic-web`).
- **Do NOT** add React Query, Zustand, or a second router if the repo already has Redux + `react-router-dom` or AdminJS `ApiClient`.
- **Do NOT** add a service worker by default. Most AdminJS apps are online-only. If `src/admin/offline/` already exists, do not invent a second stack.
- **Do NOT** use `dangerouslySetInnerHTML` without `piximind-security-js-ts`. Authenticated admin UIs stay `noindex` (SEO skill only if the app is a public site).
- **Do NOT** use `any`, `as any`, `Record<string, any>`, implicit any, or `Promise<any>` on API methods, JWT payloads, or DS props.
- **Do NOT** inline API path strings when a router/config const exists. Do not declare fetch response shapes only inside a Page.
- **Do NOT** inline DS props as `type Props = {` in the `.tsx`. Props live in `Interface/` (Vite) or the repo’s interfaces folder (`piximind-atomic-web`).

## Approved patterns
- **Do** (Vite SPA): `Page/{Name}` fetches via `Api/{Domain}` (`{Domain}API`) + Redux + existing domain hooks, then renders `Template/{Name}`. Templates compose `DesignSystem/{Atoms,Molecules,Organisms}` (`OrganismTable`). Types in `Interface/` mirroring those folders — `IAtom*`, `ITemplate*`, `IPage*`, plus `I{Domain}{Action}Response` for HTTP.
- **Do** keep `{Domain}API` as HTTP-only (`StandardApi` + `Config.getInstance()` / `import.meta.env.REACT_APP_*`). Exported methods have explicit `Promise<I{Domain}…Response>`. New routes go through `Route/Router.tsx` + `AccessVerification` + `ACCESSES`.
- **Do** (AdminJS): UI under `src/admin/components/{pages,actions,properties}/{domain}/`. Register paths in `register-components.ts` only; `component-loader.ts` adds/overrides. Wire `component: '{ActionName}'` from `src/resources/{domain}/`. Type handler payloads with named interfaces (colocate `*.interface.ts` if `Interface/` is absent). Authz stays on the server (`isAccessible` / `isVisible`).
- **Do** reuse the repo’s DS. `Atom*` / `OrganismTable` outside App Router: atomic skill. If you find `components/base` + `application`, map base ≈ atoms, application ≈ organisms — do not also add `DesignSystem/`.
- **Do** keep TypeScript strict. `jwtDecode<IDecodedToken>(token)` — never untyped / `as any`. No secrets or refresh tokens on types imported by atoms.
- **Do** keep cognitive complexity down: HTTP in `{Domain}API`, mapping next to it or in an existing converter, Page only wires props. No unused locals.

## Seams (test / depend here)
- Vite SPA: `{Domain}API` public methods + `I{Domain}*Response`; `Page` → `Template` props (`IPage*` / `ITemplate*`); `AccessVerification` / `ACCESSES`; `IOrganismTable` (`piximind-tdd-react`).
- AdminJS: action handlers + `isVisible` / `isAccessible` in `src/resources/**`; registered component public props; do not unit-test `register-components.ts` unless registration is the bug.
- Atoms: public HTML/props only (`IAtom*`) — no API, no role checks.

## Workflow
1. Detect the tree: AdminJS `src/admin/components` vs Vite `src/Page` + `DesignSystem` vs existing `ds/` / `components/base`.
2. Follow the nearest screen in **that** tree. Match folder case (PascalCase vs kebab files). Match the existing interfaces folder — do not mix colocated and `Interface/`.
3. Run the checklist.
4. Auth, secrets, PII, or HTML → `piximind-security-js-ts`. Tests → `piximind-tdd-react`. Structure/naming → `piximind-atomic-web`. Palette → `piximind-frontend-design`.

## Checklist
- [ ] Inspected `src/` and copied an existing feature, not a textbook Clean Architecture / FSD layout.
- [ ] Not Next App Router work (that skill is `piximind-architecture-nextjs`).
- [ ] Vite SPA: new screen is Page → Template → DS; HTTP only in `{Domain}API`; route + `ACCESSES` wired; responses are named interfaces.
- [ ] AdminJS: `.tsx` under `admin/components/{pages|actions|properties}/…`; registered in `register-components.ts`; resources use string names only.
- [ ] Atoms have no fetch, Prisma, or permission gates. DS props are `IAtom*` / `IMolecule*` / `IOrganism*` in the interfaces folder.
- [ ] Exported functions have return types; no `any`; JWT typed; no secrets on client DS types.
- [ ] Single atomic naming system on the new screen. No new SW / FSD / second HTTP client stack.
- [ ] Raw HTML / tokens flagged for `piximind-security-js-ts`.

## Out of scope
- Next.js App Router, RSC, `generateMetadata`, `server-side` mappers (`piximind-architecture-nextjs`).
- Nest module layout (`piximind-architecture-nestjs`). Do not “migrate” AdminJS `resources/` into Nest `components/{domain}`.
- Adding PWA to an online-only admin. If admin offline helpers already exist, do not duplicate them here.
- Palette / visual identity (`piximind-frontend-design`). SEO for public sites (`piximind-seo-web`).

## Token efficiency
- Inspect `src/`; do not paste SPA/AdminJS trees into chat — use [references/tree.md](references/tree.md).
- One concern: React layers + typed HTTP/DS contracts. Tests → `piximind-tdd-react`. Secrets → `piximind-security-js-ts`.
