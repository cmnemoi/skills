---
name: ralph-loop
description: Proactively apply when running autonomous coding loops with context resets. Triggers on ralph loop, ralph wiggum, context rot, spec-first, docs/specs, progress.txt, and binary checks. Use when a task is mechanical, externally verifiable, and guided by a short repo spec.
---

# Ralph Loop

> Core idea: Ralph Loop works best when it executes a **short, testable spec stored in the repo**. By default, think **spec first, loop second**.
> Role of this document: **canonical operational reference**. See [`references/CHEATSHEET.md`](references/CHEATSHEET.md) for the short version and [`examples/bounded-migration.md`](examples/bounded-migration.md) for a concrete example.

---

## When to Use / Skip

| Use When | Skip When |
|----------|-----------|
| Large mechanical migration | Complex architecture decision |
| Hundreds of lint fixes | API design or domain modeling |
| Adding tests to existing code | UX/UI or aesthetic choices |
| Repetitive refactor with binary checks | Sensitive security work without human review |
| Task driven by tests/build/lint | Vague or subjective completion rule |

Rule of thumb: Ralph Loop is an **execution engine**, not a design tool.

---

## Terminology

- **spec**: repo file, by default `docs/specs/<feature>.md`
- **progress**: `progress.txt`, inter-iteration memory
- **completion rule**: binary exit condition
- **bounded loop**: loop with an iteration, time, or cost budget
- **reviewer**: agent or independent step that returns a verdict
- **review-feedback.txt**: detailed reviewer feedback
- **review-result.txt**: `SHIP` / `REVISE` verdict
- **tool-specific variant**: `task.md` may exist in some tools

## Quick Decisions

### "Should I use Ralph Loop here?"

```text
What kind of work is this?
├─ Mechanical change with objective checks           → Yes
├─ Test-driven bugfix                               → Yes
├─ Repetitive refactor without behavior change      → Yes
├─ Architecture or design still unclear             → No
└─ Subjective completion rule                       → No, rewrite the spec first
```

### "What contract should I prepare?"

```text
How much formality is needed?
├─ Standard case                                    → One spec in docs/specs + progress.txt
├─ High review stakes                               → + review-feedback.txt / review-result.txt
├─ Tool requires a different file                   → task.md as a spec-aligned copy
└─ Small coherent change                            → One spec, no separate product requirements document
```

### "When can the loop stop?"

```text
What result is objectively verifiable?
├─ Valid build                                      → command returns 0
├─ Valid tests                                      → suite returns 0
├─ Migration complete                               → forbidden-pattern search finds no results
├─ Review approved                                  → binary verdict is SHIP
└─ All acceptance criteria are satisfied            → agent emits <promise>COMPLETE</promise>
```

---

## Recommended Model: spec, tests, and code stay aligned

```text
spec <-> tests <-> code
          ↑
     Ralph Loop iterates here
```

- **spec**: states intent and rules
- **tests**: prove the expected behavior
- **code**: implements the behavior

The loop should not bypass this triangle.

---

## Recommended Workflow
1. Write or repair the spec.
2. Link tests to the spec.
3. Link code to the spec when useful.
4. Calibrate a first iteration with a human in the loop.
5. Run the bounded loop.
6. Verify final alignment between spec, tests, and code.

Two human checkpoints matter:

1. **Before implementation**: are the spec and its criteria sufficient?
2. **Before merge**: do the spec, tests, and code still say the same thing?

---

## What a Good Spec Should Contain

Minimum useful structure:

1. **Why**
2. **Scope**
3. **Rules**
4. **Acceptance criteria**
5. **Out of scope**

Important rules should ideally have **stable IDs**:

- `migration::replace-jest-imports`
- `checkout::applies-member-discount`

The `{#rule-id}` notation is only one way to mark stable IDs in markdown. If your renderer does not support it, keep the ID as plain text near the rule heading.

---

