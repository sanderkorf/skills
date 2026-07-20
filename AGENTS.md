# AGENTS.md

Guidance for AI coding agents working in this repository.

## Repository overview

`sanderkorf/skills` is a collection of shareable Agent Skills. Skills are Markdown workflows installed via `npx skills add` and invoked on demand (e.g. `/ship`).

## Structure

```
skills/
  {skill-name}/           # kebab-case — general workflows (ship, commit, …)
    SKILL.md              # required
    references/           # optional supporting docs
    scripts/              # optional helper scripts

{domain-skill}/           # kebab-case at repo root — domain/API skills (e.g. bambuser-api)
  SKILL.md
  reference/              # optional
```

## Adding a skill

1. Create the skill directory (`skills/<kebab-name>/` for workflows, or `<kebab-name>/` at repo root for domain skills).
2. Add `SKILL.md` with YAML frontmatter (`name`, `description`).
3. Write clear Workflow and Rules sections the agent can follow step by step.
4. **Keep [`README.md`](README.md) up to date** (required — see below).

### Keep the README in sync

Whenever you add, rename, remove, or change the purpose of a skill, update `README.md` in the same change:

- Skills tables (Workflows vs Domain) — correct path and one-line description
- Install example (`--skill …`) if you want the new skill discoverable in the sample command
- Usage block (`/skill-name`) when the skill is slash-invokable
- Do not leave the README listing stale or missing skills

Keep the README scannable: short sections, tables over prose. Preserve the Author section (link to [sanderkorf.nl](https://www.sanderkorf.nl/), contact) unless the owner asks to change it.

### SKILL.md format

```markdown
---
name: example-skill
description: What it does and when to use it. Include trigger phrases.
---

# Example Skill

## Workflow

1. First step
2. Second step

## Rules

- Hard constraints the agent must not violate
```

### Naming

- Directory: `kebab-case` (e.g. `create-pull-request`)
- File: always `SKILL.md`
- Frontmatter `name`: match the directory name

### Description

Write in third person. Include both **what** the skill does and **when** to use it so agents can route correctly.

## Out of scope for this repo

- Always-on project rules belong in a consuming project's `AGENTS.md` / `CLAUDE.md`, not here.
- Skills load only when invoked — keep them focused workflows, not general style guides.
