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

**If the task prescribes a specific dependency option, confirming that the member exists
is not verification - read what it does in the shipped implementation.** Especially for
anything time-, retry-, or cancellation-related, and especially when an existing code path
already gives that same setting a meaning users depend on. Two SDKs offered the same
`timeout` option where one armed an abort timer at request start and never cleared it (a
hard whole-stream deadline) and the other cleared it once headers arrived; the setting
silently meant two different things, and the difference would have cut healthy replies into
billed, unrefundable partials.

## Step 3: Implement

Make the change. Keep the diff scoped to what the task asks for - no drive-by refactors,
no extra features. If you discover adjacent work that should happen, do not do it now;
it becomes a follow-up in Step 6.

**Every comment and commit-message sentence is a claim you are asserting, and it will be
trusted.** A confidently wrong comment is worse than none - it gets believed and
propagated, and reviewers burn rounds disproving it. Before you write one:

- If it asserts something **measurable** - a size, a timing, a type-system consequence,
  what another module does - measure or read it right then, and note where the number came
  from. ("~7.7MB" turned out to be the byte count of a grep's *output*; the module was ~1MB.)
- If it claims a change **closes an attack class** - "the only thing preventing", "cannot",
  "impossible" - enumerate every writer of that resource first: permissions and grants,
  other routes, other services. If any path remains open, scope the claim to the one you
  closed and record the rest as a residual. An over-claiming comment tells the next reader
  the door is locked.
- If it explains **why**, state only the part actually load-bearing for the code beneath it.
- Re-check these after any refactor. The refactor is what most often turns a true comment
  false.

## Step 4: Write and run tests

For any non-trivial logic you added or changed, add a test that would fail without your
change and passes with it. Run the project's test command (see `project/CONTEXT.md`). If
tests fail, fix the code - not the test - until they pass. If the change is pure config,
copy, or docs with no logic to exercise, say so explicitly and skip; do not fabricate a
hollow test.

### Then prove each test can fail

A passing test is not evidence; a test you have watched go red is. For every test you
added, and every guard or branch you claim is covered, break the thing it tests and confirm
that specific test fails:

- **A test written for a fix**: revert the fix, run the named test, confirm it FAILS,
  restore. A test written just after a fix is the most likely to be vacuous - you know the
  mechanism and unconsciously write the setup that makes it pass, which then retires
  suspicion.
- **A guard or branch**: mutate it to the NARROW form rather than deleting it (`!(x > 0)` →
  `x === 0`, `!= null` → `!== undefined`, an untyped catch-all → a typed one). Deletion
  fails loudly and proves little; narrowing is the realistic regression a later
  "simplification" introduces, and the one a single-input test cannot see. If a comment
  says why the guard is deliberately broad, drive a case for each reason it names.
- **Each assertion**: name the specific wrong implementation it rules out, then check that
  a *missing* or *absent* value does not also satisfy it. An "expect no error" passes when
  zero rows were touched; a `not.toBe(x)` passes when the left side is undefined because a
  read failed. Negated comparisons and absence-shaped successes are where a hollow
  assertion hides. An assertion whose discriminating power rests on a precondition the test
  never checks is the same bug one level up.

Restore mutations through version control, never through a copy the same script made:
commit or stash first, mutate with ABSOLUTE paths, restore with `git checkout -- <abs-path>`,
and assert `git status --short` is empty after each one. A mutation check that cannot prove
it cleaned up can land the very mutation it was meant to prove was dangerous.

### Test doubles must mirror the real contract

Read the dependency's actual behaviour from its source - not the contract that makes the
test convenient. A stub that short-circuits on an "already cancelled" flag certifies
behaviour a dependency does not have if the real one only ever attaches a listener; the
test then passes with the guard deleted. And when the real dependency delivers results
through **events or callbacks, the double must deliver them asynchronously too**. A
recorder fake whose `stop()` fires its data and stop handlers synchronously removes every
interleaving the production code exists to handle - which made two real bugs untestable and
therefore invisible behind seventeen passing tests.

**Assert at the layer the criterion names.** When an acceptance criterion is phrased in
observable terms - "renders", "shows", "is disabled", "returns 403", "writes the row" -
assert on that observable output after the real interaction, not on an intermediate value.
Internal state can report `failed` while the component that renders the warning is
unmounted in the same tick. Internal-state assertions are for invariants no one can observe.

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

Note what Pass 1 structurally cannot catch: a diff can satisfy every stated criterion and
still break an invariant the spec never mentions. A change that added a new failure path
inside a metered window read as fully compliant to the spec pass, because the spec said
nothing about metering. Only Pass 2 can see that class of defect - never let a PASS on
Pass 1 shorten Pass 2.

## Step 6: Resolve the review

- **CRITICAL / IMPORTANT** findings: fix them now, then repeat Step 5 on the new diff.
- **MINOR** findings: do not expand this task to fix them. Hand them to `/extract-followups`
  so they become their own queued tasks. Chasing every minor point inside one task is how
  a one-line change becomes a five-round slog.
- Loop Steps 5-6 until Pass 1 is PASS and Pass 2 is `complete`.
- **If a finding is revenue-, quota-, or abuse-class, "fixed the finding" is not the exit
  condition - "the reviewer can no longer attack it" is.** Keep re-submitting to the SAME
  reviewer, context intact, until they explicitly fail to trace any exploit. One metering
  design passed a spec review and then needed four successive rounds with one reviewer,
  each fix revealing the next exploit: client-controlled idempotency key → replay farm →
  regenerating a completed exchange → predicate drift.

A fix round is the highest-risk moment in the task, not the safe wind-down. Two failure
modes recur:

- **New false claims.** A correction feels authoritative and gets less scrutiny than the
  original, and you are writing confidently about ground you were just shown you did not
  know. Treat every sentence written during a fix round as a fresh unverified claim and
  hold it to the Step 3 standard: if it points at a file, section, or recorded decision,
  open it and confirm the entry is actually there. Prefer claims that are cheap to verify
  ("the check below reads the finish reason, never the content") over ones that merely
  sound authoritative ("this is recorded in CONCERNS.md") - the second silently rots.
- **A fix that moves the failure instead of removing it.** Adding a `throw` to code that
  previously could not throw changes which side of a consume/refund, lock/unlock, or
  open/close boundary a failure lands on - trace where the new throw lands in every caller,
  and move the call rather than the boundary. Adding an `await` to a teardown or
  ownership-releasing path inserts a window in which all shared mutable state can change:
  snapshot what the post-await code needs BEFORE yielding and pass it as parameters, treat
  any shared reference read after an await as belonging to whoever owns it now, and ask
  "what else can start during this await?" Closing a minor cache race this way opened a far
  worse one, filing one client's speech under another client's record.

Then prove the fix with a test that asserts the bad side effect did NOT happen - and
falsify that test per Step 4 before you believe it.

## Step 7: Close out

- Set `**Status**: done`, add a one-line Progress note, and move the file to `tasks/done/`.
- Commit the change with a message naming the task. Commit mechanics, each of which has
  already cost a session:
  - **Check the index is yours before committing.** Another agent may be working in the
    same clone with its own work staged. Run `git diff --staged --stat`, and commit with
    `git commit --only <your explicit paths>` so the index's other contents cannot ride
    along. Never `--amend` unless the staged set is verified to be yours alone - amend
    commits the whole index, and one blind `--amend` to fix a typo swallowed another
    session's 19 half-finished files. If a commit did capture too much,
    `git reset --soft HEAD~1` is the safe undo: it preserves both index and working tree,
    so nothing is lost.
  - **Use the shell you are actually in.** An environment offering more than one shell
    offers more than one quoting syntax, and they are not interchangeable - a here-string
    form pasted into a POSIX shell leaves its delimiters literally in the subject line.
    Prefer a form that cannot be re-parsed: `git commit -F <file>`, or the heredoc your
    actual shell documents. See `project/CONTEXT.md` for this project's shell traps.
  - **Read the commit back.** `git log -1` and `git status` after committing. A malformed
    message and a swallowed index are both invisible from the exit code, which is 0 either
    way.
- Run `/extract-followups` to capture anything the task surfaced. It exits right away when
  there is nothing to record, so run it every time rather than judging in advance whether
  there is something worth capturing. Then run `/loop` to see what is next.

## The one rule

Do not report a task done on the strength of "it should work." Done means an independent
pass read the actual diff against the actual spec, and a test actually ran green **after
you watched it run red**. The value of this loop is that "done" is earned, not asserted.
