# Introduction to Software Testing

## What is Software Testing?

Software testing is the process of evaluating a software application to find defects, verify that it behaves as expected, and validate that it meets the requirements of stakeholders. Testing can be performed manually by a human tester or automatically by tools and scripts.

At its core, testing answers two fundamental questions:

1. **Verification** – "Are we building the product right?" (Does the software conform to specifications?)
2. **Validation** – "Are we building the right product?" (Does the software meet user needs?)

---

## Why Testing is Important

### The Cost of Bugs

Defects discovered late in the software development life cycle (SDLC) are exponentially more expensive to fix than those found early. The classic **rule of ten** states that the cost of fixing a bug multiplies by roughly 10 at each phase:

| Phase Found | Relative Cost to Fix |
|-------------|----------------------|
| Requirements | 1× |
| Design | 5× |
| Coding | 10× |
| Integration testing | 20× |
| System testing | 50× |
| Production | 100× |

Early testing reduces risk, saves money, and protects reputation.

### Real-World Consequences of Poor Testing

- **Ariane 5 rocket (1996):** A data-type conversion overflow caused the rocket to self-destruct 37 seconds after launch, destroying a $370 million satellite payload.
- **Therac-25 radiation therapy machine (1980s):** A race condition in the software caused lethal radiation overdoses, injuring and killing patients.
- **Knight Capital (2012):** A deployment error activated untested trading code, losing $440 million in 45 minutes.

---

## Core Principles of Software Testing

The **ISTQB (International Software Testing Qualifications Board)** defines seven fundamental testing principles:

1. **Testing shows the presence of defects, not their absence.**  
   A passing test suite does not mean software is bug-free; it means no bugs were found with the current tests.

2. **Exhaustive testing is impossible.**  
   Testing every possible input, output, and execution path is not feasible. Risk-based testing prioritises the most important areas.

3. **Early testing saves time and money.**  
   Shift-left testing finds problems when they are cheapest to fix.

4. **Defects cluster together.**  
   A small number of modules typically contain the majority of defects (Pareto principle / 80-20 rule).

5. **Beware of the pesticide paradox.**  
   Repeating the same tests stops finding new bugs. Tests must be reviewed and updated regularly.

6. **Testing is context-dependent.**  
   Testing a safety-critical medical device requires a different approach than testing a marketing website.

7. **Absence of errors is a fallacy.**  
   Finding and fixing bugs is worthless if the software does not meet user needs.

---

## The Testing Mindset

Effective testers approach software with constructive scepticism:

- **Assume defects exist** until the software proves otherwise.
- **Think like an adversary** – try to break the system, not just confirm it works on the happy path.
- **Be curious** – explore edge cases, boundary values, and unusual user behaviours.
- **Communicate clearly** – a defect report that cannot be reproduced or understood by a developer is nearly useless.

---

## The Software Testing Life Cycle (STLC)

The STLC defines the phases of a testing effort from inception to closure:

```
1. Requirement Analysis
        ↓
2. Test Planning
        ↓
3. Test Case Design & Development
        ↓
4. Test Environment Setup
        ↓
5. Test Execution
        ↓
6. Test Cycle Closure
```

### Phase Descriptions

| Phase | Key Activities | Outputs |
|-------|----------------|---------|
| Requirement Analysis | Review requirements, identify testable conditions | Requirement Traceability Matrix (RTM) |
| Test Planning | Define scope, strategy, resources, schedule | Test Plan |
| Test Case Design | Write test cases and scripts, prepare test data | Test Cases, Test Scripts |
| Environment Setup | Configure hardware, software, test data | Test environment ready |
| Test Execution | Run tests, log defects, retest fixes | Defect reports, Test execution logs |
| Cycle Closure | Evaluate completion criteria, write test summary | Test Summary Report |

---

## Static vs. Dynamic Testing

| Aspect | Static Testing | Dynamic Testing |
|--------|---------------|-----------------|
| Definition | Examining artefacts without executing code | Executing the software with test inputs |
| When | Early phases (requirements, design, code reviews) | After code is compiled or deployed |
| Examples | Code reviews, walkthroughs, static analysis | Unit tests, integration tests, exploratory testing |
| Goal | Find defects in documents and code structure | Find defects in runtime behaviour |

---

## Verification vs. Validation

| | Verification | Validation |
|-|--------------|------------|
| Question | Are we building the product right? | Are we building the right product? |
| Focus | Process and standards compliance | End-user satisfaction |
| Techniques | Reviews, inspections, walkthroughs | Testing, prototyping, user acceptance |
| Artefacts | Specifications, design docs, code | Executable software |

---

## Further Reading

- [Types of Testing →](02-types-of-testing.md)
- [Testing Levels →](03-testing-levels.md)
- ISTQB Foundation Level Syllabus (https://www.istqb.org)
