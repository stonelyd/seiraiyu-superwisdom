# Flaky tests — condition-based waiting

Don't use `setTimeout` or `sleep` in tests. Wait for the actual condition you care
about, not a guess about how long it takes.

## The `waitFor` pattern

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

## Quick patterns

| Scenario | Pattern |
|----------|---------|
| Wait for event | `waitFor(() => events.find(e => e.type === 'DONE'))` |
| Wait for state | `waitFor(() => machine.state === 'ready')` |
| Wait for count | `waitFor(() => items.length >= 5)` |
| Wait for file | `waitFor(() => fs.existsSync(path))` |
| Complex condition | `waitFor(() => obj.ready && obj.value > 10)` |

## Before/after

```typescript
// BAD: guessing at timing
await new Promise(r => setTimeout(r, 300)); // Hope tools start in 300ms
expect(toolResults.length).toBe(2);         // Fails randomly

// GOOD: waiting for condition
await waitFor(() => toolResults.length >= 2, 'tool results');
expect(toolResults.length).toBe(2);         // Always succeeds
```

## When an arbitrary timeout IS correct

Only when testing actual timing behavior (debounce, throttle, tick intervals):

```typescript
await waitForEvent(manager, 'TOOL_STARTED'); // First: wait for condition
await new Promise(r => setTimeout(r, 200));   // Then: wait for timed behavior
// 200ms = 2 ticks at 100ms intervals — documented and justified
```

Requirements: wait for the triggering condition first, base the delay on known
timing (not guessing), and comment explaining WHY.

## Common mistakes

- Polling too fast (`setTimeout(check, 1)`) — wastes CPU. Use 10ms.
- No timeout — loops forever. Always include a timeout with a clear error message.
- Stale data — caching state before the loop. Call the getter inside the loop.
