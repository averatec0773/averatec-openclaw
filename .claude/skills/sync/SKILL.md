---
name: sync
description: Use when updating the averatec-openclaw repo after server changes, or syncing server workspace files back to repo. Triggered by "update repo", "sync docs", "同步", "更新 repo".
user-invocable: true
allowed-tools: Read, Edit, Write, Bash, Glob, Grep
---

# averatec-openclaw Repo Sync

Consult [file-map.md](file-map.md) for the complete server↔repo mapping table and redaction rules.

## When to use

- After editing server files via `ssh openclaw` (AGENTS.md, SOUL.md, openclaw.json, skills)
- After deploying a new skill to the server
- After local doc-only edits that need committing
- When the user says "更新 repo" / "sync" / "update repo"

---

## Step 1 — Identify what changed

Recall this session's server edits. Categorize:

| Changed | Action |
|---|---|
| `workspace/AGENTS.md` | Update `templates/AGENTS.md` in repo |
| `workspace/SOUL.md` | Update `templates/SOUL.md` in repo |
| `openclaw.json` (structure) | Update `config/openclaw.json` (redacted) |
| New skill deployed to `skills/` | Update `notes/skills.md` + `CONTEXT.md` skill tables |
| Discord config changed | Update `notes/discord.md` + `CONTEXT.md` Active Channels |
| Doc-only local edit | Skip server step, go straight to commit |

If unsure which files changed, ask before proceeding — do not update everything blindly.

---

## Step 2 — Update repo files

Update only the files identified in Step 1.

**For `config/openclaw.json`:** apply redaction rules from [file-map.md](file-map.md) before writing.

**For skill tables** (`notes/skills.md`, `CONTEXT.md`): update both files — they have duplicate skill tables that must stay in sync.

---

## Step 3 — Commit and push

Stage only the files you actually changed:

```bash
git add <specific files only>
git commit -m "docs: <what changed and why>"
git push origin main
```

Commit message convention: `docs: <verb> <subject>` — e.g., `docs: update AGENTS.md act-first philosophy`.

---

## What NEVER goes into the repo

See [file-map.md](file-map.md) — Never sync section.
