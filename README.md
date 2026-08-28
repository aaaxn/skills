# skills

Personal agent skills. Every skill lives under [`skills/`](./skills/) so the
repository root stays readable.

Most of these were written by other people. Each one is credited below with a
link to its upstream source and the command that installs it. Only the skills
under [Maintained here](#maintained-here) are original to this repository.

Install commands use the [skills.sh](https://www.skills.sh/) CLI:

```bash
npx skills@latest add <owner>/<repo> --skill <name>
```

Drop `--skill` to install every skill in a repository at once.

## Maintained here

| Skill | Purpose |
| --- | --- |
| [`deslop`](./skills/deslop/) | Remove AI-generated noise from a branch diff |
| [`ruff`](./skills/ruff/) | Reference for the [ruff](https://docs.astral.sh/ruff/) Python linter and formatter |
| [`ty`](./skills/ty/) | Reference for the [ty](https://docs.astral.sh/ty/) Python type checker |
| [`uv`](./skills/uv/) | Reference for the [uv](https://docs.astral.sh/uv/) Python package manager |

```bash
npx skills@latest add aaaxn/skills --skill deslop
npx skills@latest add aaaxn/skills --skill ruff
npx skills@latest add aaaxn/skills --skill ty
npx skills@latest add aaaxn/skills --skill uv
```

## From [`mattpocock/skills`](https://github.com/mattpocock/skills)

By [Matt Pocock](https://github.com/mattpocock). Browse them on
[skills.sh](https://www.skills.sh/mattpocock/skills).

| Skill | Purpose |
| --- | --- |
| [`ask-matt`](./skills/ask-matt/) | Router that picks which skill or flow fits your situation |
| [`claude-handoff`](./skills/claude-handoff/) | Hand the conversation off to a fresh background agent |
| [`code-review`](./skills/code-review/) | Review changes since a fixed point along standards and spec axes |
| [`codebase-design`](./skills/codebase-design/) | Shared vocabulary for designing deep modules |
| [`diagnosing-bugs`](./skills/diagnosing-bugs/) | Diagnosis loop for hard bugs and performance regressions |
| [`domain-modeling`](./skills/domain-modeling/) | Build and sharpen a project's domain model |
| [`edit-article`](./skills/edit-article/) | Restructure and tighten an article draft |
| [`git-guardrails-claude-code`](./skills/git-guardrails-claude-code/) | Hooks that block dangerous git commands |
| [`grill-me`](./skills/grill-me/) | A relentless interview to sharpen a plan or design |
| [`grill-with-docs`](./skills/grill-with-docs/) | The same interview, producing ADRs and a glossary as it goes |
| [`grilling`](./skills/grilling/) | Stress-test a plan, decision, or idea |
| [`handoff`](./skills/handoff/) | Compact the conversation into a handoff document |
| [`implement`](./skills/implement/) | Implement work from a spec or set of tickets |
| [`improve-codebase-architecture`](./skills/improve-codebase-architecture/) | Scan for deepening opportunities and report them visually |
| [`loop-me`](./skills/loop-me/) | Grill the user about specs for workflows to build |
| [`migrate-to-shoehorn`](./skills/migrate-to-shoehorn/) | Replace `as` assertions in tests with `@total-typescript/shoehorn` |
| [`prototype`](./skills/prototype/) | Build a throwaway prototype to answer a design question |
| [`research`](./skills/research/) | Investigate a question against primary sources, capture it as Markdown |
| [`resolving-merge-conflicts`](./skills/resolving-merge-conflicts/) | Resolve an in-progress merge or rebase |
| [`scaffold-exercises`](./skills/scaffold-exercises/) | Create exercise directory structures |
| [`setup-matt-pocock-skills`](./skills/setup-matt-pocock-skills/) | One-time repo setup for the engineering skills |
| [`setup-pre-commit`](./skills/setup-pre-commit/) | Husky pre-commit hooks with lint-staged, typecheck, and tests |
| [`tdd`](./skills/tdd/) | Test-driven development |
| [`teach`](./skills/teach/) | Teach a new skill or concept within the workspace |
| [`to-spec`](./skills/to-spec/) | Turn the conversation into a spec on the issue tracker |
| [`to-tickets`](./skills/to-tickets/) | Break a plan into tracer-bullet tickets with blocking edges |
| [`triage`](./skills/triage/) | Move issues and PRs through a triage state machine |
| [`wayfinder`](./skills/wayfinder/) | Plan multi-session work as decision tickets on a tracker |
| [`wizard`](./skills/wizard/) | Generate an interactive bash wizard for human-only steps |
| [`writing-beats`](./skills/writing-beats/) | Assemble raw material into a journey of beats |
| [`writing-for-agents`](./skills/writing-for-agents/) | Writing skills, `AGENTS.md`, and `CLAUDE.md` |
| [`writing-fragments`](./skills/writing-fragments/) | Mine raw fragments before imposing structure |
| [`writing-great-skills`](./skills/writing-great-skills/) | Compatibility alias for `writing-for-agents` |
| [`writing-shape`](./skills/writing-shape/) | Shape raw material into an article, paragraph by paragraph |

```bash
npx skills@latest add mattpocock/skills --skill ask-matt
npx skills@latest add mattpocock/skills --skill claude-handoff
npx skills@latest add mattpocock/skills --skill code-review
npx skills@latest add mattpocock/skills --skill codebase-design
npx skills@latest add mattpocock/skills --skill diagnosing-bugs
npx skills@latest add mattpocock/skills --skill domain-modeling
npx skills@latest add mattpocock/skills --skill edit-article
npx skills@latest add mattpocock/skills --skill git-guardrails-claude-code
npx skills@latest add mattpocock/skills --skill grill-me
npx skills@latest add mattpocock/skills --skill grill-with-docs
npx skills@latest add mattpocock/skills --skill grilling
npx skills@latest add mattpocock/skills --skill handoff
npx skills@latest add mattpocock/skills --skill implement
npx skills@latest add mattpocock/skills --skill improve-codebase-architecture
npx skills@latest add mattpocock/skills --skill loop-me
npx skills@latest add mattpocock/skills --skill migrate-to-shoehorn
npx skills@latest add mattpocock/skills --skill prototype
npx skills@latest add mattpocock/skills --skill research
npx skills@latest add mattpocock/skills --skill resolving-merge-conflicts
npx skills@latest add mattpocock/skills --skill scaffold-exercises
npx skills@latest add mattpocock/skills --skill setup-matt-pocock-skills
npx skills@latest add mattpocock/skills --skill setup-pre-commit
npx skills@latest add mattpocock/skills --skill tdd
npx skills@latest add mattpocock/skills --skill teach
npx skills@latest add mattpocock/skills --skill to-spec
npx skills@latest add mattpocock/skills --skill to-tickets
npx skills@latest add mattpocock/skills --skill triage
npx skills@latest add mattpocock/skills --skill wayfinder
npx skills@latest add mattpocock/skills --skill wizard
npx skills@latest add mattpocock/skills --skill writing-beats
npx skills@latest add mattpocock/skills --skill writing-for-agents
npx skills@latest add mattpocock/skills --skill writing-fragments
npx skills@latest add mattpocock/skills --skill writing-shape
```

`edit-article` has since been removed upstream; it is kept here from an earlier
revision of that repository. `writing-great-skills` is a local compatibility
alias for `writing-for-agents` and has no upstream command of its own.

## From [`cursor/plugins`](https://github.com/cursor/plugins)

By the [Cursor](https://github.com/cursor) team.

| Skill | Purpose | Upstream plugin |
| --- | --- | --- |
| [`unslop`](./skills/unslop/) | Cut AI tells from any writing | [`pstack`](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop) |
| [`show-me-your-work`](./skills/show-me-your-work/) | Keep a reviewable TSV decision trail for long-running work | [`pstack`](https://github.com/cursor/plugins/tree/main/pstack/skills/show-me-your-work) |
| [`thermos`](./skills/thermos/) | Run both thermo-nuclear reviews in parallel and synthesize | [`thermos`](https://github.com/cursor/plugins/tree/main/thermos) |
| [`thermo-nuclear-review`](./skills/thermo-nuclear-review/) | Security and correctness audit of a branch's changes | [`thermos`](https://github.com/cursor/plugins/tree/main/thermos) |
| [`thermo-nuclear-code-quality-review`](./skills/thermo-nuclear-code-quality-review/) | Strict maintainability and abstraction review | [`thermos`](https://github.com/cursor/plugins/tree/main/thermos) |

```bash
npx skills@latest add cursor/plugins --skill unslop
npx skills@latest add cursor/plugins --skill show-me-your-work
npx skills@latest add cursor/plugins --skill thermos
npx skills@latest add cursor/plugins --skill thermo-nuclear-review
npx skills@latest add cursor/plugins --skill thermo-nuclear-code-quality-review
```

## From individual authors

| Skill | Purpose | Author | Source |
| --- | --- | --- | --- |
| [`arxiv`](./skills/arxiv/) | Search arXiv by keyword, author, category, or ID | Nous Research | [`NousResearch/hermes-agent`](https://github.com/NousResearch/hermes-agent/tree/main/skills/research/arxiv) |
| [`bro`](./skills/bro/) | Restate the last message in plain language | [Dillon Mulroy](https://github.com/dmmulroy) | [`dmmulroy/skills`](https://github.com/dmmulroy/skills/tree/main/bro) |
| [`code-like-luke`](./skills/code-like-luke/) | Happy-path-first, use-case-oriented implementation | [Hona](https://github.com/Hona) | [gist](https://gist.github.com/Hona/53142c07c9decb735392f132ace34003) |
| [`frontend-design`](./skills/frontend-design/) | Distinctive, intentional visual design for new UI | Anthropic | [`anthropics/skills`](https://github.com/anthropics/skills/tree/main/skills/frontend-design) |
| [`guided-review`](./skills/guided-review/) | Walk a human through a large diff in semantic chunks | [Hona](https://github.com/Hona) | [gist](https://gist.github.com/Hona/a65d2ca408ce1092fcf00e762b7a6aab) |
| [`improve-claude-md`](./skills/improve-claude-md/) | Rewrite a `CLAUDE.md` using `<important if>` blocks | [HumanLayer](https://github.com/humanlayer) | [`humanlayer/skills`](https://github.com/humanlayer/skills/tree/main/plugins/improve-claude-md) |
| [`prune-dead-code`](./skills/prune-dead-code/) | Audit for dead and safe-to-remove code | [Luke Berry](https://github.com/LukeberryPi) | [`lukeberrypi/skills`](https://github.com/LukeberryPi/skills/tree/main/skills/prune-dead-code) |
| [`remove-dumb-comments`](./skills/remove-dumb-comments/) | Flag comments that restate the code | [Luke Berry](https://github.com/LukeberryPi) | [`lukeberrypi/skills`](https://github.com/LukeberryPi/skills/tree/main/skills/remove-dumb-comments) |
| [`show-me`](./skills/show-me/) | Explain the current topic visually | [HumanLayer](https://github.com/humanlayer) | [`humanlayer/skills`](https://github.com/humanlayer/skills/tree/main/plugins/show-me) |
| [`stop-slop`](./skills/stop-slop/) | Remove AI writing patterns from prose | [Hardik Pandya](https://hvpandya.com) | [`hardikpandya/stop-slop`](https://github.com/hardikpandya/stop-slop) |
| [`wandb-primary`](./skills/wandb-primary/) | Broad Weights & Biases work: runs, artifacts, Weave, Reports | Weights & Biases | [`wandb/skills`](https://github.com/wandb/skills/tree/main/skills/wandb-primary) |
| [`write-better`](./skills/write-better/) | Improve, rewrite, and review prose | [Plannotator](https://github.com/plannotator) | [`plannotator/write-better`](https://github.com/plannotator/write-better/tree/main/skills/write-better) |

```bash
npx skills@latest add NousResearch/hermes-agent --skill arxiv
npx skills@latest add dmmulroy/skills --skill bro
npx skills@latest add anthropics/skills --skill frontend-design
npx skills@latest add humanlayer/skills --skill improve-claude-md
npx skills@latest add lukeberrypi/skills --skill prune-dead-code
npx skills@latest add lukeberrypi/skills --skill remove-dumb-comments
npx skills@latest add humanlayer/skills --skill show-me
npx skills@latest add hardikpandya/stop-slop --skill stop-slop
npx skills@latest add wandb/skills --skill wandb-primary
npx skills@latest add plannotator/write-better --skill write-better
```

`code-like-luke` and `guided-review` come from gists rather than repositories,
so they have no `skills add` command. Copy them into `skills/` by hand.

## Deprecated

The old SONIC SLURM skills remain under [`depreceted/`](./depreceted/) for
historical reference.
