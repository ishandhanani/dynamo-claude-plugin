---
name: linear-project-creation
description: Create and refine Linear projects through structured questioning
---

# Linear Project Creation

Create or refine a Linear project through structured dialogue. This skill requires Linear MCP.

## Requirements

- Linear MCP must be configured and authenticated
- User must have access to create/edit projects in Linear

## Process

### Step 1: Get or Create Linear Project

Ask the user:
- **Existing project**: Provide the Linear project ID to refine
- **New project**: Provide a name and team, Claude will create it

If creating new:
1. Use Linear MCP to create the project
2. Return the project ID to the user

If existing:
1. Fetch the project via Linear MCP
2. Read current description and status

### Step 2: Structured Questioning

Ask clarifying questions in batches of 2-3 to build out the project:

**Problem & Goals**
- What problem does this solve?
- What does success look like?
- How will we measure it?

**Scope**
- What's explicitly in scope?
- What's explicitly out of scope?
- What's deferred to future iterations?

**Requirements**
- What are the must-haves?
- What are the nice-to-haves?
- Are there performance requirements?

**Dependencies & Risks**
- What does this depend on?
- What are the known risks?

**Release Planning**
- Which release is this targeting? (e.g., 0.7.1, 0.9.0)
- Are there dependencies on other releases?

### Step 3: Update Linear Project

After gathering answers, update the Linear project description with:

```
## Problem Statement
[1-2 sentences on what problem this solves]

## Goals
- [Concrete, measurable goal]

## Non-Goals
- [Explicitly out of scope]

## Requirements

### Must Have
- [Requirement with acceptance criteria]

### Nice to Have
- [Lower priority]

## Technical Approach
[High-level approach, not implementation details]

## Success Metrics
- [How we measure success]

## Target Release
[Release label, e.g., "0.7.1"]

## Risks & Dependencies
- [Known risks and dependencies]
```

Use Linear MCP to update the project description directly.

### Step 4: Confirm

- Show the user the updated project description
- Provide the Linear project URL
- Ask if they want to proceed to creating tickets (`linear-project-to-tickets`)

## Behavior Notes

- Always use Linear MCP - no fallback to local markdown
- Ask questions in batches of 2-3, not all at once
- Prioritize questions that unblock implementation
- Keep projects focused - don't over-engineer
- Always ask about target release for labeling
