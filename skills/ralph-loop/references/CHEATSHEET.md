# Ralph Loop — Cheat Sheet

> Role of this document: quick reference. The canonical reference remains [`../SKILL.md`](../SKILL.md), which defines the reference prompt and bounded loop.

## Golden Rule

**Spec first, loop second.**

Default setup:

```text
docs/specs/<feature>.md
progress.txt
tests + code aligned with the spec
```

Some platforms may require `task.md`, but the repo spec remains the primary reference.

## Reference Pattern

Use the **canonical prompt** and **bounded loop** defined in [`../SKILL.md`](../SKILL.md).

Structure reminder:

- One spec in `docs/specs/<feature>.md`
- One `progress.txt`
- A bounded iteration budget
- One behavior per iteration
- Binary completion rules

## Minimal Anti-Pattern to Avoid in Production

```bash
while :; do cat docs/specs/<feature>.md | my-agent; done
```

❌ Anti-pattern: this form includes no `progress.txt`, no bounded budget, and no explicit stop condition.

## Mental Model

```text
spec <-> tests <-> code
          ↑
     Ralph Loop iterates here
```

## Typical Binary Checks

- `npm run test` returns 0
- `npm run build` returns 0
- `npm run lint` returns 0
- forbidden-pattern search finds no results
- the independent review verdict is `SHIP`

## Canonical Prompt

See [`../SKILL.md`](../SKILL.md), section **Canonical Prompt Pattern**.

## Quick Decision Table

| Situation | Decision |
|---|---|
| Tests/build/lint produce a binary verdict | Ralph Loop is a good fit |
| The task is local and repetitive | Ralph Loop is a good fit |
| Human judgment is required | Bring in a human |
| Acceptance criteria are vague | Rewrite the spec before launching |

## Warning Signs

- The loop runs for a long time without convergence.
- No visible progress appears in `progress.txt`.
- Acceptance criteria are rewritten just to make checks pass.
- Abstractions are added without need.
- Cost rises while quality signals do not improve.
