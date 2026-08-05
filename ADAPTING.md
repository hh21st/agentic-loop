# Adapting the loop

The six skills are short on purpose. They are meant to be read and edited, not treated as
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
Claude Code (`/next`, `/create-task`, and so on). To use a different coding agent:

- Keep the skill bodies as they are. They are plain markdown instructions.
- Change how you invoke them. Most agents can be handed a markdown file as a prompt, or you
  can paste the relevant skill body into a fresh session.
- The one capability the verify step wants is a **fresh, independent reviewer**: a context
  that did not write the code and does not see the implementer's reasoning. In Claude Code
  that is a subagent. In a plainer setup it is a new session handed only the task file and
  the diff. Preserve that independence however your tool allows, because it is what makes
  the review catch the bugs the writer cannot see.
- If your harness gates subagent or multi-agent tool calls behind an explicit per-call user
  request, invoking `/start-task` is that request - Step 5 says so directly so the agent does
  not need to re-derive consent and does not quietly fall back to self-review because a
  generic gate misread "the user ran a skill that dispatches reviewers" as "the user did not
  ask for this." If your harness cannot resolve that from having been invoked, have the skill
  ask once, not skip silently.

## 4. Tune the skills to your taste

Common changes people make:

- **Task shape**: edit the template in `/create-task` to add fields your team cares about
  (owner, estimate, linked issue).
- **Review strictness**: adjust the labels and gates in `/start-task` Step 5. Some teams
  add a security-focused third pass.
- **When to learn**: change the threshold `/next` uses to suggest `/learn-and-improve` (it
  defaults to 5 unprocessed learnings).

## 5. Let the loop tune itself

The intended way to evolve these skills over time is the loop's own meta-step. As you run
real work through it, `/extract-followups` records the mistakes it makes into
`memory/learnings/inbox/`, and `/learn-and-improve` proposes edits to the skill files based
on the recurring ones. You approve each edit. Over a few weeks the skills drift toward
fitting your project, driven by evidence rather than guesswork.

Not all of that drift is yours to keep, though. Some corrections are about your stack — a
shell that mangles a command, a test helper that deadlocks — and belong in
`project/CONTEXT.md` under Toolchain traps, which is where `/extract-followups` routes a
learning filed as `target_skill: project`. Others are about how an agent fails, and are true
of any project this loop is pointed at: a test that could not fail, a claim written without
checking, a review that stopped one round too early. Those are worth more than one repo.

So record where you got these skills in `project/CONTEXT.md`, and `/learn-and-improve` Step 6
will offer to apply the portable half back there too — diffing before it copies, checking the
docs it contradicts, and never pushing your fork's public history without asking. Keeping the
skill files identical to the template's makes pulling later improvements a clean fast-forward;
it is also what forces the useful discipline of writing a rule generally instead of in your
own stack's vocabulary.

That is the wager of the whole template: you do not need it to be perfect on day one, only
to get a little less wrong each week — and with the portable half travelling upstream, each
project starts less wrong than the last.

## 6. Stay current without losing your changes

The other half of that round trip is `/sync-template`, which pulls later improvements to these
skills down into your copy.

The reason it is a skill and not `git pull` or a file copy: by the time the template moves, you
have edited these files too. Your `/start-task` gates reviews the way your team wants; your
`project/CONTEXT.md` is entirely yours. Copying the template's version over the top deletes
that, and deletes it *invisibly* — the copy reports success, and the loss only shows up weeks
later when a step you relied on is quietly gone. So each incoming change is judged one at a
time and either adopted, adapted to fit what you have, or refused.

Set it up once by filling in [`project/TEMPLATE-SYNC.md`](project/TEMPLATE-SYNC.md): the
upstream URL, which paths the template is allowed to touch, and the commit you cloned from.
Then run `/sync-template` whenever `/next` reports the bookmark is behind.

Two details that matter more than they look:

- **Record refusals, not just position.** A single "synced to" commit cannot express "I saw
  that change and did not want it." Without the ledger you either advance past a refused change
  and lose all trace of why your file differs — so the next reader treats a deliberate choice
  as an accident — or you sit behind it and get asked about it on every run. The ledger is what
  makes divergence a documented position instead of drift.
- **Write down deliberate differences** under **Known local divergence**. An intentional
  difference that is not recorded looks identical to an accidental one, and the next sync will
  helpfully "fix" it back.

If you keep the skill files identical to the template's, syncing stays close to a fast-forward
and the two directions barely interact. That is worth aiming for — and it is exactly why
stack-specific facts belong in `project/CONTEXT.md` rather than edited into a skill.
