# NestJS Prisma tree

Read this when adding a domain under `src/components/{domain}/` in a **Prisma** Nest app.

## Stack
- NestJS + TypeScript. PostgreSQL via Prisma (`prisma/schema.prisma`, `provider = "postgresql"`, `url = env("DATABASE_URL")`).
- Client: `PrismaService extends PrismaClient`, `$connect` in `onModuleInit`. `@Global()` `PrismaModule` exports `PrismaService` + `PrismaCommonService`.
- Config: `@nestjs/config`, typed `IEnv` in `src/common/env/env.ts`, `ConfigModule.forRoot({ load: [env] })`. Read via `ConfigService.get<IEnv>('env')`.
- HTTP: class-validator + class-transformer, custom `ValidationPipe`, Swagger, Helmet, CORS, express-rate-limit.
- Auth: Keycloak (`KCAuthGuard`, `KCRoleGuard`, domain guards). Not staff/guest Passport JWT.
- Envelope: `LoggerServices.response` → `{ ...data, statusCode, message }`.
- Jobs: `@nestjs/schedule` under `src/cronJob` (may inject `PrismaService`; prefer calling a repository for new writes).
- Tests: colocated `*.spec.ts`. Mock `PrismaService` with `src/__mocks__/prisma.mock.ts` (add the model name to `PRISMA_MODELS`).

## Source tree
```text
prisma/
  schema.prisma              models (plural), enums, @@unique / @@index
  migrations/                prisma migrate
src/
  main.ts                    global pipe, filter, interceptor, Swagger, Helmet, rate-limit
  app.module.ts              PrismaModule + feature modules
  prisma/
    prisma.module.ts         @Global(); exports PrismaService, PrismaCommonService
    prisma.service.ts        extends PrismaClient; onModuleInit → $connect
    prisma.common.service.ts shared Prisma.{model}WhereInput (and similar) builders
  components/{domain}/       feature modules (canonical)
  dto/request/{domain}/      create / update / list DTOs
  dto/request/shared/        pagination, sort, id, email
  dto/dto.index.ts
  interface/                 ICreate{Name}, IFind{Name}, ISearch{Name} (payloads, not IService)
  model/                     Prisma aliases + IDB{Name} relation graphs
  enum/                      re-exports schema enums from @prisma/client + app enums
  defaultImport/             {DOMAIN}_SERVICE = [Service, Repository, …]
  common/                    env (IEnv), guards, logger, pipes, filters, interceptors, auth, storage
  cronJob/
  __mocks__/prisma.mock.ts
```

## Feature module
```text
src/components/{domain}/
  {domain}.module.ts
  {domain}.controller.ts              HTTP only; optional {domain}.admin.controller.ts
  {domain}.service.ts                 business rules; inject repositories
  {domain}.repository.ts              PrismaService only
  {domain}.{role}.repository.ts       optional extra accessor (stats, protocol, …)
  actions/{domain}.include.service.ts optional Prisma.{model}Include graphs
  actions/{domain}.filter.service.ts  optional Prisma.{model}WhereInput
  {domain}.constants.ts               optional field maps (`as const satisfies keyof model`)
```

## Prisma client
```ts
@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit(): Promise<void> {
    await this.$connect();
  }
}
```
Do not construct another `PrismaClient`. Do not pass `log: ['query']`. Inject `PrismaService` into repositories (and into `PrismaCommonService` consumers).

## Schema conventions
- Model names = client accessors (`this.prisma.{model}`). Match `schema.prisma`.
- `id String @id @default(uuid())`, `createdAt` / `updatedAt`.
- Relations: `onDelete: Cascade` where the schema already does. Writes use nested `create` / `connect`.
- Enums in `schema.prisma`; import from `@prisma/client` or `src/enum`.
- `@@unique([...])` and `@@index([fk])` when lists filter or join on that column.
- After edits: `prisma migrate dev` (or `deploy`) then `prisma generate`.

## Include / select / lists
- Wide graphs: extract `Prisma.{model}Include` (see `{domain}.include.service.ts` or `generateGetAllIncludes()`).
- Narrow reads: `select: { id: true, email: true }`. Channel payloads: `Pick<model, …>` + `as const` select maps.
- Lists: same `where` for `findMany` and `count`. Pagination: `skip: (page - 1) * limit`, `take: limit`.
- Filters: `Prisma.{model}WhereInput` with `AND` / `OR` / `some` / `contains` + `mode: 'insensitive'`. Reuse `PrismaCommonService` for cross-domain member filters.
- Never query inside a loop. Batch with `id: { in: ids }`.

## Transactions
```ts
await this.prisma.$transaction(async (tx) => {
  await tx.files.create({ data });
  await tx.companies.update({ where: { id }, data });
});

await this.prisma.$transaction([
  this.prisma.{model}.updateMany({ … }),
  this.prisma.{model}.create({ … }),
]);
```
Pass `tx` into private helpers. Do not interleave non-tx writes that must roll back together.

## Types (strict)
```ts
import { Prisma, users as User } from '@prisma/client';

interface IDBUser extends User {
  company?: IDBCompany | null;
}

async findUser(data: IFindUser): Promise<IDBUser | null>
async create(data: Prisma.alertsCreateInput): Promise<alerts>
```
- No `any`. No `as unknown as Prisma.*`.
- Services are classes (not `implements I{Domain}Service`).
- Contracts: `src/interface` payloads + repository/service method signatures.

## DI (two accepted styles)
1. Bundle: `src/defaultImport` `{DOMAIN}_SERVICE = [{Domain}Service, {Domain}Repository, …]`. Feature module spreads the array; export the service.
2. Self-contained: providers declared on `{domain}.module.ts` (small isolated features). Still uses global `PrismaModule`.

Register the module in `AppModule` the same way existing feature modules are registered.

## HTTP
- Controller: DTO → service → `LoggerServices.response({ response, request, query, statusCode, data, message, loggerMessage })`.
- Do not log request bodies, tokens, or Prisma rows with PII. Logger payload is `{ userId, userRole, timestamp, action }`.
- Guards already on the domain stay on. Public routes use the existing public decorator.

## When adding a domain
1. Models + enums in `schema.prisma` (index FKs you filter on). Migration + generate.
2. `IDB{Name}` in `src/model` if the row is returned with relations. Payloads in `src/interface`.
3. DTOs under `dto/request/{domain}/`.
4. Repository (`PrismaService`) + service + HTTP controller + module.
5. `defaultImport` bundle **or** local providers. Register in `AppModule`.
6. If tests touch Prisma: add the model to `PRISMA_MODELS`. Specs next to the unit (`piximind-tdd-nestjs`).

## Existing leaks (do not migrate in an unrelated PR)
Some services (request, auth, email, seeder, cron, third-party integrations) inject `PrismaService` today. New queries for a domain you own go in that domain’s repository. Do not move foreign-domain Prisma calls as a drive-by.

## Cross-cutting (reuse)
`PrismaCommonService`, `ConfigService` / `IEnv`, `LoggerServices`, `DateService`, `CommonService`, existing email / storage / socket / Keycloak modules. Do not add a second global filter or a second DB module.
