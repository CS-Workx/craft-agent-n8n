# Sequential Thinking

Structured, multi-step reasoning engine for breaking down complex problems.

## Scope

Provides a `sequentialthinking` tool that enables step-by-step reasoning with the ability to revise and branch thinking paths. Useful for architecture decisions, debugging strategies, and implementation planning.

## Guidelines

- Use for problems that benefit from explicit step-by-step reasoning
- Each thought can revise or branch from previous thoughts
- Set `nextThoughtNeeded: false` on the final thought to complete the chain
- Particularly useful before writing complex code — think first, code second
- Works well in combination with `/debug` and `/scaffold` skills

## When to Use

- Breaking down multi-step implementation tasks
- Analyzing trade-offs between architectural approaches
- Debugging complex issues systematically
- Planning refactoring strategies
- Evaluating library/framework choices
