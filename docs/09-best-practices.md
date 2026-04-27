# Best Practices for Software Testing

## Overview

Writing tests is easy; writing *good* tests is hard. Good tests are fast, reliable, readable, and maintainable. They provide confidence when they pass and clear, actionable information when they fail.

This document collects battle-tested principles and practices for building a high-quality test suite.

---

## 1. Follow the FIRST Principles

**FIRST** is an acronym summarising the properties of a good unit test:

| Letter | Property | Description |
|--------|----------|-------------|
| **F** | Fast | Tests should run in milliseconds. Slow tests discourage frequent execution. |
| **I** | Independent | Each test should stand alone and not depend on execution order or shared mutable state. |
| **R** | Repeatable | Tests should produce the same result in any environment, at any time. |
| **S** | Self-validating | Tests should automatically pass or fail — no manual inspection required. |
| **T** | Thorough / Timely | Cover edge cases; write tests at the right time (ideally before or alongside the code). |

---

## 2. One Assertion per Concept

Each test should verify a single behaviour or concept. When a test fails, the name should immediately tell you what is broken.

**Avoid:**
```java
@Test
void testUser() {
    User user = userService.create("alice", "alice@example.com");
    assertNotNull(user.getId());
    assertEquals("alice", user.getName());
    assertTrue(user.isActive());
    assertFalse(user.isAdmin());
}
```

**Prefer:**
```java
@Test void createsUserWithGeneratedId() { ... }
@Test void storesUserName() { ... }
@Test void newUserIsActiveByDefault() { ... }
@Test void newUserIsNotAdminByDefault() { ... }
```

---

## 3. Use Descriptive Test Names

A test name should describe:
- The **subject** (what component/function is under test).
- The **scenario** (what conditions apply).
- The **expected result**.

**Pattern:** `methodName_scenario_expectedBehaviour`

| Bad Name | Good Name |
|----------|-----------|
| `test1()` | `add_twoPositiveNumbers_returnsSum()` |
| `testLogin()` | `login_withInvalidPassword_throwsAuthException()` |
| `testOrder()` | `placeOrder_whenCartIsEmpty_returnsValidationError()` |

---

## 4. Arrange-Act-Assert (AAA)

Structure every test in three clearly separated sections:

```java
@Test
void deposit_validAmount_increasesBalance() {
    // Arrange
    BankAccount account = new BankAccount(100.00);

    // Act
    account.deposit(50.00);

    // Assert
    assertEquals(150.00, account.getBalance(), 0.001);
}
```

The AAA pattern makes tests easy to read and reason about.

---

## 5. Use Test Doubles Appropriately

**Test doubles** replace real dependencies in unit tests. Choose the right type:

| Type | When to Use |
|------|-------------|
| **Dummy** | Object is passed but never used (fills a parameter list) |
| **Stub** | Returns canned responses to specific calls; no verification |
| **Mock** | Verifies that specific interactions (method calls) happened |
| **Fake** | Working simplified implementation (e.g., in-memory database) |
| **Spy** | Wraps a real object and can verify interactions after the fact |

**Guideline:** Prefer stubs and fakes over mocks. Over-using mocks ties tests to implementation details and makes refactoring painful.

---

## 6. Avoid Testing Implementation Details

Tests should verify *behaviour* (what the code does), not *implementation* (how it does it).

**Fragile test (tests implementation):**
```java
verify(userRepository, times(1)).save(any(User.class));
verify(emailService, times(1)).sendWelcomeEmail(any(String.class));
```

**Robust test (tests behaviour):**
```java
// After registration, the user can log in
loginPage.submit("alice", "password");
assertTrue(dashboard.isDisplayed());
```

If you refactor the internals (e.g., change from direct email sending to an event queue), the fragile test breaks even though the behaviour is unchanged.

---

## 7. Keep Tests Independent

- **Never share mutable state** between tests (class-level fields that are modified by tests).
- **Never rely on test execution order.** Each test must set up its own preconditions.
- **Use `@BeforeEach` / `setUp()`** to create fresh objects for each test.

