---
name: piximind-offline-flutter
description: Enforces offline-first Flutter reads — NetworkBloc, Floor HTTP cache via IApiRepository.supportOffline, local+remote datasources, HydratedBloc session, connectivity banner. Use when adding Flutter network/cache/offline behavior, Floor datasources, or connectivity UX.
paths:
  - "**/*floor*"
  - "**/*network*"
  - "**/*offline*"
  - "**/*connectivity*"
  - "**/lib/**/database/**"
---

# Flutter offline-first

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

Reuse `NetworkBloc`, Floor, `IApiRepository`, and `template_scaffold`. Do not add a second local database.

## Anti-patterns
- **Do NOT** invent a second DB (Hive, Isar, another Floor instance) when Floor already caches HTTP.
- **Do NOT** store tokens, passwords, or PII in Floor. Use `flutter_secure_storage` (`piximind-security-flutter`).
- **Do NOT** disable TLS / `HttpOverrides.global` to “make offline work”.
- **Do NOT** skip the connectivity banner on `template_scaffold` for a new full-screen flow that still hits the network.
- **Do NOT** assume every mutation should queue. Queue vs fail-fast is **per feature** — follow the feature you are in; do not invent a global outbox.
- **Do NOT** bypass `IApiRepository` / `supportOffline` with a raw `http` client in a feature that already uses the cache.
- **Do NOT** refetch on every rebuild when Floor already has a fresh cache. Do not log request bodies while debugging offline (`piximind-security-flutter`).

## Approved patterns
- **Do** drive connectivity from `NetworkBloc` + `connectivity_plus`.
- **Do** put local + remote datasources behind the repository implementation. Reads degrade to Floor cache when offline when `supportOffline` is set.
- **Do** use `HydratedBloc` only for session-ish state the app already persists that way — not as a substitute for Floor API cache.
- **Do** load images through `cached_network_image` (or the repo’s existing image cache).
- **Do** show degraded UI when the cache is empty and the device is offline (empty/error on the template, not a hang).
- **Do** keep domain UseCases unaware of connectivity widgets; they talk to the repository contract.

## Seams (test / depend here)
- `IApiRepository` + `supportOffline` (read path).
- Repository that orchestrates local vs remote datasource.
- `NetworkBloc` events/states.
- `template_scaffold` connectivity banner (public template API).
- Do not treat Floor DAO internals as the feature seam.

## Workflow
1. Locate `NetworkBloc`, Floor, `IApiRepository`, and how the nearest feature caches reads.
2. Match that feature’s `supportOffline` and mutation policy (queue vs fail-fast).
3. Run the checklist.
4. If tokens, PII, or TLS are involved, also follow `piximind-security-flutter`. Tests: `piximind-tdd-flutter`.

## Checklist
- [ ] Reads degrade to Floor cache when offline; no second DB.
- [ ] `supportOffline` / `IApiRepository` used where the feature already caches HTTP.
- [ ] Tokens stay in `flutter_secure_storage`, not Floor.
- [ ] TLS left intact.
- [ ] Connectivity banner on the template scaffold for networked screens.
- [ ] Mutation policy copied from the feature (not a new global queue).
- [ ] Empty-cache + offline has an explicit degraded state.

## Out of scope
- Server-side offline mode.
- Flutter web PWA / service workers (native app is the default; web only if the project already ships it).
- Replacing `go_router` or BLoC to support offline.

## Token efficiency
- Inspect `NetworkBloc` / `IApiRepository` / the nearest feature’s cache flag; do not paste Floor schemas into chat.
- One concern: offline reads + connectivity UX. Storage secrets → `piximind-security-flutter`. Tests → `piximind-tdd-flutter`.
