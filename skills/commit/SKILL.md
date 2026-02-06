---
name: commit
description: Stage and commit changes with standardized format
user-invocable: true
disable-model-invocation: true
---

# Standardized Commit

## Current State

!`git status --short`

!`git diff --stat`

!`git diff --cached --stat`

## Instructions

Based on the git status and diff above:

1. **Stage changes** if not already staged:
   - Stage relevant files (use judgment on what belongs together)
   - Do not stage unrelated changes
   - Do not stage files that likely contain secrets (.env, credentials, tokens)

2. **Write commit message** in this format:
   ```
   <type>: <description>
   ```

   **Types**:
   - `fix`: Bug fix
   - `feat`: New feature
   - `refactor`: Code restructuring (no behavior change)
   - `docs`: Documentation only
   - `test`: Adding/updating tests
   - `chore`: Build, config, tooling changes

   **Rules**:
   - Description is lowercase, no period at the end
   - Keep under 72 characters total
   - Be specific: not "fix bug" but "fix null pointer in request handler"
   - Use imperative mood: "add feature" not "added feature"

3. **Commit** without asking for confirmation. Use a HEREDOC for the message:
   ```bash
   git commit -m "$(cat <<'EOF'
   <type>: <description>
   EOF
   )"
   ```

## Examples

Good:
- `fix: handle empty response in sglang backend`
- `feat: add per-user rate limiting middleware`
- `refactor: extract token validation to separate module`
- `docs: update sglang disaggregation deployment guide`
- `chore: bump maturin version to 1.8`

Bad:
- `Fix bug` (too vague, wrong case)
- `Updated code` (meaningless, past tense)
- `WIP` (not a complete commit)
- `fix: fix the thing.` (period, vague)
