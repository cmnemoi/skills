# Incremental Refactoring — Parallel Change & Branch by Abstraction

These are **delivery/migration strategies**, not replacements for small refactoring operations such as Extract Method or Move Method. Use them when a single atomic change would break callers, require a long-lived branch, or make integration/rollback too risky.

## Decision Guide

| Problem | Technique | What coexists temporarily? |
|---|---|---|
| Break an API, method signature, event, or database schema | **Parallel Change (Expand → Migrate → Contract)** | Old and new **contracts** |
| Replace a large subsystem/algorithm/vendor while callers stay stable | **Branch by Abstraction** | Old and new **implementations** |
| Discover unknown blockers and hidden dependencies | **Mikado Method** | A graph of prerequisite changes |
| Find a point where dependencies can be substituted for testing | **Seam** (Feathers) | Test and production collaborators |
| Small change with all callers under control | **Direct refactoring** | No coexistence needed |

Parallel Change may be used *inside* Branch by Abstraction to introduce the new boundary safely. Neither technique mandates feature flags or new interfaces when simpler means are sufficient.

## How this fits existing techniques

1. **Characterization tests** capture observable current behavior before risky edits. Include relevant failures and side effects. Avoid testing implementation details.
2. **Seams / dependency breaking** allow the old implementation to be isolated, tested, and eventually substituted. Favor explicit dependency injection at a composition root; use the smallest stable boundary.
3. **Mikado Method** is for exploration: attempt the change, discover prerequisites, revert the failed exploration, then work from the leaves of the dependency graph. Some leaves may be Parallel Changes or prerequisite seams.
4. **Parallel Change / Branch by Abstraction** structure the *sequence of integrated changes*, allowing each intermediate state to build, pass tests, and preserve current supported behavior.
5. **Extract Method, Move Method, Extract Class, polymorphism, etc.** are the *mechanical refactorings* performed within those steps. Avoid extracting an interface merely because it is named in the pattern.
6. **Finish the migration** by deleting obsolete compatibility paths, old implementations, temporary toggles, and unused adapters. The final design should not carry scaffolding without a reason.

## Parallel Change: Expand → Migrate → Contract

**When**: old and new consumers cannot all be updated safely at once, or the contract has independently deployed consumers.

**Expand** — Introduce a compatible new contract. Keep old callers running; implement the old entry point in terms of the new one when possible to avoid divergent business logic.

~~~python
class Users:
    # Old entry point stays usable during migration.
    def get_user(self, raw_id: str) -> User:
        return self.find_user(UserId(raw_id))

    def find_user(self, user_id: UserId) -> User:
        return self._store.load(user_id.value)
~~~

**Migrate** — Move callers gradually from `get_user(raw_id)` to `find_user(UserId(raw_id))`. Verify both paths and take small, reviewable, integrated steps.

**Contract** — Remove `get_user` only after checking that no supported caller (including externally deployed versions) still needs it.

**Important for data/contracts**:
- A database migration requires sequencing compatible schema/application versions; backfill, dual reads/writes, data consistency, and rollback need explicit treatment where applicable. Do not imply dual writes are always safe or mandatory.
- Events and HTTP APIs may have consumers outside the repository. Searchability of local callers is not proof of zero consumers.
- State the removal criterion and owner at the expand step. "Temporary" compatibility otherwise becomes permanent technical debt.

**Avoid when**: the change is truly local, all callers can be changed atomically, or compatibility is not required.

## Branch by Abstraction: Old → Seam → New → Switch → Remove Old

**When**: a *supplier implementation* needs a substantial replacement over multiple integrations while its callers should stay stable.

1. **Protect behavior** with characterization/contract tests around the supplier and its callers.
2. **Introduce a small abstraction** around the existing supplier. Make all callers use it **without changing behavior**. Validate and integrate this as an independent step.
3. **Build the replacement** behind the same boundary. Keep the old implementation as the default until the new one is ready.
4. **Validate and switch** at the composition root. Optional routing, canary rollout, or shadow comparison may help; only compare side-effectful operations using a safe strategy (never casually execute payments/writes twice).
5. **Delete the obsolete supplier and transition logic**. Keep the abstraction only if it still provides real value.

~~~python
from typing import Protocol

class SearchGateway(Protocol):
    def find(self, query: str) -> list[Result]: ...

class SearchService:
    def __init__(self, search: SearchGateway):
        self._search = search

    def search(self, query: str) -> list[Result]:
        return self._search.find(query)

# A composition root selects LegacySearchGateway initially,
# then NewSearchGateway after verification.
# SearchService remains unchanged while the implementation is migrated.
~~~

**Quality gates**:
- Each intermediate commit/PR is independently understandable, testable, and safely deployable with the existing implementation.
- Behavior contracts matter more than interface shape; include errors, ordering, performance constraints, and side effects as relevant.
- Prefer construction-time selection/DI to scattering `if new_backend` through domain code.
- A feature flag is **optional**, not synonymous with Branch by Abstraction; if used, define its retirement.
- Don't call unreferenced incomplete code a valid "slice" unless it serves a planned integration step; build the new implementation behind the seam and use contract tests before switching.
- For a tiny local replacement, direct refactoring is simpler.

## Comparing to Other Migration Patterns

- **Parallel Change** migrates the *consumer contract*. **Branch by Abstraction** migrates the *supplier implementation*. Either can appear within the other.
- **Strangler Fig** redirects traffic between whole applications/services (often at an HTTP/routing boundary), whereas Branch by Abstraction usually changes dependencies within the codebase or service.
- **Mikado** helps *discover and order prerequisites*, but does not itself provide runtime coexistence.
- **Feature flags** control activation; they are a delivery tool, not a required precondition for any of these techniques.

## Agent Checklist

Before introducing a migration pattern, explicitly answer:
1. What must change: contract, implementation, or both?
2. Can all affected callers change atomically? Is there an independently deployed consumer?
3. Which existing tests establish the observable behavior? What seam is needed?
4. What is the **first independently safe integration step**?
5. What makes each intermediate state correct and releasable?
6. How will old/new behavior be compared safely if parity matters?
7. What is the rollback approach, and when can the old path be removed?
8. Which added abstraction/compatibility code is temporary? Who will delete it?

## Sources

- Martin Fowler — [Parallel Change](https://martinfowler.com/bliki/ParallelChange.html)
- Martin Fowler — [Branch By Abstraction](https://martinfowler.com/bliki/BranchByAbstraction.html)
- Paul Hammant — [Branch by Abstraction (Trunk-Based Development)](https://trunkbaseddevelopment.com/branch-by-abstraction/)
- Sam Newman — [Branch by Abstraction](https://samnewman.io/patterns/architectural/branch-by-abstraction/)
- Martin Fowler — [Feature Toggles](https://martinfowler.com/articles/feature-toggles.html)
