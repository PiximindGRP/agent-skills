# Design-system props as interfaces

Inspect the repo first. One interface layout **per repo**. Do not mix.

## Folder (match whichever exists)

| If you find | Put new DS props here |
|-------------|------------------------|
| `src/common/interface/DS/{Atoms,Molecules,Organisms}/` | `atomInput.interface.ts` → `IAtomInput`. Re-export from that folder’s `index.ts` and the interface barrel. |
| `src/client-side/ds/{atoms,molecules,organisms}/*.interface.ts` | Colocate next to the component (`AtomInput.interface.ts`). |
| `ds/{atoms,molecules,organisms}/interfaces/` | `interfaces/atomInput.interface.ts`. |
| `src/Interface/` (Vite SPA) | `Interface/` mirroring `DesignSystem/{Atoms,Molecules,Organisms}`. |
| `components/base` + `application` | Props interfaces next to the primitive, same naming the repo already uses. Do not also add `DS/`. |

Never invent `common/interface/DS/` in a repo that already colocates `*.interface.ts`.

## Shape

```ts
export interface IAtomInput extends IClassName, IId, IValue {
  type?: "text" | "password" | "email";
  onChange?: (event: React.ChangeEvent<HTMLInputElement>) => void;
  disabled?: boolean;
}
```

- Prefix: `IAtom*`, `IMolecule*`, `IOrganism*`.
- Shared primitives (`IId`, `IClassName`, `IValue`, `IStatus`) live in the existing DS default/base interface file — extend them; do not duplicate.
- The `.tsx` imports the interface; it does **not** declare `type Props = { … }` inline.
- Organisms may take feature types (`IOrganismTable<TSchema>`). Atoms stay generic — no API response types, no router consts, no `server-side`.
- Page / template / modal props follow the same rule when the repo already has `common/interface/templates` or `Interface/Template` (`I{Name}Template`, `I{Name}Modal`). Do not invent that folder if it is absent — colocate `*.interface.ts` instead.

## Component seam

```ts
const AtomInput = memo(
  React.forwardRef((props: IAtomInput, ref: React.Ref<HTMLInputElement>): JSX.Element => { … }),
);
```

Exported components and helpers have an explicit return type (`JSX.Element`, `void`, …). `"use client"` only when the atom/molecule/organism uses hooks or DOM events.
