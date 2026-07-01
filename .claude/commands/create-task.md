---
description: Turn a rough request into one well-formed, self-contained task file in tasks/todo/.
---

# /create-task - intake

Turn a request (`$ARGUMENTS`, or ask the user for one) into a single task file that a
fresh agent could pick up and implement **without asking follow-up questions**. A good
task file is the highest-leverage artifact in the loop: everything downstream inherits
its quality.

## Step 1: Understand before you write

- Read `project/CONTEXT.md` so the task is grounded in the real codebase.
- Search the code for the files, functions, and patterns the request touches. A task that
  names real paths and existing conventions is worth ten that hand-wave.
- If the request bundles several unrelated changes, split it: one task file per coherent
  unit of work. Say so, then create each file.

## Step 2: Resolve the ambiguities yourself

For each thing the request leaves open, pick the sensible default from the codebase's
existing conventions and **write the decision into the task** rather than leaving a
question for later. Only escalate to the user a fork that genuinely changes the outcome
and that conventions cannot settle. A task file with an open question in it is a task file
that will stall at implementation time.

## Step 3: Write the file

Name it `tasks/todo/<short-kebab-slug>.md` and use this shape:

```markdown
# <Task title>

**Priority**: high | medium | low
**Status**: todo

## Goal
One or two sentences: what outcome this task produces and why it matters. Describe the
intent, not just the steps - the implementer needs to know what "done" is *for*.

## Context
The real files, functions, and patterns involved (with paths). Any decision already made
and the reason. Anything a fresh agent would otherwise have to rediscover.

## Plan
- [ ] Concrete step, naming the file it touches
- [ ] ...

## Acceptance criteria
- [ ] A specific, checkable outcome. "The badge shows the unread count and hides at zero."
- [ ] ... (never "works correctly" or "looks good" - those are not testable)

## Notes
Edge cases, things explicitly out of scope, links to related tasks.
```

## Step 4: Self-check before handing off

Re-read the file as if you were the implementer and had never seen the request. Ask:

- Could I start without asking a single question?
- Is every acceptance criterion something I could objectively verify?
- Does it name real paths, or am I about to send someone hunting?

Fix anything that fails, then report the created path(s) and suggest `/start-task` next.

## Why intake is its own step

Separating "decide what to do" from "do it" keeps each cheap. When the task is written
down and self-contained, the implementer runs with full context, the reviewer has a spec
to check against, and follow-ups have somewhere to land. Skipping intake and coding
straight from a chat message is how scope drifts and half the request gets forgotten.
