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

Every skill in this diagram, not just `/extract-followups`, ends by writing straight to
`memory/learnings/inbox/` when its own run went sideways - the diagram only draws the arrow
where it does that in bulk, from a finished task's diff and review.

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
echo "working tree:" ; git status --short
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

Tell the user, in four short lines:

- **Where you are**: one sentence naming the phase from the table.
- **Queue**: `todo: N · doing: M · done: K · learnings: L · template: N behind`
  (omit the last field if this repo has no template bookmark).
- **Close**: `yes`, or `no - <the specific thing outstanding>`. See the close check below.
- **Next**: the single recommended command, ready to copy.

Then stop. The conductor never runs the next command for the user - it hands them the
baton. Running one command at a time, with a human able to redirect between each, is the
point of the loop.

### The close check

Answer this every time, not just when asked "is it safe to close" or "green light to
close" - those phrases should get the same line you would have reported anyway, not a
separate path.

**Close: yes** only if both hold:
- `tasks/doing/` is empty - nothing is claimed and mid-flight with no one owning it.
- `git status --short` is clean - nothing sits uncommitted where the next session (or the
  next agent in this one) won't see it.

Otherwise **Close: no**, and name the exact thing outstanding - the task file still in
`tasks/doing/`, or the specific paths `git status` shows dirty. Do not round a partial state
up to "basically done."

A non-zero learnings count does **not** block close - that queue drains on its own schedule
(`/learn-and-improve` at 5+), and every skill now logs to it as it goes rather than saving it
for the end. Mention the count in the Queue line; it is information, not a blocker.

## Step 5: Log anything that went sideways

This is about running `/next` itself - a queue count that didn't match what you expected, a
row in the Step 3 table that fired on the wrong condition, a close check that gave the wrong
answer. Write it down now, even if you already worked around it:

```markdown
memory/learnings/inbox/<YYYY-MM-DD-HHMMSS>-<slug>.md
---
date: <YYYY-MM-DD HH:MM:SS>
target_skill: /next | project
category: verification-gap | false-claim | environment | intake-gap | skill-design
---

<One or two sentences: what happened, and the rule or better way that would prevent it next time.>
```

Most runs have nothing to record - do not manufacture one.

## Why a conductor exists

A pile of skills is not a workflow. The hard part of an agentic system is not any single
step - it is knowing, at any moment, which step you are on and what comes next. The
conductor makes the loop legible: a newcomer runs `/next`, sees the whole cycle, and is
never lost. It is the onboarding and the orchestration in one file.
