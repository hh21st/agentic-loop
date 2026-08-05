---
description: Turn a recurring job into a new skill file that matches this loop's own conventions, instead of hand-writing one from a blank page.
---

# /create-skill - grow the loop

The six core skills are not the ceiling. When a job keeps coming up - in this repo or in
one cloned from it - that deserves its own skill rather than being redone from scratch or
folded awkwardly into an existing one, this is how you add it without drifting from the
conventions the rest of the loop already follows.

Argument (`$ARGUMENTS`): what the new skill should do, or ask the user for one.

## Step 1: Understand the job before you name it

- What triggers this skill, and what does "done" look like when it finishes?
- Read two or three files in `.claude/commands/` first, even ones you already know well -
  the point is to absorb the *current* shape, not a remembered one, since these files drift
  as `/learn-and-improve` edits them.
- Check it is not already covered. A near-duplicate of an existing skill is a sign that skill
  needs a step added, not that a sibling needs to exist.

## Step 2: Decide the format

- **The job is "how do we install and launch this project"**: that is not this skill's job.
  Claude Code already ships `/run-skill-generator` for exactly that case - it captures the
  recipe and scaffolds it under `.claude/skills/run-<unit>/` on its own. Point the user at it
  directly rather than reinventing it here.
- **No bundled files** (the common case for everything else - a markdown file of
  instructions is the whole skill): write `.claude/commands/<name>.md`, the same flat format
  every skill in this loop uses.
- **Needs bundled files** - a script it runs, a state or bookmark file it reads and writes
  across runs, reference docs too long to inline: use a folder-based Agent Skill instead,
  `.claude/skills/<name>/SKILL.md` with its files alongside it. Scaffold the folder and
  frontmatter, then apply Step 3 on top of it - the mechanical shape Claude's Skills format
  wants is not the same thing as this loop's conventions, and those are what make the result
  feel like it belongs here.

## Step 3: Write it in this loop's shape

Whichever format Step 2 chose, the body follows the same conventions as the six skills
already in `.claude/commands/`:

- Frontmatter: a single `description:` field, one sentence, in the same terse voice as the
  others - it is what shows up in the skill list, so it has to tell a reader whether this is
  the skill they want without opening the file.
- `# /<name> - <short tagline>`, then numbered `## Step N: <verb phrase>` sections. Each step
  is something to *do*, not background - keep the "why" out of the steps and give it its own
  section at the end.
- End the last step before the closing rationale with the same step every other skill in
  this loop now has:

```markdown
## Step <N>: Log anything that went sideways

This is about running `/<name>` itself. Did something about it go differently than these
instructions assumed, or did you find a better way? Write it down now:

memory/learnings/inbox/<YYYY-MM-DD-HHMMSS>-<slug>.md
---
date: <YYYY-MM-DD HH:MM:SS>
target_skill: /<name> | project
category: verification-gap | false-claim | environment | intake-gap | skill-design
---

<One or two sentences: what happened, and the rule or better way that would prevent it next time.>
```

- Close with a `## Why <...>` section: one or two paragraphs on why this needed to be its
  own skill rather than a step bolted onto an existing one. That is the passage a future
  `/learn-and-improve` or `/sync-template` run leans on when judging whether an edit still
  fits the skill's purpose.

## Step 4: Wire it into the loop

- If `/next` should ever recommend this skill, add a row to its Step 3 table (or say
  explicitly why it should not be one - not every skill is meant to be auto-recommended;
  `/extract-followups` and this one both are not).
- If another skill should call this one, or hand off to it, add that line to the caller's
  own file - do not leave the connection only in this new file, where the caller's reader
  will never see it.
- Add it to `README.md` under "Adding a new skill" (see `ADAPTING.md`), not to "The six
  skills" table - that table names the core intake-build-verify-learn-sync cycle, and this
  is a tool for growing that cycle, not a step inside it.

## Step 5: Self-check before handing off

Read the new file as a fresh agent who has never seen this conversation. Could you run it
without asking a question? Does every step name a real path or command? Does it end with
the logging step in the exact shape above, so `/learn-and-improve` can read it the same way
it reads the other six?

## Step 6: Log anything that went sideways

This is about running `/create-skill` itself - a convention in Step 3 that did not fit the
job, a format decision in Step 2 that turned out wrong once you started writing. Write it
down now:

```markdown
memory/learnings/inbox/<YYYY-MM-DD-HHMMSS>-<slug>.md
---
date: <YYYY-MM-DD HH:MM:SS>
target_skill: /create-skill | project
category: verification-gap | false-claim | environment | intake-gap | skill-design
---

<One or two sentences: what happened, and the rule or better way that would prevent it next time.>
```

Most runs have nothing to record - do not manufacture one.

## Why this is not just "write a markdown file"

A skill written from a blank page reflects whatever the author remembers of the conventions
at that moment, and that memory is exactly what drifts - a missing closing step, a
frontmatter field in the wrong shape, a "why" section that never got written because nothing
forced it. `/learn-and-improve` keeps the six core skills consistent with each other over
time by editing them under review; this is the same discipline applied at the moment a new
one is born, so it starts consistent instead of needing to be caught up later. And because
`.claude/commands/*.md` is a tracked path in `project/TEMPLATE-SYNC.md`, a skill created this
way is already shaped to travel cleanly to - or arrive cleanly from - every other project
built on this template.
