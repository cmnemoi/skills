# Example: DSL-First Unit Test with In-Memory Dependencies

## Scenario

You need to test that a checkout service rejects orders when stock is insufficient.

## Step-by-step

1. Start with a small business DSL that speaks business language.
2. Run it through an in-memory path.
3. Keep the test to one Act step.
4. Assert the business outcome and a visible side effect.

## Result

✅ Behavior-focused test with in-memory dependencies

```typescript
it('rejects checkout when stock is insufficient', (): void => {
  const checkoutScenarioDriver: CheckoutScenarioDriver = createOutOfStockCheckoutScenarioDriver()

  checkoutScenarioDriver.whenCustomerChecksOut()

  checkoutScenarioDriver.thenCheckoutIsRejected()
  checkoutScenarioDriver.thenNoOrderIsSaved()
})
```

## Why this is good

- It follows the DSL-first default for new code.
- It checks a business rule.
- It avoids mocking internal collaboration.
- It would survive internal refactoring if behavior stays the same.
- It runs quickly and deterministically.

## ❌ Anti-Pattern: Interaction-heavy test coupled to implementation

Avoid dropping back to low-level setup in the test body, for example by instantiating repositories and services directly inside the test. Even when that style is still technically fast, it weakens the DSL-first default and makes the test speak implementation structure instead of business language.

## See also

- [Main skill](../SKILL.md)
- [Test doubles](../references/TEST-DOUBLES.md)
- [Cheat sheet](../references/CHEATSHEET.md)
- [DSL-driven testing](../../dsl-driven-testing/SKILL.md)
