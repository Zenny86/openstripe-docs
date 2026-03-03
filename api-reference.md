# API Reference

Base URL: `http://localhost:3848`

Authentication: `Authorization: Bearer <jwt>` header on all protected endpoints.

Get a JWT via `POST /api/auth/login`.

---

## Authentication

### `POST /api/auth/login`
No auth required.
```json
// Request
{ "username": "admin", "password": "secret" }

// Response 200
{ "token": "eyJ...", "user": { "id": 1, "username": "admin" } }
```

### `POST /api/auth/logout`
Auth required. Clears server-side session.

### `GET /api/auth/me`
Auth required. Returns current user info.

### `POST /api/auth/change-password`
Auth required.
```json
{ "currentPassword": "old", "newPassword": "new" }
```

---

## Tasks

### `GET /api/tasks`
Auth required. List tasks with optional filters.

**Query params:**
| Param | Values | Default |
|-------|--------|---------|
| `status` | pending/running/completed/failed/cancelled | all |
| `source` | web/telegram/n8n | all |
| `search` | text string | — |
| `limit` | integer | 20 |
| `offset` | integer | 0 |

**Response:**
```json
{
  "tasks": [...],
  "total": 142,
  "limit": 20,
  "offset": 0
}
```

### `GET /api/tasks/:id`
Auth required. Full task detail including metadata.

### `GET /api/tasks/:id/logs`
Auth required. Paginated log chunks.

**Query params:**
- `after` — sequence number (for pagination)
- `limit` — default 100

**Response:**
```json
{
  "chunks": [
    { "id": 1, "chunk_type": "text", "content": "...", "sequence": 1 }
  ],
  "hasMore": true
}
```

### `POST /api/tasks`
Auth required. Create a new task.

```json
{
  "prompt": "Search for the latest news on AI",
  "title": "AI news search",           // optional
  "model": "claude-sonnet-4-6",        // optional, uses default
  "max_budget_usd": 2.00,              // optional
  "permission_mode": "default",        // optional
  "working_directory": "/home/...",    // optional
  "resume_session_id": "uuid"          // optional, continue conversation
}
```

**Response:**
```json
{ "task": { "id": 42, "external_id": "uuid", "status": "pending", ... } }
```

### `POST /api/tasks/:id/cancel`
Auth required. Sends SIGTERM to claude subprocess.

### `POST /api/tasks/:id/retry`
Auth required. Clones the task and re-queues it.

### `DELETE /api/tasks/:id`
Auth required. Deletes task, log chunks, and associated files.

---

## Dashboard

### `GET /api/dashboard/stats`
Auth required. Aggregated metrics.

**Response:**
```json
{
  "counts": {
    "total": 142,
    "pending": 1,
    "running": 2,
    "completed": 130,
    "failed": 9
  },
  "daily": [
    { "date": "2026-03-03", "count": 14, "cost": 1.23 }
  ],
  "recent": [...],
  "bySource": {
    "web": 45,
    "telegram": 90,
    "n8n": 7
  },
  "monthlyBudget": {
    "spent": 24.50,
    "limit": 50.00
  }
}
```

---

## Settings

### `GET /api/settings`
Auth required. All current settings.

### `PUT /api/settings`
Auth required.
```json
{
  "default_model": "claude-sonnet-4-6",
  "default_budget": 5.00,
  "max_concurrent": 2,
  "task_timeout_ms": 600000,
  "monthly_budget": 50.00
}
```

### `GET /api/settings/telegram-whitelist`
Auth required.

### `POST /api/settings/telegram-whitelist`
Auth required.
```json
{ "chat_id": "123456789", "label": "My iPhone" }
```

### `DELETE /api/settings/telegram-whitelist/:chatId`
Auth required.

---

## Memory

### `GET /api/memory`
Auth required. Lists all memory files with dates.

### `GET /api/memory/file?name=MEMORY.md`
Auth required. Returns file contents.

### `PUT /api/memory/file`
Auth required. Overwrites MEMORY.md.
```json
{ "name": "MEMORY.md", "content": "# Memory\n..." }
```

### `GET /api/memory/search?q=query`
Auth required. Full-text search across all memory files.

### `POST /api/memory/rebuild-index`
Auth required. Rebuilds FTS5 search index.

---

## Webhooks

### `POST /api/webhooks/n8n`
No JWT required. Authenticated via header.

**Headers:** `X-Webhook-Secret: <N8N_WEBHOOK_SECRET>`

**Body:**
```json
{
  "prompt": "Run this task",
  "title": "Task from n8n",
  "model": "sonnet",
  "max_budget_usd": 3.00
}
```

---

## Health

### `GET /api/health`
No auth required.
```json
{ "status": "ok", "running_tasks": 2 }
```

---

## WebSocket

**URL:** `ws://localhost:3848/ws?token=<jwt>`

### Client → Server messages
```json
// Subscribe to specific task
{ "type": "SUBSCRIBE_TASK", "taskId": 42 }

// Unsubscribe
{ "type": "UNSUBSCRIBE_TASK", "taskId": 42 }

// Subscribe to all events
{ "type": "SUBSCRIBE_ALL" }
```

### Server → Client messages
```json
// New task created
{ "type": "TASK_CREATED", "task": { ... } }

// Task status changed
{ "type": "TASK_UPDATED", "task": { ... } }

// Log chunk (subscribed tasks only)
{ "type": "TASK_CHUNK", "taskId": 42, "chunk": { "type": "text", "content": "..." } }

// Task finished
{ "type": "TASK_COMPLETED", "task": { ... } }

// Task failed
{ "type": "TASK_FAILED", "task": { ... } }
```
