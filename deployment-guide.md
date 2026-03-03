# Deployment Guide

How to deploy OpenStripe from scratch on your own server.

---

## Prerequisites

- A Linux server (bare metal, VPS, or VM — Proxmox/PVE works great)
- A domain or local IP you can reach
- A Telegram account
- An Anthropic account with Claude Code CLI access

---

## Step 1: Server Setup

Any modern Linux distro works. This guide uses Debian/Ubuntu.

```bash
# Update system
apt update && apt upgrade -y

# Install Node.js (v20+)
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs

# Install git, build tools (needed for better-sqlite3)
apt install -y git build-essential python3

# Install PM2 globally
npm install -g pm2
```

---

## Step 2: Install Claude Code CLI

Claude Code is the AI engine that powers OpenStripe. You need an Anthropic account.

```bash
npm install -g @anthropic-ai/claude-code
```

Verify install:
```bash
claude --version
which claude   # note this path — you'll need it
```

Authenticate (run once interactively):
```bash
claude
# Follow prompts to log in with your Anthropic account
# Exit after login is confirmed
```

---

## Step 3: Create a Telegram Bot

1. Open Telegram → search for **@BotFather**
2. Send `/newbot`
3. Follow prompts → choose a name and username
4. BotFather gives you a **token** like `7123456789:AAF...` — save it

Get your personal chat ID:
1. Search for **@userinfobot** in Telegram
2. Start it → it replies with your chat ID (a number like `123456789`)

---

## Step 4: Clone and Configure OpenStripe

```bash
# Clone the repo (or copy your files)
git clone https://github.com/YOUR_USERNAME/openstripe.git
cd openstripe

# Install server dependencies
npm install

# Install and build frontend
cd client && npm install && npm run build && cd ..
```

Create your `.env` file:
```bash
cp .env.example .env   # if example exists
# or create from scratch:
nano .env
```

Paste and fill in:
```env
PORT=3848
BIND_HOST=0.0.0.0
JWT_SECRET=change-this-to-a-long-random-string

# Telegram
TELEGRAM_BOT_TOKEN=7123456789:AAF...your-bot-token...
TELEGRAM_ALLOWED_CHAT_IDS=123456789

# Claude defaults
CLAUDE_DEFAULT_MODEL=sonnet
CLAUDE_DEFAULT_BUDGET=5.00
CLAUDE_MAX_CONCURRENT=2
CLAUDE_TASK_TIMEOUT_MS=600000
```

Generate a strong JWT secret:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

---

## Step 5: Create First User

```bash
node scripts/create-user.js
# Enter username and password when prompted
```

This is your web UI login. You can create multiple users.

---

## Step 6: Start with PM2

```bash
pm2 start ecosystem.config.js
pm2 save
pm2 startup   # follow the printed command to enable auto-start on reboot
```

Verify it's running:
```bash
pm2 list
curl http://localhost:3848/api/health
# Should return: {"status":"ok","running_tasks":0}
```

---

## Step 7: Test Telegram

Send your bot a message. It should reply.

If it doesn't:
```bash
pm2 logs openstripe   # look for Telegram errors
```

Common issues:
- Wrong bot token → re-check from BotFather
- Chat ID not whitelisted → double-check `TELEGRAM_ALLOWED_CHAT_IDS` in `.env`
- Server firewall blocking outbound → Telegram polling uses outbound HTTPS, ensure port 443 out is open

---

## Step 8: Access the Web UI

Open a browser and go to:
```
http://YOUR_SERVER_IP:3848
```

Log in with the username/password you created in Step 5.

---

## Step 9: Optional — Reverse Proxy with Nginx

If you want HTTPS or a cleaner URL:

```bash
apt install -y nginx certbot python3-certbot-nginx
```

Create Nginx config (`/etc/nginx/sites-available/openstripe`):
```nginx
server {
    server_name openstripe.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:3848;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Enable and get SSL:
```bash
ln -s /etc/nginx/sites-available/openstripe /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
certbot --nginx -d openstripe.yourdomain.com
```

---

## Step 10: Optional — Voice Transcription (Groq)

1. Sign up at [console.groq.com](https://console.groq.com)
2. Create an API key
3. Add to `.env`:
   ```env
   GROQ_API_KEY=gsk_...
   ```
4. Restart: `pm2 restart openstripe`

Now send voice messages to your Telegram bot — they'll be transcribed and run as tasks.

---

## Step 11: Optional — N8N Integration

If you run N8N and want it to trigger Claude tasks:

1. Add to `.env`:
   ```env
   N8N_WEBHOOK_URL=http://your-n8n:5678/webhook/openstripe-callback
   N8N_WEBHOOK_SECRET=pick-a-secret
   ```
2. In N8N, create an HTTP Request node pointing to:
   ```
   POST http://your-openstripe:3848/api/webhooks/n8n
   Header: X-Webhook-Secret: pick-a-secret
   Body: { "prompt": "Your task here" }
   ```
3. Restart OpenStripe: `pm2 restart openstripe`

---

## Memory Setup (Optional but Recommended)

OpenStripe can inject persistent memory into every Claude task. The memory files live at `data/memory/`.

Create an initial memory file:
```bash
mkdir -p data/memory
cat > data/memory/MEMORY.md << 'EOF'
# Memory

## User
- Name: Your Name
- Preferences: concise responses

## Environment
- Server: your-server-ip
EOF
```

Claude will read this before every task and write back to it automatically when you instruct it to.

---

## Updating OpenStripe

```bash
cd /path/to/openstripe
git pull
npm install
cd client && npm run build && cd ..
pm2 restart openstripe
```

---

## Directory Structure After Setup

```
openstripe/
├── server/          # Node.js backend (don't edit while running)
├── client/dist/     # Built React frontend (served as static files)
├── data/
│   ├── dashboard.db # SQLite database (auto-created on first run)
│   ├── memory/      # Persistent memory files
│   └── uploads/     # Task file uploads
├── .env             # Your secrets (never commit this)
└── ecosystem.config.js
```

---

## Security Checklist

Before exposing to the internet:

- [ ] Change `JWT_SECRET` to a strong random value
- [ ] Set a strong web UI password (via `create-user.js`)
- [ ] Only whitelist your own Telegram chat IDs
- [ ] Use Nginx + HTTPS (certbot) instead of direct port exposure
- [ ] Set `BIND_HOST=127.0.0.1` if using Nginx (so port 3848 isn't publicly exposed)
- [ ] Set a `monthly_budget` in Settings to cap Claude spending
- [ ] Keep `.env` out of git (it's in `.gitignore` by default)
- [ ] Restrict server SSH access with key-based auth

---

## Minimum Hardware

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 1 core | 2+ cores |
| RAM | 512 MB | 2 GB |
| Disk | 5 GB | 20 GB |
| OS | Debian 11+ / Ubuntu 22.04+ | Same |

Claude tasks run as subprocesses — more RAM = more concurrent tasks safely.

---

## Cost Considerations

OpenStripe uses Claude via the Claude Code CLI, which consumes API credits from your Anthropic account.

- Default budget: $5.00 per task
- Default concurrency: 2 tasks at once
- Set a monthly budget cap in web UI → Settings

Monitor costs in the Dashboard → monthly spend is displayed.
