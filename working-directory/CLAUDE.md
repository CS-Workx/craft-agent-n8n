# n8n Workflow Automation

Your self-hosted n8n Community Edition instance at `{{N8N_DOMAIN}}`.

## Two Sources, One Goal

| Source | Type | Purpose | Tool prefix |
|--------|------|---------|-------------|
| **n8n** | REST API | CRUD operations — create, read, update, delete workflows, executions, credentials | `mcp__n8n__api_n8n` |
| **n8n-mcp** | MCP (stdio) | Knowledge — node docs, templates, validation (1,084 nodes, 2,700+ templates) | `mcp__n8n-mcp__*` |

**Rule of thumb:** Use **n8n-mcp** to figure out *what* to build. Use the **n8n API** to *build* it.

## Critical Rules

1. **ALWAYS search templates first.** Before building from scratch, search for a matching template (`n8n-mcp → search_templates`). Templates are battle-tested by real users. Only build from scratch if no suitable template exists.
2. **NEVER guess node types or parameters.** Always look up via `n8n-mcp → search_nodes` + `get_node` first. Document which lookups you made.
3. **ALWAYS validate before deploying.** Run `n8n-mcp → validate_workflow` on the complete JSON.
4. **ALWAYS smoke test after deploying.** Trigger a test execution and verify the output before reporting success.
5. **NEVER create credentials via API.** Credential secrets can't be set programmatically. Tell the user to configure them in the n8n UI at `https://{{N8N_DOMAIN}}`.
6. **Confirm before destructive actions.** Always ask before deleting or deactivating workflows.
7. **Fetch before updating.** When modifying a workflow, `GET /workflows/{id}` first to avoid overwriting changes.

## Skills

| Skill | When to use |
|-------|-------------|
| `/n8n-build` | Build a new workflow from scratch |
| `/n8n-template` | Search and deploy from 2,700+ pre-built templates |
| `/n8n-debug` | Diagnose and fix failing workflows |
| `/n8n-list` | List and inspect workflows, executions, credentials |
| `/n8n-browser` | Browser-based tasks the API can't handle — credentials, community nodes, OAuth, settings, tagging |

## Building a Workflow (Step by Step)

```
1. Understand requirements (trigger, actions, logic)
2. n8n-mcp → search_templates("description")        # Search templates FIRST
3. If template found → customize it, skip to step 7
4. n8n-mcp → search_nodes("webhook")                # Find the right node
5. n8n-mcp → get_node("n8n-nodes-base.webhook")     # Get exact params + typeVersion
6. Repeat steps 4-5 for EVERY node — document each lookup
7. Validate expressions against upstream data shapes (see pitfalls below)
8. n8n-mcp → validate_workflow(json)                 # Catch errors before deploy
9. n8n API → POST /workflows                         # Create (inactive by default)
10. n8n API → POST /workflows/{id}/activate           # Activate if requested
11. Trigger a test execution and verify output        # NEVER skip this step
```

## Workflow JSON Structure

```json
{
  "name": "Descriptive Workflow Name",
  "nodes": [
    {
      "id": "uuid-string",
      "name": "Unique Node Name",
      "type": "n8n-nodes-base.exactType",
      "typeVersion": 2,
      "position": [250, 300],
      "parameters": {},
      "credentials": {
        "credentialType": { "id": "123", "name": "My Credential" }
      }
    }
  ],
  "connections": {
    "Source Node Name": {
      "main": [[{ "node": "Target Node Name", "type": "main", "index": 0 }]]
    }
  },
  "settings": {
    "executionOrder": "v1",
    "timezone": "Europe/Brussels"
  }
}
```

### Node layout
- First node: `[250, 300]`
- Horizontal spacing: 220px → `[470, 300]`, `[690, 300]`, etc.
- Branch offset: 160px vertically

### Connections
- Reference nodes by **name** (not ID)
- `main` array index = source output index
- `index` field = target input index
- IF node: output 0 = true, output 1 = false

### Multi-Output Nodes (Switch)
Switch node creates one output per rule. Each rule's output is a separate array in `main`:
```json
"Route by Type": {
  "main": [
    [{ "node": "Handle Order", "type": "main", "index": 0 }],
    [{ "node": "Handle Refund", "type": "main", "index": 0 }],
    [{ "node": "Handle Inquiry", "type": "main", "index": 0 }]
  ]
}
```

