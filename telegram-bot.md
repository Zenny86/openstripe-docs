# Telegram Bot

## Overview

The Telegram bot is the primary interface for running Claude tasks. It supports commands, multi-task parsing, conversation continuity, and voice transcription.

Only whitelisted chat IDs can use the bot (configured in `.env` or via web UI Settings → Telegram Whitelist).

---

## Commands

| Command | Description |
|---------|-------------|
| `/newtask` | Start a fresh task (clears conversation context) |
| `/status` | Show running and recent tasks |
| `/cancel` | Cancel the most recent running task |
| `/history` | Show last 10 tasks |
| `/conversation` | Show current session info |
| `/model opus` | Switch to claude-opus-4-6 |
| `/model sonnet` | Switch to claude-sonnet-4-6 (default) |
| `/model haiku` | Switch to claude-haiku-4-5 |
| `/help` | Show help message |
| `/chatid` | Show your Telegram chat ID |

---

## Sending Tasks

### Single task
Just send a message — it becomes the task prompt.

### Multi-task (numbered list)
```
1. Search for the latest Node.js release notes
2. Summarize the top 3 changes
3. Check if package.json needs updating
```
Creates 3 separate tasks queued in order.

### Multi-task (separator)
```
Fix the bug in telegramBot.js where voice messages fail

---

Update the README with the new Groq setup instructions
```
Creates 2 tasks.

---

## Voice Messages

Send a voice message or audio file → bot transcribes via Groq Whisper → runs as task.

**Requirements:** `GROQ_API_KEY` must be set in `~/.claude/secrets.env` or `.env`.

**Model:** `whisper-large-v3-turbo` (fast, accurate)

---

## Progress Updates

While a task runs, the bot edits its progress message every ~4 seconds with:
- Current status emoji
- Tool activity (what Claude is doing)
- Progress percentage if available

On completion:
```
✅ Task completed

[output summary]

💰 $0.08 | 📊 1,234 → 456 tokens | ⏱ 45s
Monthly: $2.41 / $50.00
```

---

## Conversation Continuity

The bot maintains conversation context per chat:
- Each chat has a `session_id` stored in the `conversations` table
- New tasks automatically use `resume_session_id` from the last completed task
- Claude sees previous conversation history

To reset context (start fresh): use `/newtask` before your message.

---

## Model Selection

Per-chat model preference is saved:
```
/model opus     → claude-opus-4-6
/model sonnet   → claude-sonnet-4-6 (default)
/model haiku    → claude-haiku-4-5
```

---

## Message Limits

Telegram has a 4096 character message limit. Long outputs are automatically split into multiple messages.

---

## Whitelist Management

Via web UI → Settings → Telegram Whitelist:
- Add chat IDs with labels
- Remove access
- View current whitelist

Or directly via API:
```bash
# Add
curl -X POST http://localhost:3848/api/settings/telegram-whitelist \
  -H "Authorization: Bearer <token>" \
  -d '{"chat_id": "123456789", "label": "My phone"}'

# List
curl http://localhost:3848/api/settings/telegram-whitelist \
  -H "Authorization: Bearer <token>"
```

---

## Getting Your Chat ID

1. Message the bot `/chatid`
2. Or message [@userinfobot](https://t.me/userinfobot) on Telegram

---

## Troubleshooting

**Bot not responding:**
- Check `pm2 logs openstripe` for errors
- Verify `TELEGRAM_BOT_TOKEN` is set correctly in `.env`
- Confirm your chat ID is in `TELEGRAM_ALLOWED_CHAT_IDS`

**Voice transcription not working:**
- Confirm `GROQ_API_KEY` is in `~/.claude/secrets.env`
- Check `pm2 logs openstripe | grep groq`

**Task stuck as pending:**
- Check concurrency limit: `GET /api/settings` → `max_concurrent`
- Check if other tasks are running: `/status`
