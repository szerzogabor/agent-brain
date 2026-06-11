---
name: test-writing
description: Write tests that catch real bugs - clear naming, arrange-act-assert structure, edge-case coverage, minimal mocking. Use when adding tests, improving coverage, or when the user asks to "write tests for this" or "add test coverage".
---

# Test Writing

Write tests that fail when the code is wrong and clearly say why. A test that cannot fail, or whose failure is cryptic, is worse than no test.

## When to use

- Adding tests for new or existing behavior.
- After a bug fix (regression test) or during a refactor (safety net).

## Process

1. First, find and follow the project's existing test conventions: framework, file location, naming style, fixture patterns. Match them; do not import your own habits.
2. Decide what behavior to test, not what code to cover. List the cases:
   - The happy path with typical input.
   - Boundaries: empty, zero, one, maximum, just-over-maximum.
   - Error paths: invalid input, dependency failure, timeout — confirm the code fails the way it promises to.
   - Any case from a real bug report (regression).
3. Name each test after the behavior and expectation, so a failure reads like a sentence: `rejects_expired_token`, `returns_empty_list_when_no_matches` — not `test_case_3` or `test_works`.
4. Structure each test as Arrange–Act–Assert: set up the scenario, perform exactly one action, assert the outcome. One logical assertion per test; if you need a second act, write a second test.
5. Mock only at true boundaries: network, clock, filesystem, external services. Do not mock the code under test or its in-process collaborators — that tests your mocks, not your code.
6. Make assertions specific: assert the actual expected value, not just "is not null" or "no exception thrown".
7. Verify the test works: run it and see it pass, then mentally (or actually) break the code and confirm the test would fail. A test you have never seen fail is unverified.
8. Keep tests independent: no shared mutable state, no required execution order, no dependence on real time or randomness without controlling the seed.

## Anti-patterns to avoid

- Testing implementation details (private methods, internal call counts) so the test breaks on every refactor.
- Snapshot/golden tests for logic that has a precise, assertable answer.
- Chasing a coverage percentage with assertion-free tests.
- Sleeps and real-time waits instead of controlled clocks — the source of most flaky tests.
- Copy-pasting a test and changing one value when a parameterized test expresses the table cleanly.
