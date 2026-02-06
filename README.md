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

### 3. Set up Linear MCP

The Linear skills require Linear MCP. Set it up at: https://mcp.linear.app

Or add manually:
```bash
claude mcp add --transport http linear https://mcp.linear.app/mcp
```

## Skills

### User-Invocable (slash commands)

| Skill | Command | Description |
|-------|---------|-------------|
| commit | `/dynamo:commit` | Stage and commit with standardized format (`<type>: <description>`) |
| pr-create | `/dynamo:pr-create` | Create PR with team template and Linear ticket references |
| debug-session | `/dynamo:debug-session` | Start debugging session with worklog file |
| dynamo-explore | `/dynamo:dynamo-explore` | Explore Dynamo codebase architecture and trace component interactions |
| release-check | `/dynamo:release-check` | Check Dynamo/SGLang version compatibility before releases |

### Auto-Activating

| Skill | Triggers when... |
|-------|------------------|
| linear-project-creation | Creating or refining Linear projects |
| linear-project-to-tickets | Breaking down Linear projects into tickets |
| tool-parser-generator | Adding tool calling support for HuggingFace models |

**Note:** Linear skills require Linear MCP - no offline fallback.

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
- [ ] [Step to verify the change works]

## Checklist
- [ ] Human has reviewed AI code
- [ ] Relevant context for this fix has been tracked in this issue and/or in associated linear ticket for future debugging
- [ ] Code follows project conventions
```

## Workflows

### Development Workflow

1. **Explore** (`/dynamo:dynamo-explore`): Understand the relevant codebase area
2. **Implement**: Make changes
3. **Test** (`/live-server-test`): Launch servers and send test traffic
4. **Commit** (`/dynamo:commit`): Standardized commit
5. **PR** (`/dynamo:pr-create`): Create PR with template

### Project Planning Workflow

1. **Create project** (`linear-project-creation`): Create or refine a Linear project through structured questioning
2. **Create tickets** (`linear-project-to-tickets`): Break the project into Linear tickets with acceptance criteria and release labels
3. **Implement**: Each ticket has clear verification steps

### Debug Workflow

`/dynamo:debug-session` starts a structured debugging session:

1. **Get bug report** - from Linear ticket, GitHub issue, or paste
2. **Discover environment** - runs `nvidia-smi`, checks python, reads user's CLAUDE.md
3. **Create worklog** - `<issue>.md` file to track investigation
4. **Set up testing** - points to examples in `/home/ubuntu/dynamo/examples/backends/`
5. **Begin investigation** - reproduce first, minimal changes, document findings

### Release Workflow

`/dynamo:release-check` verifies compatibility before releases:

1. Gather version info from Dynamo and SGLang
2. Check API compatibility (ConfigArgumentMerger, argument parsing)
3. Verify Python binding compatibility
4. Run integration smoke test
5. Report findings with recommendation

## Updating

```bash
/plugin update dynamo@dynamo-dev
```

## Requirements

- Claude Code CLI
- Linear MCP (see step 3)
- GitHub CLI (`gh`) for PR creation

## License

MIT
