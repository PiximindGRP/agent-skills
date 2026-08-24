---
name: piximind-architecture-prisma
description: Enforces NestJS + Prisma layout — PrismaService client, components/{domain} module/controller/service/repository, Prisma WhereInput/Include, class-validator DTOs, IDB models. Use when adding or reviewing NestJS Prisma modules, schema.prisma, repositories, services, or DTOs. If the repo uses TypeORM, DataSource, or src/entities, follow piximind-architecture-nestjs instead.
paths:
  - "**/*.ts"
  - "**/*.prisma"
  - "**/prisma/**"
  - "**/src/components/**"
---

# NestJS Prisma architecture

Inspect `src/` first. Follow its existing folders. Do not introduce a second architecture (no TypeORM, no `src/entities`, no hexagonal ports, no Nest CQRS).

**Fork:** `prisma/schema.prisma` + `PrismaService` → this skill. TypeORM / `DataSource` / `src/entities` / `src/components/{domain}` with `getRepository` → `piximind-architecture-nestjs`. Do not mix both.

For the tree, DI, schema, and include/transaction patterns, read [references/tree.md](references/tree.md).

## Anti-patterns
- **Do NOT** put `this.prisma.*`, `$transaction`, or `$queryRaw` in controllers.
- **Do NOT** add TypeORM, Mongoose, or a second Prisma client. One global `PrismaService` from `PrismaModule`.
- **Do NOT** enable Prisma `log: ['query']` (or log query args / `DATABASE_URL` / emails / tokens). Defer PII to `piximind-security-js-ts`.
- **Do NOT** concatenate SQL or use `$queryRawUnsafe`. Prefer the Prisma query API. If tagged SQL already exists in the domain, use `Prisma.sql` only.
- **Do NOT** use `any`. Public methods, DTO fields, and Prisma payloads stay explicit (`Prisma.{model}WhereInput`, `IDB{Name}`, interfaces). No `as unknown as Prisma.*` in new code.
- **Do NOT** create `src/domain`, `implements I{Domain}Service`, or TypeORM-style `PROVIDERS` tokens unless the repo already has them.
- **Do NOT** skip `prisma migrate` + `prisma generate` when `schema.prisma` changes. Do not edit generated client by hand.
- **Do NOT** N+1: no `findUnique`/`findFirst` inside `map`/`for`. No default `include: true` on wide graphs.
- **Do NOT** hardcode secrets or raw `process.env` in new services. Use `ConfigService.get<IEnv>('env')`.
- **Do NOT** invent a new JSON envelope or a second global filter/logger/database module.

## Approved patterns
- **Do** put new features in `src/components/{domain}/`: `{domain}.module.ts`, `{domain}.controller.ts`, `{domain}.service.ts`, `{domain}.repository.ts`. Extra Prisma accessors (`{domain}.{role}.repository.ts`) stay in that folder.
- **Do** keep controllers HTTP-only: parse class-validator DTO, call service, return via `LoggerServices.response` (`{ ...data, statusCode, message }`). `@ApiTags`, existing Keycloak / role guards (`@Roles`, `AdminGuard`).
- **Do** put business rules in the service. Persistence in the repository: inject `PrismaService` (`this.prisma.{model}`). Shared `where` builders go in `PrismaCommonService` or `{domain}` filter/include helpers.
- **Do** type Prisma with generated names: `Prisma.{model}WhereInput`, `Prisma.{model}Include`, `Prisma.{model}CreateInput`. Client accessors match schema models.
- **Do** wrap rows in `src/model`: alias Prisma models and extend graphs as `IDB{Name}`. Payload contracts in `src/interface` (`ICreate{Name}`, `IFind{Name}`, `ISearch{Name}`). DTOs under `src/dto/request/{domain}/` (+ shared pagination/sort).
- **Do** extract include graphs (`select` for narrow reads, nested `include` for graphs). List + `count` share the same `where`. Paginate with `skip` / `take`.
- **Do** use `$transaction(async (tx) => …)` for multi-write (pass `tx` into helpers) or `$transaction([op1, op2])` for sequential ops. Nested `create` / `connect` for relations.
- **Do** add `@@index` / `@@unique` in `schema.prisma` when filtering or joining on that field. Enums live in the schema and are re-exported from `src/enum`.
- **Do** match the domain’s DI: `defaultImport` arrays (`USER_SERVICE = [UserService, UserRepository, …]`) **or** providers declared in the module. Register in `AppModule`. `PrismaModule` is `@Global()`.

## Seams (test / depend here)
- Controller HTTP contract (path, guards, `{ ...data, statusCode, message }`).
- Service public methods (typed; no Prisma client in the signature).
- Repository public methods (`Promise<IDB{Name} | null>`, `Prisma.*Input`).
- DTOs (`class-validator` whitelist) and `src/interface` payloads.
- `PrismaService` (mock via `src/__mocks__/prisma.mock.ts`) and `ConfigService.get<IEnv>('env')`.

## Workflow
1. Confirm Prisma (`schema.prisma`, `src/prisma/`). If TypeORM, stop and use `piximind-architecture-nestjs`.
2. Copy the **domain you touch** (module + DI style + include/filter helpers). Do not migrate Prisma-in-service leaks inside an unrelated PR.
3. Run the checklist.
4. Auth, secrets, PII, HTML → `piximind-security-js-ts`. Tests → `piximind-tdd-nestjs`.

## Checklist
- [ ] Inspected `src/`; Prisma repo, not TypeORM. Copied an existing feature module.
- [ ] Controller has no Prisma. Service has business rules; new queries live in the repository.
- [ ] Public APIs explicit: no `any`. Models/DTOs/payloads use `IDB*`, `I*`, `Prisma.*`.
- [ ] DTOs under `dto/request/{domain}/` with class-validator. Envelope unchanged.
- [ ] Includes/selects extracted; list+count share `where`; no query-in-loop. Transaction used for multi-write.
- [ ] Schema change: model + enum + `@@index` if filtered; migration + `prisma generate`. Mock model added if tests need it.
- [ ] Env keys extend `IEnv`. No Prisma query logs, no raw SQL concat, no secrets in logs.
- [ ] DI matches the domain (`defaultImport` vs local providers). Module registered in `AppModule`.
- [ ] Sonar-oriented: unused imports gone; duplicated `where`/`include` extracted; cognitive complexity split into filter/include helpers; no dead catches that hide typed errors in new code.

## Out of scope
- TypeORM layout, `src/entities`, `PROVIDERS.*`, staff/guest JWT (`piximind-architecture-nestjs`).
- Rewriting existing service-level `PrismaService` usage to repositories inside a feature PR.
- Offline mode, atomic design, SEO.
- A second Prisma package (`nestjs-prisma`), Accelerate, or extra `PrismaClient`.
