---
name: tdd
description: Red-green-refactor discipline. Write the test first, watch it fail, write minimal code to pass.
---

# TDD

No production code without a failing test first.

## Red — Write failing test

- One minimal test for one behavior
- Clear name describing what's being tested
- Run it. Confirm it fails for the right reason.

## Green — Write minimal code

- Simplest code that makes the test pass
- Don't add features, don't refactor, don't anticipate
- Run tests. Confirm all pass.

## Refactor — Clean up

- Remove duplication, improve names, extract patterns
- Only after green
- Run tests. Confirm all still pass.

Repeat for the next behavior.

## Anti-patterns

- Testing mock behavior instead of real code through mocks
- Adding test-only methods to production code
- Mocking without understanding what you're replacing
- Writing production code before the test
- Test passes on first run — you didn't verify it catches failures

## Async timing

- Don't use `setTimeout` or `sleep` in tests
- Use condition-based waiting: `waitFor(() => condition)` or poll every 10ms with a 5s timeout
- Wait for state first, then optionally add a documented delay — never reverse the order

## Watch for

- "Too simple to need a test" → write it anyway, takes 30 seconds
- "I'll write the test after" → test-after proves nothing about failure detection
- "Just this once" → discipline only works if it's every time
