---
name: version-migration
description: Break up large version migration PRs into smaller, reviewable chunks by analyzing module dependencies, validating migration justifications, and generating PR documentation
---

# Version Migration

Help developers break up large version migration PRs into smaller, reviewable chunks. This skill analyzes module dependencies, validates migration justifications, and generates PR documentation with Linear integration.

## When to Use This Skill

- Migrating a new implementation to replace existing code
- Breaking large refactors into incremental PRs
- Need dependency analysis to determine PR order
- Want validated justifications for each migration step

## Requirements

- Git repository with clean working directory
- Linear MCP configured (optional, for ticket creation)
- GitHub CLI (`gh`) for PR creation

## State File

All progress is persisted in `.version-migration.json` in the repo root. This allows resuming work across sessions.

---

## Phase 1: Initialization

### First Run

If `.version-migration.json` doesn't exist, gather initial configuration:

**1. Source Directory (new code)**
Ask: "What directory contains the new code you're migrating FROM?"
- This is the new implementation to be incrementally introduced
- Example: `lib/kvbm/src/v2/`

**2. Target Directory (old code being replaced)**
Ask: "What directory contains the old code being REPLACED?"
- This is the existing code that will eventually be deprecated
- Example: `lib/llm/src/block_manager/`

**3. Linear Project (optional)**
Ask: "Do you have a Linear project for this migration? If so, provide the project ID or URL. Or say 'skip' to handle Linear later."
- If provided, validate via Linear MCP
- If skipping, can add later

**4. Git Configuration**
Ask: "What base branch should PRs target? (default: main)"
Ask: "What branch prefix for migration PRs? (default: migration/)"

**5. Initialize State**
- Scan both directories
- Create `.version-migration.json` with initial structure
- Report: "Found X modules in source, Y modules in target"

### Resuming

If `.version-migration.json` exists:

1. Load existing state
2. Show progress summary:
   ```
   Migration: lib/kvbm/src/v2/ → lib/llm/src/block_manager/
   Progress: 3/12 modules mapped, 2/5 PRs created, 0 merged
   Current phase: planning
   ```
3. Offer options:
   - **Continue**: Resume from current phase
   - **Re-analyze**: Rescan modules (keeps mappings/justifications)
   - **Show plan**: Display current PR plan
   - **Start fresh**: Delete state and reinitialize

---

## Phase 2: Module Discovery & Dependency Analysis

### 2.1 Parse Module Structure

For each directory (source and target):

1. **Find all mod.rs files**
   ```
   Glob: **/*.rs
   ```

2. **Identify module boundaries**
   - Directories with `mod.rs` = module
   - Standalone `.rs` files = module
   - Track: path, file count, approximate LOC

3. **Parse imports**
   For each `.rs` file, extract:
   - `use crate::` statements (internal dependencies)
   - `use super::` statements (parent module access)
   - `pub use` re-exports (public API)

### 2.2 Build Dependency Graph

Create a directed acyclic graph (DAG):

1. **Nodes**: Each module
2. **Edges**: Module A → Module B if A imports from B

Example output:
```
logical/blocks: [] (leaf - no dependencies)
logical/pools: [logical/blocks]
distributed/leader: [logical/pools, logical/blocks]
```

### 2.3 Identify Coupling Issues

Flag problematic patterns:

- **Bidirectional dependencies**: A imports B, B imports A
  - Suggest: Combine into single PR or extract shared types

- **Circular dependencies**: A → B → C → A
  - Suggest: Identify cycle, recommend refactoring order

- **High fan-in**: Module imported by many others
  - Note: Must be migrated early, needs thorough review

### 2.4 Compute Topological Order

Sort modules for migration order (leaves first):

1. Modules with no dependencies → migrate first
2. Modules depending only on migrated modules → next
3. Continue until all modules ordered

Save to state:
```json
"analysis": {
  "topological_order": ["logical/blocks", "logical/pools", ...],
  "coupling_issues": [...]
}
```

---

## Phase 3: Module Mapping Interview

For each discovered module in the **source** directory, determine its relationship to target:

### Interview Flow

Present modules in topological order:

```
Module 1 of 12: v2/logical/blocks
Files: mod.rs, mutable.rs, immutable.rs, state.rs (4 files, ~800 LOC)
Dependencies: none (leaf module)

Q: Does this replace existing code in lib/llm/src/block_manager/?
   Suggested mappings based on name:
   - block_manager/block/ (similar name)
   - block_manager/block/data/logical/ (path similarity)

   Options:
   1. Accept suggestion: block_manager/block/data/logical/
   2. Different mapping: [specify path]
   3. New functionality (no replacement)
   4. Skip for now
```

