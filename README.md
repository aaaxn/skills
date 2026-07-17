# skills

Personal agent skills. Skill directories stay at the repository root so the
available commands are visible without navigating through category folders.

## Included skills

| Skill | Purpose | Source |
| --- | --- | --- |
| [`deslop`](./deslop/) | Remove AI-generated noise from a branch diff | Maintained in this repository |
| [`bro`](./bro/) | Restate the previous response in plain language | [`dmmulroy/skills`](https://github.com/dmmulroy/skills) |
| [`ty`](./ty/) | Reference for the ty Python type checker | Maintained here using the [official ty documentation](https://docs.astral.sh/ty/) |
| [`uv`](./uv/) | Reference for the uv Python package manager | Maintained here using the [official uv documentation](https://docs.astral.sh/uv/) |

## Deprecated

The old SONIC SLURM skills remain under [`depreceted/`](./depreceted/) for
historical reference.

## Other installed skills

These skills are installed from their upstream repositories and are not
vendored here.

| Skills | Upstream |
| --- | --- |
| Matt Pocock engineering and productivity skills | [`mattpocock/skills`](https://github.com/mattpocock/skills) |
| `thermos`, `thermo-nuclear-review`, `thermo-nuclear-code-quality-review` | [`cursor/plugins`](https://github.com/cursor/plugins/tree/main/thermos) |
| `ast-grep` | [`ast-grep/agent-skill`](https://github.com/ast-grep/agent-skill) |
| `arxiv` | [`NousResearch/hermes-agent`](https://github.com/NousResearch/hermes-agent/tree/main/skills/research/arxiv) |
| `wandb-primary` | [`wandb/skills`](https://github.com/wandb/skills) |
| `improve` | [`shadcn/improve`](https://github.com/shadcn/improve) |
| `write-better` | [`plannotator/write-better`](https://github.com/plannotator/write-better) |

The locally maintained `merge`, `ship`, `ruff`, and `sync-experiments` skills
are installed separately and are not part of this repository.

## Install

Symlink the desired top-level skill directory into `~/.agents/skills/`.
