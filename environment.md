# Environment

## System

| Item | Value |
|------|-------|
| OS | Linux on Proxmox/PVE |
| Shell | bash |
| User | claude-user |
| Home | `/home/claude-user` |
| Python | `/usr/bin/python3` (3.12) |
| Node.js | `/usr/bin/node` |
| Display | None (headless — `DISPLAY` unset) |

---

## Key Paths

| What | Path |
|------|------|
| OpenStripe project | `/home/claude-user/claude-projects/openstripe/` |
| Claude-dashboard project | `/home/claude-user/claude-projects/claude-dashboard/` |
| Claude Code binary | `/home/claude-user/.local/bin/claude` |
| Claude Code settings | `~/.claude/settings.json` |
| Claude sessions | `~/.claude/projects/-home-claude-user/*.jsonl` |
| Claude memory | `~/.claude/projects/-home-claude-user/memory/` |
| Skills (ClawHub) | `~/.agents/skills/` |
| Skills (n8n) | `~/.claude/skills/` |
| Secrets file | `~/.claude/secrets.env` |
| N8N wrapper | `~/.claude/n8n-mcp-wrapper.sh` |
| CLAUDE.md | `/home/claude-user/CLAUDE.md` |

---

## Secrets (`~/.claude/secrets.env`)

File mode: `600` (owner read-only only)

```bash
N8N_API_KEY=...
GEMINI_API_KEY=...
DEEPSEEK_API_KEY=...
GROQ_API_KEY=...
GITHUB_TOKEN=...
BRAVE_API_KEY=...
GMAIL_APP_PASSWORD=...
```

Loaded into shell via `~/.bashrc`:
```bash
set -a
source ~/.claude/secrets.env
set +a
```

Also loaded at OpenStripe startup via `server/index.js`.

---

## Installed Tools

| Tool | Location | Purpose |
|------|----------|---------|
| `claude` | `~/.local/bin/claude` | Claude Code CLI |
| `gh` | system | GitHub CLI (authenticated as Zenny86) |
| `pm2` | npm global | Process manager |
| `gemini` | `/usr/bin/gemini` v0.31.0 | Gemini CLI |
| `himalaya` | `~/.local/bin/himalaya` v1.2.0 | Email CLI |
| `node` | `/usr/bin/node` | Node.js runtime |
| `python3` | `/usr/bin/python3` | Python 3.12 |
| `playwright` | `~/.local/bin/playwright` | Browser automation |

---

## External Services

| Service | URL / Endpoint | Auth |
|---------|---------------|------|
| N8N | `http://localhost:5678` | N8N_API_KEY |
| Groq (Whisper) | `https://api.groq.com/openai/v1/audio/transcriptions` | GROQ_API_KEY |
| Brave Search | `https://api.search.brave.com/res/v1/web/search` | BRAVE_API_KEY |
| GitHub | `https://api.github.com` | GITHUB_TOKEN |
| Gmail | `imap.gmail.com:993` / `smtp.gmail.com:587` | GMAIL_APP_PASSWORD |
| DeepSeek | `https://api.deepseek.com/v1` | DEEPSEEK_API_KEY |
| Gemini | API | GEMINI_API_KEY |

---

## Gmail Configuration

- **Email:** `your-email@gmail.com`
- **Tool:** himalaya v1.2.0
- **Config:** `~/.config/himalaya/config.toml`
- **Auth:** App password (not regular password)
- **IMAP:** `imap.gmail.com:993` (TLS)
- **SMTP:** `smtp.gmail.com:587` (STARTTLS)

---

## Browser Automation

**Preferred:** MCP Playwright tools (`mcp__playwright__*`)
- No temp files
- Native accessibility snapshots
- Works in headless environment
- Pattern: `navigate → snapshot → act on refs → snapshot to verify`

**Fallback:** Python Playwright
- Playwright 1.58.0 at `~/.local/bin/playwright`
- Chromium installed via `playwright install chromium`
- Always use `headless=True` (no display server)

---

## MCP Servers

Configured in `~/.claude/settings.json`. Key servers:
- `n8n-mcp` — N8N workflow management (via wrapper script)
- `filesystem` — File system access
- `memory` — Knowledge graph
- `playwright` — Browser automation
- `sequential-thinking` — Chain-of-thought
- `context7` — Documentation lookup

N8N MCP uses wrapper: `~/.claude/n8n-mcp-wrapper.sh` (sources secrets.env before running npx n8n-mcp)

---

## Claude Code Settings (`~/.claude/settings.json`)

Key settings:
- Model: claude-sonnet-4-6
- Hooks: PreToolUse, PostToolUse, SubagentStart → relay scripts
- Stop hook → post-tool-relay.sh
- Permissions: various tool allowances

---

## SSH Recovery

If web/Telegram interface breaks:
```bash
ssh claude-user@<server-ip>
cd /home/claude-user && claude -c   # continue last session
claude -r <uuid>                    # resume specific session
```

Sessions stored at: `~/.claude/projects/-home-claude-user/*.jsonl`
