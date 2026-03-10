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

## OpenClaw 自定义镜像（averatec-custom）

### 镜像分层结构

- `openclaw:latest` — 上游原始镜像（从 Docker Hub pull）
- `openclaw:averatec-custom` — 基于 latest 的自定义镜像（由 `Dockerfile.custom` 构建）

自定义镜像定义在 `Dockerfile.custom`（备份于 averatec-openclaw repo），额外安装了：
- `clawhub`（npm global）
- `gh`（GitHub CLI）

镜像名通过服务器 `~/openclaw/.env` 中的 `OPENCLAW_IMAGE=openclaw:averatec-custom` 控制，`.env` 被 gitignore 所以上游更新不会覆盖。

### 正常更新流程

```bash
# 1. 拉取最新上游镜像
docker pull openclaw:latest

# 2. 重新构建自定义层（自动基于新的 latest）
docker compose build

# 3. 重启容器
docker compose up -d
```

### 从上游源码 build 原始镜像（通常不需要）

```bash
# 仅在需要修改上游代码时使用
docker build -f Dockerfile -t openclaw:latest .
docker compose build
docker compose up -d
```

### 上游更新后各文件处理方式

| 文件 | 被 git pull 覆盖 | 处理策略 |
|------|------|------|
| `.env` | 不会（gitignored）| 安全，镜像名配置在此 |
| `docker-compose.yml` | 会 | 不动它，用 env 变量控制 |
| `Dockerfile.custom` | 会 | 手动合并，把自定义层重新加回去 |
