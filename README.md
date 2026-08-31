<p align="center">
  <img src="docs/assets/piximind-logo.jpg" alt="Piximind — Web & Mobile Solutions" width="720" />
</p>

# Piximind Agent Skills

Install these skills from [skills.sh](https://skills.sh) so your AI agent follows Piximind conventions for Flutter, NestJS, Next.js, and React.

They work with **Cursor** (commands below). The same packages work with Claude Code, Codex, and other agents supported by the [`npx skills`](https://github.com/vercel-labs/skills) CLI — change `-a cursor` to your agent.

[![skills.sh](https://skills.sh/b/PiximindGRP/agent-skills)](https://skills.sh/PiximindGRP/agent-skills)

---

## Quick start

You need [Node.js](https://nodejs.org/). In the project where you want the skill:

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/<skill-name>/skills/<skill-name> --skill <skill-name> -a cursor --copy -y
```

Then in Cursor: **Customize → Skills** and check that the skill is listed under Agent Decides. In chat, type `/` plus the skill name (for example `/piximind-architecture-nextjs`) and press Enter.

| After install | Command |
|---------------|---------|
| See what you have | `npx skills ls -a cursor` |
| Update | `npx skills update -p` |
| Remove one skill | `npx skills remove <skill-name> -a cursor` |

`--copy` avoids Windows symlink issues. If GitHub asks you to sign in, you need access to `PiximindGRP/agent-skills`.

---

## Architecture

### piximind-architecture-flutter

Keeps Flutter apps in a feature-first layout: domain, data, presentation, BLoC, routing, and dependency injection.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-architecture-flutter/skills/piximind-architecture-flutter --skill piximind-architecture-flutter -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-architecture-flutter/skills/piximind-architecture-flutter)

### piximind-architecture-nestjs

Keeps NestJS APIs on TypeORM: HTTP controllers, services, repositories, and validated DTOs. Use the Prisma skill instead if the project uses Prisma.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-architecture-nestjs/skills/piximind-architecture-nestjs --skill piximind-architecture-nestjs -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-architecture-nestjs/skills/piximind-architecture-nestjs)

### piximind-architecture-prisma

Same NestJS layout as above, with Prisma (`schema.prisma`, repositories, transactions). Use the TypeORM skill instead if the project uses TypeORM.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-architecture-prisma/skills/piximind-architecture-prisma --skill piximind-architecture-prisma -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-architecture-prisma/skills/piximind-architecture-prisma)

### piximind-architecture-nextjs

Keeps Next.js App Router apps in server / client / shared layers, with typed API modules. Not for Vite or AdminJS (use the React skill).

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-architecture-nextjs/skills/piximind-architecture-nextjs --skill piximind-architecture-nextjs -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-architecture-nextjs/skills/piximind-architecture-nextjs)

### piximind-architecture-react

Keeps Vite React SPAs and AdminJS apps structured (pages, design system, typed API). Not for Next.js App Router.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-architecture-react/skills/piximind-architecture-react --skill piximind-architecture-react -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-architecture-react/skills/piximind-architecture-react)

---

## Tests (TDD)

### piximind-tdd-flutter

Guides test-first Flutter work at use cases, repositories, and BLoC.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-tdd-flutter/skills/piximind-tdd-flutter --skill piximind-tdd-flutter -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-tdd-flutter/skills/piximind-tdd-flutter)

### piximind-tdd-nestjs

Guides test-first NestJS work with Jest. Unit tests do not hit a real database or Keycloak.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-tdd-nestjs/skills/piximind-tdd-nestjs --skill piximind-tdd-nestjs -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-tdd-nestjs/skills/piximind-tdd-nestjs)

### piximind-tdd-nextjs

Guides test-first Next.js App Router work (pure mappers, mocked APIs, component tests).

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-tdd-nextjs/skills/piximind-tdd-nextjs --skill piximind-tdd-nextjs -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-tdd-nextjs/skills/piximind-tdd-nextjs)

### piximind-tdd-react

Guides test-first React and AdminJS UI work (public component props, colocated tests).

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-tdd-react/skills/piximind-tdd-react --skill piximind-tdd-react -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-tdd-react/skills/piximind-tdd-react)

---

## Offline

### piximind-offline-flutter

Keeps Flutter screens usable offline: cached reads, connectivity banner, no tokens in the local database.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-offline-flutter/skills/piximind-offline-flutter --skill piximind-offline-flutter -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-offline-flutter/skills/piximind-offline-flutter)

### piximind-offline-nextjs

Reuses the Next.js PWA you already have (service worker, install prompt, manifest). Does not add a second offline stack.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-offline-nextjs/skills/piximind-offline-nextjs --skill piximind-offline-nextjs -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-offline-nextjs/skills/piximind-offline-nextjs)

---

## UI structure

### piximind-atomic-flutter

Names and organizes Flutter widgets as atoms, molecules, organisms, and templates. For colors and look, also install `piximind-frontend-design`.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-atomic-flutter/skills/piximind-atomic-flutter --skill piximind-atomic-flutter -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-atomic-flutter/skills/piximind-atomic-flutter)

### piximind-atomic-web

Names and organizes Next.js and React components as atoms, molecules, and organisms. For colors and look, also install `piximind-frontend-design`.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-atomic-web/skills/piximind-atomic-web --skill piximind-atomic-web -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-atomic-web/skills/piximind-atomic-web)

### piximind-frontend-design

Helps the agent pick palette, type, layout, and copy tone (Figma first, otherwise project tokens). Pair with an atomic skill for folder names.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-frontend-design/skills/piximind-frontend-design --skill piximind-frontend-design -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-frontend-design/skills/piximind-frontend-design)

---

## SEO

### piximind-seo-web

Improves Next.js search visibility: unique titles and descriptions, sitemap, robots, and indexable content.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-seo-web/skills/piximind-seo-web --skill piximind-seo-web -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-seo-web/skills/piximind-seo-web)

### piximind-seo-flutter

Improves store listings (ASO), app flavors, and App Links / Universal Links.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-seo-flutter/skills/piximind-seo-flutter --skill piximind-seo-flutter -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-seo-flutter/skills/piximind-seo-flutter)

---

## Security

### piximind-security-js-ts

Protects Next.js, React, and NestJS work: no secrets in the client, safe env handling, no TypeScript `any` on untrusted data.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-security-js-ts/skills/piximind-security-js-ts --skill piximind-security-js-ts -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-security-js-ts/skills/piximind-security-js-ts)

### piximind-security-flutter

Protects Flutter apps: secure token storage, TLS, and careful pub.dev dependencies.

```bash
npx skills add https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-security-flutter/skills/piximind-security-flutter --skill piximind-security-flutter -a cursor --copy -y
```

[Open on GitHub](https://github.com/PiximindGRP/agent-skills/tree/skill/piximind-security-flutter/skills/piximind-security-flutter)

---

## Which skill should I install?

Install the **architecture** skill for your stack, then add extras as needed:

| Your project | Start with | Often add |
|--------------|------------|-----------|
| Flutter app | `piximind-architecture-flutter` | TDD, offline, atomic, SEO, security-flutter |
| NestJS + TypeORM | `piximind-architecture-nestjs` | TDD, security-js-ts |
| NestJS + Prisma | `piximind-architecture-prisma` | TDD, security-js-ts |
| Next.js App Router | `piximind-architecture-nextjs` | TDD, offline, atomic-web, SEO-web, security-js-ts, frontend-design |
| React Vite / AdminJS | `piximind-architecture-react` | TDD, atomic-web, security-js-ts, frontend-design |

You can install several skills in the same project. They apply when you open matching files, or when you call them with `/skill-name`.

---

## For Piximind authors

How we write and review skills: [docs/SKILL-RULES.md](docs/SKILL-RULES.md) · [docs/SKILL-TEMPLATE.md](docs/SKILL-TEMPLATE.md).
