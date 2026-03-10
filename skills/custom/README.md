# Custom Skills

Custom skills (slash commands) are Markdown files placed in:
- `~/.claude/commands/` — available globally across all projects
- `.claude/commands/` — available only in the current project

## File Format

```markdown
# Skill Title

Describe what this skill does and any instructions for Claude.
Use $ARGUMENTS to pass user-provided arguments into the prompt.

Example: fix the bug in $ARGUMENTS
```

## Naming Convention

The filename (without `.md`) becomes the slash command:
- `~/.claude/commands/deploy.md` → `/deploy`
- `~/.claude/commands/standup.md` → `/standup`

## My Custom Skills

| File | Command | Description |
|------|---------|-------------|
| *(add your skills here)* | | |

## Template

```markdown
Describe the task you want Claude to perform.

$ARGUMENTS
```

Save as `~/.claude/commands/<your-command>.md` and use `/your-command [args]`.
