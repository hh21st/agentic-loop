# Template sync

> Read and written by [`/sync-template`](../.claude/commands/sync-template.md), and updated by
> [`/learn-and-improve`](../.claude/commands/learn-and-improve.md) Step 6 when it sends a fix
> upstream. Replace every `<<< ... >>>` below, then delete this line.
>
> **If this repo *is* the template, delete this file.** There is nothing upstream of you.

<<< **Upstream**: the repo these skills came from, and the branch to track.
**Local clone**: where it is checked out locally, if it is. Optional — `/sync-template` clones
fresh into a scratch directory when this is missing or stale, which is often the safer choice.
**Git remote**: add the template once as a second remote in this repo
(`git remote add template <url>`) so its blobs are fetchable and git's own three-way merge can
do the work. The histories being unrelated is fine.
**Synced to**: the template commit this repo currently reflects. On first setup, use the commit
you copied from; if that is unknowable, use the template's first commit and expect the first
sync to review a lot. >>>

## What the template owns

<<< Which paths a sync is allowed to touch, and how. Be explicit — this is what stops a sync
reaching into your own work. A starting point:

| Path | Rule |
|------|------|
| `.claude/commands/*.md` | tracked — kept identical unless a ledger line says otherwise |
| `memory/learnings/README.md` | tracked |
| `project/CONTEXT.md` | structure only — its headings come from the template, its content is yours |
| the template's own README / ADAPTING | not tracked |

Everything under `tasks/` and `memory/learnings/inbox|processed/` is yours and is never
touched by a sync. >>>

## Ledger

<<< One line per template commit you have decided on, newest first. Start empty; `/sync-template`
appends to it.

A `skipped` line is a decision, not an oversight, and is never re-offered — which is the whole
reason this is a ledger and not a single "synced to" pointer. A bare pointer cannot record a
refusal: you would either advance past the change and lose any trace of why your file differs,
or sit behind it and be asked about it forever.

- `<sha>` <date> — **adopted** — <commit subject>
- `<sha>` <date> — **adapted** — <subject> — kept <the local specific> because <reason>
- `<sha>` <date> — **skipped** — <subject> — <why it does not apply here>
- `<sha>` <date> — **ours** — <subject> — upstreamed from this repo by /learn-and-improve
>>>

## Known local divergence

<<< Where your copy knowingly differs from the template, and why. Keep this current: an
intentional difference that is not written down here reads as drift to the next sync, which
will helpfully "fix" it back. >>>
