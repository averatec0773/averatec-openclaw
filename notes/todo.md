# TODO — Known Issues & Future Improvements

Tracks things that work but could be better. Not urgent unless noted.

---

## Workspace Backup

**Current state:** No automated backup. Skill-based approach removed (LLM non-determinism + token injection issues).

**Why it was removed:**
- LLM re-interprets bash on every call — not deterministic, can skip steps or improvise
- `GITHUB_TOKEN` not available as shell env var in container; embedding token in remote URL is fragile across rebuilds

**Planned:**
- [ ] VPS cron job running a shell script directly — fully deterministic, no LLM involvement
- [ ] Script: tar workspace → git push to `averatec0773/averatec-openclaw-backup`
- [ ] Also back up `/root/.openclaw/skills/` (custom skills dir)

---

## Multi-Agent Setup

**Current state:** Single agent handles all Discord messages (guild + DMs).

**Planned:**
- [ ] Add `public` agent for guild messages — restricted model, no personal tools
- [ ] Bind Scott's DM (Discord ID `360785034438901761`) to `private` agent
- [ ] Create `workspace-public/` with minimal SOUL.md (no USER.md, no TOOLS.md)
- [ ] Use `agents.public.tools.deny` to block bash/exec on public agent

See [notes/workspace-files.md](workspace-files.md) for workspace file structure reference.

---

## Model

**Current:** `openai/gpt-5.4` — switched from Anthropic Sonnet 4.6 due to Tier 1 rate limits.

- [ ] Revisit Claude Sonnet 4.6 once API tier upgrades
