---
name: piximind-security-js-ts
description: Enforces dynamic environment variable discovery, dependency risk analysis, zero-trust data handling, and a ban on TypeScript any for Next.js, React, and NestJS. Use when env, deps, untrusted payloads, secrets in logs/tests, or JWT typing are involved.
paths:
  - "**/*.{ts,tsx,js,jsx}"
---

# JS/TS Security, Dependency & Config Protocols

You are operating as an enterprise AI assistant. You MUST strictly adhere to these security and ethical guidelines on every action, generation, or refactor in a JavaScript/TypeScript environment.

## 1. Dynamic Environment Variable Discovery (Never Hardcode)
- **Do not assume `process.env`:** Before writing or reading environment variables, inspect the codebase to identify the established pattern:
  - **NestJS:** Check for `ConfigService` (`@nestjs/config`) and inject it properly.
  - **Next.js / React:** Check for schema validators like `t3-oss/env-nextjs`, `zod` schemas (`env.mjs`), or dedicated config modules.
  - **Client-Side Safety:** Ensure public variables match the framework's prefix conventions (e.g., `NEXT_PUBLIC_`) and never expose server secrets to client components.

## 2. Package Management & Compatibility
- **Engine Verification:** Check `package.json` for `engines` (Node version) and lockfiles to determine the active package manager.
- **Compatible Versions:** Ensure packages are compatible with current major framework versions (e.g., Next.js App Router).

## 3. Dependency Risk & Vulnerability Analysis
- **Pre-Install Risk Assessment:** Before adding a dependency, evaluate known CVEs, maintenance health, and bundle size impact.
- **Lockfile Integrity:** Never suggest commands with `--force` or `--legacy-peer-deps` unless explicitly instructed.

## 4. Zero-Trust Data Handling
- Strip real credentials, JWT secrets, and connection strings from generated code, tests, mock fixtures, and logs.
- Prevent prototype pollution and injection attacks by enforcing validation via `zod` or `class-validator`.
- Never log tokens, cookies, or PII. Empty `catch` that hides auth/validation failures is a vulnerability.

## 5. Strict typing (compose with architecture / TDD skills)
- Never `any` or `as any`. Use a named interface, generics, or `unknown` + narrowing.
- Untrusted HTTP bodies: validate to a DTO/Zod schema, then use that type — do not cast.
- Decode JWTs with a named interface (`jwtDecode<IDecodedToken>(token)`). Never `any` on token payloads.
- Do not put secrets, refresh tokens, raw env, or Bearer strings on types imported by client DS / atoms.

## 6. Sonar-style (compact — not a full quality profile)
- Fix **bugs** and **vulnerabilities** before smells.
- Smells: high cognitive complexity, duplicated logic, unused locals, empty `catch`.
- Coverage at public seams (guards, validators, env accessors) — not private helpers.

## 7. Token efficiency
- Discover the existing env/validator module first. Do not paste `.env` samples or `src/` trees into chat.
- One concern: security. Defer folder layout to architecture skills.