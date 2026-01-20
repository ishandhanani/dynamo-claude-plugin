# dynamo-claude-plugin

Claude Code plugin for Dynamo development workflows with Linear integration.

## Installation

### 1. Add the marketplace

```bash
/plugin marketplace add https://github.com/ishandhanani/dynamo-claude-plugin
```

### 2. Install the plugin

```bash
/plugin install dynamo@dynamo-dev
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
| `/dynamo:commit` | Stage and commit with standardized format (`<type>: <description>`) |
| `/dynamo:pr-create` | Create PR with team template |
| `/dynamo:debug-session` | Start debugging session with worklog file |

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

`/dynamo:debug-session` starts a structured debugging session:

1. **Get bug report** - from Linear ticket, GitHub issue, or paste
2. **Discover environment** - runs `nvidia-smi`, checks python, reads user's CLAUDE.md
3. **Create worklog** - `<issue>.md` file to track investigation
4. **Set up testing** - points to examples in `/home/ubuntu/dynamo/examples/backends/` (sglang, vllm, trtllm)
5. **Begin investigation** - reproduce first, minimal changes, document findings

### Recommended: Add Dev Environment to Your CLAUDE.md

For best results with `/dynamo:debug-session`, add a dev environment section to your personal `~/.claude/CLAUDE.md`:

```markdown
## Dev Environment

- **VM**: 4xL40s GPU (or describe your setup)
- **Dynamo**: /home/ubuntu/dynamo
- **SGLang**: /home/ubuntu/sglang
- **venv**: dynamo (default)
- **Build alias**: `build` runs full Dynamo rebuild
```

This helps Claude understand your specific setup without re-discovering it each session.

## Updating

```bash
/plugin update dynamo@dynamo-dev
```

## Requirements

- Claude Code CLI
- Node.js (for Linear MCP via npx)
- GitHub CLI (`gh`) for PR creation

## Future Plugins

This marketplace (`dynamo-dev`) can host multiple plugins. Potential additions:

- **`router`** - Router-specific debugging and testing workflows
- **`onboarding`** - New team member setup and codebase orientation
- **`prefix-benchmark`** - Prefix caching benchmark workflows

To add a new plugin, create a directory and add an entry to `.claude-plugin/marketplace.json`.

## License

MIT
