# Test-Driven Development (TDD)

## What is TDD?

**Test-Driven Development (TDD)** is a software development practice in which tests are written *before* production code. The developer writes a failing test that captures a desired behaviour, then writes just enough production code to make the test pass, and finally refactors the code while keeping all tests green.

TDD was popularised by Kent Beck in his book *Test-Driven Development: By Example* (2002) and is a cornerstone of Extreme Programming (XP).

---

## The Red-Green-Refactor Cycle

TDD follows a tight, iterative loop known as the **Red-Green-Refactor cycle**:

```
  ┌─────────────────────────────────────┐
  │                                     │
  │   1. RED   – Write a failing test   │
  │        ↓                            │
  │   2. GREEN – Make the test pass     │
  │        ↓                            │
  │   3. REFACTOR – Clean up the code   │
  │        ↓                            │
  │   (repeat for next requirement)     │
  │                                     │
  └─────────────────────────────────────┘
```

### Step 1 – Red: Write a Failing Test

Write the smallest possible test that describes one new behaviour. Run it. It should fail because the behaviour has not been implemented yet. A failing test confirms that the test is actually testing something.

### Step 2 – Green: Make the Test Pass

Write the minimum amount of production code needed to make the test pass. Resist the temptation to write more than necessary. The goal is green, not perfect.

### Step 3 – Refactor: Clean Up

Improve the design of both production code and test code without changing external behaviour. Remove duplication, clarify names, simplify logic. Every refactoring step must leave all tests green.

---

## A Complete TDD Example

### Requirement

Implement a `FizzBuzz` function that returns:
- `"Fizz"` for multiples of 3
- `"Buzz"` for multiples of 5
- `"FizzBuzz"` for multiples of both
- The number as a string for everything else

### Iteration 1 – Basic Number

**Red:**
```java
@Test
void returnsNumberAsString() {
    assertEquals("1", FizzBuzz.of(1));
}
```

**Green:**
```java
public class FizzBuzz {
    public static String of(int n) {
        return String.valueOf(n);
    }
}
```

**Refactor:** Nothing to clean up yet.

---

### Iteration 2 – Fizz

**Red:**
```java
@Test
void returnsFizzForMultipleOfThree() {
    assertEquals("Fizz", FizzBuzz.of(3));
}
```

**Green:**
```java
public static String of(int n) {
    if (n % 3 == 0) return "Fizz";
    return String.valueOf(n);
}
```

---

### Iteration 3 – Buzz

**Red:**
```java
@Test
void returnsBuzzForMultipleOfFive() {
    assertEquals("Buzz", FizzBuzz.of(5));
}
```

**Green:**
```java
public static String of(int n) {
    if (n % 3 == 0) return "Fizz";
    if (n % 5 == 0) return "Buzz";
    return String.valueOf(n);
}
```

---

### Iteration 4 – FizzBuzz

**Red:**
```java
@Test
void returnsFizzBuzzForMultipleOfBoth() {
    assertEquals("FizzBuzz", FizzBuzz.of(15));
}
```

**Green:**
```java
public static String of(int n) {
    if (n % 3 == 0 && n % 5 == 0) return "FizzBuzz";
    if (n % 3 == 0) return "Fizz";
    if (n % 5 == 0) return "Buzz";
    return String.valueOf(n);
}
```

All four tests are now green. The code is clean. TDD is complete for these four requirements.

---

## Benefits of TDD

| Benefit | Explanation |
|---------|-------------|
| **Design feedback** | Difficult-to-test code signals poor design. TDD nudges developers toward modular, loosely coupled code. |
| **Living documentation** | Tests describe what the system does in concrete, executable terms. |
| **Confidence during refactoring** | A comprehensive test suite acts as a safety net when changing existing code. |
| **Reduced debugging time** | Failing tests pinpoint the exact location of a defect. |
| **Smaller, focused commits** | Each Red-Green-Refactor cycle results in a small, verifiable increment. |

---

## Common TDD Anti-Patterns

| Anti-Pattern | Description | Fix |
|--------------|-------------|-----|
| **Writing tests after the fact** | Defeats the design benefit of TDD | Commit to writing tests first |
| **Testing implementation details** | Tests break on every refactor | Test behaviour, not internal method calls |
| **Giant tests** | One test covers many behaviours | One assertion per concept |
| **Skipping the refactor step** | Accumulates technical debt | Always clean up after going green |
| **Test-only coverage** | Tests pass but code is not truly verified | Run tests against real failing code |

---

## TDD Variants

### Inside-Out (Classic / Chicago Style)

Begin with the smallest unit (e.g., a domain model) and build outward. Let design emerge from the tests. Associated with Kent Beck.

### Outside-In (London / Mockist Style)

Begin with a failing acceptance test and work inward, using mocks to define the interfaces of collaborators before implementing them. Focuses on collaboration between objects.

---

## Tools by Language

| Language | Popular TDD Frameworks |
|----------|------------------------|
| Java | JUnit 5, TestNG, AssertJ |
| Python | pytest, unittest |
| JavaScript / TypeScript | Jest, Vitest, Mocha + Chai |
| C# | xUnit, NUnit, MSTest |
| Go | `testing` package, Testify |
| Ruby | RSpec, Minitest |
| Kotlin | Kotest, JUnit 5 |

---

## Further Reading

- [Behavior-Driven Development →](05-behavior-driven-development.md)
- [Test Coverage →](06-test-coverage.md)
- [Best Practices →](09-best-practices.md)
- Kent Beck, *Test-Driven Development: By Example* (Addison-Wesley, 2002)
