# ClawHub — Prerequisite for Installing Skills

> ClawHub is the official public skill registry for OpenClaw.
> You need the `clawhub` CLI before you can install, update, or publish any skills.
>
> Registry: https://clawhub.ai — Reference: https://docs.openclaw.ai/tools/clawhub

---

## Install the CLI

```bash
npm i -g clawhub
# or
pnpm add -g clawhub
```

By default, skills are installed into `./skills` in your working directory,
or fall back to the OpenClaw workspace config path.

---

## Authentication

```bash
clawhub login            # opens browser flow
clawhub login --token <token>   # token-based login
clawhub logout
clawhub whoami           # check who you are logged in as
```

---

## Search & Install

```bash
# Search the registry
clawhub search "postgres backups"
clawhub search "git" --limit 10

# Install a skill
clawhub install <slug>
clawhub install <slug> --version 1.2.3
clawhub install <slug> --force     # overwrite existing
```

---

## Update

```bash
# Update a specific skill
clawhub update <slug>
clawhub update <slug> --version 1.3.0

# Update all installed skills
clawhub update --all
clawhub update --all --no-input    # non-interactive (CI-friendly)
```

---

## List Installed Skills

```bash
clawhub list    # reads from .clawhub/lock.json
```

---

## Publish Your Own Skill

```bash
clawhub publish ./my-skill \
  --slug my-skill \
  --name "My Skill" \
  --version 1.0.0 \
  --changelog "Initial release" \
  --tags "git,automation"

# Batch scan and upload a skills folder
clawhub sync
```

---

## Delete / Restore

```bash
clawhub delete <slug> --yes      # owner or admin only
clawhub undelete <slug> --yes
```

---

## Global Options

| Flag | Description |
|------|-------------|
| `--workdir <dir>` | Override working directory |
| `--dir <dir>` | Skills subdirectory (default: `skills`) |
| `--site <url>` | Override site base URL |
| `--registry <url>` | Override registry API endpoint |
| `--no-input` | Disable interactive prompts |

---

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `CLAWHUB_SITE` | Custom site URL |
| `CLAWHUB_REGISTRY` | Custom registry API endpoint |
| `CLAWHUB_CONFIG_PATH` | Custom config file path |
| `CLAWHUB_WORKDIR` | Default working directory |
| `CLAWHUB_DISABLE_TELEMETRY=1` | Opt out of telemetry |

---

## Typical Workflow

```bash
# 1. Install the CLI (one time)
npm i -g clawhub

# 2. Log in (one time)
clawhub login

# 3. Search for a skill
clawhub search "github"

# 4. Install it
clawhub install steipete/clawdhub

# 5. Keep skills up to date
clawhub update --all
```
