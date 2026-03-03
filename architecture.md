# Architecture

## System Overview

```
                        ┌─────────────────────────────────────┐
                        │           OpenStripe Server          │
                        │              Port 3848               │
                        │                                      │
  Telegram ────────────►│  TelegramBot     TaskEngine          │
                        │       │               │              │
  Web UI ──────────────►│  REST API         ClaudeRunner       │
  (React SPA)           │       │               │              │
                        │  WebSocket ◄──────────┤              │
  n8n webhook ─────────►│       │           SQLite DB          │
                        │  MemoryService    (dashboard.db)     │
                        │       │               │              │
                        │  N8nBridge        data/memory/       │
                        └───────┴───────────────┴─────────────┘
                                │
                                ▼
                         claude CLI subprocess
                         (streams JSON output)
```

---

## Component Map

### Entry Point: `server/index.js`
- Loads `.env` and `~/.claude/secrets.env`
- Initializes Express, HTTP server, WebSocket
- Mounts all routes
- Starts TaskEngine and TelegramBot
- Serves React build in production
- Graceful shutdown on SIGTERM/SIGINT (8s drain)

### `server/services/taskEngine.js` — Orchestrator
The heart of OpenStripe. Polls DB every 2 seconds for pending tasks.

**Lifecycle:**
1. Finds pending task → marks `running` → stores PID
2. Calls `memoryService.injectMemory(prompt)` to prepend context
3. Spawns `claudeRunner.runClaude()` as subprocess
4. Streams chunks → inserts into `task_log_chunks`
5. Emits events via `notifier`
6. On completion → calls `memoryService.parseAndPersist(output)`
7. Recovers orphaned tasks (restart after crash) by re-queuing

**Key settings:**
- `maxConcurrent` — default 2 simultaneous tasks
- `taskTimeoutMs` — default 600,000ms (10 min)
- `defaultBudget` — default $5.00/task

### `server/services/claudeRunner.js` — Claude Subprocess
Spawns the `claude` CLI and streams output.

**How it runs claude:**
```
/home/claude-user/.local/bin/claude \
  -r <sessionId> \           # resume session (omitted for new)
  -p "<prompt>" \            # task prompt
  --output-format stream-json \
  --verbose \
  --model claude-sonnet-4-6 \
  --max-turns 50
```

**Output parsing:**
- Writes stdout/stderr to temp JSONL file
- Polls file every 500ms for new lines
- Parses event types: `assistant`, `content_block_delta`, `content_block_start`, `result`, `system`, `error`
- Emits `chunk` events back to TaskEngine
- Cleans up temp file on exit

### `server/services/telegramBot.js` — Telegram Interface
Polls Telegram API for messages from whitelisted chat IDs.

**Multi-task parsing:**
- Numbered list: `1. task A\n2. task B` → creates 2 tasks
- Separator: `task A\n---\ntask B` → creates 2 tasks

**Voice transcription:**
- Downloads audio file from Telegram
- Sends to Groq Whisper API (`whisper-large-v3-turbo`)
- Uses transcribed text as task prompt

**Conversation continuity:**
- Tracks `session_id` per chat in `conversations` table
- New tasks use `resume_session_id` from last completed task
- `/newtask` command clears session → starts fresh context

### `server/services/websocket.js` — Real-time Updates
WebSocket server on `ws://localhost:3848/ws`.

**Authentication:** JWT token in query param `?token=<jwt>`

**Message types:**
- `SUBSCRIBE_TASK` / `UNSUBSCRIBE_TASK` — per-task stream
- `SUBSCRIBE_ALL` — all task events

**Broadcast events:**
- `TASK_CREATED`, `TASK_UPDATED`, `TASK_COMPLETED`, `TASK_FAILED` → all clients
- `TASK_CHUNK` → subscribed clients only

Heartbeat ping/pong every 30s to detect dead connections.

### `server/services/memoryService.js` — Persistent Memory
**Memory directory:** `data/memory/`

**Files:**
- `MEMORY.md` — long-term durable facts, preferences, architecture
- `YYYY-MM-DD.md` — daily timestamped logs

**Inject flow (before task):**
1. Reads `MEMORY.md` contents
2. Reads today's daily log
3. Prepends as system context block in the prompt

**Extract flow (after task):**
1. Scans Claude output for memory blocks:
   ```
   <!-- MEMORY:LONG_TERM
   content
   /MEMORY:LONG_TERM -->

   <!-- MEMORY:DAILY
   content
   /MEMORY:DAILY -->
   ```
2. Appends LONG_TERM content to `MEMORY.md`
3. Appends DAILY content with timestamp to today's log
4. Updates FTS5 index

