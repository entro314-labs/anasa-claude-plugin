---
name: turn-items-into-tasks
description: Turn action items into tasks in someone's Anasa vault. Use when they ask to add a task or a to-do to Anasa, to pull the action items out of a note, meeting notes or this conversation and make tasks of them, or to put something on a date.
---

# Turning action items into tasks

`create_task` and `schedule_task` belong to the `anasa-vault` server. Each call adds one new
task to the vault open in Anasa and returns its id.

## What you cannot do afterwards

There is no tool here that lists, changes, completes or deletes a task. So:

- You cannot check whether a task already exists. Ask if you suspect a duplicate.
- You cannot take a task back. The person removes it in Anasa.
- A repeated call makes a second task. Never re-send a call that already returned an id.

For that reason, agree the list first whenever there is more than one item: show the tasks you
are about to create, with their dates, and create them after the person says yes. A single task
they dictated needs no second confirmation.

## Finding the items

- From a note: read it first with `read_note`, or `get_open_note` for the note on screen.
- From the conversation: use what was said.

Take only what is an action for the person: a thing to do, with a verb. Leave out decisions,
facts and other people's actions unless they ask for those too. Keep their wording.

## Which tool

**No date:** `create_task`.

- `title` (required): the task as one line. An empty title is refused.
- `due_date` (optional): an ISO-8601 date such as `2026-06-30`, or a full timestamp.
- `project` (optional): the project or area it belongs to.
- `description` (optional): leave it out in most cases. Anasa stores one text for a task: the
  `description` when you send one, otherwise the `title`. If you send both, the title is not
  kept. To add detail, put the whole wording in `description` and make sure it still reads as
  the task.

**A date:** `schedule_task`.

- `title` and `scheduled_date` are both required. The date becomes the task's due date.
- `project` is optional. There is no `description` here.

Both tools store the date text as you send it and do not check it, so send a real calendar date
in `YYYY-MM-DD` form. Work out "Friday" or "next week" from today's date, and say the date you
used. Set a date only when the person or the source named one; do not invent deadlines.

Use `project` only with a project name the person used or one that appears in their notes.
Priority and tags cannot be set from here.

## The limit

The two tools share one limit: at most 30 task writes in 60 seconds, refilled steadily. Past it
a call fails with "write rate limit exceeded". Nothing was written by that call; wait a few
seconds and send it again, once. For a long list, send the calls one after another and expect to
pause.

## When a call is refused

"no vault is open", "Anasa is locked" and "the vault ... is encrypted" mean the app does not
allow it right now; see the `anasa` skill. Stop and tell the person. Do not keep the rest of
the list going.

## Afterwards

Say in one or two lines what was added and with which dates. Do not paste the ids unless asked.
If some calls failed, name the items that were not added.
