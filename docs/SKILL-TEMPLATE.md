# Skill template (copy into `skills/piximind-{domain}-{tech}/SKILL.md`)

```markdown
---
name: piximind-{domain}-{tech}
description: {What it enforces}. Use when {framework + task triggers}.
paths:
  - "**/*.{ts,tsx}"   # or **/*.dart — scope to matching files; do not set alwaysApply (rules only)
---

# {Title}

Inspect the current repository first. Follow its existing folders. Do not introduce a second architecture.

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
```

Keep the body under 200 lines. Put long examples in `references/examples.md`.
