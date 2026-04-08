---
name: create-prd
description: Generates a detailed Product Requirements Document (PRD) in Markdown format based on user prompts. Includes a process for asking clarifying questions.
---

# Create PRD

## Process

1.  **Receive Initial Prompt:** The user provides a brief description or request for a new feature or functionality.
2.  **Ask Clarifying Questions:** Before writing the PRD, the AI *must* ask only the most essential clarifying questions needed to write a highly clear PRD. Limit questions to 3-5 critical gaps in understanding. Use letter/number lists for easy response.
3.  **Generate PRD:** Based on the initial prompt and the user's answers to the clarifying questions, generate a PRD using the structure outlined below.
4.  **Save PRD:** Save the generated document as `prd-[feature-name].md` inside the `/tasks` directory.

## Clarifying Questions (Guidelines)

Ask only the most critical questions. Focus on areas where the initial prompt is ambiguous or missing essential context. Common areas:

*   **Problem/Goal:** "What problem does this feature solve for the user?"
*   **Core Functionality:** "What are the key actions a user should be able to perform?"
*   **Scope/Boundaries:** "Are there any specific things this feature *should not* do?"
*   **Success Criteria:** "How will we know when this feature is successfully implemented?"

**Formatting Requirements:**
- **Number all questions** (1, 2, 3, etc.)
- **List options as A, B, C, D, etc.**
- **Simple response format:** e.g., "1A, 2C, 3B"

## PRD Structure

The generated PRD should include:

1.  **Introduction/Overview:** Describe the feature and the problem it solves.
2.  **Goals:** List specific, measurable objectives.
3.  **User Stories:** Detail user narratives.
4.  **Functional Requirements:** List specific functionalities. Use clear language (e.g., "The system must allow users to upload a profile picture.").
5.  **Non-Goals (Out of Scope):** Clearly state what this feature will *not* include.
6.  **Design Considerations (Optional):** Link to mockups or UI/UX requirements.
7.  **Technical Considerations (Optional):** Mention constraints or dependencies.
8.  **Success Metrics:** How will success be measured? (e.g., "Increase engagement by 10%").
9.  **Open Questions:** List any remaining questions.

## Target Audience

Assume the primary reader is a **junior developer**. Requirements should be explicit, unambiguous, and avoid jargon.

## Output

*   **Format:** Markdown (`.md`)
*   **Location:** `/tasks/`
*   **Filename:** `prd-[feature-name].md`

## Final instructions

1.  Do NOT start implementing the PRD immediately.
2.  Ensure you ask the user clarifying questions first.
3.  Use the user's answers to refine the PRD.
