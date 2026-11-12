# taskman

A coding-agent skill for project task management, following the
[Agent Skills](https://agentskills.io/specification) specification.

`taskman` splits a task into small, exact steps (each ≤ 15 minutes), stores
them under `.taskman/<task-name>/` in your project, and executes them one at a
time — always re-reading the project docs and the task/step plans first,
logging findings into comment sections, tracking status per step, and (when a
git repo allows it) committing the work step by step.

## Commands

```text
taskman create <task-name> <task-description>
taskman execute <task-name>
```

Multi-word task names must be quoted:

```text
taskman create "refactor auth" "Split the login module into ..."
taskman execute "refactor auth"
```

## Layout created by taskman

```text
.taskman/
└── <task-name>/            # TASK_HOME
    ├── task.md             # high-level description + global comments
    ├── step-001/
    │   ├── status          # pending | in-progress | done | failed
    │   └── step.md         # exact how-to + step comments
    └── step-002/ ...
```

## Files in this repo

- `SKILL.md` — the skill instructions (entry point)
- `templates/task.md` — template for `task.md`
- `templates/step.md` — template for every `step.md`
- `templates/status` — initial status file content (`pending`)
- `LICENSE` — MIT

## Installation

`taskman` follows the open [Agent Skills](https://agentskills.io) standard, so
it works with any agent that supports skills. The idea is always the same:
make this folder available under the agent's skills directory under the name
`taskman`, so `SKILL.md` sits directly inside `.../skills/taskman/`.

A symlink to a clone of this repo works everywhere below and keeps the skill
up to date when you `git pull`; a plain `git clone` (or copy) works too:

```bash
git clone https://github.com/Kulminaator/taskman /path/to/taskman
```

### Pi

```bash
mkdir -p ~/.agents/skills
ln -s /path/to/taskman ~/.agents/skills/taskman
```

Project-wide instead: `.agents/skills/taskman`. Restart Pi to pick it up,
then invoke with `/skill:taskman` or just describe your task.

### Claude Code

```bash
mkdir -p ~/.claude/skills
ln -s /path/to/taskman ~/.claude/skills/taskman
```

Project-wide instead: commit the skill to `.claude/skills/taskman` in the
repository. Symlinked skill folders are supported. Restart Claude Code to
pick up a new skills directory (it watches skills dirs while running). Invoke
with `/taskman`, or ask and Claude loads the skill when the request matches
its description.

### OpenAI Codex

```bash
mkdir -p ~/.agents/skills
ln -s /path/to/taskman ~/.agents/skills/taskman
```

Project-wide instead: `.agents/skills/taskman` at the repository root (Codex
scans `.agents/skills` from the current directory up to the repo root).
Symlinked skill folders are supported. Codex detects skill changes
automatically; restart it if a new skill doesn't appear. Invoke with
`$taskman` or `/skills`, or describe your task.

### OpenCode

```bash
mkdir -p ~/.opencode/skills
ln -s /path/to/taskman ~/.opencode/skills/taskman
```

Project-wide instead: `.opencode/skills/taskman` in the project root. OpenCode
loads the skill's full instructions when your request matches its
description, or reference it directly in your prompt.

### Other agents

Any agent implementing the Agent Skills standard works, as long as the folder
containing `SKILL.md` sits in a location that agent scans for skills.

Once installed, ask the agent things like:

```text
taskman create "refactor auth" "Split the login module into ..."
taskman execute "refactor auth"
```

## License

MIT — see [LICENSE](LICENSE).
