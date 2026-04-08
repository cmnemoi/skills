# Example: Creating a Docker Skill

## Scenario

You want to create a skill for "Docker optimization" (more focused than "Docker general").

## Step-by-Step

### Step 1: Scope the Skill

```
Docker
├─ Basics (run, build)         → Too basic, skip
├─ Optimization (size, speed)   → This is the skill
├─ Compose                      → Separate skill?
├─ Security                     → Separate skill?
```

**Decision**: Focused skill on optimization only.

### Step 2: Write Header

```yaml
---
name: docker-optimization
description: Proactively apply when optimizing Docker images and containers. Triggers on docker, Dockerfile, image-size, build-time, multi-stage, layers, dockerignore, slim-image. Use when creating Dockerfiles, reducing image size, speeding up builds, or optimizing for production. Docker optimization techniques and best practices.
---
```

### Step 3: Identify Key Decision Trees

1. "Which base image?"
2. "How to reduce size?"
3. "How to speed up builds?"

### Step 4: Draft Content

```markdown
## Decision Tree: "Which base image?"

What do you need?
├─ Minimal size        → alpine, scratch
├─ Compatibility       → debian, ubuntu
├─ Official lib        → [library/name]
└─ Build tools included → builder pattern
```

### Step 5: Add ❌/✅ Examples

```dockerfile
# ❌ Multiple RUN commands (creates layers)
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# ✅ Single RUN (fewer layers)
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

### Step 6: Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| No .dockerignore | Larger context sent | Add .dockerignore |
| Multiple RUN | More layers | Combine commands |
| Latest tag | Unpredictable | Pin versions |
| No multi-stage | Large final image | Use builder pattern |

### Step 7: Add References

```
docker-optimization/
├── SKILL.md
└── references/
    ├── MULTI-STAGE.md   # Detailed multi-stage examples
    ├── BUILD-CACHE.md   # Cache optimization
    └── CHEATSHEET.md    # Quick commands
```

## Result

```
docker-optimization/
├── SKILL.md
└── references/
    ├── MULTI-STAGE.md
    ├── BUILD-CACHE.md
    └── CHEATSHEET.md
```

## What Made This Skill Effective

1. **Focused scope** — Only optimization, not "everything Docker"
2. **Specific triggers** — Users searching for "size" or "speed" match
3. **Actionable examples** — Copy-paste Dockerfile snippets
4. **Clear boundaries** — What to use vs skip
5. **Deep dives available** — references/ for more details