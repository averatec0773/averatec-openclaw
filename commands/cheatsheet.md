# Command Cheatsheet

## Built-in Slash Commands

| Command | Description |
|---------|-------------|
| `/help` | Show help |
| `/clear` | Clear conversation history |
| `/compact` | Compact conversation to save context |
| `/cost` | Show token usage and cost |
| `/status` | Show current model and context info |
| `/model` | Switch model |
| `/fast` | Toggle fast mode (Opus 4.6 faster output) |
| `/memory` | Edit global memory (CLAUDE.md) |
| `/init` | Initialize CLAUDE.md in current project |
| `/review` | Review recent changes |
| `/pr_comments` | Review PR comments |
| `/bug` | Report a bug |

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Enter` | Send message |
| `Shift+Enter` | New line |
| `Ctrl+C` | Cancel current operation |
| `Ctrl+L` | Clear screen |
| `↑` / `↓` | Navigate message history |
| `Esc` | Exit current mode / cancel |

## Permission Modes

```bash
# Run with auto-approve for all tools
claude --dangerously-skip-permissions

# Allow specific tools without prompt
# Configure in settings.json under "permissions"
```

## Context Management

```bash
/compact          # Summarize conversation, keep context
/clear            # Wipe conversation entirely
```
