# PM2 Management — CRITICAL

## The Golden Rule

**NEVER run `node server/index.js` directly.** PM2 owns port 3848. Running node directly while PM2 is active causes:
- `EADDRINUSE` crash loop
- PM2 restarts repeatedly (up to `max_restarts: 10`)
- Telegram goes dark
- Recovery requires manual `pm2 delete openstripe && pm2 start ecosystem.config.js`

## Essential Commands

```bash
# Restart after code changes (ALWAYS use this)
pm2 restart openstripe

# View status
pm2 list

# View live logs
pm2 logs openstripe

# View last 100 lines
pm2 logs openstripe --lines 100

# Stop (Telegram will go offline)
pm2 stop openstripe

# Full stop and remove from PM2
pm2 delete openstripe

# Re-register after delete
cd /home/claude-user/claude-projects/openstripe
pm2 start ecosystem.config.js
pm2 save
```

## Checking Health

```bash
curl http://localhost:3848/api/health
# Returns: {"status":"ok","running_tasks":0}
```

## Recovery After Crash Loop

If you see repeated restarts in `pm2 list`:

```bash
# 1. Stop everything
pm2 delete openstripe

# 2. Check if port is still in use
lsof -i :3848

# 3. Kill any orphaned node processes if needed
kill -9 <pid>

# 4. Wait 2-3 seconds, then restart
pm2 start ecosystem.config.js
pm2 save
```

## Crash Loop Symptoms

Signs that OpenStripe is in a crash loop:
- `pm2 list` shows high restart count
- `pm2 logs openstripe` shows repeated `Error: listen EADDRINUSE :::3848`
- Telegram bot not responding
- Web UI returns connection refused

## Auto-Start on Boot

```bash
pm2 startup    # generates systemd command — run it as shown
pm2 save       # saves current process list
```

This ensures OpenStripe restarts automatically after server reboot.

## Log Rotation

```bash
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 5
```

## Viewing Claude Subprocess Logs

Claude subprocess output goes to task_log_chunks in the DB, not PM2 logs. To see raw output:
- Web UI → Task Detail
- Telegram: task completion message
- DB: `SELECT content FROM task_log_chunks WHERE task_id = X ORDER BY sequence`
