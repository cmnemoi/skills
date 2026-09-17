---
name: spec-driven-development
description: Proactively apply when defining feature behavior before implementation, keeping specs aligned with tests and code, or introducing lightweight spec workflows. Triggers on spec-driven development, SDD, spec-first, acceptance criteria, spec -> tests -> code, @spec, behavioral traceability, feature spec, bugfix spec, design-first spec. Use when building or changing production features, clarifying expected behavior, or reducing drift between intent and implementation. Pragmatic Spec-Driven Development centered on a lightweight spec-test-code triangle.
---

# Spec-Driven Development

> Core idea: write a **short, testable, repo-anchored spec**, then use it as the pivot between **tests** and **code**.
>
> Recommended model: not `spec -> code`, but a **triangle**: `spec <-> tests <-> code`.

---

## When to Use / Skip

| Use When | Skip When |
|----------|-----------|
| Behavior is important and should remain understandable after delivery | Throwaway demo, spike, or one-shot script |
| A feature needs clear acceptance criteria before coding | Small exploratory change where the shape is still unknown |
| You want durable intent, not a chat prompt that disappears | Hotfix requiring immediate manual intervention |
| Multiple people or agents will touch the feature over time | Very small local refactor with no behavior change |
| The cost of drift or regression is significant | The process cost would exceed the protection gained |

Rule of thumb: use SDD for **durable production behavior**, not for disposable exploration.

---

## Quick Decision Trees

### "Should I use SDD here?"

```
What kind of work is this?
├─ New or changed production behavior              → Yes, use SDD
├─ Bugfix with regression risk                     → Yes, use bugfix spec
├─ Architecture already decided, behavior to pin   → Yes, use design-first spec
├─ Pure refactor, no behavior change               → Usually no
└─ Prototype / spike / throwaway demo              → No, skip formal SDD
```

### "Which spec type should I write?"

```
What are you trying to control?
├─ New feature or extension                        → Feature spec
├─ Existing design is already chosen               → Design-first spec
├─ Fix one bug without widening scope              → Bugfix spec
└─ Large unknown area                              → Investigate first, then spec
```

### "How much documentation do I need?"

```
How complex is the change?
├─ Small/medium feature                            → One spec file
├─ Complex multi-step delivery                     → Spec + small local plan
├─ External contract matters                       → Spec + separate API contract
└─ Tiny isolated fix                               → Short bugfix spec only
```

### "When is a behavior really implemented?"

```
Is the behavior triangulated?
├─ Exists in spec only                             → Not implemented yet
├─ Spec + tests, no code reference                 → Fragile / incomplete
├─ Spec + code, no test                            → Unverified
└─ Spec + test + code all linked                   → Properly implemented
```

---

## Workflow

Use the lightest workflow that still preserves intent: spec first, tests second, code third.

1. Write or update the spec.
2. Write tests linked to the spec.
3. Implement code linked to the spec.
4. Revise the spec if tests or implementation reveal a missing rule.
5. Review alignment before merge.

On an existing system, start by extracting or repairing the relevant spec, then follow the same order.

Two human checkpoints matter more than extra documents:

1. Before implementation: the spec and acceptance criteria are sufficient.
2. Before merge: spec, tests, and code still say the same thing.

None of this requires code generation or spec-as-source dogma.

---

## The Spec-Test-Code Triangle

```text
        spec
       /    \
      /      \
   tests ---- code
```

- **Spec** states intent and observable behavior.
- **Tests** prove the intended behavior.
- **Code** implements the behavior.

This is safer than test/code alone:

- tests can validate the wrong thing
- code can pass tests while missing business intent
- specs can become vague if never connected to execution

Treat drift anywhere in the triangle as a problem to reconcile.

---

## What a Good Spec Contains

Keep specs short, behavioral, and testable.

### Minimum structure

1. **Why**: what problem this change solves
2. **Scope**: what the spec covers
3. **Rules**: the important behaviors or invariants
4. **Acceptance criteria**: concrete, testable examples
5. **Out of scope**: what is intentionally excluded

### Quality checks

For each important rule, ask:

- Can a test be written against it?
- Can a human understand it without reverse engineering the code?
- Can an agent use it to produce or modify code?
- Is it describing observable behavior rather than internal mechanics?

### Avoid

- file-by-file structure
- speculative implementation details
- pseudo-code too early
- long lists of nice-to-have ideas
- multi-document process by default

---

## Adversarial Spec Review

A spec that reads well is not yet a good spec. Before planning or coding, **try to break it**: find the rule that is false, ambiguous, or missing while it is still cheap to fix.

For each important rule, ask:

- Which business assumption could simply be wrong?
- Which nominal or edge case would produce an incorrect answer under this rule?
- Which external failure or missing data is left undefined?
- Which rule contradicts behavior the system already has?
- Which scenario would pass with a shallow implementation but be wrong in production?
- Which non-functional requirement that really matters is not observable anywhere?

A rule that cannot be attacked at all is usually too vague to test.

### ZOMBIES: a search grid for missing behavior

