---
name: commit
description: Stage changes and create a commit with a repo-matched message. Use when committing, saying /commit, or asking for a commit without pushing.
---

# Commit Changes

## Workflow

1. Run `git status` (never `-uall`)
2. Review all staged and unstaged changes (`git diff` and `git diff --staged`)
3. Analyze recent commit style from `git log`
4. Stage relevant files by name (prefer explicit paths over `git add -A`)
5. Commit with HEREDOC format
6. Verify with `git status`

## Rules

- Match repo commit style from `git log`
- Do NOT push
- NEVER skip hooks (`--no-verify`, `--no-gpg-sign`)
- Do not commit secrets (`.env`, credentials, keys)
- If pre-commit hook fails: fix issues, re-stage, create a NEW commit (never `--amend` unless explicitly requested and safe)
- If there is nothing to commit, say so and stop — do not create an empty commit
