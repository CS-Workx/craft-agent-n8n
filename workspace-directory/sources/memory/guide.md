# Memory (Knowledge Graph)

Persistent memory across sessions using a local knowledge graph. Stores entities, relations, and observations about projects, decisions, and patterns.

## Scope

Provides tools for creating and querying a local knowledge graph:
- `create_entities` — Add new nodes (projects, people, technologies, decisions)
- `create_relations` — Link entities together (uses, depends-on, decided-by)
- `add_observations` — Attach facts to existing entities
- `search_nodes` — Find entities by name or content
- `open_nodes` — Retrieve specific entities by name
- `delete_entities` / `delete_relations` / `delete_observations` — Remove data

## Guidelines

- Store important project context that should persist across sessions
- Use clear, descriptive entity names (e.g., "auth-service" not "service1")
- Create relations to link related concepts (e.g., "auth-service" --uses--> "JWT")
- Search before creating to avoid duplicates
- Use observations for facts, preferences, and decisions
- The knowledge graph is stored locally — no data leaves your machine

## Common Patterns

### Remembering project architecture
```
create_entities: [{ name: "backend-api", entityType: "service", observations: ["Built with FastAPI", "Uses PostgreSQL", "Deployed on Railway"] }]
```

### Linking related concepts
```
create_relations: [{ from: "backend-api", to: "postgres-db", relationType: "uses" }]
```

### Recalling context
```
search_nodes: { query: "backend" }
```
