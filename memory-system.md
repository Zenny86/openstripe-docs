# Memory System

## Overview

OpenStripe maintains persistent memory across all Claude sessions. Memory is stored as plain Markdown files and automatically injected into every task prompt. Claude can write back to memory using special comment blocks in its output.

This is what gives Claude continuity — it "remembers" your preferences, past decisions, key facts, and daily activity.

---

## Memory Files

**Location:** `data/memory/` (relative to OpenStripe root)

| File | Purpose |
|------|---------|
| `MEMORY.md` | Long-term durable facts — preferences, architecture, key decisions |
| `YYYY-MM-DD.md` | Daily logs — timestamped entries for that day's activity |

---

## How Memory is Injected

Before every task, `memoryService.injectMemory(prompt)` prepends context to the prompt:

```
[System context — today's date, MEMORY.md contents, today's log]

[User's actual prompt]
```

Claude sees this context at the start of every task, giving it continuity across sessions.

---

## How Claude Writes Memory

Claude uses special HTML comment blocks in its output responses. The memory service scans completed task output for these patterns and persists them.

### Long-term memory block
```
<!-- MEMORY:LONG_TERM
Durable fact or preference to remember forever.
/MEMORY:LONG_TERM -->
```
→ Appended to `MEMORY.md`

### Daily log block
```
<!-- MEMORY:DAILY
2026-03-03: What was done today, context for tomorrow.
/MEMORY:DAILY -->
```
→ Appended to today's `YYYY-MM-DD.md` with timestamp

---

## Memory Search

Full-text search via FTS5 SQLite index across all memory files.

**Via API:**
```
GET /api/memory/search?q=telegram
```

**Via web UI:** Memory page → search box

**Rebuild index** (if out of sync):
```
POST /api/memory/rebuild-index
```

---

## Editing Memory

### Via web UI
Memory page → select file → edit → save (MEMORY.md only editable via UI)

### Via API
```bash
curl -X PUT http://localhost:3848/api/memory/file \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"name": "MEMORY.md", "content": "# Memory\n..."}'
```

### Directly on disk
```bash
nano /home/claude-user/claude-projects/openstripe/data/memory/MEMORY.md
```
Then rebuild index via API or web UI.

---

## Claude Code's Own Memory

Separate from OpenStripe's memory, Claude Code itself has a memory system at:

```
~/.claude/projects/-home-claude-user/memory/
├── MEMORY.md       # Long-term facts (loaded into every conversation)
└── browser-automation.md  # Topic-specific notes
```

This memory is for Claude Code's own context when running interactively (not via OpenStripe task queue). It's injected via the system prompt in Claude Code sessions.

**Key difference:**
- `openstripe/data/memory/` → Memory injected into tasks run *through* OpenStripe
- `~/.claude/projects/.../memory/MEMORY.md` → Memory for Claude Code interactive sessions

---

## Best Practices

**What to store long-term:**
- User preferences (communication style, tools to use)
- Architecture decisions and key file paths
- Credentials locations (not the secrets themselves)
- Solutions to recurring problems
- Key project facts

**What to store daily:**
- What was done today
- In-progress context
- Tasks completed and their outcomes

**What NOT to store:**
- Secrets or API keys
- Session-specific temporary state
- Large amounts of code (store paths, not contents)
- Speculative or unverified conclusions

---

## MEMORY.md Size Limit

The MEMORY.md file loaded into Claude Code interactive sessions is truncated after 200 lines. Keep it concise. Use topic-specific files (linked from MEMORY.md) for detailed notes.

OpenStripe's own `data/memory/MEMORY.md` has no enforced limit but large files slow down injection.
