---
description: Implement a task, then verify it with a fresh-eyes review that catches where the agent is confidently wrong, plus tests. Move it to done.
---

# /start-task - implement, then verify

Take a task from the queue, implement it, and do not call it done until an independent
review and a test run agree that it is. Verification lives **inside** this skill, not as a
separate step you might skip. The whole reason this skill earns trust is that it assumes
the first pass is probably wrong somewhere and goes looking.

Argument (`$ARGUMENTS`): a task file path, or empty to auto-pick.

## Step 1: Pick and claim the task

- If given a path, use it. Otherwise pick the highest-priority file in `tasks/todo/`.
- Move it to `tasks/doing/` and set `**Status**: doing`. This marks it as owned so a
  second agent will not grab the same task. Record the starting commit:
  `git rev-parse HEAD` - you will diff against this later.

## Step 2: Plan the change

Read the task in full. Read the files it names. If the plan in the task no longer matches
the code (paths moved, a function was renamed), STOP and fix the task file first - an
out-of-date plan implemented faithfully is still wrong. Then write down, in the task's
Notes, the concrete edits you are about to make.

## Step 3: Implement

Make the change. Keep the diff scoped to what the task asks for - no drive-by refactors,
no extra features. If you discover adjacent work that should happen, do not do it now;
it becomes a follow-up in Step 6.

## Step 4: Write and run tests

For any non-trivial logic you added or changed, add a test that would fail without your
change and passes with it. Run the project's test command (see `project/CONTEXT.md`). If
tests fail, fix the code - not the test - until they pass. If the change is pure config,
copy, or docs with no logic to exercise, say so explicitly and skip; do not fabricate a
hollow test.

## Step 5: Verify with fresh eyes (the part that matters)

Stage your changes (`git add` the files you touched) and get the diff:
`git diff --staged`. Now run a review by someone who did **not** write the code and does
**not** see your reasoning - only the task file and the diff. In an agent that supports
subagents, dispatch a fresh reviewer; otherwise open a clean context and review as a
stranger. Run it in two passes, in order:

**Pass 1 - does it match the spec?** Give the reviewer only the task file and the diff.
Ask: does the diff satisfy every acceptance criterion? Is anything missing? Is anything
here that the task did not ask for? Output PASS or FAIL with the specific gap. If FAIL,
fix and re-review - do not continue past a failing spec check.

**Pass 2 - is it any good?** Only after Pass 1 passes. Ask a fresh reviewer: correctness
(bugs, missed edge cases, off-by-ones), reuse (does this reinvent something the codebase
already has), and clarity. The reviewer returns concrete findings labelled CRITICAL /
IMPORTANT / MINOR, and a status of `complete` or `needs-work`.

Why two independent passes instead of re-reading your own work: an agent that just wrote
code is the worst judge of it. It is primed to see what it intended, not what it typed -
this is exactly where a model is confidently wrong. A reviewer with no memory of the
intent reads the diff as written and catches the gap between the two. Splitting spec from
quality keeps each pass honest: "it does the wrong thing well" and "it does the right
thing badly" are different failures and blur together when judged at once.

## Step 6: Resolve the review

- **CRITICAL / IMPORTANT** findings: fix them now, then repeat Step 5 on the new diff.
- **MINOR** findings: do not expand this task to fix them. Hand them to `/extract-followups`
  so they become their own queued tasks. Chasing every minor point inside one task is how
  a one-line change becomes a five-round slog.
- Loop Steps 5-6 until Pass 1 is PASS and Pass 2 is `complete`.

## Step 7: Close out

- Set `**Status**: done`, add a one-line Progress note, and move the file to `tasks/done/`.
- Commit the change with a message naming the task.
- Run `/extract-followups` to capture anything the task surfaced. It exits right away when
  there is nothing to record, so run it every time rather than judging in advance whether
  there is something worth capturing. Then run `/loop` to see what is next.

## The one rule

Do not report a task done on the strength of "it should work." Done means an independent
pass read the actual diff against the actual spec and a test actually ran green. The value
of this loop is that "done" is earned, not asserted.