## Example Spec
```md
# Jest → Vitest Migration

## Why

The current test suite uses Jest, which is incompatible with the target tooling.

## Scope

This spec covers the import and script changes required to run the test suite with Vitest.

## Rules

### Jest imports are replaced

`{#migration::replace-jest-imports}`

No test file may import from `jest` or `@jest/globals`.

### Test scripts use Vitest

`{#migration::use-vitest-scripts}`

`package.json` must point test scripts to `vitest`.

## Acceptance criteria

- When `npm run test` is executed, the command returns exit code 0.
- When searching for imports from `jest` or `@jest/globals`, the search finds no results.
- When `npm run build` is executed, the command returns exit code 0.

## Out of scope

- Rewriting business assertions
- Migrating complex mocks
```

---

## Minimal State Files

| File | Role | Status |
|---|---|---|
| `docs/specs/<feature>.md` | Behavior contract | Required |
| `progress.txt` | Inter-iteration memory | Required |
| `review-feedback.txt` | Review feedback | Optional |
| `review-result.txt` | Binary verdict | Optional |

Use **one repo spec** per coherent behavior rather than a separate product requirements document.

If `task.md` exists alongside a repo spec, the repo spec remains the primary reference; `task.md` should only be an aligned copy.

---

## Binary Criteria
| ❌ Vague | ✅ Binary |
|---|---|
| `Make the code cleaner` | `npm run lint` returns exit code 0 |
| `Improve quality` | forbidden-pattern search finds no results |
| `Make it work correctly` | `npm run test` returns exit code 0 |
| `Improve performance` | an explicit measurement stays within a documented threshold |

---

## Canonical Prompt Pattern

```text
@docs/specs/<feature>.md @progress.txt
1. Read the spec and progress.
2. Pick the highest-priority unresolved rule or acceptance criterion.
3. Work on exactly one behavior.
4. Update tests and code for that behavior.
5. Run the relevant objective checks.
6. Update progress.txt with the spec IDs touched and any blocker.
7. Do not expand the scope.
8. Do not change the acceptance criteria to make checks pass.
9. If all acceptance criteria are satisfied, output `<promise>COMPLETE</promise>`.
```

Use this token when your tool can detect a final text marker. Otherwise, use a reliable exit code or verdict file.

---

## Spec Modification Policy

By default, the spec stays **stable during the loop**.

There is **no durable conversational memory** between iterations; continuity comes from repo files.

Acceptable, but explicitly justified:

- Clarify a real ambiguity.
- Correct a factual error.
- Add a missing rule revealed by tests.

Not acceptable:

- Widen the scope.
- Remove a completion rule to make the loop pass.
- Turn the spec into a file-by-file checklist.

---

## Traceability with Stable IDs

When possible, link each important behavior to:

1. the spec
2. at least one test
3. at least one implementation location
4. `progress.txt`
5. an atomic commit

Example:

```text
Iteration 3: {#migration::replace-jest-imports} implemented.
Tests linked to @spec migration::replace-jest-imports updated.
Blocker: 3 files still use jest.mock() with no direct equivalent.
```

---

## Separate Review

When the stakes justify it:

- **reviewer**: the agent or independent step that returns a binary verdict
- **worker**: the agent that changes tests and code

Recommended verdicts:

- `SHIP`
- `REVISE`

The reviewer should read files, not the full conversational history.

---

## Anti-Patterns

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Infinite loop without budget | unbounded cost | define an iteration, time, or cost budget |
| Vague spec | drift and overengineering | rewrite a short, testable spec |
| Too many markdown artifacts | cognitive overhead | start with one repo spec |
| Multiple behaviors per iteration | scope creep | enforce one behavior per pass |
| Weak checks | false sense of completion | strengthen tests, build, lint, or review |
| Tests and code not linked to the spec | weak traceability | use stable IDs and `@spec` |

---

## Quick Checklist

- [ ] A short spec exists in the repo.
- [ ] The spec contains rules, acceptance criteria, and out-of-scope boundaries.
- [ ] Completion rules are binary.
- [ ] `progress.txt` exists.
- [ ] The loop is bounded.
- [ ] Tests actually catch regressions.
- [ ] The loop works on one behavior per iteration.
- [ ] A final checkpoint verifies alignment between spec, tests, and code.

---

## Sources

### Primary references
- Geoffrey Huntley — *Inventing the Ralph Wiggum Loop* — https://linearb.io/dev-interrupted/podcast/inventing-the-ralph-wiggum-loop
- Geoffrey Huntley — *Mastering Ralph loops transforms software engineering with LLM automation* — https://linearb.io/blog/ralph-loop-agentic-engineering-geoffrey-huntley
- Vercel Labs — `ralph-loop-agent` — https://github.com/vercel-labs/ralph-loop-agent
- Block / Goose — *Ralph Loop* — https://goose-docs.ai/docs/tutorials/ralph-loop

### Complementary references
- Skill `spec-driven-development` — use it to write or revise the spec before launching the loop
- Blake Crosley — *Why My AI Agent Has a Quality Philosophy* — https://blakecrosley.com/blog/jiro-quality-philosophy
