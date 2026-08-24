---
name: piximind-tdd-react
description: Enforces RTL TDD for React admin and AdminJS UIs at component public props, colocated *.spec.ts / *.test.tsx, typed fixtures with no any. Use when testing React SPAs, AdminJS actions/pages, or design-system components outside App Router.
paths:
  - "**/*.{ts,tsx}"
  - "**/*.{spec,test}.{ts,tsx}"
  - "**/src/admin/**"
---

# React TDD

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

Public props under test are the named interfaces from `piximind-architecture-react` / `piximind-atomic-web` (`IAtom*`, `IOrganism*`, `I{Domain}*Response`).

## Anti-patterns
- **Do NOT** write all tests then all components. Vertical slices only.
- **Do NOT** assert CSS classes, inline styles, or AdminJS theme tokens as the behavior.
- **Do NOT** dump React tests (or components) into a Nest `resources/` tree.
- **Do NOT** introduce Feature-Sliced Design to create test folders.
- **Do NOT** import ESM-hostile libraries for real in unit tests. Mock them the way the repo already mocks `adminjs`.
- **Do NOT** use `dangerouslySetInnerHTML` in test fixtures with unsanitized HTML (`piximind-security-js-ts`).
- **Do NOT** use `any` / `as any` in fixtures, JWT stubs, or `ApiClient` mocks.

## Approved patterns
- **Do** use React Testing Library at the component’s public props (`IAtom*` / `IOrganism*`) and roles.
- **Do** colocate `*.spec.ts` / `*.test.tsx` as the repo already does (AdminJS actions sit next to the action).
- **Do** test handlers, visibility, and authorization outcomes (action shown/hidden, confirm invoked) — not layout chrome.
- **Do** follow AdminJS seams when that is the app: `pages/` / `actions/` / `properties/` registered in `register-components.ts`.
- **Do** reuse the repo’s `ds/` or `components/base` + `components/application` naming; do not mix both in a new test file.
- **Do** type `{Domain}API` mocks as `I{Domain}*Response`. Name cases after behavior (`hides delete when role cannot delete`).

## Seams (test / depend here)
- Component public props and callbacks (`IAtom*`, `IOrganism*`, `IPage*` / `ITemplate*` when those exist).
- `{Domain}API` public methods + response interfaces (Vite SPA).
- AdminJS action handlers and `isVisible` / `isAccessible` (or the repo’s equivalent).
- Do not treat `register-components.ts` as something to unit-test unless registration itself is the bug.

## Workflow
1. Locate existing `*.test.tsx` / `*.spec.ts` and how `adminjs` (or other ESM-hostile deps) are mocked.
2. Write one failing test at the public seam, using the existing props/response interface.
3. Implement the minimum code. Repeat.
4. Run the repo’s test script (Jest/Vitest already present).
5. If authz, XSS, or HTML are involved, also follow `piximind-security-js-ts`. Next App Router: `piximind-tdd-nextjs`.

## Checklist
- [ ] Colocation matches the repo.
- [ ] RTL tests public props / handlers / visibility; fixtures match `I*` interfaces; no `any`.
- [ ] ESM-hostile libs mocked (`adminjs` pattern or local equivalent).
- [ ] No CSS-only assertions. No live secrets in fixtures.
- [ ] Test command is green.

## Out of scope
- Adding a service worker or PWA test harness to an online-only AdminJS app.
- Next.js `generateMetadata` / RSC (use `piximind-tdd-nextjs`).
- Snapshot-only suite as the only coverage.

## Token efficiency
- Inspect existing `*.test.tsx`; do not paste the AdminJS/SPA tree into chat.
- One concern: tests. Layout → `piximind-architecture-react`. Props → `piximind-atomic-web`.
