---
description: The conductor. Orients you in the dev loop, reports queue state, and tells you the single next command to run.
---

# /next - the conductor

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
                                       │
                        portable fixes │ ▲ template improvements
                                       ▼ │
                              the upstream template
                                       │
                                /sync-template
                     (adopt what fits, refuse what does not,
                      record both in project/TEMPLATE-SYNC.md)
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

If `project/TEMPLATE-SYNC.md` exists, also read how far behind the template this repo is -
using the last-fetched ref, without going to the network:

```bash
grep -m1 '^\*\*Synced to\*\*' project/TEMPLATE-SYNC.md
git rev-list --count "<that sha>..template/<branch>" 2>/dev/null || echo "  (no template remote fetched)"
```

Report that count as **template: N behind**, and say plainly that it is only as fresh as the
last fetch - `/sync-template` fetches for real. Do not fetch here: the conductor is read-only
and instant, and a network call on every orientation would make people stop running it.

### Step 3: Decide where you are and what runs next

Apply the first rule that matches, top to bottom, and recommend exactly one command:

| If...                                                        | You are at...        | Recommend                                   |
|--------------------------------------------------------------|----------------------|---------------------------------------------|
| a task is in `tasks/doing/`                                  | mid-implementation   | `/start-task` (resume the in-flight task)   |
| `tasks/todo/` has tasks and `tasks/doing/` is empty         | ready to build       | `/start-task` (pick the highest priority)   |
| `tasks/todo/` is empty and you have a new request in mind    | intake               | `/create-task "<your request>"`             |
| `memory/learnings/inbox/` has 5 or more files               | time to meta-improve | `/learn-and-improve`                        |
| the template bookmark is behind (see below)                  | template has moved   | `/sync-template`                            |
| everything is empty                                          | idle                 | `/create-task "<your next request>"`        |

The template row sits below the learnings row on purpose: pulling someone else's changes into
the skills while your own recorded mistakes are still unprocessed means reviewing an upstream
diff against instructions you were about to rewrite anyway. Drain first, then sync. And it
fires on a *stale* count - if the remote has never been fetched the count is unknown, not zero,
so mention it as unknown rather than reporting the repo up to date.

`/extract-followups` is deliberately not a row here. Whether a finished task left
follow-ups is not something you can read off the queue, so the loop does not ask the
conductor to guess. It is chained straight from `/start-task`, which runs it at the end of
every task (see that skill). The conductor only routes on what it can actually observe.

### Step 4: Report

Tell the user, in three short lines:

- **Where you are**: one sentence naming the phase from the table.
- **Queue**: `todo: N · doing: M · done: K · learnings: L · template: N behind`
  (omit the last field if this repo has no template bookmark).
- **Next**: the single recommended command, ready to copy.

Then stop. The conductor never runs the next command for the user - it hands them the
baton. Running one command at a time, with a human able to redirect between each, is the
point of the loop.

## Why a conductor exists

A pile of skills is not a workflow. The hard part of an agentic system is not any single
step - it is knowing, at any moment, which step you are on and what comes next. The
conductor makes the loop legible: a newcomer runs `/next`, sees the whole cycle, and is
never lost. It is the onboarding and the orchestration in one file.
