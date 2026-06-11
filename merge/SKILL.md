---
name: merge
description: Use when a PR is approved and ready to merge — verifies CI and tests, reviews the diff one final time, merges into the default branch, and cleans up local and remote feature branches.
---

# Merge

Verify a PR is safe, merge it, and clean up. Be frugal with subagents: the CI
check runs inline, and the two subagents below are the only ones this skill
spawns — do not add more.

## Preconditions

- Detect the default branch (don't assume `main`):
  `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`. Use it everywhere `<default>` appears below.
- Confirm which PR is in play (`gh pr view --json number,headRefName`). Capture the feature branch name for cleanup.

## Flow

1. **Gate on CI (inline, no subagent)** — Run `gh pr checks` and
   `gh pr view --json reviewDecision,mergeable` yourself. If any check is
   failing or the PR isn't approved/mergeable, **stop and report — do NOT
   merge and do NOT spawn the verification agents.**

2. **Verify (two parallel subagents)** — Only if CI is green, dispatch both
   in a single message via the Agent tool:

   **Agent A — Local test runner** (`model: haiku`): Identify relevant tests
   from the changed files (`gh pr diff --name-only`) and run them. Instruct it
   to reply with only a verdict line (`PASS` or `FAIL`), and on failure the
   failing test names plus the key error lines (≤20 lines) — never the full
   test log.

   **Agent B — Diff reviewer** (`model: opus`): Review the PR diff
   (`gh pr diff`). Look for anything missed: debug code, secrets, logic
   errors, unintended changes. Instruct it to reply with only `CLEAN` or a
   short list of concrete issues with file:line — no diff quoting beyond the
   offending lines.

   Wait for both. **If either reports a failure or issue, stop and report —
   do NOT merge.**

3. **Merge** — If all clean, merge into `<default>`:
   ```
   gh pr merge --merge --delete-branch
   ```
   (`--delete-branch` removes the remote branch automatically.)

4. **Cleanup** — After a successful merge:
   ```
   git checkout <default>
   git pull
   git branch -d <feature-branch>
   git fetch --prune
   ```

## Rules

- Never merge if CI is red or tests fail locally.
- Never `--force`, `--no-verify`, or `reset --hard`.
- Never add "🤖 Generated with Claude Code" to PR descriptions or commit messages.
- Always checkout `<default>` and pull after merge before deleting the local branch.
- If anything looks wrong during final review, stop and report instead of merging.
