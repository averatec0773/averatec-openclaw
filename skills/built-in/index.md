# Built-in Skills Reference

Built-in skills ship with Claude Code and are invoked via `/skill-name`.

## Available Built-in Skills

| Skill | Trigger | Description |
|-------|---------|-------------|
| `commit` | `/commit` | Stage, draft, and create a git commit |
| `review-pr` | `/review-pr <number>` | Review a GitHub pull request |
| `simplify` | `/simplify` | Review changed code for quality/efficiency |
| `loop` | `/loop <interval> <command>` | Run a command on a recurring interval |
| `statusline-setup` | `/statusline-setup` | Configure the status line |
| `claude-api` | auto-triggered | Help building apps with Anthropic SDK |
| `claude-code-guide` | auto-triggered | Answer Claude Code usage questions |

## How Skills Work

Skills are Markdown files stored in:
- `~/.claude/commands/` — global skills
- `.claude/commands/` — project-level skills

A skill file is a `.md` file whose filename becomes the slash command.
Example: `~/.claude/commands/standup.md` → `/standup`

Skills support **argument passing** via `$ARGUMENTS` in the prompt body.
