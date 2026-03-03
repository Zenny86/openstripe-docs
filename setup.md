# Setup & Configuration

## Prerequisites

- Node.js (system: `/usr/bin/node`)
- PM2 (`npm install -g pm2`)
- Claude Code CLI (`~/.local/bin/claude`)
- SQLite3 (bundled via better-sqlite3)

## Directory Structure

```
/home/claude-user/claude-projects/openstripe/
├── server/          # Node.js backend
├── client/          # React frontend (build → client/dist)
├── data/
│   ├── dashboard.db # SQLite database
│   ├── memory/      # Persistent memory files
│   └── uploads/     # Task file uploads
├── scripts/
│   └── create-user.js
├── .env             # Runtime secrets
└── ecosystem.config.js
```

## Environment Variables (`.env`)

```env
PORT=3848
BIND_HOST=0.0.0.0
JWT_SECRET=<strong-random-secret>

# Telegram
TELEGRAM_BOT_TOKEN=<from BotFather>
TELEGRAM_ALLOWED_CHAT_IDS=<comma-separated chat IDs>

# Claude defaults
CLAUDE_DEFAULT_MODEL=sonnet
CLAUDE_DEFAULT_BUDGET=5.00
CLAUDE_MAX_CONCURRENT=2
CLAUDE_TASK_TIMEOUT_MS=600000

# N8N (optional)
N8N_WEBHOOK_URL=http://localhost:5678/webhook/xxx
N8N_WEBHOOK_SECRET=<secret>

# Groq (optional — for voice transcription)
GROQ_API_KEY=<from groq.com>
```

Additional secrets are loaded from `~/.claude/secrets.env` at startup (N8N API key, etc.).

## Initial Setup

### 1. Install dependencies
```bash
cd /home/claude-user/claude-projects/openstripe
npm install
cd client && npm install && npm run build && cd ..
```

### 2. Create first user
```bash
node scripts/create-user.js
# Prompts for username and password
```

### 3. Start with PM2
```bash
pm2 start ecosystem.config.js
pm2 save  # persist across reboots
pm2 startup  # generate systemd service
```

### 4. Verify
```bash
pm2 list
curl http://localhost:3848/api/health
```

## PM2 Configuration (`ecosystem.config.js`)

```js
module.exports = {
  apps: [{
    name: 'openstripe',
    script: 'server/index.js',
    cwd: '/home/claude-user/claude-projects/openstripe',
    kill_timeout: 10000,   // 10s graceful shutdown before SIGKILL
    restart_delay: 2000,   // 2s delay for OS to free port 3848
    max_restarts: 10,
    min_uptime: '5s',
  }]
};
```

## Telegram Bot Setup

1. Create bot via [@BotFather](https://t.me/BotFather) → get `TELEGRAM_BOT_TOKEN`
2. Get your chat ID: message [@userinfobot](https://t.me/userinfobot) → get your chat ID
3. Add chat ID to `TELEGRAM_ALLOWED_CHAT_IDS` in `.env`
4. Restart: `pm2 restart openstripe`

## Groq Voice Transcription Setup

1. Sign up at groq.com → get API key
2. Add `GROQ_API_KEY=<key>` to `~/.claude/secrets.env`
3. Restart OpenStripe — voice messages in Telegram will now be transcribed automatically

## Building the Client

After frontend changes:
```bash
cd /home/claude-user/claude-projects/openstripe/client
npm run build
pm2 restart openstripe
```

The server serves `client/dist` as static files in production.

## Adding Users

```bash
cd /home/claude-user/claude-projects/openstripe
node scripts/create-user.js
```

There is no user management in the web UI — users are created via CLI only.

## Settings (Runtime)

Accessible via Settings page in web UI or `PUT /api/settings`:

| Setting | Default | Notes |
|---------|---------|-------|
| `default_model` | sonnet | claude-sonnet-4-6 |
| `default_budget` | 5.00 | USD per task |
| `max_concurrent` | 2 | Simultaneous claude processes |
| `task_timeout_ms` | 600000 | 10 minutes |
| `monthly_budget` | — | Optional cap |
