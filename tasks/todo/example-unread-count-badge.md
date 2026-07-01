# Add an unread-count badge to the header nav

**Priority**: medium
**Status**: todo

> This is a worked example task that ships with the template so you can watch one flow
> through the loop. It is deliberately generic. Delete it once you have written your own,
> or run `/start-task` on it against a toy project to see the cycle end to end. Fill in
> `project/CONTEXT.md` first: `/start-task` reads it for the real paths and the test
> command, so without it this example will send the agent hunting.

## Goal

Show the number of unread notifications on the header's bell icon, so a user can see at a
glance that something needs attention without opening the panel. The badge should be
honest about zero: no badge when there is nothing unread.

## Context

- The header nav renders the bell icon (find the component that owns the top navigation).
- An unread count is already available from the notifications state or endpoint; this task
  displays it, it does not compute it.
- Follow the existing badge or pill styling if the project has one, rather than inventing a
  new visual.

## Plan

- [ ] Locate the header nav component and the bell icon within it.
- [ ] Read the unread count from the existing notifications source.
- [ ] Render a small badge over the bell showing the count.
- [ ] Hide the badge entirely when the count is zero.
- [ ] Cap the display at "9+" when the count is above nine.

## Acceptance criteria

- [ ] With unread notifications, the bell shows a badge with the correct number.
- [ ] With zero unread, no badge is rendered at all (not a badge showing "0").
- [ ] A count above nine renders as "9+".
- [ ] The badge uses the project's existing badge styling, not a bespoke one-off.

## Notes

Out of scope: changing how unread state is computed or marking notifications as read. Those
are separate concerns. If the project has no existing notifications source, that is a
missing prerequisite, not part of this task. Stop and flag it.
