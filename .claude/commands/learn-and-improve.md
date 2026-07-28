---
description: Read the accumulated learnings, find recurring mistakes, and propose edits to the skills' own instructions - offering to feed the portable ones back to the upstream template. You approve each change before it lands.
---

# /learn-and-improve - the meta-loop

The other four skills do the work. This one makes the work get better. It reads the
mistakes the loop recorded, finds the ones that keep happening, and proposes edits to the
skill files themselves so the same mistake is harder to make next time. A human approves
every edit - the system proposes, you dispose.

Fixes that are not specific to this project get offered back to the template these skills
came from, so the improvement outlives this repo.

Run it when the inbox has built up (the conductor suggests it at 5+), or on a schedule.

## Step 1: Read the inbox

Each file in `memory/learnings/inbox/` is one recorded mistake or insight, written by
`/extract-followups` (or by you). List them:

```bash
ls memory/learnings/inbox/*.md 2>/dev/null
```

If the inbox is empty, say "nothing to learn from yet" and stop.

## Step 2: Group and find patterns

Read each file's frontmatter (`target_skill`, `category`) and body. Group by
`target_skill`, then by `category` within it. Within each group, look for the signal that
justifies a change:

- **A recurrence**: two or more learnings describing the same class of mistake. This is
  the strongest signal - the skill has a real gap, not a one-off.
- **A single sharp insight**: one learning that names a concrete, fixable weakness in a
  skill's instructions.

One-off vague notes with no clear fix: leave them in the inbox for now. A pattern needs
evidence, and a fix needs to be specific enough to write down.

Trust the bodies over the frontmatter. A learning is filed by whoever hit it, in the moment,
under whichever label seemed closest - so read all of them before grouping, and regroup
across the labels when the bodies say the same thing. Two signals to act on directly:

- **A skewed distribution is itself a finding.** If almost every learning names one skill,
  or one step inside it, that step is underspecified relative to what it claims to do - and
  the fix is to give it a procedure, not another warning.
- **A field that takes one value is doing no work.** If nearly everything shares a
  `category`, do not silently ignore it - propose better values as one of your edits to
  `/extract-followups`. The classification exists to save this reading; when it stops, fix
  it rather than working around it.

**Learnings targeting `project`** are not skill gaps - they are facts about this codebase or
toolchain (a shell that mangles a command, a helper that deadlocks, a value the database
normalizes). Propose those as edits to `project/CONTEXT.md`, which every skill already
reads, and say plainly in the proposal that it edits a non-skill file. Resist the pull to
put them in a skill: generic instructions carrying one project's trivia stop being portable,
and a skill is the wrong place to look for them anyway.

## Step 3: Propose edits

For each pattern, draft a concrete edit to the target skill file. Show it as a diff the
user can judge at a glance:

```
### Proposal: /start-task - stop trusting self-review of your own diff

Based on: <2 inbox filenames>
Category: skill-design

Current (Step 5):
> ... open a clean context and review as a stranger.

Proposed:
> ... open a clean context and review as a stranger. Never review from the same context
> that wrote the code - reopen the diff cold, or the review inherits the writer's blind spot.

Why: two learnings recorded the reviewer waving through a bug because it reused the
implementer's context and its assumptions.
```

Keep each proposal small and surgical. You are tuning instructions, not rewriting skills.
Prefer adding a guardrail to the exact step that failed over restructuring the file.

### Give every proposal a destination

Some of what you are about to propose is about *this* project - its stack, its shell, its
database. The rest is about how an agent fails, and is just as true of any project this loop
gets pointed at. Label each proposal:

- **local** - it names a tool, framework, or platform. Goes to `project/CONTEXT.md`, or to a
  skill only if the rule genuinely cannot be stated without the specifics.
- **portable** - it is a failure mode of the agent rather than of the code: a test that could
  not fail, a claim written without checking, a review that stopped too early, an intake that
  guessed. Goes to the skill file - and, if these skills came from an upstream template,
  there too (Step 6).

