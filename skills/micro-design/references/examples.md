# Before / after examples

All examples come from the eMush codebase (PHP 8 / Symfony / Doctrine). Transpose idioms to the current language.

## Contents
1. Naming an input guard
2. Replacing "what" comments with names
3. Placing a behaviour on its owner
4. Local extraction instead of feeding a god object
5. Value object holding values only
6. Negative predicates
7. Named arguments
8. Magic numbers
9. Code that should be left alone
10. General knowledge needs no name
11. Contextual projection and a value object crossing aggregates
12. Varying rules as a composed strategy
13. Invariant over a set of objects
14. A recipe hiding two responsibilities
15. Same knowledge duplicated
16. Units and formulas instead of comments

---

## 1. Naming an input guard

Before:
```php
public function __invoke(UpdateUserStatisticCommand $command): void
{
    if ($command->statisticName === StatisticEnum::NULL || $command->count <= 0) {
        return;
    }
    // ...
}
```

After:
```php
public function __invoke(UpdateUserStatisticCommand $command): void
{
    if ($this->doesNotUpdate($command)) {
        return;
    }
    // ...
}

private function doesNotUpdate(UpdateUserStatisticCommand $command): bool
{
    return $command->statisticName === StatisticEnum::NULL || $command->count <= 0;
}
```
Why: the guard encodes domain knowledge ("a NULL statistic or a non-positive count updates nothing") that the raw expression leaves the reader to infer. `isNoOp` would be a technical name.

## 2. Replacing "what" comments with names

Before:
```php
public function onTitleRemoved(PlayerEvent $event): void
{
    // Don't attribute if it was not Commander
    if ($event->doesNotHaveTag(TitleEnum::COMMANDER)) {
        return;
    }

    // Don't attribute if the ship is going with living players
    if (!$event->getDaedalus()->getDaedalusInfo()->isDaedalusFinished()
        && $event->getDaedalus()->getAlivePlayers()->count() > 1) {
        return;
    }

    // Don't attribute if player did not die
    if (!array_intersect($event->getTags(), EndCauseEnum::getDeathEndCauses()->toArray())) {
        return;
    }

    $this->updatePlayerStatisticService->execute(
        player: $event->getPlayer(),
        statisticName: StatisticEnum::COMMANDER_SHOULD_GO_LAST,
    );
}
```

After:
```php
public function onTitleRemoved(PlayerEvent $event): void
{
    if ($event->doesNotHaveTag(TitleEnum::COMMANDER)) {
        return;
    }

    if ($this->shipLeavesWithLivingCrew($event)) {
        return;
    }

    if ($this->playerSurvived($event)) {
        return;
    }

    $this->updatePlayerStatisticService->execute(
        player: $event->getPlayer(),
        statisticName: StatisticEnum::COMMANDER_SHOULD_GO_LAST,
    );
}

private function shipLeavesWithLivingCrew(PlayerEvent $event): bool
{
    $daedalus = $event->getDaedalus();

    return !$daedalus->getDaedalusInfo()->isDaedalusFinished() && $daedalus->getAlivePlayers()->count() > 1;
}

private function playerSurvived(PlayerEvent $event): bool
{
    return !array_intersect($event->getTags(), EndCauseEnum::getDeathEndCauses()->toArray());
}
```
Why: the first condition is already a single domain call, so its comment is simply deleted. The other two comments were names waiting to happen.

## 3. Placing a behaviour on its owner

Before (in `DecodeRebelSignalService`):
```php
if ($rebelBase->getConfig()->getMoral() < 1) {
    return;
}
// ...
new PlayerVariableEvent($player, PlayerVariableEnum::MORAL_POINT, $rebelBase->getConfig()->getMoral(), $tags, new \DateTime());
```

After:
```php
// RebelBase — small, cohesive, already delegates getName(), getContactOrder()... to its config
public function givesMorale(): bool
{
    return $this->moraleBoost() >= 1;
}

public function moraleBoost(): int
{
    return $this->rebelBaseConfig->getMoral();
}

// DecodeRebelSignalService
if (!$rebelBase->givesMorale()) {
    return;
}
// ...
new PlayerVariableEvent($player, PlayerVariableEnum::MORAL_POINT, $rebelBase->moraleBoost(), $tags, new \DateTime());
```
Why: the information belongs to the rebel base, the object is small, delegation to its config is already its style, and "does this base give morale?" is an obvious question to ask it. There is no exact antonym for `givesMorale`, so the caller uses `!`.