### Multi-Input Nodes (Merge)
Merge node supports 2-10 inputs via `numberInputs` parameter. Each source branch must connect to a **different input index** on the Merge node:
```json
"Handle Order":  { "main": [[{ "node": "Merge Results", "type": "main", "index": 0 }]] },
"Handle Refund": { "main": [[{ "node": "Merge Results", "type": "main", "index": 1 }]] },
"Handle Inquiry":{ "main": [[{ "node": "Merge Results", "type": "main", "index": 2 }]] }
```
**Common mistake:** Connecting all branches to `index: 0` — this sends everything to input 1 and leaves other inputs empty. The Merge node will hang waiting for data on its other inputs, or produce incorrect results.

### AI Node Connections
AI nodes (Agent, Chain) use special connection types — NOT `main`:

| Connection type | Purpose | Example source node |
|----------------|---------|-------------------|
| `ai_languageModel` | LLM model | `lmChatOpenAi`, `lmChatAnthropic`, `lmChatGoogleGemini` |
| `ai_tool` | Tool for agent | Any node connected as tool (Slack, HTTP Request, Code, etc.) |
| `ai_outputParser` | Output format | `outputParserStructured`, `outputParserAutofixing` |
| `ai_memory` | Conversation memory | `memoryBufferWindow`, `memoryPostgresChat` |

```json
"OpenAI Model": {
  "main": [],
  "ai_languageModel": [[{ "node": "AI Agent", "type": "ai_languageModel", "index": 0 }]]
},
"Slack Tool": {
  "main": [],
  "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]]
}
```
**API requirement:** Every AI sub-node connection entry MUST include an empty `"main": []` alongside the AI connection type. Without it, the n8n API rejects the workflow with "Invalid connections." The `main` connection into the AI Agent carries the trigger/input data. The AI sub-connections carry the model, tools, memory, and parser.

## n8n-mcp Tools

### Discovery (look up before you build)
- `search_nodes(query)` — Find nodes by keyword
- `get_node(nodeType)` — Get full parameter schema + typeVersion
- `search_templates(query)` — Search 2,700+ workflow templates
- `get_template(id)` — Get complete template JSON
- `validate_node(config)` — Check a single node config
- `validate_workflow(json)` — Validate a complete workflow

### Workflow Management
- `n8n_create_workflow` — Deploy a workflow
- `n8n_get_workflow` — Retrieve a workflow
- `n8n_update_partial_workflow` — Diff-based update (80-90% token savings vs full PUT)
- `n8n_delete_workflow` — Delete a workflow
- `n8n_test_workflow` — Run a test execution
- `n8n_list_workflows` — List all workflows
- `n8n_activate_workflow` / `n8n_deactivate_workflow` — Toggle active state

## n8n API Endpoints

### Workflows
- `GET /workflows` — List all (supports `?limit=N&cursor=...&tags=...&active=true|false`)
- `GET /workflows/{id}` — Get single workflow with nodes and connections
- `POST /workflows` — Create new (always created inactive)
- `PUT /workflows/{id}` — Update (full replacement, no partial updates)
- `DELETE /workflows/{id}` — Delete
- `POST /workflows/{id}/activate` — Activate
- `POST /workflows/{id}/deactivate` — Deactivate

### Executions
- `GET /executions` — List (supports `?workflowId=...&status=...&limit=N&cursor=...`)
- `GET /executions/{id}` — Get details with input/output data
- `DELETE /executions/{id}` — Delete

### Credentials
- `GET /credentials` — List (names and types only, secrets never exposed)
- `POST /credentials` — Create (secrets unreliable via API — use n8n UI)

### Tags & Variables
- `GET /tags` / `POST /tags` — List and create tags
- `GET /variables` / `POST /variables` — List and create variables

## Common Node Types

Always verify with `get_node`, but these are the most common:

| Category | Nodes |
|----------|-------|
| Triggers | `webhook`, `scheduleTrigger`, `manualTrigger`, `formTrigger` |
| HTTP | `httpRequest` |
| Code | `code`, `set` |
| Logic | `if`, `switch`, `merge`, `splitInBatches`, `aggregate` |
| Data | `spreadsheetFile`, `convertToFile`, `html` |
| Apps | `slack`, `gmail`, `googleSheets`, `notion`, `airtable` |
| AI | `@n8n/n8n-nodes-langchain.agent`, `@n8n/n8n-nodes-langchain.chainLlm` |

## API Gotchas (learned from testing)

