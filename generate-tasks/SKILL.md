---
name: generate-tasks
description: Creates a step-by-step task list in Obsidian under Pi Notes from user requirements. Use when the user wants to generate tasks, create a task list, break down a feature into tasks, or create sub-tasks. Saves to Pi Notes/[project]/[feature]/tasks-[feature].md.
---

# Generate Tasks

Creates a structured task list from requirements and saves it into the project's Pi Notes folder in Obsidian, organized by feature alongside any existing PRD.

## Setup: Obsidian Must Be Running

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

All notes are stored inside the Obsidian vault. Determine the vault’s filesystem path:

```bash
obsidian vault 2>/dev/null
```

This outputs several fields. Extract the `path` field using a tab delimiter (important: vault paths can contain spaces like “Obsidian Vault”):

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

### Step 2: Determine the Project Name

The project name is derived from the current workspace's root directory, then sanitized for Obsidian compatibility.

```bash
# For git projects, use the repo root directory name:
git rev-parse --show-toplevel 2>/dev/null | xargs basename

# If not in a git repo, use the current working directory name:
basename "$(pwd)"
```

**Sanitize the name** — Obsidian hides dot-prefixed files/folders and special characters cause issues. Apply these transformations in order:

1. **Strip leading dots** — `.pi` → `pi`
2. **Replace spaces with hyphens** — `my project` → `my-project`
3. **Remove or replace special characters** (keep only `a-z`, `0-9`, and hyphens) — `project_v2!` → `project-v2`
4. **Collapse consecutive hyphens** — `my--project` → `my-project`
5. **Strip leading/trailing hyphens** — `-project-` → `project`
6. **Fallback**: If empty after sanitization, use `unnamed-project`

```bash
RAW_NAME=$(git rev-parse --show-toplevel 2>/dev/null | xargs basename || basename "$(pwd)")
PROJECT_NAME=$(echo "$RAW_NAME" | sed 's/^\.//' | sed 's/[[:space:]]/-/g' | sed 's/[^a-zA-Z0-9-]//g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
PROJECT_NAME=${PROJECT_NAME:-unnamed-project}
echo "Project name: $PROJECT_NAME (raw: $RAW_NAME)"
```

### Step 3: Determine the Feature Name

The feature name comes from the user's request. Derive a short, hyphenated slug from whatever feature name the user provides. For example:

- "User Authentication System" → `user-authentication`
- "Shopping Cart" → `shopping-cart`
- "API Rate Limiting" → `api-rate-limiting`

Apply the same sanitization rules as step 1. Store this as `FEATURE_NAME`.

### Step 4: Check for Existing PRD or Context

Before generating tasks, check if there's an existing PRD or prior notes for this feature. The `VAULT_PATH` was already determined in Step 1:

```bash
# Check for existing PRD
if [ -f "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/${FEATURE_NAME}/prd-${FEATURE_NAME}.md" ]; then
  echo "Found existing PRD:"
  cat "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/${FEATURE_NAME}/prd-${FEATURE_NAME}.md"
fi

# Check for any other notes in this feature folder
ls "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/${FEATURE_NAME}/" 2>/dev/null

# Check for project-level notes
ls "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/"*.md 2>/dev/null
```

Read any existing PRD or notes and use them as context when generating tasks. If a PRD exists, the task list should directly map to the PRD's requirements.

### Step 5: Generate Parent Tasks

Create high-level tasks. **IMPORTANT: Always include task 0.0 "Create feature branch" as the first task** (unless explicitly requested otherwise). Use short, appropriate branch names.

Present the parent tasks to the user and say: "I have generated the high-level tasks based on your requirements. Ready to generate the sub-tasks? Respond with 'Go' to proceed."

**Wait for the user to respond with "Go" before continuing.**

### Step 6: Generate Sub-Tasks

Break down each parent task into smaller, actionable sub-tasks. Identify files that need to be created or modified and list them under the `Relevant Files` section.

### Step 7: Ensure the Pi Notes Directory Structure Exists