## 4. Local extraction instead of feeding a god object

`Player` has ~1650 lines and ~190 public methods.

Before:
```php
$nbSleptCycles = $player->getPlayerInfo()->getStatistics()->getSleptCycles();
if ($closedPlayer->getDayDeath() < 7 || $nbSleptCycles > 0) {
    return;
}
```

After (in the subscriber):
```php
private const MINIMUM_SURVIVED_DAYS = 7;

if ($this->diedTooEarly($player) || $this->hasPlayerSlept($player)) {
    return;
}

private function hasPlayerSlept(Player $player): bool
{
    return $player->getPlayerInfo()->getStatistics()->getSleptCycles() > 0;
}

private function diedTooEarly(Player $player): bool
{
    return $player->getPlayerInfo()->getClosedPlayer()->getDayDeath() < self::MINIMUM_SURVIVED_DAYS;
}
```
Why: the data sits two sub-objects deep, one feature uses it, and the target is a god object. Adding `Player::hasSlept()` would fail the "obvious" test.

## 5. Value object holding values only

Before: `GameDate` holds a `Daedalus` entity and reaches through it for configuration.
```php
final class GameDate
{
    public function __construct(private Daedalus $daedalus, private int $day, private int $cycle) { /* ... */ }

    private function toMinutes(): int
    {
        return $this->toElapsedCycles() * $this->daedalus->getGameConfig()->getDaedalusConfig()->getCycleLength();
    }
}
```

After: the configuration it needs becomes a value object of its own.
```php
final readonly class GameCalendar
{
    public function __construct(public int $cyclesPerDay, public int $cycleLengthInMinutes)
    {
        if ($cyclesPerDay < 1 || $cycleLengthInMinutes < 1) {
            throw new \InvalidArgumentException('Calendar values must be positive');
        }
    }

    public function dateAt(int $day, int $cycle): GameDate
    {
        return GameDate::fromDayAndCycle($day, $cycle, $this);
    }
}

final readonly class GameDate
{
    private function __construct(private int $elapsedCycles, private GameCalendar $calendar) {}

    public static function fromDayAndCycle(int $day, int $cycle, GameCalendar $calendar): self
    {
        if ($day < 1) {
            throw new \InvalidArgumentException('Day must be greater than 0');
        }

        return new self(max(1, ($day - 1) * $calendar->cyclesPerDay + $cycle), $calendar);
    }

    public function day(): int
    {
        return intdiv($this->elapsedCycles - 1, $this->calendar->cyclesPerDay) + 1;
    }

    public function cycle(): int
    {
        return (($this->elapsedCycles - 1) % $this->calendar->cyclesPerDay) + 1;
    }

    public function cyclesAgo(int $cycles): self
    {
        return new self(max(1, $this->elapsedCycles - $cycles), $this->calendar);
    }

    public function previous(): self
    {
        return $this->cyclesAgo(1);
    }

    public function lessThanOrEqual(self $other): bool
    {
        return $this->elapsedCycles <= $other->elapsedCycles;
    }

    public function moreThanOrEqualMinutes(int $minutes): bool
    {
        return $this->elapsedCycles * $this->calendar->cycleLengthInMinutes >= $minutes;
    }
}
```
Why: a value object holds values only. Storing `elapsedCycles` internally makes navigation and comparison trivial. Since `GameCalendar` would be shared by `Daedalus` and every caller, introducing it is a **proposal** in a real refactoring (shared abstraction), not an unilateral change. Callers then chain freely: `$date->cyclesAgo($n)->lessThanOrEqual($logDate)` is a fluent API, not a Demeter violation.

## 6. Negative predicates

Good: exact antonyms from the domain.
```php
public function isMush(): bool { /* ... */ }
public function isHuman(): bool { return !$this->isMush(); }
```

