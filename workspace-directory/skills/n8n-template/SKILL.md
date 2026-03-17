---
name: "n8n Template"
description: "Search and deploy pre-built n8n workflow templates"
requiredSources:
  - n8n-mcp
  - n8n
---

# Deploy an n8n Template

Search 2,700+ pre-built workflow templates and deploy them to your n8n instance at `{{N8N_DOMAIN}}`.

## Process

### 1. Search Templates
Use the user's description to find matching templates:
```
n8n-mcp → search_templates("send slack notification on new github issue")
```

### 2. Present Options
Show the user a summary of matching templates:
- Template name and description
- Nodes used
- Complexity (number of nodes)

### 3. Get Full Template
Once the user picks one:
```
n8n-mcp → get_template(templateId)
```

### 4. Customize
Help the user adapt the template:
- **Credentials**: Identify which credentials are needed and note them
- **Parameters**: Adjust URLs, channels, filters, etc. to match user's needs
- **Nodes**: Add, remove, or modify nodes if needed

### 5. Validate
Templates can be outdated or break after customization. Always validate:
```
n8n-mcp → validate_workflow({ customized template JSON })
```
Also manually check expressions against the expression pitfalls in CLAUDE.md — especially if you modified any data paths or field references during customization.

### 6. Deploy
```
n8n API → POST /workflows  (body: customized template JSON)
```

### 7. Smoke Test
**NEVER skip this step.** After deploying, trigger a test execution:

- **Manual trigger** → Run from the n8n UI or use `n8n_test_workflow`
- **Webhook trigger** → Fire a test HTTP request with realistic sample data
- **Schedule trigger** → Trigger a manual run via the n8n UI

After the test run:
1. Check execution status: `GET /executions?workflowId={id}&limit=1`
2. Get execution details: `GET /executions/{executionId}`
3. Verify output data is correct — not just that status is "success"
4. If the test fails, debug and fix before reporting success

### 8. Report Back
Only after a successful smoke test, show the user:
- Workflow name and ID
- URL: `https://{{N8N_DOMAIN}}/workflow/{id}`
- Template used (name and ID)
- Smoke test result: what was tested and what the output looked like
- List of credentials they need to configure in the n8n UI
- Any parameters they should review

## Tips
- Templates often include placeholder credentials — always flag these
- Some templates use community nodes that may not be installed
- If no template matches exactly, suggest combining elements from multiple templates or use `/n8n-build` instead
- Even trusted templates need validation after customization — don't skip step 5
