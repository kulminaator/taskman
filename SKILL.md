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
Treat the quoted phrase as one task name. The TASK_HOME folder name is derived
deterministically: lowercase the name and replace spaces with single hyphens
(e.g. `"Refactor Auth"` → `.taskman/refactor-auth/`). Both `create` and
`execute` must derive the folder name exactly this way.

**Task not found (execute):** if `.taskman/<task-name>/` or its `task.md`
does not exist, tell the user clearly that no task with that name exists and
list the available task folders under `.taskman/` (if any). Never guess a
different task.

**Task already exists (create):** if the TASK_HOME folder already exists, tell
the user and stop. Do not overwrite or merge into an existing task unless the
user explicitly asks.

---

## Command: taskman create

`taskman create <task-name> <task-description>`

1. Read the project's `README.md` and `GUIDELINES.md` if present (see the
   reading rule below) so the plan fits the project.
2. Think through the `task-description` and split the work into **small, exact,
   concise steps**. Each step must be short enough to complete in **15 minutes
   or less**. Prefer more, smaller steps over fewer, larger ones. Every step
   must be concrete: say exactly what file(s) to touch, what to write, and how
   to verify it worked. This is the only chance to define the step structure:
   keep splitting until every step is genuinely small, because the structure
   will never be reworked later — re-splitting during execution is forbidden.
   Stay compact in the wording of steps but do not omit important details.
3. Create the task structure (first check the TASK_HOME folder does not
   already exist — see *Task already exists (create)* above):

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
6. Important! Do not forget to create the task and step files.

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
3. **Find the first step to work on**: scan the `step-*` folders in numeric
   order and pick the first step whose `status` is `pending` or
   `in-progress`. That is the step to work on now.
   - `failed` steps are not automatically selected. When the scan passes a
     `failed` step, report it and ask the user whether to retry or skip it
     **once**; if the user says skip, append a `skipped by user` bullet to
     that step's `step.md` comments and move past it without asking again in
     this run.
   - If there are no `pending`/`in-progress` steps left: end the run with a
     summary — count of `done` steps and any `failed`/skipped steps. If all
     steps are `done`, note the task is complete, and (if git is allowed)
     commit any remaining step changes.
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
   (the next `pending`/`in-progress` `step-*` folder, passing skipped `failed`
   steps as described in point 3) and repeat from point 4. Continue until no
   workable step remains, the user stops you, or a step fails and needs the
   user's decision.
10. Important! Do not consider your actions complete before all the steps in 
   the task have been handled and reached done!

---

## Overall rules

- **Reading rule:** whenever the plan says to read `README.md`,
  `GUIDELINES.md`, `task.md`, or `step.md`, actually read the whole file with 
  the read tool at that moment. Never rely on memory or assumptions.
- **Step size rule:** no step should take more than 15 minutes. All splitting
  into small steps happens during **create** (initial planning) — invest the
  effort there, splitting until every step is small and concrete. During
  execute, splitting, merging, inserting, deleting, or renumbering steps is
  **forbidden**. If a step turns out to be too big in practice, do not
  restructure the task: record that in its step comments, do as much of the
  step as is reasonable, and let the user decide what to do (finish it anyway,
  mark it failed, or create a follow-up task via a new `taskman create`).
- **Single-focus rule:** during execute, exactly one step is in play at a
  time. No parallel or opportunistic work.
- **Comments rule:** comments are append-only; never delete or rewrite
  existing comment content, and never remove the comments sections.
- **Status rule:** a `status` file always contains exactly one word:
  `pending`, `in-progress`, `done`, or `failed`.
- **Git rule:** when a git repo is present and README/GUIDELINES do not
  prohibit it, track task work with git — commit on create and after each
  finished step. Otherwise, make no git operations.
