---
name: "n8n Build"
description: "Build and deploy a new n8n workflow from scratch"
requiredSources:
  - n8n-mcp
  - n8n
---

# Build an n8n Workflow

You are building a workflow on your n8n instance at `{{N8N_DOMAIN}}`.

## Process

### 1. Understand Requirements
Ask the user (if not already clear):
- **Trigger**: What starts the workflow? (webhook, schedule, app event, manual)
- **Actions**: What should happen? (send message, update database, call API, etc.)
- **Logic**: Any conditions, loops, or transformations needed?

### 2. Search Templates First
Before building from scratch, always check for existing templates:
```
n8n-mcp → search_templates("description of the workflow")
```

If a suitable template is found:
- Get the full template: `n8n-mcp → get_template(id)`
- Customize credentials, parameters, and nodes to match the user's needs
- Skip to step 5 (Validate Expressions)

If no template matches, proceed to step 3.

### 3. Look Up Every Node
For EVERY node you plan to use, you MUST look it up:

```
n8n-mcp → search_nodes("webhook")                    # Find the right node
n8n-mcp → get_node("n8n-nodes-base.webhook")          # Get exact params + typeVersion
```

**NEVER guess node type strings or parameters.** Always use `search_nodes` + `get_node`.

**Verify credentials exist.** For every node that requires credentials, run `GET /credentials` and confirm the credential type and ID exist on the instance. If a required credential is missing, flag it to the user — they must configure it in the n8n UI before the workflow can run. `validate_workflow` does NOT check credential existence.

**Document your lookups.** After completing all lookups, list them as a receipt:
```
Node lookups completed:
- n8n-nodes-base.webhook → typeVersion 2.1
- n8n-nodes-base.if → typeVersion 2.2
- n8n-nodes-base.slack → typeVersion 2.2
```
This ensures every node was actually looked up and nothing was guessed from memory.

### 4. Assemble the Workflow JSON

Use this structure:
```json
{
  "name": "Descriptive Workflow Name",
  "nodes": [
    {
      "id": "<uuid>",
      "name": "Unique Node Name",
      "type": "n8n-nodes-base.exactType",
      "typeVersion": "<from get_node>",
      "position": [x, y],
      "parameters": { "<from get_node schema>" },
      "credentials": {}
    }
  ],
  "connections": {
    "Source Node Name": {
      "main": [[{ "node": "Target Node Name", "type": "main", "index": 0 }]]
    }
  },
  "settings": {
    "executionOrder": "v1"
  }
}
```

**Layout rules:**
- Start the first node at position `[250, 300]`
- Space nodes horizontally by 220px: `[470, 300]`, `[690, 300]`, etc.
- For branches, offset vertically by 160px

**Connection rules:**
- Connections use **node names** (not IDs)
- Multiple outputs = multiple arrays in `main`
- If-node: index 0 = true branch, index 1 = false branch
- Switch node: each rule = one output array in `main` (rule 0 → `main[0]`, rule 1 → `main[1]`, etc.)
- Merge node: set `numberInputs` parameter to match the number of incoming branches, then connect each branch to a **different `index`** (0, 1, 2, etc.) — NOT all to index 0
- AI nodes: use `ai_languageModel`, `ai_tool`, `ai_memory`, `ai_outputParser` connection types (not `main`) — see CLAUDE.md "AI Node Connections" section

### 5. Validate Expressions
Before running `validate_workflow`, manually check every expression in the workflow:

**For each expression (`{{ }}` or `$json.*`):**
1. Identify the upstream node that produces the data
2. Confirm the exact field name exists in that node's output schema
3. Check the data path is correct (especially for webhook v2+ where POST body is in `$json.body`)
4. Verify data types match what the expression expects

