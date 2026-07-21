# sanderkorf/skills

Shareable [Agent Skills](https://agentskills.io/) for Cursor, Claude Code, and other agents that support the Skills CLI.

Workflows you can install once and invoke on demand (`/ship`, `/commit`, …).

Inspired by [Building Shareable AI Agent Skills](https://helderberto.com/posts/building-shareable-ai-agent-skills).

## Author

**[Sander Korf](https://www.sanderkorf.nl/)** — Dutch freelance Full Stack AI Engineer based in Amsterdam.

13+ years building systems that cut costs, automate manual work, and make AI useful in production — not just a buzzword.

- Site: [sanderkorf.nl](https://www.sanderkorf.nl/)
- Email: [sander@sdkode.dev](mailto:sander@sdkode.dev)

## Install

All skills:

```sh
npx skills add sanderkorf/skills --all
```

Pick a few:

```sh
npx skills add sanderkorf/skills --skill ship --skill commit --skill bambuser-api --skill bambuser-live
```

Update later:

```sh
npx skills update
```

Browse more skills at [skills.sh](https://skills.sh/).

## Skills

### Workflows

| Skill | What it does |
|-------|----------------|
| [`ship`](skills/ship/SKILL.md) | Stage, commit (repo-matched message), push |
| [`commit`](skills/commit/SKILL.md) | Stage and commit only (no push) |
| [`coverage`](skills/coverage/SKILL.md) | Run test/coverage scripts when they exist |

### Domain

| Skill | What it does |
|-------|----------------|
| [`bambuser-api`](skills/bambuser-api/SKILL.md) | Bambuser Live Shopping REST API — auth, shows, products, webhooks, CMS sync |
| [`bambuser-live`](skills/bambuser-live/SKILL.md) | Bambuser Live web player — cart, hydration, wishlist, miniplayer, Player API map |

## Usage

After install, invoke in your agent:

```text
/ship
/commit
/coverage
/bambuser-api
/bambuser-live
```

## Local development

From this repo:

```sh
npx skills add ./ --all
```

## License

[MIT](LICENSE)
