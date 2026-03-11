# gog — Google Workspace CLI (gogcli)

> `gog` is the CLI for accessing Gmail, Calendar, Drive, and other Google services.
> Repo: https://github.com/steipete/gogcli

Installed in `openclaw:averatec-custom` via `Dockerfile.custom`.

---

## Auth Setup (headless Linux)

Headless servers have no system keyring. gog must use the **file keyring backend** with a password.

Required entries in `~/openclaw/.env`:
```
GOG_KEYRING_PASSWORD=<your-keyring-password>   # encrypts the token file; value is arbitrary
GOG_ACCOUNT=<your-gmail-address>              # default account; removes need for --account flag
```

Both vars are picked up automatically via `env_file: - .env` in `docker-compose.yml` — no override needed.

This is already configured. Token file lives at `/home/node/.openclaw/gogcli/keyring/` (config volume, persistent).

Authorized account: set via `GOG_ACCOUNT` in `.env`

---

## Re-auth (if token expires or needs reset)

OAuth callback requires a browser. Since the server is headless, use an SSH tunnel:

```bash
# 1. On the server: set file keyring and start auth
ssh openclaw
/tmp/gog auth keyring file   # only needed once
GOG_KEYRING_PASSWORD=<your-keyring-password> /tmp/gog auth add <your-gmail-address>
# Note the port in the output, e.g. 127.0.0.1:40055

# 2. On local machine: tunnel that port
ssh -f -N -L <PORT>:127.0.0.1:<PORT> openclaw

# 3. Open the Google auth URL in local browser
# After authorizing, copy the callback URL and run on server:
curl 'http://127.0.0.1:<PORT>/oauth2/callback?...'

# 4. Copy token files into container
docker cp /root/.config/gogcli/. openclaw:/home/node/.openclaw/gogcli/
docker compose exec --user root openclaw-gateway chown -R node:node /home/node/.openclaw/gogcli/

# 5. Verify
docker compose exec openclaw-gateway bash -c 'GOG_KEYRING_PASSWORD=<your-keyring-password> gog auth list'
```

> Note: `gog` binary on the server is at `/tmp/gog` (temporary). The permanent one is inside the container at `/usr/local/bin/gog`.

---

## Usage (inside container)

```bash
docker compose exec openclaw-gateway bash -c 'gog gmail search "is:unread"'
docker compose exec openclaw-gateway bash -c 'gog calendar list'
```

Or just ask the bot in Discord — it uses `gog` via the built-in skill automatically.
