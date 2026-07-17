---
name: merge
description: Merge an approved PR with adaptive verification. Use when the user wants to merge a PR, optionally with `easy` to trust prior local verification or `full` to force independent checks.
argument-hint: "[easy|full]"
---

# Merge

Merge an approved PR without repeating fresh verification work.

## Modes

- `/merge easy`: the user confirms local tests and review are already complete. Do not spawn subagents or rerun local tests and review. Keep the remote CI and mergeability gate.
- `/merge full`: always run independent tests and diff review in parallel subagents.
- `/merge` with no argument: choose the smallest verification path justified by the diff and fresh evidence.

Use an explicit mode without asking for confirmation. Reject unknown mode names and show the three valid forms.

## Flow

### 1. Resolve the target

- Detect the default branch with `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`.
- Resolve the PR with `gh pr view --json number,headRefName,headRefOid,reviewDecision,mergeable`.
- Capture the feature branch and current PR head SHA.

### 2. Run the remote gate

Run `gh pr checks` and inspect `reviewDecision` and `mergeable`. Stop if required CI is failing, the PR is not approved when approval is required, or GitHub reports it cannot merge.

This gate applies to every mode. `easy` skips duplicate local work, not branch protection or remote safety checks.

### 3. Verify according to the mode

Evidence is fresh only when it covers the current PR head SHA. Reuse fresh test or review results already present in the conversation.

**Easy**

- Do not spawn subagents.
- Do not rerun local tests or review the diff.
- Record that local verification was trusted rather than rerun.

**Adaptive**

Inspect the changed-file list and diff first, then choose:

- Documentation, metadata, or a small isolated configuration change: review inline. Run no tests unless the repository defines a mandatory lightweight check.
- A scoped behavior change with a clear test seam: run relevant tests inline and review the diff inline.
- A broad or high-risk change involving security, data loss, migrations, concurrency, public interfaces, or several independent subsystems: dispatch test and diff-review subagents in parallel.

Subagents are for genuine parallel work, not a fixed ceremony.

**Full**

Dispatch two parallel subagents:

1. A test runner that identifies and runs relevant tests.
2. A reviewer that checks the PR diff for secrets, debug code, unintended changes, logic errors, and repository-style violations.

Stop if any verification performed in the selected mode fails.

### 4. Merge

Run:

```bash
gh pr merge --merge --delete-branch
```

### 5. Clean up

After GitHub confirms the merge:

```bash
git checkout <default>
git pull
git branch -d <feature-branch>
git fetch --prune
```

Report the merged PR, verification performed, evidence reused, and checks intentionally skipped.

## Rules

- Never claim a skipped check passed.
- If the PR head changes after verification, reassess the mode against the new diff.
- Never use `--force`, `--no-verify`, or `reset --hard`.
- Delete branches only after the merge succeeds.
- Never add AI attribution to commits, PRs, or comments.
