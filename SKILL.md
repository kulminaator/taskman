---
name: taskman
description: Task management via "taskman create" and "taskman execute". Splits a task into small numbered steps under .taskman/<task-name>/, tracks each step's status, and executes steps one at a time while reading project docs first. Use when the user asks to plan, track, or carry out a multi-step task with taskman.
license: MIT
---

# taskman — project task manager

Manage multi-step tasks stored in the `.taskman/` folder of the current project.

Two commands:

```text
taskman create <task-name> <task-description>
taskman execute <task-name>
```

**Task-name** may be a single word or multiple words. When it is multiple words
the user must pass it in double quotes (e.g. `taskman create "refactor auth" "..."`).
Treat the quoted phrase as one task name; store the folder under `.taskman/`
using the name with spaces preserved (or hyphenated, but consistently — pick
hyphenated lowercase for folder safety, e.g. `refactor-auth`).

If the user asks to `taskman execute` (or `create` against an existing task with
a "find the task" intent) and the task cannot be found under `.taskman/`,
**tell the user clearly** that no task with that name exists and list the
available task folders (if any). Never guess a different task.

---

## Command: taskman create

`taskman create <task-name> <task-description>`

1. Read the project's `README.md` and `GUIDELINES.md` if present (see the
   reading rule below) so the plan fits the project.
2. Think through the `task-description` and split the work into **small, exact,
   concise steps**. Each step must be short enough to complete in **15 minutes
   or less**. Prefer more, smaller steps over fewer, larger ones. Every step
   must be concrete: say exactly what file(s) to touch, what to write, and how
   to verify it worked.
3. Create the task structure:

   ```text
   .taskman/
   └── <task-name>/            # TASK_HOME
       ├── task.md             # high-level description + empty comments section
       ├── step-001/
       │   ├── status          # contains: pending
       │   └── step.md         # detailed how-to + step-comments section
       ├── step-002/
       │   ├── status
       │   └── step.md
       └── ...                 # step-003, step-004, ... (zero-padded, in order)
   ```

   - `task.md`: use the template in `templates/task.md` (relative to this skill
     directory). It holds a high-level description of the task, the ordered
     step list (titles only — do **not** track step statuses in `task.md`; the
     `status` files are the single source of truth), and a `## Comments`
     section that starts **empty**.
   - Each step folder is named `step-001`, `step-002`, … zero-padded to three
     digits, numbered in execution order.
   - Each step folder contains a file named `status` whose content is exactly
     one of: `pending`, `in-progress`, `done`, `failed`. Initial content is
     `pending` (see `templates/status`).
   - Each step folder contains a `step.md` written from
     `templates/step.md`: a detailed, exact description of how to perform this
     step, ending with a `## Step comments` section (initially empty).

4. **Git:** if a git repository is present and `README.md` / `GUIDELINES.md` do
   not prohibit it, commit the newly created task structure (e.g.
   `taskman: create task <task-name> with N steps`). Do not commit anything
   unrelated that the user had staged/unstaged.

5. Report to the user: the TASK_HOME path and the numbered list of steps created.

---

## Command: taskman execute

`taskman execute <task-name>`

Follow this order strictly — do not skip reads even if you believe you have
read these files earlier in the session:

1. **Find and read the project docs**: locate `README.md` and `GUIDELINES.md`
   in the project (check the project root and parent folders of the current
   directory). Read each one in detail, now, with the read tool — even if you
   already read them some time ago. Note any rules that affect execution
   (e.g. git policy).
2. **Read the task**: read `.taskman/<task-name>/task.md` carefully. If the
   task folder or `task.md` does not exist, tell the user the task was not
   found and stop.
3. **Find the first unfinished step**: scan the `step-*` folders in numeric
   order and pick the first step whose `status` file is not `done`. That is the
   step to work on now.
   - If every step is `done`: congratulate the user, note the task is complete,
     and (if git is allowed) commit any remaining step changes.
   - If a step's status is `failed`, mention it when you reach it and ask the
     user whether to retry or skip before touching it.
4. **Announce the start**: tell the user the number of the step you started,
   e.g. "Starting step-003: <short title>". Write `in-progress` into that
   step's `status` file.
5. **Focus only on this step.** Do not start, modify, or "helpfully fix"
   anything outside this step. Finish it before touching anything else.
6. **Read the step plan**: read this step's `step.md` carefully and execute the
   actions it describes.
   - If during execution you learn **step-specific details** (gotchas, exact
     paths, corrections, deviations), append them as bullet points under the
     `## Step comments` section of that step's `step.md`.
   - If during execution you learn **globally important findings** (project
     conventions, blocking constraints, discoveries affecting other steps),
     append them as bullet points under the `## Comments` section of
     `task.md`.
7. **Finish the step**: when the step's work is verified complete, write `done`
   into its `status` file (write `failed` instead, plus a comment explaining
   why, if the step could not be completed). Tell the user the number of the
   step you finished, e.g. "Finished step-003".
8. **Git:** if a git repository is present and the README/GUIDELINES do not
   prohibit it, commit the step's changes after each finished step (e.g.
   `taskman: complete step-003 — <title>`). Task bookkeeping files
   (`status`, `step.md` comments) may be included in the same commit.
9. **Continue**: find the next step in sequence after the one just finished
   (the next not-`done` `step-*` folder) and repeat from point 4. Continue
   until all steps are done, the user stops you, or a step fails and needs the
   user's decision.

---

## Overall rules

- **Reading rule:** whenever the plan says to read `README.md`,
  `GUIDELINES.md`, `task.md`, or `step.md`, actually read the file with the
  read tool at that moment. Never rely on memory or assumptions.
- **Step size rule:** no step should take more than 15 minutes. If a step
  turns out to be bigger than that in practice, record that in its step
  comments and, with the user's agreement, split it into additional
  `step-NNN` folders at the end (or in place, renumbering, only if the user
  agrees).
- **Single-focus rule:** during execute, exactly one step is in play at a
  time. No parallel or opportunistic work.
- **Comments rule:** comments are append-only; never delete or rewrite
  existing comment content, and never remove the comments sections.
- **Status rule:** a `status` file always contains exactly one word:
  `pending`, `in-progress`, `done`, or `failed`.
- **Git rule:** when a git repo is present and README/GUIDELINES do not
  prohibit it, track task work with git — commit on create and after each
  finished step. Otherwise, make no git operations.
