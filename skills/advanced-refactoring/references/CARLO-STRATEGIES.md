# Carlo's Legacy Code Strategies

Based on Nicolas Carlo's work at [Understand Legacy Code](https://understandlegacycode.com/).

## Philosophy: Make It Better, Not Perfect

> Legacy code improvement is a marathon, not a sprint. 1% improvement every day compounds over time.

Key principles:
- **Refactor as you go** — don't treat refactoring as separate from feature work
- **Write missing tests** — just enough to cover the code you're changing
- **Practical over idealistic** — use Sprout/Wrap when under time pressure

---

## The Sprout Methods

### When to Use Sprout Method

- Need to add new behavior to existing method
- Can't get the existing code under test easily
- New code is clearly separable from existing code

### Sprout Method Pattern

```javascript
// Original code
class TransactionGate {
    postEntries(entries) {
        for (let entry of entries) {
            entry.postDate();
        }
        transactionBundle.getListManager().add(entries);
    }
}

// Using Sprout Method - add deduplication
class TransactionGate {
    // 1. Create new method elsewhere (sprout) - testable!
    uniqueEntries(entries) {
        // Some clever logic to dedupe, fully tested
    }

    // 2. Call from legacy code with minimal change
    postEntries(entries) {
        const uniqueEntries = this.uniqueEntries(entries);
        for (let entry of uniqueEntries) {
            entry.postDate();
        }
        transactionBundle.getListManager().add(uniqueEntries);
    }
}
```

### When to Use Sprout Class

- New functionality is substantial enough for its own class
- The new code needs its own dependencies
- You want cleaner separation of concerns

```javascript
// Instead of adding to existing class, create new class
class LoyaltyPointProcessor {
    process(order) {
        // New implementation - fully testable!
    }
}

class OrderProcessor {
    private loyaltyProcessor = new LoyaltyPointProcessor();
    
    process(order) {
        // existing logic...
        loyaltyProcessor.process(order);
    }
}
```

### Advantages & Disadvantages

| | Sprout Method | Sprout Class |
|---|---|---|
| **Pros** | Minimal change, separates new from old | Better separation, new class can have own deps |
| **Cons** | Doesn't improve existing code | More code, adds to the system |

---

## The Wrap Methods

### When to Use Wrap Method

- The change should happen *before* or *after* existing code
- Want to add behavior to existing method calls
- Can't easily modify the original method

### Wrap Method Pattern

```javascript
// Step 1: Rename old method
class OrderProcessor {
    processOrderThatIsUnique(order) { /* original */ }
    
    // Step 2 & 3: New method with original name, wrapping old
    processOrder(order) {
        // New logic BEFORE
        validate(order);
        
        // Original logic
        processOrderThatIsUnique(order);
        
        // New logic AFTER
        logCompletion(order);
    }
}
```

### When to Use Wrap Class

- Want to wrap behavior of entire class
- Adding cross-cutting concern (logging, caching, etc.)
- Need to add behavior without modifying original

```javascript
// Decorator pattern
class OrderProcessorDecorator {
    constructor(wrapped) {
        this.wrapped = wrapped;
    }
    
    execute(order) {
        // BEFORE
        this.logBefore(order);
        this.validate(order);
        
        // Original
        const result = this.wrapped.execute(order);
        
        // AFTER
        this.logAfter(result);
        
        return result;
    }
}
```

---

## Approval Testing (Golden Master)

For quick safety nets without deep understanding of code.

### Process

1. **Generate a snapshot** — Run code, capture output to file
2. **Find uncovered paths** — Use coverage to discover input combinations
3. **Introduce mutations** — Break code to verify tests catch regressions
4. **Refactor** — Now with safety net

### Tools

| Language | Tool |
|----------|------|
| Java | [ApprovalTests](https://approvaltests.com/) |
| C# | ApprovalTests.NET |
| Python | [py-spy](https://github.com/benfred/py-spy), [pytest-snapshottest](https://pypi.org/project/pytest-snapshottest/) |
| JavaScript | [Jest Snapshot](https://jestjs.io/docs/snapshot-testing), [ApprovalTests.js](https://github.com/approvals/approvaltests-js) |

### Example

```java
// Using ApprovalTests
@Test
void shouldGenerateCorrectReport() {
    ReportGenerator gen = new ReportGenerator();
    String output = gen.generateMonthlyReport();
    
    // Approve written to file: report.approved.txt
    Approvals.verify(output);
}
```

---

## The Mikado Method

A structured approach for dealing with unknown scope.

### Steps

1. **Write your goal** on paper (circle it twice)
2. **Set a 10-15 minute timer**
3. **Try to achieve the goal** within the time limit
4. **When you fail** (which is expected):
   - Stop and analyze what you discovered
   - Write down subtasks needed
   - **Revert all changes** (critical!)
   - Connect subtasks to your main goal
5. **Pick a subtask** and repeat
6. **When you succeed** on a subtask, commit and celebrate

### Visual

```
                    [Goal]
                       │
                  ┌────┴────┐
                  │ subtask │
                  └────┬────┘
                  │
            ┌─────┴─────┐
        subtask A   subtask B
```

### Key Principle

> Small commits, frequent checkpoints, always revert when stuck

---

## Hotspot Analysis

Find where to focus refactoring effort.

### Process

1. **Extract churn data** — Git history, most frequently changed files
2. **Get complexity** — Static analysis (SonarQube, CodeClimate, etc.)
3. **Combine** — Files that are both complex AND frequently changed
4. **Prioritize** — Focus on high churn + high complexity

### Tools

| Tool | What it measures |
|------|------------------|
| CodeClimate | Complexity, coverage, duplication |
| SonarQube | Bugs, code smells, complexity |
| [Codescene](https://codescene.io/) | Hotspots, technical debt, trends |
| git log --shortstat | File change frequency |

### Presenting to Stakeholders

- Translate to business terms: "This file has X changes and Y complexity"
- Show ROI: "Fixing this will reduce time-to-change by estimated Z%"

---

## The "Peel and Slice" Technique

From Llewellyn Falgo, via Carlo.

### Process

1. Push infrastructure bits (HTTP, DB) to edges of function
2. Extract domain logic in the middle
3. Domain becomes testable

```javascript
// Before: Logic mixed with I/O
async function calculatePrice(req, res) {
    const type = req.body.type;
    const date = req.body.date;
    const age = req.body.age;
    
    // I/O in the middle - hard to test
    const basePrice = await db.fetchBasePrice(type);
    const holidays = await db.fetchHolidays();
    
    // Logic mixed with I/O
    let price = basePrice;
    if (holidays.includes(date)) price *= 1.2;
    if (age < 18) price *= 0.8;
    
    res.json({ price });
}

// After: I/O at edges, domain in middle
async function calculatePrice(req, res) {
    const basePrice = await fetchBasePrice(req.body.type);
    const holidays = await fetchHolidays();
    
    // Testable domain logic!
    const price = calculate(basePrice, req.body.date, req.body.age, holidays);
    
    res.json({ price });
}
```

---

## Sources

- [Understand Legacy Code](https://understandlegacycode.com/) — Nicolas Carlo
- "Working Effectively with Legacy Code" — Michael Feathers (foundation)
- [The Mikado Method](https://www.mikadomethod.info/) — Book
- [Your Code as a Crime Scene](https://www.adamtornhill.com/) — Adam Tornhill