**Search:** SQLite FTS5 virtual table (`memory_fts`) for full-text search across all memory files.

### `server/services/notifier.js` — Event Bus
Bridges TaskEngine events to WebSocket clients, Telegram, and n8n.

- TaskEngine emits → Notifier receives → broadcasts to appropriate channels
- Sanitizes task data before broadcast (strips full prompt/output for `TASK_UPDATED`)

### `server/services/n8nBridge.js` — N8N Integration
On task completion or failure: POSTs callback to n8n webhook URL.

**Payload:**
```json
{
  "taskId": "uuid",
  "status": "completed|failed",
  "output": "...",
  "error": null,
  "cost_usd": 0.05,
  "usage": { "input": 1234, "output": 567 },
  "duration_ms": 12000
}
```

---

## Database Schema

**File:** `data/dashboard.db` (SQLite, WAL mode)

### `tasks`
| Column | Type | Notes |
|--------|------|-------|
| `id` | INTEGER PK | Auto |
| `external_id` | TEXT | UUID |
| `title` | TEXT | Auto-generated or user-set |
| `prompt` | TEXT | Full task prompt |
| `status` | TEXT | pending/running/completed/failed/cancelled |
| `model` | TEXT | claude-sonnet-4-6, etc. |
| `max_budget_usd` | REAL | Per-task budget cap |
| `permission_mode` | TEXT | default/auto |
| `working_directory` | TEXT | Working dir for claude |
| `session_id` | TEXT | Claude session UUID |
| `source` | TEXT | web/telegram/n8n |
| `source_chat_id` | TEXT | Telegram chat ID |
| `progress_message_id` | INTEGER | Telegram message to edit |
| `pid` | INTEGER | Claude subprocess PID |
| `exit_code` | INTEGER | Process exit code |
| `output_text` | TEXT | Full output |
| `output_summary` | TEXT | Condensed summary |
| `token_input` | INTEGER | Input tokens |
| `token_output` | INTEGER | Output tokens |
| `cost_usd` | REAL | Actual cost |
| `error_message` | TEXT | Error if failed |
| `created_at` | DATETIME | |
| `started_at` | DATETIME | |
| `completed_at` | DATETIME | |
| `conversation_id` | TEXT | Telegram conversation |
| `resume_session_id` | TEXT | Session to resume |

### `task_log_chunks`
| Column | Type | Notes |
|--------|------|-------|
| `id` | INTEGER PK | |
| `task_id` | INTEGER FK | → tasks.id |
| `chunk_type` | TEXT | text/tool_use/tool_result/system/error |
| `content` | TEXT | Raw content |
| `sequence` | INTEGER | Ordering |
| `created_at` | DATETIME | |

Indexed on `(task_id, sequence)` for efficient streaming pagination.

### `users`
| Column | Type |
|--------|------|
| `id` | INTEGER PK |
| `username` | TEXT UNIQUE |
| `password_hash` | TEXT |
| `created_at` | DATETIME |

### `conversations`
Tracks per-Telegram-chat session state.
| Column | Notes |
|--------|-------|
| `chat_id` | Telegram chat ID |
| `session_id` | Last Claude session |
| `model` | Current model preference |
| `last_task_id` | Last completed task |

### `telegram_whitelist`
| Column | Notes |
|--------|-------|
| `chat_id` | Allowed Telegram chat ID |
| `label` | Friendly name |

### `settings`
Key-value store for runtime config (model, budget, concurrency, timeout, monthly budget).

### `memory_fts`
SQLite FTS5 virtual table for full-text search over memory files.

---

## Data Flow: Task Lifecycle

```
1. INPUT (Telegram/Web/n8n)
   └─► POST /api/tasks → DB: status=pending

2. PICK UP (TaskEngine polls every 2s)
   └─► status=running, PID stored
   └─► memoryService.injectMemory(prompt)
   └─► claudeRunner.runClaude(augmentedPrompt)

3. EXECUTION (Claude subprocess)
   └─► streams JSON events to temp file
   └─► claudeRunner polls file, emits chunks
   └─► chunks → DB task_log_chunks
   └─► notifier → WebSocket broadcast + Telegram updates

4. COMPLETION
   └─► status=completed/failed
   └─► memoryService.parseAndPersist(output)
   └─► notifier → final broadcast
   └─► Telegram: sends result message
   └─► n8nBridge: callback webhook (if n8n task)
```

---

## Port Map

| Service | Port |
|---------|------|
| OpenStripe (HTTP + WS) | 3848 |
| N8N | 5678 |
| (Killed) claude-dashboard | 3847 |
