# Craft Agent — n8n Workspace

A complete [Craft Agent](https://craft.do/craft-agent) workspace for building, debugging, and managing workflows on your [n8n](https://n8n.io) instance.

## What's Inside

### 7 Sources (data connections)

| Source | Type | Purpose | Auth needed? |
|--------|------|---------|-------------|
| **n8n** | REST API | Create, update, delete workflows and executions | Yes — n8n API key |
| **n8n-mcp** | MCP (stdio) | Node docs, 2,700+ templates, workflow validation | Yes — n8n API key |
| **Context7** | MCP (stdio) | Up-to-date library documentation | No |
| **Sequential Thinking** | MCP (stdio) | Step-by-step reasoning for complex problems | No |
| **Memory** | MCP (stdio) | Persistent knowledge graph across sessions | No |
| **Perplexity** | MCP (stdio) | Real-time web search with citations | Optional — Perplexity API key |
| **GitHub** | MCP (remote) | Repo, PR, issue, and code search access | Optional — GitHub PAT |

### 5 Skills (reusable workflows)

| Skill | Trigger | What it does |
|-------|---------|-------------|
| `/n8n-build` | Build a new workflow | Full guided process: requirements → template search → node lookup → validate → deploy → test |
| `/n8n-template` | Deploy from template | Search 2,700+ templates, customize, validate, deploy, and test |
| `/n8n-debug` | Fix a failing workflow | Analyze executions, trace data flow, catch silent failures, apply fixes |
| `/n8n-list` | Overview & inspect | List workflows, executions, credentials with interactive tables and diagrams |
| `/n8n-browser` | Browser-based tasks | Credentials, community nodes, OAuth, settings, tagging — things the API can't do |

### CLAUDE.md (working directory instructions)

277 lines of battle-tested n8n instructions covering workflow JSON structure, API gotchas, expression pitfalls, AI node connections, and a build quality checklist.

---

## Prerequisites

- **Craft Agent** installed on your machine
- **An n8n instance** (self-hosted or cloud) with API access enabled
- **n8n API key** — generate one in your n8n instance: Settings → API → Create API Key
- **Node.js 18+** — required for the MCP sources that run via `npx`

Optional:
- **Perplexity API key** — for real-time web search ([get one here](https://www.perplexity.ai/settings/api))
- **GitHub PAT** — for repo/PR/issue access ([create one here](https://github.com/settings/tokens))

---

## Setup (3 Steps)

### Step 1 — Clone this repo

```bash
git clone https://github.com/CS-Workx/craft-agent-n8n.git
cd craft-agent-n8n
```

### Step 2 — Copy the workspace into Craft Agent

Copy the contents of `workspace-directory/` into your Craft Agent workspaces folder:

```bash
cp -r workspace-directory/ ~/.craft-agent/workspaces/n8n/
```

> **Note:** If you already have a workspace named `n8n`, either back it up first or choose a different folder name.

### Step 3 — Replace the placeholders

There are **two placeholders** to replace with your own values:

| Placeholder | Replace with | Example |
|-------------|-------------|--------|
| `{{N8N_DOMAIN}}` | Your n8n instance domain (no `https://`) | `n8n.example.com` |
| `{{N8N_API_KEY}}` | Your n8n API key | `eyJhbGci...` |

**Quick find-and-replace across all files:**

```bash
# macOS / Linux
cd ~/.craft-agent/workspaces/n8n

# Replace the domain placeholder
find . -type f \( -name '*.json' -o -name '*.md' \) -not -path '*/sessions/*' \
  -exec sed -i '' 's/{{N8N_DOMAIN}}/n8n.example.com/g' {} +

# Replace the API key placeholder
find . -type f \( -name '*.json' -o -name '*.md' \) -not -path '*/sessions/*' \
  -exec sed -i '' 's/{{N8N_API_KEY}}/YOUR_ACTUAL_API_KEY/g' {} +
```

> **On Linux**, use `sed -i` (without the `''`) instead of `sed -i ''`.

---

## Set Up Your Working Directory

The `working-directory/` folder contains a `CLAUDE.md` file with comprehensive n8n instructions. Copy it to your n8n project folder:

```bash
# Create a project folder (or use an existing one)
mkdir -p ~/n8n-projects
cp working-directory/CLAUDE.md ~/n8n-projects/

# Replace the placeholder in CLAUDE.md too
cd ~/n8n-projects
sed -i '' 's/{{N8N_DOMAIN}}/n8n.example.com/g' CLAUDE.md
```

Then update the workspace config to point to your project folder:

```bash
# Edit ~/.craft-agent/workspaces/n8n/config.json
# Change the "workingDirectory" value to your project folder path
```

---

## Authenticate Sources

After copying the workspace, open Craft Agent and navigate to the n8n workspace. Some sources need authentication:

### Required: n8n API source
The n8n REST API source authenticates via the `X-N8N-API-KEY` header. If you ran the sed commands above, it should be pre-configured. Open the n8n source in Craft Agent to verify the connection.

### Required: n8n-mcp source
The API key is set in the `env` block of `sources/n8n-mcp/config.json`. If you ran the sed commands above, this is already configured.

### Optional: Perplexity
Edit `sources/perplexity/config.json` and add your API key in the `env.PERPLEXITY_API_KEY` field.

### Optional: GitHub
Open the GitHub source in Craft Agent and enter your PAT when prompted.

---

## Verify Everything Works

Open a new session in the n8n workspace and try:

1. **List workflows:** Type `/n8n-list` to see your workflows
2. **Search templates:** Ask "Find me a template for sending Slack notifications on new GitHub issues"
3. **Build a workflow:** Type `/n8n-build` and describe what you want

If you get authentication errors, double-check that the `{{N8N_DOMAIN}}` and `{{N8N_API_KEY}}` placeholders were properly replaced.

---

## Customization

### Labels
The workspace includes labels for task type (Build, Debug, Template, Optimize), node category (Trigger, Integration, AI, Logic, Code), environment (Production, Development), priority, and workflow tracking. The workflow ID and n8n Link labels auto-extract from URLs in your conversations.

### Statuses
Workflow statuses: Backlog → Todo → In Progress → Testing → Needs Review → Done / Cancelled.

### Automations
- **Urgent Triage**: Adding the "urgent" label triggers an automatic assessment of the session
- **Testing Checklist**: Moving a session to "Testing" status triggers validation and execution checks

---

## File Structure

```
craft-agent-n8n/
├── README.md                           ← You are here
├── CHANGELOG.md                        ← Version history
├── .gitignore
├── workspace-directory/                ← Copy to ~/.craft-agent/workspaces/n8n/
│   ├── config.json                     ← Workspace settings
│   ├── icon.svg                        ← n8n workspace icon
│   ├── views.json                      ← Session views (New, Plan, Explore, Processing)
│   ├── automations.json                ← Automated triggers
│   ├── labels/config.json              ← Label definitions
│   ├── statuses/config.json + icons/   ← Workflow statuses with icons
│   ├── sources/                        ← 7 data source configurations
│   └── skills/                         ← 5 reusable skill definitions
└── working-directory/                  ← Copy CLAUDE.md to your n8n project folder
    └── CLAUDE.md                       ← Comprehensive n8n instructions
```

---

## Credits

Built by [CS Workx](https://github.com/CS-Workx) for use in AI Agents workshops and training programs.

Powered by [Craft Agent](https://craft.do/craft-agent) and [n8n](https://n8n.io).
