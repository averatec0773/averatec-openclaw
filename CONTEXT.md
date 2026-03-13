# OpenClaw Setup Context — averatec

> Docs: https://docs.openclaw.ai/start/getting-started | https://docs.openclaw.ai/install/hetzner

This repository is a knowledge base for understanding and managing a self-hosted OpenClaw instance.
Read this file first to understand the environment.

---

## What is OpenClaw

OpenClaw is a self-hosted AI gateway that connects messaging channels (Discord, WhatsApp, Telegram, iMessage)
to AI coding agents. It runs as a single Docker container on a VPS.

Official docs: https://docs.openclaw.ai

---

## Infrastructure

| Component | Value |
|---|---|
| VPS provider | Hetzner (~$5/month, Ubuntu) |
| SSH alias | `openclaw` (configured in `~/.ssh/config`) |
| Working directory on server | `~/openclaw/` |
| Gateway port | `18789` (loopback only, access via SSH tunnel) |
| Control UI | `http://127.0.0.1:18789/` after tunnel |

---

## Docker Setup

| Image | Purpose |
|---|---|
| `openclaw:latest` | Upstream image from Docker Hub |
| `openclaw:averatec-custom` | Custom image built on top via `Dockerfile.custom` |

`Dockerfile.custom` adds: `clawhub` (npm global), `gh` (GitHub CLI), `gog` (Gmail CLI, steipete/gogcli), `goplaces` (Google Places CLI).

The custom image name is controlled by `OPENCLAW_IMAGE=openclaw:averatec-custom` in `~/openclaw/.env` on the server.
The `.env` is gitignored by the upstream repo — upstream `git pull` will never overwrite it.

### Container management

```bash
ssh openclaw
cd ~/openclaw

docker compose build          # rebuild custom image layer
docker compose up -d          # start / recreate (picks up .env changes)
docker compose down           # stop
docker compose restart        # soft restart (picks up openclaw.json changes only)
docker compose logs -f openclaw-gateway
```

### When to restart

| Change | Action |
|---|---|
| Edit `openclaw.json` | `docker compose restart` |
| Edit `.env` | `docker compose up -d` (recreates container) |
| Rebuild image (`Dockerfile.custom`) | `docker compose build && docker compose up -d` |

---

## Volume Mapping

| Container path | Host path | Persistent |
|---|---|---|
| `/home/node/.openclaw/` | `/root/.openclaw/` | Yes — config, credentials |
| `/home/node/.openclaw/skills/` | `/root/.openclaw/skills/` | Yes — user-installed skills |
| `/home/node/.openclaw/workspace/` | `/root/.openclaw/workspace/` | Yes — agent context files |
| `/app/skills/` | *(none)* | No — baked into image |

---

## OpenClaw Configuration

Config file on server: `~/.openclaw/openclaw.json`
Template (secrets redacted): [config/openclaw.json](config/openclaw.json)

Key settings in use:
- `agents.defaults.model.primary` → `openai/gpt-5.4`
- `skills.load.extraDirs` → `["/home/node/.openclaw/skills"]` (makes user skills auto-loaded)
- `channels.discord` → enabled, `groupPolicy: allowlist`, `streaming: partial`
- `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` → `true` (required for SSH tunnel access)

After editing `openclaw.json`: `docker compose restart`

---

## Authenticated Services

| Service | Account | Storage | Notes |
|---|---|---|---|
| Google (gog) | set via `GOG_ACCOUNT` in `.env` | `/home/node/.openclaw/gogcli/keyring/` | File keyring, `GOG_KEYRING_PASSWORD` in `.env` |
| Google Places (goplaces) | — | — | API key, `GOOGLE_PLACES_API_KEY` in `.env` |
| GitHub (gh) | `averatec0773` | `/home/node/.openclaw/gh/hosts.yml` | Token-based |
| ClawHub | `averatec0773` | `/home/node/.openclaw/clawhub/config.json` | Token in `.env` |
| Tavily Search | — | — | API key, `TAVILY_API_KEY` in `openclaw.json` env section |

Re-auth process for gog: [notes/gog.md](notes/gog.md)

---

## Web Search & Networking

Three ways the bot can access the internet:

| Method | Tool | Requires | Notes |
|---|---|---|---|
| `web_search` | Tavily/Brave Search API | `TAVILY_API_KEY` in `openclaw.json` | Best for open-ended queries |
| `web_fetch` | Built-in HTTP fetch | Nothing | Requires known URL; may fail on JS-heavy pages |
| `browser` | Chromium (headless) | Chromium in image or remote browser service | Not configured on VPS |

**After adding `TAVILY_API_KEY` to `openclaw.json`:** run `docker compose down && docker compose up -d` (full recreate required — `docker compose restart` is not enough for env changes).

Tavily free tier: 1000 queries/month, no credit card required. Sign up at tavily.com.

---

## Active Channels

| Channel | Status | Notes |
|---|---|---|
| Discord | Active | Guild: AVERATEC-WORKPLACE, `requireMention: false` |

---

## Multi-Agent Setup

Two agents running on the same gateway instance, each with its own Discord bot and workspace:

| Agent ID | Model | Workspace | Scope |
|---|---|---|---|
| `main` | `openai/gpt-5.4` | `workspace/` | Owner DMs (via bot A) |
| `public` | `openai/gpt-5-mini` | `workspace-public/` | Guild messages (via bot B) |

**Routing:** `bindings` array in `openclaw.json`, matched by `accountId` — each Discord bot account routes to its agent.

```json
{ "agentId": "main",   "match": { "channel": "discord", "accountId": "private" } },
{ "agentId": "public", "match": { "channel": "discord", "accountId": "public" } }
```

**`public` agent restrictions:**
- `tools.deny: ["exec", "bash", "computer"]` — no shell access
- No USER.md, no MEMORY.md in workspace — no personal context loaded
- Model: `gpt-5-mini` (cheaper for public traffic)

**Directory structure:**
```
/root/.openclaw/
├── workspace/            ← main agent workspace
├── workspace-public/     ← public agent workspace
│   ├── SOUL.md           ← public persona, mentions averatec as creator
│   └── AGENTS.md         ← simplified rules, no MEMORY.md loading
└── agents/
    ├── main/sessions/
    └── public/sessions/
```

---

## Installed Skills

| Skill | Version | Source | Path |
|---|---|---|---|
| `self-improving-agent` | 3.0.0 | clawhub | `/home/node/.openclaw/skills/` |
| `discord` | 1.0.1 | clawhub (steipete) | `/home/node/.openclaw/skills/` |
| `averatec-discord` | 1.0.0 | averatec-skills | `/home/node/.openclaw/skills/` |
| `averatec-email` | 1.0.0 | averatec-skills | `/home/node/.openclaw/skills/` |

Built-in skills (image layer, `/app/skills/`): 1password, apple-notes, canvas, clawhub, coding-agent,
discord, gemini, gh-issues, github, healthcheck, notion, obsidian, openai-image-gen, openai-whisper,
slack, spotify-player, summarize, tmux, trello, weather, and more.

### Install a new skill (persistent)

```bash
docker compose exec --user root openclaw-gateway \
  clawhub install <slug> --workdir /home/node/.openclaw --dir skills
```

---

## Access

SSH tunnel (run on local machine):
```bash
ssh -N -L 18789:127.0.0.1:18789 openclaw
```
Then open: `http://127.0.0.1:18789/`

Direct server access:
```bash
ssh openclaw
```
