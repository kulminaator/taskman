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

Pi discovers skills as directories containing `SKILL.md`. To use this skill,
place (or symlink) this folder in a skills location, e.g.:

```text
.agents/skills/taskman/SKILL.md          # project skill
~/.agents/skills/taskman/SKILL.md        # user skill
```

Then invoke it with `/skill:taskman taskman execute my-task` or just ask the
agent to use taskman.

## License

MIT — see [LICENSE](LICENSE).
