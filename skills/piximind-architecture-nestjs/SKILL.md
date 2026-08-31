---
name: piximind-architecture-nestjs
description: Enforces NestJS TypeORM layout — src/entities, HTTP-only controllers, TypeORM repositories, class-validator DTOs. Use when editing TypeORM entities, DTOs, or components/{domain}. If schema.prisma / PrismaService exists, use piximind-architecture-prisma instead.
paths:
  - "**/src/entities/**"
  - "**/*.entity.ts"
  - "**/src/dto/**"
  - "**/src/components/**"
---

# NestJS architecture

Inspect `src/` first. Follow its existing folders. Do not introduce a second architecture (no `src/domain`, no Feature-Sliced Design, no Nest CQRS, no Prisma).

Canonical new work: `src/components/{domain}/`. If the repo uses legacy modules at `src/` root, follow **that** tree inside a feature PR — do not migrate it.

For the tree, DI bundles, and HTTP envelope, read [references/tree.md](references/tree.md).

## Anti-patterns
- **Do NOT** put `getRepository`, QueryBuilder, Prisma, or Mongoose calls in controllers.
- **Do NOT** create `src/domain`, `src/usecases`, or hexagonal ports unless the repo already has them.
- **Do NOT** add Prisma, Mongoose, or `TypeORM forFeature()` as a parallel persistence style. Use `PROVIDERS` + `DataSource.getRepository`.
- **Do NOT** hardcode secrets or raw `process.env` in new services. Use existing `ConfigService` / env interfaces (`IEnv`). Defer to `piximind-security-js-ts`.
- **Do NOT** enable `synchronize: true` or skip a migration when an entity changes.
- **Do NOT** invent a new JSON response shape. Envelope is `{ statusCode, data }` or `{ statusCode, error }`.
- **Do NOT** disable JWT / Roles / scope guards on admin routes that mutate scoped data.
- **Do NOT** mix staff JWT (`JwtAuthGuard`) with guest JWT (`JwtGuestAuthGuard`).
- **Do NOT** use `any` or `as any`. Type DTOs, entities, and `I{Domain}*` contracts. Use `unknown` + narrowing for untrusted JSON.
- **Do NOT** N+1: no `findOne` / extra query inside a loop in a service. Load relations in the repository.
- **Do NOT** dump `src/` trees into chat. Read [references/tree.md](references/tree.md).

## Approved patterns
- **Do** put new features in `src/components/{domain}/`: `{domain}.module.ts`, `{domain}.controller.ts`, `{domain}.service.ts`, `{domain}.repository.ts`.
- **Do** keep controllers HTTP-only: parse DTO, call service, return the envelope. `@ApiTags`, `@UseGuards`, `@Res()` as the domain already does.
- **Do** put business rules in the service (`implements I{Domain}Service`). Persistence only in the repository (`implements I{Domain}Repository`).
- **Do** validate input with class-validator DTOs under `src/dto/{domain}/` (shared DTOs under `dto/shared/`).
- **Do** place TypeORM entities in `src/entities/`, extend `Based` (`entities/shared/based.shared.ts`), table names from `EDataTable`.
- **Do** place contracts in `src/interfaces/{domain}/` (`I{Name}Service`, `I{Name}Repository`).
- **Do** match the domain’s DI style: `defaultModuleImport` `{DOMAIN}_REPOSITORY_MODULE` / `{DOMAIN}_SERVICE_MODULE`, or a small self-contained module with providers declared locally. Consistency inside a domain beats a global refactor.
- **Do** register the module in `AppModule` the same way existing feature modules are registered.
- **Do** keep repository methods focused; extract when cognitive complexity / duplication would fail Sonar. Empty `catch` is forbidden — log via the existing logger (no secrets) or rethrow.

## Seams (test / depend here)
- Controller HTTP contract (path, guards, envelope).
- `I{Domain}Service` public methods.
- `I{Domain}Repository` public methods.
- DTOs (`class-validator` whitelist).
- `PROVIDERS.*` tokens and `ConfigService.get<IEnv>('env')`.

## Workflow
1. Locate the existing tree for this technology (`src/components`, or legacy src-root modules).
2. Match naming and layers already in the **domain you touch**. If that domain diverges from `components/{domain}`, follow the domain.
3. Run the checklist.
4. If auth, secrets, PII, or HTML are involved, also follow `piximind-security-js-ts`. Tests: `piximind-tdd-nestjs`.

## Checklist
- [ ] Inspected `src/` and copied an existing feature module, not a textbook layout.
- [ ] Controller has no TypeORM / business rules.
- [ ] Service implements the interface; repository is the only persistence.
- [ ] DTOs live under `dto/{domain}/` and use class-validator.
- [ ] Entity extends `Based`, table via `EDataTable`; migration added if the schema changed.
- [ ] Provider token + module registration match the domain’s DI style.
- [ ] Staff vs guest guards are not mixed; scoped mutations keep guards.
- [ ] No hardcoded secrets; new env keys extend `IEnv` and are called out for DevOps.
- [ ] Response envelope unchanged.
- [ ] No `any`. DTOs/interfaces cover inputs and returns.
- [ ] No N+1 in the new service/repository path.
- [ ] Sonar-style: unused locals, empty `catch`, duplicated blocks, high complexity — fix at the service/repository seam (`piximind-tdd-nestjs` for coverage).

## Out of scope
- Offline mode (client concern). Mention idempotent writes / cache headers only if a client already depends on them.
- Atomic design or SEO (no UI).
- Rewriting a legacy src-root module layout to `src/components/{domain}` inside a feature PR.
- Inventing a second global filter, logger, or database module.

## Token efficiency
- Inspect `src/`; do not paste the module tree into chat — use [references/tree.md](references/tree.md).
- One concern: Nest layers. Secrets → `piximind-security-js-ts`. Specs → `piximind-tdd-nestjs`.
