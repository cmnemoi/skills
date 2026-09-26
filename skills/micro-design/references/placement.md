# Placing responsibilities

Read this when deciding where a behaviour, predicate or rule should live. These heuristics are still being explored: weigh them against the existing code, and choose what best reveals the domain with the best coupling/cohesion trade-off.

## Contents
- The "obvious" test
- Intrinsic property vs policy
- Central actors
- Invariants
- Rules that vary
- Law of Demeter vs fluent APIs

## The "obvious" test

Put a behaviour on an object only when all three hold:
- it concerns the object's **direct state**, not a distant sub-object or another aggregate reached through it;
- it is a **question one would naturally ask that object** in the domain's vocabulary ("is this player alive?");
- it is **transversal**: it keeps the same meaning across several features.

Examples:
- `$player->hasSkill($skill)` → on the object.
- `$player->getPlayerInfo()->getStatistics()->getSleptCycles() > 0`, used by one achievement → local `hasPlayerSlept($player)`.
- "Is the game finished?" asked through a player → local; it is a question about the game.
- A predicate spanning two objects that belongs to neither → local.
- Parameterise rather than multiply: `isInPlace($room)`, not `isInLaboratory()`, `isInBridge()`…

## Intrinsic property vs policy

Owning the data is not enough. A predicate can read only an object's state and still belong to one feature's rules.
- **Intrinsic** (on the object): `isAlive()`, `increaseDecodingProgress()`, true across the whole domain.
- **Policy** (in the use case, the module or a dedicated policy object): `hasHighMorale()` with a threshold one feature defines, `canApplyForPsychologicalSupport()`: rules someone applies to the object.
- The tell is **the reason to change**: if the rule changes when a feature's rules change, not when the object changes, it does not belong on the object. This is also why a Strategy may legitimately "envy" another object's data (Feature Envy is not the last word).

## Central actors

`User`, `Player`, `Account`, `Customer` attract behaviour simply because they are the grammatical subject of many actions (`$user->reserve($lesson)`).
- **Never feed a god object.**
- **Try inverting the sentence** (`$lesson->reserveFor($userId)`) as a design question, not a rule: it may reveal another owner.
- **Look for a contextual projection** of the actor, such as a `Learner` or `Agenda` in the booking context, rather than `User` doing everything.

## Invariants

- **Internal to one object** → that aggregate (a lesson's capacity, one discount per cart).
- **No owner** → a missing concept (no overlapping bookings → `Learner`/`Agenda`).
- **Over a set of objects** (a customer redeems a coupon at most once) → a declarative infrastructure constraint (unique index, optimistic locking). No single aggregate can enforce it without loading the whole set.

## Rules that vary

- Rules that vary by kind (coupon types) → a **Strategy composed as value objects and carried by the object** (`Coupon` holds a `DiscountRule`; rules can wrap each other, like a minimum total around a percentage).
- A rule needing infrastructure → first **fetch the data in the use case and pass it as a value**, so the rule stays pure. Use a strategy as a service only when the rule itself must decide what to fetch.

## Law of Demeter vs fluent APIs

- **Healthy**: chaining on one abstraction, such as fluent APIs, builders and operations returning the same type (`$date->cyclesAgo($n)->lessThanOrEqual($other)`).
- **Suspect**: each call descends into a different type (`$order->customer()->preferences()->notifications()`), immutable or not. The caller depends on a structure it has no business knowing.
- **Red flag**: reaching another aggregate (`$player->getDaedalus()`).
- An inherited object graph is macro design: tolerate it in the code you touch.
- Chain length is not a criterion.
