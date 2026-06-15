---
name: workflows
description: Manage n8n Cloud workflows — list, view, create, update, activate/deactivate, check executions, and trigger via webhook. Use when the user asks about n8n workflows, automations, or wants to query/modify their n8n instance.
user-invocable: true
allowed-tools:
  - Bash(curl *)
  - Read
  - Write
---

# /n8n-manager:workflows — n8n Workflow Management

Manages workflows on the user's n8n instance via the n8n REST API.

Arguments passed: `$ARGUMENTS`

## Configuration

Requires two environment variables:

- `N8N_BASE_URL` — e.g. `https://saadetsengokstudio.app.n8n.cloud`
- `N8N_API_KEY` — generated in n8n under Settings > n8n API

If either is missing, tell the user how to set them (Settings > n8n API in
the n8n Cloud UI to create a key) and stop. Never hardcode the key or print
it back in full.

All API calls use base `${N8N_BASE_URL}/api/v1` with header
`X-N8N-API-KEY: ${N8N_API_KEY}`.

## Dispatch on arguments

Parse `$ARGUMENTS` (space-separated). If empty or unrecognized, show usage
(list the subcommands below).

### `list`

```bash
curl -s "$N8N_BASE_URL/api/v1/workflows" -H "X-N8N-API-KEY: $N8N_API_KEY"
```

Summarize: id, name, active status, updatedAt. Present as a table.

### `get <id>`

```bash
curl -s "$N8N_BASE_URL/api/v1/workflows/<id>" -H "X-N8N-API-KEY: $N8N_API_KEY"
```

Summarize the workflow: name, active status, node list (name + type), and
connections at a high level. Don't dump the raw JSON unless asked.

### `create <file.json>`

Read the local JSON file (must contain `name`, `nodes`, `connections`,
`settings`). POST it:

```bash
curl -s -X POST "$N8N_BASE_URL/api/v1/workflows" \
  -H "X-N8N-API-KEY: $N8N_API_KEY" -H "Content-Type: application/json" \
  --data-binary @<file.json>
```

Report the new workflow's id and editor URL
(`$N8N_BASE_URL/workflow/<id>`).

### `update <id> <file.json>`

1. GET the existing workflow first (see `get`) so edits are applied on top
   of current state, not blind overwrites.
2. Apply the requested change to the JSON (nodes/connections/settings).
3. PUT the result:

```bash
curl -s -X PUT "$N8N_BASE_URL/api/v1/workflows/<id>" \
  -H "X-N8N-API-KEY: $N8N_API_KEY" -H "Content-Type: application/json" \
  --data-binary @<file.json>
```

Only `name`, `nodes`, `connections`, and `settings` are accepted by the API
— strip other fields (id, createdAt, etc.) before sending.

### `activate <id>` / `deactivate <id>`

```bash
curl -s -X POST "$N8N_BASE_URL/api/v1/workflows/<id>/activate" -H "X-N8N-API-KEY: $N8N_API_KEY"
curl -s -X POST "$N8N_BASE_URL/api/v1/workflows/<id>/deactivate" -H "X-N8N-API-KEY: $N8N_API_KEY"
```

### `executions [workflow_id]`

```bash
curl -s "$N8N_BASE_URL/api/v1/executions?workflowId=<id>&limit=10" -H "X-N8N-API-KEY: $N8N_API_KEY"
```

Omit the `workflowId` filter to list recent executions across all
workflows. Summarize: id, workflow name, status, startedAt, stoppedAt.

### `trigger <webhook-path> [json-body]`

Webhook triggers do **not** use the API key — they hit the workflow's
webhook URL directly:

```bash
curl -s -X POST "$N8N_BASE_URL/webhook/<webhook-path>" \
  -H "Content-Type: application/json" -d '<json-body>'
```

If the workflow uses a *test* webhook, use `/webhook-test/<path>` instead
and tell the user the workflow must be in "listen for test event" mode.

## Implementation notes

- Pipe responses through `python3 -m json.tool` (or similar) for readability
  before summarizing — don't paste raw minified JSON into chat unless asked.
- Before `update`, always show the user a short diff-style summary of what
  will change and confirm if the change is structural (added/removed nodes),
  since this overwrites the live workflow.
- Activating a workflow makes it live (e.g. starts listening on webhooks /
  schedules) — treat `activate` as a meaningful state change and mention the
  effect when reporting success.
- Never echo `N8N_API_KEY` back into chat output.
