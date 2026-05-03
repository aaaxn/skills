---
name: ship
description: Use when changes are ready to be shipped — creates a branch, commits, pushes, opens a PR, runs relevant tests, and self-reviews the diff before reporting readiness.
---

# Ship

After changes are made, package them into a PR and verify quality.

## Flow

1. **Branch** — Auto-generate a branch name from the changes using Conventional Commits prefix (`feat/`, `fix/`, `refactor/`, etc.) followed by a short kebab-case description. Create and checkout the branch.

2. **Commit** — Stage changed files with explicit paths. Write a Conventional Commits message. Use atomic commits: `git commit -m "feat: ..." -- path/to/file1 path/to/file2`.

3. **Push** — Push to remote with `-u` to set upstream tracking.

4. **Open PR** — Use `gh pr create`. Write a clear title and body summarizing what changed and why.

5. **Run tests** — Identify which tests are relevant based on the changed files (look at the test directory structure, imports, and naming conventions). Run them. If tests fail, report the failures — do NOT merge or claim success.

6. **Self-review** — Review the full PR diff (`gh pr diff`). Check for:
   - Leftover debug code (print statements, console.log, TODO comments)
   - Unintended changes or files that shouldn't be committed
   - Security issues (hardcoded secrets, credentials)
   - Obvious logic errors
   - Style inconsistencies with surrounding code

7. **Report** — Summarize findings. State clearly whether the PR is good to go or what needs fixing.

## Rules

- Never `--force`, `--no-verify`, or `reset --hard`.
- Never commit `.env`, credentials, or secrets.
- If tests fail or review finds issues, stop and report — do not push forward.
