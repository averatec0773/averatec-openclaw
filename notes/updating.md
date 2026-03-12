# Updating OpenClaw

Reference: https://docs.openclaw.ai/install/updating

## Our setup (Docker + custom image)

We run `openclaw:averatec-custom` built on top of `openclaw:latest`. The official update paths (npm global, source install) do not apply — we must rebuild the custom image.

### Update flow

```bash
ssh openclaw "cd ~/openclaw && \
  docker compose pull && \
  docker compose build && \
  docker compose up -d"
```

1. `docker compose pull` — pulls new `openclaw:latest` base image
2. `docker compose build` — rebuilds `openclaw:averatec-custom` (reinstalls clawhub, gh, goplaces, gog)
3. `docker compose up -d` — restarts container with new image

After restart: re-approve device pairing in dashboard (`openclaw devices approve <requestId>`).

### Pre-update checklist

- Check release notes at https://github.com/openclaw/openclaw/releases
- Note any breaking changes (especially cron/delivery behavior)
- Workspace and config are on the host volume — safe across rebuilds

### Post-update verify

```bash
# Check container is running
ssh openclaw "docker ps"

# Check logs for errors
ssh openclaw "docker logs openclaw --tail 50"

# Re-establish SSH tunnel if needed
ssh -f -N -L 18789:127.0.0.1:18789 openclaw
```

### Rollback

Pin a specific image tag in `docker-compose.yml`:

```yaml
image: openclaw:2026.3.8  # pin to known-good version
```

Or rebuild from a specific base:

```dockerfile
FROM openclaw:2026.3.8  # in Dockerfile.custom
```

---

## Official update paths (not used, for reference)

**Global npm install:**
```bash
npm i -g openclaw@latest
```

**Channel switching:**
```bash
openclaw update --channel beta
openclaw update --channel stable
```

**Doctor (safe update + migrate):**
```bash
openclaw doctor
```

**Useful CLI:**
```bash
openclaw gateway status
openclaw gateway restart
openclaw logs --follow
openclaw health
```
