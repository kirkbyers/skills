---
name: generate-tasks
description: Creates a step-by-step task list in Markdown format from user requirements, including parent tasks, sub-tasks, and relevant files.
---

# Generate Tasks

## Process

1.  **Receive Requirements:** The user provides a feature request, task description, or points to existing documentation.
2.  **Analyze Requirements:** Analyze the functional requirements, user needs, and implementation scope.
3.  **Phase 1: Generate Parent Tasks:** Create high-level tasks. **IMPORTANT: Always include task 0.0 "Create feature branch" as the first task (unless explicitly requested otherwise).** Use short, appropriate branch names. Present these to the user. Inform the user: "I have generated the high-level tasks based on your requirements. Ready to generate the sub-tasks? Respond with 'Go' to proceed."
4.  **Wait for Confirmation:** Pause and wait for the user to respond with "Go".
5.  **Phase 2: Generate Sub-Tasks:** Break down each parent task into smaller, actionable sub-tasks.
6.  **Identify Relevant Files:** Identify files that need to be created or modified. List them under the `Relevant Files` section.
7.  **Generate Final Output:** Combine parent tasks, sub-tasks, relevant files, and notes into the final Markdown structure.
8.  **Save Task List:** Save the document in the `/tasks/` directory with the filename `tasks-[feature-name].md`.

## Output Format

The generated task list _must_ follow this structure:

```markdown
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
