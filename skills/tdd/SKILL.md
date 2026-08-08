---
name: tdd
description: Red-green-refactor discipline. Write the test first, watch it fail, write minimal code to pass.
allowed-tools: Bash Read Edit Write
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

### Test real behavior

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

### Keep test-only methods in test utilities

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

### Understand dependencies before mocking

Before mocking any method:
1. What side effects does the real method have?
2. Does this test depend on any of those side effects?
3. If yes — mock at a lower level, not the method the test depends on
4. If unsure — run the test with the real implementation first, observe what needs to happen, THEN add minimal mocking

Red flags: "I'll mock this to be safe." "This might be slow, better mock it." Mocking without understanding the dependency chain.

### Mock complete data structures

Mock the COMPLETE data structure as it exists in reality, not just fields your immediate test uses. Partial mocks hide structural assumptions and fail silently when downstream code depends on fields you didn't include.

## Flaky Tests

Never `setTimeout`/`sleep` in tests — wait for the actual condition. The `waitFor`
pattern, ready-made condition snippets, and the one case where a timed wait is
legitimate: [references/condition-based-waiting.md](references/condition-based-waiting.md).

## Gut checks

- Feels too simple → write the test anyway, takes 30 seconds
- Tempted to code first → test-first proves the test catches failures
- Tempted to skip once → discipline works because it's every time
- Test passes on first run → flip the assertion to verify it can fail
