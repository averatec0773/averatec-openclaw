# Settings & Configuration Reference

## Global Settings

Location: `~/.claude/settings.json`

```json
{
  "model": "claude-opus-4-6",
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(npm *)"
    ],
    "deny": []
  },
  "hooks": {
    "PreToolUse": [...],
    "PostToolUse": [...],
    "Notification": [...]
  }
}
```

## Project Settings

Location: `.claude/settings.json` (per-project, overrides global)

## Permission Rules

Permissions use glob-style patterns on tool calls:

```json
"allow": [
  "Bash(git *)",        // allow all git commands
  "Bash(npm run *)",    // allow npm run only
  "Read",               // allow all file reads
  "Write(**/*.md)"      // allow writing markdown only
]
```

## Hooks

Hooks run shell commands at specific lifecycle events:

| Event | Description |
|-------|-------------|
| `PreToolUse` | Before any tool call |
| `PostToolUse` | After any tool call |
| `Notification` | When Claude sends a notification |
| `Stop` | When Claude finishes responding |

### Hook Example

```json
"hooks": {
  "PostToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        { "type": "command", "command": "echo 'Bash tool used'" }
      ]
    }
  ]
}
```

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | API key (alternative to login) |
| `CLAUDE_MODEL` | Default model override |
