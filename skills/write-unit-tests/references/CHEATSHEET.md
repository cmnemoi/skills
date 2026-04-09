# Write Unit Tests Cheat Sheet

## Defaults

- For new code, start with a **small business DSL**.
- Test **behavior**, not implementation.
- Prefer **real in-memory dependencies**.
- Prefer **fakes** over **mocks**.
- Use **AAA**.
- Keep **one behavior per test**.
- Treat **coverage as a clue, not a goal**.

## Default Architecture

```text
Test case / scenario → small business DSL → in-memory driver → system under test
```

Brownfield exception: skip or delay the DSL only when migration cost and team disruption clearly outweigh the near-term value.

## Good Unit Test Formula

```text
Small business scenario + small DSL + one action + meaningful assertion + deterministic runtime
```

## Choose a Test Double

```text
Real dependency possible?                      → Use it
Need a realistic in-memory replacement?        → Fake
Need fixed response only?                      → Stub
Need to record calls?                          → Spy
Need to verify an unavoidable outbound command? → Mock rarely
```

## Smell Radar

- Breaks after harmless refactor → too implementation-coupled
- Large setup, tiny assertion → obscure test
- Many assertions, vague failure → assertion roulette
- DB/files/network in unit test → hidden integration test
- No assertion → fake confidence

## See also

- [Main skill](../SKILL.md)
- [Test doubles](TEST-DOUBLES.md)
- [Service example](../examples/SERVICE-EXAMPLE.md)
- [DSL-driven testing](../../dsl-driven-testing/SKILL.md)

## Pre-merge Checklist

- Fast?
- Deterministic?
- Refactor-safe?
- Easy to read?
- Valuable failure message?
- No unnecessary mocks?