```text
Z — Zero        : absence, empty input, no result
O — One         : minimal case with a single element
M — Many        : several elements, repetition, volume
B — Boundary    : thresholds, limits, extreme formats
I — Interface   : public contract or external boundary
E — Exceptional : error, timeout, unavailability, invalid input
S — Simple      : the minimal nominal scenario
```

Keep **only the dimensions that carry risk**.

```
Which ZOMBIES dimensions apply here?
├─ Collection, list, or search result       → Zero, One, Many
├─ Threshold, quota, date, size, format     → Boundary
├─ Public contract or external call         → Interface, Exceptional
├─ Depends on a system that can fail        → Exceptional
└─ Pure deterministic transformation        → Simple, Boundary
```

It is a grid for finding defects, not a quota: seven dimensions per rule is padding, not strength.

**Exceptional needs a decision, not a reflex.** A functional edge case (empty result, expired membership, a quantity on the threshold) is a rule you can decide and write today. A technical failure mode (timeout, concurrent write, partial outage) becomes a spec rule only once the business has an answer for it. If nobody has decided, write the question rather than an invented rule: specifying a retry or a reconciliation nobody asked for commits the implementation to machinery that costs forever.

```md
<!-- ❌ Happy path only: nothing here can be wrong -->
- When the customer searches for a product, matching products are returned.

<!-- ✅ Attacked: the failing and empty cases are decided, not implied -->
- When the query matches nothing, an empty result is returned, not an error.
- When the catalog service is unavailable, the search fails explicitly instead of returning an empty result.
```

### Make important non-functional rules observable

A violated non-functional requirement (latency, a size limit, an ordering guarantee, a retention rule) is a real defect. When one genuinely matters, state it as an observable rule with acceptance criteria; otherwise put it in **Out of scope** on purpose.

Do not promote every maintainability preference to a spec rule: it inflates the spec and blurs impact, risk, and debt.

### When to stop

A system always has more potential defects than anyone will find, so a spec is never finished "because no case is left". Stop when the marginal cost of hunting one more case exceeds the criticality of what you would find, record the risks you knowingly accept in **Out of scope**, and never claim exhaustiveness.

---

## Traceability with Stable IDs

The most useful SDD habit is explicit linking inside the repo. Give each behavior a stable, behavior-oriented ID such as `checkout::applies-member-discount` or `auth.session::expires-after-inactivity`, then reference it in the spec, in at least one test, and in at least one implementation location.

