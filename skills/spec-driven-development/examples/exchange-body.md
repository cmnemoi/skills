# Example: Exchange Body

This example shows a realistic feature spec with non-trivial side effects, explicit behavior IDs, acceptance criteria, and `@spec` traceability across tests and code.

---

## Spec

```md
# Exchange Body

## Why

This action lets a Mush player take control of another human player's body.
The change is risky because it swaps ownership, moves Mush identity, and must preserve several character-bound invariants.

## Scope

This spec covers action visibility, execution preconditions, identity changes, invariant preservation, and emitted side effects.

## Rules

### Visibility rules

`{#action.exchange-body::visibility-rules}`

The action is visible only when the source player can target another player in range and the target is not Mush.

### Execution preconditions

`{#action.exchange-body::execution-preconditions}`

The action is executable only when the target has at least 1 spore and the resulting Mush player has not already used the exchange.

### Body ownership is swapped

`{#action.exchange-body::swaps-player-ownership}`

When the action succeeds, the users attached to the source and target players are swapped.

### Mush identity moves to target body

`{#action.exchange-body::moves-mush-identity}`

When the action succeeds, the target body becomes Mush and the source body becomes human.

### Spores are removed from both bodies

`{#action.exchange-body::removes-spores}`

When the action succeeds, spores are removed from both source and target bodies.

### Human skills are reset

`{#action.exchange-body::resets-human-skills}`

When the action succeeds, human skills are removed from both involved bodies.

### Mush-only state follows the Mush

`{#action.exchange-body::transfers-mush-state}`

When the action succeeds, Mush-specific skills and transferable Mush statuses move with the Mush identity.

### Character-bound values stay on bodies

`{#action.exchange-body::preserves-character-bound-values}`

When the action succeeds, character-bound values remain attached to their body.

### Notifications and side effects are emitted

`{#action.exchange-body::emits-side-effects}`

When the action succeeds, the source player receives the "became human" notification, the target player receives the "became Mush" notification, and room side effects are emitted when required.

## Acceptance criteria

- Given a target without spores, when the source tries to exchange bodies, then the action is not executable.
- Given a valid source and target, when the exchange succeeds, then the two users are swapped.
- Given a valid source and target, when the exchange succeeds, then Mush identity moves to the target body.
- Given an exchange where character-bound values differ on both bodies, when the exchange succeeds, then those values remain attached to their original bodies.
- Given room equipment with a public side effect, when the exchange succeeds, then that side effect is emitted.

## Out of scope

- balance tuning for spores or costs
- unrelated Mush actions
```

---

## Tests

```php
/** @spec action.exchange-body::execution-preconditions */
public function shouldNotBeExecutableIfTargetPlayerDoesNotHaveSpores(FunctionalTester $I): void
{
    $this->givenTargetPlayerHasSpores(0);

    $this->whenSourceTriesToExchangeBodyWithTarget();

    $this->thenActionShouldNotBeExecutable(
        message: ActionImpossibleCauseEnum::TRANSFER_NO_SPORE,
        I: $I,
    );
}

/** @spec action.exchange-body::swaps-player-ownership */
public function shouldMakeUsersExchangePlayers(FunctionalTester $I): void
{
    $this->whenSourceExchangesBodyWithTarget();

    $this->thenPlayerUsersAreSwapped($I);
}

/** @spec action.exchange-body::moves-mush-identity */
public function shouldMakeTargetPlayerMush(FunctionalTester $I): void
{
    $this->whenSourceExchangesBodyWithTarget();

    $this->thenTargetPlayerIsMush($I);
}

/** @spec action.exchange-body::preserves-character-bound-values */
public function shouldTriumphStayIntact(FunctionalTester $I): void
{
    $this->givenSourcePlayerHasTriumph(2);
    $this->givenTargetPlayerHasTriumph(3);

    $this->whenSourceExchangesBodyWithTarget();

    $this->thenSourcePlayerShouldHaveTriumph(2, $I);
    $this->thenTargetPlayerShouldHaveTriumph(3, $I);
}

/** @spec action.exchange-body::emits-side-effects */
public function shouldMakeMycoAlarmRing(FunctionalTester $I): void
{
    $this->givenSourcePlayerHasSpores(1);
    $this->givenMycoAlarmInRoom();

    $this->whenSourceExchangesBodyWithTarget();

    $this->thenMycoAlarmPrintsPublicLog($I);
}
```

---

## Implementation

```php
/** @spec action.exchange-body::swaps-player-ownership */
private function exchangePlayerUsers(): void
{
    $player = $this->player;
    $target = $this->target();

    $playerUser = $player->getUser();
    $targetUser = $target->getUser();

    $playerInfo = $player->getPlayerInfo();
    $targetInfo = $target->getPlayerInfo();

    $playerInfo->updateUser($targetUser);
    $targetInfo->updateUser($playerUser);

    $this->playerRepository->save($player);
    $this->playerRepository->save($target);
}

/** @spec action.exchange-body::moves-mush-identity */
private function makeTargetMush(): void
{
    $target = $this->target();

    $event = new PlayerEvent(
        $target,
        $this->getTags(),
        new \DateTime(),
    );
    $event->setAuthor($this->player);
    $this->eventService->callEvent($event, PlayerEvent::CONVERSION_PLAYER);
}

/** @spec action.exchange-body::transfers-mush-state */
private function transferMushSkills(): void
{
    $this->player
        ->getMushSkills()
        ->map(fn (Skill $skill) => $this->addSkillToPlayer->execute($skill->getName(), $this->target()));
}

/** @spec action.exchange-body::emits-side-effects */
private function createTargetNotification(): void
{
    $this->updatePlayerNotification->execute($this->target(), PlayerNotificationEnum::EXCHANGE_BODY_MUSH);
}
```

---

## Why This Example Matters

This feature is a good SDD example because the hard part is not just "swap two players". The real difficulty is making explicit:

- what follows Mush identity
- what remains attached to the body
- what is removed or reset
- what side effects must still happen

Without a spec, these rules get scattered across tests, listeners, and business code. With SDD, the rules become visible, testable, and reviewable.

---

## Reusable Lessons

Apply this pattern when a feature has:

- strong preconditions
- identity transfer or ownership changes
- important invariants
- side effects across multiple subsystems
- high regression risk

In those cases, a short spec with behavior IDs is usually more valuable than a large design document.
