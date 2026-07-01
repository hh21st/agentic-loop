# Adapting the loop

The five skills are short on purpose. They are meant to be read and edited, not treated as
a fixed framework. Here is how to make them yours.

## 1. Fill in the one placeholder

[`project/CONTEXT.md`](project/CONTEXT.md) is the only file the skills genuinely need you
to edit. It tells them what codebase they are driving and how to run its tests. Everything
else works out of the box.

## 2. Point the skills at your test command

`/start-task` runs tests in its verify step. It reads the test command from
`project/CONTEXT.md`. Put the real command there (for example `npm test`, `pytest`, or
`go test ./...`) and the verify step will use it. If your project has no tests yet, say so
in CONTEXT.md and the skill will skip the test run and lean harder on the review pass.

## 3. Wire it to your agent

The skills live in `.claude/commands/` so they are directly invocable as slash-commands in
Claude Code (`/loop`, `/create-task`, and so on). To use a different coding agent:

- Keep the skill bodies as they are. They are plain markdown instructions.
- Change how you invoke them. Most agents can be handed a markdown file as a prompt, or you
  can paste the relevant skill body into a fresh session.
- The one capability the verify step wants is a **fresh, independent reviewer**: a context
  that did not write the code and does not see the implementer's reasoning. In Claude Code
  that is a subagent. In a plainer setup it is a new session handed only the task file and
  the diff. Preserve that independence however your tool allows, because it is what makes
  the review catch the bugs the writer cannot see.

## 4. Tune the skills to your taste

Common changes people make:

- **Task shape**: edit the template in `/create-task` to add fields your team cares about
  (owner, estimate, linked issue).
- **Review strictness**: adjust the labels and gates in `/start-task` Step 5. Some teams
  add a security-focused third pass.
- **When to learn**: change the threshold `/loop` uses to suggest `/learn-and-improve` (it
  defaults to 5 unprocessed learnings).

## 5. Let the loop tune itself

The intended way to evolve these skills over time is the loop's own meta-step. As you run
real work through it, `/extract-followups` records the mistakes it makes into
`memory/learnings/inbox/`, and `/learn-and-improve` proposes edits to the skill files based
on the recurring ones. You approve each edit. Over a few weeks the skills drift toward
fitting your project, driven by evidence rather than guesswork.

That is the wager of the whole template: you do not need it to be perfect on day one, only
to get a little less wrong each week.
