# skills

Personal agent skills. Skill directories stay at the repository root so the
available commands are visible without navigating through category folders.

Most of these skills were written by other people. Every one is credited below
with a link to its upstream source. Only the skills marked *maintained here* are
original to this repository.

## Maintained here

| Skill | Purpose |
| --- | --- |
| [`deslop`](./deslop/) | Remove AI-generated noise from a branch diff |
| [`merge`](./merge/) | Merge a PR with easy, adaptive, or full verification |
| [`ship`](./ship/) | Commit, push, and open a PR with easy, adaptive, or full verification |
| [`ruff`](./ruff/) | Reference for the [ruff](https://docs.astral.sh/ruff/) Python linter and formatter |
| [`ty`](./ty/) | Reference for the [ty](https://docs.astral.sh/ty/) Python type checker |
| [`uv`](./uv/) | Reference for the [uv](https://docs.astral.sh/uv/) Python package manager |

## From [`mattpocock/skills`](https://github.com/mattpocock/skills)

By [Matt Pocock](https://github.com/mattpocock). Browse them on
[skills.sh](https://www.skills.sh/mattpocock/skills).

| Skill | Purpose |
| --- | --- |
| [`ask-matt`](./ask-matt/) | Router that picks which skill or flow fits your situation |
| [`claude-handoff`](./claude-handoff/) | Hand the conversation off to a fresh background agent |
| [`code-review`](./code-review/) | Review changes since a fixed point along standards and spec axes |
| [`codebase-design`](./codebase-design/) | Shared vocabulary for designing deep modules |
| [`diagnosing-bugs`](./diagnosing-bugs/) | Diagnosis loop for hard bugs and performance regressions |
| [`domain-modeling`](./domain-modeling/) | Build and sharpen a project's domain model |
| [`edit-article`](./edit-article/) | Restructure and tighten an article draft |
| [`git-guardrails-claude-code`](./git-guardrails-claude-code/) | Hooks that block dangerous git commands |
| [`grill-me`](./grill-me/) | A relentless interview to sharpen a plan or design |
| [`grill-with-docs`](./grill-with-docs/) | The same interview, producing ADRs and a glossary as it goes |
| [`grilling`](./grilling/) | Stress-test a plan, decision, or idea |
| [`handoff`](./handoff/) | Compact the conversation into a handoff document |
| [`implement`](./implement/) | Implement work from a spec or set of tickets |
| [`improve-codebase-architecture`](./improve-codebase-architecture/) | Scan for deepening opportunities and report them visually |
| [`loop-me`](./loop-me/) | Grill the user about specs for workflows to build |
| [`migrate-to-shoehorn`](./migrate-to-shoehorn/) | Replace `as` assertions in tests with `@total-typescript/shoehorn` |
| [`prototype`](./prototype/) | Build a throwaway prototype to answer a design question |
| [`research`](./research/) | Investigate a question against primary sources, capture it as Markdown |
| [`resolving-merge-conflicts`](./resolving-merge-conflicts/) | Resolve an in-progress merge or rebase |
| [`scaffold-exercises`](./scaffold-exercises/) | Create exercise directory structures |
| [`setup-matt-pocock-skills`](./setup-matt-pocock-skills/) | One-time repo setup for the engineering skills |
| [`setup-pre-commit`](./setup-pre-commit/) | Husky pre-commit hooks with lint-staged, typecheck, and tests |
| [`tdd`](./tdd/) | Test-driven development |
| [`teach`](./teach/) | Teach a new skill or concept within the workspace |
| [`to-spec`](./to-spec/) | Turn the conversation into a spec on the issue tracker |
| [`to-tickets`](./to-tickets/) | Break a plan into tracer-bullet tickets with blocking edges |
| [`triage`](./triage/) | Move issues and PRs through a triage state machine |
| [`wayfinder`](./wayfinder/) | Plan multi-session work as decision tickets on a tracker |
| [`wizard`](./wizard/) | Generate an interactive bash wizard for human-only steps |
| [`writing-beats`](./writing-beats/) | Assemble raw material into a journey of beats |
| [`writing-for-agents`](./writing-for-agents/) | Writing skills, `AGENTS.md`, and `CLAUDE.md` |
| [`writing-fragments`](./writing-fragments/) | Mine raw fragments before imposing structure |
| [`writing-great-skills`](./writing-great-skills/) | Compatibility alias for `writing-for-agents` |
| [`writing-shape`](./writing-shape/) | Shape raw material into an article, paragraph by paragraph |

`edit-article` has since been removed upstream; it is kept here from an earlier
revision of that repository.

## From [`cursor/plugins`](https://github.com/cursor/plugins)

By the [Cursor](https://github.com/cursor) team.

| Skill | Purpose | Upstream |
| --- | --- | --- |
| [`unslop`](./unslop/) | Cut AI tells from any writing | [`pstack`](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop) |
| [`show-me-your-work`](./show-me-your-work/) | Keep a reviewable TSV decision trail for long-running work | [`pstack`](https://github.com/cursor/plugins/tree/main/pstack/skills/show-me-your-work) |
| [`thermos`](./thermos/) | Run both thermo-nuclear reviews in parallel and synthesize | [`thermos`](https://github.com/cursor/plugins/tree/main/thermos) |
| [`thermo-nuclear-review`](./thermo-nuclear-review/) | Security and correctness audit of a branch's changes | [`thermos`](https://github.com/cursor/plugins/tree/main/thermos) |
| [`thermo-nuclear-code-quality-review`](./thermo-nuclear-code-quality-review/) | Strict maintainability and abstraction review | [`thermos`](https://github.com/cursor/plugins/tree/main/thermos) |

## From individual authors

| Skill | Purpose | Author | Source |
| --- | --- | --- | --- |
| [`arxiv`](./arxiv/) | Search arXiv by keyword, author, category, or ID | Nous Research | [`NousResearch/hermes-agent`](https://github.com/NousResearch/hermes-agent/tree/main/skills/research/arxiv) |
| [`bro`](./bro/) | Restate the last message in plain language | [Dillon Mulroy](https://github.com/dmmulroy) | [`dmmulroy/skills`](https://github.com/dmmulroy/skills/tree/main/bro) |
| [`code-like-luke`](./code-like-luke/) | Happy-path-first, use-case-oriented implementation | [Hona](https://github.com/Hona) | [gist](https://gist.github.com/Hona/53142c07c9decb735392f132ace34003) |
| [`frontend-design`](./frontend-design/) | Distinctive, intentional visual design for new UI | Anthropic | [`anthropics/skills`](https://github.com/anthropics/skills/tree/main/skills/frontend-design) |
| [`guided-review`](./guided-review/) | Walk a human through a large diff in semantic chunks | [Hona](https://github.com/Hona) | [gist](https://gist.github.com/Hona/a65d2ca408ce1092fcf00e762b7a6aab) |
| [`improve-claude-md`](./improve-claude-md/) | Rewrite a `CLAUDE.md` using `<important if>` blocks | [HumanLayer](https://github.com/humanlayer) | [`humanlayer/skills`](https://github.com/humanlayer/skills/tree/main/plugins/improve-claude-md) |
| [`prune-dead-code`](./prune-dead-code/) | Audit for dead and safe-to-remove code | [Luke Berry](https://github.com/LukeberryPi) | [`lukeberrypi/skills`](https://github.com/LukeberryPi/skills/tree/main/skills/prune-dead-code) |
| [`remove-dumb-comments`](./remove-dumb-comments/) | Flag comments that restate the code | [Luke Berry](https://github.com/LukeberryPi) | [`lukeberrypi/skills`](https://github.com/LukeberryPi/skills/tree/main/skills/remove-dumb-comments) |
| [`show-me`](./show-me/) | Explain the current topic visually | [HumanLayer](https://github.com/humanlayer) | [`humanlayer/skills`](https://github.com/humanlayer/skills/tree/main/plugins/show-me) |
| [`stop-slop`](./stop-slop/) | Remove AI writing patterns from prose | [Hardik Pandya](https://hvpandya.com) | [`hardikpandya/stop-slop`](https://github.com/hardikpandya/stop-slop) |
| [`wandb-primary`](./wandb-primary/) | Broad Weights & Biases work: runs, artifacts, Weave, Reports | Weights & Biases | [`wandb/skills`](https://github.com/wandb/skills/tree/main/skills/wandb-primary) |
| [`write-better`](./write-better/) | Improve, rewrite, and review prose | [Plannotator](https://github.com/plannotator) | [`plannotator/write-better`](https://github.com/plannotator/write-better/tree/main/skills/write-better) |

## Deprecated

The old SONIC SLURM skills remain under [`depreceted/`](./depreceted/) for
historical reference.

## Installed separately

These are used locally but are not part of this repository.

| Skill | Source |
| --- | --- |
| `paper-writing` | [`SNL-UCSB/paper-writing-skill`](https://github.com/SNL-UCSB/paper-writing-skill) |
| `sync-experiments` | Local, unpublished |