### Mapping Types

Record the relationship:
- **replaces**: Direct replacement (1:1)
- **partially_replaces**: Covers some functionality
- **new**: No existing equivalent
- **consolidates**: Combines multiple old modules

### Update State

```json
"modules": {
  "logical/blocks": {
    "old_code_mapping": {
      "mapped_to": "block_manager/block/data/logical",
      "mapping_type": "replaces"
    }
  }
}
```

---

## Phase 4: Justification Interview (Critical Step)

For each mapped module, capture **why** this migration is necessary. This creates the "Motivation" section of PR documentation.

### 4.1 Problem Question

```
Module: v2/logical/blocks
Mapped to: block_manager/block/data/logical

Q: What problem does the new implementation solve that the old code doesn't?
   (Be specific about limitations, bugs, or architectural issues)
```

Wait for developer response.

### 4.2 Improvements Question

```
Q: What are the key improvements in the new implementation?
   Consider: Performance, type safety, testability, maintainability,
   correctness, API ergonomics, error handling
```

Wait for developer response.

### 4.3 Validation

**Analyze both codebases** to verify claims:

1. Read relevant files from both source and target
2. Look for evidence supporting/contradicting claims
3. Provide validation feedback:

```
Validation Results:

You mentioned: "Type-safe state transitions - compile-time errors instead of runtime panics"

I analyzed both implementations:
- Old (block_manager/block/state.rs):
  - Uses enum State { Free, Allocated, Pinned }
  - State transitions via match statements
  - Found 3 panic!() calls for invalid state access (lines 45, 78, 112)

- New (v2/logical/blocks/):
  - Uses generic Block<T, State> with PhantomData markers
  - State transitions return new typed values
  - No panic sites for state violations (compile-time enforcement)

✓ VALIDATED: Your claim is accurate. The new code eliminates runtime state
  validation panics through type-state pattern.

---

You mentioned: "Better performance through zero-copy operations"

I analyzed both implementations:
- Old: Uses Vec<u8> clones in transfer() method (line 89)
- New: Uses &[u8] references, no allocations found

✓ VALIDATED: Confirmed zero-copy pattern in new implementation.
```

### 4.4 Handle Discrepancies

If claims don't match analysis:

```
⚠ NEEDS CLARIFICATION:

You mentioned: "Eliminates memory leaks"

I couldn't find evidence of memory leaks in the old code:
- Old code uses RAII patterns correctly
- No manual memory management found
- Valgrind reports show clean runs (if available)

Can you clarify:
1. What specific leak scenario does the new code address?
2. Is this perhaps about logical resource leaks (not memory)?
3. Should we revise this justification?
```

Continue until justification is validated or revised.

### 4.5 Store Justification

```json
"justification": {
  "problem_solved": "Runtime state checks can panic on invalid access...",
  "improvements": [
    "Type-safe state transitions (compile-time enforcement)",
    "Zero-copy block transfers",
    "Cleaner async integration"
  ],
  "validated": true,
  "validation_notes": "Confirmed 3 panic sites eliminated via type-state pattern..."
}
```

---

## Phase 5: Interactive PR Planning

### 5.1 Present Plan

Show modules in topological order with full context:

```
═══════════════════════════════════════════════════════════════
PR PLAN: Module Migration Order
═══════════════════════════════════════════════════════════════

PR 1: logical/blocks (LEAF - can merge independently)
├── Files: 4 files, ~800 LOC
├── Dependencies: none
├── Dependents: logical/pools, distributed/leader
├── Replaces: block_manager/block/data/logical
├── Justification: Type-safe state transitions, zero-copy transfers
└── Status: planned

PR 2: logical/pools
├── Files: 3 files, ~600 LOC
├── Dependencies: PR 1 (logical/blocks)
├── Dependents: distributed/leader, distributed/workers
├── Replaces: block_manager/pool/
├── Justification: Lock-free allocation, better NUMA awareness
└── Status: planned

... (continue for all modules)
```

### 5.2 Allow Adjustments

```
Options:
1. Accept plan as-is
2. Combine modules into single PR (specify which)
3. Split a module into multiple PRs
4. Reorder (if dependencies allow)
5. Add notes/requirements to specific PRs
```

### 5.3 Validate Plan

