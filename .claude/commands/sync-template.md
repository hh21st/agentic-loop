---
description: Pull the template's improvements into this repo without wiping local divergence - review each upstream change, adopt or skip it, and record the decision so it is never re-offered.
---

# /sync-template - take what is worth taking

These skills came from a template, and that template keeps improving - partly because
`/learn-and-improve` Step 6 sends fixes back to it from projects like this one. This skill
brings those improvements down.

The reason it is not `git pull` or a file copy: **this repo has diverged on purpose.** Local
edits fit local constraints - a shell that mangles commands, a step your team gates
differently - and a blind copy silently deletes them. Worse, it deletes them invisibly, since
the copy looks like a clean success. So this skill reviews the template's changes one at a
time and applies only the ones worth having here.

Run it when the conductor says the bookmark is behind, when you know the template moved, or
every few weeks.

## Step 1: Read the bookmark

`project/TEMPLATE-SYNC.md` records the upstream repo, which paths it owns, the commit this
repo last synced to, and a ledger of every template commit already decided on.

If the file does not exist, this repo has never synced. Create it from the shape in Step 6,
set **Synced to** to the template commit this repo was originally copied from (if that is
unknowable, use the template's first commit and expect the first run to review a lot), and
say so in your report.

If no upstream is recorded and none can be determined, stop and ask. Do not guess a repo URL.

## Step 2: Fetch, and work from a clone you can trust

Get the template's objects into reach. Preferred, because it makes git's own merge machinery
available: add the template once as a second remote in **this** repo and fetch it.

```bash
git remote add template <upstream-url>     # once; skip if already present
git fetch template
```

The histories are unrelated (this repo copied files rather than forking), which is fine -
fetching brings the blobs in, and that is all a three-way merge needs.

If you instead work from a local clone of the template, **fetch it and verify it is current
and clean before reading a single file** (`git fetch && git status --short` empty, no commits
between `origin/<branch>` and its HEAD). A stale or dirty clone is the one failure mode that
looks exactly like success: you diff against an old working tree, conclude nothing changed,
and commit on top of a base that no longer exists. When in doubt, clone fresh into a scratch
directory - a fresh clone cannot be stale, and it costs seconds.

## Step 3: List what changed, and skip what is already yours

Ask the template what it did since the bookmark, restricted to the paths it owns:

```bash
git log --oneline <bookmark>..template/<branch> -- <owned paths>
git diff <bookmark>..template/<branch> -- <owned paths>
```

Before reviewing, subtract two sets:

- **Commits already in the ledger.** A `skipped` entry is a decision, not an oversight - do
  not re-offer it. If you think a past skip was wrong, say so explicitly rather than quietly
  re-proposing it.
- **Commits that originated here.** `/learn-and-improve` Step 6 pushes this project's
  portable fixes upstream, so some incoming changes are your own edits coming home. The
  ledger marks them `ours`. Applying one is harmless but noisy; applying a *reworded* version
  of one silently reverts your local wording. Recognise them and pass.

If nothing survives, say "template has nothing new for us" and stop.

## Step 4: Judge each change on its merits

For each remaining commit, read its diff and decide. This is the whole point of the skill -
the template is not automatically right about your project.

- **adopt** - a general improvement with nothing project-specific about it. Most changes.
- **adapt** - the intent is right but the letter is wrong here: it hardcodes an assumption
  your stack breaks, or it rewrites a passage you deliberately localised. Take the intent,
  keep your specifics, and note in the ledger that your copy differs on purpose.
- **skip** - genuinely irrelevant (guidance for a language you do not use, a workflow you do
  not run). Record **why**, in one line. That line is what stops a future run relitigating it.
- **ours** - your own upstreamed fix returning. Pass.

Apply an adopted change as a patch, not a copy, so local edits elsewhere in the file survive:

```bash
git diff <bookmark>..template/<branch> -- <path> | git apply --3way
```

If it applies cleanly, done. If it conflicts, resolve by reading both sides and asking which
side each hunk's reason belongs to - **template-side reasons win on portable rules, local-side
reasons win on anything naming your stack.** That is the same local/portable split
`/learn-and-improve` Step 3 uses, and it is the whole basis for deciding a conflict. Never
resolve a conflict by taking a whole file.

## Step 5: Verify before you believe it

An adopted change to a skill is a change to how every future task gets built, and nothing
will fail loudly if you got it wrong.

- Re-read each merged file end to end. A three-way merge can apply every hunk and still
  produce a step that contradicts itself two paragraphs later.
- Check the file still means something here: a rule referring to a tool this project does not
  use is a merge that succeeded mechanically and failed in substance.
- Confirm you did not lose local content - diff against `HEAD` and account for every removed
  line. Removals are what a bad sync produces, and they are the easy thing to miss when
  reading an otherwise plausible diff.
- If a skill's frontmatter, step numbering, or a documented file shape changed, grep this repo
  for anything documenting the old shape (`project/CONTEXT.md`, `memory/learnings/README.md`,
  any loop overview doc) and update it too.

## Step 6: Move the bookmark and record the reasoning

Update `project/TEMPLATE-SYNC.md`: set **Synced to** to the template commit you reviewed up
to, and append one ledger line per commit decided.

```markdown
# Template sync

**Upstream**: <url> (branch `<branch>`)
**Git remote**: `template`
**Synced to**: `<sha>` (<YYYY-MM-DD>)

## What the template owns

| Path | Rule |
|------|------|
| `.claude/commands/*.md` | tracked - kept identical unless a ledger line says otherwise |
| `memory/learnings/README.md` | tracked |
| `project/CONTEXT.md` | structure only - its headings come from the template, its content is ours |
| the template's own README / ADAPTING | not tracked |

## Ledger

Newest first. A `skipped` line is a decision and is never re-offered.

- `<sha>` <date> - **adopted** - <commit subject>
- `<sha>` <date> - **adapted** - <subject> - kept <the local specific> because <reason>
- `<sha>` <date> - **skipped** - <subject> - <why it does not apply here>
- `<sha>` <date> - **ours** - <subject> - upstreamed from this repo by /learn-and-improve
```

Advance **Synced to** past skipped commits too. The ledger, not the pointer, is what remembers
the skip - that is why the pointer is safe to move and why a bare watermark is not enough on
its own.

Commit the merged files and the updated bookmark **together**, with the same index discipline
as `/start-task` Step 7 (`git diff --staged --stat`, `git commit --only <paths>`, read back
with `git log -1`). One commit, so the history shows which upstream changes arrived and which
were refused with what reasoning.

## Step 7: Report

Say: reviewed N template commits, adopted A, adapted D, skipped S, recognised O as ours, and
name the new bookmark. Call out anything you adapted, since that is where this repo now
knowingly differs from the template. Then hand back to `/loop`.

## Why a ledger and not just a pointer

A single "synced to" SHA cannot express a refusal. Skip a template commit and you must either
advance past it - losing any record of why your file differs, so the next reader assumes it is
an accident - or stay behind it and be re-offered it forever. Neither is a decision you can
build on. Writing down what you refused and why turns divergence from drift into a documented
position, and it means this skill gets *cheaper* each run instead of re-arguing settled ground.

Paired with `/learn-and-improve` Step 6, which sends portable fixes the other way, the two
close a round trip: lessons this project learns reach the template, improvements the template
gains reach this project, and neither direction overwrites what the other side knows.
