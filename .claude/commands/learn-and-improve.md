---
description: Read the accumulated learnings, find recurring mistakes, and propose edits to the skills' own instructions. You approve each change before it lands.
---

# /learn-and-improve - the meta-loop

The other four skills do the work. This one makes the work get better. It reads the
mistakes the loop recorded, finds the ones that keep happening, and proposes edits to the
skill files themselves so the same mistake is harder to make next time. A human approves
every edit - the system proposes, you dispose.

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
`target_skill`. Within each group, look for the signal that justifies a change:

- **A recurrence**: two or more learnings describing the same class of mistake. This is
  the strongest signal - the skill has a real gap, not a one-off.
- **A single sharp insight**: one learning that names a concrete, fixable weakness in a
  skill's instructions.

One-off vague notes with no clear fix: leave them in the inbox for now. A pattern needs
evidence, and a fix needs to be specific enough to write down.

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

## Step 6: Report

Summarise: processed N learnings, proposed P edits, applied A (with the user's approval),
deferred D. Then hand back to `/loop`.

## Why this is the capstone

Any agent can follow instructions. The leverage is in a loop that turns its own corrected
mistakes into better instructions, compounding over time, without ever taking the human
out of the decision. Compounding improvement beats one-time perfection: you do not need
the skills to be right on day one, only to get less wrong every week. That is the whole
wager of a self-improving system.
