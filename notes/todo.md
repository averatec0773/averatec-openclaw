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

**Current state:** ✅ Deployed (2026-03-12).

- `main` agent — Scott's Discord DMs, `gpt-5.4`, full tools, private workspace
- `public` agent — Guild messages, `gpt-5-mini`, `tools.deny: [exec, bash, computer]`, public workspace

See [CONTEXT.md](../CONTEXT.md) Multi-Agent Setup section for routing details.

---

## Model

**Current:** `openai/gpt-5.4` — switched from Anthropic Sonnet 4.6 due to Tier 1 rate limits.

- [ ] Revisit Claude Sonnet 4.6 once API tier upgrades
