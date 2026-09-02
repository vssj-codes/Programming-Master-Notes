# Table-of-Contents

<!-- toc -->

- [Testing and Code Quality](#testing-and-code-quality)
  * [What Is a Test?](#what-is-a-test)
  * [The Test Runner](#the-test-runner)
  * [Assertions](#assertions)
  * [Types of Tests — Scope Mental Model](#types-of-tests--scope-mental-model)
    + [Traditional Terminology (still used in the industry)](#traditional-terminology-still-used-in-the-industry)
    + [The Test Pyramid](#the-test-pyramid)
    + [Variations (all are specializations of the above three)](#variations-all-are-specializations-of-the-above-three)
  * [Test Doubles](#test-doubles)
  * [State Verification vs Interaction Verification](#state-verification-vs-interaction-verification)
  * [Don't Mock What You Don't Own](#dont-mock-what-you-dont-own)
  * [Dependency Injection](#dependency-injection)
    + [Functional Core / Imperative Shell](#functional-core--imperative-shell)
  * [Replacing Specific Dependencies](#replacing-specific-dependencies)
    + [Database — Use a Real One](#database--use-a-real-one)
    + [Network — Use a Real HTTP Server](#network--use-a-real-http-server)
    + [Time — Don't Sleep](#time--dont-sleep)
  * [Hermetic Testing](#hermetic-testing)
  * [Test-Driven Development (TDD)](#test-driven-development-tdd)
    + [The Red-Green-Refactor Loop](#the-red-green-refactor-loop)
    + [When TDD Works](#when-tdd-works)
    + [When TDD Doesn't Work](#when-tdd-doesnt-work)
    + [The TDD Discourse](#the-tdd-discourse)
  * [Flaky Tests](#flaky-tests)
    + [Top 3 Causes (Luo et al., 2014 — 201 real flaky test fixes)](#top-3-causes-luo-et-al-2014--201-real-flaky-test-fixes)
    + [Fixing Test Order Dependency](#fixing-test-order-dependency)
  * [Measuring Test Suites — Code Coverage](#measuring-test-suites--code-coverage)
    + [Coverage Flavors](#coverage-flavors)
    + [The Coverage Benchmark](#the-coverage-benchmark)
    + [The Right Way to Use Coverage](#the-right-way-to-use-coverage)
  * [Mutation Testing — Testing Your Tests](#mutation-testing--testing-your-tests)
  * [Code Complexity Metrics](#code-complexity-metrics)
    + [Cyclomatic Complexity (McCabe, 1976)](#cyclomatic-complexity-mccabe-1976)
    + [Cognitive Complexity (SonarSource)](#cognitive-complexity-sonarsource)
  * [Static Analysis Tools](#static-analysis-tools)
  * [Summary](#summary)

<!-- tocstop -->

---

# Testing and Code Quality

**Source:** Sriniously — Backend from First Principles (Video 27)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## What Is a Test?

A **test** is a second program that runs your program and makes claims about what happened.

A bad test — written in a hurry or left incomplete — is **more harmful than no test at all**. It gives false confidence while hiding actual bugs.

**Primary reason to write tests:** Catch **regressions** — bugs that were already fixed but reappear as new code is added. A deterministic test suite lets teams move fast without fear of breaking existing features.

---

## The Test Runner

A **test runner** is a program that:
1. Finds all test files in the codebase
2. Generates and compiles a `main` function that registers and executes every test
3. Runs the resulting binary
4. Reports how many tests passed and failed

Go's `go test` compiles the test binary, runs it, then deletes the build artifact. Pass `-work` to keep the artifact and inspect the generated `_test_main.go`.

---

## Assertions

An **assertion** is a check that fails or stops the test when a condition is not met. Most languages provide assertion libraries. In Go, assertions are manual `if` conditions that call `t.Errorf()`.

**Rule:** Always write a meaningful failure message that includes what was expected and what was actually received.

---

## Types of Tests — Scope Mental Model

Instead of debating the imprecise definitions of unit/integration/E2E, classify tests by **what they are allowed to touch**:

| Level | What it touches | Characteristics |
|-------|----------------|-----------------|
| **Small** | One process only — no network, no database, no filesystem, no `sleep` | Fast, deterministic, cheap |
| **Medium** | Same machine — local Docker containers (Postgres, Redis), no internet | Moderate speed, moderate cost |
| **Large** | Anything — external APIs, real services, internet | Slow, expensive, risk of flakiness |

This classification is **enforceable by the machine** (did the test open a socket or not?), unlike "unit test" which has no universal definition.

### Traditional Terminology (still used in the industry)

| Term | Scope |
|------|-------|
| **Unit test** | Tests one unit in isolation; no agreed definition of "unit" |
| **Integration test** | Two or more components working together, often with a real external dependency (database) |
| **End-to-end test** | Full user journey — starts with authentication and follows every path a real user would take |

### The Test Pyramid

```
          /\
         /E2E\       ← few, expensive, slow
        /------\
       /  Integ  \   ← some, moderate cost
      /------------\
     /  Unit tests  \ ← many, cheap, fast
    /__________________\
```

The pyramid is about **cost**. Higher = more expensive and slower.

### Variations (all are specializations of the above three)

| Variation | Purpose |
|-----------|---------|
| **Regression testing** | Specifically targets previously-fixed bugs to prevent recurrence |
| **Functional / TDD testing** | Spec written as tests before code; test serves as definition of done |
| **Performance testing** | Measures speed, memory, CPU usage |
| **Security testing** | Checks for privacy violations, auth bypasses, insecure behavior |

---

## Test Doubles

When code depends on external systems (database, payment provider, email service), you replace the real dependency with a **test double** — the category name, analogous to a stunt double in film.

All five types are commonly (and incorrectly) called "mocks":

| Type | What it does |
|------|-------------|
| **Dummy** | Satisfies a function signature requirement but is never actually used (e.g., a no-op logger passed to a constructor) |
| **Stub** | Always returns the same pre-configured response regardless of how many times it's called or what arguments are passed |
| **Spy** | Same as a stub, but also records every call made to it — arguments, call count, call order |
| **Mock** | Has pre-set expectations; fails the test if those expectations aren't met (e.g., "this method must be called exactly once with argument X") |
| **Fake** | A simplified but functionally correct alternative implementation (e.g., an in-memory map instead of a real Postgres database) |

---

## State Verification vs Interaction Verification

**State verification:** Assert what the system *is* after the operation.
> "Is the task now in the 'done' column?"

**Interaction verification (behavior verification):** Assert *how* the system got there.
> "Was `publish()` called exactly twice with events of type `task.moved`?"

**Prefer state verification.** Interaction verification creates **change detector tests** — tests that break when implementation changes even though observable behavior is identical (e.g., combining two events into one without changing the end result). Google has written about this problem.

> **Rule:** Only use interaction verification when *the interaction itself is the requirement* (e.g., verifying that an email is sent exactly once).

---

## Don't Mock What You Don't Own

Testing a third-party client (payment provider, email service) by mocking it directly means:
- All response shapes, field names, error formats, and edge cases were defined by you
- Any mismatch between your mock and reality only surfaces in production

**Fix:**
1. **Wrap** the third-party in your own interface with your own types
2. Mock your wrapper (you own its contract)
3. Write a small set of **contract tests** against the provider's sandbox API to verify your wrapper matches reality — fast, no payment data involved

---

## Dependency Injection

When a function creates its own dependencies (database client, Redis client), you cannot replace them during testing.

**Fix:** Pass dependencies as arguments instead of creating them internally. This is **dependency injection** — the calling code decides what gets injected.

### Functional Core / Imperative Shell

Structure your codebase in two layers:

```
┌────────────────────────────────────────────┐
│            Imperative Shell                │  ← thin layer; talks to DB, external APIs
│  ┌──────────────────────────────────────┐  │
│  │         Functional Core              │  │  ← pure functions: takes values, returns values
│  │   (business logic, transformations) │  │  ← no external I/O, easy to test exhaustively
│  └──────────────────────────────────────┘  │
└────────────────────────────────────────────┘
```

- **Core:** Easy to test — nothing to mock, all owned code
- **Shell:** Hard to test — but thin, so there's little logic to verify

---

## Replacing Specific Dependencies

### Database — Use a Real One

**Temptation:** Replace Postgres with an in-memory fake for speed.

**Problem:** Bugs actually live in the database layer — wrong column names, missing indexes, unique constraint violations, transaction semantics, Postgres-specific JSON operators. An in-memory fake makes all of these invisible.

**Solution 1 — Transaction per test:**
- Open a transaction at test start
- Run all test logic inside it
- Roll back at end (never commit)
- Database stays clean for the next test
- Requires code to accept a transaction handle instead of a raw DB connection

**Solution 2 — Template database (Postgres):**
- Run all migrations on a template database once
- Each test creates its own database from the template in milliseconds
- Tests can run in parallel — each has its own database

**Tool: [testcontainers](https://testcontainers.com/)** — starts a real Docker container from test code and provides the connection address. Available for virtually all languages. Container starts once per package (~7s); individual tests using the template database run in milliseconds.

### Network — Use a Real HTTP Server

For testing your own HTTP client code (headers, serialization, error handling):

- **Go:** `net/http/httptest` starts a real HTTP server on a real port in the test process
- **Node.js:** [MSW](https://mswjs.io/) intercepts at the network layer without replacing the client

Requests go through real TCP — same as production.

### Time — Don't Sleep

**Wrong approach:** `sleep(100ms)` to wait for an async operation to complete. Flaky on loaded CI machines — the operation might take 110ms.

**Fix 1 — Wait for a condition, not a duration:**
- Go: wait on a channel signal from the goroutine completing
- Node: await a callback or promise

**Fix 2 — Injectable clock:**
- Pass a clock object as a dependency
- In tests, pass a fake clock you control completely
- No real-time dependency, no flakiness

---

## Hermetic Testing

A **hermetic test** brings its own environment — it starts everything it needs and stops it when done. It does not depend on externally-managed services or hand-configured state.

- `testcontainers` = hermetic database
- Injectable clock = hermetic time
- Hermetic tests run on any machine: local, CI, remote server

---

## Test-Driven Development (TDD)

A **workflow** (not a type of test) where tests are written before the application code.

### The Red-Green-Refactor Loop

```
RED    → Write a test for non-existent behavior → it fails
GREEN  → Write the minimum code that makes it pass (quality doesn't matter)
REFACTOR → Clean up the code; run tests; confirm still green
```

**Red phase purpose:** Forces you to design the API from the caller's perspective before thinking about implementation. Awkward tests signal awkward interfaces — discovered in minutes, not after the implementation is complete.

**Green phase purpose:** Making a test go from red to green proves the test actually checks something. Writing code to a passing test removes this proof — you never know if the test would have failed first.

**Refactor phase:** Mandatory — do not skip. The permission to write bad code in the green phase only exists because of this cleanup step.

### When TDD Works

- The expected output is clear, but the implementation is uncertain (parsers, pricing rules, state machines)
- Writing the specification is the easy part

### When TDD Doesn't Work

- You're still exploring the problem space and don't yet know what the right answer looks like
- Multiple approaches need to be prototyped before committing

### The TDD Discourse

| Source | Position |
|--------|----------|
| **Kent Beck** (TDD inventor) | The "unit" is a behavior, not a class; test the public interface, not each internal method |
| **Ian Cooper** ("TDD, Where Did It All Go Wrong?") | Mapping "unit" to "class" caused test-per-class patterns, heavy mocking, and tests that broke on every refactor |
| **DHH** ("TDD is Dead, Long Live Testing") | Against test-first culture; against designing systems for mockability over correctness |

---

## Flaky Tests

A **flaky test** passes and fails on identical code depending on environment or timing. A flaky test is more harmful than no test — it trains the team to re-run failures instead of investigating them.

### Top 3 Causes (Luo et al., 2014 — 201 real flaky test fixes)

| Cause | Description |
|-------|-------------|
| **Async/await** | Test sleeps for a fixed duration instead of waiting for a completion signal |
| **Concurrency** | Two parts of the test race each other |
| **Test order dependency** | Test B only passes because Test A ran first and left behind some state (database row, global variable, file) |

### Fixing Test Order Dependency

Options (in order of preference):

1. **Delete it** — a test that only works in a fixed order provides false safety
2. **Quarantine** — move out of the blocking suite; annotate with owner name, date, and reason so it can be tracked and fixed
3. **Add retry** — not recommended; masks the root cause

---

## Measuring Test Suites — Code Coverage

**Code coverage** = percentage of code executed while tests ran. The compiler instruments each basic block with a counter; blocks with count 0 were never exercised.

### Coverage Flavors

| Type | What it measures |
|------|-----------------|
| **Line coverage** | Did this line execute? |
| **Branch coverage** | Was each direction of each condition taken? |

A function with one `if` and no `else`: a test with the condition `true` gives 100% line coverage but 50% branch coverage.

### The Coverage Benchmark

Laura and Holmes, 2014 (ICSE) — studied large real-world Java programs, controlled for test suite size:

| Coverage | Rating |
|----------|--------|
| 60% | Acceptable |
| 75% | Commendable |
| 90% | Exemplary |

Correlation between coverage and bugs found is **low to moderate** once suite size is controlled. Coverage varies by system — no single ideal number applies universally.

### The Right Way to Use Coverage

1. **What is NOT covered matters more than the percentage.** The value is in the red lines in the report, not the number at the top.
2. **Do not mandate 100% coverage.** Engineers will write zero-value tests purely to hit the number, then stop improving.

---

## Mutation Testing — Testing Your Tests

**Mutation testing** measures how well your tests actually detect defects.

**How it works:**
1. The tool (e.g., Stryker for JS/TS) modifies your source code in small ways — one change at a time (a `>` becomes `>=`, a `+` becomes `-`, a return value becomes `0`)
2. Each modified version is a **mutant**
3. The full test suite runs against each mutant
4. If a test fails → mutant **killed** (good — your tests noticed the damage)
5. If all tests pass → mutant **survived** (bad — your tests missed the defect)
6. **Mutation score** = killed / total mutants

**Example:** A function with 100% line coverage and tests asserting only "it returns something" and "the length is > 0" might have a mutation score of ~52% — nearly half of all introduced bugs go undetected.

Mutation testing is **the most meaningful measure of test quality** — it tests the tests themselves.

---

## Code Complexity Metrics

### Cyclomatic Complexity (McCabe, 1976)

Count the number of independent execution paths through a function.

**Formula:**
```
cyclomatic complexity = edges − nodes + 2
```

**Shortcut:**
```
1 + (number of if, loop, case, &&, || in the function)
```

A function with no branches = 1. Each decision point adds 1.

**General target: < 10** (from the original paper). Exception: a flat `switch` with many independent cases is easy to read but scores high — the formula fails here.

### Cognitive Complexity (SonarSource)

Measures how hard the code is to understand as a human reader, not just how many paths it has. Penalizes **nesting** heavily.

| Code structure | Cyclomatic complexity | Cognitive complexity |
|---------------|----------------------|---------------------|
| Flat switch with 20 cases | High (21) | Low — easy to read |
| 3 levels of nesting | Moderate | High — hard to hold in your head |

**Rule:** Don't trust either metric blindly. Use them as **flagging tools** to identify code worth reviewing, then make a team decision on the right action.

---

## Static Analysis Tools

Tools that find problems **without running the code**:

| Tool | What it does |
|------|-------------|
| **Linter** | Scans source code for known bad patterns (condition always true, unchecked error, unused variable). Does not check syntax (compiler's job) or correctness (no reliable way). |
| **Formatter** | Rewrites code to a standard layout; doesn't change behavior |
| **Type checker** | Catches type mismatches at compile time or in the editor (Go, TypeScript, Rust) |
| **Static analyzer** | Advanced linter; tracks a value through the program across functions to detect issues like nil dereferences, SQL injection, taint flows |

---

## Summary

| Concept | Key point |
|---------|----------|
| **Test definition** | A second program that runs your code and checks its output; a bad test is worse than none |
| **Primary purpose** | Catch regressions — bugs that already happened don't come back |
| **Scope mental model** | In-process → in-machine → touches internet; enforceable, unlike unit/integration/E2E |
| **Test doubles** | Dummy / Stub / Spy / Mock / Fake — all colloquially called "mock" |
| **State vs interaction** | Assert what the system is, not how it got there; interaction verification creates change detector tests |
| **Don't mock externals** | Wrap third-party APIs in your own interface; write contract tests against their sandbox |
| **Dependency injection** | Pass dependencies as arguments — never let functions create their own external connections |
| **Functional core** | Push business logic into pure functions; keep I/O in a thin shell |
| **Real database in tests** | Transaction-per-test (rollback) or template database; use testcontainers |
| **Time** | Wait for a condition, not a duration; or inject a clock |
| **Hermetic** | Test brings its own environment; runs anywhere without external setup |
| **TDD loop** | Red → Green → Refactor; proves tests actually check something |
| **Flaky causes** | Async/await timing, concurrency, test order dependency |
| **Coverage** | 60/75/90% = acceptable/commendable/exemplary; what's NOT covered matters more than the number |
| **Mutation testing** | The tool damages your code; your tests should notice; mutation score = killed / total |
| **Cyclomatic complexity** | Count paths; target < 10; breaks down on flat switch statements |
| **Cognitive complexity** | Measures readability; penalizes nesting; better reflects human understanding |
| **Linters / static analysis** | Flag bad patterns without running code; use as signal, not law |
