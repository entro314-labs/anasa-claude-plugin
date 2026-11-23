# Anasa for Claude

[Anasa](https://anasa.md) is a Markdown notes app that keeps your vault on your own computer.
This plugin lets Claude work with it: search and read your notes, follow backlinks and tags, add
tasks, revise the note you have open once you approve the edit in Anasa, and send something from
a conversation to your Anasa inbox.

## What's inside

Anasa has two MCP servers, and the plugin carries both. They do different jobs and load in
different places.

- **The vault server** (`anasa-vault` in `.mcp.json`): the `anasa-mcp` program that ships inside
  the Anasa desktop app, started through `scripts/anasa-mcp`. It runs on your computer and works
  on the vault open in Anasa. Eight tools: `search_notes`, `read_note`, `list_tags`,
  `find_backlinks`, `create_task`, `schedule_task`, `get_open_note`, `edit_note`.
  **It loads only in Claude Code and in Cowork sessions that run on your computer.** Claude chat
  on the web, desktop and mobile does not start local servers and ignores it.
- **The Anasa connector** (`anasa` in `.mcp.json`): the remote MCP server at
  `https://anasa.md/api/mcp`, signed in with your Anasa account. Three tools: `capture_note`,
  `list_inbox`, `get_plan_status`. It works everywhere, including chat, and it is the only part
  of the plugin that chat shows.
- **Skills**
  - `anasa` - what each server is for, and what to tell you when the vault tools are refused.
  - `answer-from-the-vault` - answer a question from your notes and name the notes it came from.
  - `turn-items-into-tasks` - make tasks out of the action items in a note or a conversation.
  - `revise-the-open-note` - change the note on screen with exact edits you approve in Anasa.
  - `capture-to-anasa` - send a note to your Anasa inbox and see what is waiting there.

## Requirements

For the vault tools:

- The Anasa desktop app, installed where its installers put it. Get it from
  [anasa.md/download](https://anasa.md/download).
  - **macOS**: supported (`/Applications` or `~/Applications`, including a Homebrew cask
    install).
  - **Linux**: the `.deb`, `.rpm` and pacman packages are supported. The AppImage is not,
    because it keeps `anasa-mcp` inside its own image.
  - **Windows**: not supported by this plugin, because its launcher is a POSIX shell script. In
    Anasa, switch on **Enable MCP server** under **Settings → AI Features → MCP server**; **Add
    Anasa to an AI app** there adds the same server to Claude Code or Claude Desktop directly.
- **Enable MCP server** switched on in Anasa, under **Settings → AI Features → MCP server**. It
  is off until you turn it on, and while it is off Anasa refuses every vault tool.
- A vault open in Anasa, with Anasa unlocked. The six tools that read notes and add tasks keep
  working after you close Anasa, on the vault it last had open.
- For `get_open_note` and `edit_note`: Anasa running, with **Let agents use the open app** also
  switched on in the same place. It is off until you turn it on.

For the connector: an Anasa account with a license. Listing your inbox and checking your plan
work with any license; sending a note needs an active [Anasa Cloud](https://anasa.md/pricing)
plan, and the desktop app to import the note into your vault.

If you already added Anasa to Claude Code from inside the app, remove that entry (in Anasa, or
with `claude mcp remove anasa`) or the vault tools appear twice.

## Use it

Install the plugin. In Claude Code the vault server starts with it; run `/mcp` to sign in to the
Anasa connector. In chat and Cowork, connect Anasa from the plugin's **Connectors** tab and sign
in with your Anasa account.

Then ask for what you need:

- "What did I decide about the pricing page? Tell me which notes you used."
- "Which notes link to my 'Project Alpha' note?"
- "Pull the action items out of today's meeting note and make them tasks. The report is due on
  the 30th."
- "Tighten the paragraph I have selected in Anasa."
- "Save this summary to Anasa as 'Q3 planning notes', tagged work."

## What Claude can and cannot do

With the vault server:

- It reads only the vault open in Anasa. Notes of your other vaults are out of reach.
- It is refused while Anasa is locked, while no vault is open, and for a vault that is encrypted
  at rest ("Encrypt this workspace at rest"). Claude is told the reason and passes it on.
- Notes in your "Folders the AI can't see", and notes with `ai: false` in their properties, are
  hidden from it: they are missing from search, tags and backlinks, and cannot be read or
  edited.
- It can add tasks, at most 30 in a minute. It cannot list, change, complete or delete tasks.
- It can change a note only through `edit_note`, one passage at a time, and an edit waits for
  your approval in the Anasa window, unless you chose there to allow further edits for the rest
  of that Anasa session. An edit you do not answer in about two minutes is dropped.
- It cannot create, move, rename or delete notes.

With the connector:

- It can send a Markdown note to your inbox; Anasa files it under `Inbox/` on its next sync. It
  cannot send attachments, and it cannot change or remove a capture once it is sent.
- It can list the captures still waiting in your inbox, including ones sent from other devices,
  and report your Anasa Cloud plan and storage.
- It cannot read, search, change or delete the notes in your vault. Vault sync is end-to-end
  encrypted, so Anasa's servers hold only ciphertext.

## Data

What runs on your computer:

- `scripts/anasa-mcp`, a short shell script in this plugin. It looks for the `anasa-mcp` program
  in the standard install locations listed above and starts it. It downloads nothing, installs
  nothing and makes no network requests. If Anasa is not installed it prints a message and
  stops.
- `anasa-mcp`, which is part of the Anasa desktop app, not of this plugin. It reads the vault's
  database on your disk, writes the tasks you ask for, and for `get_open_note` and `edit_note`
  talks to the running Anasa app over a local connection on your own machine (`127.0.0.1`). It
  does not go through Anasa's servers.

What goes to `anasa.md`:

- Calls to the three connector tools, signed in as you. `capture_note` sends the title, text and
  tags of the note Claude was asked to save, and nothing else from your conversation.
  `list_inbox` and `get_plan_status` return your pending captures and your plan and storage
  status. A capture is held until your desktop app imports it and is deleted from Anasa's
  servers straight after.
- Signing in happens in your browser at `accounts.anasa.md`.

The plugin fetches nothing else and sends nothing to any other service. Whatever Claude reads
through either server, including the text of your notes, becomes part of your conversation with
Claude, which Anthropic handles under its own terms. See Anasa's
[privacy policy](https://anasa.md/privacy), under "Captures sent from Claude".

To disconnect, remove the connector in Claude, or uninstall the plugin.

## Install in Claude Code

```
/plugin install anasa --marketplace entro314-labs/anasa-claude-plugin
```

## Support

[Anasa for Claude](https://anasa.md/integrations/claude) describes the connector and what to do
when a call fails. For anything else, write to hey@anasa.md with what you asked Claude and the
message it showed.

## License

LGPL-3.0-only. See [LICENSE](LICENSE) for the GNU Lesser General Public License, version 3, and
[COPYING](COPYING) for the GNU General Public License it builds on. The license covers this plugin's
files, not the Anasa app.
