---
description: The conductor. Orients you in the dev loop, reports queue state, and tells you the single next command to run.
---

# /loop - the conductor

This is the entry point. Run it whenever you are unsure what to do next. It reads the
current state of the queue and the memory inbox, works out where you are in the cycle,
and recommends the **one** next command to run. Nothing here edits code - it only orients.

The loop it conducts:

```
        /create-task ──▶ tasks/todo/
             ▲                │
             │                ▼
   /extract-followups ◀── /start-task ──▶ tasks/done/
             │            (implement +
             │             verify + tests)
             │
             └──────────────── memory/learnings/inbox/
                                       │
                                       ▼
                               /learn-and-improve
                       (propose edits to these skills; you approve)
```

## What to do

### Step 1: Read the map

Read `project/CONTEXT.md` (the description of the codebase this loop is driving). If it
still contains the placeholder text, tell the user: "Fill in `project/CONTEXT.md` first -
the skills need to know what they are working on." Then stop.

### Step 2: Take stock of the queue

Count the task files in each queue folder and read their titles:

```bash
echo "todo:"  ; ls tasks/todo/*.md   2>/dev/null || echo "  (empty)"
echo "doing:" ; ls tasks/doing/*.md  2>/dev/null || echo "  (empty)"
echo "done:"  ; ls tasks/done/*.md   2>/dev/null | wc -l
echo "unprocessed learnings:" ; ls memory/learnings/inbox/*.md 2>/dev/null | wc -l
```

### Step 3: Decide where you are and what runs next

Apply the first rule that matches, top to bottom, and recommend exactly one command:

| If...                                                        | You are at...        | Recommend                                   |
|--------------------------------------------------------------|----------------------|---------------------------------------------|
| a task is in `tasks/doing/`                                  | mid-implementation   | `/start-task` (resume the in-flight task)   |
| `tasks/todo/` has tasks and `tasks/doing/` is empty         | ready to build       | `/start-task` (pick the highest priority)   |
| `tasks/todo/` is empty and you have a new request in mind    | intake               | `/create-task "<your request>"`             |
| `memory/learnings/inbox/` has 5 or more files               | time to meta-improve | `/learn-and-improve`                        |
| everything is empty                                          | idle                 | `/create-task "<your next request>"`        |

`/extract-followups` is deliberately not a row here. Whether a finished task left
follow-ups is not something you can read off the queue, so the loop does not ask the
conductor to guess. It is chained straight from `/start-task`, which runs it at the end of
every task (see that skill). The conductor only routes on what it can actually observe.

### Step 4: Report

Tell the user, in three short lines:

- **Where you are**: one sentence naming the phase from the table.
- **Queue**: `todo: N · doing: M · done: K · learnings: L`.
- **Next**: the single recommended command, ready to copy.

Then stop. The conductor never runs the next command for the user - it hands them the
baton. Running one command at a time, with a human able to redirect between each, is the
point of the loop.

## Why a conductor exists

A pile of skills is not a workflow. The hard part of an agentic system is not any single
step - it is knowing, at any moment, which step you are on and what comes next. The
conductor makes the loop legible: a newcomer runs `/loop`, sees the whole cycle, and is
never lost. It is the onboarding and the orchestration in one file.
