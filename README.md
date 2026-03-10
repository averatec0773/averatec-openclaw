# averatec-openclaw

Knowledge base for a self-hosted OpenClaw instance. Primarily intended for LLM reading
to quickly understand the environment and assist with management tasks.

**Start here:** [CONTEXT.md](CONTEXT.md) — full environment overview, paths, and current state.

---

## Structure

```
averatec-openclaw/
├── CONTEXT.md               # LLM entry point: environment, setup, current state
├── Dockerfile.custom        # Custom image definition (openclaw:averatec-custom)
├── config/
│   └── openclaw.json        # openclaw.json template (secrets redacted)
├── installation/
│   └── setup.md             # Full setup guide (Hetzner VPS, Docker, initial config)
├── skills/
│   └── clawhub.md           # ClawHub CLI reference + Docker container usage
└── notes/
    ├── docker.md            # Image management, skill install, volume reference
    ├── discord.md           # Discord channel config options
    ├── gog.md               # Google OAuth setup (headless server)
    └── ssh.md               # SSH alias and tunnel for server access
```
