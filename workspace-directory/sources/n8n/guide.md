# n8n Workflow Automation

Your self-hosted n8n Community Edition instance at `{{N8N_DOMAIN}}`, used for workflow automation.

## Scope

Full access to the n8n REST API for managing:
- Workflows (list, create, update, delete, activate/deactivate)
- Executions (list, get details, delete)
- Credentials (list, create, update, delete)
- Tags, variables, and audit events

## Guidelines

- **Base URL**: `https://{{N8N_DOMAIN}}/api/v1/`
- **Auth**: API key sent via `X-N8N-API-KEY` header
- Always confirm before deleting or deactivating workflows
- When updating workflows, fetch the current version first to avoid overwriting changes
- Execution data can be large — use `limit` and `cursor` for pagination

## Workflow JSON Structure

When creating or updating workflows, use this structure:

### Node Object
```json
{
  "id": "uuid-string",
  "name": "Unique Node Name",
  "type": "n8n-nodes-base.nodeType",
  "typeVersion": 2,
  "position": [250, 300],
  "parameters": {},
  "credentials": {
    "credentialType": { "id": "123", "name": "My Credential" }
  }
}
```

**Rules:**
- `id` — UUID, unique within the workflow
- `name` — human-readable, must be unique within the workflow
- `type` — exact node type string (use n8n-mcp `get_node` to look up)
- `typeVersion` — numeric, from node documentation
- `position` — `[x, y]` coordinates. Start at `[250, 300]`, space by 220px horizontally
- `credentials` — reference existing credentials by `id` and `name`

### Connections Object
```json
{
  "Source Node Name": {
    "main": [
      [
        { "node": "Target Node Name", "type": "main", "index": 0 }
      ]
    ]
  }
}
```

**Rules:**
- Uses **node names** (not IDs)
- `main` array index = source output index
- `index` field = target input index
- IF node: output 0 = true branch, output 1 = false branch
- Empty connections: `{}`

### Settings Object
```json
{
  "executionOrder": "v1",
  "saveManualExecutions": true,
  "saveExecutionProgress": true,
  "timezone": "Europe/Brussels"
}
```

### Full Example
```json
{
  "name": "Webhook to Slack",
  "nodes": [
    {
      "id": "a1b2c3d4-0000-0000-0000-000000000001",
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [250, 300],
      "parameters": { "path": "my-hook", "httpMethod": "POST" }
    },
    {
      "id": "a1b2c3d4-0000-0000-0000-000000000002",
      "name": "Send Slack Message",
      "type": "n8n-nodes-base.slack",
      "typeVersion": 2,
      "position": [470, 300],
      "parameters": { "channel": "#general", "text": "New webhook received!" },
      "credentials": { "slackApi": { "id": "1", "name": "Slack" } }
    }
  ],
  "connections": {
    "Webhook": {
      "main": [[{ "node": "Send Slack Message", "type": "main", "index": 0 }]]
    }
  },
  "settings": { "executionOrder": "v1" }
}
```

## API Reference

### Workflows

#### GET /workflows
List all workflows. Supports `?limit=N&cursor=...&tags=...&active=true|false`.

#### GET /workflows/{id}
Get a single workflow by ID, including its nodes and connections.

#### POST /workflows
Create a new workflow. Body: full workflow JSON (see structure above). Created inactive by default.

#### PUT /workflows/{id}
Update an existing workflow. Send the **full** workflow object (no partial updates).

#### DELETE /workflows/{id}
Delete a workflow by ID.

#### POST /workflows/{id}/activate
Activate a workflow.

#### POST /workflows/{id}/deactivate
Deactivate a workflow.

#### POST /workflows/{id}/transfer
Transfer workflow ownership.

### Executions

#### GET /executions
List executions. Supports `?workflowId=...&status=...&limit=N&cursor=...`.

#### GET /executions/{id}
Get execution details including input/output data.

#### DELETE /executions/{id}
Delete a specific execution.

### Credentials

#### GET /credentials
List all credentials. Supports `?limit=N&cursor=...`.

#### POST /credentials
Create new credentials. Body: `{ "name": "...", "type": "...", "data": {...} }`
**Note:** Credential secrets can't be reliably set via API. Configure credentials in the n8n UI.

#### PATCH /credentials/{id}
Update existing credentials.

#### DELETE /credentials/{id}
Delete credentials by ID.

### Tags

#### GET /tags
List all tags.

#### POST /tags
Create a new tag. Body: `{ "name": "..." }`

### Variables

#### GET /variables
List all environment variables.

#### POST /variables
Create a variable. Body: `{ "key": "...", "value": "..." }`

### Users

#### GET /users
List all users (owner/admin only).

### Audit

#### POST /audit
Generate a security audit report.

## Examples

- List all active workflows: `GET /workflows?active=true`
- Get recent failed executions: `GET /executions?status=error&limit=10`
- Search workflows by tag: `GET /workflows?tags=production`
- Create a workflow: `POST /workflows` with full JSON body
- Activate after creation: `POST /workflows/{id}/activate`
