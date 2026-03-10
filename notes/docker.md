# Docker Common Commands

## Images

```bash
# Pull an image
docker pull image:tag

# List local images
docker images

# Build image from current directory Dockerfile
docker build -t my-image:latest .

# Build with a specific Dockerfile
docker build -f Dockerfile.prod -t my-image:prod .

# Remove an image
docker rmi image:tag

# Remove all unused images
docker image prune -a
```

## Containers

```bash
# Run a container
docker run -d --name my-container image:tag

# Common run options
docker run -d \
  --name my-container \
  -p 8080:80 \                      # port mapping: host:container
  -v /host/path:/container/path \   # mount directory
  -e ENV_VAR=value \                # environment variable
  --restart unless-stopped \        # auto-restart policy
  image:tag

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Start / stop / restart
docker start my-container
docker stop my-container
docker restart my-container

# Remove a container
docker rm my-container

# Force remove a running container
docker rm -f my-container

# Remove all stopped containers
docker container prune
```

## Debugging

```bash
# View logs
docker logs my-container

# Follow logs in real time
docker logs -f my-container

# Show last 100 lines
docker logs --tail 100 my-container

# Open a shell inside container
docker exec -it my-container bash
docker exec -it my-container sh   # if bash is not available

# Run a command inside container
docker exec my-container which gog

# Show container details
docker inspect my-container

# Show resource usage
docker stats
docker stats my-container
```

## Docker Compose

```bash
# Start all services in background
docker compose up -d

# Build then start
docker compose up -d --build

# Build only, do not start
docker compose build

# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v

# Show service status
docker compose ps

# Follow service logs
docker compose logs -f
docker compose logs -f openclaw-gateway

# Open shell in a service container
docker compose exec openclaw-gateway bash

# Run a command in a service container
docker compose exec openclaw-gateway which gog

# Restart a specific service
docker compose restart openclaw-gateway

# Pull latest image and restart
docker compose pull && docker compose up -d
```

## Volumes

```bash
# List all volumes
docker volume ls

# Show volume details
docker volume inspect volume_name

# Remove unused volumes
docker volume prune

# Create a named volume
docker volume create my-volume
```

## Networks

```bash
# List networks
docker network ls

# Show network details
docker network inspect bridge

# Create a custom network
docker network create my-network
```

## System Cleanup

```bash
# Remove all unused resources (images, containers, networks, cache)
docker system prune -a

# Include volumes
docker system prune -a --volumes

# Show disk usage
docker system df
```

## OpenClaw Shortcuts

```bash
# Build and start
docker compose build && docker compose up -d openclaw-gateway

# Follow gateway logs
docker compose logs -f openclaw-gateway

# Verify binaries are baked into the image
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli

# Rebuild after code changes
docker compose down && docker compose build && docker compose up -d
```

## OpenClaw Skill Management (clawhub)

Skills must be installed into `/home/node/.openclaw/skills/` to be persistent and auto-loaded.
This path is on the config volume (`/root/.openclaw/` on host) and is registered in `openclaw.json` via `skills.load.extraDirs`.

```bash
# Install a skill (persistent, auto-loaded by openclaw)
docker compose exec --user root openclaw-gateway \
  clawhub install <slug> --workdir /home/node/.openclaw --dir skills

# List installed skills
docker compose exec -w /home/node/.openclaw openclaw-gateway clawhub list

# Update all skills
docker compose exec --user root openclaw-gateway \
  clawhub update --all --workdir /home/node/.openclaw --dir skills

# Login with token (run once; credentials saved in config volume)
docker compose exec --user root openclaw-gateway \
  clawhub login --token <YOUR_CLAWHUB_TOKEN>
```

### Volume Mapping Reference

| Container path | Host path | Persistent |
|---|---|---|
| `/home/node/.openclaw/` | `/root/.openclaw/` | Yes — config volume |
| `/home/node/.openclaw/skills/` | `/root/.openclaw/skills/` | Yes — user skills here |
| `/home/node/.openclaw/workspace/` | `/root/.openclaw/workspace/` | Yes — agent context |
| `/app/skills/` | *(none)* | No — baked into image |

### Currently Installed Skills

| Skill | Version | Path |
|---|---|---|
| `self-improving-agent` | 3.0.0 | `/home/node/.openclaw/skills/` |

---

## OpenClaw Custom Image (averatec-custom)

### Image Layers

- `openclaw:latest` — upstream image pulled from Docker Hub
- `openclaw:averatec-custom` — custom image built on top of latest via `Dockerfile.custom`

`Dockerfile.custom` is backed up in the averatec-openclaw repo. It adds:
- `clawhub` (npm global)
- `gh` (GitHub CLI)

The image name is controlled by `OPENCLAW_IMAGE=openclaw:averatec-custom` in `~/openclaw/.env` on the server. Since `.env` is gitignored, upstream updates will never overwrite it.

### Standard Update Workflow

```bash
# 1. Pull the latest upstream image
docker pull openclaw:latest

# 2. Rebuild the custom layer on top of it
docker compose build

# 3. Restart the container
docker compose up -d
```

### Build the Upstream Image from Source (rarely needed)

```bash
# Only needed if modifying upstream source code
docker build -f Dockerfile -t openclaw:latest .
docker compose build
docker compose up -d
```

### Handling Upstream Updates

| File | Overwritten by git pull | Strategy |
|------|------|------|
| `.env` | No (gitignored) | Safe — image name lives here |
| `docker-compose.yml` | Yes | Don't modify — control via env vars |
| `Dockerfile.custom` | Yes | Manually re-apply custom layers after merge |
