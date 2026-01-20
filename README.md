# dynamo-claude-plugin

Claude Code plugin for Dynamo/SGLang development workflows with Linear integration.

## Installation

### 1. Add the marketplace

```bash
/plugin marketplace add https://github.com/ishandhanani/dynamo-claude-plugin
```

### 2. Install the plugin

```bash
/plugin install workflow-tools@dynamo-dev
```

### 3. Set up Linear MCP (included)

The plugin includes Linear MCP config. On first use of Linear features, you'll be prompted to authenticate via OAuth.

To manually add Linear MCP:
```bash
claude mcp add --transport http linear https://mcp.linear.app/mcp
```

## Commands

| Command | Description |
|---------|-------------|
| `/workflow-tools:commit` | Stage and commit with standardized format (`<type>: <description>`) |
| `/workflow-tools:pr-create` | Create PR with team template |
| `/workflow-tools:debug-session` | Start debugging session with worklog file |

## Skills (Auto-activate)

| Skill | Triggers when... |
|-------|------------------|
| `spec-refine` | Discussing requirements, specs, or project planning |
| `spec-to-tasks` | Breaking down specs into implementable tasks |

Both skills integrate with Linear MCP for issue management.

## Commit Format

```
<type>: <description>
```

**Types:**
- `fix` - Bug fix
- `feat` - New feature
- `refactor` - Code restructuring
- `docs` - Documentation
- `test` - Tests
- `chore` - Build/config

## PR Template

```markdown
## Summary
- [What changed]

## Context
[Why this change was needed]
[Linear ticket: DYN-123]

## Testing
- [ ] Verification steps

## Checklist
- [ ] Tests pass
- [ ] No performance regression
- [ ] Code follows project conventions
```

## Spec Workflow

1. **Refine spec** (`spec-refine`): Discuss requirements, Claude asks clarifying questions, syncs to Linear
2. **Convert to tasks** (`spec-to-tasks`): Break spec into Linear tickets with acceptance criteria
3. **Implement**: Each ticket has clear verification steps

## Debug Workflow

`/workflow-tools:debug-session` creates a structured worklog:

1. Creates `<issue>.md` file
2. Sets up reproduction-first debugging approach
3. Includes rebuild commands:
   - Dynamo: `cd lib/bindings/python && maturin develop --uv && cd ../../.. && uv pip install -e .`
   - SGLang: `uv pip install -e "python"`

## Updating

```bash
/plugin update workflow-tools@dynamo-dev
```

## Requirements

- Claude Code CLI
- Node.js (for Linear MCP via npx)
- GitHub CLI (`gh`) for PR creation

## License

MIT
