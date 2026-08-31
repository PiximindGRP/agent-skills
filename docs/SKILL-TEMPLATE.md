# Skill template (copy into `skills/piximind-{domain}-{tech}/SKILL.md`)

```markdown
---
name: piximind-{domain}-{tech}
description: {What it enforces}. Use when {framework + folder/task triggers, and when not}.
paths:
  - "**/src/app/**"   # discriminant globs only — never **/*.{ts,tsx} or **/*.dart alone
---

# {Title}

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

House rules (also `.cursor/rules/piximind-always.mdc` in the **client** project): never TypeScript `any` / `as any`; Flutter widget props typed (`*Params` or named constructors); no secrets in fixtures, logs, Floor, Cache Storage, or `NEXT_PUBLIC_`.

## Anti-patterns
- **Do NOT** …
- **Do NOT** …

## Approved patterns
- **Do** …
- **Do** …

## Seams (test / depend here)
- …

## Workflow
1. Locate the existing tree for this technology.
2. Match naming and layers already in the repo.
3. Run the checklist.
4. If auth, secrets, PII, or HTML are involved, also follow `piximind-security-js-ts` or `piximind-security-flutter`.

## Checklist
- [ ] …
- [ ] …

## Out of scope
- …

## Token efficiency
- Inspect the repo; do not paste folder trees into chat — use [references/tree.md](references/tree.md) when that file exists.
- One concern per skill. Tests, secrets, palette, and SEO belong in sibling skills.
```

Keep the body under 200 lines. Put long trees and examples in `references/` (one level deep):

```text
skills/piximind-{domain}-{tech}/
├── SKILL.md
└── references/          # optional
    ├── tree.md          # folder tree (architecture)
    ├── interfaces.md    # named TS contracts
    ├── ds-interfaces.md # IAtom* / IMolecule* / IOrganism*
    └── examples.md      # extra patterns
```

`paths` must discriminate stacks (e.g. `**/src/app/**` vs `**/src/admin/**` vs `**/prisma/**`). Do not set `alwaysApply` on a skill — that field is rules-only.
