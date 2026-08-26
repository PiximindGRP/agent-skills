# Flutter feature slice

New business features use `lib/features/{feature}/` with `domain/`, `data/`, and `presentation/`. Older features may put concrete repositories in `domain/` and call `ApiRepository` / Firebase there. **Do not copy those leaks for new work.**

## Feature tree

```text
lib/features/{feature}/
  domain/
    entities/{feature}.dart                 # Equatable; no Flutter
    repositories/{feature}_repository.dart  # abstract class (contract)
    usecases/{action}_{feature}_usecase.dart
  data/
    datasources/{feature}_local_datasource.dart
    datasources/{feature}_remote_datasource.dart
    repositories/{feature}_repository_impl.dart
  presentation/
    {feature}_page.dart                     # pages / sheets / feature widgets
```

Shared (do not duplicate per feature):

```text
lib/core/
  blocs/{name}/bloc/          # app-wide BLoCs (auth, home, network, …)
  common/presentation/        # atoms / molecules / organisms / templates (atomic skill)
  common/repositories/        # IApiRepository, Floor, storage
  interfaces/repositories/    # ApiRepository, FloorRepo contracts
  services/injector/injector.dart
  services/routing/router.dart
  config/api/api_config.dart
```

## Dependency rule

```text
presentation  →  domain (UseCase, entities)  +  core BLoCs / atomic widgets
data          →  domain contracts  +  core ApiRepository / Floor / Storage
domain        →  Dart only (entities, abstract Repository, UseCase)
core/blocs    →  UseCases or repository contracts
```

`presentation` and `data` may import `lib/core`. `domain` must not.

## DI order (`injector.dart`)

Bind the **contract** type to the impl:

```dart
injector.registerLazySingleton<FeatureLocalDataSource>(
  () => FeatureLocalDataSource(injector<StorageRepository>()),
);
injector.registerLazySingleton<FeatureRemoteDataSource>(
  () => FeatureRemoteDataSource(injector<ApiRepository>()),
);
injector.registerLazySingleton<FeatureRepository>(
  () => FeatureRepositoryImpl(
    local: injector<FeatureLocalDataSource>(),
    remote: injector<FeatureRemoteDataSource>(),
  ),
);
injector.registerLazySingleton<SubmitFeatureUseCase>(
  () => SubmitFeatureUseCase(injector<FeatureRepository>()),
);
```

HTTP: inject `ApiRepository` (abstract). The class that implements it is `IApiRepository`. Floor follows the same inverted names (`FloorRepo` / `IFloorRepo`). Do not rename.

New app-wide BLoC: `registerLazySingleton` here, then `BlocProvider(create: (_) => injector<XBloc>())` in `main.dart` `MultiBlocProvider`.

## UseCase

```dart
class SubmitFeatureUseCase {
  const SubmitFeatureUseCase(this._repository);
  final FeatureRepository _repository;

  Future<bool> call(FeatureEntity entity) =>
      _repository.submit(entity);
}
```

Presentation calls `injector<SubmitFeatureUseCase>()(entity)` or a BLoC event that calls `useCase.call`. Do not call datasources from widgets.

## Adding a domain (checklist order)

1. Entity under `domain/entities/`.
2. Abstract repository under `domain/repositories/`.
3. Use case(s) with `call`.
4. Local and/or remote datasource; remote uses `ApiRepository` + `ApiConfig`.
5. `*_repository_impl.dart` implementing the contract.
6. Presentation page/sheet composing templates/organisms.
7. Register the chain in `injector.dart`.
8. Route in `AppRouter` + `Paths` if it is a screen.
9. If it needs session/cache/connectivity, reuse existing auth / Floor / network BLoCs — do not add a second DB.

## Variants

| Pattern | Where | What to do |
|---------|--------|------------|
| Full CA slice | `features/{feature}/` with domain + data + presentation | **Template for new business features.** |
| Use cases + leaky domain repo | some existing features | Extend in place; do not add Flutter imports to new domain files. |
| Concrete repo in domain calling API | older `domain/repositories/` | Match that file if you are inside that feature; do not use as a new-feature template. |
| Presentation-only page | features that are pages only | Add a page that talks to existing BLoCs; do not invent empty domain/data folders. |
| BLoCs in `core/blocs` | whole app | Keep them there. Feature folders hold pages, not a parallel BLoC tree. |
