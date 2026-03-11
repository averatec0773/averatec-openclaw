<div align="center">
  <img src="assets/logo.jpg" alt="OpenClaw Logo" width="160" />
  <h1>averatec-openclaw</h1>
  <p>Self-hosted OpenClaw AI gateway — knowledge base for LLM-assisted management</p>
</div>

---

Read [CONTEXT.md](CONTEXT.md) first — full environment overview, paths, credentials, and current state.

## Structure

```
averatec-openclaw/
├── CONTEXT.md               # LLM entry point: environment, setup, current state
├── Dockerfile.custom        # Custom image definition (openclaw:averatec-custom)
├── config/
│   └── openclaw.json        # openclaw.json template (secrets redacted)
├── installation/
│   └── setup.md             # Full deployment guide (Hetzner VPS, Docker, initial config)
├── skills/
│   └── clawhub.md           # ClawHub CLI reference + Docker container usage
├── notes/
│   ├── docker.md            # Image management, skill install, volume reference
│   ├── discord.md           # Discord channel config
│   ├── gog.md               # Google OAuth setup (headless server)
│   └── ssh.md               # SSH alias and tunnel
└── assets/
    └── logo.jpg             # Project logo
```

## What's inside

- **Custom Docker image** on top of `openclaw:latest`, adding `clawhub`, `gh`, `gog` (Gmail/Calendar), `goplaces`
- **Discord integration** via guild allowlist, no mention required
- **Web search** via Tavily API
- **Google Workspace** access via OAuth (gog) — Gmail, Calendar, Drive
- **Google Places** search via `goplaces` CLI

> Secrets live in `~/openclaw/.env` and `~/.openclaw/openclaw.json` on the server — never committed.
