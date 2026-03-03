# Claude Code Settings & Configuration

## Settings File

**Path:** `~/.claude/settings.json`

Contains model preferences, tool permissions, MCP servers, and hooks.

---

## CLAUDE.md (Global Preferences)

**Path:** `/home/claude-user/CLAUDE.md`

Project-wide instructions loaded into every Claude Code session. Contents:

- User: Bostjan, Platform: Linux on Proxmox/PVE
- Communication: concise, no fluff, no emojis unless asked
- Environment: Python at `/usr/bin/python3`, Node at `/usr/bin/node`, headless only
- Key projects paths
- OpenStripe server management rules (CRITICAL section)
- Secrets policy
- SSH recovery instructions

**Note:** CLAUDE.md instructions override default Claude behavior.

---

## Hooks

Hooks run shell scripts on tool events during Claude Code sessions.

| Event | Script | Mode |
|-------|--------|------|
| `PreToolUse` | `pre-tool-relay.sh` | async |
| `PostToolUse` | `post-tool-relay.sh` | async |
| `SubagentStart` | `pre-tool-relay.sh` | async |
| `Stop` | `post-tool-relay.sh` | async |

**Hook scripts location:**
```
/home/claude-user/claude-projects/claude-dashboard/server/hooks/pre-tool-relay.sh
/home/claude-user/claude-projects/claude-dashboard/server/hooks/post-tool-relay.sh
```

These relay tool events to external monitoring. (claude-dashboard server is stopped but hook scripts remain.)

**Fix applied 2026-03-02:** Stop hook was incorrectly pointing to pre-tool-relay.sh — corrected to post-tool-relay.sh.

---

## MCP Servers

MCP (Model Context Protocol) servers extend Claude Code's capabilities.

| Server | Purpose |
|--------|---------|
| `filesystem` | File read/write/search |
| `memory` | Persistent knowledge graph |
| `playwright` | Browser automation |
| `sequential-thinking` | Chain-of-thought reasoning |
| `context7` | Library documentation lookup |
| `n8n-mcp` | N8N workflow management |

### N8N MCP Wrapper

Because N8N requires an API key that shouldn't be in settings.json, a wrapper script is used:

**Path:** `~/.claude/n8n-mcp-wrapper.sh`
```bash
#!/bin/bash
source ~/.claude/secrets.env
exec npx n8n-mcp
```

The settings.json references this wrapper instead of `npx n8n-mcp` directly.

---

## Memory System (Claude Code)

**Path:** `~/.claude/projects/-home-claude-user/memory/`

| File | Purpose |
|------|---------|
| `MEMORY.md` | Auto-loaded into every session (truncated after 200 lines) |
| `browser-automation.md` | MCP Playwright and Python Playwright patterns |

### Auto-Memory Protocol

Claude writes memory using comment blocks:

**Long-term memory:**
```
<!-- MEMORY:LONG_TERM
content
/MEMORY:LONG_TERM -->
```

**Daily log:**
```
<!-- MEMORY:DAILY
content
/MEMORY:DAILY -->
```

These are placed at the END of responses and parsed by the memory system.

---

## Session Management

**Session files:** `~/.claude/projects/-home-claude-user/*.jsonl`

Each session is a JSONL file containing the full conversation history.

**Commands:**
```bash
claude          # new session
claude -c       # continue last session
claude -r <id>  # resume specific session by UUID
```

**Total sessions as of 2026-03-03:** 77+ session files

---

## Skill Router

When you invoke a skill, the `skill-router` skill maps requests to the correct skill. It covers 107 skills across 15 categories.

Skills are invoked in two ways:
1. Explicit: `Skill tool` with skill name
2. Slash command: `/skill-name` expands to skill prompt
