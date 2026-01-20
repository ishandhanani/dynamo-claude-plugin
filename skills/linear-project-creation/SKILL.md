---
name: linear-project-creation
description: Create and refine Linear projects through structured questioning and exploration
---

# Linear Project Creation

Create or refine a Linear project through structured dialogue and codebase exploration. This skill requires Linear MCP.

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

### Step 2: Question & Explore Loop

This step repeats until both Claude and the user are satisfied with the gathered context.

#### 2a: Ask Questions

Ask clarifying questions in batches of 2-3:

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

#### 2b: Explore

Based on the user's answers, gather relevant context:

**Codebase exploration:**
- Read files in the relevant area of the codebase
- Look for existing patterns or similar implementations
- Check tests to understand expected behavior
- Identify interfaces or contracts that matter

**Linear exploration:**
- Fetch related Linear projects or issues
- Check for prior work or dependencies
- Look at how similar features were scoped

**Summarize findings:**
After exploring, briefly summarize what you learned and how it affects the spec.

#### 2c: Check Satisfaction

Ask the user:
> "Based on what we've discussed and what I found, do you have more context to add? Or is there anything else I should explore?"

**Exit conditions (move to Step 3 when either is met):**
- **User signals done**: "That's enough" / "Looks good" / "Let's write it up"
- **Claude is confident**: "I think I have enough context to write a solid spec - ready to proceed?"

If neither condition is met, loop back to 2a with more targeted questions based on gaps identified.

### Step 3: Update Linear Project

After the loop completes, update the Linear project description with:

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

## Context Gathered
[Brief summary of relevant code, patterns, or related issues discovered during exploration]
```

Use Linear MCP to update the project description directly.

### Step 4: Confirm

- Show the user the updated project description
- Provide the Linear project URL
- Ask if they want to proceed to creating tickets (`linear-project-to-tickets`)

## Behavior Notes

- Always use Linear MCP - no fallback to local markdown
- Ask questions in batches of 2-3, not all at once
- Explore proactively - don't wait to be asked
- Keep exploration focused - read relevant files, not the entire codebase
- Summarize findings concisely before asking for more input
- The loop should feel collaborative, not interrogative
- Always ask about target release for labeling
