# NestJS tree

Read this when adding a new domain under `src/components/{domain}/`.

## Stack
- NestJS 9 + TypeScript
- PostgreSQL via TypeORM 0.3 (`DataSource`, `synchronize: false`, migrations in `src/migrations`)
- Auth: Passport JWT (staff + guest), Google OAuth, refresh / reset / accept-account strategies
- Config: `@nestjs/config`, typed `IEnv` in `src/interfaces/env/IEnv.ts`, `ConfigModule.forRoot({ load: [env] })`. Read via `ConfigService.get<IEnv>('env')`.
- HTTP: class-validator + class-transformer, custom global `ValidationPipe`, Swagger at `/api`, Helmet, CORS, express-rate-limit
- Observability: nest-winston, `LoggingInterceptor`, `ExceptionsFilter` (Sentry on 5xx)
- Realtime: Socket.IO (`SocketGateway` + `MobileSocketGateway`), Redis adapter exists
- Jobs: `@nestjs/schedule` in `src/cronJob`
- Files: Multer + GCS (`FileStorage`)

## Source tree
```text
src/
  main.ts                 bootstrap: global pipe, filter, interceptor, Swagger, Helmet, rate-limit
  app.module.ts           feature modules; scope middleware on all routes
  config.ts               PROVIDERS token map
  defaultModuleImport/    {DOMAIN}_REPOSITORY_MODULE then {DOMAIN}_SERVICE_MODULE
  components/{domain}/    feature modules (canonical new work)
  entities/               TypeORM entities; EDataTable
  entities/shared/based.shared.ts   UUID id + createdAt + updatedAt
  dto/{domain}/           create, update, search, v2/v3 when versioned
  dto/shared/             scopeId.dto, image.dto, lang.dto, search.dto
  interfaces/{domain}/    I{Name}Service, I{Name}Repository
  interfaces/env/IEnv.ts
  providers/{domain}/     TypeORM repository providers
  common/                 guards, strategy, pipes, filters, interceptors, database, socket, email
  enum/                   EDataTable, roles, folders, brands
  accesses/               ACCESSES keys used by @Roles()
  cronJob/
  migrations/
```

## Feature module
```text
src/components/{domain}/
  {domain}.module.ts
  {domain}.controller.ts            HTTP only
  {domain}.guest.controller.ts      optional guest JWT
  {domain}.v2.controller.ts         optional versioned admin
  {domain}.service.ts               I{Domain}Service; no @Res()
  {domain}.repository.ts            I{Domain}Repository; TypeORM only
  {domain}-third-party.service.ts   optional outbound API
```

## HTTP contract
- Envelope: `{ statusCode, data }` or `{ statusCode, error }`. Controllers catch and map `HttpException`.
- Admin staff: `JwtAuthGuard` + often `RolesGuard` + scope guard. `@Roles([{ module, accesses }])`.
- Guest app: `JwtGuestAuthGuard` / `JwtGuestStrategy`. Paths often `/guest/...`.
- Versioned admin: `/v2/admin/{resource}`.
- Scope id: middleware copies query/body onto `req['scopeId']`. Prefer a shared `ScopeIdDto`. Match the name the repo already uses.
- OpenAPI: `@ApiTags`, `@ApiBearerAuth`, `@OpenApiResponse`.

## DI (two accepted styles)
1. Canonical bundle: `src/providers/{domain}/{domain}.provider.ts` + `defaultModuleImport` `{DOMAIN}_REPOSITORY_MODULE` / `{DOMAIN}_SERVICE_MODULE`. Feature module spreads those arrays. Import `DatabaseModule`.
2. Self-contained: providers declared in the module. Still uses `PROVIDERS.*` and `DatabaseModule`. Prefer only for small isolated features.

## When adding a domain
1. Entity `src/entities/{name}.entity.ts` extending `Based`, table via `EDataTable`.
2. Provider token on `PROVIDERS` in `config.ts` + `src/providers/{domain}/`.
3. Repository + `I{Domain}Repository`.
4. Service + `I{Domain}Service`.
5. DTOs under `dto/{domain}/`.
6. Controller with Swagger + guards + envelope.
7. Module registered in `AppModule`.
8. Bundle in `defaultModuleImport` **or** keep providers local in the module.
9. Migration. Tests: `*.spec.ts` next to the unit (see `piximind-tdd-nestjs`).

## Module shapes
- Isolated new feature: `src/components/{domain}/` with providers declared in the module.
- Typical domain + DI bundle: `src/components/{domain}/` + `providers/{domain}/` + `{DOMAIN}_*_MODULE`.
- Multi-controller + guards + v2: optional `{domain}.guest.controller.ts` and `{domain}.v2.controller.ts`.
- Auth: existing user module under `src/components/`.

## Cross-cutting (reuse)
`DatabaseModule` / `PROVIDERS.database`, `FileStorage`, `EmailService`, `CommonFunctionService`, `PushNotificationService`, `TranslateService` / `LangRepository`, `SocketGateway` / `MobileSocketGateway`. Do not add a second global filter.

Staff vs guest are different JWT strategies. SuperAdmin bypasses `RolesGuard`. Do not call outbound third-party APIs from random modules; reuse existing client services.
