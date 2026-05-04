---
name: take-notes
description: Creates, reads, and updates project notes in an Obsidian vault. Use when the user asks to take notes, write notes, document something, save notes, jot down notes, or update notes for the current project. Also use when the user mentions Pi Notes or wants to persist information between sessions.
---

# Take Notes

Manage project-specific notes inside the "Pi Notes" folder in your Obsidian vault. Each project gets its own subfolder named after the project's working directory, so notes are always organized by context.

## Setup

The Obsidian desktop app must be running for CLI commands to work. Check and launch if needed:

```bash
# Check if Obsidian is running
pgrep -f "electron.*obsidian" >/dev/null && echo "RUNNING" || echo "NOT_RUNNING"

# If NOT running, launch it:
nohup obsidian &>/dev/null & disown

# Wait for it to be ready
for i in $(seq 1 20); do
  if pgrep -f "electron.*obsidian" >/dev/null; then
    sleep 3
    break
  fi
  sleep 1
done
```

Do NOT proceed until the app is confirmed running.

## Process

### Step 1: Determine the Vault Path

All notes are stored inside the Obsidian vault. Determine the vault's filesystem path:

```bash
obsidian vault 2>/dev/null
```

This outputs several fields. Extract the `path` field using a tab delimiter (important: vault paths can contain spaces like "Obsidian Vault"):

```bash
VAULT_PATH=$(obsidian vault 2>/dev/null | awk -F'\t' '/^path/{print $2}')
```

If the CLI is unavailable, fall back to a common path:

```bash
VAULT_PATH="${VAULT_PATH:-$HOME/Obsidian Vault}"
```

**Always verify the path exists** before writing files:

```bash
ls "$VAULT_PATH" >/dev/null 2>&1 && echo "VAULT_OK" || echo "VAULT_MISSING"
```

If `VAULT_MISSING`, ask the user for their vault path or check common locations like `~/Obsidian Vault`, `~/obsidian`, `~/Documents/Obsidian`.

Store this value as `VAULT_PATH` for use in all subsequent steps.

### Step 2: Determine and Sanitize the Project Name

The "notes directory" is named after the current workspace's root directory. Determine it as follows:

```bash
# For git projects, use the repo root directory name:
git rev-parse --show-toplevel 2>/dev/null | xargs basename

# If not in a git repo, use the current working directory name:
basename "$(pwd)"
```

**Then sanitize the name** for Obsidian compatibility. Obsidian hides dot-prefixed files/folders and special characters cause issues. Apply these transformations in order:

1. **Strip leading dots** — `.pi` → `pi`
2. **Replace spaces with hyphens** — `my project` → `my-project`
3. **Remove or replace special characters** (keep only `a-z`, `0-9`, and hyphens) — `project_v2!` → `project-v2`
4. **Collapse consecutive hyphens** — `my--project` → `my-project`
5. **Strip leading/trailing hyphens** — `-project-` → `project`
6. **Fallback**: If the name is empty after sanitization, use `unnamed-project`

Example sanitization in bash:

```bash
RAW_NAME=$(git rev-parse --show-toplevel 2>/dev/null | xargs basename || basename "$(pwd)")
PROJECT_NAME=$(echo "$RAW_NAME" | sed 's/^\.//' | sed 's/[[:space:]]/-/g' | sed 's/[^a-zA-Z0-9-]//g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
PROJECT_NAME=${PROJECT_NAME:-unnamed-project}
echo "Project name: $PROJECT_NAME (raw: $RAW_NAME)"
```

Store the sanitized value as `PROJECT_NAME` for use in subsequent steps.

### Step 3: Ensure the "Pi Notes" Folder Exists

Check if the top-level "Pi Notes" folder exists in the vault:

```bash
obsidian folders 2>/dev/null | grep -q "^Pi Notes$" && echo "EXISTS" || echo "NOT_FOUND"
```

If it does NOT exist, create it:

```bash
obsidian create name="Pi Notes/.gitkeep" content="" overwrite
```

### Step 4: Ensure the Project Subfolder Exists

Check if a subfolder for this project exists:

```bash
obsidian files folder="Pi Notes" 2>/dev/null | head -50
```

Look for a folder named `Pi Notes/{PROJECT_NAME}` in the listing. If it does not exist, create it using the filesystem (the Obsidian CLI has issues with nested path creation):

```bash
mkdir -p "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}"
```

### Step 5: Survey Existing Notes (Selective Reading)

List all notes in the project folder to understand what's there:

```bash
obsidian files folder="Pi Notes/{PROJECT_NAME}" 2>/dev/null
```

Also check for feature-specific subfolders:

```bash
ls -R "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/" 2>/dev/null
```

**Do NOT read every note.** Instead, read selectively based on what's relevant to the current request.

#### What to scan (just the outline/headings):

- **Recent session notes** — only the most recent 1–2 (e.g., `session-2025-04-23.md`). Use `obsidian outline` to get just the structure:
  ```bash
  obsidian outline file="Pi Notes/{PROJECT_NAME}/session-YYYY-MM-DD" format=tree
  ```
- **Top-level notes with topic-relevant names** — if the user is asking about architecture, read `architecture-decisions.md`. If about progress, read `progress.md`. Only read notes whose name matches the topic.

#### What NOT to read automatically:

