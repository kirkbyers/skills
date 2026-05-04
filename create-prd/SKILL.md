---
name: create-prd
description: Generates a detailed Product Requirements Document (PRD) in Obsidian under Pi Notes. Use when the user wants to create a PRD, product requirements document, or requirements spec for a feature or project. Saves to Pi Notes/[project]/[feature]/prd-[feature].md.
---

# Create PRD

Generates a PRD and saves it into the project's Pi Notes folder in Obsidian, organized by feature.

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

Apply the same sanitization rules as step 1 (strip leading dots, replace spaces with hyphens, remove special characters, collapse hyphens). Store this as `FEATURE_NAME`.

### Step 4: Ask Clarifying Questions

Before writing the PRD, ask only the most essential clarifying questions needed. Limit to 3–5 critical gaps in understanding. Use letter/number lists for easy response.

Common areas:
- **Problem/Goal:** "What problem does this feature solve?"
- **Core Functionality:** "What are the key actions a user should be able to perform?"
- **Scope/Boundaries:** "What should this feature *not* do?"
- **Success Criteria:** "How will we know this is successfully implemented?"

**Format:** Number all questions (1, 2, 3), list options as A, B, C, D. Simple response format: e.g., "1A, 2C, 3B".

**Wait for the user's answers before proceeding.**

### Step 5: Ensure the Pi Notes Directory Structure Exists

The PRD will be saved at: `Pi Notes/{PROJECT_NAME}/{FEATURE_NAME}/prd-{FEATURE_NAME}.md`

Create the necessary directories using the filesystem (the Obsidian CLI has issues with nested path creation):

```bash
# The vault path was already determined in Step 1
mkdir -p "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/${FEATURE_NAME}"
```

If the "Pi Notes" folder doesn't yet exist in Obsidian, also create a placeholder so it appears:

```bash
obsidian folders 2>/dev/null | grep -q "^Pi Notes$" || touch "${VAULT_PATH}/Pi Notes/.obsidian"
```

### Step 6: Generate the PRD

Based on the initial prompt and the user's answers, generate a PRD using this structure:

1. **Introduction/Overview:** Describe the feature and the problem it solves.
2. **Goals:** List specific, measurable objectives.
3. **User Stories:** Detail user narratives.
4. **Functional Requirements:** List specific functionalities. Use clear language (e.g., "The system must allow users to upload a profile picture.").
5. **Non-Goals (Out of Scope):** Clearly state what this feature will *not* include.
6. **Design Considerations (Optional):** Link to mockups or UI/UX requirements.
7. **Technical Considerations (Optional):** Mention constraints or dependencies.
8. **Success Metrics:** How will success be measured?
9. **Open Questions:** List any remaining questions.

Add YAML frontmatter with `date`, `tags`, `project`, and `feature` fields. Cross-link to the tasks file if it exists using a wikilink.

### Step 7: Save the PRD

Write the PRD file:

```bash
cat > "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/${FEATURE_NAME}/prd-${FEATURE_NAME}.md" << 'EOF'
---
date: YYYY-MM-DD
tags: [pi-notes, prd, {PROJECT_NAME}, {FEATURE_NAME}]
project: "{PROJECT_NAME}"
feature: "{FEATURE_NAME}"
related:
  - "[[Pi Notes/{PROJECT_NAME}/{FEATURE_NAME}/tasks-{FEATURE_NAME}]]"
---

# PRD: {Feature Title}

... PRD content ...
EOF
```

Verify the file was written:

```bash
obsidian read path="Pi Notes/${PROJECT_NAME}/${FEATURE_NAME}/prd-${FEATURE_NAME}.md" 2>/dev/null | head -5
```

If the Obsidian CLI can't find it (it may need a moment to re-index), fall back to:

```bash
cat "${VAULT_PATH}/Pi Notes/${PROJECT_NAME}/${FEATURE_NAME}/prd-${FEATURE_NAME}.md" | head -5
```

### Step 8: Confirm to the User

Tell the user:

1. The PRD has been created
2. The Obsidian path: `Pi Notes/{PROJECT_NAME}/{FEATURE_NAME}/prd-{FEATURE_NAME}.md`
3. A brief summary of what the PRD covers

**Do NOT start implementing the PRD immediately.** Wait for the user's direction.

## Target Audience

Assume the primary reader is a **junior developer**. Requirements should be explicit, unambiguous, and avoid jargon.

## Error Handling

- **Obsidian not running**: Follow the Setup instructions to launch it. If it still won't start after 20 seconds, suggest the user open the app manually. PRDs can still be written to the filesystem.
- **Feature name conflicts**: If `prd-{FEATURE_NAME}.md` already exists, read it first and ask the user whether to overwrite or append. Never silently overwrite.
- **Empty project/feature name**: Handled by sanitization fallback (`unnamed-project`).