---
name: piximind-tdd-nestjs
description: Enforces Jest vertical-slice TDD for NestJS — colocated *.spec.ts, mocked repositories, no real Keycloak or DB in unit tests. Use when adding NestJS features, writing service/controller specs, or reviewing e2e tests.
paths:
  - "**/*.spec.ts"
  - "**/test/**"
---

# NestJS TDD

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

## Anti-patterns
- **Do NOT** write the full feature then a pile of specs. One behavior per red-green slice.
- **Do NOT** hit a real database, Keycloak, Redis, GCS, or outbound third-party APIs in unit tests.
- **Do NOT** mock the class under test. Mock repositories, `PROVIDERS.*`, and outbound services.
- **Do NOT** put unit specs only in a remote `__tests__` folder when the repo colocates `*.spec.ts` next to the unit.
- **Do NOT** skip auth guards on protected-route specs. Assert guard presence / forbidden paths.
- **Do NOT** embed live JWTs, connection strings, or PII in fixtures (`piximind-security-js-ts`).
- **Do NOT** type fixtures or mocks as `any`. Typed doubles of `I{Domain}Repository` / DTO classes only.
- **Do NOT** swallow errors in specs with empty `catch`. Unused locals in tests are still smells.

## Approved patterns
- **Do** colocate `*.spec.ts` next to the service or controller under test (Jest).
- **Do** keep HTTP e2e in `test/` (or the repo’s e2e folder). E2E asserts the HTTP contract and envelope, not TypeORM internals.
- **Do** reuse `src/test/*-mock.ts` or the repo’s existing mock helpers.
- **Do** test `I{Domain}Service` public methods as the unit seam; inject a mocked `I{Domain}Repository`.
- **Do** cover staff vs guest guards separately when both controllers exist.
- **Do** name cases after behavior (`rejects create when scope id missing`), not method names alone.

## Seams (test / depend here)
- Service public methods (`I{Domain}Service`).
- HTTP contract for e2e: path, status, `{ statusCode, data | error }` envelope, guard rejection.
- DTO validation (whitelist / 400) when the pipe is in play.
- Repository **interface** only in repository specs (mocked `Repository<Entity>` / QueryBuilder).
- Do not treat private helpers or `defaultModuleImport` arrays as the primary seam.

## Workflow
1. Locate an existing `*.spec.ts` and the `src/test/` mock pattern.
2. Write one failing spec at the service (or HTTP e2e if the change is the contract).
3. Implement the minimum production code. Repeat.
4. Run the repo’s Jest command (`npm test` / `npx jest path/to/file.spec.ts`).
5. If auth, secrets, or PII are involved, also follow `piximind-security-js-ts`. Structure: `piximind-architecture-nestjs`.

## Checklist
- [ ] `*.spec.ts` sits next to the unit; e2e stays in `test/`.
- [ ] Repositories / PROVIDERS mocked; no real Keycloak or DB.
- [ ] Protected routes assert guards / 401 / 403.
- [ ] Fixtures contain no live credentials.
- [ ] No `any` in specs or mocks; expected values are independent typed fixtures.
- [ ] One behavior per case; independent expected values.
- [ ] Jest run for the new specs is green.
- [ ] New production code covered at the seam (service/HTTP), not private helpers.

## Out of scope
- Server-side “offline mode”.
- Rewriting a legacy src-root module layout to `src/components/{domain}` as part of adding a test.
- Snapshotting entire Swagger documents as the only contract test.

## Token efficiency
- Inspect an existing `*.spec.ts`; do not paste the module tree into chat.
- One concern: tests. Structure → `piximind-architecture-nestjs`. Secrets → `piximind-security-js-ts`.
