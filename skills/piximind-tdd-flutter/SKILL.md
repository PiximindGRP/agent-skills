---
name: piximind-tdd-flutter
description: Enforces red-green vertical-slice TDD for Flutter features at UseCase, Repository, and BLoC seams. Use when adding Flutter tests, implementing a feature test-first, or reviewing *_test.dart / bloc_test coverage.
paths:
  - "**/test/**"
  - "**/*_test.dart"
---

# Flutter TDD

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

If the repo has no `*_test.dart` yet, place new tests under `test/features/{feature}/...` mirroring `lib/features/{feature}/`. If the repo already has a test tree, follow that tree.

## Anti-patterns
- **Do NOT** write all tests then all production code (horizontal batching). One failing behavior → minimal code → next behavior.
- **Do NOT** test private widgets, private methods, or generated files.
- **Do NOT** mock UseCases when testing presentation. Mock datasources / repository **implementations** at the data seam; presentation talks to real UseCases + mocked `Repository` contracts.
- **Do NOT** assert a value you just hardcoded into the SUT (tautological tests). Expected values are independent fixtures.
- **Do NOT** put widget tests on pages that only assemble atoms. Widget-test organism / template **public API** only.
- **Do NOT** import Floor, `http`, or Flutter into domain tests that should stay pure Dart.
- **Do NOT** type widget-test props as `dynamic` / `Map`. Use `*Params` or the widget’s named constructor types (`piximind-atomic-flutter`).
- **Do NOT** put secrets, JWTs, or PII in fixtures (`piximind-security-flutter`).

## Approved patterns
- **Do** name tests after observable behavior (`loads cached items when offline`), not implementation (`calls getItems`).
- **Do** slice vertically: one UseCase or one BLoC event → state transition per red-green cycle.
- **Do** use `bloc_test` for `flutter_bloc` / `hydrated_bloc` event → state.
- **Do** depend on seams: `UseCase.call`, repository **interfaces**, BLoC events/states.
- **Do** keep domain tests free of Flutter bindings (`TestWidgetsFlutterBinding` only where UI is under test).
- **Do** colocate fixtures next to the spec; do not read production JSON parsers to build the expected entity.

## Seams (test / depend here)
- `UseCase.call` (domain; inject repository contract).
- Repository **interface** (data tests hit `*_repository_impl.dart` with mocked local/remote datasources).
- BLoC: event in → state out (`bloc_test`).
- Organism / template public constructors and callbacks (widget tests).
- Do **not** treat private feature widgets or `injector.dart` factories as seams.

## Workflow
1. Locate `lib/features/{feature}/` and any existing `test/` layout.
2. Write the first failing test at the seam that encodes the behavior (usually UseCase or BLoC).
3. Write the minimum production code to pass. Repeat per behavior.
4. Run `dart test` (or the repo’s test script) and `dart analyze` before presenting results.
5. If auth, secrets, PII, or tokens are involved, also follow `piximind-security-flutter`.

## Checklist
- [ ] Test path is `test/features/{feature}/...` (or the repo’s existing test tree).
- [ ] One behavior per test; name describes the outcome.
- [ ] Domain tests import no Flutter / Floor / `http`.
- [ ] Presentation tests mock the repository contract (or datasources), not the UseCase.
- [ ] Widget tests target organism/template public API only.
- [ ] Expected values are independent of the SUT’s internals.
- [ ] `dart test` / `dart analyze` run on the new files.
- [ ] No secrets in fixtures. Coverage at seams only (UseCase / repository contract / BLoC / organism params).

## Out of scope
- Introducing Riverpod, AutoRoute, or a repo-root `lib/{data,domain,presentation}` tree (architecture skill).
- Snapshot-only golden tests as the sole coverage.
- Testing `get_it` registration graphs unless the user asked for a DI smoke test.

## Token efficiency
- Inspect existing `*_test.dart`; do not paste `lib/` into chat.
- One concern: tests. Layers → `piximind-architecture-flutter`. Widget prefixes → `piximind-atomic-flutter`.
