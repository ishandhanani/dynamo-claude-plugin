---
name: linear-project-to-tickets
description: Convert a Linear project into child tickets with verification steps
---

# Linear Project to Tickets

Convert a Linear project into actionable tickets. This skill requires Linear MCP.

## Requirements

- Linear MCP must be configured and authenticated
- A Linear project must exist (use `linear-project-creation` first if needed)

## Philosophy

> "The project defines the END STATE. The agent/developer decides HOW to get there."

Focus on:
- **What** needs to exist when done
- **How to verify** it's correct
- NOT implementation details

## Process

### Step 1: Fetch Linear Project

Ask the user for the Linear project ID, then:
1. Fetch the project via Linear MCP
2. Read the project description
3. Understand the team and project structure
4. Get available labels (especially release labels)

### Step 2: Identify Target Release

Ask the user which release these tickets should be tagged with (e.g., "Dynamo 0.9.0").

Linear uses **nested labels** - release labels are under parent groups:
- Example: "Dynamo Releases > Dynamo 0.9.0"
- Fetch available labels from the team and look for the release label group first
- If the release label does not exist, create it under the appropriate parent group

### Step 3: Extract Tickets

From the project description, identify discrete units of work:
- Each task = one Linear issue
- Each task should have clear verification criteria
- Tasks ordered by dependency (blockers first)

**Granularity guidelines:**
- Each ticket should take 1-4 hours of focused work
- If a ticket feels like it would take a full day, break it down further
- If a ticket feels like it would take 30 minutes, combine it with a related task
- Rust core changes and Python binding updates can be separate tickets if they are independently testable
- "Setup" tickets (scaffolding, config) should be separate from "implementation" tickets

**Dependency ordering:**
1. Infrastructure / scaffolding tickets first
2. Core implementation tickets (ordered by data flow)
3. Integration / wiring tickets
4. Testing / validation tickets
5. Documentation / cleanup tickets (if needed)

### Step 4: Create Linear Issues

For each ticket, create a Linear issue with:

**Title**: Concise action in imperative form (e.g., "Implement per-user rate limiting")

**Description**:
```
## What
[One sentence on what this accomplishes]

## Acceptance Criteria
- [ ] [Specific, verifiable condition - not "works correctly" but "returns 429 when limit exceeded"]
- [ ] [Another condition]

## Verification Steps
[How to actually test this - concrete commands or checks]
1. Build: `cd lib/bindings/python && maturin develop --uv && cd ../../.. && uv pip install -e .`
2. Run: [specific example or test command]
3. Verify: [what to check in the output]

## Notes
[Any context, dependencies, or constraints]
[Reference to specific files or modules if helpful]
```

**Metadata**:
- Link to parent project
- **Add release label** (nested: "Dynamo Releases > Dynamo 0.9.0")
- Set labels based on category (setup, implementation, integration, testing)
- Add blocking relationships if dependencies exist
- Set priority based on dependency order (blockers get higher priority)

### Step 5: Confirm with User

Before creating tickets:
1. Show the list of issues to be created (title + acceptance criteria)
2. Show the release label that will be applied
3. Show blocking relationships
4. Ask for confirmation
5. Create all issues via Linear MCP
6. Return the issue IDs/URLs

## Guidelines

- **Granularity**: 1-4 hours per ticket, not days
- **Independence**: Minimize coupling between tickets where possible
- **Verification focus**: Every ticket must have concrete verification steps - if you cannot verify it, the ticket is not well-defined
- **No implementation details**: Do not specify *how* to code it, specify *what* it should do
- **Always tag release**: Every ticket must have a release label
- **Concrete acceptance criteria**: Not "handles errors" but "returns HTTP 503 with JSON body when backend is unavailable"

## Example

Project description snippet:
> "Add rate limiting to the API. Should limit to 100 requests per minute per user."

Created Linear issue:
- **Title**: Implement per-user rate limiting
- **Labels**: `Dynamo Releases > Dynamo 0.7.1`, `implementation`
- **Description**:
  ```
  ## What
  Add rate limiting middleware that restricts API calls to 100/min per authenticated user

  ## Acceptance Criteria
  - [ ] Make 100 requests in <60s with same user token -> all succeed
  - [ ] Make 101st request -> returns 429 Too Many Requests with retry-after header
  - [ ] Wait 60s, make another request -> succeeds
  - [ ] Different user token is not affected by first user's limit
  - [ ] Rate limit state survives server restart (or document if it does not)

  ## Verification Steps
  1. Build: `cd lib/bindings/python && maturin develop --uv && cd ../../.. && uv pip install -e .`
  2. Start server: `bash examples/backends/sglang/launch/agg.sh`
  3. Run: `for i in $(seq 1 101); do curl -s -o /dev/null -w "%{http_code}\n" -H "Authorization: Bearer test-token" http://localhost:8000/v1/chat/completions -d '{"model":"test","messages":[{"role":"user","content":"hi"}]}'; done`
  4. Verify: First 100 return 200, 101st returns 429

  ## Notes
  Rate limiting should be implemented in the Rust preprocessing layer for performance.
  ```
