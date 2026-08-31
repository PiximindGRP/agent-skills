---
name: piximind-atomic-web
description: Enforces web atomic structure for Next.js and React — Atom*/Molecule*/Organism* with props as interfaces in an interfaces folder. Use when adding UI in Next.js App Router or React SPAs (including AdminJS). Not for palette (piximind-frontend-design).
paths:
  - "**/ds/**"
  - "**/atoms/**"
  - "**/molecules/**"
  - "**/organisms/**"
  - "**/components/base/**"
  - "**/components/application/**"
  - "**/DesignSystem/**"
---

# Web atomic design (Next.js + React)

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

One naming system **per repo**. Do not mix `AtomButton` and `components/base/buttons` on a new screen.

DS props **must** be named interfaces in the repo’s interfaces folder — never inline in the `.tsx`. Folder map and examples: [references/ds-interfaces.md](references/ds-interfaces.md).

Read `design.md` / `DESIGN.md` first if present. If Figma MCP is connected, pull tokens/components via `piximind-frontend-design` — do not invent a palette.

Inspect first. Do not dump the DS tree into chat.

## Anti-patterns
- **Do NOT** let atoms import `server-side`, Nest clients, feature API modules, router path maps, or JWT/session types.
- **Do NOT** fetch Nest directly from client atoms.
- **Do NOT** invent Feature-Sliced `entities/widgets/features` next to `ds/` or `components/base`.
- **Do NOT** put business rules in presentational atoms (no role checks, no URL map, no DTO mapping).
- **Do NOT** use `dangerouslySetInnerHTML` without the JS/TS security skill.
- **Do NOT** start a second prefix system in a repo that already has one.
- **Do NOT** declare DS props as inline `type Props` / anonymous objects in the component file.
- **Do NOT** use `any` on props, callbacks, or icon maps. No `Record<string, any>`.
- **Do NOT** add `"use client"` to an atom that only wraps HTML with no hooks or events.

## Approved patterns
- **Do** (App Router): `src/client-side/ds/{atoms,molecules,organisms}` **or** `src/components/design-system/{atoms,molecules,organisms}` if that is the tree. Prefixes `Atom*`, `Molecule*`, `Organism*` (`AtomDiv` / `AtomInput` wrap native HTML only).
- **Do** define props as `IAtom*`, `IMolecule*`, `IOrganism*` in the existing interfaces location (`common/interface/DS/…`, colocated `*.interface.ts`, `ds/…/interfaces`, or Vite `Interface/`). Re-export from the existing barrel.
- **Do** extend shared primitives (`IId`, `IClassName`, `IValue`, `IStatus`) from the DS default/base interface file when it exists.
- **Do** put pages under `client-side/pages`, `components/pages`, or `src/app/`, composing organisms. `"use client"` only on DS/pages that need it — keeps the RSC bundle small.
- **Do** (AdminJS / some SPAs): map `components/base` ≈ atoms/molecules, `components/application` ≈ organisms. Register AdminJS pieces in `register-components.ts` only.
- **Do** allow organisms to use feature types (`IOrganismTable<TSchema>`); atoms stay generic.
- **Do** give exported components an explicit return type (`JSX.Element`). Leave palette to `piximind-frontend-design`.

## Seams (test / depend here)
- Organism public props (`IOrganismTable`, action callbacks) — RTL (`piximind-tdd-nextjs` / `piximind-tdd-react`).
- Atom public HTML wrapper props (`IAtomDiv`, `IAtomInput`).
- Page composition: page → organisms, never page → raw HTML soup when a DS atom exists.
- AdminJS: `pages/` / `actions/` / `properties/` public handlers.

## Workflow
1. Detect the repo’s system: `ds/{atoms,molecules,organisms}` **or** `components/design-system` **or** `components/base` + `application` **or** Vite `DesignSystem/`.
2. Detect the interfaces layout ([references/ds-interfaces.md](references/ds-interfaces.md)). Follow the nearest existing atom/organism; keep its prefix and interface folder.
3. Run the checklist.
4. If auth, secrets, PII, or HTML are involved, also follow `piximind-security-js-ts`.

## Checklist
- [ ] Single naming system used on the new screen.
- [ ] Props are `IAtom*` / `IMolecule*` / `IOrganism*` in the repo’s interfaces folder; `.tsx` does not declare them.
- [ ] Atoms wrap native HTML (or base primitives) only; no `server-side` / API / JWT imports; no `any`.
- [ ] Organisms may take feature types; pages compose organisms.
- [ ] No business rules in atoms. Explicit return types on exported components.
- [ ] `"use client"` only where hooks or events exist.
- [ ] Raw HTML flagged; security skill applied if present.
- [ ] AdminJS components registered only in `register-components.ts` when that is the app.

## Out of scope
- Visual identity / palette (`piximind-frontend-design`).
- Flutter atomic (`piximind-atomic-flutter`).
- Nest “atomic” skills (rejected).
- SEO metadata (`piximind-seo-web`).
- API route/response interfaces (`piximind-architecture-nextjs` / `piximind-architecture-react`).

## Token efficiency
- Inspect the nearest atom/organism + its `I*` file; do not paste the DS tree into chat — use [references/ds-interfaces.md](references/ds-interfaces.md).
- One concern: structure, naming, typed props. Palette → `piximind-frontend-design`. HTTP types → architecture skills.
