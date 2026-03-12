# Rule: Writing linear tickets from Datadog links

Start an investigation using the datadog mcp given a link to datadog.

## Goal

To draft a Linear ticket for a given issue from the datalink. Diagnose for the underlying issue in the codebase, figure out why the issue is happening now, and propose potential solutions to the issue.

## Process

1. Use the datadog mcp to find the relivant data for the given datadog link
2. Diagnose issues
3. Find out why the issues are happening now
4. Find potential solutions
5. Look through the git history and find who would
6. Use the linear mcp to check if there's a pre-existing issue that covers your findings
7. If there is not a pre-existing open issue, use the linear mcp to write a ticket
8. If there is a pre-existing issue, add additional missing, helpful information from your investigation.

## Output

A linear ticket that contains what the issue is, what's it impact, why its happening now, potential solutions, and who might have the most context.

**OR**

Additional information add to the ticket description or as a comment. Helpful information includes but is not limited to what the issue is, what's it impact, why its happening now, potential solutions, and who might have the most context.

**OR**

Simply link the ticket when there's already at least 1 ticket that fully documents the issue.

## Out of Scope

- **DO NOT** Assign tickets to anyone.
- **DO NOT** Change ticket status. New tickets should be left in the TODO state.

## Taget Audience

Engineers with limited context. Keep the tone casual but professional. Assume technical knowledge with limited detailed context.
