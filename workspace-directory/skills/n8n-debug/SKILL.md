---
name: "n8n Debug"
description: "Debug failing n8n workflows using execution data and validation"
requiredSources:
  - n8n-mcp
  - n8n
---

# Debug an n8n Workflow

Diagnose and fix failing workflows on your n8n instance at `{{N8N_DOMAIN}}`.

## Process

### 1. Identify the Problem

**If the user names a specific workflow:**
```
n8n API → GET /workflows/{id}
n8n API → GET /executions?workflowId={id}&status=error&limit=5
```

**If the user doesn't specify:**
```
n8n API → GET /executions?status=error&limit=10
```
Show recent failures and ask which one to investigate.

### 2. Analyze the Execution

**Watch for silent failures:** Not all bugs cause errors. A typo like `product.prce` instead of `product.price` produces `undefined` → `NaN` → `null` without crashing. The workflow shows `status: "success"` but outputs wrong data. Always compare actual output values against expected values, not just execution status.
```
n8n API → GET /executions/{executionId}
```

Look at:
- **Which node failed** — the `stoppedAt` node in execution data
- **Error message** — often in `data.resultData.error`
- **Input data** — what data reached the failing node
- **Previous node outputs** — trace the data flow

### 3. Validate the Workflow
```
n8n-mcp → validate_workflow(workflow JSON)
```
This catches configuration errors like:
- Wrong parameter types
- Missing required fields
- Invalid node connections
- Deprecated typeVersions

### 4. Diagnose Common Issues

#### Explicit errors (execution status = error)

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| "No data" error | Previous node returned empty | Add IF node to check for data |
| Authentication error | Expired/wrong credentials | User must reconfigure in n8n UI |
| Connection timeout | External service down or slow | Increase timeout or add retry |
| Expression error | Wrong expression syntax | Check `{{ }}` vs `$json.` notation |
| "Unknown node type" | Community node not installed | Install the node package |
| Webhook 404 "not registered" | Missing `webhookId` on webhook node | Add `webhookId` (UUID) as a node-level property (not in parameters) |

#### Silent failures (execution status = success, but output is wrong)

These are the hardest bugs to catch — the workflow "succeeds" but produces incorrect data. Never trust execution status alone. Always inspect actual output values.

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| All values are `null` or `undefined` | Typo in field name (e.g., `$json.prce` instead of `$json.price`) | Compare expression field names against actual upstream output fields character by character |
| All items go to wrong IF branch | Null/NaN values from upstream typo cause comparison to fail | Trace data flow — check for `null` or missing fields in previous node output |
| Webhook expressions return `undefined` | Wrong data path — `$json.field` vs `$json.body.field` | Webhook v2+ wraps POST data in `$json.body`. Check execution data to see actual structure |
| Wrong webhook response returned | `responseMode: "lastNode"` with branching | n8n returns data from the last-finishing branch, not the "main" one. Use `Respond to Webhook` node instead |
| Numbers treated as strings | Upstream returns `"100"` not `100` | Use `Number()` or `+` to coerce, or use `{{}}` expressions with type conversion |
| Empty array instead of items | Upstream returns nested array | Check if data needs `.items`, `.data`, or similar unwrapping |

### 5. Trace Data Flow (Expression Debugging)

When you suspect a silent failure, trace the data flow node by node:

1. **Get the execution details**: `GET /executions/{id}`
2. **For each node in the chain**, extract the output data from the execution
3. **Compare actual field names** against what downstream expressions expect
4. **Check for:**
   - Fields that exist but are `null` (upstream node ran but produced no value)
   - Fields that don't exist at all (typo or wrong data path)
   - Fields with unexpected types (string instead of number, array instead of object)
   - Fields with unexpected nesting (data wrapped in an extra object layer)

This is the most reliable way to find silent failures — follow the data, not the status.

### 6. Suggest and Apply Fixes
- Explain the root cause clearly
- Propose a specific fix
- If user approves, apply via `PUT /workflows/{id}` or `n8n_update_partial_workflow`
- For credential issues, guide the user to fix in the n8n UI

### 7. Verify
After fixing, suggest the user trigger a test execution:
- Manual trigger: run from the n8n UI
- Webhook trigger: provide the test URL
- Schedule trigger: wait for next run or trigger manually
