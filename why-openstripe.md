# Why OpenStripe Exists

## The Origin Story

OpenStripe was built out of a very specific frustration.

The goal was simple: run Claude tasks remotely via Telegram, with a proper queue, cost tracking, and a web UI to see what's going on. There are platforms that let you do this kind of thing — **OpenClaw** being one of them — but they come with a catch.

OpenClaw and similar platforms require you to use the **Claude API** directly (pay-per-token). Using a **Claude Pro/Max subscription** with these platforms violates Anthropic's Terms of Service. The subscription is intended for personal use via Claude.ai, not for routing tasks through third-party automation platforms.

So the options were:
1. Pay API rates for every token (expensive at scale)
2. Use a platform that's technically against TOS (risky)
3. Build something that works the right way

**OpenStripe chose option 3.**

---

## The Solution: Claude Code CLI

The key insight was **Claude Code** — Anthropic's official CLI tool.

Claude Code is designed to be run locally as an agent. It uses your Claude subscription legitimately, it's built by Anthropic, and it's explicitly intended for agentic, automated tasks. Running Claude Code programmatically is not against any TOS — it's exactly what it was built for.

OpenStripe wraps Claude Code in a proper platform:
- Spawns `claude` CLI as a subprocess per task
- Streams the JSON output in real time
- Stores everything in SQLite
- Exposes it all via REST API, WebSocket, and Telegram bot

The result is a self-hosted platform that gives you the full OpenClaw-style experience — Telegram bot, web dashboard, task queue, cost tracking, memory — but running on your own server, using your Claude subscription, legitimately.

---

## What This Means in Practice

| | OpenClaw / API-based | OpenStripe |
|--|----------------------|------------|
| Cost model | Pay per token (API pricing) | Claude subscription (flat rate) |
| TOS compliance | Subscription use violates TOS | Fully compliant (Claude Code) |
| Hosting | Their servers | Your server |
| Data privacy | Their infrastructure | 100% yours |
| Customisation | Limited | Full source access |
| Memory / context | Depends on platform | Built-in, self-hosted |

---

## Why Self-Hosted

Beyond the TOS issue, self-hosting has real advantages:

- **Privacy** — your tasks, prompts, and outputs never leave your server
- **Control** — modify anything, add integrations, change the UI
- **Cost** — Claude Pro/Max is a fixed monthly cost regardless of task volume
- **No vendor lock-in** — you own the data and the platform

---

## Current State

OpenStripe is a private, personal deployment. It's not a polished open-source product yet — it's a working system built for one user's needs. The documentation in this repo exists so that:

1. The setup can be reproduced if the server is lost
2. Future contributors or users can understand how it works
3. It could eventually be cleaned up and released publicly

If you're reading this as an outside contributor or someone considering deploying it yourself, see [deployment-guide.md](deployment-guide.md) for full setup instructions.
