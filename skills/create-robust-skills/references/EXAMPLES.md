# Example Scenarios

Real-world examples of skill usage and structure.

## Example 1: Full Skill Structure

### Skill: python-async

```
python-async/
├── SKILL.md
├── references/
│   ├── ASYNCIO.md
│   ├── AIOHTTP.md
│   └── CHEATSHEET.md
└── examples/
    ├── web-scraper.md
    └── api-client.md
```

### SKILL.md Content

```markdown
---
name: python-async
description: Proactively apply when writing asynchronous Python code. Triggers on async, await, asyncio, aiohttp, concurrent, asyncpython. Use when making async HTTP requests, running concurrent tasks, or converting sync to async code. Asynchronous Python patterns and best practices.
---

# Python Async

Write efficient async Python code.

## Quick Decision Trees

### "Which async library?"

What do you need?
├─ HTTP requests        → aiohttp
├─ Database queries    → asyncpg, aiomysql
├─ File operations     → aiofiles
├─ Multiple tasks      → asyncio.gather
└─ Rate limiting       → aiolimiter
```

## Example 2: Minimal Skill

### Skill: git-undo

```
git-undo/
└── SKILL.md
```

### SKILL.md Content

```markdown
---
name: git-undo
description: Proactively apply when undoing Git changes. Triggers on git undo, git revert, git reset, git checkout, undo commit, remove changes. Use when accidentally committed, need to undo changes, or want to revert to previous state. Git undo operations and safety guidelines.
---

# Git Undo

Quick reference for undoing Git changes.

## Decision Tree

```
What do you need?
├─ Undo working changes    → git checkout .
├─ Unstage file            → git reset HEAD file
├─ Undo last commit        → git reset --soft HEAD~1
├─ Completely undo commit → git reset --hard HEAD~1
└─ Revert commit          → git revert hash
```

## Examples

### Undo Working Changes

```bash
# ❌ Danger: removes all changes
git checkout .

# ✅ Safer: remove specific file
git checkout -- file.txt
```

### Undo Last Commit

```bash
# Keep changes staged
git reset --soft HEAD~1

# Keep changes unstaged
git reset HEAD~1

# Completely remove (DANGER)
git reset --hard HEAD~1
```
```

## Example 3: Trigger Writing

### Original (Bad)
```yaml
---
name: js
description: Helps with JavaScript.
---
```

### Improved
```yaml
---
name: modern-javascript
description: Proactively apply when writing JavaScript. Triggers on javascript, js, es6, modern js, array methods, destructuring, arrow functions, async await. Use when writing new JS code, refactoring legacy code, or learning modern JavaScript. ES6-ES2024 patterns and best practices.
---
```

## Example 4: Anti-Patterns Table

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Only theory | User can't implement | Add ❌/✅ code examples |
| No navigation | Can't find relevant info | Add decision trees |
| Missing scope | Not clear when to use | Add "When to use" section |
| Too long | Hard to find info | Split into references/ |