Before finalizing:
- All dependencies satisfied (can't merge B before A if B depends on A)
- No circular cross-PR dependencies
- Each PR is independently testable

### 5.4 Save Plan

```json
"prs": [
  {
    "id": "pr-001",
    "modules": ["logical/blocks"],
    "order": 1,
    "status": "planned",
    "branch_name": null,
    "pr_url": null,
    "linear_ticket_id": null,
    "notes": ""
  }
]
```

---

## Phase 6: PR Documentation Generation

For each planned PR, generate comprehensive documentation.

### Read Template

Read the PR template from `references/pr-template.md` in this skill directory.

### Generate Content

Fill template with:

1. **Summary**: Module path and one-line description
2. **Motivation**: From validated justification
3. **Comparison Table**: New module → old module mapping
4. **Key Improvements**: Bulleted list from justification
5. **Dependencies**: Checklist of required prior PRs
6. **Files Changed**: Collapsible list with LOC
7. **Testing**: Test commands specific to this module
8. **Linear Link**: If ticket exists

### Store Documentation

Save generated markdown to state for later use:

```json
"prs": [{
  "id": "pr-001",
  "documentation_md": "## Summary\n\nMigration PR for..."
}]
```

---

## Phase 7: Branch/PR Creation

When user is ready to create a PR:

### 7.1 Pre-flight Checks

1. Verify clean git status (no uncommitted changes)
2. Verify on correct base branch
3. Verify dependencies are merged (if any)

### 7.2 Create Branch

```bash
git checkout -b migration/<module-name>
# Example: git checkout -b migration/logical-blocks
```

### 7.3 Stage Files

Stage only the relevant module files:

```bash
git add lib/kvbm/src/v2/logical/blocks/
```

### 7.4 Create Commit

```bash
git commit -m "feat: add v2 logical blocks module

Migration step 1/N: Introduces type-safe block state management.

Replaces: lib/llm/src/block_manager/block/data/logical/"
```

### 7.5 Push and Create PR

```bash
git push -u origin migration/logical-blocks
gh pr create --title "Migration: v2/logical/blocks" --body "<generated documentation>"
```

### 7.6 Update State

```json
"prs": [{
  "id": "pr-001",
  "status": "created",
  "branch_name": "migration/logical-blocks",
  "pr_url": "https://github.com/org/repo/pull/123"
}]
```

---

## Phase 8: Linear Ticket Creation

If Linear project is configured:

### 8.1 Create Issue

Use Linear MCP to create issue:
- **Team**: From project
- **Project**: Migration project
- **Title**: "Migration: <module-name>"
- **Description**: PR summary + acceptance criteria
- **Labels**: Apply migration labels if defined

### 8.2 Link to PR

Add Linear ticket URL to PR description (edit via `gh pr edit`).

### 8.3 Update State

```json
"prs": [{
  "linear_ticket_id": "LIN-123",
  "linear_ticket_url": "https://linear.app/team/issue/LIN-123"
}]
```

---

## Interactive Commands

Throughout the workflow, the user can:

- **status**: Show current progress and phase
- **plan**: Display current PR plan
- **skip**: Skip current module/step
- **back**: Go back one step
- **reset**: Start fresh (with confirmation)
- **help**: Show available commands

---

## Error Handling

### Circular Dependencies

```
⚠ Circular dependency detected:
  module_a → module_b → module_c → module_a

Suggestions:
1. Combine all three into a single PR
2. Extract shared types to a new module (migrate first)
3. Refactor to break the cycle before migrating

Which approach would you like to take?
```

### Git State Issues

```
⚠ Cannot proceed: Uncommitted changes detected

Please either:
1. Commit your changes: git commit -am "WIP"
2. Stash your changes: git stash
3. Discard changes: git checkout . (destructive!)

Then run the command again.
```

### Invalid Justifications

```
⚠ Justification needs revision

I couldn't validate: "Fixes memory corruption bug"
- No memory corruption patterns found in either codebase
- Both use safe Rust with no unsafe blocks

Please provide:
1. More specific description of the issue
2. Link to related bug report/issue
3. Or revise the justification
```

### Linear Failures

```
⚠ Linear ticket creation failed: API error

Options:
1. Retry
2. Skip Linear integration (can link manually later)
3. Check Linear MCP configuration
```

---

## State File Schema

```json
{
  "version": "1.0.0",
  "migration": {
    "source_dir": "lib/kvbm/src/v2/",
    "target_dir": "lib/llm/src/block_manager/",
    "base_branch": "main",
    "branch_prefix": "migration/",
    "linear_project_id": "proj_abc123",
    "created_at": "2025-01-22T10:00:00Z",
    "updated_at": "2025-01-22T15:30:00Z"
  },
  "modules": {
    "<module_path>": {
      "path": "logical/blocks",
      "files": ["mod.rs", "mutable.rs", "immutable.rs", "state.rs"],
      "loc": 800,
      "dependencies": [],
      "dependents": ["logical/pools", "distributed/leader"],
      "old_code_mapping": {
        "mapped_to": "block_manager/block/data/logical",
        "mapping_type": "replaces"
      },
      "justification": {
        "problem_solved": "Runtime state checks can panic...",
        "improvements": ["Type-safe state transitions", "Zero-copy transfers"],
        "validated": true,
        "validation_notes": "Confirmed 3 panic sites eliminated..."
      },
      "coupling_issues": []
    }
  },
  "analysis": {
    "topological_order": ["logical/blocks", "logical/pools", "..."],
    "coupling_issues": []
  },
  "prs": [
    {
      "id": "pr-001",
      "modules": ["logical/blocks"],
      "order": 1,
      "status": "planned|created|merged",
      "branch_name": null,
      "pr_url": null,
      "linear_ticket_id": null,
      "linear_ticket_url": null,
      "documentation_md": "## Summary\n...",
      "notes": ""
    }
  ],
  "progress": {
    "phase": "initialization|discovery|mapping|justification|planning|execution",
    "current_module_index": 0,
    "modules_analyzed": 12,
    "modules_total": 24,
    "modules_mapped": 8,
    "modules_justified": 5,
    "prs_planned": 5,
    "prs_created": 2,
    "prs_merged": 0
  }
}
```

---

## Example Session

```
User: /dynamo:version-migration

Claude: Starting version migration workflow.

No existing state found. Let's set up the migration.

Q: What directory contains the NEW code you're migrating from?

User: lib/kvbm/src/v2/

Claude: Q: What directory contains the OLD code being replaced?

User: lib/llm/src/block_manager/

Claude: Scanning directories...

Found:
- Source (new): 12 modules in lib/kvbm/src/v2/
- Target (old): 8 modules in lib/llm/src/block_manager/

Q: Do you have a Linear project for this migration?
   Provide project ID/URL, or say 'skip'.

User: skip

Claude: Q: Base branch for PRs? (default: main)

User: main

Claude: Q: Branch prefix? (default: migration/)

User: kvbm-v2/

Claude: Configuration saved. Analyzing dependencies...

Dependency Analysis Complete:

Leaf modules (no dependencies, can migrate first):
1. logical/blocks - 4 files, ~800 LOC
2. identity/ - 2 files, ~200 LOC

Mid-tier modules:
3. logical/pools - depends on [logical/blocks]
4. distributed/workers - depends on [logical/blocks, identity]

Top-tier modules:
5. distributed/leader - depends on [logical/pools, distributed/workers]

No circular dependencies detected.

Proceeding to module mapping...

---

Module 1/12: logical/blocks
Files: mod.rs, mutable.rs, immutable.rs, state.rs

Q: Does this replace existing code?
   Suggestions: block_manager/block/, block_manager/block/data/logical/

User: block_manager/block/ - the whole thing

Claude: Recorded: logical/blocks replaces block_manager/block/ (full replacement)

Q: What problem does the new implementation solve?

User: The old code uses runtime enum matching for state transitions which can panic.
      The new code uses type-state pattern for compile-time safety.

Claude: Analyzing both codebases to validate...

✓ VALIDATED:
- Old code (block_manager/block/state.rs): Found 3 panic!() calls on lines 45, 78, 112
- New code (logical/blocks/): Uses Block<T, State> generics, no panics for state errors

Justification recorded.

Q: Any other improvements to note?

User: Zero-copy transfers and async-native API

Claude: Analyzing...

✓ VALIDATED:
- Old: Vec<u8>::clone() in transfer() method
- New: Uses references and Pin<Box<>> for zero-copy

Proceeding to next module...

[... continues through all modules ...]

---

Ready to generate PR plan?

User: yes

Claude:
═══════════════════════════════════════════════════════════════
PR PLAN
═══════════════════════════════════════════════════════════════

PR 1: logical/blocks
├── Replaces: block_manager/block/
├── Reason: Type-safe state transitions, zero-copy
└── Can merge: independently

PR 2: identity/
├── Replaces: (new functionality)
└── Can merge: independently

PR 3: logical/pools
├── Replaces: block_manager/pool/
├── Depends on: PR 1
└── Can merge: after PR 1

[...]

Accept this plan? (yes/adjust)

User: yes

Claude: Plan saved. Ready to create PRs.

To create the first PR:
> create pr 1

To see generated documentation:
> show docs 1
```

---

## Behavior Notes

- **Always validate justifications** - don't accept claims without checking code
- **Preserve state frequently** - save after each significant step
- **Be conservative with PR scope** - smaller is better for review
- **Linear is optional** - full functionality without it
- **Git safety first** - never force push, always check status
- **Module isolation** - each PR should be independently testable
