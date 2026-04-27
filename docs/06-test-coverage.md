# Test Coverage

## What is Test Coverage?

**Test coverage** is a metric that measures the proportion of source code exercised by a test suite. It answers the question: "How much of the codebase is actually tested?"

Coverage is expressed as a percentage:

```
Coverage (%) = (Number of covered items / Total number of items) × 100
```

High coverage does not guarantee the absence of bugs, but low coverage almost certainly means there are untested code paths that could contain defects.

---

## Types of Coverage Metrics

### 1. Statement (Line) Coverage

Measures the percentage of executable statements that are executed by at least one test.

```
Statement Coverage = (Executed statements / Total statements) × 100
```

**Example:**
```java
int max(int a, int b) {
    if (a > b) {         // line 1
        return a;        // line 2
    }
    return b;            // line 3
}
```

A test calling `max(5, 3)` executes lines 1, 2 — statement coverage = 67%.  
Adding a test with `max(2, 7)` executes line 3 — statement coverage = 100%.

---

### 2. Branch Coverage

Measures the percentage of branches (decision outcomes) that are executed. A branch is created by any decision point: `if`, `else`, `switch`, ternary operator, `&&`, `||`.

```
Branch Coverage = (Executed branches / Total branches) × 100
```

**Why it matters:** A line can be covered (executed) while a branch within it is not. For example, an `if` with no `else` — the false branch is a branch too.

---

### 3. Function / Method Coverage

Measures the percentage of functions or methods that are called at least once.

```
Function Coverage = (Called functions / Total functions) × 100
```

Useful as a coarse-grained sanity check: if a function is never called during testing, it is completely untested.

---

### 4. Condition Coverage

Measures whether each Boolean sub-expression within a decision evaluates to both `true` and `false`.

**Example:**
```java
if (isLoggedIn && hasPermission) { ... }
```

Condition coverage requires tests where:
- `isLoggedIn = true` and `isLoggedIn = false`
- `hasPermission = true` and `hasPermission = false`

This is stricter than branch coverage and catches more subtle bugs.

---

### 5. Path Coverage

Measures the percentage of unique execution paths through the code that are tested. Path coverage is the most thorough but also the most expensive: the number of paths grows exponentially with the number of branches.

---

### 6. Modified Condition/Decision Coverage (MC/DC)

Used in safety-critical industries (avionics, medical devices, automotive). MC/DC requires that:
- Each condition independently affects the outcome of the decision.
- Each entry and exit point is taken.

Mandated by standards such as **DO-178C** (avionics) and **ISO 26262** (automotive).

---

## Coverage Targets

There is no universally "correct" coverage threshold. Common industry guidance:

| Context | Recommended Threshold |
|---------|----------------------|
| General business application | 70–80% line/branch |
| Critical business logic | 90%+ branch |
| Safety-critical systems (e.g., medical devices) | 100% MC/DC |
| Open-source libraries | 80–90% |

### Important Caveats

1. **100% coverage is not the goal.** Some code (e.g., defensive null-checks, logging) is difficult to cover meaningfully. Chasing 100% can lead to low-value tests.
2. **Coverage does not measure test quality.** A test that never asserts anything can achieve 100% statement coverage while verifying nothing.
3. **Use coverage to find gaps, not to grade teams.** Coverage as a KPI leads to gaming (writing tests purely to hit numbers).

---

## Measuring Coverage

### Java – JaCoCo

[JaCoCo](https://www.jacoco.org/) is the most widely used Java code coverage library. It integrates with Maven, Gradle, and IntelliJ IDEA.

**Maven configuration:**
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Run with: `mvn test` — report generated at `target/site/jacoco/index.html`.

**Enforcing a minimum threshold:**
```xml
<execution>
    <id>check</id>
    <goals><goal>check</goal></goals>
    <configuration>
        <rules>
            <rule>
                <limits>
                    <limit>
                        <counter>BRANCH</counter>
                        <value>COVEREDRATIO</value>
                        <minimum>0.80</minimum>
                    </limit>
                </limits>
            </rule>
        </rules>
    </configuration>
</execution>
```

---

### Python – Coverage.py

```bash
pip install coverage
coverage run -m pytest
coverage report -m          # terminal report
coverage html               # HTML report at htmlcov/index.html
```

**pytest-cov plugin (simpler):**
```bash
pytest --cov=src --cov-report=html --cov-fail-under=80
```

---

### JavaScript / TypeScript – Istanbul (nyc) / V8

Most JS test runners have built-in coverage support:

```bash
# Jest
jest --coverage

# Vitest
vitest run --coverage
```

---

## Coverage in CI/CD Pipelines

Integrating coverage into CI ensures that coverage thresholds are enforced automatically:

1. Run tests with coverage enabled.
2. Fail the build if coverage drops below the configured threshold.
3. Publish the HTML report as a CI artefact.
4. Optionally upload to a coverage service (Codecov, Coveralls, SonarQube).

**GitHub Actions example:**
```yaml
- name: Run tests with coverage
  run: mvn test

- name: Upload coverage report
  uses: codecov/codecov-action@v4
  with:
    files: target/site/jacoco/jacoco.xml
```

---

## Mutation Testing

**Mutation testing** goes beyond coverage by checking whether the tests actually *detect* changes (mutations) in the code. A mutation that is not caught by any test indicates a weak test.

Popular tools:
- **Java:** PIT (Pitest) — `mvn org.pitest:pitest-maven:mutationCoverage`
- **Python:** mutmut, Cosmic Ray
- **JavaScript:** Stryker

Mutation testing is slower than regular coverage analysis but provides much stronger confidence in the quality of the test suite.

---

## Further Reading

- [Test-Driven Development →](04-test-driven-development.md)
- [Best Practices →](09-best-practices.md)
- [Testing Tools →](10-testing-tools.md)
