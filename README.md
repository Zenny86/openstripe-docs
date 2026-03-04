# OpenStripe — System Documentation

> Private knowledge base for the OpenStripe autonomous Claude agent platform.

OpenStripe is a self-hosted platform that lets you run Claude Code tasks via Telegram, a web UI, or n8n webhooks. It manages a task queue, streams real-time output, tracks costs, and maintains persistent memory across sessions.

---

## Quick Reference

| Item | Value |
|------|-------|
| Server port | `3848` |
| Process manager | PM2 (`pm2 restart openstripe`) |
| Database | SQLite at `data/dashboard.db` |
| Memory | `data/memory/MEMORY.md` + daily logs |
| Project path | `/home/claude-user/claude-projects/openstripe/` |
| Web UI | `http://localhost:3848` |
| Claude binary | `/home/claude-user/.local/bin/claude` |

---

## Documents

| File | Contents |
|------|----------|
| [architecture.md](architecture.md) | System architecture, component map, data flow |
| [setup.md](setup.md) | Installation, configuration, first run |
| [pm2-management.md](pm2-management.md) | PM2 process management — CRITICAL |
| [telegram-bot.md](telegram-bot.md) | Telegram commands, multi-task, voice transcription |
| [api-reference.md](api-reference.md) | All REST API endpoints |
| [memory-system.md](memory-system.md) | How persistent memory works |
| [claude-integration.md](claude-integration.md) | Claude CLI runner, streaming, sessions |
| [n8n-integration.md](n8n-integration.md) | N8N webhook setup |
| [skills.md](skills.md) | Installed skills catalog (109 skills) |
| [environment.md](environment.md) | Server environment, secrets, paths |
| [troubleshooting.md](troubleshooting.md) | Common issues and fixes |
| [deployment-guide.md](deployment-guide.md) | Step-by-step self-hosting guide from scratch |
| [discord-bot.md](discord-bot.md) | Discord bot — slash commands, auto-mod, n8n webhooks |
| [social-media-integrations.md](social-media-integrations.md) | Discord feedback agent + social media analyzer |
| [why-openstripe.md](why-openstripe.md) | Origin story — why this was built |

---

## CRITICAL: Server Management

**Never run `node server/index.js` directly.** PM2 owns port 3848. Direct node launch causes EADDRINUSE crash loops and kills Telegram access.

```bash
# Restart after code changes
pm2 restart openstripe

# View logs
pm2 logs openstripe

# Status
pm2 list
```

---

## Tech Stack

- **Backend:** Node.js, Express, WebSocket, better-sqlite3
- **Frontend:** React 19, Vite, Tailwind CSS 4, Recharts
- **AI:** Claude Code CLI (`claude`) spawned as subprocess
- **Bot:** node-telegram-bot-api, discord.js v14
- **Process manager:** PM2
- **Auth:** JWT (24h expiry) + bcrypt
- **Voice transcription:** Groq Whisper (`whisper-large-v3-turbo`)

---

## Integrations Built

| Integration | Description |
|-------------|-------------|
| Telegram bot | Primary task interface — commands, multi-task, voice |
| Discord bot | Community management + n8n webhook target (port 3849) |
| N8N | Workflow automation, webhook triggers, SSH runners |
| Groq Whisper | Voice-to-text for Telegram voice messages |
| Gmail (himalaya) | Email via IMAP/SMTP from CLI |
| Discord feedback agent | Feedback channel → Claude agent → auto-response |
| Social media analyzer | TikTok + Instagram scraper → Claude scoring → Discord embeds |
