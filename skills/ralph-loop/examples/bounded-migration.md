# Example: Bounded Migration Using Ralph Loop

> Role of this document: worked example. The canonical reference remains [`../SKILL.md`](../SKILL.md), which defines the reference prompt and bounded loop.

## Scenario

This example migrates a Jest test suite to Vitest without changing the business meaning of the tests.

## Why Ralph Loop Fits

- The task is mechanical.
- Many files need to change.
- The completion rules are binary.
- Regressions are detectable through tests and build checks.

## Initial File Layout

```text
project/
├── docs/specs/migration-jest-vitest.md
├── progress.txt
├── package.json
├── vite.config.ts
└── tests/
```

## Minimal Spec

```md
# Jest → Vitest Migration

## Why

The test suite must run on Vitest.

## Scope

This spec covers migrating Jest imports and test scripts.

## Rules

### Jest imports are replaced

`{#migration::replace-jest-imports}`

No test file may import from `jest` or `@jest/globals`.

### Test scripts use Vitest

`{#migration::use-vitest-scripts}`

`package.json` must point test scripts to `vitest`.

## Acceptance criteria

- When `npm run test` is executed, the command returns exit code 0.
- When `npm run build` is executed, the command returns exit code 0.
- When searching for imports from `jest` or `@jest/globals`, the search finds no results.

## Out of scope

- Rewriting business assertions
- Redesigning the test architecture
```

## Working Prompt

Use the **canonical prompt** defined in [`../SKILL.md`](../SKILL.md), applied here to the spec reference `@docs/specs/migration-jest-vitest.md`.

The `@` prefix is tool-specific shorthand for attaching or referencing a file in the agent prompt.

## Bounded Loop

Use the **canonical bounded loop** defined in [`../SKILL.md`](../SKILL.md), replacing the spec path with `docs/specs/migration-jest-vitest.md`.

## Example progress.txt

```text
Iteration 1: {#migration::replace-jest-imports} implemented in 24 files.
Tests linked to {#migration::replace-jest-imports} updated.
Blocker: 3 files still use jest.mock() with no direct Vitest equivalent.
Next step: {#migration::use-vitest-scripts}.
```

## Minimal Traceability Example

```ts
/** @spec migration::replace-jest-imports */
it("replaces Jest imports across the test suite", () => {
  // ...
});

/** @spec migration::replace-jest-imports */
function migrateTestImports(sourceText: string): string {
  // ...
}
```

## Why This Example Converges

- **One behavior per iteration**: the loop handles only one rule or acceptance criterion at a time.
- **Objective checks**: completion depends on `npm run test`, `npm run build`, and a forbidden-pattern search.
- **Explicit out-of-scope boundary**: the spec forbids rewriting business assertions or redesigning test architecture.
- **Bounded budget**: the loop stops after `max_iterations`, even if it fails.
- **Persistent state across sessions**: the spec and `progress.txt` let each iteration restart with the right context.
- **File-based continuity**: progress comes from the spec and `progress.txt`, not from retained conversational context.