```md
### Member discount is applied

`{#checkout::applies-member-discount}`

When a signed-in member confirms checkout, the final total includes the member discount.
```

```ts
/** @spec checkout::applies-member-discount */
it("applies the member discount at checkout", () => { /* ... */ });

/** @spec checkout::applies-member-discount */
function computeCheckoutTotal(input: CheckoutInput): Money { /* ... */ }
```

A behavior is only solid when the repo contains the full triangle. [references/SPEC-TEMPLATE.md](references/SPEC-TEMPLATE.md) holds the ID conventions and what to avoid.

---

## Acceptance Criteria

Acceptance criteria are the operational heart of the spec.

- Prefer business language over UI or transport details.
- Prefer bullets by default; promote only risky or ambiguous cases to heavier scenarios.
- Write them in a form that can be converted almost directly into tests.

```md
## Acceptance criteria

- Given a signed-in member with eligible items, when checkout is confirmed, then the final total reflects the member discount.
- Given a guest with the same items, when checkout is confirmed, then the final total remains unchanged.
```

Good criteria describe domain truth, not clicks and endpoints.

---

## Spec Variants

| Variant | Use it when | Contains |
|---------|-------------|----------|
| **Feature spec** | New behavior or an extension | Intent, scope, behavioral rules, acceptance criteria, out of scope |
| **Design-first spec** | Architecture or constraints are already fixed | The externally relevant constraints and the behavior the design must preserve |
| **Bugfix spec** | Fixing a defect without widening scope | Reproduction, current behavior, expected behavior, unchanged behavior |

The three templates live in [references/SPEC-TEMPLATE.md](references/SPEC-TEMPLATE.md). A design-first spec still describes behavior, not an implementation narrative, and a bugfix spec exists to stop the agent from "fixing" more than intended.

**Reproduce before fixing.** The first deliverable of a bugfix is not the fix, it is a failing test:

```text
observed defect -> minimal reproduction -> RED test -> fix -> suite GREEN -> permanent regression guard
```

Fixing first is how the wrong mechanism gets repaired and declared fixed because the incident stopped reproducing by hand. The red test separates the defect's existence from your explanation of it.

## File and Granularity Rules

Default to one spec file per coherent feature or sub-feature:

- `docs/specs/<feature>.md`

Only add more artifacts when there is a real reason:

- a small local plan for genuinely multi-step work
- a research note worth preserving
- a separate API contract when the interface itself matters

Do not create a PRD, design doc, task list, data model, and quickstart by default for a feature that fits in one short spec.

Process should reduce the reader's work, not move the work from code to markdown.

---

## Good vs Bad Spec Language

```md
<!-- ❌ Too technical -->
- Click the green checkout button and call POST /v1/cart/apply.

<!-- ✅ Behavioral -->
- Apply the member discount to the final checkout total.
```

```ts
// ❌ Test not linked to intent
it("applies discount", () => {
  // ...
});

// ✅ Test linked to the spec
/** @spec checkout.pricing.applies-member-discount::member-discount-applied */
it("applies the member discount at checkout", () => {
  // ...
});
```

---

## Anti-Patterns

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Specfall | Over-detailed specs create process paralysis | Keep micro-specs focused on behavior |
| Specification theater | Specs exist but are not reconciled with code | Review alignment before merge |
| Phantom completions | Tasks marked done without real implementation | Require spec/test/code evidence |
| Spec as prompt dump | Unstructured and non-maintainable | Use IDs, rules, and acceptance criteria |
| Too many files | Review overhead exceeds value | Start with one spec file |
| Spec too technical too early | Mixes intent with implementation | Keep the spec behavioral |
| Swarm of sub-agents | Coordination costs more than it helps | Keep one main agent and a short workflow |
| Tests/code without `@spec` links | Weak traceability | Add explicit stable references |
| Happy-path spec | Only confirms the author's intuition; the failure modes stay undecided | Run an adversarial review and decide the relevant ZOMBIES cases |
| ZOMBIES as a quota | Seven dimensions per rule inflate the spec without adding risk coverage | Keep only the dimensions that carry real risk |
| Fix-then-spec bugfix | The fix may address the wrong mechanism, with no lasting guard | Reproduce first, red test first |

Practical heuristic: if a spec generates more than about 10 tasks for less than 3 days of work, it is probably too granular.

---

## Working with ATDD

```text
spec -> acceptance criteria -> executable tests -> code
```

The spec defines behavior and acceptance criteria; the tests speak domain language; technical details belong in adapters, drivers, or helpers. Load [`dsl-driven-testing`](../dsl-driven-testing/SKILL.md) alongside this skill for acceptance tests that stay stable across in-memory, API, and UI levels, and [`write-unit-tests`](../write-unit-tests/SKILL.md) for the test posture itself.

---

## Review Checklist

Before considering the work complete, verify:

- the spec is short, readable, and stored in the repo
- important behaviors have stable IDs
- acceptance criteria are testable and written in business language
- tests reference the relevant `@spec` IDs
- implementation references the relevant `@spec` IDs
- spec, tests, and code do not contradict each other
- the spec went through an adversarial review instead of only being proofread
- the ZOMBIES dimensions that carry risk are decided, and the irrelevant ones are left out
- important non-functional requirements are observable or explicitly out of scope
- a bugfix spec carries a reproduction, and the red test came before the fix
- the workflow stayed proportionate to the problem size

---

## Reference Documentation

| File | Purpose |
|------|---------|
| [references/SPEC-TEMPLATE.md](references/SPEC-TEMPLATE.md) | Feature, bugfix, and design-first templates, ID conventions, writing heuristics |
| [examples/exchange-body.md](examples/exchange-body.md) | A worked spec |
| [../write-unit-tests/SKILL.md](../write-unit-tests/SKILL.md) | Turning attacked rules into tests |
| [../dsl-driven-testing/SKILL.md](../dsl-driven-testing/SKILL.md) | Acceptance test architecture |

## Sources

### Primary sources
- [Martin Fowler - Understanding Spec-Driven Development: Kiro, spec-kit, and Tessl](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)
- [Thoughtworks Technology Radar - Spec-driven development](https://www.thoughtworks.com/radar/techniques/spec-driven-development)
- [GitHub spec-kit - spec-driven.md](https://github.com/github/spec-kit/blob/main/spec-driven.md)
- [GitHub Blog - Spec-driven development with AI: Get started with a new open source toolkit](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)
- [Kiro - New spec types: fix bugs and build on top of existing apps](https://kiro.dev/blog/specs-bugfix-and-design-first/)
- [couzic/ts-graph-mcp - specs/CLAUDE.md](https://github.com/couzic/ts-graph-mcp/blob/master/specs/CLAUDE.md)

### Academic and research references
- [Constitutional Spec-Driven Development: Enforcing Security by Construction in AI-Assisted Code Generation](https://arxiv.org/abs/2602.02584)
- [Spec-Driven Development: From Code to Contract in the Age of AI Coding Assistants](https://arxiv.org/abs/2602.00180)
- [VibeContract: The Missing Quality Assurance Piece in Vibe Coding](https://arxiv.org/abs/2603.15691)
- [Spec Kit Agents: Context-Grounded Agentic Workflows](https://arxiv.org/abs/2604.05278)

### Related practices
- Yegor Bugayenko, *Angry Tests*: testing as defect search, and the unbounded-defect view behind the stopping rule. See [`write-unit-tests/references/ANGRY-TESTS.md`](../write-unit-tests/references/ANGRY-TESTS.md)
- [Dan North - Introducing BDD](https://dannorth.net/blog/introducing-bdd/)
- [Martin Fowler - Specification by Example](https://martinfowler.com/bliki/SpecificationByExample.html)
