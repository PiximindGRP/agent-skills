## Summary

- **Tree encoded:**
- **Seams:**
- **Anti-patterns:**

No project or customer names in this description.

## Review gate ([SKILL-RULES.md](../docs/SKILL-RULES.md) §3)

- [ ] Encodes **our** tree (Flutter feature-first, Nest `components/{domain}`, Next `server-side`/`client-side`, React DS or AdminJS) — not a generic blog layout
- [ ] `description` has triggers (framework + domain words) and discriminates sibling stacks
- [ ] `paths` are discriminant (no catch-all `**/*.{ts,tsx}` or `**/*.dart`)
- [ ] Frontmatter `name` matches the parent folder
- [ ] Anti-patterns + approved patterns + checklist
- [ ] Seams listed for that technology
- [ ] Security skill linked where data/auth/HTML is involved
- [ ] No FSD / hexagonal dump that contradicts our trees
- [ ] Examples use our names (`atom_cta`, `OrganismTable`, `{Domain}Api`, `IApiRepository`)
- [ ] `SKILL.md` under 200 lines; extras in `references/`
- [ ] Does not duplicate a catalog security skill

## Process ([SKILL-RULES.md](../docs/SKILL-RULES.md) §6)

After Wave 1: branch `skill/{skill-name}`, **one skill per PR**. Wave 1 bootstrap (full catalog) is an explicit exception.
