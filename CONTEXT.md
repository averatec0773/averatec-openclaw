# OpenClaw Setup Context — averatec

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
docker compose up -d          # start
docker compose down           # stop
docker compose restart        # restart (picks up openclaw.json changes)
docker compose logs -f openclaw-gateway
```

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
- `agents.defaults.model.primary` → `openai/gpt-5-mini`
- `skills.load.extraDirs` → `["/home/node/.openclaw/skills"]` (makes user skills auto-loaded)
- `channels.discord` → enabled, `groupPolicy: allowlist`, `streaming: partial`
- `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` → `true` (required for SSH tunnel access)

After editing `openclaw.json`: `docker compose restart`

---

## Authenticated Services

| Service | Account | Storage | Notes |
|---|---|---|---|
| Google (gog) | `ayetek0773@gmail.com` | `/home/node/.openclaw/gogcli/keyring/` | File keyring, `GOG_KEYRING_PASSWORD` in `.env` |
| GitHub (gh) | `averatec0773` | `/home/node/.openclaw/gh/hosts.yml` | Token-based |
| ClawHub | `averatec0773` | `/home/node/.openclaw/clawhub/config.json` | Token in `.env` |

Re-auth process for gog: [notes/gog.md](notes/gog.md)

---

## Active Channels

| Channel | Status | Notes |
|---|---|---|
| Discord | Active | Guild: AVERATEC-WORKPLACE, `requireMention: false` |

---

## Installed Skills

| Skill | Version | Path | Loaded by |
|---|---|---|---|
| `self-improving-agent` | 3.0.0 | `/home/node/.openclaw/skills/` | `extraDirs` config |

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
