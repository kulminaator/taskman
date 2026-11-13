# taskman

## TL;DR

Big scary task → tiny boring steps → done one at a time, in your repo, in git.

```text
you  ➜ taskman create "refactor auth" "Split the login module into ..."
tman ➜ 9 steps, ≤15 min each, filed under .taskman/refactor-auth/
you  ➜ taskman execute "refactor auth"
tman ➜ step-001 ✅  commit  step-002 ✅  commit  step-003 🚧 ...
```

- The plan is plain markdown in your repo, not a chat log you have to re-run.
- Exactly one step in play at a time. No drive-by "helpful" refactors.
- `status` files are the source of truth: `pending` → `in-progress` → `done`.
- Every gotcha learned mid-step gets written down for whoever runs it next.
- It commits as it goes, unless your project docs say don't.

Still here? The rest is the fine print. 🎺

---

A coding-agent skill for project task management, following the
[Agent Skills](https://agentskills.io/specification) specification.

`taskman` splits a task into small, exact steps (each ≤ 15 minutes), stores
them under `.taskman/<task-name>/` in your project, and executes them one at a
time — always re-reading the project docs and the task/step plans first,
logging findings into comment sections, tracking status per step, and (when a
git repo allows it) committing the work step by step.

Both commands start by reading the project's `README.md` and `GUIDELINES.md`
if they exist (`execute` looks in the project root and in the parent folders
of the current directory). Those docs also govern git: taskman only commits if
neither of them prohibits it.

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

The `TASK_HOME` folder name is derived deterministically from the task name:
lowercased, with spaces collapsed into single hyphens — `"Refactor Auth"`
becomes `.taskman/refactor-auth/`. `create` and `execute` derive it the same
way, so you can always predict (or recover) the folder name from the name you
typed.

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

Each `status` file holds exactly one word: `pending`, `in-progress`, `done`,
or `failed`. Status files are the single source of truth — `task.md` lists the
steps but never their statuses.

## Behavior worth knowing

- **Task not found:** `taskman execute` for an unknown name reports that no
  such task exists and lists the folders under `.taskman/`. It never guesses a
  similar task.
- **Task already exists:** `taskman create` refuses to overwrite or merge into
  an existing `TASK_HOME` and stops.
- **Step structure is frozen after planning.** All splitting happens during
  `create`; `execute` never splits, merges, inserts, deletes, or renumbers
  steps. If a step turns out too big in practice, that is recorded in its
  comments and the decision is left to you.
- **Failed steps are not retried automatically.** When the scan passes a
  `failed` step, taskman reports it and asks once whether to retry or skip it;
  a skip is recorded as a `skipped by user` bullet in that step's `step.md`.
- **Comments are append-only.** Step-specific findings go under `## Step
  comments` in `step.md`, project-wide findings under `## Comments` in
  `task.md`; existing comments are never rewritten.
- **One step at a time.** `execute` works on a single step and does not do
  opportunistic out-of-scope fixes.

## Files in this repo

- `README.md` — this file
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
# The repo lives at https://github.com/kulminaator/taskman; the clone location
# is free, only the link name inside the skills directory has to be taskman.
git clone https://github.com/kulminaator/taskman.git /path/to/taskman
```

### Pi

```bash
mkdir -p ~/.agents/skills
ln -s /path/to/taskman ~/.agents/skills/taskman
```

Project-wide instead: `.agents/skills/taskman` (Pi scans it from the working
directory up through its ancestors, stopping at the repository root).

Pi scans skill locations at startup, so start a new session to see a newly
added skill; use `/reload` to pick up edits to a skill you are already using.
Invoke with `/skill:taskman` (arguments after it are passed along, e.g.
`/skill:taskman execute "refactor auth"`), or just describe your task.

### Claude Code

```bash
mkdir -p ~/.claude/skills
ln -s /path/to/taskman ~/.claude/skills/taskman
```

Project-wide instead: commit the skill to `.claude/skills/taskman` in the
repository. Symlinked skill folders are supported. Claude Code watches skills
directories while it runs, so edits are picked up in the current session; if
you just created the top-level skills directory itself, run `/reload-skills`
(run it again after later changes there, since it is not watched yet). Run
`/skills` to confirm the skill loaded. Invoke with `/taskman`, or ask and
Claude loads the skill when the request matches its description.

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
mkdir -p ~/.config/opencode/skills
ln -s /path/to/taskman ~/.config/opencode/skills/taskman
```

Project-wide instead: `.opencode/skills/taskman` in the project root (OpenCode
searches from the current directory up to the project root). OpenCode also
reads the compatibility locations `~/.agents/skills`, `.agents/skills`,
`~/.claude/skills`, and `.claude/skills`, so a single link into any of those
serves OpenCode, Codex, and Pi at once.

OpenCode advertises the skill when it is relevant and loads the full
instructions on demand via the `skill` tool; you can also reference it
directly in your prompt.

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
