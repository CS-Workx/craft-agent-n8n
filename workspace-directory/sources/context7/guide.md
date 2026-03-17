# Context7

Fetch up-to-date, version-specific documentation for any library — directly into the conversation context. Eliminates hallucinated APIs and outdated code generation.

## Scope

Provides two tools:
- `resolve-library-id` — Resolve a library name (e.g., "react", "fastapi") into a Context7-compatible ID
- `get-library-docs` — Fetch documentation and code examples for a resolved library

## Guidelines

- Always resolve the library ID first before fetching docs
- Use when you need accurate, current API references
- Particularly useful when working with libraries that update frequently
- Combine with code generation to ensure generated code uses current APIs
- If rate-limited, consider adding a Context7 API key (free tier available at context7.com/dashboard)

## Common Patterns

### Look up a library's API
```
1. resolve-library-id({ libraryName: "next.js" })
2. get-library-docs({ libraryId: "/vercel/next.js", topic: "app router" })
```

### Verify an API before using it
```
1. resolve-library-id({ libraryName: "prisma" })
2. get-library-docs({ libraryId: "/prisma/prisma", topic: "transactions" })
```

## Tips

- Be specific with topic queries for more relevant results
- Works with virtually any open-source library
- Docs are fetched from source — always current
