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

## Anti-Patterns

### Don't test mock behavior

```typescript
// BAD: testing that the mock exists
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});

// GOOD: test real behavior
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});
```

Before asserting on any mock element: "Am I testing real behavior or mock existence?" If mock existence — delete the assertion or unmock the component.

### Don't add test-only methods to production code

```typescript
// BAD: destroy() only used in tests
class Session {
  async destroy() { /* cleanup */ }
}

// GOOD: test utilities handle test cleanup
// test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) await workspaceManager.destroyWorkspace(workspace.id);
}
```

Before adding any method to a production class: "Is this only used by tests?" If yes — put it in test utilities.

### Don't mock without understanding dependencies

Before mocking any method:
1. What side effects does the real method have?
2. Does this test depend on any of those side effects?
3. If yes — mock at a lower level, not the method the test depends on
4. If unsure — run the test with the real implementation first, observe what needs to happen, THEN add minimal mocking

Red flags: "I'll mock this to be safe." "This might be slow, better mock it." Mocking without understanding the dependency chain.

### Don't create incomplete mocks

Mock the COMPLETE data structure as it exists in reality, not just fields your immediate test uses. Partial mocks hide structural assumptions and fail silently when downstream code depends on fields you didn't include.

## Flaky Tests — Condition-Based Waiting

Don't use `setTimeout` or `sleep` in tests. Wait for the actual condition you care about, not a guess about how long it takes.

### The `waitFor` pattern

```typescript
async function waitFor<T>(
  condition: () => T | undefined | null | false,
  description: string,
  timeoutMs = 5000
): Promise<T> {
  const startTime = Date.now();
  while (true) {
    const result = condition();
    if (result) return result;
    if (Date.now() - startTime > timeoutMs) {
      throw new Error(`Timeout waiting for ${description} after ${timeoutMs}ms`);
    }
    await new Promise(r => setTimeout(r, 10)); // Poll every 10ms
  }
}
```

### Quick patterns

| Scenario | Pattern |
|----------|---------|
| Wait for event | `waitFor(() => events.find(e => e.type === 'DONE'))` |
| Wait for state | `waitFor(() => machine.state === 'ready')` |
| Wait for count | `waitFor(() => items.length >= 5)` |
| Wait for file | `waitFor(() => fs.existsSync(path))` |
| Complex condition | `waitFor(() => obj.ready && obj.value > 10)` |

### Before/after

```typescript
// BAD: guessing at timing
await new Promise(r => setTimeout(r, 300)); // Hope tools start in 300ms
expect(toolResults.length).toBe(2);         // Fails randomly

// GOOD: waiting for condition
await waitFor(() => toolResults.length >= 2, 'tool results');
expect(toolResults.length).toBe(2);         // Always succeeds
```

### When arbitrary timeout IS correct

Only when testing actual timing behavior (debounce, throttle, tick intervals):

```typescript
await waitForEvent(manager, 'TOOL_STARTED'); // First: wait for condition
await new Promise(r => setTimeout(r, 200));   // Then: wait for timed behavior
// 200ms = 2 ticks at 100ms intervals — documented and justified
```

Requirements: wait for triggering condition first, based on known timing (not guessing), comment explaining WHY.

### Common mistakes

- Polling too fast (`setTimeout(check, 1)`) — wastes CPU. Use 10ms.
- No timeout — loops forever. Always include timeout with clear error message.
- Stale data — caching state before loop. Call the getter inside the loop for fresh data.

## Watch for

- "Too simple to need a test" → write it anyway, takes 30 seconds
- "I'll write the test after" → test-after proves nothing about failure detection
- "Just this once" → discipline only works if it's every time
- Test passes on first run → you didn't verify it catches failures
