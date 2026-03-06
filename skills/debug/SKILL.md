---
name: debug
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes. Understand before fixing.
allowed-tools: Bash Read Edit Grep Glob AskUserQuestion
---

# Debug

## The Iron Law

```
COMPLETE ROOT CAUSE INVESTIGATION BEFORE EVERY FIX
```

Complete Phase 1 first, every time. Especially when the answer "seems obvious" — that's when investigation matters most.

## Phase 1 — Investigate

Do this BEFORE attempting ANY fix:

**Read error messages completely.** Don't skip past them. They often contain the exact answer. Read stack traces, note line numbers, file paths, error codes.

**Reproduce consistently.** Can you trigger it reliably? What are the exact steps? If not reproducible, gather more data — don't guess.

**Check recent changes.** `git log`, `git diff`. New dependencies, config changes, environment differences.

**Multi-component systems — add diagnostic instrumentation FIRST:**

```bash
# Log at EVERY component boundary, run ONCE, find where it breaks
# Layer 1: Entry
echo "=== Input data: ==="
echo "VALUE: ${VALUE:+SET}${VALUE:-UNSET}"

# Layer 2: Processing
echo "=== After transform: ==="
echo "RESULT: $RESULT"

# Layer 3: Output
echo "=== Final state: ==="
ls -la "$OUTPUT_PATH"
```

This reveals WHICH layer fails. Fix instrumentation first, then fix the bug.

**Trace backward through the call stack:**

Start at the symptom. At each level ask: "What called this? Where did this data come from?" Keep going until you find where the bad data originates.

When manual tracing stalls, add stack trace instrumentation:

```typescript
async function riskyOperation(directory: string) {
  const stack = new Error().stack;
  console.error('DEBUG riskyOperation:', {
    directory,
    cwd: process.cwd(),
    nodeEnv: process.env.NODE_ENV,
    stack,
  });
  // ... operation
}
```

Use `console.error()` in tests — logger may be suppressed. Log BEFORE the dangerous operation, not after it fails.

## Phase 2 — Pattern Analysis

**Find working examples.** Locate similar working code in the same codebase.

**Compare completely.** If implementing a pattern, read the reference implementation completely — every line. Don't skim.

**List every difference** between working and broken. Don't assume "that can't matter."

**Understand dependencies.** What other components, config, environment does this need?

## Phase 3 — Fix

- Form a single hypothesis — write it down: "I think X because Y"
- Make the smallest possible change. One variable at a time.
- Write a failing test that reproduces the bug
- Implement the fix, verify the test passes
- If the fix doesn't work: return to Phase 1 with new information. Don't stack another guess.

**If 3+ fixes fail: STOP.**

You're not dealing with a bug — you're dealing with an architectural problem. Each fix revealing a new problem elsewhere means you're treating symptoms. Use `AskUserQuestion` to discuss fundamentals with the user before attempting more fixes.

## Phase 4 — Harden

Make the bug structurally impossible, not just patched. Validate at EVERY layer data passes through:

**Layer 1 — Entry point validation:** Reject invalid input at the boundary
**Layer 2 — Business logic checks:** Ensure data makes sense for this operation
**Layer 3 — Environment guards:** Prevent dangerous operations in specific contexts (e.g., refuse `git init` outside tmpdir during tests)
**Layer 4 — Debug instrumentation:** Log context for forensics

All four layers are necessary. During testing, each catches bugs the others miss — different code paths bypass entry validation, mocks bypass business logic, edge cases need environment guards.

## Finding Which Test Pollutes State

Use the bisection script at @find-polluter.sh:

```bash
./find-polluter.sh '.git' 'src/**/*.test.ts'
```

Runs tests one-by-one, stops at first polluter.

## Checkpoints — return to Phase 1 when you notice

- Urge to offer the quick fix → stop, investigate more
- Urge to skip investigation → run the investigation
- Urge to guess → trace the data flow first
- Urge to try "one more fix" after 2+ attempts → step back, ask the user
- Each fix revealing a new problem → treat it as architectural, discuss with user