**Common pitfalls to check for:**
- Webhook v2+ data: use `$json.body.field`, not `$json.field`
- Case sensitivity: `$json.Email` is not the same as `$json.email`
- Null propagation: `$json.user.name` fails silently if `user` is null
- Array vs single item: does upstream return an array or a single object?
- Number as string: `$json.amount > 100` may fail if amount is `"100"`
- Typos: `$json.prce` instead of `$json.price` produces `undefined`, not an error

**Empty data paths — ask for every external data node:**
For each node that fetches external data (HTTP Request, RSS, Google Sheets, Airtable, API calls, etc.), ask: "What happens if this returns 0 items?" If the downstream node would fail or produce wrong results on empty input, add an IF node to check for data first. Common pattern:
```
Fetch Data → IF (items exist) → [true] Process Data
                               → [false] No-Op or notification
```
This prevents "No data" errors and silent failures in production when an API or feed returns nothing.

### 6. Validate Workflow
```
n8n-mcp → validate_workflow({ workflow JSON })
```
Fix any errors before proceeding. Do not skip this step even if you are confident.

### 7. Deploy
Create the workflow on n8n:
```
n8n API → POST /workflows  (body: workflow JSON)
```

### 8. Activate (if requested)
```
n8n API → POST /workflows/{id}/activate
```

### 9. Smoke Test
**NEVER skip this step.** After deploying, trigger a test execution and verify the result:

- **Manual trigger** → Run from the n8n UI or use `n8n_test_workflow`
- **Webhook trigger** → Fire a test HTTP request with realistic sample data
- **Schedule trigger** → Trigger a manual run via the n8n UI

After the test run:
1. Check execution status: `GET /executions?workflowId={id}&limit=1`
2. Get execution details: `GET /executions/{executionId}`
3. Verify the output data is correct — not just that status is "success"
4. Check for `null` values, empty arrays, or unexpected data shapes
5. If the test fails, debug and fix before reporting success

### 10. Report Back
Only after a successful smoke test, show the user:
- Workflow name and ID
- URL: `https://{{N8N_DOMAIN}}/workflow/{id}`
- List of nodes and what each does
- Whether it's active or inactive
- Smoke test result: what was tested and what the output looked like
- Any credentials they need to configure manually in the n8n UI

## Webhook Nodes — Critical API Gotchas

1. **Always include `webhookId` on webhook nodes.** The n8n UI auto-generates a `webhookId` property on each webhook node object. When creating via API, you MUST add it yourself — it's a node-level property (not a parameter). Without it, the webhook registers as "not found" even when the workflow is active.
   ```json
   {
     "id": "uuid-1",
     "name": "My Webhook",
     "type": "n8n-nodes-base.webhook",
     "typeVersion": 2.1,
     "webhookId": "uuid-for-webhook",
     "position": [250, 300],
     "parameters": { "path": "my-path", "httpMethod": "POST" }
   }
   ```

2. **Webhook v2+ wraps POST body.** Data arrives as `$json.body.{field}`, not `$json.{field}`. The webhook node exposes `headers`, `params`, `query`, and `body` as separate top-level properties. Always use `$json.body.` in downstream expressions referencing posted data.

3. **Branching + `responseMode: "lastNode"` is non-deterministic.** When a webhook workflow has branches (e.g., IF node), `lastNode` returns data from whichever branch finishes last. For predictable webhook responses, use a `Respond to Webhook` node instead.

## Credential Handling
- **Never create credentials via API** — credential secrets can't be set programmatically
- If a node needs credentials, note which ones the user must configure in the n8n UI
- Reference existing credentials by ID when the user provides them

## Common Node Types (quick reference)
Always verify with `get_node`, but these are the most common:
- Triggers: `webhook`, `scheduleTrigger`, `manualTrigger`
- HTTP: `httpRequest`
- Code: `code`, `set`
- Logic: `if`, `switch`, `merge`, `splitInBatches`
- Data: `spreadsheetFile`, `convertToFile`
- Apps: `slack`, `gmail`, `googleSheets`, `notion`, `airtable`
