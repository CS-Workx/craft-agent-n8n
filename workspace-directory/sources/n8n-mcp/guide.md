# n8n MCP — Node Documentation & Templates

Local MCP server (via npx) providing access to all 1,084 n8n node types, 2,700+ workflow templates, and workflow validation. Connected to your n8n instance at `{{N8N_DOMAIN}}`.

## Scope

- **Node discovery**: Search and look up any n8n node type with exact parameter schemas
- **Templates**: Search 2,700+ pre-built workflow templates
- **Validation**: Validate node configurations and complete workflows before deployment
- **Workflow management**: Create, update (diff-based), test, and manage workflows on the connected instance

## Guidelines

- **Always look up nodes before building** — Use `search_nodes` + `get_node` to get exact type strings, parameter schemas, and typeVersions. Never guess node types.
- **Validate before deploying** — Run `validate_workflow` on the complete JSON before pushing to the API.
- **Use templates as starting points** — Search templates first; customizing a template is faster than building from scratch.
- **Diff-based updates** — Use `n8n_update_partial_workflow` for modifications; it saves 80-90% on tokens vs full replacement.

## Tools Reference

### Discovery (no API key required)

| Tool | Purpose | When to use |
|------|---------|-------------|
| `search_nodes` | Find nodes by keyword | First step when building — find the right node type |
| `get_node` | Get full node details (params, typeVersion) | After finding a node, get its exact schema |
| `search_templates` | Search 2,700+ workflow templates | When user describes a use case, find matching templates |
| `get_template` | Get full template configuration | After finding a template, get the complete workflow JSON |
| `validate_node` | Check a node configuration | Verify node params before adding to workflow |
| `validate_workflow` | Validate complete workflow JSON | Final check before deploying |
| `tools_documentation` | Access tool specifications | Reference for MCP tool usage |

### Workflow Management (requires API key)

| Tool | Purpose | When to use |
|------|---------|-------------|
| `n8n_create_workflow` | Create a new workflow | Deploy a built workflow to n8n |
| `n8n_get_workflow` | Retrieve a workflow | Inspect an existing workflow |
| `n8n_update_partial_workflow` | Diff-based update | Modify specific nodes/connections without sending full JSON |
| `n8n_delete_workflow` | Delete a workflow | Remove a workflow |
| `n8n_test_workflow` | Test a workflow | Run a test execution |
| `n8n_list_workflows` | List all workflows | Overview of all workflows |
| `n8n_activate_workflow` | Activate a workflow | Enable triggers |
| `n8n_deactivate_workflow` | Deactivate a workflow | Disable triggers |

## Common Patterns

### Building a new workflow
```
1. search_nodes("webhook") → find trigger node
2. get_node("n8n-nodes-base.webhook") → get params + typeVersion
3. search_nodes("slack") → find action node
4. get_node("n8n-nodes-base.slack") → get params
5. validate_workflow({...}) → check the assembled JSON
6. n8n_create_workflow({...}) → deploy
```

### Finding a template
```
1. search_templates("send slack message on webhook") → browse results
2. get_template(id) → get full workflow JSON
3. Customize credentials and parameters
4. Deploy via n8n API or n8n_create_workflow
```
