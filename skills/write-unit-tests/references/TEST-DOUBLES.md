# Test Doubles

Use this reference **inside** a DSL-first or scenario-first test design whenever possible. Choose the double at the boundary, not as the main structure of the test.

## Quick Comparison

| Double | What it does | Good use | Avoid when |
|--------|---------------|----------|------------|
| **Dummy** | Placeholder value that is never used | Satisfying an argument you do not care about | You need real behavior or data |
| **Stub** | Returns pre-programmed answers | Driving a branch with known inputs | You need to observe side effects |
| **Spy** | Records what happened during the test | Checking emitted messages or call metadata after execution | You want strict up-front expectations |
| **Mock** | Fails when expected interactions do not happen | Rare verification of unavoidable outbound commands | You are testing internal collaboration |
| **Fake** | Lightweight working implementation | In-memory replacement for unstable external boundaries | The fake becomes harder to trust than the real dependency |

## Selection Order

Prefer this order unless you have a strong reason not to:

1. Real dependency
2. Fake
3. Stub
4. Spy
5. Mock

## Why this order?

- **Real dependencies** preserve realistic collaboration when they are fast and deterministic.
- **Fakes** preserve behavior while isolating unstable infrastructure.
- **Stubs** are fine for fixed answers.
- **Spies** help observe effects after execution.
- **Mocks** create the most coupling to implementation details and should be the rarest choice.

## Good Uses of Mocks

- Verifying an unavoidable outbound command to an unmanaged dependency
- Guarding a side effect that cannot be observed through state or a fake

## Bad Uses of Mocks

- Mocking domain entities or value objects
- Mocking the system under test
- Mocking internal collaborators just because a framework makes it easy
- Verifying query calls or harmless internal sequencing

## Fake Example

```python
class InMemoryEmailGateway:
    def __init__(self) -> None:
        self.sent_messages: list[EmailMessage] = []

    def send(self, email: EmailMessage) -> None:
        self.sent_messages.append(email)
```

```python
notification_scenario_driver = NotificationScenarioDriver(gateway=InMemoryEmailGateway())

notification_scenario_driver.when_a_welcome_email_is_sent_to(user)

notification_scenario_driver.then_the_email_was_sent(expected_email)
```

This is usually more stable than a mock with strict call expectations.

## See also

- [Main skill](../SKILL.md)
- [Cheat sheet](CHEATSHEET.md)
- [Service example](../examples/SERVICE-EXAMPLE.md)
- [DSL-driven testing](../../dsl-driven-testing/SKILL.md)
