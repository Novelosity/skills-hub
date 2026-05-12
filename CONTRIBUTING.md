# Contributing to Skills Hub

First off — thanks for being here. Every skill you add makes this more powerful for everyone.

---

## What You Can Contribute

- **New skills** — Any AI skill that fits Claude Code, Windsurf, Cursor, or other assistants
- **Skill improvements** — Better prompts, more data, new routing logic
- **Bug fixes** — Broken references, outdated instructions, wrong routing
- **Stack additions** — New tech stacks for `ui-ux-pro-max`
- **Data updates** — Fresh palettes, font pairings, ad templates, market data

---

## Skill File Format

Every skill must have a `SKILL.md` at its root with this frontmatter:

```markdown
---
name: your-skill-name
description: >
  One paragraph describing exactly when to trigger this skill.
  Be specific — the AI uses this to decide whether to activate it.
---

# Skill Title

...your skill content...
```

Skills can be:
- A single `.md` file (simple skills)
- A `.skill` file (zipped bundle)
- A folder with `SKILL.md` + `/data` + `/scripts` (full skills)

---

## Adding a New Skill

1. Fork the repo
2. Create your skill in a new folder or file:
   ```
   skills-hub/
   └── your-skill-name/
       ├── SKILL.md         ← required
       ├── data/            ← optional CSV/JSON data
       └── scripts/         ← optional helper scripts
   ```
3. Add your skill to the manifest table in `SKILL.md` (the master concierge) and `README.md`
4. Test it — make sure the routing logic in `SKILL.md` Phase 1 covers your skill's trigger keywords
5. Open a pull request

---

## Pull Request Guidelines

- **One skill per PR** — keeps reviews focused
- **PR title format:** `feat: add [skill-name]` or `fix: [skill-name] - description`
- Include a short description of what the skill does and when it triggers
- If updating an existing skill, explain what changed and why

---

## Skill Quality Bar

A good skill should:

- Have a clear, specific trigger (when should the AI activate it?)
- Follow a phased structure — collect info first, confirm before acting, deliver output
- Never ask more than 3 questions at once
- End with next steps (never leave the user with a dead end)
- Work across at least 2 AI assistants

---

## Reporting Issues

Open an issue if:
- A skill gives wrong or outdated output
- Routing in the concierge sends you to the wrong skill
- A skill file is broken or missing data

Use this format:
```
Skill: [skill-name]
Issue: [what went wrong]
Expected: [what should have happened]
```

---

## Code of Conduct

Be constructive. Critique skills, not people. Everyone's building here.

---

Questions? Open an issue or start a discussion on GitHub.