### Webhook nodes require `webhookId`
When creating workflows via API, webhook nodes MUST have a `webhookId` property at the node level (not inside `parameters`). The n8n UI auto-generates this, but the API does not. Without it, the webhook returns 404 "not registered" even when the workflow is active.
```json
{
  "id": "node-uuid",
  "name": "My Webhook",
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 2.1,
  "webhookId": "webhook-uuid",
  "position": [250, 300],
  "parameters": { "path": "my-path", "httpMethod": "POST" }
}
```

### Webhook v2+ data structure
POST data is wrapped inside `$json.body`, not available directly as `$json.{field}`. The webhook exposes: `$json.headers`, `$json.params`, `$json.query`, `$json.body`.

### Branching + `responseMode: "lastNode"`
Returns data from whichever branch finishes last — non-deterministic with IF/Switch nodes. Use a `Respond to Webhook` node for predictable responses.

### AI sub-node connections require `main: []`
When creating workflows via API, every AI sub-node (model, tool, memory, parser) connection entry must include `"main": []` alongside the AI connection type. Example: `"Claude Model": {"main": [], "ai_languageModel": [[...]]}`. Without the empty `main` array, the API returns "Invalid connections" even though `validate_workflow` passes.

### Tags via API
The `PUT /workflows/{id}/tags` endpoint requires an array body `[{"id": "..."}]` — the MCP REST source can't send array bodies (only objects). Use curl or the n8n UI for tagging.

### n8n CE limitations
Community Edition does not support project folders. Use tags or naming conventions (e.g., `Test L1 —`, `Prod —`) to organize workflows.

## Debugging Workflows

1. Find failures: `GET /executions?status=error&limit=10`
2. Get details: `GET /executions/{id}` — check `stoppedAt` node and `data.resultData.error`
3. Validate: `n8n-mcp → validate_workflow(json)`
4. Common issues:

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| "No data" error | Previous node returned empty | Add IF node to check for data |
| Authentication error | Expired/wrong credentials | User reconfigures in n8n UI |
| Connection timeout | External service slow | Increase timeout or add retry |
| Expression error | Wrong syntax | Check `{{ }}` vs `$json.` notation |
| "Unknown node type" | Community node not installed | Install the node package |
| Webhook 404 "not registered" | Missing `webhookId` on node | Add `webhookId` (UUID) as node-level property |
| All values `null` / wrong branch | Typo in field name (silent failure) | Trace data flow in execution — compare actual vs expected values |
| `$json.field` returns undefined | Webhook v2+ wraps POST body | Use `$json.body.field` instead |
| "Invalid connections" on AI workflow | AI sub-nodes missing `main: []` | Add `"main": []` to every AI sub-node connection entry |

## Build Quality Checklist

Before reporting a workflow as done, verify ALL of these:

- [ ] Searched templates first (documented result: used template #X or no match found)
- [ ] Called `get_node` for every node type used (documented typeVersions)
- [ ] Validated all expressions against upstream data shapes
- [ ] Verified credentials exist: `GET /credentials` and cross-referenced against every credential ID in the workflow
- [ ] Checked empty-data paths: for each node that fetches external data, considered what happens if it returns 0 items
- [ ] Ran `validate_workflow` — no errors
- [ ] Deployed and triggered a test execution
- [ ] Verified execution output matches expectations (not just "status: success")

## Expression Pitfalls

Common sources of silent failures — expressions that don't crash but produce wrong data:

| Pitfall | Wrong | Correct |
|---------|-------|---------|
| Webhook v2+ body wrapping | `$json.field` | `$json.body.field` |
| Case sensitivity | `$json.Email` when field is `email` | Match exact casing from upstream output |
| Null propagation | `$json.user.name` when `user` is null | Use `$json.user?.name` or add IF node |
| Array vs single item | `$json.items` on a single item | Check if upstream returns array or object |
| Number as string | `$json.amount > 100` when amount is `"100"` | Use `Number($json.amount)` or `+$json.amount` |
| Missing field (no error) | `$json.prce` (typo for `price`) | Always cross-reference field names with actual upstream output |

**Rule of thumb:** When writing expressions, always trace the data path from the source node. If unsure about the shape, check the execution data of the upstream node — never assume.

## After Creating a Workflow

Always report back to the user with:
- Workflow name and ID
- Direct link: `https://{{N8N_DOMAIN}}/workflow/{id}`
- List of nodes and what each does
- Active/inactive status
- Any credentials that need manual configuration in the n8n UI
