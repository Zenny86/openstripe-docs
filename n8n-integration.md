# N8N Integration

## Overview

OpenStripe integrates with N8N in two directions:
1. **N8N → OpenStripe:** N8N triggers tasks via webhook
2. **OpenStripe → N8N:** OpenStripe calls back N8N on task completion

---

## N8N Instance

- **URL:** `http://localhost:5678`
- **API key:** stored in `~/.claude/secrets.env` as `N8N_API_KEY`
- **MCP integration:** `~/.claude/n8n-mcp-wrapper.sh` (sources secrets, runs npx n8n-mcp)

---

## Direction 1: N8N → OpenStripe (Trigger Tasks)

### Webhook Endpoint

```
POST http://<openstripe-host>:3848/api/webhooks/n8n
Header: X-Webhook-Secret: <N8N_WEBHOOK_SECRET>
```

### Request Body
```json
{
  "prompt": "Task description for Claude",
  "title": "Optional task title",
  "model": "sonnet",
  "max_budget_usd": 3.00
}
```

### Authentication
The webhook uses a shared secret (`N8N_WEBHOOK_SECRET` in `.env`) rather than JWT. Set the same value in both OpenStripe's `.env` and your N8N webhook node.

### N8N Node Setup
- Node type: HTTP Request
- Method: POST
- URL: `http://<openstripe-ip>:3848/api/webhooks/n8n`
- Headers: `X-Webhook-Secret: <your-secret>`
- Body: JSON with `prompt` field

---

## Direction 2: OpenStripe → N8N (Callbacks)

When a task created via n8n webhook completes or fails, OpenStripe calls back to a configured N8N webhook URL.

### Configuration (`.env`)
```env
N8N_WEBHOOK_URL=http://localhost:5678/webhook/openstripe-callback
N8N_WEBHOOK_SECRET=<same-secret>
```

### Callback Payload
```json
{
  "taskId": "uuid",
  "status": "completed",
  "output": "Full task output text",
  "error": null,
  "cost_usd": 0.05,
  "usage": {
    "input_tokens": 1234,
    "output_tokens": 567
  },
  "duration_ms": 45000
}
```

### N8N Webhook Trigger Setup
- Create a Webhook node in N8N to receive callbacks
- Set it to POST method
- Use the URL in `N8N_WEBHOOK_URL`

---

## N8N MCP Tools

Claude Code can manage N8N workflows directly via MCP:

```bash
# Available via n8n-mcp MCP server
# Example usage in Claude sessions:
# - List workflows
# - Execute workflows
# - Get execution results
# - Create/update workflows
```

The N8N MCP server is loaded automatically in Claude Code sessions. Use it to build and trigger N8N workflows from Claude directly.

---

## Common Patterns

### Pattern 1: Scheduled AI Task
1. N8N Schedule Trigger → HTTP Request → OpenStripe webhook
2. OpenStripe runs Claude task
3. On completion → callback to N8N → N8N sends result via email/Slack

### Pattern 2: Event-Driven Processing
1. N8N receives external event (form submit, webhook, etc.)
2. N8N formats prompt → sends to OpenStripe
3. OpenStripe processes → calls back N8N
4. N8N routes result to appropriate destination

### Pattern 3: Workflow Automation
1. Claude session → uses n8n-mcp tools to create/trigger N8N workflow
2. N8N workflow runs → calls back OpenStripe with results
3. Claude continues task based on results
