---
name: synapse-cli
description: >
  Operate the Synapse note-taking app programmatically via its local CLI binary (synapse-cli).
  Use when a task involves creating, reading, updating, or deleting notes and repos in a Synapse
  database; managing note attributes (tags, pin, trash); searching notes; building link graphs;
  manipulating file trees (move, copy, rename, delete); reading or writing workspace/settings state;
  or performing Git operations (status, commit, push, pull, log, branch) on repos.
  Triggers: "synapse", "synapse-cli", "note", "vault", "repo", "knowledge base", "markdown notes",
  "link graph", "note attributes", "note tags", "note search".
---

# Synapse CLI

`synapse-cli` is a local command-line binary that exposes all Synapse features for programmatic
access. It outputs JSON to stdout on success and `{"error":"..."}` to stderr on failure.

## Prerequisites

- Binary location: `src-tauri/target/release/synapse_cli(.exe)` (or `target/debug/` for dev builds)
- Build: `cd src-tauri && cargo build --release --bin synapse_cli`
- Requires a Synapse database directory (any folder — Synapse creates `.database/` inside it)

## Invocation

```
synapse-cli -d <database_path> [-r <repo>] <command> [args...]
```

| Flag | Short | Required | Description |
|------|-------|----------|-------------|
| `--database <path>` | `-d` | Yes (most commands) | Synapse database root directory |
| `--repo <name>` | `-r` | No | Target repo. Auto-detects if omitted |
| `--help` | `-h` | — | Print help |
| `--version` | `-v` | — | Print version |

## Output Convention

- **Success**: JSON on **stdout**, exit code `0`
- **Error**: `{"error":"..."}` on **stderr**, exit code `1`

Always parse stdout as JSON. Check exit code to determine success/failure.

## Command Reference

For full details and examples of every command, read the English API reference [API.md](API.md) or the Traditional Chinese translation [API.zh-TW.md](API.zh-TW.md).

### Database & Repo

| Command | Description |
|---------|-------------|
| `init-database` | Ensure `.database/` structure exists |
| `list-repos` | List all repo names → `["Repo1","Repo2"]` |
| `list-repo-summaries` | Repos with noteCount, tagCount, lastModified |
| `create-repo <name>` | Create a new repo |
| `select-repo <name>` | Set active repo |
| `delete-repo <name>` | Delete repo permanently |
| `get-active-repo-path` | Get filesystem path of active repo |

### File Tree

| Command | Description |
|---------|-------------|
| `list-tree` | Nested TreeNode structure of current repo |
| `create-folder <path>` | Create folder at relative path |
| `move-item <from> <to_folder>` | Move file/folder |
| `copy-item <path>` | Duplicate file |
| `rename-item <path> <new_name>` | Rename file/folder |
| `delete-item <path>` | Delete file/folder |

### Notes

| Command | Description |
|---------|-------------|
| `list-notes` | All notes as `[{path, title}]` |
| `read-note <path>` | Read note content (markdown string) |
| `save-note <path> <content>` | Overwrite note content. Use `@file` to read from file |
| `create-note <folder> <name>` | Create new blank `.md` note |

### Attributes

| Command | Description |
|---------|-------------|
| `get-note-attributes <path>` | Get tags, pinned, trashed, etc. |
| `set-note-attributes <path> <json>` | Set attributes (JSON object) |
| `get-all-note-attributes` | Attributes for every note in repo |
| `list-tags-and-groups` | All tags and groups → `{tags:[...], groups:[...]}` |

### Search & Graph

| Command | Description |
|---------|-------------|
| `search <query> [limit]` | Full-text search → `[{path, title, excerpt}]` |
| `scan-links` | Build wiki-link graph → `{nodes:[...], edges:[...]}` |

### State & Settings

| Command | Description |
|---------|-------------|
| `get-workspace-state` | Current workspace state (JSON) |
| `set-workspace-state <json>` | Update workspace state |
| `get-graph-view-state` | Graph view configuration |
| `set-graph-view-state <json>` | Update graph view config |
| `get-setting-state` | App settings |
| `set-setting-state <json>` | Update app settings |
| `get-app-prefs` | Global preferences (no `-d` needed) |
| `set-app-prefs <json>` | Update global preferences |
| `get-app-prefs-path` | Filesystem path to prefs file |

### Git

| Command | Description |
|---------|-------------|
| `git-status` | `git status --porcelain` |
| `git-log [n]` | Last n commits (default 20) |
| `git-diff` | Staged + unstaged diff |
| `git-add-all` | Stage all changes |
| `git-commit <message>` | Commit staged changes |
| `git-push` | Push to remote |
| `git-pull` | Pull from remote |
| `git-branch` | Current branch name |
| `git-branch-list` | All branches |
| `git-remote-url` | Remote origin URL |
| `git-init` | Initialize git repo |
| `git-clone <url>` | Clone into current repo path |

## Common Workflows

### Create a note and tag it

```bash
CLI="synapse-cli -d /path/to/db -r MyRepo"
$CLI create-note "" "idea.md"
$CLI save-note "idea.md" "# My Idea\nSome content here"
$CLI set-note-attributes "idea.md" '{"tags":["brainstorm","priority"],"pinned":true}'
```

### Search and read

```bash
$CLI search "machine learning" 5    # top 5 hits
# parse the path from first result, then:
$CLI read-note "AI/ml-notes.md"
```

### Build a knowledge graph

```bash
$CLI scan-links | jq '.nodes | length'   # count of linked notes
$CLI scan-links | jq '.edges[] | select(.source == "index.md")'  # outgoing links from index
```

### Git workflow

```bash
$CLI git-add-all
$CLI git-commit "feat: add new notes"
$CLI git-push
```

## Key Data Types

- **NoteEntry**: `{path: string, title: string}`
- **TreeNode**: `{name, path, node_type: "file"|"folder", depth, children: TreeNode[]}`
- **NoteAttributes**: `{tags?: string[], groups?: string[], pinned?: bool, trashed?: bool, trashedDate?: string, aliases?: string[], cssclasses?: string[], publish?: bool, [key: string]: any}`
- **SearchHit**: `{path, title, excerpt}`
- **LinkGraph**: `{nodes: [{id, label, group}], edges: [{source, target}]}`
- **RepoSummary**: `{name, noteCount, tagCount, groupCount, lastModified}`
- **WorkspaceState**: `{openFiles, activeFile, sidebarWidth, ...}`
