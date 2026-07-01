---
description: Capture the follow-ups and concerns a finished task surfaced, turning them into queued tasks and durable notes so nothing is lost.
---

# /extract-followups - close the loop

Implementation always turns up more than it fixes: a minor review finding you deferred, a
smell in nearby code, a "we should really also..." thought. Left in your head or buried in
a review, they evaporate. This skill routes each one to where it will be acted on, so the
loop feeds itself instead of leaking.

Run this after `/start-task`, or any time a session ends with loose ends.

## Step 1: Gather the loose ends

Collect, from the task you just finished and its review:

- MINOR review findings that were deferred rather than fixed.
- Adjacent problems you noticed but kept out of scope.
- Assumptions you made that a human should sanity-check.
- Anything you thought "someone should look at this" about.

If there are none, say so and stop. Not every task spawns follow-ups, and inventing them
is noise.

## Step 2: Sort each one

For every item, decide which bucket it belongs in:

- **A new task** - it is concrete work with a clear outcome. This is most follow-ups.
- **A concern** - it is a risk or an open question, not yet a defined change. It needs a
  human decision before it becomes a task.
- **A learning** - the item is really "the agent got this wrong in a way that could
  recur." That belongs in the memory inbox for `/learn-and-improve`, not the task queue
  (see Step 4).

## Step 3: Route tasks and concerns

**For each new task**, write a `tasks/todo/` file using the `/create-task` shape - a real
goal, real acceptance criteria, and a Notes line pointing back at the task that spawned it
(`Follows up from: tasks/done/<file>.md`). Keep it small and specific.

**For each concern**, append a dated bullet to `project/CONCERNS.md` (create it if absent):

```markdown
- <YYYY-MM-DD> [from <task-slug>]: <the risk or open question>. Suggested resolution: <your recommendation>.
```

Always include a recommendation. A concern with a suggested answer is a decision the human
can make in ten seconds; a bare worry is homework.

## Step 4: Route learnings

If an item is "the agent made a mistake that could recur," write it to the memory inbox
instead - one file per learning, so `/learn-and-improve` can find the pattern later:

```markdown
memory/learnings/inbox/<YYYY-MM-DD-HHMMSS>-<slug>.md
```

```markdown
---
date: <YYYY-MM-DD HH:MM:SS>
target_skill: /start-task | /create-task | /loop | /extract-followups | /learn-and-improve
category: pattern | skill-design | input-quality
---

<One or two sentences: what went wrong and the rule that would prevent it next time.>
```

## Step 5: Report

List what you routed and where: N tasks into `tasks/todo/`, M concerns into
`project/CONCERNS.md`, L learnings into the inbox. Then hand back to `/loop`.

## Why the loop needs a drain

A workflow that only ever consumes tasks and never files new ones is a dead end - it runs
until the initial list empties and stops improving. Routing discoveries back into the
queue is what makes this a loop rather than a line. The learnings branch is what makes it
a loop that gets *better*: mistakes become inputs to `/learn-and-improve`.
