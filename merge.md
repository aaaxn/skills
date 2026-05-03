---
name: merge
description: Use when a PR is approved and ready to merge — verifies CI and tests, reviews the diff one final time, merges into main, and cleans up local and remote feature branches.
---

# Merge

Verify a PR is safe, merge it, and clean up.

## Flow

1. **Verify PR status** — Check CI checks and review status on GitHub (`gh pr checks`, `gh pr view`). If checks are failing or reviews are pending, stop and report.

2. **Run tests locally** — Identify relevant tests from the changed files and run them. If anything fails, stop and report — do NOT merge.

3. **Final review** — Review the PR diff one last time (`gh pr diff`). Look for anything missed: debug code, secrets, logic errors, unintended changes.

4. **Merge** — If all clear, merge with a regular merge commit into `main`:
   ```
   gh pr merge --merge
   ```

5. **Cleanup** — After successful merge:
   ```
   git checkout main
   git pull
   git branch -d <feature-branch>
   git push origin --delete <feature-branch>
   ```

## Rules

- Never merge if CI is red or tests fail locally.
- Never `--force`, `--no-verify`, or `reset --hard`.
- Always checkout `main` and pull after merge before deleting branches.
- If anything looks wrong during final review, stop and report instead of merging.
