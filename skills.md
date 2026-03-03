# Skills

Claude Code has 109 skills installed across two locations.

## Locations

| Location | Count | Source |
|----------|-------|--------|
| `~/.agents/skills/` | 102 | ClawHub (installed 2026-03-02) |
| `~/.claude/skills/` | 7 | N8N-specific skills |

---

## Install Command

```bash
npx skills add <owner/repo@skill> -g -y
```

---

## Skill Categories

### AI & Agents
- `claude-developer-platform` — Build apps with Claude API/Anthropic SDK
- `gemini` — Gemini CLI for Q&A, summaries, generation
- `research` — Deep research via Gemini CLI (background sub-agent)
- `deep-research` — Multi-step research with planning and decomposition
- `subagent-driven-development` — Execute plans with independent sub-tasks
- `dispatching-parallel-agents` — Coordinate 2+ independent parallel tasks
- `self-improving-agent` — Universal self-improving agent with multi-memory

### Development
- `git-essentials` — Git commands and workflows
- `github` — GitHub ops via gh CLI (issues, PRs, CI)
- `gh-issues` — Fetch issues, spawn fix agents, open PRs
- `github-actions-templates` — Production-ready GitHub Actions workflows
- `docker-essentials` — Docker container management
- `test-automator` — Automated test framework creation
- `test-driven-development` — TDD workflow (write tests before code)
- `refactoring-specialist` — Code structure and readability improvements
- `performance-engineer` — Application speed optimization
- `security-auditor` — OWASP Top 10 and vulnerability scanning
- `security-reviewer` — Security audits and code review
- `code-reviewer` — PR and code quality review
- `debugger` — Advanced bug diagnosis
- `systematic-debugging` — Structured debugging before proposing fixes

### Documentation & Planning
- `documentation-engineer` — Technical docs creation
- `writing-plans` — Implementation planning before coding
- `writing-skills` — Create/edit/verify skills
- `prd-planner` — Product requirements documents
- `architecting-solutions` — Technical architecture design
- `api-design-principles` — REST and GraphQL API design
- `api-designer` — API architecture
- `api-documenter` — OpenAPI/Swagger specs

### Infrastructure
- `pm2` — PM2 process management
- `proxmox` — Proxmox VE REST API (list/start/stop VMs, LXCs)
- `proxmox-full` — Full Proxmox management (snapshots, backups, storage)
- `linux-service-triage` — Diagnose Linux service issues
- `sysadmin-toolbox` — Sysadmin shell one-liners and tool discovery
- `ssh-essentials` — SSH commands, key management, tunneling
- `k8s-manifest-generator` — Kubernetes manifests
- `terraform-module-library` — Reusable Terraform modules
- `deployment-engineer` — CI/CD and deployment automation
- `deployment-pipeline-design` — Multi-stage CI/CD design
- `secrets-management` — Secrets in CI/CD (Vault, AWS Secrets Manager)

### N8N (in `~/.claude/skills/`)
- `n8n-workflow-automation` — Design N8N workflow JSON
- `n8n-workflow-patterns` — Proven N8N architectural patterns
- `n8n-node-configuration` — N8N node configuration guidance
- `n8n-expression-syntax` — N8N expression syntax and fixing errors
- `n8n-code-javascript` — JavaScript in N8N Code nodes
- `n8n-code-python` — Python in N8N Code nodes
- `n8n-mcp-tools-expert` — Using n8n-mcp MCP tools
- `n8n-validation-expert` — Interpret and fix N8N validation errors

### Web & Search
- `web-search` — DuckDuckGo web search (fixed to use `ddgs` package)
- `summarize` — Summarize URLs, podcasts, local files
- `youtube-transcript` — Extract YouTube transcripts
- `blogwatcher` — Monitor RSS/Atom feeds

### Communication
- `himalaya` — Email via IMAP/SMTP
- `discord` — Discord channel messaging
- `slack` — Slack control
- `xurl` — X (Twitter) API

### Productivity
- `notion` — Notion pages, databases, blocks
- `trello` — Trello boards, lists, cards
- `obsidian` — Obsidian vault management
- `caldav-calendar` — CalDAV calendar sync (iCloud, Google, Nextcloud)
- `gog` — Google Workspace (Gmail, Calendar, Drive, Sheets, Docs)
- `spotify-player` — Terminal Spotify playback

### Media & Files
- `video-frames` — Extract frames/clips with ffmpeg
- `camsnap` — Capture from RTSP/ONVIF cameras
- `openai-whisper-api` — Transcribe audio via OpenAI Whisper
- `openai-whisper` — Local Whisper (no API key)
- `nano-pdf` — Edit PDFs with natural language
- `markdown-converter` — Convert PDF/Word/PowerPoint to Markdown
- `openai-image-gen` — Batch image generation via OpenAI
- `nano-banana-pro` — Image generation via Gemini

### Database & APIs
- `postgres` — PostgreSQL management
- `jq` — JSON processing
- `mcporter` — MCP server management
- `goplaces` — Google Places API

### System & Meta
- `skill-router` — Routes requests to appropriate skill
- `skill-creator` — Create or update skills
- `clawhub` — Browse/install/publish skills from clawhub.com
- `skill-vetter` — Security vetting for new skills
- `auto-updater` — Apply improvements to skills
- `context-manager` — AI context management
- `memory-manager` — Local memory management
- `super-save` — Save project knowledge to memory
- `super-search` — Search coding memory
- `session-logs` — Search/analyze session logs
- `session-logger` — Save conversation history
- `model-usage` — Summarize per-model usage/costs
- `tmux` — Remote-control tmux sessions
- `weather` — Weather and forecasts
- `1password` — 1Password CLI setup and usage
- `proxmox` / `proxmox-full` — Proxmox VE management

### Workflow
- `workflow-orchestrator` — Multi-skill workflow coordination
- `auto-trigger` — Workflow automation hooks
- `executing-plans` — Execute implementation plans with checkpoints
- `planning-with-files` — Persistent markdown-based planning
- `brainstorming` — Pre-creative work brainstorming
- `prompt-engineering-patterns` — Advanced prompt techniques
- `architecture-patterns` — Clean/Hexagonal/DDD architecture
- `postmortem-writing` — Blameless postmortems
- `incident-runbook-templates` — Incident response runbooks

### Code Quality
- `simplify` — Review and fix code for quality
- `verification-before-completion` — Run tests before claiming done
- `requesting-code-review` — Request code review
- `receiving-code-review` — Handle review feedback
- `finishing-a-development-branch` — Branch integration guidance
- `using-git-worktrees` — Isolated feature work with git worktrees
- `create-pr` — Create PRs with documentation
- `commit-helper` — Conventional commit messages

---

## Skipped Skills (Security Concerns)
The following were not installed during the 2026-03-02 batch install:
- `proactive-agent` — suspicious behavior
- `find-skills` — suspicious
- `brave-search` — suspicious (Brave API used directly instead)
- `exa-web-search-free` — suspicious

---

## Adding New Skills

```bash
# Search ClawHub
npx clawhub search <query>

# Vet before installing
# Use skill-vetter skill to analyze first

# Install
npx skills add <owner/repo@skill> -g -y
```
