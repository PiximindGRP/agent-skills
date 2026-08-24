---
name: piximind-architecture-flutter
description: Enforces feature-first Clean Architecture — lib/features/{feature}/{domain,data,presentation}, UseCase + repository contracts, get_it, go_router, flutter_bloc. Use when adding or reviewing Flutter features, layers, DI, routing, or BLoCs.
paths:
  - "**/*.dart"
  - "**/*.yaml"
  - "**/*.arb"
---

# Flutter architecture

Inspect `lib/` first. Follow its existing folders. Do not introduce a second architecture.

New business features use `lib/features/{feature}/` with `domain/`, `data/`, and `presentation/`. Do not copy older features that put concrete repositories, Firebase, or `injector` inside `domain/`.

For the tree, DI order, and dependency arrows, read [references/examples.md](references/examples.md).

## Anti-patterns
- **Do NOT** add Riverpod, AutoRoute, or a repo-root `lib/{data,domain,presentation}` tree.
- **Do NOT** import Flutter, Floor, `http`, `get_it`, BLoC, or widgets into `domain/`.
- **Do NOT** put raw `http` calls in pages. HTTP goes through `ApiRepository` (`IApiRepository` impl) or a feature remote datasource.
- **Do NOT** invent Feature-Sliced Design, hexagonal ports, or a second design-system folder.
- **Do NOT** move existing BLoCs out of `lib/core/blocs/`. Register new app-wide BLoCs there + in `injector.dart` + `MultiBlocProvider` in `main.dart`.
- **Do NOT** instantiate BLoCs in widgets. Resolve from `get_it` / `context.read`. UI dispatches **events**, never custom BLoC methods.
- **Do NOT** “migrate” presentation-only features to full CA inside an unrelated PR.
- **Do NOT** store tokens in Floor / SharedPreferences. Defer to `piximind-security-flutter`.
- **Do NOT** pass widget props as `Map` / `dynamic`. Presentation widgets use typed named constructors or `*Params` (`piximind-atomic-flutter`).
- **Do NOT** dump `lib/` trees into chat. Read [references/examples.md](references/examples.md) instead.

## Approved patterns
- **Do** place new business features under `lib/features/{feature}/` with `domain/`, `data/`, `presentation/`. Shared UI and infra stay in `lib/core`.
- **Do** keep domain = Equatable entities + abstract `Repository` + `UseCase` with `call`. Data = datasources + `*_repository_impl.dart` (+ models if JSON is non-trivial).
- **Do** keep presentation = pages / feature widgets / sheets that compose `template_` / `organism_` widgets (`piximind-atomic-flutter`).
- **Do** register in `lib/core/services/injector/injector.dart`: datasource → impl bound to the **domain** repository type → use case → BLoC.
- **Do** add routes in `lib/core/services/routing/router.dart` and `Paths` in `routes.dart` (`go_router`). Shell screens go through `TemplateScaffold`.
- **Do** use `flutter_bloc` / `hydrated_bloc` already in the app. Immutable Equatable states (`copyWith`). `BlocBuilder` to render; `BlocListener` for navigation / toasts.
- **Do** use relative imports (`prefer_relative_imports` in `analysis_options.yaml`).
- **Do** reuse `ApiRepository` / Floor / `StorageRepository` from core. Offline reads: `piximind-offline-flutter`.
- **Do** type page → template/organism props (`*Params` under `core/common/models/params/` when that folder exists; otherwise typed constructors). Never `dynamic` for widget props unless the repo documents an exception.
- **Do** keep rebuilds cheap: `const` widgets, `BlocBuilder`/`BlocSelector` `buildWhen` when only a slice of state is needed. No `http` or Floor in `build`.

## Seams (test / depend here)
- `UseCase.call` (inject the repository **contract**).
- Domain `abstract class {Feature}Repository`.
- Datasource public methods (mock these when testing `*_repository_impl.dart`).
- BLoC event → state (`lib/core/blocs/...`).
- `ApiRepository` (contract name; implementation is `IApiRepository` — do not rename).
- Pages depend on use cases / BLoCs, not on Floor DAOs or `http`.
- Organism/template public params (`*Params` or typed constructors).

## Workflow
1. Locate `lib/features/` and the nearest complete slice (`domain` + `data` + `presentation`). If the file you touch is an older leaky domain type, match **that** file only — do not refactor the feature.
2. Match naming already used by complete slices (`*_usecase.dart`, `*_remote_datasource.dart`, `*_repository_impl.dart`).
3. Run the checklist. `dart analyze` on touched files.
4. If auth, secrets, PII, or storage are involved, also follow `piximind-security-flutter`. Tests: `piximind-tdd-flutter`. Offline HTTP: `piximind-offline-flutter`. UI structure: `piximind-atomic-flutter`.

## Checklist
- [ ] Inspected `lib/` and followed a complete `features/{feature}/` slice (or the domain already being edited).
- [ ] New feature lives under `features/{feature}/{domain,data,presentation}` — not repo-root layers.
- [ ] Domain has no Flutter / Floor / `http` / `injector` imports.
- [ ] Repository contract in domain; implementation + datasources in data.
- [ ] Use cases expose `call`; BLoCs depend on use cases or contracts, not datasources.
- [ ] Wired in `injector.dart`; new routes in `AppRouter` / `Paths`.
- [ ] New BLoC (if any) is in `core/blocs`, provided from get_it, event-driven.
- [ ] Pages compose templates/organisms; no second DS; no Riverpod / AutoRoute.
- [ ] Widget props typed (`*Params` or named constructors); no `dynamic` / `Map` props.
- [ ] Tokens stay out of Floor (`piximind-security-flutter`).
- [ ] Sonar/analyzer: unused locals, empty `catch`, duplicated layer code, high cognitive complexity in UseCases/BLoCs — fix or extract. Coverage at seams (`piximind-tdd-flutter`).

## Out of scope
- NestJS / Next.js / React trees (`piximind-architecture-nestjs` and later web skills).
- Palette / visual identity (`frontend-design-pixi`).
- Rewriting older domain leaks, or adding an architecture linter, unless the user asked.
- Inventing Isar/Hive/Dio when Floor + `http` via `ApiRepository` already exist.

## Token efficiency
- Inspect `lib/`; do not paste the feature tree into chat — use [references/examples.md](references/examples.md).
- One concern: layers, DI, routing, BLoC. UI prefixes → atomic skill. Tests → TDD skill.
