# skills

My personal Claude Code skills. Skills sourced from other repos
(e.g. [mattpocock/skills](https://github.com/mattpocock/skills)) are not
vendored here — they're installed directly from their official repos into
`~/.agents/skills` and tracked by `.skill-lock.json`.

## Skills

| Skill | Purpose |
| --- | --- |
| `merge` | Verify CI/tests/diff, merge an approved PR, clean up branches |
| `ship` | Verify diff and tests, then branch, commit, push, and open a PR |
| `deslop` | Remove AI-generated noise from the branch diff |
| `ruff` | Reference for ruff (Python linter/formatter) |
| `ty` | Reference for ty (Python type checker) |
| `uv` | Reference for uv (Python package/project manager) |
| `slurm` | Core reference for the SONIC SLURM cluster |
| `slurm-status` | Cluster GPU availability check |
| `slurm-job` | SLURM job script creation |
| `slurm-debug` | SLURM job failure diagnosis |

## Conventions

- `merge` and `ship` are token-frugal: trivial checks run inline, mechanical
  subagents (test runs) use `haiku`, judgment subagents (diff review) use
  `opus`, and all subagents return terse verdicts instead of full logs.
- Install locally by symlinking a skill directory into `~/.claude/skills/`.
