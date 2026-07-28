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

<<< If you took `.claude/commands/` from a template — agentic-loop or your own fork — record
it here: the repo URL, where it is cloned locally, and the branch. `/learn-and-improve`
Step 6 reads this to offer feeding its portable fixes back, so an improvement you earn here
reaches every project you start from that template later. Without it, each repo re-learns the
same lesson alone.

Say also whether the skill files are meant to stay identical to the template's copies. Keeping
them so makes pulling later template improvements a clean fast-forward, and it is the reason
stack-specific facts belong under Toolchain traps below rather than inside a skill.

Delete this section if this repo *is* the template — Step 6 then has nothing to do. >>>

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
