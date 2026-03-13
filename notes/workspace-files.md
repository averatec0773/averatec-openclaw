# Workspace Files — Usage Rules

> Ref: https://docs.openclaw.ai/reference/templates/TOOLS
> Workspace dir on server: `/root/.openclaw/workspace/`

Each file in the workspace has a specific purpose. Mixing content between files degrades agent behavior — the LLM reads each file expecting a certain type of content.

---

## File-by-File Rules

### AGENTS.md — Behavioral Instructions
**What belongs here:**
- Session startup sequence (what to read, in what order)
- Behavioral rules ("always do X", "never do Y")
- Skill usage routing rules (which skill to use for what)
- Memory management conventions
- Platform-specific formatting rules (Discord, WhatsApp)
- Red lines / hard limits

**What does NOT belong here:**
- Infrastructure specifics (accounts, IDs) → TOOLS.md
- Personality / tone → SOUL.md
- User profile → USER.md

---

### TOOLS.md — Infrastructure Specifics
**What belongs here:**
- Specific accounts/emails in use (e.g. which Gmail account)
- Device identifiers, SSH hosts, camera names
- API defaults that are environment-specific
- IDs and handles unique to this deployment (Discord guild ID, owner user ID)
- Timezone for the owner

**What does NOT belong here:**
- Behavioral rules ("always use skill X") → AGENTS.md
- How a CLI tool works in general → the skill's SKILL.md
- Personality → SOUL.md

> Official definition: "Skills define *how* tools work. TOOLS.md holds *your* specifics —
> the environment-specific details unique to your setup."

---

### SOUL.md — Personality & Values
**What belongs here:**
- Agent personality traits and tone
- Core values and principles
- Communication style preferences
- How the agent should handle uncertainty or disagreement

**What does NOT belong here:**
- Specific tool configs → TOOLS.md
- Behavioral rules → AGENTS.md
- User info → USER.md

---

### USER.md — Human Profile
**What belongs here:**
- Name, preferred address
- Timezone
- Language/communication preferences
- GitHub, Discord handles
- Brief context about who this person is and their workflow preferences

**What does NOT belong here:**
- Behavioral rules for the agent → AGENTS.md
- Infrastructure config → TOOLS.md

---

### MEMORY.md — Long-Term Agent Memory
**What belongs here:**
- Distilled learnings from past sessions
- Decisions made, context behind them
- Patterns the agent should remember across restarts
- Updated periodically during heartbeats

**Load rules (important):**
- Load ONLY in main session (direct chat with owner)
- Do NOT load in shared contexts (group chats, public Discord channels)
- Reason: contains personal context that must not leak to strangers

---

### HEARTBEAT.md — Periodic Task List
**What belongs here:**
- Short checklist of things to check on each heartbeat poll
- Reminders or time-sensitive tasks

**Usage:**
- Keep this file small — every line costs tokens on every heartbeat
- Leave empty (or only comments) to skip heartbeat API calls entirely
- Use cron jobs for exact-timing tasks; use HEARTBEAT.md for batched periodic checks

---

### IDENTITY.md — Agent Self-Definition
**What belongs here:**
- Agent's chosen name, creature type, vibe, emoji, avatar path
- Filled in during first conversation, then persists

**Note:** This file is auto-generated as a template by OpenClaw. The agent fills it in.

---

## Current State (as of 2026-03-12)

| File | Status | Notes |
|------|--------|-------|
| `AGENTS.md` | Good | Session startup + behavioral rules + skill usage rules |
| `TOOLS.md` | Good | Gmail account, Discord guild/user ID, timezone |
| `SOUL.md` | Good | Default OpenClaw template, lightly customized |
| `USER.md` | Good | Scott's profile filled in |
| `MEMORY.md` | Active | Agent writes to this |
| `HEARTBEAT.md` | Empty | No periodic tasks configured |
| `IDENTITY.md` | Empty | Agent hasn't filled this in yet |

---

## Common Mistakes to Avoid

| Mistake | Problem | Fix |
|---------|---------|-----|
| Skill routing rules in TOOLS.md | LLM treats TOOLS.md as config, not instructions | Move to AGENTS.md |
| Backup instructions in TOOLS.md | Same issue | Move to AGENTS.md |
| Personal context in MEMORY.md loaded in group chats | Privacy leak | AGENTS.md must gate MEMORY.md to main sessions only |
| Long HEARTBEAT.md | Token burn on every poll | Keep minimal |
