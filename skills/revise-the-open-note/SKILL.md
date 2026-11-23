---
name: revise-the-open-note
description: Revise the note someone has open in Anasa. Use when they ask to fix, tighten, rewrite, reword, correct, shorten or reformat "this note", "this paragraph", "what I have selected" or "the note I'm working on" in Anasa, or to apply a change you proposed to a note in their vault.
---

# Revising the open note

`get_open_note` and `edit_note` belong to the `anasa-vault` server and act on the running Anasa
app. They need Anasa open with **Enable MCP server** and **Let agents use the open app** switched
on (Settings → AI Features → MCP server). When they are not, the tool answers "Anasa is not open,
or its live bridge is off"; tell the person where the switches are and stop.

An edit waits for the person to approve it in Anasa. Nothing you send changes a note unless they
approved it there, either that edit or, by their own choice, edits for the rest of that Anasa
session.

## 1. Read what is on screen

Call `get_open_note` (no arguments). It returns:

- `id`, `path`, `title`
- `content`: the note as it is now, including edits not yet saved
- `selection`: the text selected in the editor, or null

Always read first, even if the note was read earlier in the conversation: the person may have
typed since. "No note is open in Anasa." and "The open note is not available to AI." are final
for now; pass them on.

When the request is about "this", "the selected part" or "this paragraph" and `selection` has
text, work on the selection. When it is null, ask which part they mean unless the request is
plainly about the whole note.

## 2. Decide the change

Change only what was asked. Keep their voice, their Markdown, their links and their headings
unless those are the thing to change. For anything beyond a small fix, say in a line what you
are about to change before you send it, so the approval prompt in Anasa is no surprise.

## 3. Send exact edits

`edit_note` takes:

- `note_id`: the `id` from `get_open_note`.
- `original`: a passage copied from `content`, character for character. It must not be empty
  and must occur exactly once in the note.
- `replacement`: what takes its place. An empty string deletes the passage.

One call replaces one passage.

- Copy `original` out of the content you just read; do not retype it from memory or tidy it on
  the way.
- If the words you want to change appear more than once (a repeated phrase, a common word), take
  more of the surrounding sentence until the passage is unique. Keep it as short as it can be
  while unique, and repeat the unchanged surrounding words in `replacement`.
- The edit is refused, and nothing is written, when `original` is not found or matches several
  places. Read the note again and send a longer, exact passage.
- For several changes, send several calls one after another, never in parallel. Each is checked
  against the note as it is at that moment, so an earlier edit that changed a passage changes
  what the next `original` must be.
- To add text, use the passage it should follow as `original` and repeat it at the start of
  `replacement`.

## 4. Wait for the approval

Each call opens an approval prompt in Anasa and returns once the person answers. Tell them to
look at Anasa. The call waits about two minutes; after that it comes back as an error, the
prompt is withdrawn, and nothing was changed.

- Declined: accept it. Do not send the same edit again; ask what they would like instead.
- No answer in time: say so and offer to send it again when they are ready.
- The person can choose to allow further edits for the rest of that Anasa session. That is
  theirs to offer; do not ask for it.

## 5. Report

Say in a line or two what changed. If some edits were applied and others were not, say which.
Do not claim a change that was declined or timed out.

## Other notes

`edit_note` also accepts the `id` of a note found with `search_notes`. Read that note with
`read_note` first, copy `original` from what it returned, and name the note to the person
before you send the edit, since it is not the one in front of them.

A note the person has hidden from AI cannot be read or edited from here.
