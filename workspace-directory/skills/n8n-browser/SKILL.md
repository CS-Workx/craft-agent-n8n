---
name: "n8n Browser"
description: "Use the browser to perform n8n tasks the API cannot handle — credentials, community nodes, OAuth, settings, tagging, and visual debugging"
requiredSources:
  - n8n
icon: "🌐"
---

# n8n Browser Operations

Use the Craft Agents built-in browser to interact with your n8n instance at `https://{{N8N_DOMAIN}}` for tasks the REST API cannot handle.

**Read `~/.craft-agent/docs/browser-tools.md` before your first browser call if you haven't already this session.**

## When to Use This Skill

Use `/n8n-browser` when the user needs to:
- Configure credential secrets (API can't set them reliably)
- Install or uninstall community nodes (no API endpoint)
- Complete OAuth authorization flows (requires browser redirects)
- Tag workflows (REST source can't send array bodies)
- Manage instance settings (SMTP, users, execution settings)
- Test-run a workflow with pin data from the canvas
- Visually inspect execution data flow between nodes

For everything else, prefer the API-based skills (`/n8n-build`, `/n8n-list`, `/n8n-debug`, `/n8n-template`).

## Browser Lifecycle

1. **Open** the browser at the start of the task
2. **Navigate** to the relevant n8n page
3. **Perform** the task using snapshot → find → click/fill patterns
4. **Release** the browser when done so the user can continue in the UI, OR **hide** if more tasks may follow

```
browser_tool open
browser_tool navigate https://{{N8N_DOMAIN}}
```

## Login Handling

n8n may require authentication. If the page shows a login form:

1. `snapshot` to find the email/password fields
2. Ask the user for credentials — NEVER guess or store login credentials
3. `fill` the fields and `click` the sign-in button
4. `wait network-idle 5000` after login

If already authenticated (cookie session active), the dashboard loads directly.

## Task Procedures

### 1. Configure Credential Secrets

The API can create credential records but cannot reliably set secret values (API keys, tokens, passwords). Use the browser instead.

**Navigate to credentials:**
```
browser_tool navigate https://{{N8N_DOMAIN}}/credentials
```

**To edit an existing credential:**
1. `snapshot` → `find` the credential name → `click` it
2. `snapshot` the credential form to see all fields
3. Ask the user for the secret values — never assume or fabricate them
4. `fill` each secret field with the provided values
5. `click` the Save button
6. Confirm the save was successful (look for success toast or absence of errors)

**To create a new credential:**
1. `snapshot` → `find` the "Add Credential" or "+" button → `click`
2. `find` or search for the credential type (e.g., "Slack API", "OpenAI")
3. `click` the correct type
4. `fill` the fields with values provided by the user
5. `click` Save

**Safety:** Never display credential secret values back to the user in your text output. Confirm only that they were entered and saved.

### 2. Install Community Nodes

Community nodes extend n8n with third-party integrations. There is no API for this.

```
browser_tool navigate https://{{N8N_DOMAIN}}/settings/community-nodes
```

**To install a node:**
1. `snapshot` → `find` the "Install" or "Install a community node" button → `click`
2. `fill` the npm package name field (e.g., `n8n-nodes-puppeteer`)
3. Check the risk acknowledgment checkbox if present (`click` it)
4. `click` Install
5. `wait network-idle 10000` — installation can take a moment
6. `snapshot` to verify the node appears in the installed list

**To uninstall a node:**
1. `snapshot` → `find` the node in the list
2. `click` the uninstall/remove button next to it
3. Confirm the action if a dialog appears
4. `wait network-idle 5000`

### 3. Complete OAuth Authorization Flows

Some credentials (Google, Microsoft, Slack, etc.) require an OAuth consent flow in the browser.

```
browser_tool navigate https://{{N8N_DOMAIN}}/credentials
```

1. Find and open the credential (or create a new one of the right type)
2. `snapshot` → `find` the "Sign in with..." or "Connect" button → `click`
3. A new page or popup loads with the OAuth provider's consent screen
4. `snapshot` the consent screen
5. If it asks the user to choose an account or grant permissions, guide them through each step
6. After authorization completes, n8n redirects back and shows a success state
7. `snapshot` to confirm the credential is now connected
8. `click` Save if needed

**Important:** OAuth flows may open in a new tab or redirect. Use `snapshot` after each step to stay oriented. If a popup appears, it should be in the same browser window.

### 4. Tag Workflows

The REST source cannot send the array body required by `PUT /workflows/{id}/tags`. Use the browser.

```
browser_tool navigate https://{{N8N_DOMAIN}}/workflow/{id}
```

1. `snapshot` the workflow canvas page
2. `find` the tags area (usually near the workflow name at the top)
3. `click` the tags area or "Add tag" button
4. `fill` or `find` the desired tag name
5. `click` to select/create the tag
6. Click outside or press Escape to close the tag dropdown
7. The workflow auto-saves tags, but `snapshot` to verify

**To tag multiple workflows:** Repeat for each workflow, or use the workflow list view if bulk-tagging is available.

### 5. Manage Instance Settings

n8n instance settings are not exposed via API.

```
browser_tool navigate https://{{N8N_DOMAIN}}/settings
```

Available settings sections (navigate via sidebar or sub-pages):
- **General** — Instance name, default timezone
- **Users** — `https://{{N8N_DOMAIN}}/settings/users`
- **API** — `https://{{N8N_DOMAIN}}/settings/api`
- **External Secrets** — `https://{{N8N_DOMAIN}}/settings/external-secrets`
- **SMTP / Email** — `https://{{N8N_DOMAIN}}/settings/smtp` (if available in CE)
- **Community Nodes** — `https://{{N8N_DOMAIN}}/settings/community-nodes`
- **Log Streaming** — `https://{{N8N_DOMAIN}}/settings/log-streaming`

For each:
1. Navigate to the specific settings page
2. `snapshot` to see current values
3. `fill` or `select` fields as requested by the user
4. `click` Save
5. `snapshot` to confirm changes took effect

### 6. Test-Run Workflows with Pin Data

Testing a workflow with specific input data from the canvas.

```
browser_tool navigate https://{{N8N_DOMAIN}}/workflow/{id}
```

1. `snapshot` the workflow canvas
2. To pin data on a trigger node: `find` the trigger node → `click` it
3. In the node panel, look for "Pin data" or the pin icon
4. `click` it and enter the test JSON data using `fill` or `paste`
5. `find` the "Test workflow" or "Execute" button at the bottom of the canvas → `click`
6. `wait network-idle 10000` — execution may take time
7. `snapshot` to check execution results

### 7. Visual Execution Inspection

The n8n execution viewer shows data flowing through each node — much richer than the API's JSON dump.

**From the executions list:**
```
browser_tool navigate https://{{N8N_DOMAIN}}/executions
```

**For a specific workflow's executions:**
```
browser_tool navigate https://{{N8N_DOMAIN}}/workflow/{id}/executions
```

1. `snapshot` the execution list
2. `click` the execution you want to inspect
3. The execution viewer opens showing the workflow with data on each node
4. `click` individual nodes to see their input/output data
5. `screenshot` or `screenshot --annotated` to capture the visual state if the user needs a reference
6. Report findings back to the user in text

## General Browser Patterns

### Snapshot-first interaction
Always `snapshot` before interacting. Refs (`@e1`, `@e2`, ...) change after navigation or DOM updates. Re-snapshot after every click that causes navigation or significant UI change.

### Waiting for n8n to load
n8n is a single-page app that can be slow to hydrate:
```
browser_tool wait network-idle 5000
browser_tool snapshot
```

### Handling modals and dialogs
n8n uses modal dialogs for confirmations and forms. After triggering a modal:
1. `snapshot` — the modal elements will appear in the accessibility tree
2. Interact with modal fields using the new refs
3. `click` the confirm/save/cancel button

### Error handling
If an operation fails (red toast, error message):
1. `screenshot` to capture the error visually
2. `snapshot` to read the error text
3. Report the error to the user and suggest next steps

## Safety Rules

1. **Never fabricate credentials or secrets.** Always ask the user for values.
2. **Confirm before destructive actions** — uninstalling nodes, deleting credentials, changing settings.
3. **Never display secret values** in text output after entering them.
4. **Release the browser** when done so the user can continue manually if needed.
5. **Prefer the API** for anything it can handle. Only use the browser for true API gaps.
