# Tips & Tricks

## CLAUDE.md

Place a `CLAUDE.md` in the project root — Claude loads it automatically as project context.
Use it to document project conventions, architecture, and rules you want Claude to always follow.

```bash
claude /init    # auto-generate a CLAUDE.md for your project
```

## Memory

Global memory lives at `~/.claude/CLAUDE.md` — persists across all sessions.
Use `/memory` to edit it inline, or edit the file directly.

## One-shot Mode

```bash
claude -p "fix all TypeScript errors in src/"
```

Useful for scripting or CI pipelines.

## Continue Last Conversation

```bash
claude --continue
```

## Context Window Tips

- Use `/compact` when the conversation gets long — it summarizes history and frees up tokens
- Use `/clear` to start completely fresh
- Keep CLAUDE.md focused — long CLAUDE.md files consume context on every message

## Sub-agents

For complex multi-step tasks, Claude spins up sub-agents automatically.
Use the `Agent` tool in custom prompts to delegate work explicitly.

## MCP Servers

Claude Code supports MCP (Model Context Protocol) servers for extended tool access.
Configure in `~/.claude/settings.json` under `"mcpServers"`.
