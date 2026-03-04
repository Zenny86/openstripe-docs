# Social Media Integrations

Two autonomous systems: a Discord feedback response agent and a social media analysis pipeline.

---

## 1. Feedback Response System

### Overview

Community feedback posted in a Discord channel is automatically routed to a Claude agent that reads, responds, and updates server configuration autonomously.

### Flow

```
Discord #feedback-channel
    → discord-bot forwards to n8n
    → n8n runs Claude (SSH or subprocess)
    → Claude reads/edits config (CLAUDE.md or similar)
    → Reply posted back in Discord
```

### Components

- **Discord channel:** Any designated feedback channel
- **n8n webhook:** `POST /webhook/feedback-handler`
- **Claude config file:** `CLAUDE.md` on the target server (defines agent context, changelog format, reply style)
- **Transport:** SSH key auth or local subprocess

### CLAUDE.md Agent Section

Claude is instructed via a dedicated section in `CLAUDE.md`:
- Understand current posting/content configuration
- Log feedback and update changelog
- Output replies in a structured `AGENT_RESULT` format for the bot to forward
- Operate autonomously for standard feedback — no human-in-the-loop required

### Example CLAUDE.md Section

```markdown
## Section 8 — Feedback Agent

You are responding to community feedback forwarded from Discord.

1. Read this file to understand current configuration
2. Address the feedback and update the changelog below
3. Reply in this format:

AGENT_RESULT:
<your reply here>
```

---

## 2. Social Media Analysis System

### Overview

Scrapes recent posts from TikTok and Instagram, runs them through Claude Sonnet for strategic analysis, and posts embed reports to a Discord channel.

### Flow

```
!analyze command in Discord
    → scraper fetches recent posts + thumbnails
    → unseen posts filtered via tracker file
    → Claude Sonnet analyzes all posts in one call
    → Embed reports posted to #social-analysis channel
```

### Scraper

- **TikTok:** via `yt-dlp` — bypasses bot detection, extracts metadata + thumbnails
- **Instagram:** via mobile `web_profile_info` API (no login required, public profiles)
- **Post tracker:** `tracker.json` — prevents re-analyzing already-seen posts

### Analysis (Claude Sonnet 4.6)

Each post is scored on:

| Metric | Description |
|--------|-------------|
| Hook strength | Does the opening grab attention in <3s? |
| Visual quality | Lighting, composition, production value |
| CTA effectiveness | Is there a clear call-to-action? |
| Posting time | Does the publish time match peak audience hours? |
| Algorithm signals | Comments, shares, watch time indicators |

Claude also benchmarks against competitor accounts in the same niche and outputs **3 actionable improvement tips** per post.

### Example Analysis Output

```
Post: "Behind the scenes printing a 40hr model"
- Hook: 7/10 — Good but could start with the finished result
- Visual: 9/10 — Clean lighting, good camera angle
- CTA: 4/10 — No CTA present
- Timing: ✅ Posted at 7pm (peak)
- Algorithm: High comment potential, consider asking a question

Improvements:
1. Add "Would you print this?" as end-frame CTA
2. Show finished model first, then reveal process (reverse hook)
3. Add trending audio from Creator Marketplace
```

### Discord Output

- **Channel:** `#social-analysis` (or any configured channel)
- One embed per analyzed post
- Triggered via `!analyze` command

### Triggering

```
In Discord: !analyze
```

---

## SSH Setup (for remote agent execution)

If running Claude on a separate server:

```bash
# Generate key if needed
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519

# Copy to target server
ssh-copy-id user@target-server

# Test
ssh user@target-server 'echo ok'
```

n8n SSH node or `Execute Command` node then handles the agent invocation.
