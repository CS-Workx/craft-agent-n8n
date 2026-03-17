# Perplexity Search

Real-time web search powered by Perplexity's Sonar models. Returns cited, up-to-date results for technical queries, documentation, and troubleshooting.

## Scope

Provides search tools for querying the web in real-time:
- `perplexity_web_search` — Search the entire web for information
- `perplexity_local_search` — Search for local/regional information

## Guidelines

- Use for up-to-date information beyond the model's training data
- Great for troubleshooting error messages and finding solutions
- Returns results with citations — always verify critical information
- Prefer this over WebSearch when you need more comprehensive, AI-synthesized answers
- Use specific, detailed queries for better results

## Common Patterns

### Troubleshoot an error
```
perplexity_web_search({ query: "Next.js 15 hydration mismatch error with server components" })
```

### Find current best practices
```
perplexity_web_search({ query: "PostgreSQL connection pooling best practices 2026" })
```

### Research a library
```
perplexity_web_search({ query: "drizzle vs prisma ORM comparison performance 2026" })
```
