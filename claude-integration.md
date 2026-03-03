# Claude Integration

## How OpenStripe Runs Claude

OpenStripe spawns the Claude Code CLI as a subprocess for each task. It does not use the Claude API directly — it uses the `claude` binary which manages its own authentication and session state.

---

## Claude Binary

**Path:** `/home/claude-user/.local/bin/claude`

This is the Claude Code CLI installed via `npm install -g @anthropic-ai/claude-code` or equivalent.

---

## Command Structure

For a new task:
```bash
/home/claude-user/.local/bin/claude \
  -p "Your task prompt here" \
  --output-format stream-json \
  --verbose \
  --model claude-sonnet-4-6 \
  --max-turns 50
```

For resuming a conversation:
```bash
/home/claude-user/.local/bin/claude \
  -r <session_uuid> \
  -p "Follow-up task prompt" \
  --output-format stream-json \
  --verbose \
  --model claude-sonnet-4-6
```

---

## Output Format

Claude outputs newline-delimited JSON events to stdout. Each line is a JSON object with a `type` field.

**Key event types:**

| Type | Description |
|------|-------------|
| `assistant` | Full message block (non-streaming) |
| `content_block_start` | Tool invocation start |
| `content_block_delta` | Streaming text delta |
| `result` | Final result with session_id, usage, cost |
| `system` | System events (session info, etc.) |
| `error` | Error events |

**Example `result` event:**
```json
{
  "type": "result",
  "session_id": "uuid-here",
  "usage": {
    "input_tokens": 1234,
    "output_tokens": 567
  },
  "cost_usd": 0.048,
  "result": "Task completed successfully."
}
```

---

## Streaming Implementation

`claudeRunner.js` uses a polling approach rather than direct stream parsing:

1. Claude subprocess stdout/stderr redirected to a temp JSONL file (`/tmp/claude-task-<uuid>.jsonl`)
2. Runner polls the file every 500ms for new lines
3. Each new line is parsed as JSON and emitted as a `chunk` event
4. On subprocess exit: reads remaining lines, emits final `complete` or `error`
5. Temp file deleted on cleanup

**Why polling instead of pipe?** More resilient to buffering issues and allows reattachment if needed.

---

## Session Management

Claude Code maintains conversation sessions identified by UUID.

**Session lifecycle:**
1. New task → no `-r` flag → Claude creates new session → returns `session_id` in result
2. OpenStripe stores `session_id` on the task
3. Telegram conversation → `conversations` table stores last `session_id`
4. Next task from same chat → uses `resume_session_id` → continues conversation

**Session files** are stored by Claude Code at:
```
~/.claude/projects/-home-claude-user/<session_uuid>.jsonl
```

---

## Models

Available models (as of 2026-03):

| Model | ID | Speed | Cost |
|-------|----|-------|------|
| Claude Opus 4.6 | `claude-opus-4-6` | Slow | High |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | Medium | Medium |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | Fast | Low |

Default: `claude-sonnet-4-6`

Set per-chat via Telegram `/model` command or globally via Settings.

---

## Budget & Cost Tracking

- Each task has a `max_budget_usd` (default $5.00)
- Claude Code enforces this internally via `--max-budget` flag
- Actual cost returned in `result` event `cost_usd` field
- OpenStripe stores `token_input`, `token_output`, `cost_usd` per task
- Monthly budget tracked in settings → compared vs sum of task costs

---

## Timeout

Default: 600,000ms (10 minutes)

TaskEngine sets a watchdog timer. On timeout:
1. Sends SIGTERM to claude subprocess
2. Waits 5s
3. Sends SIGKILL if still alive
4. Marks task as `failed` with timeout error message

---

## Permission Mode

Claude Code has permission modes that control what it can do without asking:

| Mode | Behavior |
|------|----------|
| `default` | Asks for dangerous operations |
| `auto` | Auto-approves most operations |

Set per-task or globally. Most tasks use `default`.

---

## Hooks

Claude Code hooks execute shell scripts on tool events. Configured in `~/.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [{ "command": "~/.claude/pre-tool-relay.sh" }],
    "PostToolUse": [{ "command": "~/.claude/post-tool-relay.sh" }],
    "Stop": [{ "command": "~/.claude/post-tool-relay.sh" }]
  }
}
```

These relay tool events to external services (previously to claude-dashboard). The hook scripts are at:
```
/home/claude-user/claude-projects/claude-dashboard/server/hooks/pre-tool-relay.sh
/home/claude-user/claude-projects/claude-dashboard/server/hooks/post-tool-relay.sh
```

---

## Skills

Claude Code has 109 skills installed:

- **102 skills** in `~/.agents/skills/` (installed from ClawHub)
- **7 skills** in `~/.claude/skills/` (n8n-specific skills)

Skills are invoked with the `Skill` tool using the skill name. See [skills.md](skills.md) for the full catalog.

---

## Interactive Sessions

For direct Claude Code sessions (not via OpenStripe):

```bash
# Start new session
cd /home/claude-user && claude

# Continue last session
claude -c

# Resume specific session by UUID
claude -r <uuid>

# Resume bypassing hooks (if hooks are broken)
claude -r --settings '{"hooks":{}}'
```

Session files: `~/.claude/projects/-home-claude-user/*.jsonl`
