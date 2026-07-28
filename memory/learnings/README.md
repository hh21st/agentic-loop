# memory/learnings

This is the loop's memory of its own mistakes. Each file in `inbox/` is one recorded
learning: a place the agent got something wrong, and the rule that would prevent it next
time. `/extract-followups` writes them; `/learn-and-improve` drains them.

## Why one file per learning

The directory itself is the queue. One learning per file means two agents recording
lessons at the same time never collide: each writes its own file, and there is no shared
list to fight over. The folder listing is the count. This is the simplest conflict-free
queue there is, and it scales from one agent to many without changing anything.

## The shape of a learning

```markdown
---
date: 2026-01-15 14:30:00
target_skill: /start-task
category: verification-gap
---

The review passed a bug because it ran in the same context that wrote the code and inherited
its assumptions. Rule: always review from a fresh context that never saw the implementation.
```

- **date** - when it was recorded.
- **target_skill** - which skill's instructions this is about, or `project` when the fix is
  a fact the agent needed about this codebase or toolchain rather than a change to a skill.
  Those land in `project/CONTEXT.md`, which every skill already reads.
- **category** - what let the mistake through: `verification-gap` (a test, review, or check
  passed over it - the most common and most valuable kind), `false-claim` (a comment, commit
  message, or report asserted something untrue), `environment` (a shell, tool, or platform
  behaved other than assumed), `intake-gap` (the task was underspecified going in), or
  `skill-design` (a structural hole in a skill's instructions).

Pick the category by what would have caught the mistake, not by which part of the code it
touched. If everything you file lands in one category the field has stopped grouping
anything, and `/learn-and-improve` should be told to propose better values.

## The lifecycle

```
inbox/            written by /extract-followups, waiting to be reviewed
   │
   ▼  /learn-and-improve finds the pattern, proposes a skill edit, you approve
processed/        moved here once acted on, kept as the audit trail
```

A learning in `processed/` is not deleted: paired with the commit that changed a skill, it
is the record of the system improving itself, and the evidence for whether the change held.
