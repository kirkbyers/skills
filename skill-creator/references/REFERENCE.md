# Agent Skills Specification Reference

This document provides the detailed specification for creating skills compliant with the [Agent Skills standard](https://agentskills.io/specification).

## Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Max 64 chars. Lowercase a-z, 0-9, hyphens. Must match parent directory. |
| `description` | Yes | Max 1024 chars. What this skill does and when to use it. |
| `license` | No | License name or reference to bundled file. |
| `compatibility` | No | Max 500 chars. Environment requirements. |
| `metadata` | No | Arbitrary key-value mapping. |
| `allowed-tools` | No | Space-delimited list of pre-approved tools (experimental). |
| `disable-model-invocation` | No | When `true`, skill is hidden from system prompt. Users must use `/skill:name`. |

## Name Rules

- 1–64 characters
- Lowercase letters (`a-z`), numbers (`0-9`), and hyphens (`-`) only
- No leading or trailing hyphens
- No consecutive hyphens
- Must match parent directory name

**Valid examples:**
- `pdf-processing`
- `data-analysis`
- `code-review`
- `brave-search`

**Invalid examples:**
- `PDF-Processing` (uppercase)
- `-pdf` (leading hyphen)
- `pdf--processing` (consecutive hyphens)
- `my skill` (spaces)
- `my_skill` (underscores)

## Description Guidelines

The description is the most important field. It determines when the agent decides to load the skill via progressive disclosure.

**Requirements:**
- Maximum 1024 characters
- Must describe *what* the skill does
- Must describe *when* to use it (trigger conditions)

**Good descriptions:**
```yaml
description: Extracts text, tables, and images from PDF files. Use when the user wants to read, parse, or extract content from any PDF document.
```
```yaml
description: Creates and manages GitHub pull requests including drafting, updating, and merging. Use when the user mentions PR, pull request, merge request, or wants to submit code changes for review.
```
```yaml
description: Performs web searches and extracts page content using the Brave Search API. Use for finding documentation, looking up facts, or fetching any web content.
```

**Poor descriptions:**
```yaml
description: Helps with PDFs.
```
```yaml
description: A utility for GitHub operations.
```
```yaml
description: Web search skill.
```

## Skill Directory Structure

```
my-skill/
├── SKILL.md              # Required: frontmatter + instructions
├── scripts/              # Optional: helper scripts
│   └── process.sh
├── references/           # Optional: detailed docs loaded on-demand
│   └── api-reference.md
└── assets/               # Optional: templates, configs
    └── template.json
```

### SKILL.md Format

```markdown
---
name: my-skill
description: What this skill does and when to use it. Be specific.
---

# My Skill

## Setup

Run once before first use:
\`\`\`bash
cd /path/to/skill && npm install
\`\`\`

## Usage

\`\`\`bash
./scripts/process.sh <input>
\`\`\`
```

### Key Design Principles

1. **Progressive disclosure**: Only the name and description are always in context. The full SKILL.md is loaded on-demand. Keep the main instructions focused; put deep reference content in `references/`.

2. **Relative paths**: Always use relative paths from the skill directory when referencing scripts, assets, or other files.

3. **Setup section**: If the skill requires any installation or configuration, include a clear **Setup** section as the first section after the heading.

4. **Command examples**: Show exact command invocations with example inputs and expected outputs.

5. **Error handling**: Document common failure modes and how the agent should respond.

## Skill Discovery Locations

Pi discovers skills from:

| Location | Scope | Behavior |
|----------|-------|----------|
| `~/.pi/agent/skills/` | Global | Root `.md` files discovered as individual skills; directories with `SKILL.md` discovered recursively |
| `~/.agents/skills/` | Global | Root `.md` files ignored; directories with `SKILL.md` discovered |
| `.pi/skills/` | Project | Root `.md` files discovered as individual skills; directories with `SKILL.md` discovered recursively |
| `.agents/skills/` | Project | Root `.md` files ignored; directories with `SKILL.md` discovered |
| `skills/` in packages | Package | Via `package.json` `pi.skills` entry or `skills/` directory |
| Settings `skills` array | Config | Explicit file or directory paths |
| `--skill` CLI flag | Invocation | Explicit paths, additive even with `--no-skills` |

## Validation Checklist

When creating or reviewing a skill, verify:

- [ ] SKILL.md exists in the skill directory
- [ ] Frontmatter is valid YAML between `---` delimiters
- [ ] `name` field is present and valid (lowercase, hyphens, ≤64 chars, matches directory)
- [ ] `description` field is present (≤1024 chars, describes what and when)
- [ ] All relative links in SKILL.md point to existing files
- [ ] Scripts have correct shebangs (`#!/usr/bin/env bash`, `#!/usr/bin/env node`, etc.)
- [ ] Setup instructions are clear if dependencies are needed
- [ ] Commands use relative paths from the skill directory

## Common Patterns

### Skill with Helper Script
```
my-skill/
├── SKILL.md
└── scripts/
    └── process.sh
```

SKILL.md references: `./scripts/process.sh`

### Skill with Reference Documentation
```
my-skill/
├── SKILL.md
└── references/
    └── api-reference.md
```

SKILL.md links: `[API Reference](references/api-reference.md)`

### Skill with Templates
```
my-skill/
├── SKILL.md
├── scripts/
│   └── generate.sh
└── assets/
    └── config-template.json
```

SKILL.md references: `./assets/config-template.json`