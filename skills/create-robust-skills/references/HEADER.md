# YAML Header Best Practices

The header is critical — it's how users discover your skill.

## Required Format

```yaml
---
name: skill-name-kebab-case
description: Proactively apply when [trigger]. Triggers on [keywords]. Use when [use cases]. Summary sentence.
---
```

## Fields Explained

### name

- **Format**: kebab-case (lowercase with hyphens)
- **Length**: 2-50 characters
- **Content**: Descriptive, searchable

```yaml
# Good
name: react-typescript
name: docker-optimization
name: clean-ddd-hexagonal

# Bad
name: react          # Too generic
name: RT             # Unclear
name: react_and_ts   # Use kebab-case
```

### description

Structure: `[Action] + [Triggers] + [Use Cases] + [Summary]`

```
Proactively apply when [context/condition].
Triggers on [keyword1], [keyword2], [keyword3].
Use when [specific use cases].
[One sentence what it provides].
```

## Examples

### Example 1: Technical Skill

```yaml
---
name: react-typescript
description: Proactively apply when writing React applications with TypeScript. Triggers on react, typescript, react-ts, tsx, component typing, props typing, hook types. Use when creating React components, typing props and state, or migrating JavaScript React to TypeScript. Type-safe React development patterns and best practices.
---
```

### Example 2: Architecture Skill

```yaml
---
name: clean-ddd-hexagonal
description: Proactively apply when designing APIs, microservices, or scalable backend structure. Triggers on DDD, Clean Architecture, Hexagonal, ports and adapters, entities, value objects, domain events, CQRS, event sourcing, repository pattern, use cases, onion architecture. Use when working with domain models, aggregates, repositories, or bounded contexts. Clean Architecture + DDD + Hexagonal patterns for backend services.
---
```

### Example 3: Tool/CLI Skill

```yaml
---
name: docker-debugging
description: Proactively apply when debugging Docker containers and troubleshooting containerized applications. Triggers on docker, container, docker-compose, debug, logs, inspect, network, volume. Use when containers won't start, health checks fail, or network issues occur. Docker debugging commands and troubleshooting workflow.
---
```

## Trigger Selection

### Good Triggers

- **Core technology**: react, docker, python, postgresql
- **Actions**: testing, debugging, deploying, refactoring
- **Concepts**: ddd, cqrs, oop, functional
- **Problems**: bug, error, performance, memory leak

### Bad Triggers

- **Too generic**: code, programming, developer
- **Too specific**: Only edge cases
- **Not searchable**: Internal jargon
- **Too many**: >20 triggers is hard to match

### How Many Triggers?

| Skill Scope | Recommended Triggers |
|-------------|---------------------|
| Single topic | 5-10 |
| Multiple topics | 10-15 |
| Broad domain | 15-20 |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| No triggers in description | Add "Triggers on..." |
| Description too long | Keep under 200 chars |
| Vague "helps with X" | Be specific about when to use |
| Missing use cases | Add "Use when..." section |
| Name not kebab-case | Use lowercase with hyphens |

## Testing Your Header

1. **Read it aloud** — Does it make sense?
2. **Check triggers** — Would a user say these words?
3. **Verify use cases** — Are they specific enough?
4. **Test discovery** — Would this match relevant queries?