# Project context

> This is the one placeholder the skills read to know what they are working on.
> Replace every `<<< ... >>>` below with your project's real details, then delete this line.

## What this project is

<<< One paragraph: what the software does, who uses it, and the shape of it (web app, CLI,
library, service). Enough that an agent picking up a task has the gist without reading the
whole codebase. >>>

## Where the code lives

<<< The main directories and what is in them. For example:
- `src/` - application code
- `src/api/` - HTTP handlers
- `tests/` - test suite
>>>

## How to run the tests

<<< The exact command that runs the test suite, for example `npm test` or `pytest -q`.
`/start-task` uses this in its verify step. If there are no tests yet, write "none yet"
and the skill will skip the test run. >>>

## Conventions worth knowing

<<< Anything an agent would otherwise rediscover the hard way: the language and framework,
the formatter or linter, naming conventions, "we always do X this way", things that are
off-limits. Keep it short and high-signal. >>>

## Upstream template for the loop skills

<<< If you took `.claude/commands/` from a template — agentic-loop or your own fork — fill in
[`TEMPLATE-SYNC.md`](TEMPLATE-SYNC.md) instead of writing the details here. That file is the
one place `/sync-template` and `/learn-and-improve` Step 6 both read, and keeping it the only
copy is what stops two docs disagreeing about which commit you are on.

Say here only what a task author needs to know: that the skills track a template, so anything
stack-specific belongs in this file rather than inside a skill — see Toolchain traps below.

Delete this section if this repo *is* the template. >>>

## Toolchain traps

<<< Start this empty and let the loop fill it. This is where a learning filed as
`target_skill: project` lands: a shell that mangles a command, a test helper that deadlocks
the suite, a value the database silently normalizes. Facts about *this* stack that an agent
has already been burned by once.

Keep them here rather than in a skill. The skills are meant to stay portable, and an agent
looking for "why did my commit message come out mangled" will look at the project notes, not
at generic instructions. One bullet each, naming the symptom and the workaround. >>>

## Out of scope

<<< Areas the loop should not touch without asking (generated files, vendored code,
infra). Optional, but useful. >>>
