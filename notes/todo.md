# TODO — Known Issues & Future Improvements

Tracks things that work but could be better. Not urgent unless noted.

---

## Workspace Backup

**Current state:** `averatec-backup` skill — LLM reads SKILL.md, rewrites bash, executes via bash tool.

Two-step flow:
1. Local tar.gz snapshot → `/home/node/.openclaw/workspace-backups/` (last 10 kept)
2. Git push → private repo `averatec0773/averatec-openclaw-backup`

**Known limitations:**
- LLM re-interprets the bash script on every call — not deterministic. Can skip steps or improvise.
- skill description uses ALWAYS/NEVER wording to guide model, but not a hard guarantee.
- `GITHUB_TOKEN` is not available as a shell env var inside the container (OpenClaw injects it into LLM context only). Token is embedded directly in `.git/config` remote URL as workaround.
- `.git-credentials` at `/home/node/.git-credentials` is in the container layer — lost on image rebuild. Currently mitigated by embedding token in remote URL (stored in config volume).

**Possible future improvements:**
- [ ] Replace skill-based backup with a VPS cron job running a shell script directly — fully deterministic, no LLM involvement
- [ ] Consider `command-dispatch` frontmatter if OpenClaw adds support for exact-script execution
- [ ] Evaluate whether OpenClaw natively exposes env vars to bash tool in future versions (would simplify token handling)
- [ ] Add backup of `/root/.openclaw/skills/` (custom skills) to the same git repo or a separate one

---
