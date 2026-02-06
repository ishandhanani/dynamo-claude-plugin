---
name: pr-create
description: Create a PR with standardized team format
user-invocable: true
disable-model-invocation: true
---

# Create Pull Request

## Current State

!`git branch --show-current`

!`git remote show origin 2>/dev/null | grep 'HEAD branch' | cut -d' ' -f5`

!`git log --oneline $(git merge-base HEAD origin/$(git remote show origin 2>/dev/null | grep 'HEAD branch' | cut -d' ' -f5))..HEAD 2>/dev/null`

!`git diff --stat origin/$(git remote show origin 2>/dev/null | grep 'HEAD branch' | cut -d' ' -f5)...HEAD 2>/dev/null`

## Instructions

Based on the branch, commits, and diff above:

1. **Push** if not already pushed:
   - Check if the branch has an upstream: `git rev-parse --abbrev-ref @{upstream} 2>/dev/null`
   - If no upstream: `git push -u origin <branch>`
   - If upstream exists: `git push`

2. **Determine PR title**:
   - If single commit: use the commit message
   - If multiple commits: summarize the change concisely
   - Keep under 70 characters

3. **Check for Linear ticket references**:
   - Look in branch name for patterns like `DYN-123`, `LIN-123`
   - Look in commit messages for ticket references
   - Include in Context section if found

4. **Create PR** using `gh pr create` with this exact template:

```
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
- [Bullet point 1: what changed]
- [Bullet point 2: if needed]

## Context
[Why this change was needed]

[Linear ticket link if found, e.g., "Closes DYN-123"]

## Testing
- [ ] [Step to verify the change works]

## Checklist
- [ ] Human has reviewed AI code
- [ ] Relevant context for this fix has been tracked in this issue and/or in associated linear ticket for future debugging
- [ ] Code follows project conventions
EOF
)"
```

5. **Execute** without asking for confirmation

6. **Output** the PR URL when done
