# Troubleshooting

## OpenStripe Won't Start / Port Conflict

**Symptoms:** `pm2 logs openstripe` shows `EADDRINUSE :::3848`, high restart count

**Cause:** `node server/index.js` was run directly while PM2 was active, or a previous process didn't release the port.

**Fix:**
```bash
pm2 delete openstripe
lsof -i :3848          # find if anything still owns the port
kill -9 <pid>          # if needed
sleep 3
pm2 start ecosystem.config.js
pm2 save
```

**Prevention:** NEVER run `node server/index.js` directly. Always use `pm2 restart openstripe`.

---

## Telegram Bot Not Responding

**Check 1:** Is OpenStripe running?
```bash
pm2 list
curl http://localhost:3848/api/health
```

**Check 2:** Is the token valid?
```bash
pm2 logs openstripe | grep telegram
```

**Check 3:** Is your chat ID whitelisted?
- Check `TELEGRAM_ALLOWED_CHAT_IDS` in `.env`
- Or web UI → Settings → Telegram Whitelist

**Check 4:** Did you restart after `.env` changes?
```bash
pm2 restart openstripe
```

---

## Task Stuck in Pending

**Cause 1:** Max concurrent limit reached.
```bash
curl http://localhost:3848/api/health
# Check running_tasks vs max_concurrent setting
```

**Fix:** Cancel a running task or increase concurrency in Settings.

**Cause 2:** TaskEngine not polling (server issue).
```bash
pm2 restart openstripe
```

---

## Task Fails Immediately

**Check logs:**
```bash
# Via web UI: Task Detail → log stream
# Via DB:
sqlite3 /home/claude-user/claude-projects/openstripe/data/dashboard.db \
  "SELECT content FROM task_log_chunks WHERE task_id = <id> ORDER BY sequence"
```

**Common causes:**
- Claude binary not found: check `/home/claude-user/.local/bin/claude`
- Out of budget: check `max_budget_usd` setting
- Invalid model name: check model setting
- Session not found (for resume tasks): use a new session

---

## Voice Transcription Not Working

**Check:** Is GROQ_API_KEY set?
```bash
source ~/.claude/secrets.env && echo $GROQ_API_KEY
```

**Check:** PM2 logs for Groq errors
```bash
pm2 logs openstripe | grep -i groq
```

**Fix:** Add key to `~/.claude/secrets.env`, restart:
```bash
echo "GROQ_API_KEY=your-key" >> ~/.claude/secrets.env
pm2 restart openstripe
```

---

## Web UI Not Loading

**Check 1:** Is client built?
```bash
ls /home/claude-user/claude-projects/openstripe/client/dist
```

**Fix:** Rebuild
```bash
cd /home/claude-user/claude-projects/openstripe/client
npm run build
pm2 restart openstripe
```

**Check 2:** Is server running?
```bash
curl http://localhost:3848/api/health
```

---

## Memory Not Persisting

**Check:** Are memory blocks formatted correctly?
```
<!-- MEMORY:LONG_TERM
content
/MEMORY:LONG_TERM -->
```

**Check:** Task output stored in DB?
```bash
sqlite3 .../data/dashboard.db "SELECT output_text FROM tasks WHERE id = <id>"
```

**Check:** Memory files exist?
```bash
ls /home/claude-user/claude-projects/openstripe/data/memory/
```

**Fix:** Rebuild FTS index:
```bash
curl -X POST http://localhost:3848/api/memory/rebuild-index \
  -H "Authorization: Bearer <token>"
```

---

## Claude Code Session Issues

**Can't resume session:**
```bash
# List recent sessions
ls -lt ~/.claude/projects/-home-claude-user/*.jsonl | head -5

# Resume specific
claude -r <uuid>

# If hooks are broken, bypass them
claude -r <uuid> --settings '{"hooks":{}}'
```

**SSH recovery (if all else fails):**
```bash
ssh claude-user@<server-ip>
cd /home/claude-user && claude -c
```

---

## GitHub Token Issues

**Symptoms:** `gh` commands fail with authentication errors

**Fix:**
```bash
source ~/.claude/secrets.env
gh auth login --with-token <<< "$GITHUB_TOKEN"
gh auth status
```

---

## N8N MCP Not Working

**Check wrapper script:**
```bash
cat ~/.claude/n8n-mcp-wrapper.sh
bash ~/.claude/n8n-mcp-wrapper.sh  # test it directly
```

**Check API key:**
```bash
source ~/.claude/secrets.env && echo $N8N_API_KEY
```

---

## Database Corruption

**Backup first:**
```bash
cp data/dashboard.db data/dashboard.db.bak
```

**Check integrity:**
```bash
sqlite3 data/dashboard.db "PRAGMA integrity_check"
```

**WAL checkpoint:**
```bash
sqlite3 data/dashboard.db "PRAGMA wal_checkpoint(FULL)"
```

---

## Checking Everything at Once

```bash
#!/bin/bash
echo "=== PM2 Status ==="
pm2 list

echo "=== Health Check ==="
curl -s http://localhost:3848/api/health | python3 -m json.tool

echo "=== Recent Logs ==="
pm2 logs openstripe --lines 20 --nostream

echo "=== Port 3848 ==="
lsof -i :3848

echo "=== Secrets Loaded ==="
source ~/.claude/secrets.env
echo "GITHUB_TOKEN: ${GITHUB_TOKEN:0:20}..."
echo "GROQ_API_KEY: ${GROQ_API_KEY:0:10}..."
echo "BRAVE_API_KEY: ${BRAVE_API_KEY:0:10}..."
```
