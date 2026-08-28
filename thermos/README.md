# Thermos (Claude Code install)

Thermo-nuclear branch review: deep correctness/security audits + harsh code-quality
rubrics, run via parallel review subagents and synthesized into one verdict.

Adapted from the upstream `thermos` plugin (MIT, © 2026 Cursor — see `LICENSE`).
Cursor-specific packaging has been removed; the skills and agents below are installed
directly as personal Claude Code components.

## Components

Skills (`~/.claude/skills/`):

| Skill | Purpose |
|:------|:--------|
| `thermos` | Orchestrator: runs both review subagents in parallel, then synthesizes findings. |
| `thermo-nuclear-review` | Deep correctness + security audit of a branch diff. |
| `thermo-nuclear-code-quality-review` | Strict maintainability / structure / 1k-line / spaghetti audit. |

Subagents (`~/.claude/agents/`):

| Agent | Purpose |
|:------|:--------|
| `thermo-nuclear-review-subagent` | Diff-scoped deep-review pass. |
| `thermo-nuclear-code-quality-review-subagent` | Diff-scoped code-quality pass. |

## Usage

All three skills set `disable-model-invocation: true`, so invoke them explicitly:

- `/thermos` — gather the branch diff + changed-file contents, launch both subagents in
  parallel (`run_in_background: true`), and synthesize a deduped, prioritized verdict.
- `/thermo-nuclear-review` — run only the correctness/security rubric in the main agent.
- `/thermo-nuclear-code-quality-review` — run only the maintainability rubric.

The subagents are spawned by the orchestrator via the Agent tool with the matching
`subagent_type`; they read their rubric from the skill of the same name.
