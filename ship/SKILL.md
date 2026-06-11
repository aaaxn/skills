---
name: ship
description: Use when changes are ready to be shipped — verifies the diff and tests first, then creates a branch, commits, pushes, and opens a PR. Reports readiness before and after.
---

# Ship

After changes are made, verify quality first, then package them into a PR. Verification happens **before** push/PR so broken code never reaches the remote.

## Preconditions

- Confirm there are actual changes: `git status --porcelain`. If clean, stop — nothing to ship.
- Detect the default branch (don't assume `main`):
  `gh repo view --json defaultBranchRef -q .defaultBranchRef.name` (fallback: `git symbolic-ref refs/remotes/origin/HEAD --short | sed 's@^origin/@@'`).

## Flow

1. **Verify (two parallel subagents)** — Before touching git, dispatch both in a single message via the Agent tool. These are the only subagents this skill spawns — do not add more.

   **Agent A — Test runner** (`model: haiku`): Identify which tests are relevant based on the changed files (look at the test directory structure, imports, and naming conventions). Run them. Instruct it to reply with only a verdict line (`PASS` or `FAIL`), and on failure the failing test names plus the key error lines (≤20 lines) — never the full test log.

   **Agent B — Diff reviewer** (`model: opus`): Review the working diff (`git diff HEAD`). Check for:
   - Leftover debug code (print statements, console.log, stray TODOs)
   - Unintended changes or files that shouldn't be committed
   - Security issues (hardcoded secrets, credentials)
   - Obvious logic errors
   - Style inconsistencies with surrounding code

   Instruct it to reply with only `CLEAN` or a short list of concrete issues with file:line — no diff quoting beyond the offending lines.

   Wait for both. **If either finds a blocking issue, stop and report — do not branch, commit, or push.**

2. **Branch** — If already on a non-default feature branch, reuse it. Otherwise auto-generate a branch name from the changes using a Conventional Commits prefix (`feat/`, `fix/`, `refactor/`, etc.) followed by a short kebab-case description. Create and checkout.

3. **Commit** — Stage changed files with explicit paths. Write a Conventional Commits message. Prefer atomic commits: `git commit -m "feat: ..." -- path/to/file1 path/to/file2`.

4. **Push** — Push with `-u` to set upstream tracking.

5. **Open PR** — `gh pr create` with a clear title and a body summarizing what changed and why. Include the test results from Agent A in the PR body.

6. **Report** — State the PR URL and confirm tests passed and the self-review was clean.

## Rules

- Verify before you push. Never open a PR with failing tests or a self-review that found issues.
- Never `--force`, `--no-verify`, or `reset --hard`.
- Never commit `.env`, credentials, or secrets.
- Never add "🤖 Generated with Claude Code" to PR descriptions or commit messages.
