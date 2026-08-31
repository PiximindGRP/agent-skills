# React trees (SPA / AdminJS)

Not Next.js App Router. If you see `src/app/`, `server-side/`, `client-side/`, or `"use client"`, stop and use `piximind-architecture-nextjs`.

## 1. Vite CSR SPA

Stack: React + Vite + TypeScript, `react-router-dom` v6, Redux Toolkit + persist, `StandardApi` HTTP.

```text
src/
  Api/{Domain}/{Domain}.ts     {Domain}API — HTTP only via StandardApi
  StandardApi/                 shared GET/POST wrapper + Httpstatus
  Common/Config/Config.ts      import.meta.env.REACT_APP_* (API_URL, SOCKET_URL)
  Common/Data/accesses.ts      ACCESSES keys for AccessVerification
  Page/{PageName}/             route containers: fetch, Redux, access hooks, pass props
  Template/{TemplateName}/     layout: compose DesignSystem, no new HTTP clients
  DesignSystem/
    Atoms/{AtomX}/             AtomButton, AtomText, …
    Molecules/{MoleculeX}/
    Organisms/{OrganismX}/     OrganismTable, OrganismAgenda, …
  Interface/                   mandatory: IAtom*, IMolecule*, IOrganism*, ITemplate*, IPage* mirroring those folders; I{Domain}*Response for Api/{Domain}
  Modal/
  Route/Router.tsx             BrowserRouter, lazy pages, Private/Public
  Route/AccessVerification/    ACCESSES + moduleGaurd
  Redux/{Store,Reducers}/      auth, domain, setting, modal, toast, infra, files
  CustomHook/hooks/            domain and access hooks
  Providers/                   ModalProvider, LangProvider, ToastProvider
  Lang/  Enum/  Socket/
```

Dependency arrows:

```text
Route + AccessVerification  →  Page
Page                        →  {Domain}API + Redux + hooks + Template
Template                    →  DesignSystem (Atom / Molecule / Organism)
Atom                        →  props only (no Api, no ACCESSES, no feature Redux)
{Domain}API                 →  StandardApi + Config.getInstance()
```

Follow an existing slice (`Page{Name}` + `Template{Name}` + `Api/{Domain}`) for new screens. Interfaces live under `Interface/`, not next to the component (this tree’s split; some repos colocate `*.interface.ts` — do not mix in one repo). `{Domain}API` methods return `Promise<I{Domain}…Response>`. No `any`. DS props are never inline in the `.tsx` (`piximind-atomic-web`). JWT: `jwtDecode<IDecodedToken>(token)`.

Env: `import.meta.env.REACT_APP_*` via `Config`. Do not invent `NEXT_PUBLIC_` here.

## 2. AdminJS UI

React lives under `src/admin/`. Nest AdminJS handlers live under `src/resources/` — **not** React.

```text
src/admin/
  options.ts                   AdminJS options, dashboard, assets
  component-loader.ts          ComponentLoader.add / .override from register maps
  components/register-components.ts
    ADMIN_PAGES / ADMIN_ACTIONS / ADMIN_PROPERTIES / ADMIN_OVERRIDES
  components/pages/{domain}/   list pages, dashboards, custom pages
  components/actions/{domain}/ custom actions, guarded-edit
  components/actions/shared/
  components/properties/{domain}/
  components/properties/shared/   datetime, user-reference, shared fields
  components/app/              TopBar, ActionHeader, Sidebar* (core overrides)
  theme/  locale/  hooks/  context/  utils/
  offline/                     existing PWA helpers — do not add a second SW stack

src/resources/{domain}/        Prisma + AdminJS handlers, isAccessible / isVisible
  {domain}.resource.ts         components: { edit: '{PropertyName}' }  ← string names only
  actions/{action}.action.ts   component: '{ActionName}', handler, authz
```

Registration: add the kebab path in `register-components.ts`, then `componentLoader.add` picks it up. Resource files only reference the **PascalCase registry name**, never a `.tsx` import.

Type action/page props as named interfaces (colocate `*.interface.ts` if `Interface/` is absent). No `any` on `ApiClient` responses or JWT payloads.

Authz seam is server-side: `isAccessible` / `isVisible` in `src/resources/**`. The React action may hide chrome; it is not the permission gate.

## 3. Other layouts (follow if already present)

- Atomic `ds/` **without** App Router: `ds/{atoms,molecules,organisms}` + `Atom*` / `OrganismTable` — atomic skill. If `src/app/` exists, this is Next — other skill.
- `components/base` + `components/application`: map base ≈ atoms/molecules, application ≈ organisms (`piximind-atomic-web`). Only follow if the repo already has it.
