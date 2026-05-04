---
name: merge
description: Use when a PR is approved and ready to merge — verifies CI and tests, reviews the diff one final time, merges into main, and cleans up local and remote feature branches.
---

# Merge

Verify a PR is safe, merge it, and clean up.

## Flow

1. **Verify PR + Run tests + Final review (parallel)** — Dispatch three subagents simultaneously via the Agent tool:

   **Agent A — CI status checker:** Check CI checks and review status on GitHub (`gh pr checks`, `gh pr view`). Report whether checks pass and reviews are approved.

   **Agent B — Local test runner:** Identify relevant tests from the changed files and run them. Report pass/fail results and any failure output.

   **Agent C — Diff reviewer:** Review the PR diff (`gh pr diff`). Look for anything missed: debug code, secrets, logic errors, unintended changes.

   Wait for all three agents to complete. If ANY agent reports failures or issues, stop and report — do NOT proceed to merge.

2. **Merge** — If all agents report clean, merge with a regular merge commit into `main`:
   ```
   gh pr merge --merge
   ```

3. **Cleanup** — After successful merge:
   ```
   git checkout main
   git pull
   git branch -d <feature-branch>
   git push origin --delete <feature-branch>
   ```

## Rules

- Never merge if CI is red or tests fail locally.
- Never `--force`, `--no-verify`, or `reset --hard`.
- Never add "🤖 Generated with Claude Code" to PR descriptions or commit messages.
- Always checkout `main` and pull after merge before deleting branches.
- If anything looks wrong during final review, stop and report instead of merging.
