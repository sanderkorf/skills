---
name: ship
description: Stage changes, commit with a repo-matched message, and push. Use when shipping, saying /ship, or asking to commit and push.
---

# Ship Changes

## Workflow

1. Run `git status` (never `-uall`)
2. Review all staged and unstaged changes (`git diff` and `git diff --staged`)
3. Generate a commit message matching recent repo style from `git log`
4. Stage relevant files by name (prefer explicit paths over `git add -A`)
5. Commit with HEREDOC format
6. Push to the current branch
7. Verify with `git status`

## Rules

- Match repo commit style from `git log`
- NEVER force push
- NEVER skip hooks (`--no-verify`, `--no-gpg-sign`)
- Push to the current branch only
- Do not commit secrets (`.env`, credentials, keys)
- If pre-commit hook fails: fix issues, re-stage, create a NEW commit (never `--amend` unless explicitly requested and safe)
