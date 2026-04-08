# Writing Style Guide

Guidelines for writing clear, actionable skill content.

## Tone and Voice

### Do
- Be direct and actionable
- Use imperative: "Use X" not "You should use X"
- Provide solutions, not just explanations
- Be concise — get to the point

### Don't
- Be overly formal or academic
- Use filler phrases ("As you can see", "It is worth noting")
- Be vague ("consider using...")
- Explain things unnecessarily

## Examples

```markdown
# Good
Use .map() to transform array elements.

# Bad
When working with arrays, you might want to consider using the .map() method which allows you to transform each element.
```

## Formatting Rules

### Headers
- Use ## for main sections
- Use ### for subsections
- Keep headers descriptive

```markdown
## Array Methods

### .map() vs .filter()
```

### Lists
- Use bullet points for unordered
- Use numbers for sequential steps
- Keep items parallel in structure

```markdown
- Create the component
- Add the props
- Export the result
```

### Tables
- Use for comparisons, decisions
- Keep columns aligned
- Use header row

```markdown
| Method | Use When |
|--------|----------|
| .map() | Transform elements |
| .filter() | Keep some elements |
```

### Code Blocks
- Always specify language
- Use comments sparingly (only when needed)
- Show complete, runnable examples

```javascript
// ✅ Good
const doubled = nums.map(n => n * 2);

// ❌ Bad (no language, incomplete)
const doubled = nums.map(n => n * 2)
```

## Content Organization

### Lead with the Answer
Start with the solution, then explain.

```markdown
## Pattern: Use .at(-1) for Last Element

❌ Legacy:
const last = arr[arr.length - 1];

✅ Modern:
const last = arr.at(-1);
```

### Show Trade-offs
When there are multiple approaches:

```markdown
## Option A: .map()

Use when transforming every element.

## Option B: .filter() + .map()

Use when you need to filter first, then transform.
```

### Group Related Content
Keep similar topics together.

```markdown
## Error Handling

### Try/Catch Pattern
[content]

### Result Type Pattern
[content]
```

## Common Style Mistakes

| Mistake | Fix |
|---------|-----|
| Walls of text | Use headers, lists, tables |
| No code examples | Add ❌/✅ code blocks |
| Missing context | Explain when to use |
| Incomplete examples | Show full, working code |
| Inconsistent formatting | Follow this guide |