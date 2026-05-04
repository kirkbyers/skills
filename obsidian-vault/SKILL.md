---
name: obsidian-vault
description: Interacts with Obsidian vault using the Obsidian CLI. Use when the user mentions Obsidian, Obsidian CLI, obsidian-cli, vault notes, daily notes, or wants to read, write, search, or manage notes in their Obsidian vault. Also use when the user asks to sync, back up, or inspect their Obsidian data.
---

# Obsidian Vault Management

Use this skill to interact with your Obsidian vault via the `obsidian` command-line interface.

## ⚠️ Prerequisite: Obsidian Desktop App Must Be Running

The `obsidian` CLI communicates with the running Obsidian desktop app. **If the app is not running, CLI commands will hang or fail.**

Always check if the app is running before issuing CLI commands:

```bash
pgrep -f "electron.*obsidian" >/dev/null && echo "RUNNING" || echo "NOT_RUNNING"
```

### If the app is NOT running

1. **Inform the user**: Say something like "The Obsidian desktop app isn't currently running — I'll launch it for you now. It may take a few seconds to start up."
2. **Launch it**:

```bash
nohup obsidian &>/dev/null & disown
```

3. **Wait for it to be ready** (the app needs a few seconds to initialize):

```bash
for i in $(seq 1 20); do
  if pgrep -f "electron.*obsidian" >/dev/null; then
    sleep 3  # give the app time to fully load the vault
    break
  fi
  sleep 1
done
```

4. **Verify the CLI works** by running a lightweight command like `obsidian vault`.

Do NOT proceed with the user's request until the app is confirmed running and the CLI is responsive.

## Vault Info

- **Show vault info**: `obsidian vault`
- **List known vaults**: `obsidian vaults`
- **List files**: `obsidian files`
- **List files in a folder**: `obsidian files folder=<path>`
- **List folders**: `obsidian folders`

## Reading Notes

- **Read file content**: `obsidian read file=<name>` or `obsidian read path=<path>`
- **Show file info**: `obsidian file file=<name>`
- **Show headings (outline)**: `obsidian outline file=<name> format=tree`

## Writing & Modifying Notes

- **Create a new note**: `obsidian create name=<name> content=<text>`
- **Create with overwrite**: `obsidian create name=<name> content=<text> overwrite`
- **Append content to a note**: `obsidian append file=<name> content=<text>`
- **Prepend content to a note**: `obsidian prepend file=<name> content=<text>`
- **Delete a note**: `obsidian delete file=<name>`
- **Move/rename a note**: `obsidian move file=<name> to=<destination>`

## Daily Notes

- **Get daily note path**: `obsidian daily:path`
- **Read today's daily note**: `obsidian daily:read`
- **Append to daily note**: `obsidian daily:append content=<text>`
- **Prepend to daily note**: `obsidian daily:prepend content=<text>`

## Search & Discovery

- **Search vault**: `obsidian search query=<text>`
- **Search with context lines**: `obsidian search:context query=<text>`
- **List backlinks**: `obsidian backlinks file=<name>`
- **Find unresolved links**: `obsidian unresolved`
- **Find orphans (no incoming links)**: `obsidian orphans`
- **Find dead ends (no outgoing links)**: `obsidian deadends`
- **Recent files**: `obsidian recents`

## Properties (YAML Frontmatter)

- **List properties**: `obsidian properties file=<name>`
- **Set a property**: `obsidian property:set name=<prop> value=<value> type=text file=<name>`
- **Remove a property**: `obsidian property:remove name=<prop> file=<name>`
- **Read a property**: `obsidian property:read name=<prop> file=<name>`

## Tasks

- **List all tasks**: `obsidian tasks`
- **List incomplete tasks**: `obsidian tasks todo`
- **List completed tasks**: `obsidian tasks done`
- **Tasks in a file**: `obsidian tasks file=<name>`
- **Toggle task done**: `obsidian task ref=<path:line> done`

## Tags & Aliases

- **List all tags**: `obsidian tags`
- **Tag info with counts**: `obsidian tags counts`
- **List aliases**: `obsidian aliases`

## Sync

- **Check sync status**: `obsidian sync:status`
- **View sync history for a file**: `obsidian sync:history file=<name>`
- **Read a sync version**: `obsidian sync:read file=<name> version=<n>`
- **Restore a sync version**: `obsidian sync:restore file=<name> version=<n>`
- **List deleted files in sync**: `obsidian sync:deleted`

> **Note**: The `obsidian sync` commands require Obsidian Sync to be configured for the vault. If sync is not set up, these commands will return an error. See the [headless sync tool](https://github.com/obsidianmd/obsidian-headless) (`ob`) for CLI-driven sync without the desktop app.

## Tips

- Use `file=<name>` for wikilink-style name matching, or `path=<path>` for exact folder/file.md paths.
- The `obsidian help` command lists all available commands and their options.
- Most commands default to the active file when `file`/`path` is omitted.