---
name: coverage
description: Run available test and coverage scripts for the current changes. Use when checking coverage, saying /coverage, or asking whether tests cover recent work.
---

# Coverage Check

## Workflow

1. Detect the project package manager and scripts:
   - If `package.json` exists, inspect `scripts` for `test`, `coverage`, `test:coverage`, or similar
   - If no `package.json`, look for other common test entry points (e.g. `pytest`, `go test`, `cargo test`) only when clearly present
2. Run available checks in parallel when safe:
   - Prefer an explicit coverage script when present
   - Otherwise run the project's test script
3. Report pass/fail and any coverage summary from the tool output
4. If no test or coverage scripts exist, skip and say so clearly — do not invent a test runner

## Rules

- Never invent scripts that are not defined in the project
- Prefer project-configured commands over ad-hoc one-liners
- Keep the run scoped to what the repo already supports
- On failure, surface the failing command and relevant output; do not hide errors
