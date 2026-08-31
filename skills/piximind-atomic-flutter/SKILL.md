---
name: piximind-atomic-flutter
description: Enforces Flutter atomic structure and prefixes (atom_, molecule_, organism_, template_) plus DS-typed widget params (*Params or named constructors, never dynamic Maps). Use when adding Flutter widgets, pages, skeletons, sheets, or tokens — not for palette (piximind-frontend-design).
paths:
  - "**/lib/core/common/presentation/**"
  - "**/atom_*.dart"
  - "**/molecule_*.dart"
  - "**/organism_*.dart"
  - "**/template_*.dart"
---

# Flutter atomic design

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

Atomic design owns **structure and naming**. Palette, type, and “make it distinctive” stay in `piximind-frontend-design` / `tokens/`. Read `design.md` / `DESIGN.md` (or Figma MCP via that skill) before inventing colors.

Inspect first. Do not dump the presentation tree into chat.

## Anti-patterns
- **Do NOT** put one-off `Container` / `Column` soup in `features/*/presentation` pages. Compose templates and organisms.
- **Do NOT** let atoms import `lib/features/...`.
- **Do NOT** mix prefixes (`AtomButton` vs `atom_button`) in a new file. Use snake prefixes: `atom_`, `molecule_`, `organism_`, `template_`.
- **Do NOT** store colors or text styles on the widget when they belong in `tokens/`.
- **Do NOT** create a second design system folder (`ui_kit/`, `components/`) next to `lib/core/common/presentation`.
- **Do NOT** widget-test private inner widgets; public API of organisms/templates only (`piximind-tdd-flutter`).
- **Do NOT** pass widget props as `Map`, `Map<String, dynamic>`, untyped JSON, or `dynamic`. Never `dynamic` for widget props unless the repo already documents that exception.
- **Do NOT** rebuild the whole tree on every BLoC tick when `buildWhen` / `BlocSelector` / `const` constructors already exist in the nearest widget.

## Approved patterns
- **Do** place shared UI in `lib/core/common/presentation/{atoms,molecules,organisms,templates,skeletons,sheets,tokens}`.
- **Do** prefix files and widgets: `atom_`, `molecule_`, `organism_`, `template_`.
- **Do** keep pages in `features/{feature}/presentation` as composition of templates/organisms + BLoC.
- **Do** keep atoms wrapping Flutter primitives only (Text, Icon, GestureDetector, etc.) with token-based styling.
- **Do** put loading placeholders in `skeletons/` and bottom sheets in `sheets/`.
- **Do** reuse `template_scaffold` (connectivity banner lives there — `piximind-offline-flutter`).
- **Do** type widget props like the DS: named constructor params, `required` where the value is mandatory, concrete types (`String`, enums, `VoidCallback`, entity classes).
- **Do** match the repo’s params convention (inspect nearest organism/template):
  - If `lib/core/common/models/params/` (or equivalent) exists → add `{Widget}Params` there (`organism_*_params.dart` / `template_*_params.dart`) and take `final XxxParams params` on the widget.
  - Else if a colocated `*_params.dart` sits next to the widget → follow that.
  - Else → typed named fields on the widget constructor (atoms usually stay here). Small helper types may live in the same file.
- **Do** prefer `const` constructors. Avoid work in `build` that belongs in a BLoC or a params factory.

## Seams (test / depend here)
- Organism and template **public constructors** (`XxxParams` or typed named params + callbacks).
- Token getters (colors, typography) — depend on tokens, not hardcoded `Color(0xFF…)`.
- Feature pages depend on templates/organisms, not on other features’ private widgets.
- Atoms have no feature imports (enforced by review / analyzer where present).

## Workflow
1. Locate `lib/core/common/presentation` and follow the nearest atom/molecule/organism — including how **that** widget types its props.
2. If the UI is feature-specific composition, put it in `features/*/presentation`, not in core.
3. Run the checklist. `dart analyze` on touched files.
4. If HTML/webview, tokens, or PII display is involved, also follow `piximind-security-flutter`. Visual identity: `piximind-frontend-design`.

## Checklist
- [ ] New shared widget lives under the correct atomic folder with the correct prefix.
- [ ] Atoms import no features.
- [ ] Pages compose templates/organisms; no layout soup.
- [ ] Colors/typography come from `tokens/` (after `design.md` / Figma if present).
- [ ] Widget props are named + typed; no `dynamic` / ad-hoc `Map` props.
- [ ] Organism/template params follow the repo’s `*Params` / `*_params.dart` convention.
- [ ] No second DS folder.
- [ ] Skeletons/sheets used instead of ad-hoc loaders/modals when those folders already cover the case.
- [ ] Sonar/analyzer: no unused fields, empty catches, or duplicated widget trees; `const` where the nearest widgets use it.

## Token efficiency
- Inspect the nearest widget; do not paste the atomic tree into chat.
- One concern: structure, naming, and typed props. Palette → `piximind-frontend-design`. Layers → `piximind-architecture-flutter`.

## Out of scope
- Choosing a new brand palette (visual identity skill).
- Domain/data layers (`piximind-architecture-flutter` when present).
- Store listing / ASO (`piximind-seo-flutter`).