Avoid: invented negatives.
```php
public function isNotEstablished(): bool { return $this->isEstablished === false; }
```
Prefer `!$linkWithSol->isEstablished()` at the call site, usually inside a named predicate:
```php
private function shouldAbortKillingLink(int $successRate, LinkWithSol $linkWithSol): bool
{
    return $this->d100Roll->isAFailure($successRate) || !$linkWithSol->isEstablished();
}
```
Existing `isNotX()` methods in a codebase are not renamed spontaneously; list them as remaining violations.

## 7. Named arguments

```php
// Literal → named
$this->killLinkWithSol->execute($daedalusId, successRate: 100);

// Self-named variables → not named, even in a constructor
new LinkWithSol($daedalusId, $strength, $isEstablished);

// Constructor with non-self-describing values → named
new Statistic(config: $config, userId: $command->userId, count: $command->count);

// Same-type neighbours → named
$this->findByNameUserIdAndClosedDaedalusIdOrNull(name: $name, userId: $userId, closedDaedalusId: $closedDaedalusId);
```

## 8. Magic numbers

```php
// Extract: not self-explanatory
$numberOfCyclesToCheck = $player->hasSkill(SkillEnum::TRACKER) ? 16 : 8;
// →
$numberOfCyclesToCheck = $player->hasSkill(SkillEnum::TRACKER) ? self::TRACKER_LOOKBACK_CYCLES : self::DEFAULT_LOOKBACK_CYCLES;

// Keep: bounds of an encapsulated invariant, self-explanatory in context
$this->value = max(0, min(100, $value)); // LinkStrength
return $this->signal >= 100;             // RebelBase::isDecoded()
```

## 9. Code that should be left alone
10. General knowledge needs no name
11. Contextual projection and a value object crossing aggregates
12. Varying rules as a composed strategy
13. Invariant over a set of objects
14. A recipe hiding two responsibilities
15. Same knowledge duplicated
16. Units and formulas instead of comments

```php
final readonly class KillLinkWithSolService
{
    public function execute(int $daedalusId, int $successRate = 100, array $tags = []): void
    {
        $linkWithSol = $this->linkWithSolRepository->findByDaedalusIdOrThrow($daedalusId);

        if ($this->shouldAbortKillingLink($successRate, $linkWithSol)) {
            return;
        }

        $linkWithSol->unestablish();
        $this->linkWithSolRepository->save($linkWithSol);

        $this->eventService->callEvent(
            event: new LinkWithSolKilledEvent($daedalusId, tags: $tags),
            name: LinkWithSolKilledEvent::class
        );
    }
}
```
The method reads as a recipe, its condition is named, and its entity exposes verbs. The only remaining point (`isNotEstablished()` inside `shouldAbortKillingLink`) is a minor naming issue to list, not a reason to rewrite the class.

---

## 10. General knowledge needs no name

Keep:
```php
public function incrementCount(int $amount): void
{
    if ($amount <= 0) {
        throw new \InvalidArgumentException('Amount must be positive');
    }

    $this->count += $amount;
}
```
Do not turn it into `if ($this->isNotAPositiveAmount($amount))`: "must be positive" is general knowledge, not domain knowledge. Compare with `isFirstContact($rebelBase)`, where the name tells the reader something `=== null` does not.

## 11. Contextual projection and a value object crossing aggregates

Rules: a lesson holds at most 12 participants; a user cannot book two overlapping lessons.

Avoid: the central actor drives everything.
```php
$user->reserve($lesson, $now); // User checks overlaps, then mutates Lesson
```

Prefer: each invariant in its own aggregate, the user represented by its booking-context projection, and a value object for what crosses aggregates.
```php
final readonly class BookLessonService
{
    public function execute(int $userId, int $lessonId): void
    {
        $lesson = $this->lessonRepository->findByIdOrThrow($lessonId);
        $learner = $this->learnerRepository->findByUserIdOrThrow($userId);

        $learner->book($lesson->slot()); // LessonSlot(lessonId, schedule); throws if overlapping
        $lesson->takeSeat($userId);      // throws if full

        $this->learnerRepository->save($learner);
        $this->lessonRepository->save($lesson);
    }
}
```
Why: `User` does not grow; the no-overlap invariant gets an owner (`Learner`); `book($lesson->schedule(), $lessonId)` would leak two detached primitives with no obvious order, while `LessonSlot` says what is booked.

## 12. Varying rules as a composed strategy

