# Testing Levels

Testing levels define the scope and granularity of a test. Each level has a distinct purpose, a set of best practices, and a typical owner. Together, they form a layered safety net for the software.

---

## 1. Unit Testing

### Definition

A **unit test** verifies a single, isolated unit of code — typically a function, method, or class — in complete isolation from its dependencies.

### Characteristics

- **Fast:** Milliseconds per test; thousands can run in seconds.
- **Isolated:** External dependencies (databases, HTTP services, file systems) are replaced with test doubles (mocks, stubs, fakes).
- **Developer-owned:** Written by the developer alongside or before the production code.
- **Deterministic:** The same input always produces the same output regardless of environment.

### What to Test

- Happy path (expected inputs produce expected outputs).
- Edge cases (empty collections, zero, `null`/`nil`, maximum values).
- Error cases (exceptions thrown for invalid inputs).
- Boundary values (values just inside and just outside valid ranges).

### Example (Java with JUnit 5)

```java
class CalculatorTest {

    private final Calculator calculator = new Calculator();

    @Test
    void addsTwoPositiveNumbers() {
        assertEquals(5, calculator.add(2, 3));
    }

    @Test
    void throwsOnDivisionByZero() {
        assertThrows(ArithmeticException.class, () -> calculator.divide(10, 0));
    }
}
```

### Example (Python with pytest)

```python
def test_add_two_positive_numbers():
    assert add(2, 3) == 5

def test_divide_by_zero_raises():
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)
```

### Common Pitfalls

- Testing implementation details (tightly coupling tests to internal method calls).
- Overusing mocks, making tests pass even when behaviour is wrong.
- Writing tests after the fact without actually running them against failing code.

---

## 2. Integration Testing

### Definition

An **integration test** verifies that two or more components or services work correctly together.

### Types of Integration Testing

| Approach | Description |
|----------|-------------|
| **Big-Bang** | Integrate all components at once and test as a whole |
| **Top-Down** | Integrate and test from the top of the module hierarchy downward, using stubs for lower layers |
| **Bottom-Up** | Integrate and test from lower-level modules upward, using drivers for upper layers |
| **Sandwich (Hybrid)** | Combines top-down and bottom-up |
| **Incremental** | Add and test one component at a time |

### What to Test

- Data flow between modules (correct data is passed and returned).
- API contracts between services.
- Database persistence (data is actually saved and retrieved correctly).
- Message queue consumers and producers.
- Third-party service integrations.

### Example (Spring Boot with `@SpringBootTest`)

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void returnsUserById() throws Exception {
        mockMvc.perform(get("/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("Alice"));
    }
}
```

### Test Containers

[Testcontainers](https://testcontainers.com/) allows integration tests to spin up real databases, message brokers, and other services in Docker containers, providing high confidence without needing a shared environment.

---

## 3. System Testing

### Definition

**System testing** evaluates the complete, integrated system against the specified requirements. It is a black-box testing level performed by QA teams in an environment that mirrors production.

### Scope

- End-to-end user journeys.
- Cross-cutting concerns (logging, auditing, error handling).
- Non-functional requirements (performance, security, accessibility).

### Entry and Exit Criteria

| Criteria | Entry | Exit |
|----------|-------|------|
| Code | All code is written and unit/integration tested | All critical and high defects are resolved |
| Environment | Production-like environment is ready | Test completion rate ≥ 95% |
| Test Cases | All test cases are reviewed and approved | All test cases executed at least once |

### Differences from Integration Testing

| Aspect | Integration Testing | System Testing |
|--------|---------------------|----------------|
| Scope | Two or more components | The entire system |
| Performed by | Developers or QA | QA team |
| Environment | Development or test | Staging / pre-production |
| Focus | Component interactions | System behaviour vs. requirements |

---

## 4. Acceptance Testing

### Definition

**Acceptance testing** determines whether the system satisfies acceptance criteria defined by the business. It is typically the final testing level before software is released.

### User Acceptance Testing (UAT)

UAT is performed by the actual end users or their representatives:

1. Business analysts define acceptance criteria in user stories.
2. Testers or business users execute real-world scenarios.
3. Defects are reported and resolved.
4. The business formally signs off on the software.

### Example Acceptance Criteria (Gherkin)

```gherkin
Feature: User login

  Scenario: Successful login with valid credentials
    Given the user is on the login page
    When the user enters a valid username and password
    And clicks the "Login" button
    Then the user is redirected to the dashboard
    And a welcome message is displayed

  Scenario: Failed login with invalid password
    Given the user is on the login page
    When the user enters a valid username and an invalid password
    And clicks the "Login" button
    Then an error message "Invalid credentials" is displayed
    And the user remains on the login page
```

---

## 5. Summary Comparison

| Level | Granularity | Owner | Speed | Environment |
|-------|-------------|-------|-------|-------------|
| Unit | Function / Class | Developer | Very fast | Local machine |
| Integration | Component interactions | Developer / QA | Medium | Dev / CI |
| System | Full application | QA | Slow | Staging |
| Acceptance | Business requirements | QA / Business | Slowest | Staging / Production-like |

---

## The Test Trophy (Alternative to the Pyramid)

Kent C. Dodds proposed the **Test Trophy** which places static analysis at the base, fewer unit tests, a larger proportion of integration tests, and a small number of end-to-end tests:

```
         /\
        /E2E\
       /------\
      /  Integ  \
     /------------\
    /    Unit      \
   /----------------\
  /  Static Analysis \
 /--------------------\
```

The trophy reflects that integration tests often provide the best balance between confidence and speed for modern web and API applications.

---

## Further Reading

- [Test-Driven Development →](04-test-driven-development.md)
- [Behavior-Driven Development →](05-behavior-driven-development.md)
- [Test Coverage →](06-test-coverage.md)
