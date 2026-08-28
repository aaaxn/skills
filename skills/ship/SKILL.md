---
name: ship
description: Package changes into a branch, commit, push, and PR with adaptive verification. Use `easy` to trust prior verification or `full` to force independent checks.
argument-hint: "[easy|full]"
---

# Ship

Package completed work without repeating fresh verification.

## Modes

- `/ship easy`: the user confirms tests and review are already complete. Do not spawn subagents or rerun them.
- `/ship full`: always run independent tests and diff review in parallel subagents.
- `/ship` with no argument: choose the smallest verification path justified by the diff and fresh evidence.

Use an explicit mode without asking for confirmation. Reject unknown mode names and show the three valid forms.

## Flow

### 1. Inspect the work

- Confirm there are changes with `git status --porcelain`.
- Detect the default branch instead of assuming `main`.
- Inspect `git diff --stat`, `git diff --name-only`, and the current diff.
- Exclude credentials, `.env` files, generated artifacts, and unrelated changes from the commit.

Stop if there is nothing to ship or the intended commit boundary is unclear.

### 2. Verify according to the mode

Reuse test or review evidence from the current conversation only when the working diff has not changed since that evidence was produced.

**Easy**

- Do not spawn subagents.
- Do not rerun tests or review.
- Record that verification was trusted rather than rerun.

**Adaptive**

Choose after inspecting the diff:

- Documentation, metadata, or a small isolated configuration change: review inline and run only mandatory lightweight repository checks.
- A scoped behavior change with a clear test seam: run relevant tests inline and review the diff inline.
- A broad or high-risk change involving security, data loss, migrations, concurrency, public interfaces, or several independent subsystems: dispatch test and diff-review subagents in parallel.

Subagents are for genuine parallel work, not a fixed ceremony.

**Full**

Dispatch two parallel subagents:

1. A test runner that identifies and runs relevant tests.
2. A reviewer that checks the working diff for secrets, debug code, unintended changes, logic errors, and repository-style violations.

Stop if any verification performed in the selected mode fails.

### 3. Package the change

1. Reuse the current feature branch, or create a short Conventional Commits-style branch when on the default branch.
2. Stage explicit paths only.
3. Create one or more atomic Conventional Commits.
4. Push with upstream tracking.
5. Open a PR with a clear title, summary, and accurate verification section.

When `easy` reused prior verification, say so in the PR instead of claiming tests were rerun.

### 4. Report

Report the branch, commits, PR URL, verification performed, evidence reused, and checks intentionally skipped.

## Rules

- Never claim a skipped check passed.
- If the diff changes after verification, reassess the mode against the new diff.
- Never use `--force`, `--no-verify`, or `reset --hard`.
- Never commit credentials, `.env` files, or secrets.
- Never add AI attribution to commits, PRs, or comments.
