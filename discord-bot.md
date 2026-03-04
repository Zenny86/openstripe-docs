# Discord Bot

## Overview

A hybrid Discord bot + Express webhook server for community management and n8n workflow integration.

- **Path:** `/home/claude-user/claude-projects/discord-bot/`
- **PM2 name:** `discord-bot`
- **Webhook port:** `3849`

---

## Architecture

```
Discord ←→ discord.js v14 bot
n8n     →  Express webhook (port 3849)
```

The same Node.js process runs both the Discord bot and the HTTP webhook server.

---

## Slash Commands

| Command | Description |
|---------|-------------|
| `/kick` | Kick a member |
| `/ban` | Ban a member |
| `/mute` / `/unmute` | Mute/unmute a member |
| `/clear` | Delete bulk messages |
| `/warn` | Issue a warning to a member |
| `/announce` | Post an announcement embed |
| `/serverinfo` | Show server stats |
| `/userinfo` | Show user profile |

---

## Auto-Moderation

- **Spam detection:** Flags repeated messages in quick succession
- **Bad word filter:** Configurable word list

---

## n8n Webhook Endpoints

| Method | Path | Body | Description |
|--------|------|------|-------------|
| `POST` | `/discord/send` | `{ channelId, message }` | Send message to channel |
| `POST` | `/discord/announce` | `{ channelId, message }` | Post announcement embed |
| `POST` | `/discord/dm` | `{ userId, message }` | Send DM to user |
| `GET` | `/discord/status` | — | Bot health check |

---

## Environment Variables

```env
DISCORD_BOT_TOKEN=...
GUILD_ID=...
LOG_CHANNEL_ID=...        # Channel for mod logs
WELCOME_CHANNEL_ID=...    # Channel for new member welcome messages
```

---

## PM2 Management

```bash
pm2 restart discord-bot
pm2 logs discord-bot
pm2 list
```

---

## Feedback Agent Integration

The bot can be wired to forward messages from a designated feedback channel to an n8n webhook, which triggers a Claude agent to respond autonomously.

**Example flow:**
1. User posts in `#feedback`
2. Bot reacts ⏳ while processing
3. Message forwarded to n8n → `POST /webhook/your-feedback-handler`
4. n8n runs Claude (via SSH or subprocess)
5. Claude reads config, generates reply, posts back to Discord
6. Bot reacts ✅ when done

---

## Social Media Analysis Integration

The bot supports a `!analyze` command that triggers a scraper + Claude analysis pipeline and posts results to a dedicated channel.

**Example flow:**
1. User runs `!analyze` in any channel
2. Scraper fetches recent posts from configured platforms (e.g. TikTok, Instagram)
3. Claude scores posts and generates recommendations
4. Embed report posted to `#social-analysis`

See [social-media-integrations.md](social-media-integrations.md) for full setup.