- **Old session notes** — skip anything older than the most recent 2
- **PRD and task files** — these live inside feature subfolders (`Pi Notes/{PROJECT_NAME}/{FEATURE_NAME}/`) and can be very long. Only read them if the user's request is specifically about that feature. Even then, start with the outline:
  ```bash
  obsidian outline file="Pi Notes/{PROJECT_NAME}/{FEATURE_NAME}/prd-{FEATURE_NAME}" format=tree
  ```
- **Notes clearly unrelated to the topic** — if the user is asking about testing, don't read `design-decisions.md`

#### Reading strategy:

1. Start by listing all files and scanning filenames for relevance
2. For relevant files, read just the outline first (`obsidian outline`)
3. Only read the full content of a note if the outline suggests it contains directly useful or conflicting information

This approach keeps context lean and avoids pulling in large, irrelevant documents.

### Step 6: Document Relevant Information

Based on the user's request and the current state of the project, create or update notes.

**Creating a new note:**

```bash
obsidian create name="Pi Notes/{PROJECT_NAME}/<note-title>" content="<content>"
```

**Appending to an existing note:**

```bash
obsidian append file="Pi Notes/{PROJECT_NAME}/<note-name>" content="<content>"
```

**Prepending to an existing note:**

```bash
obsidian prepend file="Pi Notes/{PROJECT_NAME}/<note-name>" content="<content>"
```

### Pi Notes Directory Structure

The Pi Notes folder follows this structure:

```
Pi Notes/
└── {PROJECT_NAME}/                    # One folder per project/workspace
    ├── session-2025-04-23.md            # General project notes
    ├── architecture-decisions.md
    ├── progress.md
    └── {FEATURE_NAME}/                  # One subfolder per feature/task
        ├── prd-{FEATURE_NAME}.md        # PRD for this feature
        └── tasks-{FEATURE_NAME}.md      # Task list for this feature
```

- **Project-level notes** (sessions, architecture, conventions, etc.) go directly in `Pi Notes/{PROJECT_NAME}/`
- **Feature-specific documents** (PRDs and task lists) go in `Pi Notes/{PROJECT_NAME}/{FEATURE_NAME}/`
- When creating notes, check whether the content is feature-specific or project-general and place it accordingly

### Note-Taking Guidelines

- **What to document**: Anything the user explicitly asks you to write down, plus relevant project context such as architecture decisions, current progress, open issues, key file locations, conventions, and anything worth persisting between sessions.
- **Note naming**: Use short, descriptive names in lowercase with hyphens (e.g., `architecture-decisions`, `progress`, `conventions`, `open-issues`).
- **Note format**: Use standard Obsidian-flavored Markdown. Include a YAML frontmatter block with a `date` field and a `tags` field for discoverability.
- **Structure**: Each note should have a clear heading, a brief summary, and organized sections. Use checkboxes for action items.
- **Linking**: Use `[[wikilinks]]` to cross-reference related notes within the Pi Notes folder (e.g., `[[Pi Notes/{PROJECT_NAME}/architecture-decisions]]` or `[[Pi Notes/{PROJECT_NAME}/{FEATURE_NAME}/prd-{FEATURE_NAME}]]`).
- **Be additive**: When updating notes, prefer appending new information rather than rewriting. Only overwrite when explicitly asked or when the existing content is clearly stale.
- **Avoid duplication**: Before creating a new note, check if an existing note already covers the topic. If so, append to it instead.

### Example Note Format

```markdown
---
date: 2025-04-23
tags: [pi-notes, {PROJECT_NAME}]
---

# {Note Title}

Brief summary of what this note covers.

## Details

- Key point 1
- Key point 2

## Open Items

- [ ] Thing that still needs to be done
- [x] Thing that was completed
```

### Step 7: Confirm to the User

After writing or updating notes, tell the user:

1. Which notes were created or updated
2. A brief summary of what was documented
3. The Obsidian path where the notes can be found (e.g., `Pi Notes/{PROJECT_NAME}/progress`)

## Error Handling

- **Obsidian not running**: Follow the Setup instructions to launch it. If it still won't start after 20 seconds, tell the user and suggest they open the app manually.
- **Folder creation fails**: Try using the filesystem directly. The vault path can be found with `obsidian vault` (look for the `path` field). Then use `mkdir -p` to create folders and write Markdown files directly.
- **Note already exists when creating**: Use `overwrite` only if you intend to replace. Otherwise, use `append` or read the existing note first and merge content.
- **Empty project name**: This is handled by the sanitization fallback in Step 1, which defaults to `unnamed-project`.

## Filesystem Fallback

If the Obsidian CLI is unavailable or unreliable, you can operate directly on the filesystem. The vault path was already determined in Step 1, but if needed:

```bash
# Find the vault path (use awk with tab delimiter to handle spaces in path)
VAULT_PATH=$(obsidian vault 2>/dev/null | awk -F'\t' '/^path/{print $2}')
VAULT_PATH="${VAULT_PATH:-$HOME/Obsidian Vault}"

# Create directories
mkdir -p "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}"

# Write a note
cat > "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/progress.md" << 'EOF'
---
date: 2025-04-23
tags: [pi-notes]
---
# Progress
...
EOF

# Read existing notes
cat "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/"*.md 2>/dev/null
```