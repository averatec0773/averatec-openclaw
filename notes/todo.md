# TODO — Known Issues & Future Improvements

Tracks things that work but could be better. Not urgent unless noted.

---

## Backup

Two separate strategies — sensitive and non-sensitive files handled differently.

### Auto Backup: Workspace → GitHub (non-sensitive)

**Scope:**
- `/root/.openclaw/workspace/` — main agent workspace (SOUL, AGENTS, USER, TOOLS, MEMORY, memory/, skills/)
- `/root/.openclaw/workspace-public/` — public agent workspace
- `/root/.openclaw/skills/averatec-discord/` — custom skill
- `/root/.openclaw/skills/averatec-email/` — custom skill

**Plan:** VPS cron job (daily) running a shell script on the host — no LLM involvement, fully deterministic.
- Script syncs above dirs into a local git repo on VPS, commits, pushes to private GitHub repo
- Auth via SSH Deploy Key on the backup repo (no PAT in scripts)
- Target repo: `averatec0773/averatec-openclaw-backup` (create if not exists)

**Why not skill-based:** LLM re-interprets bash on every call — not deterministic. `GITHUB_TOKEN` also not available as shell env var in container.

- [ ] Create `averatec-openclaw-backup` private repo on GitHub
- [ ] Generate SSH key on VPS, add as Deploy Key with write access
- [ ] Write `/root/scripts/backup-workspace.sh`
- [ ] Add cron job: `0 3 * * * /root/scripts/backup-workspace.sh`

### Manual Backup: Sensitive Files → Local Machine

**Scope** (pulled via rsync/scp from local machine on demand):
- `/root/openclaw/.env` — all service secrets
- `/root/.openclaw/openclaw.json` — API keys (Tavily etc.)
- `/root/.openclaw/gh/hosts.yml` — GitHub CLI token
- `/root/.openclaw/gogcli/` — Google OAuth credentials + encrypted keyring
- `/root/.openclaw/clawhub/config.json` — ClawHub token
- `/root/.openclaw/identity/` — device keypair + operator token (critical — needed for re-pairing)
- `/root/.openclaw/credentials/discord-default-allowFrom.json` — main bot allowlist
- `/root/.openclaw/credentials/discord-public-allowFrom.json` — public bot allowlist
- `/root/client_secret.json` — Google OAuth app client secret

**Why not GitHub:** Even private repos carry leak risk. Tokens revokable but Google keyring harder to re-auth.

**Notes:**
- `/root/.config/gogcli/` deleted — was an unused duplicate of `.openclaw/gogcli/`
- `discord-pairing.json` deleted — ephemeral, always empty
- `/root/openclaw/` source code not backed up — upstream, rebuildable from Docker Hub

- [ ] Write `backup-sensitive.sh` on local machine (rsync pull over SSH)

---

## Multi-Agent Setup

**Current state:** ✅ Deployed (2026-03-12).

- `main` agent — Scott's Discord DMs, `gpt-5.4`, full tools, private workspace
- `public` agent — Guild messages, `gpt-5-mini`, `tools.deny: [exec, bash, computer]`, public workspace

See [CONTEXT.md](../CONTEXT.md) Multi-Agent Setup section for routing details.

---

## Install Claude CLI on Server

- [ ] Install Node.js + Claude CLI (`npm install -g @anthropic-ai/claude-code`) on the Hetzner VPS

---

## Model

**Current:** `openai/gpt-5.4` — switched from Anthropic Sonnet 4.6 due to Tier 1 rate limits.

- [ ] Revisit Claude Sonnet 4.6 once API tier upgrades