Avoid: the cart knows every coupon type.
```php
$this->discount = match ($coupon->type()) {
    CouponType::PERCENTAGE => /* ... */,
    CouponType::CATEGORY => /* ... */,
};
```

Prefer: the coupon carries its rule as a composable value object; the cart receives a `Discount` and keeps its own invariants.
```php
final class Coupon
{
    private DiscountRule $rule; // PercentageRule, CategoryRule, MinimumTotal(inner: ...)

    public function discountFor(CartSummary $cart, \DateTimeImmutable $now): Discount
    {
        if ($this->isExpiredAt($now)) {
            throw new CouponExpired();
        }

        return $this->rule->discountFor($cart);
    }
}

// use case
$cart->applyDiscount($coupon->discountFor($cart->summary(), $this->clock->now()));
```
Why: the rule is intrinsic to the coupon and varies by kind. Reading cart data through a `CartSummary` value object is legitimate "envy" for a strategy. Use a strategy service only if the rule itself must decide what infrastructure data to fetch.

## 13. Invariant over a set of objects

Rule: a customer redeems a given coupon at most once.
```php
#[ORM\UniqueConstraint(columns: ['customer_id', 'coupon_id'])]
final class Redemption
{
    public function __construct(
        private int $customerId,
        private int $couponId,
        private \DateTimeImmutable $redeemedAt,
    ) {}
}
```
Why: no single aggregate can enforce uniqueness across all redemptions without loading them all (a `Coupon` used 100,000 times would load 100,000 ids). A declarative constraint is simple and holds under concurrency; translate the constraint violation into a domain exception (`CouponAlreadyRedeemed`) at the boundary.

---

## 14. A recipe hiding two responsibilities

`DecodeRebelSignalService` reads well, but its steps use different collaborators and change for different reasons (decoding rules vs rewards):
```php
public function execute(RebelBase $rebelBase, int $progress, Player $author, array $tags = []): void
{
    $this->increaseDecodingProgress($rebelBase, $progress);

    if ($rebelBase->isDecoded()) {
        $daedalus = $this->daedalusRepository->findByIdOrThrow($rebelBase->getDaedalusId());
        $this->createRebelBaseModifiersForDaedalus($rebelBase, $daedalus, $tags);
        $this->createRebelBaseStatusForDaedalus($rebelBase, $daedalus, $tags);
        $this->boostMoral($rebelBase, $daedalus, $tags);
        $this->endRebelBaseContact($rebelBase);
        $this->eventService->callEvent(/* RebelBaseDecodedEvent */);
    }
}
```
What the agent does when touching it: keep the recipe, then **propose**:
- move "a decoded base ends its contact" into `RebelBase::increaseDecodingProgress()` (intrinsic);
- keep `DecodeRebelSignalService` for decoding (2 dependencies instead of 5);
- extract `ApplyRebelBaseRewardsService` (modifiers, status, morale), called explicitly by the use case or triggered by an event — a macro choice for the user.

## 15. Same knowledge duplicated

This `match` appears in `UpdateUserStatisticCommandHandler` (on a `Statistic`) and in `UpdatePlayerStatisticService` (on a `PendingStatistic`):
```php
match ($config->getStrategy()) {
    StatisticStrategyEnum::MAX => $statistic->updateIfSuperior($count),
    StatisticStrategyEnum::INCREMENT => $statistic->incrementCount($count),
    default => throw new \LogicException('Undefined statistic strategy.'),
};
```
Same source of truth, same reason to change (a new `MIN` strategy touches both): it is one piece of knowledge. Since the copies live in two files, the agent **proposes** a single home, for instance a method on the strategy or a shared interface for both statistic types, instead of creating it silently.

## 16. Units and formulas instead of comments

```php
// Before
private const LOBBY_TIME_LIMIT = 2880; // minutes (48h)

// After
private function lobbyTimeLimit(): Duration
{
    return Duration::hours(48);
}
```
The unit lives in the type, not in a comment or a constant name, so it can never be misread.

```php
// Before
// Days and cycles are 1-indexed: day 1 cycle 1 is elapsed cycle 1.
return max(1, ($this->day - 1) * $cyclesPerDay + $this->cycle);

// After
return $this->toElapsedCycles();
```