The task list will be saved at: `Pi Notes/{PROJECT_NAME}/{FEATURE_NAME}/tasks-{FEATURE_NAME}.md`

Create the necessary directories. The `VAULT_PATH` was already determined in Step 1:

```bash
mkdir -p "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/${FEATURE_NAME}"
```

### Step 8: Save the Task List

Write the task list file with YAML frontmatter and cross-link to the PRD if it exists:

```bash
cat > "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/${FEATURE_NAME}/tasks-${FEATURE_NAME}.md" << 'EOF'
---
date: YYYY-MM-DD
tags: [pi-notes, tasks, {PROJECT_NAME}, {FEATURE_NAME}]
project: "{PROJECT_NAME}"
feature: "{FEATURE_NAME}"
related:
  - "[[Pi Notes/{PROJECT_NAME}/{FEATURE_NAME}/prd-{FEATURE_NAME}]]"
---

# Tasks: {Feature Title}

... task content ...
EOF
```

Set the `related` wikilink to point to the PRD file. If no PRD exists yet, omit the `related` field or leave it empty.

Also, if a PRD exists for this feature, update its frontmatter to add a backlink:

```bash
# Add a related link to the PRD pointing to this tasks file
# (append or update the related list in prd-[feature].md frontmatter)
```

Verify the file was written:

```bash
obsidian read path="Pi Notes/${PROJECT_NAME}/${FEATURE_NAME}/tasks-${FEATURE_NAME}.md" 2>/dev/null | head -5
```

If the Obsidian CLI can't find it (may need a moment to re-index), fall back to:

```bash
cat "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/${FEATURE_NAME}/tasks-${FEATURE_NAME}.md" | head -5
```

### Step 9: Confirm to the User

Tell the user:

1. The task list has been created
2. The Obsidian path: `Pi Notes/{PROJECT_NAME}/{FEATURE_NAME}/tasks-{FEATURE_NAME}.md`
3. Whether a PRD was found and used as context
4. A brief summary of the task breakdown

## Output Format

The generated task list _must_ follow this structure:

```markdown
---
date: YYYY-MM-DD
tags: [pi-notes, tasks, {PROJECT_NAME}, {FEATURE_NAME}]
project: "{PROJECT_NAME}"
feature: "{FEATURE_NAME}"
related:
  - "[[Pi Notes/{PROJECT_NAME}/{FEATURE_NAME}/prd-{FEATURE_NAME}]]"
---

# Tasks: {Feature Title}

## Relevant Files

- `path/to/file` - Brief description of purpose.
- `path/to/file.test.ts` - Unit tests for `file.ts`.

### Notes

- Unit tests should typically be placed alongside the code files they are testing.
- Use `npx jest [optional/path/to/test/file]` to run tests.

## Instructions for Completing Tasks

**IMPORTANT:** As you complete each task, you must check it off by changing `- [ ]` to `- [x]`.

Example:
- `- [ ] 1.1 Read file` → `- [x] 1.1 Read file` (after completing)

## Tasks

- [ ] 0.0 Create feature branch
  - [ ] 0.1 Create and checkout a new branch (e.g., `git checkout -b feature/[feature-name]`)
- [ ] 1.0 Parent Task Title
  - [ ] 1.1 [Sub-task description]
```

## Target Audience

The primary reader is a **junior developer** implementing the feature. Provide clear, actionable steps.

## Error Handling

- **Obsidian not running**: Follow the Setup instructions to launch it. If it still won't start after 20 seconds, suggest the user open the app manually. Task lists can still be written to the filesystem.
- **Task list already exists**: If `tasks-{FEATURE_NAME}.md` already exists, read it first and ask the user whether to overwrite or create a new version (e.g., `tasks-{FEATURE_NAME}-v2.md`). Never silently overwrite.
- **Empty project/feature name**: Handled by sanitization fallback (`unnamed-project`).
- **No PRD found**: Proceed without PRD context, but note this in the task list's frontmatter (omit the `related` field).