# Spec Template

Use this template for most feature-level SDD work. Keep it short. Add only what helps behavior stay testable and understandable.

---

## Standard Feature Spec

```md
# <Feature Name>

## Why

<What problem this solves and why it matters>

## Scope

<What this spec covers>

## Rules

### <Behavior name>

`{#<domain-or-feature>::<behavior-id>}`

<Observable behavior stated in business language>

### <Behavior name>

`{#<domain-or-feature>::<behavior-id>}`

<Observable behavior stated in business language>

## Acceptance criteria

- Given <context>, when <action>, then <outcome>.
- Given <context>, when <action>, then <outcome>.

## Out of scope

- <Explicitly excluded item>
- <Explicitly excluded item>
```

---

## Bugfix Spec

Use this variant when you need to fix behavior without widening scope.

```md
# <Bugfix Name>

## Why

<What is broken and why it matters>

## Current behavior

<What happens today>

## Expected behavior

### <Behavior name>

`{#<domain-or-feature>::<behavior-id>}`

<What must happen after the fix>

## Unchanged behavior

- <Behavior that must remain unchanged>
- <Behavior that must remain unchanged>

## Acceptance criteria

- Given <broken context>, when <action>, then <correct outcome>.
- Given <neighboring context>, when <action>, then <unchanged outcome>.
```

---

## Design-First Spec

Use this variant when architecture or constraints are already fixed and must be respected.

```md
# <Feature Name>

## Why

<What problem this solves>

## Scope

<What this spec covers>

## Required constraints

- <Constraint with user-visible or design impact>
- <Constraint with user-visible or design impact>

## Rules

### <Behavior name>

`{#<domain-or-feature>::<behavior-id>}`

<Behavior the chosen design must support>

## Acceptance criteria

- Given <context>, when <action>, then <outcome>.
```

---

## ID Conventions

Prefer stable, behavior-oriented IDs:

- `checkout::applies-member-discount`
- `auth.session::expires-after-inactivity`
- `action.exchange-body::preserves-character-bound-values`

Good IDs are:

- stable over time
- tied to behavior, not file names
- specific enough to be testable
- readable by humans and agents

Avoid:

- `discount-code`
- `feature-1`
- `checkout-service-line-42`

---

## Traceability Checklist

For each important behavior, verify:

1. The spec contains the behavior and its ID.
2. At least one test references `@spec <id>`.
3. At least one implementation location references `@spec <id>`.

Example:

```ts
/** @spec checkout::applies-member-discount */
it("applies the member discount at checkout", () => {
  // ...
});

/** @spec checkout::applies-member-discount */
function computeCheckoutTotal(input: CheckoutInput): Money {
  // ...
}
```

---

## Writing Heuristics

Keep a rule if the answer is yes to all of these:

- Can I write a test for it?
- Can a reviewer understand it quickly?
- Does it describe observable behavior?
- Would it still make sense if files were reorganized?

Cut or rewrite lines that are:

- implementation-first
- speculative
- too broad to verify
- just backlog notes in disguise
