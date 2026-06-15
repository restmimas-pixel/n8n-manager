# n8n-manager

A Claude Code plugin for managing n8n Cloud workflows: list, view, create,
update, activate/deactivate, inspect executions, and trigger workflows via
webhook — directly from a Claude Code session.

## Setup

1. In n8n, go to **Settings > n8n API** and create an API key.
2. Set these environment variables in your shell (or `.env`, loaded by your
   shell profile):

   ```
   N8N_BASE_URL=https://your-instance.app.n8n.cloud
   N8N_API_KEY=<your-api-key>
   ```

## Usage

```
/n8n-manager:workflows list
/n8n-manager:workflows get <workflow-id>
/n8n-manager:workflows create workflow.json
/n8n-manager:workflows update <workflow-id> workflow.json
/n8n-manager:workflows activate <workflow-id>
/n8n-manager:workflows deactivate <workflow-id>
/n8n-manager:workflows executions [workflow-id]
/n8n-manager:workflows trigger <webhook-path> '{"key":"value"}'
```
