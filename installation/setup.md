# Installation & Setup

> Reference: https://docs.openclaw.ai/install/hetzner

## What is OpenClaw?

OpenClaw is a self-hosted gateway that connects messaging apps (WhatsApp, Telegram, Discord, iMessage) to AI coding agents. Runs as a single process, bridging chat platforms with an always-available assistant while keeping data under your control.

---

## Hetzner VPS Setup (~$5/month)

### Prerequisites

- Hetzner VPS (Ubuntu / Debian) with root SSH access
- SSH client on your laptop
- Model authentication credentials
- Optional: WhatsApp QR, Telegram bot token, Gmail OAuth
- ~20 minutes setup time

---

### Step 1 — Provision VPS

```bash
ssh root@YOUR_VPS_IP
```

---

### Step 2 — Install Docker

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

Verify:

```bash
docker --version
docker compose version
```

---

### Step 3 — Clone Repository

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

---

### Step 4 — Create Persistent Directories

```bash
mkdir -p /root/.openclaw/workspace
chown -R 1000:1000 /root/.openclaw
```

---

### Step 5 — Configure Environment Variables

Create `.env` in the repository root:

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Generate secure values:

```bash
openssl rand -hex 32
```

> **Warning:** Never commit `.env` to version control.

---

### Step 6 — Docker Compose Configuration

Create `docker-compose.yml`:

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build:
      context: .
      dockerfile: Dockerfile.custom   # use custom Dockerfile (default is "Dockerfile")
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
        "--allow-unconfigured",
      ]
```

> Port is bound to loopback (`127.0.0.1`) by default. Remove the prefix to expose publicly — manage firewall and tokens accordingly.

---

### Step 7 — Bake Required Binaries (Critical)

Create `Dockerfile`:

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

> **Critical:** All external binaries required by skills must be baked at image build time. Installing at runtime will be lost on container restart.

#### Using Dockerfile.custom (recommended)

Instead of modifying the base `Dockerfile`, keep your additions separate in `Dockerfile.custom`:

```dockerfile
FROM openclaw:latest

USER root

RUN npm install -g clawhub
RUN apt-get update && apt-get install -y gh && rm -rf /var/lib/apt/lists/*

USER node
```

> The base image runs as the `node` user by default, which has no write access to `/usr/local/lib/node_modules/`.
> Switch to `root` before installing, then switch back to `node` afterward.

Then point `docker-compose.yml` to it:

```yaml
build:
  context: .
  dockerfile: Dockerfile.custom
```

This way, your customizations (extra tools, skills CLI) stay cleanly separated from the base image.

---

### Step 8 — Build and Launch

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Verify binaries:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
# Expected: /usr/local/bin/{gog,goplaces,wacli}
```

---

### Step 9 — Access the Gateway

Check logs:

```bash
docker compose logs -f openclaw-gateway
# Look for: "listening on ws://0.0.0.0:18789"
```

Create SSH tunnel from your laptop:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Open: `http://127.0.0.1:18789/` (use your `OPENCLAW_GATEWAY_TOKEN` to authenticate)

---

## Persistence Architecture

| Component | Host Path | Notes |
|-----------|-----------|-------|
| Gateway config | `/root/.openclaw/` | Tokens, configs |
| Auth profiles | `/root/.openclaw/` | OAuth / API keys |
| Skill configs | `/root/.openclaw/skills/` | Skill state |
| Workspace | `/root/.openclaw/workspace/` | Agent artifacts |
| WhatsApp session | `/root/.openclaw/` | QR login persistence |
| Gmail keyring | `/root/.openclaw/` | Requires `GOG_KEYRING_PASSWORD` |
| External binaries | `/usr/local/bin/` | Baked into Docker image at build time |

---

## Infrastructure-as-Code (Optional)

Community Terraform configs for reproducible Hetzner deployments:

- [openclaw-terraform-hetzner](https://github.com/andreesg/openclaw-terraform-hetzner)
- [openclaw-docker-config](https://github.com/andreesg/openclaw-docker-config)
