# Agent Skills Repository

This directory contains a collection of self-contained capability packages (Skills) designed for the [pi agent harness](https://agentskills.io/). Each skill is a specialized workflow that the agent can load on-demand to perform specific tasks like generating PRDs, creating task lists, or opening Pull Requests.

## 🛠 Available Skills

| Skill Name | Description |
| :--- | :--- |
| `create-prd` | Generates detailed Product Requirements Documents (PRD) in Markdown, including a structured process for clarifying requirements. |
| `generate-tasks` | Converts requirements into actionable, step-by-step Markdown task lists with parent/sub-tasks and relevant file tracking. |
| `open-pr` | Automates the creation of draft Pull Requests on GitHub with professional and contextual descriptions. |

---

## 🚀 Creating New Skills

When adding a new skill to this repository, follow the [Agent Skills Specification](https://agentskills.io/specification).

### 1. Directory Structure
Each skill must reside in its own directory named after the skill itself.

```text
skill-name/
├── SKILL.md              # Required: contains frontmatter and instructions
├── scripts/              # (Optional) Helper scripts (e.g., shell, python, node)
├── references/           # (Optional) Detailed documentation or templates
└── assets/               # (Optional) Static assets like JSON templates or images
```

### 2. `SKILL.md` Format
The `SKILL.md` file is the entry point. It must contain valid YAML frontmatter and clear instructions.

#### Frontmatter Requirements
| Field | Required | Constraint | Description |
| :--- | :--- | :--- | :--- |
| `name` | **Yes** | 1-64 chars, lowercase, hyphens only | **Must match parent directory name.** |
| `description`| **Yes** | Max 1024 chars | A specific description of what the skill does to help the agent discover it. |
| `license` | No | String | The license for the skill. |

#### Content Structure
Use Markdown headers to organize the skill's logic:
- `# Skill Name`: A clear title.
- `## Setup`: Instructions for any one-time configuration (e.g., `npm install`).
- `## Process` or `## Usage`: Step-by-step instructions for the agent to follow.
- `## Output`: Describe the expected resulting file format or location.

### 3. Best Practices

- **Be Specific in Descriptions**: Avoid "Helps with X". Instead, use "Extracts data from Y and formats it as Z. Use when working with Y files." This improves the agent's ability to trigger the skill.
- **Target a Junior Developer**: Write instructions as if the person executing them is a junior developer. Be explicit and avoid ambiguity.
- **Use Relative Paths**: Always reference scripts or reference files using paths relative to the skill's directory.
- **Validate Naming**: Ensure `name` in frontmatter does not have leading/trailing hyphens or consecutive hyphens (e.g., `my-skill`, not `-my--skill`).

## 📋 Checklist for New Skills
- [ ] Directory name is lowercase, hyphenated, and matches the `name` in `SKILL.md`.
- [ ] `SKILL.md` has a `name` and `description` in the frontmatter.
- [ ] The `description` is descriptive enough for the agent to know *when* to use it.
- [ ] The `Process` section provides clear, actionable steps.
- [ ] All paths within the skill are relative to the skill directory.
