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

---

## OpenClaw Gateway Config

Location on server: `~/.openclaw/openclaw.json`

Template (secrets redacted): [config/openclaw.json](openclaw.json)

### Key Fields

| Field | Description |
|---|---|
| `agents.defaults.model.primary` | Default model for agents |
| `skills.load.extraDirs` | Extra directories for skill discovery |
| `channels.discord.groupPolicy` | Guild access policy (`allowlist` / `open`) |
| `channels.discord.streaming` | Streaming mode (`partial` / `block` / `off`) |
| `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` | Allow SSH tunnel access |

### Installing Custom Skills (Persistent)

Skills installed to `~/.openclaw/skills/` are auto-loaded via `extraDirs`:

```bash
docker compose exec --user root openclaw-gateway \
  clawhub install <slug> --workdir /home/node/.openclaw --dir skills
```
