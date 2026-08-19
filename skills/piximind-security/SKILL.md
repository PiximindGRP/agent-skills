---
name: piximind-security
description: Enforces dynamic environment variable discovery, dependency risk analysis, and zero-trust data handling for Next.js, React, and NestJS.
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
- Strip real credentials, JWT secrets, and connection strings from generated code, tests, and mock fixtures.
- Prevent prototype pollution and injection attacks by enforcing validation via `zod` or `class-validator`.