Write every portable proposal ecosystem-neutral **as you draft it**, not as a cleanup pass
afterwards: state the rule generally, then keep the concrete example but strip the project's
names, versions, and product nouns. Two reasons it has to happen now. A rule stated only in
one stack's vocabulary reads as inapplicable to everyone else, so it gets skipped rather than
adapted. And the human is about to approve an exact wording - generalizing it after that
either silently changes what they approved or means asking twice.

## Step 4: Human approves - this gate is not optional

Present every proposal and wait for the user to accept, reject, or amend each one.

- **Accept** → apply the edit to the skill file.
- **Amend** → apply the user's version.
- **Reject** → leave the skill unchanged.

Never apply an edit to a skill's own instructions without explicit approval. A system that
edits its own operating rules unattended drifts somewhere no one chose. Keeping a human on
the approval step is what makes self-improvement safe rather than a runaway. This is the
one place in the loop where you always stop and ask.

## Step 5: Archive what you processed

For each learning that led to an approved edit (or was consciously dismissed), move it out
of the inbox so it is not reprocessed:

```bash
mkdir -p memory/learnings/processed
git mv memory/learnings/inbox/<file> memory/learnings/processed/<file>
```

Commit the skill edits and the archived learnings together, so the history shows which
mistakes drove which instruction change. That commit is the audit trail of the system
improving itself.

## Step 6: Offer to upstream the portable edits

If `project/CONTEXT.md` records an upstream template for these skills, every **portable**
edit you just applied is also a gap that template still has - it was written from mistakes
its own instructions allowed. Offer it: list the portable edits and ask whether to apply them
there too. If nothing is recorded but the skills plainly came from somewhere, ask once and
write the answer into `CONTEXT.md` so no later drain has to ask again. If this repo **is** the
template, there is no upstream - skip this step.

With approval, for each portable edit:

1. **Diff before copying.** Compare each skill file against the template's copy. Identical
   apart from your new edit means copying the whole file is safe. If the template has
   diverged - someone else improved it, or this repo carries a deliberate local-only change -
   do **not** copy over it: apply your edit to the template's version instead, and report the
   divergence rather than silently resolving it.
2. **Follow the claim into the docs.** If the edit changed something the template also
   documents - a frontmatter field, a directory shape, a promise in its README - grep for it
   and update those too. An instruction that contradicts its own README is worse than the gap
   you set out to fix, and this is the easiest thing in the whole step to miss.
3. **Check for leaks** before committing: grep the template for this project's name,
   dependencies, and platform words. Anything that survives was a local proposal mislabelled
   portable - pull it back out.
4. **Commit in the template separately**, checking that repo's index is yours first
   (`git diff --staged --stat`), exactly as in `/start-task` Step 7. Frame the message as
   feedback from a downstream project and say what the loop actually got wrong, with the
   numbers: the template's history should carry the evidence, not just the edit. That is what
   lets a future reader judge whether the change was warranted.
5. **Ask before pushing.** The template is likely public and shared with people who did not
   ask for your change. Commit locally, say what is waiting, and let the human decide - unless
   they have already told you to push.

Finally, if the skills are meant to stay identical across the two repos, say so in
`CONTEXT.md` and prove it with a diff at the end of every drain. Assuming they are still in
sync is how the next drain quietly clobbers someone else's improvement.

## Step 7: Report

Summarise: processed N learnings, proposed P edits, applied A (with the user's approval),
deferred D, upstreamed U. Then hand back to `/loop`.

## Why this is the capstone

Any agent can follow instructions. The leverage is in a loop that turns its own corrected
mistakes into better instructions, compounding over time, without ever taking the human
out of the decision. Compounding improvement beats one-time perfection: you do not need
the skills to be right on day one, only to get less wrong every week. That is the whole
wager of a self-improving system.

Step 6 is what makes the compounding outlive the project. A mistake corrected here only ever
protects this repo; the same correction pushed to the template protects every repo started
from it after today, including the ones you have not thought of yet. Verification gaps are
the same everywhere because they are properties of how the agent works, not of what it is
working on - so the evidence for a fix is local, but the fix itself usually is not. Learning
that travels upstream is the difference between one project getting better and the way you
build projects getting better.