**Problematic:**
```java
private List<String> names = new ArrayList<>(); // shared mutable state

@Test void addsFirstName() { names.add("Alice"); assertEquals(1, names.size()); }
@Test void addsSecondName() { names.add("Bob");   assertEquals(1, names.size()); } // fails if run after first test
```

---

## 8. Manage Test Data Carefully

- Use **realistic but synthetic data** — never production data in tests.
- Use **object mothers** or **test data builders** to create complex test objects.
- Reset or roll back the database between integration tests.
- Never hard-code IDs that may differ across environments.

**Builder pattern:**
```java
User user = UserBuilder.aUser()
    .withName("Alice")
    .withEmail("alice@example.com")
    .withRole(Role.ADMIN)
    .build();
```

---

## 9. Test Boundary Values

Most defects occur at the boundaries of valid input ranges. Always test:

- **Minimum valid value:** e.g., age = 0
- **Maximum valid value:** e.g., age = 150
- **Just below minimum:** e.g., age = -1 (should fail validation)
- **Just above maximum:** e.g., age = 151 (should fail validation)
- **Null / empty:** What happens with no input?

---

## 10. Make Tests Fast

Slow tests discourage running them frequently. Strategies:

- Keep unit tests free from I/O (disk, network, database).
- Run integration and E2E tests in a separate suite that can be triggered less frequently.
- Use in-memory databases (H2, SQLite) for integration tests where feasible.
- Parallelise test execution where tests are independent.
- Profile the test suite; fix the slowest 10% first.

---

## 11. Treat Test Code as Production Code

Test code is maintained for the life of the product. Apply the same standards:

- Follow the DRY principle: extract repeated setup into helper methods.
- Refactor tests when you refactor production code.
- Review test code in pull requests.
- Keep test files well-organised and consistently named.

---

## 12. Maintain a Clean CI/CD Pipeline

- **Never let the build stay broken.** A broken build desensitises the team to failures.
- **Fix flaky tests immediately.** A flaky test is worse than no test — it erodes trust.
- **Set coverage thresholds** and fail the build if they drop.
- **Run tests on every commit** to catch regressions early.

---

## 13. Document Non-Obvious Tests

In most cases, a well-named test with the AAA structure is self-documenting. However, tests that verify complex or non-intuitive business rules benefit from a short comment:

```java
@Test
void chargesExtraSurchargeForOrdersUnder10Euros() {
    // Orders below the minimum value incur a €2.00 handling fee per business rule BR-47.
    Order order = new Order(List.of(new Item("pen", 5.00)));
    assertEquals(7.00, pricingService.calculateTotal(order), 0.001);
}
```

---

## 14. Use Parameterised Tests for Multiple Inputs

Avoid duplicating test logic for similar scenarios — use parameterised tests instead:

**JUnit 5:**
```java
@ParameterizedTest
@CsvSource({
    "2, 3, 5",
    "0, 0, 0",
    "-1, 1, 0",
    "100, -50, 50"
})
void add_variousInputs_returnsCorrectSum(int a, int b, int expected) {
    assertEquals(expected, calculator.add(a, b));
}
```

**pytest:**
```python
@pytest.mark.parametrize("a, b, expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
    (100, -50, 50),
])
def test_add(a, b, expected):
    assert add(a, b) == expected
```

---

## 15. Adopt a Shift-Left Testing Mindset

"Shift left" means moving testing activities earlier in the SDLC:

- Write tests **before or alongside** code (TDD/BDD).
- Conduct **code reviews** that include test quality, not just production code.
- Run SAST and SCA tools **during development**, not just before release.
- Involve testers in **requirements discussions** to catch ambiguities early.

---

## Further Reading

- [Test Coverage →](06-test-coverage.md)
- [Testing Tools →](10-testing-tools.md)
- [Test-Driven Development →](04-test-driven-development.md)
- Robert C. Martin, *Clean Code* (Prentice Hall, 2008) — Chapter 9: Unit Tests
- Gerard Meszaros, *xUnit Test Patterns* (Addison-Wesley, 2007)
