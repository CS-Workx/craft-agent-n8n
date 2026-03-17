# GitHub

Full access to GitHub repositories, pull requests, issues, code search, and diffs.

## Scope

Provides comprehensive GitHub integration:
- **Repositories** — List, search, create, and manage repos
- **Issues** — Create, view, update, and search issues
- **Pull Requests** — Create, review, merge PRs; view diffs and comments
- **Code Search** — Search across repositories by code content
- **Branches** — Create, list, and manage branches
- **Commits** — View commit history and diffs

## Guidelines

- Use the GitHub PAT scopes: `repo`, `read:org`, `read:packages`
- When searching code, be specific with queries to reduce noise
- For PR reviews, always read the full diff before commenting
- Prefer searching by owner/repo when you know the specific repository
- Rate limits apply — avoid bulk operations in tight loops

## Common Patterns

### Find open issues
```
list_issues({ owner: "your-org", repo: "your-repo", state: "open" })
```

### Search code across repos
```
search_code({ query: "useEffect cleanup memory leak language:typescript" })
```

### View a pull request
```
get_pull_request({ owner: "your-org", repo: "your-repo", pullNumber: 42 })
```

### Create an issue
```
create_issue({ owner: "your-org", repo: "your-repo", title: "Bug: ...", body: "..." })
```
